# Object-Relational Mapping (ORM): A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Object-Relational Mapping (ORM) is a programming technique that creates a bridge between object-oriented application code and relational database tables, automatically converting data between incompatible type systems.

**Technical Definition**: ORM is a layer of software that maps in-memory objects (instances of classes) to rows in relational database tables, handling the translation of object operations (create, read, update, delete) into SQL statements, managing object identity, relationship traversal, transaction boundaries, and caching. The two dominant architectural patterns are Active Record (where the object itself encapsulates database access) and Data Mapper (where a separate mapper layer mediates between domain objects and the database).

**Beginner-Friendly Explanation**: An ORM is like a bilingual translator between your application code and your database. Your application speaks in objects (like `User` and `Order`), while the database speaks in tables and rows. The ORM translates between these two worlds automatically, so you can work with objects in your code without writing SQL by hand.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Mapping** | Converts object attributes to table columns and relationships to foreign keys |
| **Identity Management** | Ensures one database row maps to one in-memory object instance |
| **Change Tracking** | Detects modifications to loaded objects and generates appropriate UPDATE statements |
| **Lazy/Eager Loading** | Controls when related data is fetched from the database |
| **Caching** | First-level (session) and second-level (application) caches reduce database hits |
| **Query Language** | Provides object-oriented query languages (HQL, JPQL) or query builders |

### Prerequisites

- **Entity Classes**: Plain objects annotated or configured to map to database tables
- **Persistence Configuration**: Database connection settings, dialect, and mapping metadata
- **Migration Tooling**: Flyway, Liquibase, or native ORM migration support
- **Profiling Tools**: SQL logging, query statistics, and N+1 detection tools
- **Understanding of SQL**: ORM does not eliminate the need to understand the underlying database

### Related Programming Areas

- **Database Administration**: Schema design, indexing strategy, query performance
- **Application Architecture**: Domain-driven design, repository patterns, unit of work
- **Security Engineering**: SQL injection prevention (ORM parameterization), credential management
- **Performance Engineering**: N+1 detection, caching strategy, batch processing
- **DevOps**: Migration automation, schema version control, deployment pipelines

### Core Concepts Overview

ORM comprises nine complementary domains:

1. **ORM Fundamentals**: Data Mapper vs. Active Record design patterns
2. **Entity Mapping**: Primary keys, auto-generation strategies, custom type handlers
3. **Relationship Mapping**: One-to-One, One-to-Many, Many-to-Many, polymorphic associations
4. **Lazy Loading**: Proxy mechanisms and detached entity exceptions
5. **Eager Loading**: Join-fetching vs. subquery fetching
6. **N+1 Query Problem**: Detection, profiling, and mitigation
7. **ORM-Generated SQL**: Inspection, profiling, and native SQL overrides
8. **Cache Management**: First-level vs. second-level cache strategies
9. **Database Migrations**: Schema version control and code-first migrations

---

## Core Concept 1: ORM Fundamentals — Data Mapper vs. Active Record

### Definitions

**Core Definition**: Data Mapper and Active Record are the two fundamental architectural patterns for ORM, differing in how closely domain objects are coupled to the persistence layer.

**Technical Definition**: In the **Active Record** pattern, an object wraps a row in a database table, encapsulates database access, and adds domain logic directly on the data object. In the **Data Mapper** pattern, a separate mapper layer moves data between in-memory objects and the database while keeping the domain model completely decoupled from the persistence layer. Martin Fowler recommends never using separate Gateways with Domain Models — either use Active Records or a Data Mapper.

**Beginner-Friendly Explanation**: Active Record is like a self-driving car — the object knows how to save itself to the database. Data Mapper is like having a chauffeur — the domain object is a pure business object, and a separate mapper handles all the database work.

### Purposes

- **To** choose the appropriate level of coupling between domain logic and persistence
- **To** simplify development for simple CRUD applications (Active Record)
- **To** maintain clean separation of concerns for complex domain models (Data Mapper)
- **To** enable independent testing of domain logic without database dependencies (Data Mapper)

### Syntax Rules and Structure

#### Active Record Pattern (Ruby on Rails style)

```ruby
# The model itself knows how to persist
class User < ApplicationRecord
  has_many :orders
  validates :email, presence: true
  
  def full_name
    "#{first_name} #{last_name}"
  end
end

# Usage
user = User.new(email: "jane@example.com")
user.save          # Persists to database
user.orders        # Loads related orders
```

#### Data Mapper Pattern (Hibernate/JPA style)

```java
// The domain object is a plain Java class
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String email;
    private String firstName;
    private String lastName;
    
    // getters and setters — no persistence logic
}

// The mapper (EntityManager) handles persistence
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
User user = new User();
user.setEmail("jane@example.com");
em.persist(user);       // Mapper persists the object
em.getTransaction().commit();
```

#### Component Breakdown

| Component | Active Record | Data Mapper |
|-----------|---------------|-------------|
| **Domain Logic** | In the model class | In separate service/domain classes |
| **Persistence Logic** | In the model class (save, delete) | In the mapper/EntityManager |
| **Testing** | Requires database | Domain logic testable in isolation |
| **Complexity** | Simple, less code | More code, better separation |
| **Use Case** | Simple CRUD, rapid development | Complex domains, legacy databases |

#### Syntax Rules

- Active Record works well in MVC frameworks like Ruby on Rails where conventions drive schema design.
- Data Mapper is preferred for complex, decoupled domain models where persistence concerns should not leak into business logic.
- Data Mapper can be implemented with third-party O/R mapping packages like Hibernate or JPA.

#### Constraints and Limitations

- Active Record tightly couples the domain model to the database schema, making it difficult to refactor domain logic independently.
- Active Record is incapable of mapping to legacy databases with non-standard naming conventions.
- Data Mapper requires more boilerplate code and a steeper learning curve.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Data Mapper with Hibernate

```java
import javax.persistence.*;
import java.util.List;

// Step 1: Define the domain entity (no persistence logic)
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "product_name")
    private String name;
    
    private double price;
    
    // Standard getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
}

// Step 2: Use EntityManager (Data Mapper) to persist
public class ProductService {
    private EntityManagerFactory emf;
    
    public ProductService() {
        emf = Persistence.createEntityManagerFactory("myPU");
    }
    
    public void createProduct(String name, double price) {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            
            Product product = new Product();
            product.setName(name);
            product.setPrice(price);
            
            em.persist(product);  // Mapper handles the INSERT
            em.getTransaction().commit();
            
            System.out.println("Created product with ID: " + product.getId());
        } finally {
            em.close();
        }
    }
}
```

**Expected Output**:
```
Hibernate: insert into products (product_name, price) values (?, ?)
Created product with ID: 1
```

**Why This Output Occurs**: The `EntityManager` acts as the Data Mapper, translating the `persist()` call into an SQL INSERT statement. The `@GeneratedValue` annotation tells Hibernate to use the database's identity column for the primary key. The domain object `Product` contains no persistence code — it is a pure business object.

### Real-World Cases

**Case 1: Ruby on Rails with Active Record**: A startup builds an MVP using Rails conventions, where `User` and `Post` models directly map to tables with `save()` and `destroy()` methods. Development speed is prioritized over domain purity.

**Case 2: Enterprise Java with Hibernate**: A banking application uses Hibernate with JPA annotations. Domain entities like `Account` and `Transaction` are plain POJOs, while `EntityManager` handles all persistence. Domain logic is tested without a database.

**Case 3: Legacy Database Integration**: A company migrates a legacy COBOL system to Java. Data Mapper allows mapping Java objects to the existing, non-standard database schema without changing the domain model.

---

## Core Concept 2: Entity Mapping — Primary Keys and Auto-Generation

### Definitions

**Core Definition**: Entity mapping defines how a Java class maps to a database table, including primary key specification and automatic ID generation strategy.

**Technical Definition**: Primary key generation strategies in JPA/Hibernate include `IDENTITY` (database auto-increment), `SEQUENCE` (database sequence), `TABLE` (separate key table), `AUTO` (provider chooses), and `UUID` (application-generated). The `@GeneratedValue` annotation specifies the strategy, while `@SequenceGenerator` and `@TableGenerator` define custom generators. For composite keys, JPA provides `@IdClass` and `@EmbeddedId` approaches.

**Beginner-Friendly Explanation**: Primary key generation is how the database automatically assigns a unique ID to each new row. Some databases use auto-incrementing numbers, others use sequences, and some applications generate UUIDs. The ORM abstracts these differences so your code works the same way regardless of the database.

### Purposes

- **To** uniquely identify each entity instance and map it to a database row
- **To** automate primary key assignment so applications don't need to generate IDs manually
- **To** support different database capabilities through a consistent API
- **To** enable composite key mapping for tables with natural multi-column keys

### Syntax Rules and Structure

#### Complete General Syntax (JPA Annotations)

```java
@Entity
@Table(name = "products")
public class Product {
    
    // Strategy 1: Database identity column
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Strategy 2: Database sequence
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "product_seq")
    @SequenceGenerator(name = "product_seq", sequenceName = "product_id_seq", 
                       allocationSize = 1)
    private Long id;
    
    // Strategy 3: Table-based generator
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "id_table")
    @TableGenerator(name = "id_table", table = "id_generator",
                    pkColumnName = "gen_name", valueColumnName = "gen_value",
                    pkColumnValue = "product_id", allocationSize = 1)
    private Long id;
    
    // Strategy 4: UUID
    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    private UUID id;
}
```

#### Complete General Syntax (Composite Keys)

```java
// Approach 1: @IdClass
public class OrderId implements Serializable {
    private Long customerId;
    private Long orderNumber;
    // equals() and hashCode()
}

@Entity
@IdClass(OrderId.class)
public class Order {
    @Id
    private Long customerId;
    @Id
    private Long orderNumber;
}

// Approach 2: @EmbeddedId
@Embeddable
public class OrderId implements Serializable {
    private Long customerId;
    private Long orderNumber;
    // equals() and hashCode()
}

@Entity
public class Order {
    @EmbeddedId
    private OrderId id;
}
```

#### Component Breakdown

| GenerationType | Description | Database Support |
|----------------|-------------|------------------|
| `IDENTITY` | Auto-increment column | MySQL, SQL Server, PostgreSQL (serial) |
| `SEQUENCE` | Database sequence object | PostgreSQL, Oracle, DB2 |
| `TABLE` | Separate table for key allocation | All databases (portable) |
| `AUTO` | Provider selects appropriate strategy | All databases |
| `UUID` | Application-generated UUID | All databases |

#### Syntax Rules

- `@GeneratedValue` must be used with `@Id`; the `strategy` attribute specifies the generation approach.
- `IDENTITY` does not support batch inserts because the ID is generated only after the INSERT executes.
- `SEQUENCE` with `allocationSize > 1` enables Hi/Lo optimization, reducing database round-trips.
- `TABLE` generators are portable but require row-level locking on the generator table.
- For composite keys, `@IdClass` keeps the key fields in the entity class, while `@EmbeddedId` wraps them in a separate `@Embeddable` class.

#### Constraints and Limitations

- `IDENTITY` prevents Hibernate from using JDBC batch inserts because each INSERT must return the generated key.
- `SEQUENCE` is not supported by MySQL (before 8.0) or SQL Server.
- `AUTO` may choose different strategies on different databases, reducing portability.
- Custom type handlers (e.g., JSON, array, enum) must be registered separately.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SEQUENCE Strategy with Hi/Lo Optimization

```java
import javax.persistence.*;

@Entity
@Table(name = "orders")
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_seq")
    @SequenceGenerator(
        name = "order_seq",
        sequenceName = "order_id_seq",
        allocationSize = 50  // Hi/Lo: fetch 50 IDs at once
    )
    private Long id;
    
    private String orderNumber;
    private double total;
    
    // getters and setters
}

// Usage
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

for (int i = 0; i < 100; i++) {
    Order order = new Order();
    order.setOrderNumber("ORD-" + i);
    order.setTotal(100.0 + i);
    em.persist(order);
}

em.getTransaction().commit();
```

**Expected Output**:
```
Hibernate: select next_val as id_val from order_id_seq for update
Hibernate: update order_id_seq set next_val = ? where next_val = ?
Hibernate: insert into orders (order_number, total, id) values (?, ?, ?)
... (repeated 50 times, then another sequence fetch)
```

**Why This Output Occurs**: The `allocationSize = 50` tells Hibernate to fetch 50 IDs from the sequence at once. It uses the first fetched value as the high value and generates the low values in memory (Hi/Lo algorithm). This reduces database round-trips from 100 to 2 (one sequence fetch per 50 inserts).

### Real-World Cases

**Case 1: PostgreSQL with SEQUENCE**: An application uses `GenerationType.SEQUENCE` with `allocationSize = 100`, reducing sequence contention and improving batch insert performance by 10x.

**Case 2: MySQL with IDENTITY**: A Rails application uses MySQL auto-increment columns for primary keys. Hibernate's `IDENTITY` strategy works seamlessly, but batch inserts are disabled.

**Case 3: Composite Key for Join Tables**: A many-to-many join table uses `@EmbeddedId` with a composite key of `(student_id, course_id)`, avoiding a surrogate primary key.

---

## Core Concept 3: Relationship Mapping

### Definitions

**Core Definition**: Relationship mapping defines how associations between entity classes (one-to-one, one-to-many, many-to-many) are represented in the database and navigated in application code.

**Technical Definition**: JPA provides four relationship annotations: `@OneToOne` (one-to-one), `@OneToMany` (one-to-many), `@ManyToOne` (many-to-one), and `@ManyToMany` (many-to-many). The `mappedBy` attribute defines the inverse side of a bidirectional relationship. The `@JoinColumn` annotation specifies the foreign key column, while `@JoinTable` defines a join table for many-to-many relationships. Polymorphic associations allow a single relationship to reference multiple entity types.

**Beginner-Friendly Explanation**: Relationship mapping is how you tell the ORM that a `Customer` has many `Orders`, or that an `Order` belongs to one `Customer`. The ORM uses foreign keys to connect the tables and lets you navigate from one object to another in your code.

### Purposes

- **To** represent foreign key relationships between tables as object references
- **To** enable navigation from one entity to related entities in application code
- **To** control the ownership and cascading behavior of relationships
- **To** support polymorphic associations where a relationship can reference multiple entity types

### Syntax Rules and Structure

#### Complete General Syntax (JPA Relationship Annotations)

```java
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    // One-to-Many: A customer has many orders
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String orderNumber;
    
    // Many-to-One: Many orders belong to one customer
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

#### Many-to-Many with Join Table

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

#### One-to-One Relationship

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private Profile profile;
}

@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
    
    @OneToOne(mappedBy = "profile")
    private User user;
}
```

#### Component Breakdown

| Annotation | Description | Owning Side |
|------------|-------------|-------------|
| `@OneToOne` | One record relates to one record | Side with `@JoinColumn` |
| `@OneToMany` | One record relates to many records | Always the "many" side |
| `@ManyToOne` | Many records relate to one record | Side with `@JoinColumn` |
| `@ManyToMany` | Many records relate to many | Side with `@JoinTable` |

#### Syntax Rules

- The `mappedBy` attribute is used on the inverse side of a bidirectional relationship; it refers to the field name on the owning side.
- The owning side of a one-to-many relationship is always the "many" side (the side with `@ManyToOne`).
- `cascade` controls which operations (PERSIST, MERGE, REMOVE) propagate from parent to child.
- `orphanRemoval = true` deletes child entities when they are removed from the parent's collection.
- `fetch` defaults to `EAGER` for `@ManyToOne` and `@OneToOne`, and `LAZY` for `@OneToMany` and `@ManyToMany`.

#### Constraints and Limitations

- Bidirectional relationships require maintaining both sides in memory to avoid inconsistencies.
- `@ManyToMany` with `CascadeType.REMOVE` can cause unintended deletions; use with caution.
- Polymorphic associations (where a relationship can reference multiple entity types) are not natively supported by JPA and require custom implementations or `@Any` mappings.
- Lazy loading of `@ManyToOne` relationships can cause N+1 problems if not handled carefully.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: One-to-Many with Cascade

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

// Step 1: Parent entity
@Entity
@Table(name = "departments")
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();
    
    // Helper method to maintain both sides
    public void addEmployee(Employee emp) {
        employees.add(emp);
        emp.setDepartment(this);
    }
    
    public void removeEmployee(Employee emp) {
        employees.remove(emp);
        emp.setDepartment(null);
    }
    // getters and setters
}

// Step 2: Child entity
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    // getters and setters
}

// Step 3: Usage
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

Department dept = new Department();
dept.setName("Engineering");

Employee emp1 = new Employee();
emp1.setName("Alice");
Employee emp2 = new Employee();
emp2.setName("Bob");

dept.addEmployee(emp1);
dept.addEmployee(emp2);

em.persist(dept);  // Cascade persists employees too
em.getTransaction().commit();

System.out.println("Department saved with " + dept.getEmployees().size() + " employees");
```

**Expected Output**:
```
Hibernate: insert into departments (name) values (?)
Hibernate: insert into employees (department_id, name) values (?, ?)
Hibernate: insert into employees (department_id, name) values (?, ?)
Department saved with 2 employees
```

**Why This Output Occurs**: The `cascade = CascadeType.ALL` on the `@OneToMany` relationship tells Hibernate to persist the child employees when the department is persisted. The `mappedBy = "department"` indicates that the `Employee` entity owns the foreign key.

### Real-World Cases

**Case 1: E-Commerce Order System**: A `Customer` has many `Orders`, and each `Order` has many `OrderItems`. Cascade delete ensures that deleting a customer removes their orders and order items.

**Case 2: Student Course Registration**: A many-to-many relationship between `Student` and `Course` uses a join table `student_course`. Students can enroll in multiple courses, and courses can have multiple students.

**Case 3: User Profile with One-to-One**: A `User` has exactly one `Profile`, and a `Profile` belongs to exactly one `User`. The profile is loaded lazily and only when accessed.

---

## Core Concept 4: Lazy Loading

### Definitions

**Core Definition**: Lazy loading defers the loading of related entities or collections until they are actually accessed in application code.

**Technical Definition**: Hibernate implements lazy loading using proxy objects — runtime-generated subclasses that masquerade as the real entity but hold no state until initialized. When a lazily loaded association is first accessed, Hibernate checks if the proxy is still associated with an open persistence context (session). If the session has been closed, the proxy can no longer fetch the data and throws a `LazyInitializationException`.

**Beginner-Friendly Explanation**: Lazy loading is like ordering a book from a library catalog without actually pulling it off the shelf. The catalog entry (proxy) says "I can get you this book if you need it." But if you leave the library (close the session), you can't get the book anymore — you get an error.

### Purposes

- **To** reduce initial query time by not loading related data until needed
- **To** avoid loading large object graphs that may not be used
- **To** conserve memory by loading only what is actually accessed
- **To** enable efficient pagination and partial entity loading

### Syntax Rules and Structure

#### Complete General Syntax (JPA FetchType)

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Lazy: load customer only when getCustomer() is called
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    // Eager: load all items immediately
    @OneToMany(fetch = FetchType.EAGER, mappedBy = "order")
    private List<OrderItem> items;
}
```

#### Component Breakdown

| FetchType | Behavior | Default For |
|-----------|----------|-------------|
| `LAZY` | Load on first access | `@OneToMany`, `@ManyToMany` |
| `EAGER` | Load immediately with parent | `@ManyToOne`, `@OneToOne` |

#### Syntax Rules

- `FetchType.LAZY` is a hint — some providers may ignore it for `@OneToOne` relationships on the owning side.
- Lazy loading requires an open session; accessing a lazy proxy after the session closes throws `LazyInitializationException`.
- Lazy loading is not supported for detached entities or entities loaded with `AsNoTracking()` in Entity Framework.
- The `hibernate.enable_lazy_load_no_trans` property allows lazy loading outside a transaction but should be used with caution.

#### Constraints and Limitations

- **Detached entity exception**: Accessing a lazy proxy after the session closes throws `LazyInitializationException`.
- **N+1 problem**: Lazy loading in a loop can trigger N+1 queries if not handled with batch fetching or eager loading.
- **Serialization**: Lazy proxies cannot be serialized directly; DTOs or eager loading are required.
- **Performance**: Lazy loading may cause multiple database round-trips if many entities are accessed.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: LazyInitializationException and Its Fix

```java
import javax.persistence.*;

// Step 1: Entity with lazy relationship
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String orderNumber;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
    // getters and setters
}

// Step 2: PROBLEM — Accessing lazy proxy outside session
public class OrderService {
    public void processOrder(Long orderId) {
        EntityManager em = emf.createEntityManager();
        Order order = em.find(Order.class, orderId);
        em.close();  // Session closed!
        
        // This throws LazyInitializationException
        String customerName = order.getCustomer().getName();
        System.out.println("Customer: " + customerName);
    }
}

// Step 3: FIX — Access within session or use eager loading
public class FixedOrderService {
    public void processOrder(Long orderId) {
        EntityManager em = emf.createEntityManager();
        try {
            Order order = em.find(Order.class, orderId);
            
            // Access within the session
            String customerName = order.getCustomer().getName();
            System.out.println("Customer: " + customerName);
        } finally {
            em.close();
        }
    }
}
```

**Expected Output** (problem):
```
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
```

**Expected Output** (fix):
```
Hibernate: select ... from orders where id = ?
Hibernate: select ... from customers where id = ?
Customer: Acme Corp
```

**Why This Output Occurs**: In the problem case, the `EntityManager` is closed before `order.getCustomer()` is called. The proxy cannot initialize because it has no session to query the database. In the fix, the access occurs within the session scope, allowing Hibernate to issue the additional SELECT for the customer.

### Real-World Cases

**Case 1: REST API Serialization**: A Spring Boot REST controller returns an `Order` with a lazy `Customer` relationship. Jackson serialization triggers `LazyInitializationException` because the session is closed. The fix is to use a DTO or `@JsonIgnore` on the lazy field.

**Case 2: Batch Processing with Lazy Loading**: A batch job iterates over 1,000 orders, accessing `order.getCustomer().getName()` for each. This triggers 1,000 additional queries (N+1). The fix is to use `JOIN FETCH` or `@BatchSize`.

**Case 3: Open Session in View Pattern**: A web application uses Open Session in View (OSIV) to keep the Hibernate session open until the view is rendered, allowing lazy loading during template rendering. This is convenient but can cause connection pool exhaustion under load.

---

## Core Concept 5: Eager Loading

### Definitions

**Core Definition**: Eager loading fetches related entities or collections in the same query as the parent entity, avoiding additional database round-trips.

**Technical Definition**: SQLAlchemy and other ORMs support two primary eager loading techniques: **joined eager loading** (applies a JOIN to the parent SELECT so related rows are loaded in the same result set) and **subquery eager loading** (emits a second SELECT statement that re-states the original query as a subquery and JOINs it to the related table). Joined loading is efficient for small collections; subquery loading is better for large collections where a JOIN would produce a Cartesian product.

**Beginner-Friendly Explanation**: Eager loading is like packing everything you need for a trip in one suitcase instead of making multiple trips. Joined eager loading packs everything in one big suitcase; subquery eager loading packs the main items first, then packs the accessories in a second suitcase.

### Purposes

- **To** eliminate N+1 query problems by loading related data upfront
- **To** reduce database round-trips when related data is always needed
- **To** control the trade-off between query complexity and number of queries
- **To** optimize performance for specific use cases (reporting, API responses)

### Syntax Rules and Structure

#### Complete General Syntax (SQLAlchemy)

```python
from sqlalchemy.orm import joinedload, subqueryload, selectinload

# Joined eager loading (single query with JOIN)
users = session.query(User).options(joinedload(User.orders)).all()

# Subquery eager loading (two queries)
users = session.query(User).options(subqueryload(User.orders)).all()

# Select-in loading (three queries, best for large collections)
users = session.query(User).options(selectinload(User.orders)).all()
```

#### Complete General Syntax (JPA/Hibernate)

```java
// JPQL JOIN FETCH
@Query("SELECT DISTINCT o FROM Order o JOIN FETCH o.customer WHERE o.status = :status")
List<Order> findOrdersWithCustomer(@Param("status") String status);

// EntityGraph
@EntityGraph(attributePaths = {"customer", "items"})
List<Order> findByStatus(String status);
```

#### Component Breakdown

| Technique | Queries | Best For | Trade-off |
|-----------|---------|----------|-----------|
| **Joined eager** | 1 | Small collections | Cartesian product risk |
| **Subquery eager** | 2 | Large collections | Two round-trips |
| **Select-in eager** | 3 | Large collections | Legacy, replaced by `selectinload` |
| **Batch fetching** | N/batch_size | Lazy loading mitigation | Requires `@BatchSize` |

#### Syntax Rules

- Joined eager loading applies a LEFT OUTER JOIN; use `DISTINCT` to avoid duplicate parent rows.
- Subquery eager loading emits a second SELECT with a subquery and INNER JOIN.
- `selectinload` emits a third query using `WHERE id IN (...)`.
- In JPA, `JOIN FETCH` and `@EntityGraph` are the primary eager loading mechanisms.

#### Constraints and Limitations

- Joined eager loading can cause a Cartesian product if multiple collections are fetched simultaneously.
- Subquery eager loading cannot be used with composite primary keys or databases that don't support tuples with `IN`.
- Eager loading always fetches related data, even if it's not needed — use lazy loading when the data may not be required.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Joined Eager Loading in SQLAlchemy

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base, relationship, Session, joinedload

# Step 1: Define models
Base = declarative_base()

class Customer(Base):
    __tablename__ = "customers"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    orders = relationship("Order", back_populates="customer")

class Order(Base):
    __tablename__ = "orders"
    id = Column(Integer, primary_key=True)
    order_number = Column(String)
    customer_id = Column(Integer, ForeignKey("customers.id"))
    customer = relationship("Customer", back_populates="orders")

# Step 2: Query with joined eager loading
engine = create_engine("sqlite:///shop.db")
Base.metadata.create_all(engine)

with Session(engine) as session:
    # Without eager loading: N+1 queries
    customers = session.query(Customer).all()
    for c in customers:
        print(f"{c.name} has {len(c.orders)} orders")  # Lazy load per customer
    
    # With joined eager loading: 1 query
    customers = session.query(Customer).options(joinedload(Customer.orders)).all()
    for c in customers:
        print(f"{c.name} has {len(c.orders)} orders")
```

**Expected Output** (without eager loading):
```
SELECT * FROM customers
SELECT * FROM orders WHERE customer_id = 1
SELECT * FROM orders WHERE customer_id = 2
...
```

**Expected Output** (with joined eager loading):
```
SELECT customers.*, orders.* FROM customers LEFT OUTER JOIN orders ON ...
```

**Why This Output Occurs**: Without eager loading, each `c.orders` access triggers a separate SELECT (N+1). With `joinedload`, SQLAlchemy generates a single SELECT with a LEFT OUTER JOIN, loading all orders in one query.

### Real-World Cases

**Case 1: API Response with Related Data**: A REST API returns orders with customer names. Using `JOIN FETCH` in the repository query loads both entities in one query, reducing response time from 500ms to 50ms.

**Case 2: Reporting with Large Collections**: A report loads 10,000 customers, each with 50 orders. Joined eager loading would produce 500,000 rows (Cartesian product). Subquery eager loading emits two queries and aggregates in memory.

**Case 3: Spring Data JPA with EntityGraph**: A Spring Data repository method uses `@EntityGraph(attributePaths = {"items", "customer"})` to eagerly load both relationships, eliminating N+1 queries for an order dashboard.

---

## Core Concept 6: N+1 Query Problem

### Definitions

**Core Definition**: The N+1 query problem occurs when an application executes one query to fetch N parent entities, then executes N additional queries to fetch a related entity for each parent.

**Technical Definition**: The N+1 query problem (issuing one database query per element of a collection instead of a single batched query) arises when lazy loading is triggered inside a loop or serialization context. Runtime analysis tools like Doctrine Doctor detect N+1 queries by intercepting queries at the JDBC layer and analyzing execution context. Mitigation strategies include JOIN FETCH, `@EntityGraph`, `@BatchSize`, and eager loading.

**Beginner-Friendly Explanation**: Imagine you're a librarian asked to find 100 books and, for each book, its author. Instead of getting all 100 books and their authors in one trip, you get each book, then walk back to the author section 100 times. That's N+1 — one trip for the list plus one trip per item.

### Purposes

- **To** recognize the most common ORM performance anti-pattern
- **To** detect N+1 queries through query logging and profiling tools
- **To** mitigate N+1 with eager loading, batch fetching, or DTO projections
- **To** maintain awareness of lazy loading pitfalls in application code

### Syntax Rules and Structure

#### Detection: SQL Logging (Hibernate)

```properties
# application.properties (Spring Boot)
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql=TRACE
```

#### Detection: Hibernate Statistics

```java
// Enable statistics
SessionFactory sessionFactory = entityManagerFactory.unwrap(SessionFactory.class);
Statistics stats = sessionFactory.getStatistics();
stats.setStatisticsEnabled(true);

// After query
System.out.println("Query count: " + stats.getQueryExecutionCount());
System.out.println("Entity load count: " + stats.getEntityLoadCount());
```

#### Mitigation: JOIN FETCH (JPQL)

```java
@Query("SELECT DISTINCT o FROM Order o JOIN FETCH o.customer WHERE o.status = :status")
List<Order> findOrdersWithCustomer(@Param("status") String status);
```

#### Mitigation: @BatchSize (Hibernate)

```java
@Entity
public class Customer {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "customer")
    @BatchSize(size = 100)  // Load 100 customers' orders at once
    private List<Order> orders;
}
```

#### Mitigation: @EntityGraph (JPA)

```java
@EntityGraph(attributePaths = {"customer", "items"})
List<Order> findByStatus(String status);
```

#### Component Breakdown

| Strategy | Mechanism | Query Count |
|----------|-----------|-------------|
| **JOIN FETCH** | Single query with JOIN | 1 |
| **@EntityGraph** | Single query with JOIN | 1 |
| **@BatchSize** | Batched IN queries | N/batch_size + 1 |
| **DTO Projection** | Single query with selected columns | 1 |

#### Syntax Rules

- `JOIN FETCH` must be used with `DISTINCT` when fetching a collection to avoid duplicate parent rows.
- `@BatchSize` on a collection triggers batched IN queries when lazy loading is accessed.
- DTO projections (using `SELECT new Dto(...)`) avoid entity hydration entirely and are the most efficient mitigation.
- N+1 detection should measure the actual query count, not just enable SQL logging.

#### Constraints and Limitations

- `JOIN FETCH` with multiple collections causes a Cartesian product; use `@BatchSize` or `@EntityGraph` for multiple collections.
- `@BatchSize` only applies to lazy loading; it does not help eager loading.
- DTO projections bypass the ORM's entity management, so they cannot be used for updates.
- Setting `FetchType.EAGER` globally does not solve N+1 — it may make it worse by loading related data for every query.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: N+1 Detection and Fix with JOIN FETCH

```java
import javax.persistence.*;
import java.util.List;

// Step 1: The N+1 problem — lazy loading in a loop
public class OrderService {
    public void printOrdersWithCustomers() {
        List<Order> orders = em.createQuery(
            "SELECT o FROM Order o", Order.class).getResultList();
        
        for (Order order : orders) {
            // Each access triggers a separate SELECT for customer
            System.out.println(order.getOrderNumber() + " -> " 
                + order.getCustomer().getName());
        }
    }
}

// Step 2: The fix — JOIN FETCH
public class FixedOrderService {
    public void printOrdersWithCustomers() {
        List<Order> orders = em.createQuery(
            "SELECT DISTINCT o FROM Order o JOIN FETCH o.customer", 
            Order.class).getResultList();
        
        for (Order order : orders) {
            // No additional query — customer is already loaded
            System.out.println(order.getOrderNumber() + " -> " 
                + order.getCustomer().getName());
        }
    }
}
```

**Expected Output** (N+1):
```
Hibernate: select o.* from orders o
Hibernate: select c.* from customers c where c.id = 1
Hibernate: select c.* from customers c where c.id = 2
Hibernate: select c.* from customers c where c.id = 3
...
```

**Expected Output** (JOIN FETCH):
```
Hibernate: select distinct o.*, c.* from orders o 
           inner join customers c on o.customer_id = c.id
```

**Why This Output Occurs**: Without `JOIN FETCH`, Hibernate issues one query for the orders and then one query per order to load the customer (N+1). With `JOIN FETCH`, a single query loads both orders and customers in one result set.

### Real-World Cases

**Case 1: Laravel Eloquent N+1**: A Laravel controller loads 100 users and accesses `$user->posts->count()` for each. This triggers 100 additional queries. The fix is `User::with('posts')->get()`.

**Case 2: Hibernate Statistics Detection**: A development team enables Hibernate statistics and notices that a single page request executes 500 queries. Investigation reveals N+1 in a lazy-loaded collection. `@BatchSize(size = 50)` reduces it to 11 queries.

**Case 3: Doctrine Doctor in Symfony**: A Symfony application uses Doctrine Doctor, which detects N+1 queries at runtime and suggests eager loading with JOIN. The query count drops from 100 to 1.

---

## Core Concept 7: ORM-Generated SQL

### Definitions

**Core Definition**: ORM-generated SQL is the actual SQL statement that the ORM produces and sends to the database for a given object operation or query.

**Technical Definition**: ORMs translate object operations into SQL using their query engines. Inspecting generated SQL involves enabling SQL logging (`hibernate.show_sql`, `spring.jpa.show-sql`), profiling execution plans, and identifying issues such as missing join conditions, redundant queries, parameter patterns that prevent plan reuse, and implicit conversions that invalidate indexes. Native SQL overrides allow replacing ORM-generated SQL with hand-tuned queries for critical operations.

**Beginner-Friendly Explanation**: The ORM writes SQL for you behind the scenes. Inspecting it is like reading the translator's notes — you can see exactly what SQL was sent and whether it's efficient. Sometimes you need to override the ORM's SQL with your own better version.

### Purposes

- **To** verify that the ORM generates efficient SQL for critical queries
- **To** detect missing indexes, implicit conversions, and redundant queries
- **To** correlate ORM-generated SQL with execution plans and performance metrics
- **To** override ORM SQL with native queries for performance-critical operations

### Syntax Rules and Structure

#### Complete General Syntax (Hibernate SQL Logging)

```properties
# application.properties (Spring Boot)
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true

# Log bind parameters (careful in production)
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql=TRACE
```

#### Complete General Syntax (Native SQL Override — JPA)

```java
@Query(value = "SELECT * FROM orders WHERE customer_id = :customerId", 
       nativeQuery = true)
List<Order> findOrdersByCustomerNative(@Param("customerId") Long customerId);
```

#### Complete General Syntax (Native SQL Override — Hibernate)

```java
Session session = sessionFactory.openSession();
SQLQuery query = session.createSQLQuery(
    "SELECT * FROM orders WHERE customer_id = :customerId");
query.setParameter("customerId", customerId);
List<Order> orders = query.addEntity(Order.class).list();
```

#### Component Breakdown

| Inspection Method | Output | Use Case |
|-------------------|--------|----------|
| `hibernate.show_sql` | SQL to console | Development debugging |
| `org.hibernate.SQL=DEBUG` | SQL to log | Production-safe logging |
| `org.hibernate.type=TRACE` | Bind parameters | Detailed debugging |
| `EXPLAIN PLAN` | Execution plan | Performance tuning |
| `p6spy` | Intercepted SQL | Framework-agnostic profiling |

#### Syntax Rules

- `hibernate.show_sql` is a quick alternative to setting the `org.hibernate.SQL` logger to debug.
- `hibernate.format_sql` pretty-prints SQL for readability.
- Parameter logging (`org.hibernate.type.descriptor.sql=TRACE`) should be disabled in production to avoid leaking sensitive data.
- Native SQL overrides must map results back to entities or DTOs.

#### Constraints and Limitations

- SQL logging adds overhead; use only in development or with sampling in production.
- Native SQL is database-specific and may break if the schema changes.
- ORM-generated SQL may differ across database dialects; always test with the production database engine.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Logging and Execution Plan Analysis

```properties
# Step 1: Enable SQL logging in application.properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
```

```java
// Step 2: Execute a query and inspect the generated SQL
public class ProductRepository {
    public List<Product> findExpensiveProducts(double threshold) {
        return em.createQuery(
            "SELECT p FROM Product p WHERE p.price > :threshold ORDER BY p.name",
            Product.class)
            .setParameter("threshold", threshold)
            .getResultList();
    }
}
```

**Expected Output**:
```
Hibernate: 
    select
        product0_.id as id1_0_,
        product0_.name as name2_0_,
        product0_.price as price3_0_ 
    from
        products product0_ 
    where
        product0_.price > ? 
    order by
        product0_.name
```

```sql
-- Step 3: Take the generated SQL and run EXPLAIN PLAN
EXPLAIN ANALYZE
SELECT id, name, price FROM products WHERE price > 100 ORDER BY name;
```

**Expected Output**:
```
Sort  (cost=100.00..120.00 rows=1000 width=40) (actual time=0.500..0.600 rows=500 loops=1)
  Sort Key: name
  ->  Index Scan using idx_products_price on products  (cost=0.00..80.00 rows=1000 width=40)
        Index Cond: (price > 100)
Planning Time: 0.100 ms
Execution Time: 0.800 ms
```

**Why This Output Occurs**: The SQL logger shows the exact SQL generated by Hibernate. Running `EXPLAIN ANALYZE` on that SQL reveals the execution plan, confirming that the `idx_products_price` index is used.

#### Example 2: Native SQL Override for Performance

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface OrderRepository extends JpaRepository<Order, Long> {
    
    // ORM-generated SQL (may be suboptimal)
    List<Order> findByCustomerIdAndStatus(Long customerId, String status);
    
    // Native SQL override for performance
    @Query(value = """
        SELECT o.* FROM orders o
        INNER JOIN customers c ON o.customer_id = c.id
        WHERE o.customer_id = :customerId
          AND o.status = :status
          AND c.active = true
        ORDER BY o.created_at DESC
        LIMIT 100
        """, nativeQuery = true)
    List<Order> findActiveCustomerOrders(
        @Param("customerId") Long customerId,
        @Param("status") String status);
}
```

**Expected Output** (native SQL):
```
Hibernate: 
    SELECT o.* FROM orders o
    INNER JOIN customers c ON o.customer_id = c.id
    WHERE o.customer_id = ? AND o.status = ? AND c.active = true
    ORDER BY o.created_at DESC LIMIT 100
```

**Why This Output Occurs**: The `@Query` annotation with `nativeQuery = true` bypasses Hibernate's query generation and sends the exact SQL string to the database. This allows hand-tuning of joins, filters, and limits that the ORM may not generate optimally.

### Real-World Cases

**Case 1: Detecting Implicit Conversions**: SQL logging reveals that `WHERE product_code = 123` generates a string-to-number conversion because the column is `VARCHAR`. The implicit conversion prevents index usage, and the fix is to pass the parameter as a string.

**Case 2: Missing Join Condition**: SQL logging shows a query with a Cartesian product because a join condition was omitted. The fix is to add the missing `ON` clause or use an explicit `JOIN FETCH`.

**Case 3: SQL Profile for Oracle**: A DBA uses `DBMS_SQLTUNE.IMPORT_SQL_PROFILE` to apply statistical corrections to a suboptimal ORM-generated query without rewriting the application code.

---

## Core Concept 8: Cache Management

### Definitions

**Core Definition**: ORM cache management involves storing entity data in memory to reduce database queries and improve application performance.

**Technical Definition**: Hibernate and EF provide two levels of caching: the **first-level cache** (also known as the session cache or ObjectStateManager) is enabled by default and tracks all entities loaded within a single session/context. The **second-level cache** (L2 cache) is optional and shared across sessions, storing entity data at the SessionFactory or application level. Entity Framework also provides query plan caching and metadata caching.

**Beginner-Friendly Explanation**: First-level cache is like your shopping cart — everything you put in it stays there for that trip to the store. Second-level cache is like the store's warehouse — items are shared across all shoppers (sessions) and persist beyond a single trip.

### Purposes

- **To** reduce database round-trips by serving repeated reads from memory
- **To** maintain entity identity within a session (one row = one object)
- **To** share frequently accessed, rarely modified data across sessions (L2 cache)
- **To** improve application throughput and reduce database load

### Syntax Rules and Structure

#### Complete General Syntax (Hibernate Second-Level Cache Configuration)

```properties
# Enable second-level cache
hibernate.cache.use_second_level_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

# Enable query cache (optional)
hibernate.cache.use_query_cache=true
```

#### Entity-Level Cache Configuration

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    private String name;
    private double price;
    // getters and setters
}
```

#### Component Breakdown

| Cache Level | Scope | Default | Shared Across Sessions |
|-------------|-------|---------|------------------------|
| **First-level** | Session/EntityManager | Enabled | No |
| **Second-level** | SessionFactory | Disabled | Yes |
| **Query cache** | SessionFactory | Disabled | Yes (requires L2) |

#### Cache Concurrency Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `READ_ONLY` | Immutable data, no updates | Reference data (countries, categories) |
| `READ_WRITE` | Read-write with soft locks | Frequently read, occasionally updated |
| `NONSTRICT_READ_WRITE` | No locking, eventual consistency | Rarely updated, tolerance for staleness |
| `TRANSACTIONAL` | Full transactional (JTA) | Critical data, clustered environments |

#### Syntax Rules

- The first-level cache cannot be disabled; it is an integral part of the persistence context.
- The second-level cache is disabled unless `hibernate.cache.region.factory_class` is explicitly specified.
- Entities must be annotated with `@Cacheable` and `@Cache` to participate in L2 caching.
- The query cache stores query results (IDs) and requires L2 cache for entity data.
- Two different `EntityManager` instances have separate first-level caches.

#### Constraints and Limitations

- Second-level cache is not suitable for frequently updated data; stale data may be served.
- Distributed L2 caches (Redis, Infinispan) add network latency.
- The query cache can cause stale results if not properly invalidated.
- `Find()` in EF Core checks the first-level cache before querying, but this can be expensive with large object graphs.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Second-Level Cache with EhCache

```java
import javax.persistence.*;
import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

// Step 1: Annotate entity for L2 caching
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private double price;
    // getters and setters
}

// Step 2: Configure Hibernate (persistence.xml or application.properties)
// hibernate.cache.use_second_level_cache=true
// hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

// Step 3: Test L2 cache behavior
public class CacheTest {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPU");
        
        // First session: loads from database
        EntityManager em1 = emf.createEntityManager();
        Product p1 = em1.find(Product.class, 1L);
        System.out.println("First load: " + p1.getName());
        em1.close();
        
        // Second session: loads from L2 cache (no database hit)
        EntityManager em2 = emf.createEntityManager();
        Product p2 = em2.find(Product.class, 1L);
        System.out.println("Second load: " + p2.getName());
        em2.close();
        
        emf.close();
    }
}
```

**Expected Output**:
```
Hibernate: select p.* from products p where p.id = ?
First load: Laptop
Second load: Laptop
```

**Why This Output Occurs**: The first `find()` hits the database and stores the entity in the L2 cache. The second `find()` in a different session retrieves the entity from the L2 cache without a database query. The SQL is logged only once.

### Real-World Cases

**Case 1: Reference Data with READ_ONLY**: A `Country` entity is loaded frequently but never updated. The `READ_ONLY` cache strategy provides maximum performance and safety.

**Case 2: Product Catalog with READ_WRITE**: A product catalog is read frequently and updated occasionally. `READ_WRITE` strategy ensures that updates invalidate the cache while allowing concurrent reads.

**Case 3: Distributed Cache with Redis**: A clustered application uses Redisson as the Hibernate L2 cache provider, sharing cached entities across all application instances via Redis.

---

## Core Concept 9: Database Migrations

### Definitions

**Core Definition**: Database migrations are version-controlled scripts that apply incremental schema changes to a database, enabling consistent deployment across environments.

**Technical Definition**: Database migration tools like Flyway and Liquibase manage schema evolution by applying versioned SQL or Java migration scripts in order. Code-first migrations (Entity Framework, Rails) generate migrations from model changes, while migration-first approaches (Flyway, Prisma) require explicit migration authoring. The expand/contract pattern enables zero-downtime schema changes: add new columns nullable, dual-write, backfill, switch reads, and finally remove old columns.

**Beginner-Friendly Explanation**: Database migrations are like Git for your database schema. Each change (add a column, create a table) is a versioned script. The migration tool applies them in order, so every environment (dev, staging, production) has the same schema. Code-first means the migration is generated from your code; migration-first means you write the SQL yourself.

### Purposes

- **To** version-control database schema changes alongside application code
- **To** apply schema changes consistently across all environments
- **To** enable rollback and audit trails for schema modifications
- **To** support zero-downtime deployments through expand/contract patterns

### Syntax Rules and Structure

#### Complete General Syntax (Flyway Migration)

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V2__add_name_to_users.sql
ALTER TABLE users ADD COLUMN first_name VARCHAR(100);
ALTER TABLE users ADD COLUMN last_name VARCHAR(100);
```

#### Complete General Syntax (Entity Framework Code-First Migration)

```bash
# Step 1: Add a migration after model change
dotnet ef migrations add AddProductTable

# Step 2: Apply the migration to the database
dotnet ef database update
```

#### Complete General Syntax (Prisma Migration)

```bash
# Step 1: Create a migration from schema changes
npx prisma migrate dev --name add_product_table

# Step 2: Apply migrations in production
npx prisma migrate deploy
```

#### Component Breakdown

| Tool | Approach | Migration Format | Version Tracking |
|------|----------|------------------|------------------|
| **Flyway** | Migration-first | SQL files (`V1__name.sql`) | `flyway_schema_history` table |
| **Liquibase** | Migration-first | XML/YAML/SQL changelogs | `DATABASECHANGELOG` table |
| **EF Core** | Code-first | C# migration classes | `__EFMigrationsHistory` table |
| **Prisma** | Schema-first | Prisma schema + SQL | `_prisma_migrations` table |
| **Rails** | Code-first | Ruby migration classes | `schema_migrations` table |

#### Syntax Rules

- Migration files must be immutable once applied — never edit a migration that has been deployed; add a new migration instead.
- Migration names follow a versioned convention (e.g., `V1__`, `V2__`) to ensure ordering.
- Use the expand/contract pattern for zero-downtime changes: add new columns nullable, dual-write to both old and new, backfill, switch reads, and remove old columns.
- Always backup production databases before running migrations.
- Never modify migrations that have been pushed/deployed.

#### Constraints and Limitations

- Code-first migrations can hide risky changes until deployment; combine with a dedicated migration tool like Flyway for visibility.
- Destructive changes (drop column, drop table) are irreversible without a backup.
- Schema drift can occur if migrations are bypassed or manually applied.
- Migration locks must be respected in clustered environments to prevent concurrent execution.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Flyway Migration Workflow

```bash
# Step 1: Initialize Flyway (first time only)
flyway -url=jdbc:postgresql://localhost:5432/mydb \
       -user=app_user \
       -password=secret \
       baseline

# Step 2: Create migration files
# V1__create_customers_table.sql
cat > sql/V1__create_customers_table.sql << 'EOF'
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
EOF

# V2__add_name_columns.sql
cat > sql/V2__add_name_columns.sql << 'EOF'
ALTER TABLE customers ADD COLUMN first_name VARCHAR(100);
ALTER TABLE customers ADD COLUMN last_name VARCHAR(100);
EOF

# Step 3: Run migration
flyway -url=jdbc:postgresql://localhost:5432/mydb \
       -user=app_user \
       -password=secret \
       migrate

# Step 4: Check migration status
flyway -url=jdbc:postgresql://localhost:5432/mydb \
       -user=app_user \
       -password=secret \
       info
```

**Expected Output**:
```
Database: jdbc:postgresql://localhost:5432/mydb (PostgreSQL 14.5)
Schema version: 2
+-----------+---------+---------------------+------+---------------------+---------+
| Category  | Version | Description         | Type | Installed On        | State   |
+-----------+---------+---------------------+------+---------------------+---------+
| Versioned | 1       | create customers    | SQL  | 2026-10-07 10:00:00 | Success |
| Versioned | 2       | add name columns    | SQL  | 2026-10-07 10:01:00 | Success |
+-----------+---------+---------------------+------+---------------------+---------+
```

**Why This Output Occurs**: Flyway reads migration files in version order, applies them to the database, and records each execution in the `flyway_schema_history` table. The `info` command shows the migration history and current schema version.

### Real-World Cases

**Case 1: EF Core with Flyway Hybrid**: A .NET team uses EF Core Code First for development and Flyway for production deployments. EF generates the initial migration SQL, which is converted to Flyway scripts for controlled, versioned deployment.

**Case 2: Expand/Contract for Zero Downtime**: A high-traffic e-commerce platform renames a column without downtime: (1) add new column nullable, (2) deploy code that writes to both columns, (3) backfill old data, (4) switch reads to new column, (5) stop writing to old column, (6) drop old column.

**Case 3: Prisma Migrate in CI/CD**: A Node.js application uses Prisma Migrate to generate SQL migrations from schema changes, applies them in CI for testing, and uses `prisma migrate deploy` in production for controlled schema evolution.

---

## References

| Name | Link |
|------|------|
| Martin Fowler — Patterns of Enterprise Application Architecture (Active Record, Data Mapper) | https://martinfowler.com/eaaCatalog/ |
| SQLAlchemy — Relationship Loading Techniques | https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html |
| Hibernate — Primary Key Generation Strategies | https://docs.hibernate.org/orm/6.6/userguide/html_single/ |
| JPA — Relationship Mapping Annotations | https://docs.oracle.com/javaee/7/tutorial/persistence-intro.htm |
| Hibernate — Lazy Loading and Proxies | https://docs.hibernate.org/orm/6.6/userguide/html_single/ |
| Microsoft Learn — Performance Considerations for EF | https://learn.microsoft.com/en-us/ef/ef6/fundamentals/performance/perf-whitepaper |
| Redgate Flyway — Simple Workflows for Flyway and Entity Framework Code First | https://www.red-gate.com/hub/product-learning/flyway/simple-workflows-for-flyway-and-entity-framework-code-first |
| Redgate Flyway — Best Practices for Production | https://documentation.red-gate.com/flyway |
| Doctrine Doctor — N+1 Query Detection | https://packagist.org/packages/ahmed-bhs/doctrine-doctor |
| Baeldung — Show Hibernate/JPA SQL Statements in Spring Boot | https://www.baeldung.com/hibernate-show-sql |
| Hibernate — Second-Level Cache Configuration | https://docs.hibernate.org/orm/6.6/userguide/html_single/ |
| NCache — Hibernate Second Level Cache | https://www.alachisoft.com/resources/docs/hibernate/ |
| Prisma — Relational Data Modeling | https://www.prisma.io/docs/orm/prisma-schema/data-model/relations |
| Laravel — Eloquent Relationships | https://laravel.com/docs/11.x/eloquent-relationships |
| OWASP — Query Parameterization Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html |