# Relational Database Concepts

The relational model is the theoretical foundation of modern relational databases. Understanding its formal concepts — relations, tuples, attributes, domains, keys, and integrity rules — is essential for designing sound schemas and writing correct SQL.

---

## 1. The Relational Model

The **relational model** is a mathematical approach to organizing data, proposed by **Dr. Edgar F. Codd** in 1970 in his landmark paper *"A Relational Model of Data for Large Shared Data Banks."*

### Core Idea

Data is represented as **relations** (tables) — sets of **tuples** (rows) — with each tuple composed of **attributes** (columns). Relationships between tables are expressed through **shared attribute values**, not through physical pointers.

### Foundations

The relational model is grounded in **set theory** and **first-order predicate logic**:

| Mathematical Concept | Relational Equivalent |
|---|---|
| Set | Table (relation) |
| Element / member | Row (tuple) |
| Cartesian product | All possible row combinations |
| Predicate | Selection condition |
| Projection | Column selection |

### Why the Relational Model Won

Before Codd's model, databases were **navigational** (hierarchical, network) — programmers had to know physical storage paths to retrieve data. The relational model offered:

- **Logical data independence** — queries don't depend on physical storage
- **Declarative querying** — describe *what*, not *how*
- **Mathematical rigor** — provable properties, normalization theory
- **Ad hoc queries** — users can query without pre-planning access paths
- **Simpler data representation** — one uniform structure (the table)

### Principles of the Relational Model

| Principle | Meaning |
|---|---|
| **Relations are sets** | Rows are unordered; no duplicate rows |
| **Value-based relationships** | Links via data values, not pointers |
| **Physical independence** | Logical schema independent of storage |
| **Integrity rules** | Entity integrity and referential integrity |
| **Closed operations** | Query results are themselves relations |

---

## 2. Relations

A **relation** is the formal term for a **table**. It is a **set of tuples** (rows) that share the same set of attributes (columns).

### Properties of a Relation

| Property | Description |
|---|---|
| **Unique name** | Each relation has a distinct name within its schema |
| **Atomic values** | Each cell holds a single, indivisible value (1NF) |
| **No duplicate tuples** | The set has no repeated rows |
| **Unordered tuples** | Row order is irrelevant |
| **Unordered attributes** | Column order is irrelevant |
| **Distinct attribute names** | No two columns share a name |

### Relation Schema vs. Relation Instance

| Concept | Definition | Example |
|---|---|---|
| **Relation schema** | The structure (name + attributes + domains) | `Customer(id, name, email)` |
| **Relation instance** | The current set of tuples | The rows currently in `Customer` |

### Relation vs. Table: Terminology

| Formal (Relational Model) | Informal (SQL) |
|---|---|
| Relation | Table |
| Tuple | Row / Record |
| Attribute | Column / Field |
| Domain | Data type |
| Relation schema | Table definition |
| Relation instance | Table data |
| Cardinality | Number of rows |
| Degree | Number of columns |

### Example

**Relation schema:**
```
Employee(id, first_name, last_name, hire_date, salary)
```

**Relation instance:**

| id | first_name | last_name | hire_date | salary |
|---|---|---|---|---|
| 1 | John | Cruz | 2020-01-15 | 55000.00 |
| 2 | Maria | Santos | 2021-03-22 | 62000.00 |
| 3 | Ahmed | Khan | 2019-11-01 | 71000.00 |

The relation has **degree 5** (5 attributes) and **cardinality 3** (3 tuples).

### Relations Are Sets

Because a relation is a **set**:

- **No duplicate rows** — each tuple is unique
- **No defined order** — rows and columns are unordered

In practice, SQL tables *allow* duplicates unless a primary key or `UNIQUE` constraint prevents them, and SQL *does* preserve insertion order for retrieval (though relying on it is discouraged).

---

## 3. Tuples

A **tuple** is the formal term for a **row** (or record) in a relation. It represents a **single entity** or instance.

### Structure

A tuple is an **ordered set of attribute values**, one for each attribute in the relation schema:

```
Tuple = (v₁, v₂, v₃, ..., vₙ)
```

Where `vᵢ` is the value for attribute `i`, and `n` is the degree of the relation.

### Example

For the schema `Employee(id, first_name, last_name, hire_date, salary)`:

```
(1, 'John', 'Cruz', '2020-01-15', 55000.00)
```

### Properties of Tuples

| Property | Description |
|---|---|
| **Atomic values** | Each value is indivisible (no nested tables, no arrays in the pure model) |
| **Domain-conforming** | Each value belongs to its attribute's domain |
| **Nullable** | A value may be NULL (unknown / not applicable) |
| **Unique** | No two tuples are identical in a valid relation |
| **Unordered** | Tuples have no inherent order |

### Tuple Terminology Across Systems

| Term | Origin |
|---|---|
| **Tuple** | Relational model (mathematics) |
| **Row** | SQL / informal |
| **Record** | File systems / COBOL |

### NULL in Tuples

A `NULL` represents **unknown** or **inapplicable** — not zero, not empty string, not false.

```sql
-- A tuple with NULL salary (unknown)
(4, 'Ana', 'Reyes', '2022-05-10', NULL)
```

`NULL` interacts specially with comparisons:

```sql
SELECT * FROM employee WHERE salary = NULL;   -- returns nothing
SELECT * FROM employee WHERE salary IS NULL;  -- correct
```

---

## 4. Attributes

An **attribute** is the formal term for a **column** in a relation. It represents a **single property** or characteristic of the entity the relation describes.

### Structure

Each attribute has:

- **A name** — unique within the relation
- **A domain** — the set of allowed values
- **A position** — order within the schema (conceptually irrelevant)

### Attributes and Their Domains

| Attribute | Possible Domain |
|---|---|
| `id` | Positive integers |
| `first_name` | Character strings (1–50 chars) |
| `email` | Valid email strings |
| `hire_date` | Valid dates |
| `salary` | Non-negative decimal numbers |

### Attribute Types in SQL

| Category | SQL Types |
|---|---|
| Numeric | `INTEGER`, `BIGINT`, `NUMERIC`, `DECIMAL`, `REAL` |
| Character | `CHAR`, `VARCHAR`, `TEXT` |
| Date/Time | `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL` |
| Boolean | `BOOLEAN` |
| Binary | `BYTEA`, `BLOB` |
| Special | `UUID`, `JSON`, `XML`, `ARRAY` |

### Defining Attributes

```sql
CREATE TABLE employee (
  id          SERIAL PRIMARY KEY,
  first_name  VARCHAR(50)  NOT NULL,
  last_name   VARCHAR(50)  NOT NULL,
  email       VARCHAR(255) UNIQUE NOT NULL,
  hire_date   DATE         NOT NULL DEFAULT CURRENT_DATE,
  salary      NUMERIC(10,2) CHECK (salary >= 0)
);
```

### Attribute vs. Column vs. Field

| Term | Meaning |
|---|---|
| **Attribute** | Relational model term |
| **Column** | SQL term |
| **Field** | Flat-file / file-system term |

### Simple vs. Composite Attributes

- **Simple attribute** — indivisible (e.g., `age`)
- **Composite attribute** — conceptually composed of parts (e.g., `address` = street + city + postal code)

In the relational model, **composite attributes are flattened** into separate simple attributes:

```
❌ address (composite)
✅ street, city, postal_code (simple attributes)
```

This is part of **First Normal Form (1NF)**.

---

## 5. Domains

A **domain** is the **set of all possible values** an attribute can hold. It defines the "universe of discourse" for that attribute.

### Formal Definition

> A domain `D` is a set of atomic values. Each attribute `A` in a relation schema is associated with a domain `dom(A)`.

### Why Domains Matter

- **Type safety** — values must come from the correct domain
- **Semantic clarity** — `hire_date` should be a date, not a string
- **Data integrity** — domains prevent nonsensical values
- **Query optimization** — the DBMS knows what operations are valid

### Domain Examples

| Attribute | Domain |
|---|---|
| `employee_id` | Positive integers |
| `gender` | {`M`, `F`, `X`} |
| `hire_date` | Valid Gregorian dates |
| `salary` | Non-negative decimals with 2 places |
| `email` | Strings matching an email regex |

### Domains in SQL

SQL does not implement domains as strictly as the relational model. Instead:

- **Data types** approximate domains (`INTEGER`, `VARCHAR(50)`)
- **CHECK constraints** restrict values further
- **User-defined types (UDTs)** and **DOMAIN objects** (PostgreSQL) refine them

**PostgreSQL example:**
```sql
CREATE DOMAIN email_address AS VARCHAR(255)
  CHECK (VALUE ~ '^[^@]+@[^@]+\.[^@]+$');

CREATE TABLE customers (
  id    SERIAL PRIMARY KEY,
  email email_address NOT NULL
);
```

**SQL Server example:**
```sql
CREATE TYPE phone_number FROM VARCHAR(20) NOT NULL;
```

### Domain vs. Data Type

| Aspect | Domain | Data Type |
|---|---|---|
| Abstraction level | Higher — semantic meaning | Lower — storage format |
| Scope | Relational model concept | SQL implementation concept |
| Enforcement | Ideal: strict | Practical: via CHECK, types |
| Example | "Valid email" | `VARCHAR(255)` |

### Domain Constraints

A domain constraint limits values to a subset of the domain:

- **NOT NULL** — excludes NULL from the domain
- **CHECK** — enforces a predicate
- **UNIQUE** — imposes a relation-wide property
- **FOREIGN KEY** — restricts to values present in another relation

### Domain Compatibility

In the pure relational model, attributes can be compared only if they share the same domain. SQL relaxes this (implicit conversions), which can cause subtle bugs.

**Best practice:** Compare attributes only when their domains are conceptually compatible.

---

## 6. Primary Keys

A **primary key (PK)** is a **minimal set of attributes** that uniquely identifies each tuple in a relation.

### Formal Definition

> A primary key `K` of relation `R` is a subset of attributes of `R` such that:
> 1. **Uniqueness** — no two tuples have the same values for `K`
> 2. **Minimality** — no proper subset of `K` has the uniqueness property
> 3. **Non-null** — no attribute in `K` may be NULL

### Properties

| Property | Meaning |
|---|---|
| **Unique** | Identifies exactly one tuple |
| **Non-null** | Every tuple has a value |
| **Minimal** | No redundant attributes |
| **Stable** | Ideally, values do not change |
| **Single per relation** | At most one PK per table |

### Example

```sql
CREATE TABLE employee (
  id         SERIAL PRIMARY KEY,
  email      VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(50)  NOT NULL,
  last_name  VARCHAR(50)  NOT NULL
);
```

Here `id` is the primary key. `email` is a candidate key (unique + non-null) but not the PK.

### Composite Primary Key

A PK can span multiple attributes:

```sql
CREATE TABLE enrollment (
  student_id INTEGER NOT NULL,
  course_id  INTEGER NOT NULL,
  enrolled_at DATE NOT NULL,
  PRIMARY KEY (student_id, course_id)
);
```

### Choosing a Primary Key

| Criterion | Good PK | Poor PK |
|---|---|---|
| Stability | Never changes | Changes often (e.g., email) |
| Uniqueness | Guaranteed | Possibly reused |
| Size | Small | Large (long strings) |
| Simplicity | Single column | Many columns |
| Meaning | Surrogate (no meaning) | Natural but volatile |

### Primary Key vs. Unique Key

| Aspect | Primary Key | Unique Key |
|---|---|---|
| NULLs allowed | No | Often yes (varies by DBMS) |
| Count per table | One | Multiple |
| Index | Clustered / primary index | Secondary index |
| Purpose | Row identity | Alternate uniqueness |

---

## 7. Foreign Keys

A **foreign key (FK)** is an attribute (or set of attributes) in one relation whose values must match the primary key (or a unique key) values in another relation.

### Formal Definition

> Let `R` and `S` be relations. A foreign key in `R` is a set of attributes `FK` such that for every tuple in `R`, either the FK values are NULL, or they match the primary key values of some tuple in `S`.

### Purpose

- **Enforce referential integrity** — no orphan rows
- **Represent relationships** — 1:1, 1:N, M:N (via junction tables)
- **Enable joins** — link related data

### Example

```sql
CREATE TABLE department (
  id   SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE employee (
  id            SERIAL PRIMARY KEY,
  first_name    VARCHAR(50) NOT NULL,
  department_id INTEGER,
  CONSTRAINT fk_department
    FOREIGN KEY (department_id) REFERENCES department(id)
    ON DELETE SET NULL
    ON UPDATE CASCADE
);
```

Here `employee.department_id` is a foreign key referencing `department.id`.

### Referential Actions

When a referenced row is **deleted** or **updated**, the DBMS can take several actions:

| Action | Behavior |
|---|---|
| `CASCADE` | Propagate change to referencing rows |
| `SET NULL` | Set FK to NULL (requires nullable FK) |
| `SET DEFAULT` | Set FK to its default value |
| `RESTRICT` | Prevent the change (immediate) |
| `NO ACTION` | Prevent the change (deferrable, default) |

### NULL in Foreign Keys

If FK attributes are **nullable**, a NULL means "no relationship." This represents **optional participation**.

If FK attributes are **NOT NULL**, every row must reference a valid parent row — **mandatory participation**.

### Self-Referencing Foreign Key

A table can reference itself:

```sql
CREATE TABLE employee (
  id        SERIAL PRIMARY KEY,
  name      VARCHAR(100) NOT NULL,
  manager_id INTEGER,
  FOREIGN KEY (manager_id) REFERENCES employee(id)
);
```

Here `manager_id` points to another row in the same table.

### Composite Foreign Key

Matches a composite primary key:

```sql
CREATE TABLE order_item (
  order_id   INTEGER,
  product_id INTEGER,
  quantity   INTEGER NOT NULL,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id) REFERENCES orders(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);
```

---

## 8. Candidate Keys

A **candidate key** is any attribute (or set of attributes) that **could serve as a primary key** — it is both unique and minimal.

### Formal Definition

> A candidate key of relation `R` is a minimal subset of attributes of `R` that uniquely identifies every tuple in `R`.

### Properties

- **Unique** — no two tuples share the same values
- **Minimal** — no proper subset has the uniqueness property
- **Non-null** — every attribute value must exist
- **Multiple per relation** — a table can have many candidate keys

### Example

For the relation:
```
Customer(id, email, ssn, first_name, last_name)
```

Assuming `id`, `email`, and `ssn` are each unique and non-null, all three are **candidate keys**. The designer chooses one as the **primary key**; the others become **alternate keys**.

### Visualizing

```
Relation: Customer
Attributes: id, email, ssn, first_name, last_name

Candidate keys:
  {id}     ← unique, minimal
  {email}  ← unique, minimal
  {ssn}    ← unique, minimal

Primary key (chosen): {id}
Alternate keys:       {email}, {ssn}
```

### Finding Candidate Keys

1. Identify all **superkeys** (attribute sets that uniquely identify tuples)
2. Remove redundant attributes to find **minimal** sets
3. Each minimal set is a candidate key

**Example:**
- Superkey: `{id, email}` — but `{id}` alone is unique, so not minimal
- Candidate key: `{id}` — minimal and unique

### Why Candidate Keys Matter

- **Schema design** — knowing all candidate keys helps choose a good PK
- **Normalization** — candidate keys are central to 2NF, 3NF, BCNF
- **Integrity** — every candidate key should be enforced as `UNIQUE`
- **Documentation** — clarifies business rules

---

## 9. Alternate Keys

An **alternate key** is any **candidate key not chosen as the primary key**.

### Definition

> If a relation has multiple candidate keys, the one selected as the primary key is the **PK**; the rest are **alternate keys**.

### Properties

- **Unique** — same as candidate keys
- **Non-null** — typically (depends on DBMS if enforced as UNIQUE)
- **Enforced via UNIQUE** — in SQL, alternate keys become `UNIQUE` constraints
- **Multiple per table** — one table can have many alternate keys

### Example

```sql
CREATE TABLE customer (
  id     SERIAL PRIMARY KEY,         -- primary key
  email  VARCHAR(255) UNIQUE NOT NULL,  -- alternate key
  ssn    CHAR(11)     UNIQUE NOT NULL   -- alternate key
);
```

Here `email` and `ssn` are alternate keys.

### Why Alternate Keys Matter

- **Data integrity** — enforce uniqueness of real-world identifiers
- **Lookups** — queries can use alternate keys
- **Foreign keys** — an FK can reference a unique key (not just PK)
- **Business rules** — reflect natural uniqueness constraints

### Alternate Key vs. Unique Constraint

They are essentially the same in SQL:

| Aspect | Alternate Key | Unique Constraint |
|---|---|---|
| Conceptual level | Relational model | SQL implementation |
| Enforced by | Uniqueness rule | `UNIQUE` |
| NULL handling | Often non-null | DBMS-dependent |

---

## 10. Composite Keys

A **composite key** is a key made of **two or more attributes**. Also called a **compound key** or **concatenated key**.

### Definition

> A composite key is a key consisting of multiple attributes whose combined values uniquely identify tuples.

### Properties

- **Multi-attribute** — at least two columns
- **Collectively unique** — the combination is unique
- **Individually non-unique** — each attribute alone is not unique
- **Minimality** — no attribute can be removed without losing uniqueness

### Example

```sql
CREATE TABLE order_item (
  order_id   INTEGER NOT NULL,
  product_id INTEGER NOT NULL,
  quantity   INTEGER NOT NULL,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id)   REFERENCES orders(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);
```

Here `(order_id, product_id)` is a composite primary key. Neither `order_id` alone nor `product_id` alone is unique.

### Composite vs. Compound vs. Concatenated

These terms are largely synonymous:

| Term | Notes |
|---|---|
| **Composite key** | Most common term |
| **Compound key** | Same meaning |
| **Concatenated key** | Emphasizes concatenation of values |

### When to Use Composite Keys

**Good use cases:**
- **Junction tables** (M:N relationships) — e.g., `enrollments(student_id, course_id)`
- **Natural keys spanning multiple attributes** — e.g., `(country_code, phone_number)`
- **Versioned entities** — e.g., `(entity_id, version_number)`

**Poor use cases:**
- When a stable surrogate key is available
- When the composite key changes frequently
- When the FK references are complex

### Composite Key vs. Surrogate Key

| Aspect | Composite | Surrogate |
|---|---|---|
| Composition | Multiple natural attributes | Single artificial attribute |
| Stability | May change | Stable |
| Size | Larger | Small |
| Meaning | Business meaning | No meaning |
| Joins | Larger join keys | Smaller, faster joins |
| Common in | Junction tables, natural keys | Most modern designs |

---

## 11. Surrogate Keys

A **surrogate key** is a **system-generated identifier** with no business meaning, used as a primary key.

### Definition

> A surrogate key is an artificial attribute added to a relation solely to serve as a unique identifier for tuples.

### Characteristics

| Property | Description |
|---|---|
| **System-generated** | Created by the DBMS (auto-increment, sequence, UUID) |
| **Meaningless** | Not derived from business data |
| **Stable** | Never changes |
| **Compact** | Usually an integer or UUID |
| **Non-null** | Always assigned |
| **Unique** | Guaranteed by the DBMS |

### Examples

```sql
-- Auto-increment integer (MySQL, SQL Server)
CREATE TABLE customer (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- SERIAL (PostgreSQL)
CREATE TABLE customer (
  id   SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- IDENTITY (SQL Server, PostgreSQL 10+)
CREATE TABLE customer (
  id   INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- UUID (PostgreSQL)
CREATE TABLE customer (
  id   UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);
```

### Surrogate vs. Natural Key

| Aspect | Surrogate | Natural |
|---|---|---|
| Source | System-generated | Business data |
| Example | `id = 42` | `email = 'x@y.com'` |
| Meaning | None | Business meaning |
| Stability | Very stable | Can change |
| Size | Small | Often larger |
| Uniqueness risk | None | Real-world reuse possible |
| Recommended for PK | Yes | As alternate key |

### Advantages of Surrogate Keys

- **Stable** — never change, so FKs never need updating
- **Compact** — faster joins and indexes
- **Simple** — single-attribute keys
- **No business logic leakage** — decouples schema from business rules
- **Uniform** — consistent pattern across all tables

### Disadvantages

- **Extra column** — additional storage (minimal)
- **Meaningless** — must join to see real identifiers
- **Hidden natural keys** — must still enforce uniqueness on natural keys
- **Sequence gaps** — auto-increment values may not be contiguous

### Best Practice: Both

Use **both**:

- **Surrogate key** as the primary key (stable, compact)
- **Natural key** as a unique constraint (business identity)

```sql
CREATE TABLE customer (
  id     SERIAL PRIMARY KEY,
  email  VARCHAR(255) UNIQUE NOT NULL,  -- natural key enforced
  name   VARCHAR(100) NOT NULL
);
```

---

## 12. Referential Integrity

**Referential integrity** is the rule that **foreign key values must match existing primary key values** (or be NULL).

### Definition

> For every foreign key value in a referencing relation, either:
> - the value is NULL, or
> - the value matches a primary key value in the referenced relation.

### Formal Statement

Let `R` and `S` be relations, and `FK` be a foreign key in `R` referencing primary key `PK` of `S`:

```
∀ tuple t in R:
  either t[FK] IS NULL
  or ∃ tuple s in S such that s[PK] = t[FK]
```

### Purpose

- **Prevent orphan rows** — no child without a parent
- **Maintain consistency** — links remain valid
- **Enable joins** — related data is guaranteed to exist
- **Reflect reality** — a real-world relationship exists

### Example

```sql
CREATE TABLE orders (
  id          SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  order_date  DATE NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

**Violations that referential integrity prevents:**

- Inserting an order with `customer_id = 999` where no such customer exists
- Deleting a customer who still has orders (unless `ON DELETE CASCADE` or `SET NULL`)
- Updating a customer's `id` without cascading to orders

### Enforcement

The DBMS enforces referential integrity by:

1. **Checking on INSERT/UPDATE** of the referencing table
2. **Checking on DELETE/UPDATE** of the referenced table
3. **Applying referential actions** when the parent changes

### Referential Actions Summary

| Action | On Delete of Parent | On Update of Parent |
|---|---|---|
| `CASCADE` | Delete child rows | Update child FK values |
| `SET NULL` | Set child FK to NULL | Set child FK to NULL |
| `SET DEFAULT` | Set child FK to default | Set child FK to default |
| `RESTRICT` | Prevent delete | Prevent update |
| `NO ACTION` | Prevent delete (deferrable) | Prevent update (deferrable) |

### Deferrable Constraints

In some DBMS (PostgreSQL, Oracle), FK checks can be **deferred** to transaction end:

```sql
ALTER TABLE orders
  ADD CONSTRAINT fk_customer
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    DEFERRABLE INITIALLY DEFERRED;
```

This allows temporary violations within a transaction, as long as the final state is valid — useful for circular references or bulk loads.

### Referential Integrity vs. Application Logic

| Aspect | Database-Enforced | Application-Enforced |
|---|---|---|
| Reliability | Always enforced | Depends on code paths |
| Performance | Slight overhead | Depends on implementation |
| Portability | Standard SQL | Framework-specific |
| Recommendation | **Use DB constraints** | Supplement, don't replace |

**Best practice:** Enforce referential integrity in the **database**. Applications should not be the sole guardian of data consistency.

---

## 13. Entity Integrity

**Entity integrity** is the rule that **primary key values must be unique and non-null** in every relation.

### Definition

> For every relation `R`, no tuple may have a NULL value for any attribute of the primary key, and no two tuples may share the same primary key value.

### Purpose

- **Guarantee row identity** — every row is distinguishable
- **Prevent NULL keys** — a NULL key cannot identify anything
- **Support relationships** — FKs reference PKs, so PKs must be reliable
- **Enable indexing** — PK indexes require non-null uniqueness

### Formal Statement

Let `K` be the primary key of relation `R`:

```
1. ∀ tuple t in R: t[K] IS NOT NULL
2. ∀ tuples t₁, t₂ in R: t₁ ≠ t₂ → t₁[K] ≠ t₂[K]
```

### Example

**Valid (entity integrity holds):**

| id | name | email |
|---|---|---|
| 1 | John | john@example.com |
| 2 | Maria | maria@example.com |
| 3 | Ahmed | ahmed@example.com |

**Invalid (NULL primary key):**

| id | name | email |
|---|---|---|
| 1 | John | john@example.com |
| **NULL** | Maria | maria@example.com | ← violates entity integrity |

**Invalid (duplicate primary key):**

| id | name | email |
|---|---|---|
| 1 | John | john@example.com |
| **1** | Maria | maria@example.com | ← violates entity integrity |

### Enforcement in SQL

```sql
CREATE TABLE customer (
  id    SERIAL PRIMARY KEY,  -- enforces uniqueness + non-null
  name  VARCHAR(100) NOT NULL
);
```

The `PRIMARY KEY` constraint enforces **both** entity integrity rules automatically:

- `UNIQUE` — no duplicates
- `NOT NULL` — no nulls

### Entity Integrity vs. Referential Integrity

| Aspect | Entity Integrity | Referential Integrity |
|---|---|---|
| Rule | PK must be unique + non-null | FK must match existing PK or be NULL |
| Applies to | Primary key | Foreign key |
| Protects | Row identity | Relationship validity |
| Domain | Within a relation | Between relations |

**Together** they form the two fundamental integrity rules of the relational model.

### Why Entity Integrity Matters

- **Joins rely on PK uniqueness** — duplicates would multiply rows
- **FKs reference PKs** — NULL or duplicate PKs would break the link
- **Normalization depends on it** — functional dependencies assume unique keys
- **Applications assume it** — code often treats `id` as a stable identifier

### Codd's 12 Rules (Relevant Ones)

Entity integrity relates to **Codd's Rule 2** (Guaranteed Access Rule):

> Each and every datum (atomic value) in a relational database is guaranteed to be logically accessible by resorting to a combination of table name, primary key value, and column name.

Without entity integrity, this guarantee fails.

---

## Summary Table

| Concept | Formal Definition | SQL Equivalent |
|---|---|---|
| **Relational model** | Mathematical model based on set theory | Foundation of RDBMS |
| **Relation** | Set of tuples sharing attributes | Table |
| **Tuple** | Ordered set of attribute values | Row / Record |
| **Attribute** | Named property with a domain | Column |
| **Domain** | Set of allowed values | Data type + constraints |
| **Primary key** | Minimal unique identifier, non-null | `PRIMARY KEY` |
| **Foreign key** | References a PK or unique key in another relation | `FOREIGN KEY` |
| **Candidate key** | Any minimal unique identifier | `UNIQUE` + `NOT NULL` |
| **Alternate key** | Candidate key not chosen as PK | `UNIQUE` |
| **Composite key** | Key of multiple attributes | Multi-column `PRIMARY KEY` |
| **Surrogate key** | System-generated identifier | `SERIAL`, `IDENTITY`, `UUID` |
| **Referential integrity** | FK must match PK or be NULL | `FOREIGN KEY` constraint |
| **Entity integrity** | PK must be unique and non-null | `PRIMARY KEY` constraint |

---

## Key Takeaways

1. The **relational model** represents data as **relations** (tables) based on **set theory** and **predicate logic**.
2. A **relation** is a set of **tuples** sharing the same **attributes**, each with a **domain**.
3. **Domains** define allowed values; SQL approximates them via data types and constraints.
4. **Keys** identify tuples:
   - **Primary key** — the chosen unique identifier
   - **Candidate keys** — all possible unique identifiers
   - **Alternate keys** — candidate keys not chosen as PK
   - **Composite keys** — multi-attribute keys
   - **Surrogate keys** — system-generated identifiers
5. **Foreign keys** link relations and enforce **referential integrity**.
6. **Entity integrity** requires PKs to be unique and non-null.
7. **Referential integrity** requires FKs to match existing PKs or be NULL.
8. Together, entity and referential integrity form the two fundamental rules of the relational model.
9. Prefer **surrogate keys** as PKs, with **natural keys** as unique constraints.
10. Enforce integrity in the **database**, not just the application.
