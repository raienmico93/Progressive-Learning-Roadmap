# Database Design in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Database design in C# is the discipline of modeling application data, translating that model into a relational (or hybrid) schema, and implementing it correctly from C# code using an O/RM or data-access layer.

**Technical Definition**
Database design in C# encompasses conceptual data modeling (entities, attributes, relationships), logical design (normalization, keys, constraints), physical design (indexes, data types, storage), and the mapping of that design to C# types through an O/RM (EF Core, Dapper) or hand-written data access. It includes the design of primary keys, foreign keys, unique constraints, check constraints, indexes, and concurrency tokens, and it must account for transactions, isolation levels, and the impedance mismatch between the relational model and object-oriented C#.

**Beginner-Friendly Explanation**
Database design is "planning how your data will be stored before you write code." You decide what tables you need, what columns each table has, how tables relate to each other, and how to keep the data consistent. In C#, you then write classes that match that design and use a tool like EF Core or Dapper to read and write the data. Good design makes your application fast, correct, and easy to change; bad design causes bugs, slow queries, and painful migrations.

---

### Key Characteristics

- **Model-first or database-first** — You can design the schema first (code-first), the database first, or meet in the middle.
- **Normalization reduces redundancy** — Normal forms guide the elimination of duplicate data and update anomalies.
- **Keys enforce identity and relationships** — Primary keys, foreign keys, and unique constraints are the backbone of integrity.
- **Indexes trade write cost for read speed** — Index design is central to query performance.
- **Constraints encode business rules** — Check constraints, unique constraints, and foreign keys enforce invariants at the database level.
- **Concurrency must be designed** — Optimistic or pessimistic concurrency tokens prevent lost updates.
- **The object-relational impedance mismatch is real** — Inheritance, collections, and identity do not map one-to-one to tables.
- **Design is iterative** — Schema and queries evolve; migrations and versioning are part of design.

---

### Prerequisites

- Basic SQL: `CREATE TABLE`, `SELECT`, `INSERT`, `UPDATE`, `DELETE`, joins.
- Understanding of primary keys, foreign keys, and indexes.
- Working knowledge of C# classes, properties, and collections.
- Familiarity with an O/RM (EF Core or Dapper) or ADO.NET.
- Awareness of transactions and isolation levels.
- Basic understanding of the CAP theorem and when to use NoSQL alongside relational databases.

---

### Related Programming Areas

- **Entity Framework Core** — Code-first modeling and migrations.
- **Dapper** — Hand-written SQL mapped to objects.
- **Domain-driven design (DDD)** — Aggregates, entities, and value objects inform schema design.
- **API design** — DTOs and projections shape what the database must provide.
- **Performance engineering** — Indexing, query plans, and caching.
- **DevOps** — Migrations, schema versioning, and deployment pipelines.
- **Testing** — Integration tests against a real database engine.

---

### Core Concepts / Features

1. Data modeling
2. Normalization
3. Indexing
4. Relationships
5. Database design best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Data Modeling

### Definitions

**Core Definition**
Data modeling is the process of identifying the entities, attributes, and relationships in a domain and representing them in a structured form that can be implemented in a database.

**Technical Definition**
Data modeling proceeds through conceptual, logical, and physical stages. The conceptual model identifies entities and relationships independent of implementation. The logical model adds attributes, keys, and normalization. The physical model specifies data types, indexes, constraints, and storage for a particular database engine. In C#, the logical and physical models are often expressed as entity classes configured with EF Core's Fluent API or annotated with data annotations, and the physical model is generated or validated through migrations.

**Beginner-Friendly Explanation**
Data modeling is "figuring out what things your application needs to store and how they connect." For a bookstore, the things are books, authors, customers, and orders. You decide that a book has a title and a price, an author has a name, and a customer places orders. Then you turn those things into tables and classes.

---

### Purposes

- To capture the domain's entities, attributes, and rules before writing code.
- To provide a blueprint for the database schema and C# classes.
- To identify relationships and cardinalities early.
- To expose ambiguity and missing requirements.
- To guide normalization and indexing decisions.
- To support communication between developers, DBAs, and stakeholders.

---

### Syntax Rules and Structure

#### General Syntax (Conceptual and C# Representation)

```
// Conceptual model
Entity: Customer
  Attributes: Id, Name, Email
  Relationships: places many Orders

Entity: Order
  Attributes: Id, OrderDate, Total
  Relationships: belongs to one Customer, contains many OrderItems

Entity: OrderItem
  Attributes: Id, Quantity, UnitPrice
  Relationships: belongs to one Order, references one Product

Entity: Product
  Attributes: Id, Name, Price
```

```
// C# representation (EF Core)
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Total { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;
    public List<OrderItem> Items { get; set; } = new();
}

public class OrderItem
{
    public int Id { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}
```

**Component breakdown**

- Entity — A thing with identity and lifecycle (`Customer`, `Order`).
- Attribute — A property of an entity (`Name`, `OrderDate`).
- Relationship — An association between entities (`Customer` places `Orders`).
- Cardinality — One-to-many, one-to-one, many-to-many.
- Key — Attribute(s) that uniquely identify an entity (`Id`).

#### Syntax Rules

- Every entity should have a stable identity (primary key).
- Attributes should be atomic (one value per attribute) in the logical model.
- Relationships should specify cardinality and optionality.
- Business rules should be captured as constraints where possible.
- Naming should be consistent across the model and the database.

#### Constraints and Limitations

- The conceptual model should not be tied to a specific database engine.
- Some domain concepts (e.g., value objects, inheritance) require mapping strategies.
- Not all business rules can be expressed as database constraints.
- The model will evolve; versioning and migrations are necessary.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Code-First Model with EF Core

```csharp
using Microsoft.EntityFrameworkCore;

public class BookstoreContext : DbContext
{
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OrderItem> OrderItems => Set<OrderItem>();
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>(b =>
        {
            b.Property(c => c.Name).HasMaxLength(100).IsRequired();
            b.Property(c => c.Email).HasMaxLength(200).IsRequired();
            b.HasIndex(c => c.Email).IsUnique();
        });

        modelBuilder.Entity<Order>(b =>
        {
            b.Property(o => o.Total).HasPrecision(18, 2);
        });

        modelBuilder.Entity<OrderItem>(b =>
        {
            b.Property(i => i.UnitPrice).HasPrecision(18, 2);
            b.HasKey(i => i.Id);
        });
    }
}
```

**Line-by-line comments**

- `DbSet<T>` — Exposes each entity as a queryable collection.
- `OnModelCreating` — Configures the model with the Fluent API.
- `HasMaxLength`, `IsRequired` — Column constraints.
- `HasIndex(...).IsUnique()` — Unique constraint on email.
- `HasPrecision(18, 2)` — Decimal precision for monetary values.

**Expected Output (generated schema, conceptual)**

```
CREATE TABLE [Customers] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [Name] nvarchar(100) NOT NULL,
    [Email] nvarchar(200) NOT NULL
);
CREATE UNIQUE INDEX [IX_Customers_Email] ON [Customers] ([Email]);

CREATE TABLE [Products] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [Name] nvarchar(max) NOT NULL,
    [Price] decimal(18,2) NOT NULL
);

CREATE TABLE [Orders] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [OrderDate] datetime2 NOT NULL,
    [Total] decimal(18,2) NOT NULL,
    [CustomerId] int NOT NULL REFERENCES [Customers]([Id])
);

CREATE TABLE [OrderItems] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [Quantity] int NOT NULL,
    [UnitPrice] decimal(18,2) NOT NULL,
    [OrderId] int NOT NULL REFERENCES [Orders]([Id]),
    [ProductId] int NOT NULL REFERENCES [Products]([Id])
);
```

**Why this output occurs**

- Conventions map `Id` to primary keys and `CustomerId` to foreign keys.
- Fluent API adds constraints and precision.
- The schema reflects the conceptual model.

---

#### Example 2: Value Object Mapping (Owned Type)

```csharp
public class Address
{
    public string Street { get; set; } = "";
    public string City { get; set; } = "";
    public string PostalCode { get; set; } = "";
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public Address ShippingAddress { get; set; } = new();
}

// Configuration
modelBuilder.Entity<Customer>().OwnsOne(c => c.ShippingAddress, a =>
{
    a.Property(x => x.Street).HasMaxLength(200);
    a.Property(x => x.City).HasMaxLength(100);
    a.Property(x => x.PostalCode).HasMaxLength(20);
});
```

**Expected Output (generated columns, conceptual)**

```
ShippingAddress_Street nvarchar(200)
ShippingAddress_City nvarchar(100)
ShippingAddress_PostalCode nvarchar(20)
```

**Why this output occurs**

- `OwnsOne` maps the value object to columns on the owning table.
- Value objects have no identity of their own.

---

### Real-World Cases

- **E-commerce** — Customers, orders, products, inventory.
- **Healthcare** — Patients, appointments, prescriptions.
- **Finance** — Accounts, transactions, ledgers.
- **Content platforms** — Articles, authors, tags, comments.

---

### References

- Creating and configuring a model — EF Core - https://learn.microsoft.com/en-us/ef/core/modeling/
- Data modeling — MongoDB (for document modeling contrast) - https://www.mongodb.com/docs/manual/data-modeling/
- Relational database design basics — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/databases/database-design

---

## 2. Normalization

### Definitions

**Core Definition**
Normalization is the process of organizing a relational schema to reduce redundancy and prevent update anomalies by decomposing tables according to normal forms.

**Technical Definition**
Normalization applies a series of normal forms: First Normal Form (1NF) requires atomic attributes and no repeating groups; Second Normal Form (2NF) requires 1NF and no partial dependencies on a composite key; Third Normal Form (3NF) requires 2NF and no transitive dependencies; Boyce-Codd Normal Form (BCNF) strengthens 3NF for certain anomalies; Fourth (4NF) and Fifth (5NF) address multivalued and join dependencies. Normalization reduces redundancy but may increase the number of joins; denormalization is a deliberate trade-off for read performance.

**Beginner-Friendly Explanation**
Normalization is "organizing tables so each fact is stored in exactly one place." If a customer's address is stored in every order row, changing the address means updating many rows, and mistakes happen. Normalization moves the address into a customers table and references it by ID, so there is one place to change it.

---

### Purposes

- To eliminate redundant data and save storage.
- To prevent update, insert, and delete anomalies.
- To make the schema easier to understand and maintain.
- To enforce data integrity through keys and relationships.
- To provide a sound basis for indexing and query design.
- To reduce the risk of inconsistent data.

---

### Syntax Rules and Structure

#### General Syntax (Normal Forms)

```
1NF: Every column contains atomic values; no repeating groups.
2NF: 1NF + every non-key column depends on the whole primary key.
3NF: 2NF + every non-key column depends only on the key (no transitive dependencies).
BCNF: Every determinant is a candidate key.
4NF: No non-trivial multivalued dependencies.
5NF: No non-trivial join dependencies.
```

**Component breakdown**

- Atomic values — One value per cell.
- Functional dependency — `X -> Y` means X determines Y.
- Partial dependency — A non-key column depends on part of a composite key.
- Transitive dependency — A non-key column depends on another non-key column.
- Candidate key — A minimal set of columns that uniquely identifies a row.

#### Syntax Rules

- Identify all functional dependencies before normalizing.
- Decompose tables to remove partial and transitive dependencies.
- Preserve dependencies and avoid lossless-join violations.
- Use foreign keys to represent relationships created by decomposition.

#### Constraints and Limitations

- Over-normalization can lead to excessive joins and slow reads.
- Denormalization is sometimes necessary for performance or reporting.
- Some domains (e.g., time series, document data) do not fit relational normalization.
- Normalization does not replace indexing or query tuning.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Unnormalized vs. Normalized Schema

```csharp
// Unnormalized (1NF violated): repeating groups and redundancy
public class OrderRow
{
    public int OrderId { get; set; }
    public string CustomerName { get; set; } = "";
    public string CustomerEmail { get; set; } = "";
    public string ProductNames { get; set; } = "";   // comma-separated
}
```

```csharp
// Normalized (3NF)
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;
    public List<OrderLine> Lines { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

public class OrderLine
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
    public int Quantity { get; set; }
}
```

**Expected Output (conceptual)**

```
Customers(Id, Name, Email)
Products(Id, Name, Price)
Orders(Id, CustomerId)
OrderLines(Id, OrderId, ProductId, Quantity)
```

**Why this output occurs**

- The unnormalized row stores customer and product data redundantly.
- The normalized schema stores each fact once and uses foreign keys.

---

#### Example 2: Functional Dependency Analysis

```
Given: OrderLine(OrderId, ProductId, Quantity, ProductName, ProductPrice)
Functional dependencies:
  (OrderId, ProductId) -> Quantity
  ProductId -> ProductName, ProductPrice

Problem: ProductName and ProductPrice depend only on ProductId (partial dependency).
Solution: Move product attributes to Products(ProductId, ProductName, ProductPrice).
Result: OrderLine(OrderId, ProductId, Quantity) + Products(ProductId, ProductName, ProductPrice).
```

**Expected Output (conceptual)**

```
OrderLines(OrderId, ProductId, Quantity)
Products(ProductId, ProductName, ProductPrice)
```

**Why this output occurs**

- Removing the partial dependency achieves 2NF.
- Product data is stored once.

---

### Real-World Cases

- **Transactional systems** — Orders, invoices, and inventory benefit from normalization.
- **Data warehouses** — Star schemas deliberately denormalize for analytics.
- **Multi-tenant SaaS** — Normalized tenant, user, and resource tables.
- **Reference data** — Countries, currencies, and categories stored once.

---

### References

- Database normalization — Microsoft - https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description
- Normal forms — Wikipedia - https://en.wikipedia.org/wiki/Database_normalization
- Relational database design — PostgreSQL - https://www.postgresql.org/docs/current/ddl.html

---

## 3. Indexing

### Definitions

**Core Definition**
An index is a data structure that speeds up data retrieval at the cost of additional storage and write overhead.

**Technical Definition**
Indexes are auxiliary structures (typically B-tree or hash) that map key values to row locations. Relational databases support clustered indexes (which define physical row order) and nonclustered indexes (separate structures with pointers to rows). Composite indexes cover multiple columns; covering indexes include all columns needed by a query. Included columns, filtered indexes, unique indexes, and full-text indexes provide specialized behavior. Index design must balance read performance against write cost and storage.

**Beginner-Friendly Explanation**
An index is like the index at the back of a book. Instead of scanning every page to find a topic, you look it up in the index and jump to the right page. Databases use indexes to find rows quickly. But every index also has to be updated when data changes, so too many indexes slow down writes.

---

### Purposes

- To accelerate `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY` operations.
- To enforce uniqueness through unique indexes.
- To support efficient range queries and sorting.
- To cover queries and avoid table scans.
- To improve concurrency by reducing lock duration.
- To support full-text and geospatial search.

---

### Syntax Rules and Structure

#### General Syntax (SQL Server / EF Core)

```
-- Nonclustered index
CREATE INDEX IX_Orders_CustomerId ON Orders (CustomerId);

-- Composite index
CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders (CustomerId, OrderDate DESC);

-- Unique index
CREATE UNIQUE INDEX IX_Customers_Email ON Customers (Email);

-- Included columns (covering index)
CREATE INDEX IX_Orders_CustomerId_Include ON Orders (CustomerId)
INCLUDE (OrderDate, Total);

-- Filtered index
CREATE INDEX IX_Orders_Pending ON Orders (OrderDate)
WHERE Status = 'Pending';

-- EF Core Fluent API
modelBuilder.Entity<Order>()
    .HasIndex(o => new { o.CustomerId, o.OrderDate })
    .HasDatabaseName("IX_Orders_CustomerId_OrderDate");
```

**Component breakdown**

- `CREATE INDEX` — Creates a nonclustered index.
- Column list — Order matters for composite indexes.
- `INCLUDE` — Adds non-key columns to the leaf level.
- `WHERE` — Filtered index for a subset of rows.
- `HasIndex` — EF Core index configuration.

#### Syntax Rules

- The leftmost columns of a composite index are most selective for filtering.
- Index columns used in `JOIN` and `WHERE` are primary candidates.
- Unique indexes enforce uniqueness and can improve lookups.
- Covering indexes avoid key lookups.
- Filtered indexes reduce size for sparse conditions.

#### Constraints and Limitations

- Indexes consume storage and slow down `INSERT`, `UPDATE`, and `DELETE`.
- Too many indexes can confuse the query optimizer.
- Low-selectivity columns (e.g., boolean flags) are poor standalone index candidates.
- Functions on indexed columns can prevent index usage.
- Index maintenance (rebuild/reorganize) is required over time.
- Heap tables without clustered indexes may need forwarding pointers.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Indexing a Frequent Query

```csharp
// Query: find recent orders for a customer
var orders = await db.Orders
    .Where(o => o.CustomerId == customerId && o.OrderDate >= cutoff)
    .OrderByDescending(o => o.OrderDate)
    .ToListAsync();

// Index configuration
modelBuilder.Entity<Order>()
    .HasIndex(o => new { o.CustomerId, o.OrderDate })
    .HasDatabaseName("IX_Orders_CustomerId_OrderDate");
```

**Expected Output (conceptual execution plan)**

```
Index Seek on IX_Orders_CustomerId_OrderDate (CustomerId = @p0, OrderDate >= @p1)
```

**Why this output occurs**

- The composite index matches the filter and sort.
- The query optimizer chooses an index seek instead of a scan.

---

#### Example 2: Covering Index

```sql
CREATE INDEX IX_Orders_Customer_Covering
ON Orders (CustomerId)
INCLUDE (OrderDate, Total);
```

```csharp
var totals = await db.Orders
    .Where(o => o.CustomerId == customerId)
    .Select(o => new { o.OrderDate, o.Total })
    .ToListAsync();
```

**Expected Output (conceptual execution plan)**

```
Index Seek on IX_Orders_Customer_Covering; no key lookup
```

**Why this output occurs**

- The index contains all columns needed by the query.
- The query is covered and avoids accessing the base table.

---

### Real-World Cases

- **Search and filtering** — Indexes on status, date, and foreign keys.
- **Reporting** — Covering indexes for aggregate queries.
- **Uniqueness** — Unique indexes on email, username, and order number.
- **Full-text search** — Full-text indexes for document search.

---

### References

- SQL Server indexes — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes
- EF Core indexes - https://learn.microsoft.com/en-us/ef/core/modeling/indexes
- PostgreSQL indexes - https://www.postgresql.org/docs/current/indexes.html
- Use the Index, Luke (indexing guide) - https://use-the-index-luke.com/

---

## 4. Relationships

### Definitions

**Core Definition**
A relationship is an association between two entities that is implemented in a relational database through foreign keys and enforced by constraints.

**Technical Definition**
Relational relationships have cardinality (one-to-one, one-to-many, many-to-many) and optionality (required or optional). They are implemented with primary keys and foreign keys, with referential actions (`ON DELETE`, `ON UPDATE`) controlling behavior on modification. In C#, relationships are represented by navigation properties and configured with EF Core's Fluent API or data annotations. Many-to-many relationships use a join table, which may be implicit (EF Core 5+) or explicit (a CLR join entity).

**Beginner-Friendly Explanation**
Relationships are "how tables connect." A customer has many orders; each order belongs to one customer. The database enforces this with a foreign key: the order table stores the customer's ID, and the database refuses to insert an order for a customer that does not exist.

---

### Purposes

- To model real-world associations between entities.
- To enforce referential integrity.
- To enable navigation between related data in C#.
- To support joins and aggregate queries.
- To control cascade behavior on delete and update.
- To represent optional and required associations.

---

### Syntax Rules and Structure

#### General Syntax (One-to-Many)

```sql
CREATE TABLE Customers (
    Id int NOT NULL PRIMARY KEY,
    Name nvarchar(100) NOT NULL
);

CREATE TABLE Orders (
    Id int NOT NULL PRIMARY KEY,
    CustomerId int NOT NULL,
    OrderDate datetime2 NOT NULL,
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerId)
        REFERENCES Customers(Id) ON DELETE CASCADE
);
```

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;
    public DateTime OrderDate { get; set; }
}

modelBuilder.Entity<Customer>()
    .HasMany(c => c.Orders)
    .WithOne(o => o.Customer)
    .HasForeignKey(o => o.CustomerId)
    .OnDelete(DeleteBehavior.Cascade);
```

**Component breakdown**

- Foreign key — `CustomerId` on `Orders`.
- Principal — `Customers`.
- Dependent — `Orders`.
- Navigation — `Customer.Orders` and `Order.Customer`.
- Referential action — `ON DELETE CASCADE`.

#### Syntax Rules (One-to-One and Many-to-Many)

```csharp
// One-to-one
modelBuilder.Entity<Customer>()
    .HasOne(c => c.Profile)
    .WithOne(p => p.Customer)
    .HasForeignKey<Profile>(p => p.CustomerId);

// Many-to-many (implicit join)
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students);

// Many-to-many (explicit join)
modelBuilder.Entity<Enrollment>()
    .HasKey(e => new { e.StudentId, e.CourseId });
```

#### Syntax Rules

- Every foreign key should reference a primary key or unique key.
- Required relationships use non-nullable foreign keys.
- Optional relationships use nullable foreign keys.
- Cascade delete should be used deliberately; cycles must be avoided.
- Many-to-many requires a join table; explicit joins carry payload.

#### Constraints and Limitations

- Cascade delete cycles are rejected by some databases (e.g., SQL Server).
- Orphaned rows can occur if referential integrity is disabled.
- Soft deletes (`IsDeleted` flags) require query filters or views.
- Polymorphic associations are not natively supported in relational databases.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: One-to-Many with Cascade Delete

```csharp
var customer = new Customer
{
    Name = "Alice",
    Orders = { new Order { OrderDate = DateTime.UtcNow } }
};
db.Customers.Add(customer);
await db.SaveChangesAsync();

db.Customers.Remove(customer);
await db.SaveChangesAsync();

Console.WriteLine(await db.Orders.CountAsync());
```

**Expected Output**

```
0
```

**Why this output occurs**

- The order references the customer via a foreign key.
- Deleting the customer cascades to delete the order.
- The count is zero.

---

#### Example 2: Many-to-Many with Explicit Join

```csharp
var student = new Student { Name = "Bob" };
var course = new Course { Name = "Databases" };
db.Enrollments.Add(new Enrollment
{
    Student = student,
    Course = course,
    EnrolledAt = DateTime.UtcNow
});
await db.SaveChangesAsync();

var count = await db.Enrollments.CountAsync();
Console.WriteLine(count);
```

**Expected Output**

```
1
```

**Why this output occurs**

- The explicit join entity stores the relationship and payload.
- The enrollment row is inserted.

---

### Real-World Cases

- **E-commerce** — Customers, orders, order items, products.
- **Education** — Students, courses, enrollments.
- **Social networks** — Users, friendships, followers.
- **Content** — Articles, tags, categories.

---

### References

- Relationships — EF Core - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/
- Foreign key constraints — SQL Server - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Referential integrity — PostgreSQL - https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK

---

## 5. Database Design Best Practices

### Definitions

**Core Definition**
Database design best practices are guidelines for producing schemas that are correct, performant, maintainable, and secure.

**Technical Definition**
Best practices span naming conventions, key design, normalization and deliberate denormalization, constraint enforcement, indexing strategy, data type selection, concurrency control, migration management, security, and testing. They are informed by the relational model, the specific database engine's capabilities, and the application's access patterns.

**Beginner-Friendly Explanation**
Best practices are "the habits that keep a database healthy." They include choosing good names, using the right data types, enforcing rules in the database, indexing for your queries, and planning for change.

---

### Purposes

- To ensure data integrity and consistency.
- To achieve predictable query performance.
- To make the schema understandable and maintainable.
- To support safe schema evolution.
- To protect sensitive data.
- To reduce operational incidents.

---

### Guidelines and Structure

#### Naming and Conventions

```
Tables: PascalCase plural (Customers, Orders)
Columns: PascalCase singular (CustomerId, OrderDate)
Primary keys: Id or <Table>Id
Foreign keys: <ReferencedTable>Id
Indexes: IX_<Table>_<Columns>
Constraints: FK_<Table>_<ReferencedTable>, UQ_<Table>_<Columns>, CK_<Table>_<Rule>
```

- Use consistent naming across the schema.
- Avoid reserved words and abbreviations.
- Use singular column names and plural table names.

#### Keys and Constraints

```sql
CREATE TABLE Orders (
    Id int NOT NULL IDENTITY PRIMARY KEY,
    CustomerId int NOT NULL,
    OrderDate datetime2 NOT NULL,
    Total decimal(18,2) NOT NULL CHECK (Total >= 0),
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerId) REFERENCES Customers(Id),
    CONSTRAINT UQ_Orders_Number UNIQUE (OrderNumber)
);
```

- Use surrogate keys for stability; natural keys for uniqueness constraints.
- Enforce business rules with `CHECK`, `UNIQUE`, and `FOREIGN KEY`.
- Prefer `NOT NULL` unless the domain requires optionality.

#### Data Types

```
Monetary: decimal(18,2) or decimal(19,4)
Dates: datetime2 (SQL Server), timestamptz (PostgreSQL)
Strings: nvarchar with explicit length; avoid max when possible
Booleans: bit (SQL Server), boolean (PostgreSQL)
Identifiers: uniqueidentifier (SQL Server), uuid (PostgreSQL)
```

- Choose the smallest type that safely holds the domain.
- Use `decimal` for money, not `float`.
- Store UTC timestamps and convert at presentation.
- Avoid `NULL` in columns that participate in unique constraints unless intended.

#### Indexing Strategy

- Index foreign keys used in joins.
- Index columns used in frequent filters and sorts.
- Use composite indexes matching query predicates.
- Avoid over-indexing write-heavy tables.
- Review execution plans and missing-index suggestions.

#### Concurrency

```csharp
public class Order
{
    public int Id { get; set; }
    public byte[] RowVersion { get; set; } = Array.Empty<byte>();
}

modelBuilder.Entity<Order>().Property(o => o.RowVersion).IsRowVersion();
```

- Use rowversion/timestamp columns for optimistic concurrency.
- Use transactions and appropriate isolation levels for multi-statement operations.
- Avoid long-running transactions.

#### Migrations and Versioning

```
dotnet ef migrations add AddOrderTotal
dotnet ef database update
```

- Keep migrations in source control.
- Review generated SQL before applying.
- Use idempotent scripts for production.
- Never edit applied migrations.

#### Security

- Use least-privilege database accounts.
- Parameterize all queries.
- Encrypt sensitive columns (TDE, Always Encrypted, application-level).
- Audit access and rotate credentials.
- Back up and test restores regularly.

#### Testing

- Test against the real database engine (containers or dedicated instances).
- Use integration tests for constraints, indexes, and concurrency.
- Seed reference data consistently.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Well-Designed Table with Constraints

```sql
CREATE TABLE Products (
    Id int NOT NULL IDENTITY PRIMARY KEY,
    Sku nvarchar(50) NOT NULL,
    Name nvarchar(200) NOT NULL,
    Price decimal(18,2) NOT NULL,
    IsActive bit NOT NULL DEFAULT 1,
    CreatedAt datetime2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT UQ_Products_Sku UNIQUE (Sku),
    CONSTRAINT CK_Products_Price CHECK (Price >= 0)
);

CREATE INDEX IX_Products_IsActive_Name ON Products (IsActive, Name);
```

**Expected Output (conceptual)**

```
Table created with primary key, unique SKU, check constraint, and composite index.
```

**Why this output occurs**

- Constraints enforce data integrity.
- The index supports filtering active products by name.

---

#### Example 2: Optimistic Concurrency in EF Core

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public byte[] RowVersion { get; set; } = Array.Empty<byte>();
}

// Update with concurrency check
var order = await db.Orders.FirstAsync(o => o.Id == 1);
order.Total = 100m;
try
{
    await db.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException)
{
    Console.WriteLine("Concurrency conflict detected");
}
```

**Expected Output (conceptual)**

```
UPDATE Orders SET Total = 100 WHERE Id = 1 AND RowVersion = @original;
-- if no rows affected, DbUpdateConcurrencyException is thrown
```

**Why this output occurs**

- The rowversion column is included in the `WHERE` clause.
- A mismatch indicates another transaction modified the row.

---

#### Example 3: Parameterized Query (Security)

```csharp
var email = "alice@example.com";
var customer = await db.Customers
    .FromSqlInterpolated($"SELECT * FROM Customers WHERE Email = {email}")
    .FirstOrDefaultAsync();
```

**Expected Output (conceptual SQL)**

```
SELECT * FROM Customers WHERE Email = @p0
```

**Why this output occurs**

- `FromSqlInterpolated` parameterizes the interpolated value.
- SQL injection is prevented.

---

### Real-World Cases

- **SaaS platforms** — Multi-tenant schemas with tenant isolation.
- **Financial systems** — Decimal precision, audit trails, and concurrency control.
- **Healthcare** — Privacy constraints, encryption, and audit logging.
- **E-commerce** — Product catalogs, orders, and inventory with indexing.

---

### References

- Database design basics — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/databases/database-design
- EF Core best practices - https://learn.microsoft.com/en-us/ef/core/
- SQL Server index design guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide
- PostgreSQL constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- Use the Index, Luke - https://use-the-index-luke.com/
- OWASP SQL injection prevention - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Transaction isolation levels — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/transactions/transaction-isolation-levels

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `datetime` / `smalldatetime` (SQL Server) | Lower precision and range; prefer `datetime2`. |
| `float` for money | Rounding errors; use `decimal`. |
| `nvarchar(max)` everywhere | Can prevent index key usage; use explicit lengths. |
| `SELECT *` | Fetches unnecessary columns; specify columns. |
| `ALLOW FILTERING` (Cassandra) | Expensive; avoid in production. |
| Cascade delete cycles | Rejected by SQL Server; design around them. |
| `EnsureCreated` with migrations | Incompatible; choose one approach. |
| Over-indexing | Slows writes and increases storage. |
| `ROWLOCK`/`NOLOCK` hints | Can cause dirty reads; use deliberately. |
| Soft deletes without filters | Queries may include deleted rows; use query filters. |
| `uniqueidentifier` clustered keys | Random GUIDs fragment clustered indexes; use sequential GUIDs or nonclustered keys. |
| `IDENTITY` vs. sequences | Sequences are standard SQL; identity is SQL Server-specific. |

---

## References (Consolidated)

- Database design basics — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/databases/database-design
- Creating and configuring a model — EF Core - https://learn.microsoft.com/en-us/ef/core/modeling/
- EF Core best practices - https://learn.microsoft.com/en-us/ef/core/
- Relationships — EF Core - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/
- EF Core indexes - https://learn.microsoft.com/en-us/ef/core/modeling/indexes
- SQL Server indexes — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes
- SQL Server index design guide - https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide
- Foreign key constraints — SQL Server - https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships
- Transaction isolation levels — Microsoft - https://learn.microsoft.com/en-us/sql/relational-databases/transactions/transaction-isolation-levels
- Database normalization — Microsoft - https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description
- Normal forms — Wikipedia - https://en.wikipedia.org/wiki/Database_normalization
- PostgreSQL documentation — DDL - https://www.postgresql.org/docs/current/ddl.html
- PostgreSQL constraints - https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL indexes - https://www.postgresql.org/docs/current/indexes.html
- Use the Index, Luke - https://use-the-index-luke.com/
- OWASP SQL injection prevention - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- MongoDB data modeling (document modeling contrast) - https://www.mongodb.com/docs/manual/data-modeling/