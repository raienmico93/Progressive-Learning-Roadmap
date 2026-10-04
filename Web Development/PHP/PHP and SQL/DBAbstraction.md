# Database Abstraction — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Database abstraction is the practice of hiding the underlying database implementation details (SQL syntax, driver-specific APIs, table schemas) behind an application-level interface, so that business logic interacts with domain objects rather than raw database operations.

**Technical Definition**

Database abstraction encompasses a spectrum of architectural patterns — including the Repository Pattern, Data-Access Objects, Data Mappers, Query Objects, and Service Layers — that collectively decouple domain logic from persistence concerns. The Repository Pattern introduces a collection-like interface for storing and retrieving domain entities, hiding SQL, PDO, and table names from the domain layer. A Data Mapper performs bidirectional transfer of data between a persistent data store and in-memory domain objects, keeping both independent of each other. A Query Object represents a database query as an object, allowing queries to be composed and reused without exposing SQL on the public API. A Service Layer defines an application's boundary with coarse-grained application services that orchestrate domain logic and persistence operations.

**Beginner-Friendly Explanation**

Imagine your application is a restaurant. The chef (business logic) should not have to know how the refrigerator (database) is wired, what brand it is, or how to repair it. Instead, the chef tells a sous-chef (the repository), "Give me the ingredients for dish #42." The sous-chef knows exactly how to open the fridge, find the ingredients, and hand them over. If you replace the refrigerator with a different model, the chef never notices — the sous-chef handles the change. Database abstraction is the set of patterns that create that sous-chef layer.

### Key Characteristics

- **Separation of Concerns:** Persistence logic is isolated from business rules, so changes to the database schema or driver do not ripple through the entire application.
- **Collection-Like Interfaces:** Repositories behave like in-memory collections of domain objects, with methods such as `find`, `add`, and `remove`.
- **Dependency Inversion:** The domain layer depends on repository interfaces, not concrete database classes, enabling swapping of persistence mechanisms (PDO to ORM, or a fake for testing).
- **SQL Encapsulation:** All SQL lives inside concrete repository or data-access implementations, never in controllers or services.
- **Testability:** In-memory fakes or mock repositories can replace real database connections in unit tests.
- **Flexibility:** The same domain logic can work with MySQL, PostgreSQL, SQLite, or even non-relational data sources without modification.

### Prerequisites

- **PHP 8.1+** with PDO and a driver extension enabled.
- **Understanding of PDO prepared statements** and parameter binding (CRUD with PHP).
- **Familiarity with interfaces and dependency injection** — repositories are defined as interfaces and injected into services.
- **Domain-Driven Design (DDD) awareness** (helpful but not required) — understanding entities, value objects, and aggregates clarifies why repositories exist.
- **Composer** for autoloading and dependency management.
- **A database schema** with at least one entity table (e.g., `users`, `products`) for the examples.

### Related Programming Areas

- **Secure Query Execution & Prepared Statements:** All repository implementations must use prepared statements internally to prevent SQL injection.
- **Data Integrity & Transactions:** Repositories and services participate in transaction boundaries, ensuring atomicity across multiple operations.
- **Advanced Query Patterns:** Pagination, eager loading, and query objects often operate at the repository or data-access layer.
- **Dependency Injection and SOLID Principles:** The Repository Pattern is a direct application of the Dependency Inversion Principle.
- **Object-Relational Mapping (ORM):** ORMs like Doctrine and Eloquent implement variations of these patterns internally.

### Core Concepts / Features

1. **Repository Pattern** — A collection-like interface that mediates between the domain and data mapping layers.
2. **Data-Access Classes (Data Mapper / DAO)** — Classes that encapsulate all SQL and translate between database rows and domain objects.
3. **Query Objects** — Objects that represent database queries, allowing composition and reuse without exposing SQL.
4. **Service-Layer Interaction** — Application services that orchestrate domain logic and coordinate repository operations.

---

## Core Concept 1: Repository Pattern

### Definitions

**Core Definition**

The Repository Pattern is a design pattern that mediates between the domain and data mapping layers, acting like an in-memory collection of domain objects.

**Technical Definition**

A repository hides how objects are stored and retrieved behind a collection-like interface, so domain code asks for entities without ever seeing SQL, PDO, or table names. It introduces a single object that behaves like an in-memory collection of a given entity type: clients `find`, `add`, and `remove` domain objects, and the repository translates those intentions into persistence operations. The domain layer depends only on a repository interface, while the concrete PDO-backed implementation lives at the edge of the system.

**Beginner-Friendly Explanation**

Think of a repository as a librarian. You tell the librarian "I need the book with ISBN 978-3-16-148410-0." You do not walk into the stacks yourself, do not know the Dewey Decimal System, and do not care whether the library stores books on shelves or in a digital archive. The librarian knows exactly where and how to find the book and hands it to you. If the library reorganises its shelves, your request remains the same.

### Purposes

- To hide the persistence mechanism (PDO, ORM, REST API) behind a domain-friendly interface.
- To centralise all query logic for a given entity type in one place, eliminating scattered SQL.
- To enable swapping the persistence layer without modifying business logic.
- To provide a collection-like API that matches how developers conceptually think about domain objects.
- To facilitate unit testing by allowing an in-memory fake repository to replace the real database.
- To enforce a clean separation between domain entities and database tables.

### Syntax Rules and Structure

**Complete General Syntax: Repository Interface**

```php
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    /** @return list<User> */
    public function findAll(int $limit = 50): array;
    public function findByEmail(string $email): ?User;
    public function add(User $user): void;
    public function remove(int $id): void;
}
```

**Component Breakdown:**

- `find(int $id): ?User` — Retrieves a single entity by its unique identifier. Returns `null` if not found.
- `findAll(int $limit = 50): array` — Retrieves a collection of entities, optionally limited.
- `findByEmail(string $email): ?User` — A specialised finder for a domain-specific query.
- `add(User $user): void` — Persists a new or updated entity.
- `remove(int $id): void` — Deletes an entity by its identifier.

**Complete General Syntax: Concrete PDO Repository**

```php
final class PdoUserRepository implements UserRepositoryInterface
{
    public function __construct(private readonly PDO $pdo) {}

    public function find(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users WHERE id = :id'
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        return $row ? $this->hydrate($row) : null;
    }

    public function add(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (email, display_name) VALUES (:email, :name)'
        );
        $stmt->execute([
            ':email' => $user->email,
            ':name'  => $user->displayName,
        ]);
    }

    private function hydrate(array $row): User
    {
        return new User(
            id: (int) $row['id'],
            email: $row['email'],
            displayName: $row['display_name'],
        );
    }
}
```

**Syntax Rules:**

- The interface must contain only methods that express domain intent, not SQL details. Avoid leaking `LIMIT`, `JOIN`, or `WHERE` into interface method names.
- The concrete repository must use prepared statements for all queries involving user input.
- Hydration (mapping database rows to domain objects) belongs inside the repository, not in the service or controller.
- The repository should not contain business logic; it should only perform data access.

**Constraints and Limitations:**

- **Over-abstraction risk:** Not every entity needs a repository. Simple CRUD applications may not benefit from the additional layer.
- **Leaky abstractions:** If the interface exposes methods like `findBySql(string $sql)`, the abstraction is broken.
- **Transaction boundary ambiguity:** Repositories do not manage transactions; the service layer or a dedicated transaction manager must coordinate across multiple repositories.
- **Query complexity:** Complex reporting queries that span multiple entities often do not fit neatly into a single-entity repository; a dedicated query object or read model is more appropriate.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Repository Interface and PDO Implementation**

```php
<?php
declare(strict_types=1);

// --- Domain Entity ---
final class User
{
    public function __construct(
        public readonly ?int $id,
        public readonly string $email,
        public readonly string $displayName,
    ) {}
}

// --- Repository Interface ---
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function findAll(int $limit = 50): array;
    public function findByEmail(string $email): ?User;
    public function add(User $user): void;
    public function remove(int $id): void;
}

// --- Concrete PDO Repository ---
final class PdoUserRepository implements UserRepositoryInterface
{
    public function __construct(private readonly PDO $pdo) {}

    public function find(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users WHERE id = :id'
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row ? $this->hydrate($row) : null;
    }

    public function findAll(int $limit = 50): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users ORDER BY id LIMIT :limit'
        );
        $stmt->bindValue(':limit', $limit, PDO::PARAM_INT);
        $stmt->execute();
        return array_map([$this, 'hydrate'], $stmt->fetchAll(PDO::FETCH_ASSOC));
    }

    public function findByEmail(string $email): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users WHERE email = :email'
        );
        $stmt->execute([':email' => $email]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row ? $this->hydrate($row) : null;
    }

    public function add(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (email, display_name) VALUES (:email, :name)'
        );
        $stmt->execute([
            ':email' => $user->email,
            ':name'  => $user->displayName,
        ]);
    }

    public function remove(int $id): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute([':id' => $id]);
    }

    private function hydrate(array $row): User
    {
        return new User(
            id: (int) $row['id'],
            email: $row['email'],
            displayName: $row['display_name'],
        );
    }
}

// --- Usage ---
$pdo = new PDO('mysql:host=127.0.0.1;dbname=app', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    display_name VARCHAR(100) NOT NULL
)');

$repo = new PdoUserRepository($pdo);

// Create
$repo->add(new User(null, 'alice@example.com', 'Alice'));

// Read
$user = $repo->findByEmail('alice@example.com');
echo "Found: {$user->displayName} ({$user->email})\n";

// Update (via add — repositories often merge insert/update)
$repo->add(new User($user->id, 'alice@newdomain.com', 'Alice Smith'));

// Verify
$updated = $repo->find($user->id);
echo "Updated: {$updated->displayName} ({$updated->email})\n";
```

**Expected Output:**

```
Found: Alice (alice@example.com)
Updated: Alice Smith (alice@newdomain.com)
```

**Why:** The `PdoUserRepository` encapsulates all SQL and hydration logic. The client (the script) interacts only with domain objects (`User`) and the repository interface. The SQL is hidden behind methods that express domain intent: `find`, `findByEmail`, `add`, `remove`.

**Example 2: Swapping Repository Implementations for Testing**

```php
<?php
// In-memory fake repository for unit tests
final class InMemoryUserRepository implements UserRepositoryInterface
{
    /** @var array<int, User> */
    private array $users = [];
    private int $nextId = 1;

    public function find(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }

    public function findAll(int $limit = 50): array
    {
        return array_slice(array_values($this->users), 0, $limit);
    }

    public function findByEmail(string $email): ?User
    {
        foreach ($this->users as $user) {
            if ($user->email === $email) return $user;
        }
        return null;
    }

    public function add(User $user): void
    {
        if ($user->id === null) {
            $user = new User($this->nextId++, $user->email, $user->displayName);
        }
        $this->users[$user->id] = $user;
    }

    public function remove(int $id): void
    {
        unset($this->users[$id]);
    }
}

// --- Service under test ---
final class UserRegistrationService
{
    public function __construct(private UserRepositoryInterface $users) {}

    public function register(string $email, string $name): User
    {
        if ($this->users->findByEmail($email) !== null) {
            throw new RuntimeException('Email already registered.');
        }
        $user = new User(null, $email, $name);
        $this->users->add($user);
        return $user;
    }
}

// --- Test with fake repository ---
$fakeRepo = new InMemoryUserRepository();
$service = new UserRegistrationService($fakeRepo);

$user = $service->register('test@example.com', 'Test User');
echo "Registered: {$user->displayName} (ID: {$user->id})\n";

try {
    $service->register('test@example.com', 'Duplicate');
} catch (RuntimeException $e) {
    echo "Error: {$e->getMessage()}\n";
}
```

**Expected Output:**

```
Registered: Test User (ID: 1)
Error: Email already registered.
```

**Why:** The `InMemoryUserRepository` implements the same interface as `PdoUserRepository` but stores data in a PHP array. The `UserRegistrationService` depends only on the interface, so it works identically with both implementations. Tests can run without a database, making them fast and deterministic.

### Real-World Cases

**Case 1: Krayin CRM's Repository Layer**

Krayin CRM uses the Repository Pattern on top of Eloquent to keep data access out of controllers. Every model gets a repository that wraps standard CRUD helpers and provides a clean place to add custom query methods.

**Case 2: Bagisto's Repository Pattern**

Bagisto, an open-source e-commerce framework, uses repositories as a crucial architectural component that abstracts database operations and promotes cleaner, more maintainable code. Its repositories provide advanced features like criteria-based filtering, caching, and automatic query optimisation.

**Case 3: Laravel Repository Packages**

The Laravel ecosystem offers numerous repository pattern packages (e.g., `darwinnatha/laravel-repository-pattern`, `surazdott/laravel-repository`) that decouple the data layer from controllers, standardise query methods, and include auto-binding out of the box. These packages demonstrate the pattern's popularity in framework-driven PHP development.

---

## Core Concept 2: Data-Access Classes (Data Mapper / DAO)

### Definitions

**Core Definition**

A Data Mapper is a Data Access Layer that performs bidirectional transfer of data between a persistent data store (often a relational database) and an in-memory data representation (the domain layer), keeping both independent of each other.

**Technical Definition**

A Data Mapper is a layer of software that separates the in-memory representation of domain objects from the database, so that changes to one do not affect the other. The mapper handles all SQL and translates between database rows and domain entities. The layer is composed of one or more mappers (or Data Access Objects), performing the data transfer. Generic mappers handle many different domain entity types; dedicated mappers handle one or a few. A Data Access Object (DAO) is similar but typically operates at a lower level, providing CRUD operations without necessarily mapping to rich domain objects.

**Beginner-Friendly Explanation**

Think of a data mapper as a translator between two people who speak different languages. The domain object speaks "PHP" (with properties like `$user->email`), and the database speaks "SQL" (with columns like `email`). The data mapper translates back and forth. Neither the domain object nor the database needs to know the other's language.

### Purposes

- To keep domain objects free of persistence concerns — domain classes are Plain Old PHP Objects (POPOs) with no database knowledge.
- To isolate all SQL and database-specific code in one place, making schema changes easier to manage.
- To enable the same domain object to be persisted to different storage systems (MySQL, PostgreSQL, a REST API) by swapping mappers.
- To provide a clean separation between the domain model and the database schema, allowing them to evolve independently.
- To support complex mapping scenarios (nested objects, collections, value objects) that a simple active-record pattern cannot handle.

### Syntax Rules and Structure

**Complete General Syntax: Data Mapper**

```php
final class UserMapper
{
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users WHERE id = :id'
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row ? $this->mapRowToEntity($row) : null;
    }

    public function save(User $user): void
    {
        if ($user->id === null) {
            $this->insert($user);
        } else {
            $this->update($user);
        }
    }

    public function delete(User $user): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute([':id' => $user->id]);
    }

    private function insert(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (email, display_name) VALUES (:email, :name)'
        );
        $stmt->execute([':email' => $user->email, ':name' => $user->displayName]);
    }

    private function update(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'UPDATE users SET email = :email, display_name = :name WHERE id = :id'
        );
        $stmt->execute([
            ':email' => $user->email,
            ':name'  => $user->displayName,
            ':id'    => $user->id,
        ]);
    }

    private function mapRowToEntity(array $row): User
    {
        return new User(
            id: (int) $row['id'],
            email: $row['email'],
            displayName: $row['display_name'],
        );
    }
}
```

**Component Breakdown:**

- `findById(int $id): ?User` — Retrieves a single domain entity by identifier, mapping the database row to a `User` object.
- `save(User $user): void` — Determines whether the entity is new (insert) or existing (update) and persists accordingly. This "unit of work" style merges insert and update.
- `delete(User $user): void` — Removes the entity from the database.
- `mapRowToEntity(array $row): User` — The hydration method that translates database row values into domain object properties.
- `insert` / `update` — Private methods that contain the actual SQL for each operation.

**Syntax Rules:**

- Domain entities must not extend a database base class or contain `save()` or `delete()` methods. Those belong to the mapper.
- All SQL must use prepared statements with parameter binding.
- The mapper should be the only class aware of both the database schema and the domain model.
- Use a dedicated mapper per aggregate root (e.g., `UserMapper`, `OrderMapper`). Avoid generic mappers that handle every entity type.

**Constraints and Limitations:**

- **More code than Active Record:** Data Mapper requires separate entity classes and mapper classes, increasing the number of files compared to Active Record.
- **Mapping complexity:** Nested objects, collections, and inheritance require additional mapping logic that can become complex.
- **No built-in lazy loading:** Unlike ORMs with proxy objects, a simple Data Mapper does not provide automatic lazy loading of relationships; this must be implemented explicitly.
- **Unit of Work pattern:** For complex transactions spanning multiple mappers, a Unit of Work pattern is needed to track changes and flush them in the correct order.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Data Mapper for a User Entity**

```php
<?php
declare(strict_types=1);

// --- Domain Entity (POPO) ---
final class User
{
    public function __construct(
        private ?int $id,
        private string $email,
        private string $displayName,
    ) {}

    public function getId(): ?int { return $this->id; }
    public function getEmail(): string { return $this->email; }
    public function getDisplayName(): string { return $this->displayName; }

    public function changeEmail(string $newEmail): void
    {
        if (!filter_var($newEmail, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email.');
        }
        $this->email = $newEmail;
    }
}

// --- Data Mapper ---
final class UserMapper
{
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, display_name FROM users WHERE id = :id'
        );
        $stmt->execute([':id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row ? $this->mapRowToEntity($row) : null;
    }

    public function save(User $user): void
    {
        if ($user->getId() === null) {
            $stmt = $this->pdo->prepare(
                'INSERT INTO users (email, display_name) VALUES (:email, :name)'
            );
            $stmt->execute([
                ':email' => $user->getEmail(),
                ':name'  => $user->getDisplayName(),
            ]);
        } else {
            $stmt = $this->pdo->prepare(
                'UPDATE users SET email = :email, display_name = :name WHERE id = :id'
            );
            $stmt->execute([
                ':email' => $user->getEmail(),
                ':name'  => $user->getDisplayName(),
                ':id'    => $user->getId(),
            ]);
        }
    }

    private function mapRowToEntity(array $row): User
    {
        return new User(
            id: (int) $row['id'],
            email: $row['email'],
            displayName: $row['display_name'],
        );
    }
}

// --- Usage ---
$pdo = new PDO('mysql:host=127.0.0.1;dbname=app', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    display_name VARCHAR(100) NOT NULL
)');

$mapper = new UserMapper($pdo);

$user = new User(null, 'bob@example.com', 'Bob');
$mapper->save($user);

$loaded = $mapper->findById(1);
echo "Loaded: {$loaded->getDisplayName()} ({$loaded->getEmail()})\n";

$loaded->changeEmail('bob.new@example.com');
$mapper->save($loaded);

$reloaded = $mapper->findById(1);
echo "After update: {$reloaded->getEmail()}\n";
```

**Expected Output:**

```
Loaded: Bob (bob@example.com)
After update: bob.new@example.com
```

**Why:** The `User` entity is a plain PHP object with no database knowledge. All persistence logic lives in `UserMapper`. The entity's `changeEmail()` method enforces a domain rule (valid email format) without involving the database.

### Real-World Cases

**Case 1: Doctrine ORM**

Doctrine 2, the most widely used PHP ORM, is built on the Data Mapper pattern. Domain classes are Plain Old PHP Objects that do not extend any abstract base class. A separate mapper layer abstracts the database as generic storage.

**Case 2: Phalcon Data Mapper**

Phalcon provides a `Phalcon\DataMapper` namespace with components that help access data sources using the Data Mapper pattern, keeping domain objects independent of the database and the mapper itself.

**Case 3: Custom Data Mapper in a Legacy Migration**

A team migrating a legacy application from raw `mysql_*` queries to PDO uses Data Mappers to isolate all database code. Each legacy domain class is refactored to remove database calls, and a new mapper class is created to handle persistence. This allows the team to migrate incrementally without rewriting the entire application.

---

## Core Concept 3: Query Objects

### Definitions

**Core Definition**

A Query Object is an object that represents a database query, allowing queries to be composed, reused, and executed without exposing SQL on the public API.

**Technical Definition**

A Query Object is an interpreter (in the Gang of Four sense) — a structure of objects that can form itself into a SQL query. Queries are created by referring to classes and fields rather than tables and columns. This allows those who write the queries to do so independently of the database schema, and changes to the schema can be localised in a single place. When using a Query Object, there is no SQL on the public API at all; the transformation of the Query Object into the database vendor's appropriate SQL happens under the hood.

**Beginner-Friendly Explanation**

Imagine you are ordering a custom sandwich. Instead of telling the deli worker "put the turkey on the bread, then the lettuce, then the tomato" (raw SQL), you fill out a form with checkboxes: "Bread: white, Meat: turkey, Veggies: lettuce, tomato." The form (the Query Object) represents your order. The deli worker translates it into the actual sandwich-making steps. If the deli changes its workflow, you still fill out the same form.

### Purposes

- To encapsulate complex query logic in reusable, composable objects.
- To allow queries to be built programmatically without string concatenation or raw SQL in business logic.
- To centralise schema knowledge so that table or column renames affect only the query object classes.
- To provide a type-safe, IDE-friendly way to construct queries.
- To support dynamic query composition based on runtime conditions (filters, sorting, pagination).

### Syntax Rules and Structure

**Complete General Syntax: Query Object with Fluent Interface**

```php
final class FindActiveUsersQuery
{
    private ?string $role = null;
    private ?string $searchTerm = null;
    private int $limit = 50;

    public function withRole(string $role): self
    {
        $this->role = $role;
        return $this;
    }

    public function search(string $term): self
    {
        $this->searchTerm = $term;
        return $this;
    }

    public function limit(int $limit): self
    {
        $this->limit = $limit;
        return $this;
    }

    public function toSql(): string
    {
        $sql = 'SELECT id, email, display_name, role FROM users WHERE active = 1';
        if ($this->role !== null) {
            $sql .= ' AND role = :role';
        }
        if ($this->searchTerm !== null) {
            $sql .= ' AND (email LIKE :term OR display_name LIKE :term)';
        }
        $sql .= ' ORDER BY display_name LIMIT :limit';
        return $sql;
    }

    public function getParameters(): array
    {
        $params = [':limit' => $this->limit];
        if ($this->role !== null) {
            $params[':role'] = $this->role;
        }
        if ($this->searchTerm !== null) {
            $params[':term'] = '%' . $this->searchTerm . '%';
        }
        return $params;
    }
}
```

**Component Breakdown:**

- `withRole(string $role): self` — A setter that returns `$this` for method chaining. It configures the query to filter by role.
- `search(string $term): self` — Configures a text search across email and display name.
- `limit(int $limit): self` — Sets the maximum number of rows.
- `toSql(): string` — Builds the SQL string based on the configured options.
- `getParameters(): array` — Returns the bound parameters corresponding to the placeholders in the SQL.

**Syntax Rules:**

- All configuration methods should return `$this` to enable fluent chaining.
- The `toSql()` method should never concatenate user values into the SQL string — only placeholders.
- `getParameters()` must return parameters that exactly match the placeholders in `toSql()`.
- The query object should not execute the query itself; execution belongs to a repository or a dedicated executor.

**Constraints and Limitations:**

- **Over-engineering simple queries:** For basic `SELECT * FROM table WHERE id = ?`, a query object adds unnecessary complexity.
- **Portability illusion:** Query objects can hide SQL but cannot guarantee portability across different database engines if they generate vendor-specific SQL.
- **Maintenance overhead:** Each new query variation requires a new query object or additional configuration methods.
- **Not a replacement for an ORM:** Query objects handle query construction; they do not handle entity mapping, identity, or relationships.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Fluent Query Object for User Search**

```php
<?php
declare(strict_types=1);

final class UserSearchQuery
{
    private array $conditions = [];
    private array $params = [];
    private string $orderBy = 'display_name';
    private string $orderDir = 'ASC';
    private int $limit = 50;
    private int $offset = 0;

    public function activeOnly(): self
    {
        $this->conditions[] = 'active = 1';
        return $this;
    }

    public function withRole(string $role): self
    {
        $this->conditions[] = 'role = :role';
        $this->params[':role'] = $role;
        return $this;
    }

    public function search(string $term): self
    {
        $this->conditions[] = '(email LIKE :term OR display_name LIKE :term)';
        $this->params[':term'] = '%' . $term . '%';
        return $this;
    }

    public function orderBy(string $column, string $direction = 'ASC'): self
    {
        $allowed = ['display_name', 'email', 'created_at'];
        if (!in_array($column, $allowed, true)) {
            throw new InvalidArgumentException("Invalid order column: $column");
        }
        $this->orderBy = $column;
        $this->orderDir = strtoupper($direction) === 'DESC' ? 'DESC' : 'ASC';
        return $this;
    }

    public function paginate(int $limit, int $offset = 0): self
    {
        $this->limit = max(1, min(100, $limit));
        $this->offset = max(0, $offset);
        return $this;
    }

    public function build(): array
    {
        $sql = 'SELECT id, email, display_name, role FROM users';
        if ($this->conditions) {
            $sql .= ' WHERE ' . implode(' AND ', $this->conditions);
        }
        $sql .= " ORDER BY {$this->orderBy} {$this->orderDir}";
        $sql .= ' LIMIT :limit OFFSET :offset';

        $params = $this->params;
        $params[':limit'] = $this->limit;
        $params[':offset'] = $this->offset;

        return [$sql, $params];
    }
}

// --- Usage ---
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE users (id INTEGER PRIMARY KEY, email TEXT, display_name TEXT, role TEXT, active INTEGER)');
$pdo->exec("INSERT INTO users (email, display_name, role, active) VALUES
    ('alice@example.com', 'Alice', 'admin', 1),
    ('bob@example.com', 'Bob', 'user', 1),
    ('carol@example.com', 'Carol', 'user', 0)");

$query = (new UserSearchQuery())
    ->activeOnly()
    ->withRole('user')
    ->search('bob')
    ->orderBy('display_name', 'ASC')
    ->paginate(10);

[$sql, $params] = $query->build();

$stmt = $pdo->prepare($sql);
foreach ($params as $key => $value) {
    $type = in_array($key, [':limit', ':offset'], true) ? PDO::PARAM_INT : PDO::PARAM_STR;
    $stmt->bindValue($key, $value, $type);
}
$stmt->execute();
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

echo "Query: $sql\n";
echo "Results: " . count($rows) . "\n";
foreach ($rows as $row) {
    echo "- {$row['display_name']} ({$row['email']})\n";
}
```

**Expected Output:**

```
Query: SELECT id, email, display_name, role FROM users WHERE active = 1 AND role = :role AND (email LIKE :term OR display_name LIKE :term) ORDER BY display_name ASC LIMIT :limit OFFSET :offset
Results: 1
- Bob (bob@example.com)
```

**Why:** The `UserSearchQuery` object builds the SQL and parameter array based on the configured conditions. The caller never writes SQL directly; they chain domain-friendly methods like `activeOnly()`, `withRole()`, and `search()`. The query object handles the translation to SQL and parameters.

### Real-World Cases

**Case 1: Zend Framework's `Zend_Db_Select`**

Zend Framework provides `Zend_Db_Select` as an internal DSL for creating SQL queries. Developers build queries by chaining methods like `->from()`, `->where()`, and `->order()`, and the object generates the appropriate SQL for the underlying database.

**Case 2: Doctrine Query Builder**

Doctrine's Query Builder is a Query Object around the DQL language. Developers call `$entityManager->createQueryBuilder()` and chain methods like `->select()`, `->from()`, `->where()`, and `->setParameter()` to construct queries programmatically.

**Case 3: Yii Framework Query Builder**

Yii's Query Builder allows developers to construct SQL queries programmatically: `(new \yii\db\Query())->select(['id', 'email'])->from('user')->where(['last_name' => 'Smith'])`. The Query Object is then executed to return rows.

---

## Core Concept 4: Service-Layer Interaction

### Definitions

**Core Definition**

A Service Layer defines an application's boundary with a set of coarse-grained application services that orchestrate domain logic and coordinate repository operations.

**Technical Definition**

The Service Layer contains the application's business logic and orchestrates operations between controllers (or API endpoints) and repositories. It abstracts the application's functionality from the controller, promoting better separation of concerns. The service layer is the only place where business rules and workflow logic should reside; repositories are responsible only for data access. A typical service method corresponds to one use case (e.g., "register a user," "place an order") and may coordinate multiple repositories, enforce business invariants, and manage transaction boundaries.

**Beginner-Friendly Explanation**

Think of a service layer as the "manager" in a restaurant. The manager does not cook the food (that is the chef's job) and does not wash the dishes (that is the dishwasher's job). Instead, the manager coordinates: "Table 5 needs two steaks and a salad. Chef, cook the steaks. Dishwasher, prepare clean plates. Waiter, take them to table 5." The service layer coordinates repositories, enforces business rules, and manages the overall flow of a use case.

### Purposes

- To provide a clear boundary between the presentation layer (controllers) and the domain/persistence layers.
- To encapsulate business logic in one place, preventing it from leaking into controllers or repositories.
- To orchestrate multiple repository operations within a single use case, ensuring consistency and transactionality.
- To enforce business invariants and authorisation rules before persisting changes.
- To make the application's capabilities explicit through named service methods (e.g., `registerUser`, `placeOrder`).

### Syntax Rules and Structure

**Complete General Syntax: Service Layer**

```php
final class UserRegistrationService
{
    public function __construct(
        private readonly UserRepositoryInterface $users,
        private readonly MailerInterface $mailer,
        private readonly PDO $pdo,
    ) {}

    public function register(string $email, string $name, string $password): User
    {
        // 1. Enforce business rules
        if ($this->users->findByEmail($email) !== null) {
            throw new DomainException('Email already registered.');
        }

        // 2. Begin transaction (coordinate multiple operations)
        $this->pdo->beginTransaction();

        try {
            // 3. Create the domain entity
            $user = new User(
                id: null,
                email: $email,
                displayName: $name,
            );

            // 4. Persist via repository
            $this->users->add($user);

            // 5. Perform side effects (email, audit log, etc.)
            $this->mailer->sendWelcome($email, $name);

            // 6. Commit
            $this->pdo->commit();
            return $user;
        } catch (Throwable $e) {
            // 7. Rollback on failure
            if ($this->pdo->inTransaction()) {
                $this->pdo->rollBack();
            }
            throw $e;
        }
    }
}
```

**Component Breakdown:**

- `UserRepositoryInterface $users` — The repository dependency, injected via constructor.
- `MailerInterface $mailer` — A side-effect dependency (email sending).
- `PDO $pdo` — The database connection for transaction management. In some architectures, a dedicated `TransactionManager` is injected instead.
- `register(string $email, string $name, string $password): User` — A coarse-grained use-case method. It enforces rules, coordinates repositories and side effects, and returns the created domain entity.
- `beginTransaction()`, `commit()`, `rollBack()` — Transaction boundaries managed at the service layer, not inside repositories.

**Syntax Rules:**

- Service methods should correspond to use cases, not CRUD operations. `registerUser` is a use case; `updateUserEmail` might be too fine-grained for a service.
- Services depend on repository interfaces, never on concrete PDO repositories.
- Transaction boundaries belong in the service layer, not in repositories. A single service method may span multiple repository calls.
- Services should not expose database specifics (SQL, table names) to controllers.
- Controllers should call services, never repositories directly. This is a strict architectural rule in reference architectures.

**Constraints and Limitations:**

- **Over-engineering simple applications:** Not every application needs a service layer. A simple CRUD application may be adequately served by repositories alone.
- **Anemic domain model risk:** If services contain all the business logic and entities are mere data bags, the domain model becomes anemic. Strive for rich domain objects that enforce their own invariants.
- **Transaction boundary decisions:** Deciding where transactions begin and end in the service layer requires careful design. Long-running services that span multiple repositories increase lock contention.
- **Testing complexity:** Service layer tests require mocking multiple dependencies (repositories, mailers, transaction managers).

### Multiple Annotated Step-by-Step Code Examples

**Example 1: User Registration Service with Repository Coordination**

```php
<?php
declare(strict_types=1);

// --- Interfaces ---
interface UserRepositoryInterface
{
    public function findByEmail(string $email): ?User;
    public function add(User $user): void;
}

interface MailerInterface
{
    public function sendWelcome(string $email, string $name): void;
}

// --- Service ---
final class UserRegistrationService
{
    public function __construct(
        private readonly UserRepositoryInterface $users,
        private readonly MailerInterface $mailer,
    ) {}

    public function register(string $email, string $name): User
    {
        // Business rule: email must be unique
        if ($this->users->findByEmail($email) !== null) {
            throw new DomainException('Email already registered.');
        }

        // Business rule: email must be valid
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new DomainException('Invalid email format.');
        }

        $user = new User(null, $email, $name);
        $this->users->add($user);

        // Side effect: send welcome email
        $this->mailer->sendWelcome($email, $name);

        return $user;
    }
}

// --- In-memory implementations for testing ---
final class InMemoryUserRepository implements UserRepositoryInterface
{
    private array $users = [];
    public function findByEmail(string $email): ?User
    {
        foreach ($this->users as $u) if ($u->email === $email) return $u;
        return null;
    }
    public function add(User $user): void { $this->users[] = $user; }
}

final class FakeMailer implements MailerInterface
{
    public array $sent = [];
    public function sendWelcome(string $email, string $name): void
    {
        $this->sent[] = compact('email', 'name');
    }
}

// --- Usage ---
$repo = new InMemoryUserRepository();
$mailer = new FakeMailer();
$service = new UserRegistrationService($repo, $mailer);

$user = $service->register('dave@example.com', 'Dave');
echo "Registered: {$user->displayName}\n";
echo "Welcome emails sent: " . count($mailer->sent) . "\n";

try {
    $service->register('dave@example.com', 'Duplicate');
} catch (DomainException $e) {
    echo "Error: {$e->getMessage()}\n";
}
```

**Expected Output:**

```
Registered: Dave
Welcome emails sent: 1
Error: Email already registered.
```

**Why:** The `UserRegistrationService` orchestrates the use case. It checks the business rule (email uniqueness) via the repository, creates the entity, persists it, and triggers the side effect (welcome email). The controller would call `$service->register(...)` without knowing anything about repositories or mailers.

**Example 2: Order Placement Service with Multiple Repositories and Transaction**

```php
<?php
final class OrderPlacementService
{
    public function __construct(
        private readonly OrderRepositoryInterface $orders,
        private readonly ProductRepositoryInterface $products,
        private readonly PDO $pdo,
    ) {}

    public function placeOrder(int $customerId, array $items): Order
    {
        if (empty($items)) {
            throw new DomainException('Order must contain at least one item.');
        }

        $this->pdo->beginTransaction();

        try {
            $order = new Order(id: null, customerId: $customerId, items: []);

            foreach ($items as $item) {
                $product = $this->products->find($item['product_id']);
                if ($product === null) {
                    throw new DomainException("Product {$item['product_id']} not found.");
                }
                if ($product->stock < $item['quantity']) {
                    throw new DomainException("Insufficient stock for {$product->name}.");
                }

                $this->products->decrementStock($product->id, $item['quantity']);
                $order->addItem($product, $item['quantity']);
            }

            $this->orders->add($order);
            $this->pdo->commit();
            return $order;
        } catch (Throwable $e) {
            if ($this->pdo->inTransaction()) {
                $this->pdo->rollBack();
            }
            throw $e;
        }
    }
}
```

**Expected Output:** No visible output (this is a service class).

**Why:** The `OrderPlacementService` coordinates two repositories (`OrderRepository` and `ProductRepository`) within a single transaction. It enforces business rules (stock availability, order non-empty) and ensures that if any step fails, all stock decrements and the order insertion are rolled back.

### Real-World Cases

**Case 1: Laravel Service + Repository Pattern**

A Laravel application uses a `UserService` that depends on a `UserRepositoryInterface`. The service contains business logic such as "a user must have a unique email" and "a welcome email must be sent on registration." Controllers call `$userService->create($request->all())`, and the service coordinates the repository and mailer. This pattern is widely adopted in the Laravel community.

**Case 2: Krayin CRM Architecture**

Krayin CRM follows a strict architectural rule: controllers never write direct SQL; instead, they call service methods that orchestrate repository operations. The repository handles data access, the service handles business logic, and the controller handles HTTP concerns.

**Case 3: CakePHP Service Layer**

The `burzum/cakephp-service-layer` package provides a service layer for CakePHP applications, improving maintainability by separating business logic from controllers and models. The service layer is described as "more a design pattern and conceptual idea than a lot of code," emphasising that the architecture matters more than the implementation.

---

## References

- PHP: Introduction — PDO Manual – https://static.php.net/manual/en/intro.pdo.php
- Secure PHP Development: Repository Pattern — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Architecture/Repository-Pattern.md
- Secure PHP Development: Service Layer — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Architecture/Service-Layer.md
- Martin Fowler: Query Object — https://martinfowler.com/eaaCatalog/queryObject.html
- Martin Fowler: Data Mapper — https://martinfowler.com/eaaCatalog/dataMapper.html
- Martin Fowler: Repository — https://martinfowler.com/eaaCatalog/repository.html
- Martin Fowler: Service Layer — https://martinfowler.com/eaaCatalog/serviceLayer.html
- Packagist: dealnews/data-mapper — https://packagist.org/packages/dealnews/data-mapper
- Stack Overflow: PHP write Program for whatever Database (Query Object discussion) – https://stackoverflow.com/questions/11274179
- Packagist: darwinnatha/laravel-repository-pattern — https://packagist.org/packages/darwinnatha/laravel-repository-pattern
- Packagist: surazdott/laravel-repository — https://packagist.org/packages/surazdott/laravel-repository
- Packagist: ysm/laravel-repository-pattern — https://packagist.org/packages/ysm/laravel-repository-pattern
- Packagist: struktal/struktal-orm (DAO pattern) – https://packagist.org/packages/struktal/struktal-orm
- Krayin CRM Developer Portal: Architecture Overview — https://devdocs.krayincrm.com
- Bagisto DevDocs: Repositories — https://devdocs.bagisto.com