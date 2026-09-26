# SQL Referential Integrity: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL referential integrity is a set of rules enforced by the database engine that ensures relationships between tables remain consistent—specifically, that a foreign key value in a child table always points to an existing primary or unique key value in a parent table, or is NULL.

**Technical Definition:** Referential integrity is a relational database constraint that guarantees the consistency of foreign key relationships by rejecting any insert, update, or delete operation that would result in a foreign key value with no matching parent key value. It is enforced through declarative `FOREIGN KEY` constraints, which specify the parent table, the referenced columns, and the referential actions (CASCADE, SET NULL, SET DEFAULT, RESTRICT, NO ACTION) to take when the parent row is modified or deleted.

**Beginner-Friendly Explanation:** Referential integrity is like a rule in a filing system that says "You cannot file a document under a client name that does not exist in your client list." If you try to delete a client, the system either prevents it, deletes all their documents too, or reassigns those documents. This keeps your data consistent and prevents "orphan" records that point to nothing.

### Key Characteristics

- **Declarative:** Referential integrity is defined as part of the table schema using `FOREIGN KEY` constraints; the database engine enforces it automatically.
- **Bidirectional protection:** It prevents both invalid inserts/updates in the child table (referencing a non-existent parent) and invalid deletes/updates in the parent table (leaving orphaned children).
- **Action-configurable:** The behavior on parent modification or deletion is configurable via `ON DELETE` and `ON UPDATE` actions.
- **NULL-aware:** Foreign key columns may contain NULL values (unless explicitly constrained with `NOT NULL`), which are considered valid because NULL means "no reference."
- **Transactional:** Referential integrity checks are evaluated within the scope of a transaction; deferred constraints can postpone validation until commit.

### Prerequisites

- Understanding of primary keys and unique constraints.
- Familiarity with `CREATE TABLE` and `ALTER TABLE` syntax.
- Knowledge of transaction concepts and isolation levels.
- Awareness of the difference between immediate and deferred constraint checking.

### Related Programming Areas

- Database schema design and normalization.
- Data integrity and data quality management.
- Application data modeling (one-to-one, one-to-many, many-to-many).
- ETL and bulk data loading optimization.
- Database migration and schema evolution.

### Core Concepts / Features

1. **Parent and Child Tables** (primary/unique targets and dependent referencing tables)
2. **Foreign Key Relationships** (one-to-one, one-to-many, many-to-many)
3. **Referential Actions** (managing anomalies through cascading actions)
4. **CASCADE** (automatic deletion or update of child rows)
5. **SET NULL** (setting child foreign key columns to NULL)
6. **SET DEFAULT** (setting child foreign key columns to defaults)
7. **RESTRICT** (immediate rejection of parent modifications)
8. **NO ACTION** (deferred checking until end of transaction)
9. **Orphan Records** (prevention, detection, cleanup)
10. **Referential Integrity Validation** (enabling/disabling constraints, NOVALIDATE states, batch-loading optimizations)


## Core Concept 1: Parent and Child Tables

### Definitions

**Core Definition:** In a foreign key relationship, the parent table holds the referenced key (usually a primary key or unique key), and the child table holds the foreign key column that references the parent.

**Technical Definition:** A parent table (also called the referenced table) contains a candidate key—a primary key or unique constraint—that serves as the target of a foreign key reference. A child table (also called the referencing table or dependent table) contains one or more columns whose values must match the values of the parent's referenced columns or be NULL. Every foreign key relationship is directional: the child references the parent, never the reverse.

**Beginner-Friendly Explanation:** Think of a parent table as a list of departments and a child table as a list of employees. Each employee belongs to a department, so the employee table (child) references the department table (parent). The department table does not need to know about the employees; it just provides the list of valid department IDs.

### Purposes

- To establish a hierarchy of tables where one table's rows depend on another's.
- To identify which table is the "source of truth" for a shared key value.
- To determine the direction of referential actions (cascades flow from parent to child).
- To design normalized schemas that eliminate data redundancy.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
-- Parent table (referenced)
CREATE TABLE parent_table (
    parent_id INT PRIMARY KEY,
    parent_name VARCHAR(100)
);

-- Child table (referencing)
CREATE TABLE child_table (
    child_id INT PRIMARY KEY,
    parent_id INT,
    child_name VARCHAR(100),
    CONSTRAINT fk_parent
        FOREIGN KEY (parent_id)
        REFERENCES parent_table (parent_id)
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `parent_table` | The referenced table; must have a primary key or unique constraint. |
| `parent_id` | The referenced column (usually the primary key). |
| `child_table` | The referencing table; contains the foreign key column. |
| `FOREIGN KEY (parent_id)` | The column(s) in the child table that reference the parent. |
| `REFERENCES parent_table (parent_id)` | Specifies the parent table and referenced column. |

#### Syntax Rules

- The parent table must have a `PRIMARY KEY` or `UNIQUE` constraint on the referenced column(s).
- The data types of the foreign key columns in the child table must be comparable to the referenced columns in the parent table.
- A table can be both a parent and a child in different relationships (e.g., `orders` references `customers` and is referenced by `order_items`).
- The referenced columns must be indexed (primary keys and unique constraints are automatically indexed).

#### Constraints and Limitations

- **Circular references:** Two tables can reference each other, but the constraints must be added in the correct order (one table must exist before its foreign key is created).
- **Self-referencing tables:** A table can reference itself (e.g., `employees.manager_id` references `employees.employee_id`).
- **Cross-database references:** Most databases do not support foreign keys across different databases; Oracle requires triggers for cross-node references.

### Annotated Code Examples

#### Example 1: PostgreSQL Parent and Child Tables

```sql
-- Create parent table
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL
);

-- Create child table referencing departments
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    dept_id INT,
    CONSTRAINT fk_dept
        FOREIGN KEY (dept_id)
        REFERENCES departments (dept_id)
);

-- Insert parent rows
INSERT INTO departments VALUES (1, 'Engineering'), (2, 'Sales');

-- Insert valid child rows
INSERT INTO employees VALUES (101, 'Alice', 1), (102, 'Bob', 2);

-- Attempt invalid child row (non-existent department)
INSERT INTO employees VALUES (103, 'Carol', 99);
```

**Expected Output (for the invalid insert):**

```
ERROR:  insert or update on table "employees" violates foreign key constraint "fk_dept"
DETAIL:  Key (dept_id)=(99) is not present in table "departments".
```

**Why This Works:** The `FOREIGN KEY` constraint enforces that every `dept_id` in `employees` must exist in `departments`. The valid inserts succeed because departments 1 and 2 exist. The invalid insert fails because department 99 does not exist.

#### Example 2: MySQL Parent and Child with Named Constraint

```sql
-- Create parent table
CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

-- Create child table with named foreign key
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category_id INT,
    CONSTRAINT fk_category
        FOREIGN KEY (category_id)
        REFERENCES categories (category_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
) ENGINE=InnoDB;
```

**Expected Output (after creation):**

```
Query OK, 0 rows affected
```

**Why This Works:** The `CONSTRAINT fk_category` names the foreign key, making it easier to reference in error messages and schema documentation. MySQL requires the `InnoDB` storage engine for foreign key support.

### Real-World Cases

- **E-commerce:** `orders` (parent) and `order_items` (child); `customers` (parent) and `orders` (child).
- **HR systems:** `departments` (parent) and `employees` (child); `employees` (parent) and `employee_dependents` (child).
- **Content management:** `articles` (parent) and `comments` (child); `users` (parent) and `articles` (child).

### References

- PostgreSQL Documentation: Foreign Keys — https://www.postgresql.org/docs/current/tutorial-fk.html
- MySQL Reference Manual: FOREIGN KEY Constraints — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: Primary and Foreign Key Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints
- Oracle Database: Data Integrity — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 2: Foreign Key Relationships

### Definitions

**Core Definition:** A foreign key relationship is the association between a child table's foreign key column and a parent table's referenced key column, defining how rows in the two tables are related.

**Technical Definition:** Foreign key relationships are physically implemented through `FOREIGN KEY` constraints. A one-to-one relationship is implemented by placing a foreign key in one table that references the primary key of the other and adding a unique constraint on the foreign key column. A one-to-many relationship is implemented by placing a foreign key in the "many" table that references the primary key of the "one" table. A many-to-many relationship requires a junction (association) table with two foreign keys—one referencing each parent table—and a composite primary key or unique constraint on the pair of foreign keys.

**Beginner-Friendly Explanation:** Relationships between tables are like different types of connections between people. One-to-one: each person has one passport. One-to-many: one teacher has many students. Many-to-many: many students take many courses—you need a separate enrollment list (junction table) to connect them.

### Purposes

- To model real-world relationships between entities in a normalized schema.
- To enforce cardinality constraints (one-to-one, one-to-many, many-to-many) at the database level.
- To enable efficient joins between related tables.
- To prevent data anomalies by ensuring relationships are always valid.

### Syntax Rules and Structure

#### Complete General Syntax (One-to-One)

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE
);

CREATE TABLE user_profiles (
    profile_id INT PRIMARY KEY,
    user_id INT UNIQUE,  -- UNIQUE enforces one-to-one
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users (user_id)
);
```

#### Complete General Syntax (One-to-Many)

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,  -- No UNIQUE constraint; many orders per customer
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers (customer_id)
);
```

#### Complete General Syntax (Many-to-Many)

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),  -- Composite key
    FOREIGN KEY (student_id) REFERENCES students (student_id),
    FOREIGN KEY (course_id) REFERENCES courses (course_id)
);
```

**Component Breakdown (Junction Table):**

| Component | Description |
|-----------|-------------|
| `student_id, course_id` | Foreign keys to the two parent tables. |
| `PRIMARY KEY (student_id, course_id)` | Composite primary key prevents duplicate enrollments. |
| `FOREIGN KEY ... REFERENCES` | Enforces referential integrity to each parent. |

#### Syntax Rules

- **One-to-one:** The foreign key column must have a `UNIQUE` constraint to enforce at most one child per parent.
- **One-to-many:** The foreign key column has no unique constraint; many child rows can reference the same parent.
- **Many-to-many:** The junction table has foreign keys to both parents and a composite primary key (or unique constraint) on the pair.
- A table can participate in multiple relationships simultaneously.

#### Constraints and Limitations

- **One-to-one enforcement:** Without the `UNIQUE` constraint on the foreign key, the relationship becomes one-to-many.
- **Junction table overhead:** Many-to-many relationships require an additional table and additional joins.
- **Composite foreign keys:** Junction tables with composite primary keys require both columns to be present in queries.

### Annotated Code Examples

#### Example 1: PostgreSQL One-to-Many Relationship

```sql
-- Parent: customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name TEXT NOT NULL
);

-- Child: orders (many orders per customer)
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_total NUMERIC(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers (customer_id)
);

-- Insert data
INSERT INTO customers (customer_name) VALUES ('Alice'), ('Bob');
INSERT INTO orders (customer_id, order_total) VALUES (1, 150.00), (1, 200.00), (2, 75.00);

-- Query with join
SELECT c.customer_name, o.order_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_name, o.order_total;
```

**Expected Output:**

```
customer_name | order_total
--------------+-------------
Alice         |      150.00
Alice         |      200.00
Bob           |       75.00
```

**Why This Works:** The `orders` table has a `customer_id` foreign key without a `UNIQUE` constraint, allowing multiple orders per customer. The join retrieves all orders for each customer.

#### Example 2: MySQL Many-to-Many Relationship

```sql
-- Parent tables
CREATE TABLE students (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    student_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE courses (
    course_id INT AUTO_INCREMENT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

-- Junction table
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students (student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses (course_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Insert data
INSERT INTO students (student_name) VALUES ('Alice'), ('Bob');
INSERT INTO courses (course_name) VALUES ('SQL Basics'), ('Data Modeling');
INSERT INTO enrollments VALUES (1, 1, '2024-01-15'), (1, 2, '2024-01-15'), (2, 1, '2024-01-16');

-- Query many-to-many
SELECT s.student_name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
ORDER BY s.student_name, c.course_name;
```

**Expected Output:**

```
student_name | course_name
-------------+--------------
Alice        | Data Modeling
Alice        | SQL Basics
Bob          | SQL Basics
```

**Why This Works:** The `enrollments` junction table connects students and courses. The composite primary key `(student_id, course_id)` prevents duplicate enrollments. `ON DELETE CASCADE` ensures that deleting a student or course also removes their enrollment records.

### Real-World Cases

- **One-to-one:** User accounts and user profiles; employees and their company-issued laptops.
- **One-to-many:** Customers and orders; departments and employees; blog posts and comments.
- **Many-to-many:** Students and courses; products and tags; actors and movies.

### References

- PostgreSQL Documentation: Foreign Keys and Many-to-Many — https://www.postgresql.org/docs/current/tutorial-fk.html
- MySQL Reference Manual: FOREIGN KEY Constraints — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: Create Foreign Key Relationships — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: Defining Relationships Between Tables — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 3: Referential Actions

### Definitions

**Core Definition:** Referential actions are the operations that the database engine performs on child rows when a referenced parent key is updated or deleted, or the checks it performs to prevent such modifications.

**Technical Definition:** Referential actions are specified in the `ON DELETE` and `ON UPDATE` clauses of a `FOREIGN KEY` constraint. The available actions are `CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, and `NO ACTION`. These actions determine whether a parent modification propagates to child rows (CASCADE, SET NULL, SET DEFAULT) or is rejected (RESTRICT, NO ACTION).

**Beginner-Friendly Explanation:** Referential actions are like the rules for what happens when you delete or change a parent record. "If I delete this department, should the employees be deleted too (CASCADE), or should their department be set to nothing (SET NULL), or should I just be prevented from deleting it (RESTRICT)?"

### Purposes

- To automate the maintenance of referential integrity without application-level code.
- To define the appropriate business rule for each relationship (e.g., deleting a customer should not delete their historical orders).
- To prevent orphan records by either cascading changes or rejecting parent modifications.
- To balance data consistency with data preservation.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
[ON DELETE { CASCADE | SET NULL | SET DEFAULT | RESTRICT | NO ACTION }]
[ON UPDATE { CASCADE | SET NULL | SET DEFAULT | RESTRICT | NO ACTION }]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ON DELETE` | Action to take when the parent row is deleted. |
| `ON UPDATE` | Action to take when the parent key value is updated. |
| `CASCADE` | Delete/update child rows automatically. |
| `SET NULL` | Set child foreign key columns to NULL. |
| `SET DEFAULT` | Set child foreign key columns to their default values. |
| `RESTRICT` | Reject the parent modification immediately. |
| `NO ACTION` | Reject at the end of the transaction (deferrable). |

#### Syntax Rules

- If neither `ON DELETE` nor `ON UPDATE` is specified, the default is `NO ACTION`.
- `SET NULL` requires the foreign key column to be nullable.
- `SET DEFAULT` requires the foreign key column to have a default value defined; the default must be a valid parent key value or NULL.
- `RESTRICT` cannot be deferred; `NO ACTION` can be deferred if the constraint is declared `DEFERRABLE`.
- In SQL Server, `CASCADE` cannot be specified if the foreign key or referenced key includes a `timestamp` column.
- `SET DEFAULT` is not supported by Oracle Database's declarative foreign key constraints; it can be implemented with triggers.

#### Constraints and Limitations

- **Multiple cascade paths:** SQL Server rejects a `FOREIGN KEY` constraint if it would create multiple cascade paths to the same table.
- **Cascade depth:** Cascading actions can trigger further cascading actions on related tables; deep chains can be difficult to debug.
- **Performance:** Cascading deletes can be slow on large tables without appropriate indexes on the foreign key columns.
- **Version-specific:** `SET DEFAULT` is not supported in Oracle; `NO ACTION` is the default in PostgreSQL and SQL Server; MySQL supports all five actions.

### Annotated Code Examples

#### Example 1: PostgreSQL CASCADE on Delete

```sql
-- Parent table
CREATE TABLE authors (
    author_id SERIAL PRIMARY KEY,
    author_name TEXT NOT NULL
);

-- Child table with CASCADE delete
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES authors (author_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Insert data
INSERT INTO authors (author_name) VALUES ('Alice'), ('Bob');
INSERT INTO books (title, author_id) VALUES ('SQL Guide', 1), ('Data Modeling', 1), ('Python 101', 2);

-- Delete an author; books are automatically deleted
DELETE FROM authors WHERE author_id = 1;
```

**Expected Output:**

```
DELETE 1
-- The two books by author 1 are also deleted
```

**Why This Works:** `ON DELETE CASCADE` propagates the delete from `authors` to `books`. When author 1 is deleted, the two books referencing that author are also deleted. `ON UPDATE CASCADE` would similarly propagate primary key changes.

#### Example 2: MySQL SET NULL on Delete

```sql
-- Parent table
CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

-- Child table with SET NULL (category_id must be nullable)
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category_id INT NULL,
    FOREIGN KEY (category_id) REFERENCES categories (category_id)
        ON DELETE SET NULL
) ENGINE=InnoDB;

-- Insert data
INSERT INTO categories (category_name) VALUES ('Electronics'), ('Books');
INSERT INTO products (product_name, category_id) VALUES ('Laptop', 1), ('Novel', 2);

-- Delete a category; product category_id becomes NULL
DELETE FROM categories WHERE category_id = 1;
```

**Expected Output:**

```
Query OK, 1 row affected
-- The 'Laptop' product now has category_id = NULL
```

**Why This Works:** `ON DELETE SET NULL` sets the child foreign key column to NULL when the parent row is deleted. The `category_id` column must be nullable for this action to work. The product row is preserved but loses its category association.

#### Example 3: SQL Server RESTRICT / NO ACTION

```sql
-- Parent table
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name NVARCHAR(100)
);

-- Child table with NO ACTION (default)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name NVARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
        ON DELETE NO ACTION
);

-- Insert data
INSERT INTO departments VALUES (1, 'Engineering');
INSERT INTO employees VALUES (101, 'Alice', 1);

-- Attempt to delete parent with children
DELETE FROM departments WHERE dept_id = 1;
```

**Expected Output:**

```
Msg 547, Level 16, State 0, Line 1
The DELETE statement conflicted with the REFERENCE constraint "FK__employees__dept__..."
The conflict occurred in database "MyDB", table "dbo.employees", column 'dept_id'.
The statement has been terminated.
```

**Why This Works:** `ON DELETE NO ACTION` (the default) rejects the delete operation because child rows exist. The error message identifies the conflicting constraint and table. `RESTRICT` would behave similarly but is checked immediately rather than at the end of the transaction.

### Real-World Cases

- **CASCADE:** Order items should be deleted when an order is deleted; comments should be deleted when a post is deleted.
- **SET NULL:** Products should remain but lose their category when a category is deleted; employees should remain but lose their manager when a manager leaves.
- **RESTRICT / NO ACTION:** Prevent deletion of a customer who has active orders; prevent deletion of a department that still has employees.
- **SET DEFAULT:** Reassign orphaned records to a default category (e.g., "Uncategorized") when a parent is deleted.

### References

- PostgreSQL Documentation: Foreign Keys (`ON DELETE`, `ON UPDATE`) — https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-REFERENCES
- MySQL Reference Manual: Referential Actions — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html#foreign-key-referential-actions
- SQL Server: Cascading Referential Integrity Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: Referential Integrity Actions — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 4: CASCADE

### Definitions

**Core Definition:** `CASCADE` is a referential action that automatically deletes or updates child rows when the corresponding parent row is deleted or its key is updated.

**Technical Definition:** When `ON DELETE CASCADE` is specified, deleting a row in the parent table causes all rows in the child table whose foreign key matches the deleted parent key to be deleted automatically. When `ON UPDATE CASCADE` is specified, updating a parent key value causes all matching child foreign key values to be updated to the new value. Cascading actions are performed as part of the same statement and transaction as the parent modification.

**Beginner-Friendly Explanation:** `CASCADE` is like a chain reaction. If you delete a folder, everything inside it gets deleted too. If you rename a folder, the paths to files inside it are automatically updated.

### Purposes

- To automatically remove dependent child rows when a parent row is deleted, preventing orphans.
- To automatically propagate primary key changes to child foreign key columns.
- To simplify application logic by delegating referential maintenance to the database.
- To model ownership relationships where child rows cannot exist without the parent.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
ON DELETE CASCADE
ON UPDATE CASCADE
```

#### Syntax Rules

- `CASCADE` can be specified for both `ON DELETE` and `ON UPDATE`, or for either one independently.
- Cascading actions are recursive: if the child table is also a parent for another table with `CASCADE`, the deletion propagates further.
- `CASCADE` cannot be used if the foreign key or referenced key includes a `timestamp` column (SQL Server restriction).
- In MySQL, cascaded foreign key actions do not activate triggers.
- In PostgreSQL, `CASCADE` cannot be deferred even if the constraint is declared `DEFERRABLE`; the cascade is performed immediately.

#### Constraints and Limitations

- **Multiple cascade paths:** SQL Server rejects a foreign key if it would create multiple cascade paths to the same table, to prevent ambiguous or infinite cascades.
- **Data loss risk:** `ON DELETE CASCADE` permanently deletes child rows; ensure this is the intended business rule.
- **Performance:** Large cascading deletes can lock many rows and consume significant transaction log space.
- **Trigger interaction:** In MySQL, cascaded actions do not fire triggers; in other databases, behavior varies.

### Annotated Code Examples

#### Example 1: PostgreSQL `ON DELETE CASCADE`

```sql
-- Parent table
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    title TEXT NOT NULL
);

-- Child table with CASCADE
CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    post_id INT NOT NULL,
    comment_text TEXT,
    FOREIGN KEY (post_id) REFERENCES posts (post_id)
        ON DELETE CASCADE
);

-- Insert data
INSERT INTO posts (title) VALUES ('First Post'), ('Second Post');
INSERT INTO comments (post_id, comment_text) VALUES
(1, 'Great post!'), (1, 'Thanks for sharing'), (2, 'Interesting read');

-- Delete a post; its comments are automatically deleted
DELETE FROM posts WHERE post_id = 1;

-- Verify
SELECT * FROM comments;
```

**Expected Output:**

```
 comment_id | post_id | comment_text
------------+---------+--------------
          3 |       2 | Interesting read
```

**Why This Works:** When post 1 is deleted, the two comments referencing post 1 are automatically deleted by the `ON DELETE CASCADE` action. The comment for post 2 remains. This models the ownership relationship: comments cannot exist without their post.

#### Example 2: MySQL `ON UPDATE CASCADE`

```sql
-- Parent table with natural key
CREATE TABLE countries (
    country_code CHAR(2) PRIMARY KEY,
    country_name VARCHAR(100)
) ENGINE=InnoDB;

-- Child table with ON UPDATE CASCADE
CREATE TABLE cities (
    city_id INT AUTO_INCREMENT PRIMARY KEY,
    city_name VARCHAR(100),
    country_code CHAR(2),
    FOREIGN KEY (country_code) REFERENCES countries (country_code)
        ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Insert data
INSERT INTO countries VALUES ('US', 'United States');
INSERT INTO cities (city_name, country_code) VALUES ('New York', 'US'), ('Los Angeles', 'US');

-- Update the parent key
UPDATE countries SET country_code = 'USA' WHERE country_code = 'US';
```

**Expected Output:**

```
Query OK, 1 row affected
-- The cities' country_code values are automatically updated to 'USA'
```

**Why This Works:** `ON UPDATE CASCADE` propagates the primary key change from `countries` to `cities`. When `'US'` is changed to `'USA'`, the two city rows are automatically updated to reference the new country code.

### Real-World Cases

- **Content management:** Deleting a blog post cascades to delete its comments.
- **E-commerce:** Deleting a shopping cart cascades to delete its line items.
- **HR systems:** Deleting an employee cascades to delete their dependents.
- **Natural key updates:** Updating a country code cascades to update all addresses referencing that country.

### References

- PostgreSQL Documentation: `ON DELETE CASCADE` — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: `CASCADE` — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: `ON DELETE CASCADE` — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- SQLite: `ON DELETE CASCADE` — https://www.sqlite.org/foreignkeys.html


## Core Concept 5: SET NULL

### Definitions

**Core Definition:** `SET NULL` is a referential action that sets the child foreign key column(s) to NULL when the corresponding parent row is deleted or its key is updated.

**Technical Definition:** When `ON DELETE SET NULL` is specified, deleting a row in the parent table causes all child rows whose foreign key matches the deleted parent key to have their foreign key column(s) set to NULL. When `ON UPDATE SET NULL` is specified, updating a parent key value causes matching child foreign key values to be set to NULL. The foreign key column(s) must be nullable for this action to be valid.

**Beginner-Friendly Explanation:** `SET NULL` is like saying "If the category is deleted, keep the product but remove its category assignment." The product is not deleted, but it is no longer associated with any category.

### Purposes

- To preserve child rows when the parent is deleted, while removing the association.
- To allow optional relationships where the parent may be removed without losing child data.
- To avoid cascading deletes when child data has independent value.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
ON DELETE SET NULL
ON UPDATE SET NULL
```

#### Syntax Rules

- The foreign key column(s) must be nullable; otherwise, the constraint creation fails.
- `SET NULL` can be specified for `ON DELETE`, `ON UPDATE`, or both.
- The action applies to all child rows whose foreign key value matches the deleted or updated parent key.
- In SQL Server, `SET NULL` is not allowed if the foreign key column is part of a primary key or has a `NOT NULL` constraint.

#### Constraints and Limitations

- **Nullability requirement:** The foreign key column must allow NULL values.
- **Application logic:** After `SET NULL` executes, the child row has no parent reference; application code must handle this case.
- **Index maintenance:** Setting foreign key columns to NULL updates the child table and all its indexes.

### Annotated Code Examples

#### Example 1: PostgreSQL `ON DELETE SET NULL`

```sql
-- Parent table
CREATE TABLE managers (
    manager_id SERIAL PRIMARY KEY,
    manager_name TEXT NOT NULL
);

-- Child table with nullable foreign key
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT NOT NULL,
    manager_id INT,  -- Nullable
    FOREIGN KEY (manager_id) REFERENCES managers (manager_id)
        ON DELETE SET NULL
);

-- Insert data
INSERT INTO managers (manager_name) VALUES ('Alice'), ('Bob');
INSERT INTO employees (emp_name, manager_id) VALUES
('Carol', 1), ('Dave', 1), ('Eve', 2);

-- Delete a manager
DELETE FROM managers WHERE manager_id = 1;

-- Verify: Carol and Dave have NULL manager_id
SELECT * FROM employees;
```

**Expected Output:**

```
 emp_id | emp_name | manager_id
--------+----------+------------
      3 | Eve      |          2
      1 | Carol    |     (null)
      2 | Dave     |     (null)
```

**Why This Works:** When manager 1 (Alice) is deleted, the `manager_id` values for Carol and Dave are set to NULL. The employees are preserved but are no longer assigned to a manager. Eve remains assigned to manager 2 (Bob).

#### Example 2: MySQL `ON UPDATE SET NULL`

```sql
-- Parent table
CREATE TABLE projects (
    project_code CHAR(5) PRIMARY KEY,
    project_name VARCHAR(100)
) ENGINE=InnoDB;

-- Child table with nullable foreign key
CREATE TABLE tasks (
    task_id INT AUTO_INCREMENT PRIMARY KEY,
    task_name VARCHAR(100),
    project_code CHAR(5) NULL,
    FOREIGN KEY (project_code) REFERENCES projects (project_code)
        ON UPDATE SET NULL
) ENGINE=InnoDB;

-- Insert data
INSERT INTO projects VALUES ('PRJ01', 'Project Alpha');
INSERT INTO tasks (task_name, project_code) VALUES ('Design', 'PRJ01'), ('Development', 'PRJ01');

-- Update the parent key
UPDATE projects SET project_code = 'PRJ02' WHERE project_code = 'PRJ01';
```

**Expected Output:**

```
Query OK, 1 row affected
-- Tasks now have project_code = NULL
```

**Why This Works:** When the project code is updated from `'PRJ01'` to `'PRJ02'`, the `ON UPDATE SET NULL` action sets the `project_code` in the `tasks` table to NULL. The tasks are preserved but are no longer associated with the renamed project.

### Real-World Cases

- **Content management:** Deleting a category sets the category of its articles to NULL, preserving the articles.
- **HR systems:** Deleting a department sets the department of its employees to NULL.
- **Inventory:** Deleting a supplier sets the supplier of its products to NULL, keeping the products.
- **Project management:** Renaming a project code sets associated tasks' project codes to NULL.

### References

- PostgreSQL Documentation: `ON DELETE SET NULL` — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: `SET NULL` — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: `SET NULL` — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: `ON DELETE SET NULL` — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 6: SET DEFAULT

### Definitions

**Core Definition:** `SET DEFAULT` is a referential action that sets the child foreign key column(s) to their predefined default value when the corresponding parent row is deleted or its key is updated.

**Technical Definition:** When `ON DELETE SET DEFAULT` is specified, deleting a row in the parent table causes all child rows whose foreign key matches the deleted parent key to have their foreign key column(s) set to the default value defined for that column. The default value must itself be a valid parent key value or NULL, otherwise the operation fails with a foreign key violation.

**Beginner-Friendly Explanation:** `SET DEFAULT` is like saying "If the preferred category is deleted, move all its products to the 'Uncategorized' category." The products are reassigned to a safe default rather than being deleted or left orphaned.

### Purposes

- To reassign child rows to a known fallback parent when the original parent is removed.
- To maintain referential integrity while preserving child data.
- To implement a "soft delete" or "reassignment" strategy without application logic.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
ON DELETE SET DEFAULT
ON UPDATE SET DEFAULT
```

#### Syntax Rules

- The foreign key column must have a `DEFAULT` value defined.
- The default value must be a valid existing parent key value or NULL; otherwise, the parent modification fails with a foreign key violation.
- `SET DEFAULT` is supported by PostgreSQL, MySQL, and SQL Server.
- Oracle Database does **not** support `SET DEFAULT` in declarative foreign key constraints; it can be implemented with triggers.

#### Constraints and Limitations

- **Oracle limitation:** `SET DEFAULT` is not available as a declarative referential action in Oracle; use triggers instead.
- **Default must be valid:** If the default value does not exist in the parent table, the delete/update operation fails.
- **NULL default:** If the default is NULL, `SET DEFAULT` behaves like `SET NULL`.

### Annotated Code Examples

#### Example 1: PostgreSQL `ON DELETE SET DEFAULT`

```sql
-- Parent table with a default category
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name TEXT NOT NULL
);

-- Child table with default foreign key
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name TEXT NOT NULL,
    category_id INT DEFAULT 0,  -- Default to 'Uncategorized'
    FOREIGN KEY (category_id) REFERENCES categories (category_id)
        ON DELETE SET DEFAULT
);

-- Insert data including the default category
INSERT INTO categories VALUES (0, 'Uncategorized'), (1, 'Electronics'), (2, 'Books');
INSERT INTO products (product_name, category_id) VALUES
('Laptop', 1), ('Novel', 2), ('Mystery Box', 1);

-- Delete the Electronics category
DELETE FROM categories WHERE category_id = 1;
```

**Expected Output:**

```
DELETE 1
-- Laptop and Mystery Box now have category_id = 0 (Uncategorized)
```

**Why This Works:** When the Electronics category (id 1) is deleted, the `ON DELETE SET DEFAULT` action reassigns the two products to category 0 (Uncategorized). The products are preserved and remain referentially valid.

#### Example 2: SQL Server `SET DEFAULT`

```sql
-- Parent table
CREATE TABLE regions (
    region_id INT PRIMARY KEY,
    region_name NVARCHAR(100)
);

-- Child table with default region
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name NVARCHAR(100),
    region_id INT DEFAULT 0,
    CONSTRAINT fk_region
        FOREIGN KEY (region_id) REFERENCES regions (region_id)
        ON DELETE SET DEFAULT
);

INSERT INTO regions VALUES (0, 'Unassigned'), (1, 'North'), (2, 'South');
INSERT INTO customers (customer_id, customer_name, region_id)
VALUES (101, 'Alice', 1), (102, 'Bob', 2), (103, 'Carol', 1);

-- Delete the North region
DELETE FROM regions WHERE region_id = 1;
```

**Expected Output:**

```
(1 row affected)
-- Alice and Carol now have region_id = 0
```

**Why This Works:** SQL Server supports `SET DEFAULT` as a referential action. When region 1 (North) is deleted, customers Alice and Carol are automatically reassigned to region 0 (Unassigned).

### Real-World Cases

- **Product categorization:** Reassigning products to "Uncategorized" when a category is deleted.
- **Customer regions:** Reassigning customers to a default region when a region is discontinued.
- **Employee departments:** Reassigning employees to a default department when a department is closed.
- **Content tagging:** Reassigning articles to a default tag when a tag is removed.

### References

- PostgreSQL Documentation: `ON DELETE SET DEFAULT` — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: `SET DEFAULT` — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: `SET DEFAULT` — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: Referential Integrity Actions — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 7: RESTRICT

### Definitions

**Core Definition:** `RESTRICT` is a referential action that immediately rejects any attempt to delete or update a parent row if dependent child rows exist, without deferring the check.

**Technical Definition:** When `ON DELETE RESTRICT` or `ON UPDATE RESTRICT` is specified, the database engine checks for the existence of child rows referencing the parent row before performing the operation. If any child rows exist, the operation is rejected immediately, even if the constraint is declared `DEFERRABLE`. `RESTRICT` is the strictest referential action; it is equivalent to `NO ACTION` but with immediate checking.

**Beginner-Friendly Explanation:** `RESTRICT` is like a locked gate. If there are child records depending on a parent record, you simply cannot delete or change the parent. The database says "No" right away, before doing anything else.

### Purposes

- To strictly prevent parent modifications that would leave orphans.
- To enforce a "no deletion if dependents exist" business rule.
- To provide immediate feedback (unlike `NO ACTION`, which may defer the check).

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
ON DELETE RESTRICT
ON UPDATE RESTRICT
```

#### Syntax Rules

- `RESTRICT` checks immediately when the parent modification is attempted, not at the end of the transaction.
- `RESTRICT` cannot be deferred even if the constraint is declared `DEFERRABLE`.
- `RESTRICT` is equivalent to `NO ACTION` in most databases, except that `NO ACTION` can be deferred.
- In PostgreSQL, `RESTRICT` and `NO ACTION` are the same except for deferrability.

#### Constraints and Limitations

- **Immediate check:** Cannot be deferred; useful for interactive applications requiring immediate feedback.
- **No cascade:** `RESTRICT` does not modify child rows; it only rejects the parent operation.
- **MySQL:** `RESTRICT` and `NO ACTION` behave identically in MySQL.

### Annotated Code Examples

#### Example 1: PostgreSQL `RESTRICT`

```sql
-- Parent table
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name TEXT NOT NULL
);

-- Child table with RESTRICT
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT NOT NULL,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
        ON DELETE RESTRICT
);

-- Insert data
INSERT INTO departments (dept_name) VALUES ('Engineering');
INSERT INTO employees (emp_name, dept_id) VALUES ('Alice', 1);

-- Attempt to delete the department
DELETE FROM departments WHERE dept_id = 1;
```

**Expected Output:**

```
ERROR:  update or delete on table "departments" violates foreign key constraint "employees_dept_id_fkey" on table "employees"
DETAIL:  Key (dept_id)=(1) is still referenced from table "employees".
```

**Why This Works:** The `ON DELETE RESTRICT` action immediately rejects the delete because Alice still references department 1. The error message identifies the constraint and the child table.

#### Example 2: MySQL `RESTRICT`

```sql
-- Parent table
CREATE TABLE authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    author_name VARCHAR(100)
) ENGINE=InnoDB;

-- Child table with RESTRICT
CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES authors (author_id)
        ON DELETE RESTRICT
) ENGINE=InnoDB;

INSERT INTO authors (author_name) VALUES ('Alice');
INSERT INTO books (title, author_id) VALUES ('SQL Guide', 1);

-- Attempt to delete the author
DELETE FROM authors WHERE author_id = 1;
```

**Expected Output:**

```
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails
```

**Why This Works:** MySQL's `RESTRICT` action prevents the deletion because the `books` table still references the author. The error code 1451 indicates a foreign key constraint violation.

### Real-World Cases

- **Financial systems:** Prevent deletion of a customer who has outstanding invoices.
- **HR systems:** Prevent deletion of a department that still has employees.
- **Inventory:** Prevent deletion of a supplier who still has active products.
- **Content management:** Prevent deletion of a category that still has articles.

### References

- PostgreSQL Documentation: `RESTRICT` — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: `RESTRICT` — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: `NO ACTION` (equivalent to `RESTRICT`) — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: `ON DELETE RESTRICT` — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 8: NO ACTION

### Definitions

**Core Definition:** `NO ACTION` is a referential action that rejects a parent modification if dependent child rows exist, with the check deferred to the end of the transaction if the constraint is declared `DEFERRABLE`.

**Technical Definition:** When `ON DELETE NO ACTION` or `ON UPDATE NO ACTION` is specified, the database engine checks for child rows referencing the parent row. If the constraint is immediate (the default), the check occurs at the end of the statement. If the constraint is declared `DEFERRABLE INITIALLY DEFERRED`, the check is postponed until the end of the transaction. `NO ACTION` is the default referential action when neither `ON DELETE` nor `ON UPDATE` is specified.

**Beginner-Friendly Explanation:** `NO ACTION` is like `RESTRICT`, but it can wait. If you have a transaction that temporarily violates referential integrity, `NO ACTION` gives you until the end of the transaction to fix it. `RESTRICT` would reject it immediately.

### Purposes

- To allow temporary referential integrity violations within a transaction that are resolved before commit.
- To provide the default behavior for foreign keys (rejecting invalid parent modifications).
- To support complex transactions where child rows are re-parented before the transaction completes.

### Syntax Rules and Structure

#### Complete General Syntax

```sql
FOREIGN KEY (child_column)
REFERENCES parent_table (parent_column)
ON DELETE NO ACTION
ON UPDATE NO ACTION
DEFERRABLE INITIALLY DEFERRED  -- Optional: defer check to commit
```

#### Syntax Rules

- `NO ACTION` is the default if no `ON DELETE` or `ON UPDATE` clause is specified.
- If the constraint is `DEFERRABLE INITIALLY DEFERRED`, the check is postponed until `COMMIT`.
- If the constraint is immediate (the default), the check occurs at the end of the statement.
- `NO ACTION` does not modify child rows; it only rejects the parent operation if the check fails.

#### Constraints and Limitations

- **Deferred constraint requires declaration:** To defer the check, the constraint must be declared `DEFERRABLE INITIALLY DEFERRED`.
- **Transaction scope:** Deferred checks are evaluated at `COMMIT`; if the violation is not resolved, the transaction rolls back.
- **Performance:** Deferred checks can delay error detection, making debugging harder.

### Annotated Code Examples

#### Example 1: PostgreSQL `NO ACTION` with Deferred Check

```sql
-- Parent table
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name TEXT NOT NULL
);

-- Child table with DEFERRABLE NO ACTION
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT NOT NULL,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
        ON DELETE NO ACTION
        DEFERRABLE INITIALLY DEFERRED
);

-- Insert data
INSERT INTO departments (dept_name) VALUES ('Engineering');
INSERT INTO employees (emp_name, dept_id) VALUES ('Alice', 1);

-- Begin transaction
BEGIN;

-- Delete the department (temporarily violates integrity)
DELETE FROM departments WHERE dept_id = 1;

-- Reassign the employee before commit
UPDATE employees SET dept_id = NULL WHERE dept_id = 1;

-- Commit succeeds because the violation was resolved
COMMIT;
```

**Expected Output:**

```
BEGIN
DELETE 1
UPDATE 1
COMMIT
```

**Why This Works:** The constraint is declared `DEFERRABLE INITIALLY DEFERRED`, so the `DELETE` does not immediately fail even though Alice still references department 1. The subsequent `UPDATE` sets Alice's `dept_id` to NULL, resolving the violation before `COMMIT`. The transaction commits successfully.

#### Example 2: SQL Server `NO ACTION` (Default)

```sql
-- Parent table
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name NVARCHAR(100)
);

-- Child table with NO ACTION (default)
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name NVARCHAR(100),
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories (category_id)
);

INSERT INTO categories VALUES (1, 'Electronics');
INSERT INTO products VALUES (101, 'Laptop', 1);

-- Attempt to delete the category
DELETE FROM categories WHERE category_id = 1;
```

**Expected Output:**

```
Msg 547, Level 16, State 0, Line 1
The DELETE statement conflicted with the REFERENCE constraint "FK__products__category..."
The conflict occurred in database "MyDB", table "dbo.products", column 'category_id'.
The statement has been terminated.
```

**Why This Works:** SQL Server's default `NO ACTION` rejects the delete because products still reference the category. The check is immediate (at the end of the statement) because the constraint is not declared deferrable. SQL Server does not support deferred constraints.

### Real-World Cases

- **Batch re-parenting:** Temporarily deleting a parent, reassigning children, and re-inserting the parent within a single transaction.
- **Data migration:** Moving rows between parents in a transaction where temporary orphans are allowed.
- **Complex updates:** Updating primary keys and foreign keys in a transaction where the intermediate state violates referential integrity.

### References

- PostgreSQL Documentation: `NO ACTION` and Deferrable Constraints — https://www.postgresql.org/docs/current/sql-createtable.html
- MySQL Reference Manual: `NO ACTION` — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: `NO ACTION` (Default) — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: `ON DELETE NO ACTION` — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 9: Orphan Records

### Definitions

**Core Definition:** Orphan records are rows in a child table whose foreign key value does not match any existing primary or unique key value in the parent table, violating referential integrity.

**Technical Definition:** An orphan record occurs when a child row references a parent key that has been deleted or updated without a corresponding referential action to maintain consistency. Orphans can arise from disabled constraints, direct data manipulation bypassing constraints, or data loading errors. They are detected using `LEFT JOIN ... WHERE parent.key IS NULL` or `NOT EXISTS` subqueries, and cleaned up by either deleting the orphan rows, reassigning them to a valid parent, or re-inserting the missing parent.

**Beginner-Friendly Explanation:** An orphan record is like a library book that references a borrower who no longer exists in the system. The book record points to a borrower ID that has been deleted. Orphan records cause data quality problems and should be detected and cleaned up.

### Purposes

- To identify data quality issues caused by disabled constraints or improper data loading.
- To clean up orphan rows before re-enabling referential integrity constraints.
- To audit the completeness of referential integrity in legacy databases.
- To prevent application errors caused by missing parent references.

### Syntax Rules and Structure

#### Complete General Syntax (Orphan Detection)

```sql
SELECT child.*
FROM child_table child
LEFT JOIN parent_table parent ON child.parent_id = parent.parent_id
WHERE parent.parent_id IS NULL;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `LEFT JOIN` | Includes all child rows; parent columns are NULL for non-matching rows. |
| `WHERE parent.parent_id IS NULL` | Filters to child rows with no matching parent. |

#### Complete General Syntax (Orphan Cleanup — Delete)

```sql
DELETE FROM child_table
WHERE parent_id NOT IN (SELECT parent_id FROM parent_table);
```

#### Complete General Syntax (Orphan Cleanup — Reassign)

```sql
UPDATE child_table
SET parent_id = 0  -- Default parent
WHERE parent_id NOT IN (SELECT parent_id FROM parent_table);
```

#### Syntax Rules

- Orphan detection should be run before re-enabling a disabled foreign key constraint.
- `NOT EXISTS` is generally more efficient than `NOT IN` for large tables, especially when NULLs are involved.
- Cleanup actions (delete or reassign) should be performed within a transaction with appropriate backups.
- After cleanup, re-validate the constraint to ensure no orphans remain.

#### Constraints and Limitations

- **NOT IN with NULLs:** `NOT IN` returns no rows if the subquery contains NULL; use `NOT EXISTS` or `LEFT JOIN` instead.
- **Performance:** Orphan detection on large tables without indexes on the foreign key column can be slow.
- **Data loss:** Deleting orphans is irreversible; always back up before cleanup.
- **Reassignment validity:** Reassigned values must exist in the parent table, or the orphan problem persists.

### Annotated Code Examples

#### Example 1: PostgreSQL Orphan Detection with LEFT JOIN

```sql
-- Create tables with disabled constraint to allow orphans
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name TEXT
);

CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    emp_name TEXT,
    dept_id INT
);

-- Insert data including an orphan
INSERT INTO departments VALUES (1, 'Engineering'), (2, 'Sales');
INSERT INTO employees (emp_name, dept_id) VALUES
('Alice', 1), ('Bob', 2), ('Carol', 99);  -- 99 is an orphan

-- Detect orphans
SELECT e.emp_id, e.emp_name, e.dept_id
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

**Expected Output:**

```
 emp_id | emp_name | dept_id
--------+----------+---------
      3 | Carol    |      99
```

**Why This Works:** The `LEFT JOIN` includes all employees. `WHERE d.dept_id IS NULL` filters to employees whose `dept_id` does not match any department. Carol's `dept_id` of 99 has no matching department, making her an orphan.

#### Example 2: MySQL Orphan Cleanup by Deletion

```sql
-- Detect and delete orphans
DELETE e
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE e.dept_id IS NOT NULL AND d.dept_id IS NULL;

-- Verify no orphans remain
SELECT COUNT(*) AS orphan_count
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE e.dept_id IS NOT NULL AND d.dept_id IS NULL;
```

**Expected Output:**

```
Query OK, 1 row affected

orphan_count
------------
           0
```

**Why This Works:** The `DELETE ... LEFT JOIN` syntax in MySQL deletes only the orphan rows (Carol) while preserving valid rows (Alice and Bob). The verification query confirms that no orphans remain.

#### Example 3: SQL Server Orphan Reassignment

```sql
-- Reassign orphans to a default department
UPDATE employees
SET dept_id = 0  -- Default 'Unassigned' department
WHERE dept_id NOT IN (SELECT dept_id FROM departments WHERE dept_id IS NOT NULL)
  AND dept_id IS NOT NULL;

-- Verify
SELECT * FROM employees;
```

**Expected Output:**

```
emp_id | emp_name | dept_id
-------+----------+---------
1      | Alice    | 1
2      | Bob      | 2
3      | Carol    | 0
```

**Why This Works:** The `UPDATE` reassigns orphaned employees to department 0 (which must exist in the `departments` table). The `NOT IN` subquery excludes NULLs to avoid the `NOT IN` with NULL pitfall. Carol's `dept_id` is changed from 99 to 0.

### Real-World Cases

- **Legacy database migration:** Detecting orphans in a legacy database before applying foreign key constraints.
- **Data import validation:** Checking for orphans after a bulk data import that bypassed constraints.
- **ETL data quality:** Auditing source data for referential integrity violations before loading into a warehouse.
- **Application error investigation:** Diagnosing "parent not found" errors caused by orphan records.

### References

- PostgreSQL Documentation: Foreign Keys and Data Integrity — https://www.postgresql.org/docs/current/ddl-constraints.html
- MySQL Reference Manual: Foreign Key Constraint Checks — https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html
- SQL Server: Foreign Key Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Oracle Database: Data Integrity — https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html


## Core Concept 10: Referential Integrity Validation

### Definitions

**Core Definition:** Referential integrity validation is the process of enabling, disabling, or deferring foreign key constraint checks to balance data integrity enforcement with performance during bulk data operations.

**Technical Definition:** Constraint validation has two independent properties: enforcement (whether new data is checked) and validation (whether existing data is scanned). A constraint can be `ENABLED` (enforced) and `VALIDATED` (existing data verified), `ENABLED NOVALIDATE` (enforced for new data only), or `DISABLED` (not enforced). For bulk loading, constraints are often disabled, data is loaded, orphans are cleaned up, and constraints are re-enabled with validation. Oracle's `NOVALIDATE` and PostgreSQL's `NOT VALID` allow enabling a constraint without scanning existing data.

**Beginner-Friendly Explanation:** When you are loading a huge amount of data, checking every row for referential integrity can be very slow. So you temporarily turn off the checks (disable constraints), load the data, fix any problems, and then turn the checks back on. This is much faster than checking every row as it is inserted.

### Purposes

- To speed up bulk data loading by temporarily disabling constraint checks.
- To enable constraints on tables with existing data without a full table scan using `NOVALIDATE`.
- To validate existing data after cleanup using `VALIDATE CONSTRAINT`.
- To manage referential integrity in data warehouse ETL pipelines.

### Syntax Rules and Structure

#### Complete General Syntax (Oracle: Enable/Disable)

```sql
-- Disable a constraint (no enforcement, no validation)
ALTER TABLE table_name DISABLE CONSTRAINT constraint_name;

-- Enable with validation (enforces new data, validates existing)
ALTER TABLE table_name ENABLE CONSTRAINT constraint_name;

-- Enable without validation (enforces new data, skips existing)
ALTER TABLE table_name ENABLE NOVALIDATE CONSTRAINT constraint_name;
```

#### Complete General Syntax (PostgreSQL: NOT VALID)

```sql
-- Add a constraint without validating existing data
ALTER TABLE table_name
ADD CONSTRAINT constraint_name
FOREIGN KEY (column) REFERENCES parent_table (parent_column)
NOT VALID;

-- Validate existing data later
ALTER TABLE table_name VALIDATE CONSTRAINT constraint_name;
```

#### Complete General Syntax (SQL Server: NOCHECK / CHECK)

```sql
-- Disable a constraint
ALTER TABLE table_name NOCHECK CONSTRAINT constraint_name;

-- Enable a constraint without validating existing data
ALTER TABLE table_name CHECK CONSTRAINT constraint_name;
```

#### Syntax Rules

- **Oracle:** `ENABLE NOVALIDATE` enforces the constraint for new DML but does not validate existing rows. `VALIDATE` scans existing rows and raises an error if violations exist.
- **PostgreSQL:** `NOT VALID` is equivalent to Oracle's `ENABLE NOVALIDATE`. `VALIDATE CONSTRAINT` performs the full validation scan.
- **SQL Server:** `NOCHECK` disables enforcement; `CHECK` re-enables it. SQL Server does not have a built-in `NOVALIDATE` for existing data.
- **MySQL:** Foreign key checks can be disabled globally with `SET foreign_key_checks = 0` for a session.

#### Constraints and Limitations

- **Orphan risk:** Disabling constraints allows orphans to be created; they must be cleaned up before re-enabling with validation.
- **Dependency rules:** A `PRIMARY KEY` or `UNIQUE` constraint cannot be disabled if an enabled `FOREIGN KEY` references it.
- **Performance:** Validating a constraint scans the entire child table; on very large tables, this can take significant time.
- **Version-specific:** PostgreSQL's `NOT VALID` is available in versions 9.2+; Oracle's `ENABLE NOVALIDATE` has been available since Oracle 8i.

### Annotated Code Examples

#### Example 1: Oracle Bulk Load with NOVALIDATE

```sql
-- Disable the foreign key constraint before bulk load
ALTER TABLE employees DISABLE CONSTRAINT fk_dept;

-- Bulk load data (fast, no constraint checks)
-- ... perform bulk insert ...

-- Enable the constraint without validating existing data
ALTER TABLE employees ENABLE NOVALIDATE CONSTRAINT fk_dept;

-- Later, validate existing data after cleanup
ALTER TABLE employees VALIDATE CONSTRAINT fk_dept;
```

**Expected Output (for `VALIDATE CONSTRAINT`):**

```
Table altered.
```

**Why This Works:** `ENABLE NOVALIDATE` enables the constraint for new DML operations without scanning the existing table, which is fast. `VALIDATE CONSTRAINT` performs the full scan later, after any orphans have been cleaned up. This two-phase approach is standard for Oracle bulk loading.

#### Example 2: PostgreSQL Bulk Load with NOT VALID

```sql
-- Add a foreign key constraint without validating existing data
ALTER TABLE employees
ADD CONSTRAINT fk_dept
FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
NOT VALID;

-- Clean up any orphans
DELETE FROM employees
WHERE dept_id IS NOT NULL
  AND dept_id NOT IN (SELECT dept_id FROM departments WHERE dept_id IS NOT NULL);

-- Validate the constraint
ALTER TABLE employees VALIDATE CONSTRAINT fk_dept;
```

**Expected Output:**

```
ALTER TABLE
DELETE 0
ALTER TABLE
```

**Why This Works:** `NOT VALID` allows the constraint to be added quickly without scanning existing rows. The cleanup query removes any orphans, and `VALIDATE CONSTRAINT` then performs the full scan to confirm that all existing rows satisfy the constraint.

#### Example 3: MySQL Session-Level Constraint Disable

```sql
-- Disable foreign key checks for this session
SET foreign_key_checks = 0;

-- Bulk load data (fast, no FK checks)
LOAD DATA INFILE '/data/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ',';

-- Re-enable foreign key checks
SET foreign_key_checks = 1;

-- Check for orphans that were created during load
SELECT COUNT(*) AS orphan_count
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE e.dept_id IS NOT NULL AND d.dept_id IS NULL;
```

**Expected Output:**

```
Query OK, 0 rows affected
Query OK, 10000 rows affected
Query OK, 0 rows affected

orphan_count
------------
           0
```

**Why This Works:** MySQL's `SET foreign_key_checks = 0` disables FK checks for the session, allowing faster bulk loading. After re-enabling, the orphan check confirms that no violations occurred. If orphans were found, they would need to be cleaned up before the constraint could be fully trusted.

### Real-World Cases

- **Data warehouse ETL:** Disabling constraints during nightly loads, then re-enabling and validating after cleanup.
- **Legacy migration:** Enabling constraints on a migrated database without scanning millions of existing rows, then validating later.
- **Bulk import:** Loading CSV files with millions of rows using `LOAD DATA` or `COPY` with constraints disabled.
- **Schema evolution:** Adding a new foreign key to a large existing table using `NOT VALID` to avoid a long lock.

### References

- Oracle Database: Enabling and Disabling Constraints — https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/constraint.html
- PostgreSQL Documentation: `ADD CONSTRAINT ... NOT VALID` — https://www.postgresql.org/docs/current/sql-altertable.html
- MySQL Reference Manual: `foreign_key_checks` — https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_foreign_key_checks
- SQL Server: Disable Foreign Key Constraints — https://learn.microsoft.com/en-us/sql/relational-databases/tables/disable-foreign-key-constraints


## Summary Table: Referential Actions Comparison

| Action | ON DELETE Behavior | ON UPDATE Behavior | Nullable FK Required | Deferrable |
|--------|-------------------|-------------------|---------------------|------------|
| `CASCADE` | Delete child rows | Update child FK values | No | No |
| `SET NULL` | Set child FK to NULL | Set child FK to NULL | Yes | No |
| `SET DEFAULT` | Set child FK to default | Set child FK to default | No (default must be valid) | No |
| `RESTRICT` | Reject immediately | Reject immediately | No | No |
| `NO ACTION` | Reject (can defer) | Reject (can defer) | No | Yes (if declared) |


## Final Notes on Deprecated and Unsafe Features

- **Oracle `SET DEFAULT`:** Not supported as a declarative foreign key action; implement with triggers instead.
- **MySQL cascaded actions and triggers:** Cascaded foreign key actions do not activate triggers in MySQL.
- **SQL Server multiple cascade paths:** SQL Server rejects a foreign key if it would create multiple cascade paths to the same table.
- **`NOT IN` with NULLs:** `NOT IN` returns no rows if the subquery contains NULL; use `NOT EXISTS` or `LEFT JOIN` for orphan detection.
- **Disabling constraints:** Disabling foreign key constraints allows orphans to be created; always clean up before re-enabling with validation.
- **Version-specific:** PostgreSQL `NOT VALID` (9.2+), Oracle `ENABLE NOVALIDATE` (8i+), MySQL `foreign_key_checks` (all versions), SQL Server `NOCHECK` (all versions).
- **Unsafe if misused:** `ON DELETE CASCADE` permanently deletes child rows; ensure this is the intended business rule before applying it.