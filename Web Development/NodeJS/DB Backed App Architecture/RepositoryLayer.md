# Data Access & Persistence Layer (Repository vs. Active Record) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The data access and persistence layer is the architectural tier responsible for translating between in-memory domain objects and durable storage (databases, file systems, external APIs), using patterns such as Repository, Active Record, Data Mapper, Query Objects, and Unit of Work to manage this translation.

**Technical Definition:** The persistence layer mediates between the domain model and the data mapping layer. Martin Fowler defines a Repository as mediating "between the domain and data mapping layers using a collection-like interface for accessing domain objects". The Active Record pattern places persistence methods directly on the domain object (the object knows how to save itself), while the Data Mapper pattern moves all persistence logic into separate mapper/repository classes, keeping domain objects "very dumb" and unaware of the database. Query Scopes and Criteria Objects encapsulate reusable query filters into isolated, composable, and testable specifications. The Unit of Work pattern "maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems".

**Beginner-Friendly Explanation:** Imagine your application's business logic is a set of "things your app does" — like creating orders, updating user profiles, or generating reports. The persistence layer is the part that figures out how to save and retrieve those things from a database. 
- The **Repository pattern** says: "Don't let your business logic talk to the database directly. Instead, talk to a repository — a collection-like interface that looks like an in-memory list of objects." 
- The **Active Record vs. Data Mapper** distinction asks: "Should the object itself know how to save itself (Active Record), or should a separate class handle that (Data Mapper)?" 
- **Query Scopes and Criteria Objects** solve the problem of messy, duplicated query logic by packaging filters into reusable, testable objects. 
- The **Unit of Work** pattern ensures that when you modify several objects across different repositories, all those changes either succeed together or fail together (ACID transactions).

### Key Characteristics

- **Decoupling:** Repositories hide database implementation details from business logic, enabling testability and swappable storage engines.
- **Two ORM philosophies:** Active Record (simplicity, model-centric) vs. Data Mapper (maintainability, separation of concerns).
- **Query abstraction:** Criteria and Specification objects encapsulate filters, sorting, and pagination into composable units.
- **Transactional integrity:** Unit of Work coordinates multiple repository operations into a single atomic transaction.
- **Testability without a database:** Repositories enable in-memory fakes for unit-testing business logic.
- **Persistence ignorance:** Data Mapper keeps domain objects free of SQL, database schemas, and persistence concerns.

### Prerequisites

- **Understanding of Object-Relational Mapping (ORM):** Familiarity with TypeORM, Prisma, Sequelize, or similar.
- **Database fundamentals:** Tables, rows, primary keys, foreign keys, and transactions.
- **Object-oriented programming:** Classes, interfaces, inheritance, and composition.
- **Dependency injection:** Constructor injection and inversion of control (common in NestJS/TypeScript).
- **Async/await:** Promises and asynchronous database operations.
- **ACID properties:** Atomicity, Consistency, Isolation, Durability.

### Related Programming Areas

- **Domain-Driven Design (DDD):** Repositories are a core tactical pattern in DDD, used to access aggregates.
- **Clean Architecture / Hexagonal Architecture:** Repository interfaces define "ports" that infrastructure "adapters" implement.
- **CQRS (Command Query Responsibility Segregation):** Query Scopes and Criteria Objects align with the "Query" side of CQRS.
- **Testing:** Repositories enable unit testing with in-memory fakes instead of integration tests against a real database.
- **Transaction Management:** Unit of Work is closely tied to database transaction APIs (TypeORM `QueryRunner`, Prisma `$transaction`).

### Core Concepts

1. **Repository Pattern Essentials** — decoupling business logic from the persistence engine using explicit repository interfaces and concrete implementations.
2. **Active Record vs. Data Mapper** — evaluating the architectural trade-offs of Eloquent/TypeORM (Active Record) vs. Doctrine/Hibernate (Data Mapper).
3. **Query Scopes & Criteria Objects** — abstracting complex, reusable query filters out of repositories into isolated, testable specifications.
4. **Unit of Work & Multi-Model Transactions** — managing multi-query transaction boundaries safely to ensure ACID compliance across distinct repositories.

---

## Core Concept 1: Repository Pattern Essentials

### Definitions

**Core Definition:** The Repository pattern mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects, encapsulating the set of objects persisted in a data store and the operations performed over them.

**Technical Definition:** A Repository mediates between the domain and data mapping layers, acting like an in-memory domain object collection. Client objects construct query specifications declaratively and submit them to the Repository for satisfaction. Objects can be added to and removed from the Repository, as they can from a simple collection of objects, and the mapping code encapsulated by the Repository will carry out the appropriate operations behind the scenes. Conceptually, a Repository encapsulates the set of objects persisted in a data store and the operations performed over them, providing a more object-oriented view of the persistence layer.

**Beginner-Friendly Explanation:** A Repository is like a librarian for your data. Instead of your business code going directly into the database stacks and searching for books (rows), you ask the librarian: "Please find me the user with email alice@example.com." The librarian knows exactly where and how to look, and you never need to know whether the books are stored on shelves, in boxes, or in a different library entirely. If you want to swap databases later, you just hire a new librarian with the same skills.

### Purposes

- To decouple the business layer from the persistence engine, allowing storage implementations to be swapped without modifying application logic.
- To provide a collection-like interface (`find`, `save`, `delete`) that mimics in-memory operations on domain objects.
- To concentrate query construction code in a single layer, minimising duplicate query logic across the application.
- To enable unit testing of business logic by substituting in-memory fakes for real database repositories.
- To enforce a clean separation and one-way dependency between the domain layer and the data mapping layer.

### Syntax Rules and Structure

#### General Syntax (TypeScript/Node.js)

**Repository interface (the contract):**
```typescript
interface IRepository<T> {
  findById(id: string): Promise<T | null>;
  findOne(filter: Partial<T>): Promise<T | null>;
  find(filter: Partial<T>, options?: { limit?: number; skip?: number }): Promise<T[]>;
  create(data: Partial<T>): Promise<T>;
  updateById(id: string, update: Partial<T>): Promise<T | null>;
  deleteById(id: string): Promise<boolean>;
  count(filter: Partial<T>): Promise<number>;
}
```

| Component | Breakdown |
|-----------|-----------|
| `IRepository<T>` | Generic interface for all repositories. |
| `findById` | Retrieve a single entity by its unique identifier. |
| `findOne` | Retrieve a single entity matching a filter. |
| `find` | Retrieve multiple entities with optional pagination. |
| `create` | Persist a new entity. |
| `updateById` | Update an existing entity. |
| `deleteById` | Remove an entity by ID. |
| `count` | Count entities matching a filter. |

**Concrete implementation (Mongoose example):**
```typescript
class BaseRepository<T extends Document> implements IRepository<T> {
  constructor(protected model: Model<T>) {}

  async findById(id: string): Promise<T | null> {
    return this.model.findById(id).lean() as Promise<T | null>;
  }

  async create(data: Partial<T>): Promise<T> {
    return this.model.create(data) as Promise<T>;
  }
  // ... other methods
}
```

**Domain-specific repository:**
```typescript
class UserRepository extends BaseRepository<IUser> {
  constructor() { super(User); }

  async findByEmail(email: string): Promise<IUser | null> {
    return this.model.findOne({ email: email.toLowerCase() }).lean();
  }
}
```

#### Syntax Rules

- Repository interfaces belong to the **domain layer**; implementations belong to the **infrastructure layer**.
- The business layer depends only on the interface, never on the concrete implementation.
- Repositories should expose domain-meaningful methods (`findActiveUsers()`) rather than leaking query-builder details (`createQueryBuilder()`).
- Generic base repositories reduce boilerplate but should not replace domain-specific query methods.

#### Constraints and Limitations

- Repositories must not leak persistence-specific types (e.g., `IQueryable<T>`, `QueryBuilder`) into the domain layer; doing so negates the abstraction's benefits.
- A repository should generally correspond to an **aggregate root**, not every table.
- Overly generic repositories (`IRepository<T>` with only CRUD) can become an anti-pattern if domain-specific queries are scattered elsewhere.
- Repository methods should return domain objects, not raw database rows or partial projections (unless using a read model).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Repository Pattern with In-Memory and Prisma Implementations

```typescript
// domain/User.ts
export interface User {
  id: string;
  name: string;
  email: string;
  status: 'active' | 'inactive';
}

// domain/IUserRepository.ts — the contract
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findActive(): Promise<User[]>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}
```

```typescript
// infrastructure/InMemoryUserRepository.ts — test double
export class InMemoryUserRepository implements IUserRepository {
  private users: User[] = [];

  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) ?? null;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.users.find(u => u.email === email) ?? null;
  }

  async findActive(): Promise<User[]> {
    return this.users.filter(u => u.status === 'active');
  }

  async save(user: User): Promise<User> {
    const index = this.users.findIndex(u => u.id === user.id);
    if (index >= 0) this.users[index] = user;
    else this.users.push(user);
    return user;
  }

  async delete(id: string): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
  }
}
```

```typescript
// infrastructure/PrismaUserRepository.ts — production implementation
import { PrismaClient } from '@prisma/client';
import { IUserRepository } from '../domain/IUserRepository';
import { User } from '../domain/User';

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async findActive(): Promise<User[]> {
    return this.prisma.user.findMany({ where: { status: 'active' } });
  }

  async save(user: User): Promise<User> {
    return this.prisma.user.upsert({
      where: { id: user.id },
      update: user,
      create: user,
    });
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }
}
```

```typescript
// application/UserService.ts — business logic depends only on the interface
import { IUserRepository } from '../domain/IUserRepository';
import { User } from '../domain/User';

export class UserService {
  constructor(private userRepository: IUserRepository) {}

  async activateUser(userId: string): Promise<User> {
    const user = await this.userRepository.findById(userId);
    if (!user) throw new Error('User not found');
    user.status = 'active';
    return this.userRepository.save(user);
  }

  async listActiveUsers(): Promise<User[]> {
    return this.userRepository.findActive();
  }
}
```

**Step-by-step setup:**
1. Define the domain entity `User`.
2. Define the repository interface `IUserRepository` in the domain layer.
3. Implement `InMemoryUserRepository` for testing.
4. Implement `PrismaUserRepository` for production.
5. Inject the repository into `UserService` via constructor.

**Expected behaviour:** In tests, `UserService` is constructed with `InMemoryUserRepository` — no database required. In production, it is constructed with `PrismaUserRepository`. The business logic in `UserService` never changes.

**Why this works:** The `UserService` depends on the `IUserRepository` abstraction, not on Prisma or MongoDB or any specific database. Swapping persistence engines requires only a new implementation of the interface.

#### Example 2: Domain-Specific Query Methods

```typescript
// domain/OrderRepository.ts
export interface OrderRepository {
  findPendingOrders(customerId: string): Promise<Order[]>;
  findOrdersAboveAmount(amount: number): Promise<Order[]>;
  save(order: Order): Promise<Order>;
}

// infrastructure/TypeOrmOrderRepository.ts
export class TypeOrmOrderRepository implements OrderRepository {
  constructor(private dataSource: DataSource) {}

  async findPendingOrders(customerId: string): Promise<Order[]> {
    return this.dataSource.getRepository(OrderEntity).find({
      where: { customerId, status: 'pending' },
      order: { createdAt: 'DESC' },
    });
  }

  async findOrdersAboveAmount(amount: number): Promise<Order[]> {
    return this.dataSource.getRepository(OrderEntity)
      .createQueryBuilder('order')
      .where('order.total > :amount', { amount })
      .getMany();
  }

  async save(order: Order): Promise<Order> {
    return this.dataSource.getRepository(OrderEntity).save(order);
  }
}
```

**Expected behaviour:** The domain layer calls `orderRepository.findPendingOrders(customerId)` without knowing whether the implementation uses TypeORM, raw SQL, or an external API.

### Real-World Cases

- **Multi-database applications:** A repository interface allows the same business logic to work with PostgreSQL in production and an in-memory database in tests.
- **Gradual migration:** A team migrating from MongoDB to PostgreSQL can implement a new repository while keeping the old one, switching at runtime.
- **Third-party integrations:** A repository can hide whether data comes from a local database, a REST API, or a GraphQL endpoint.
- **Clean Architecture / Hexagonal Architecture:** Repository interfaces are the "ports" that infrastructure "adapters" implement.

---

## Core Concept 2: Active Record vs. Data Mapper

### Definitions

**Core Definition:** Active Record places persistence methods directly on the domain entity (the object knows how to save itself), while Data Mapper moves all persistence logic into separate mapper or repository classes, keeping entities completely unaware of the database.

**Technical Definition:** The Active Record pattern is "an approach to access your database within your models" — entities extend a base class (e.g., TypeORM's `BaseEntity`) and expose methods like `.save()`, `.remove()`, and static query methods like `User.find()`. The Data Mapper pattern "is an approach to access your database within repositories instead of models" — entities are "very dumb" and only define properties, while repositories handle saving, removing, and loading. A Data Mapper is "a layer of software that separates the in-memory objects from the database. With Data Mapper the in-memory objects needn't know even that there's a database present; they need no SQL interface code, and certainly no knowledge of the database schema".

**Beginner-Friendly Explanation:** Active Record is like a self-service checkout — the product (entity) knows how to scan itself and process payment. Data Mapper is like a traditional checkout with a cashier — the product is just a product, and the cashier (repository) handles all the payment processing. Active Record is simpler for small projects because everything is in one place. Data Mapper is cleaner for large projects because the product doesn't need to know anything about how it gets sold.

### Purposes

- To choose the appropriate persistence pattern based on application size, complexity, and team structure.
- To evaluate the trade-offs between simplicity (Active Record) and maintainability (Data Mapper).
- To understand how different ORMs (TypeORM, Eloquent, Doctrine, Hibernate) implement these patterns.
- To make informed architectural decisions about where persistence logic should live.
- To align persistence architecture with domain-driven design principles.

### Syntax Rules and Structure

#### Active Record (TypeORM `BaseEntity`)

```typescript
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;

  static findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder('user')
      .where('user.firstName = :firstName', { firstName })
      .andWhere('user.lastName = :lastName', { lastName })
      .getMany();
  }
}

// Usage
const user = new User();
user.firstName = 'Timber';
user.lastName = 'Saw';
user.isActive = true;
await user.save(); // Entity saves itself

const timber = await User.findByName('Timber', 'Saw'); // Static query method
```

#### Data Mapper (TypeORM Repository)

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;
}

// Usage — entity is "dumb"
const userRepository = dataSource.getRepository(User);

const user = new User();
user.firstName = 'Timber';
user.lastName = 'Saw';
user.isActive = true;
await userRepository.save(user); // Repository saves the entity

const timber = await userRepository.findOneBy({
  firstName: 'Timber',
  lastName: 'Saw',
});
```

#### Comparison Table

| Aspect | Active Record | Data Mapper |
|--------|---------------|-------------|
| **Persistence logic location** | Inside the entity | In separate repository classes |
| **Entity awareness** | Knows about the database | Completely unaware of the database |
| **Simplicity** | High — everything in one place | Lower — requires separate classes |
| **Maintainability** | Decreases as the app grows | Remains high as the app grows |
| **SRP compliance** | Violates SRP (entity has two responsibilities) | Respects SRP (entity = data, repository = persistence) |
| **Testability** | Harder to test without a database | Easier to test with repository fakes |
| **Best for** | Small apps, prototypes, CRUD-heavy apps | Large apps, complex domains, DDD |
| **Popular implementations** | Eloquent (Laravel), TypeORM (AR mode), Rails ActiveRecord | Doctrine (Symfony), Hibernate (Java), TypeORM (DM mode) |

**Constraints and Limitations:**
- TypeORM allows **both** patterns, but you must choose one per entity (or per project) for consistency.
- Active Record entities must extend `BaseEntity` to gain persistence methods.
- Data Mapper entities are plain classes with no persistence inheritance.
- Active Record violates the Single Responsibility Principle — a criticism that grows more significant as the domain model becomes richer.
- Empirical studies show Active Record (Eloquent) has lower memory consumption, while Data Mapper (Doctrine) is superior in execution duration for most operations.

### Multiple Annotated Code Examples

#### Example 1: Active Record in TypeORM

```typescript
// active-record/User.ts
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;

  static findActive() {
    return this.findBy({ isActive: true });
  }
}
```

```typescript
// active-record/main.ts
import { AppDataSource } from './data-source';
import { User } from './active-record/User';

await AppDataSource.initialize();

// Create and save
const user = new User();
user.firstName = 'Alice';
user.lastName = 'Smith';
user.isActive = true;
await user.save(); // Entity knows how to save itself

// Query
const activeUsers = await User.findActive();
console.log(activeUsers);

// Remove
await user.remove(); // Entity knows how to remove itself
```

**Expected Output:**
```
[ User { id: 1, firstName: 'Alice', lastName: 'Smith', isActive: true } ]
```

**Why this output:** The `User` entity extends `BaseEntity`, giving it `.save()` and `.remove()` methods. The static `findActive()` method uses the inherited `findBy()` method. All persistence logic lives inside the model.

#### Example 2: Data Mapper in TypeORM

```typescript
// data-mapper/User.ts — "dumb" entity
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;
}
```

```typescript
// data-mapper/UserRepository.ts
import { DataSource, Repository } from 'typeorm';
import { User } from './User';

export class UserRepository {
  private repository: Repository<User>;

  constructor(dataSource: DataSource) {
    this.repository = dataSource.getRepository(User);
  }

  async findActive(): Promise<User[]> {
    return this.repository.findBy({ isActive: true });
  }

  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }

  async remove(user: User): Promise<void> {
    await this.repository.remove(user);
  }
}
```

```typescript
// data-mapper/main.ts
import { AppDataSource } from './data-source';
import { User } from './data-mapper/User';
import { UserRepository } from './data-mapper/UserRepository';

await AppDataSource.initialize();

const userRepo = new UserRepository(AppDataSource);

const user = new User();
user.firstName = 'Alice';
user.lastName = 'Smith';
user.isActive = true;
await userRepo.save(user); // Repository saves the entity

const activeUsers = await userRepo.findActive();
console.log(activeUsers);

await userRepo.remove(user); // Repository removes the entity
```

**Expected Output:**
```
[ User { id: 1, firstName: 'Alice', lastName: 'Smith', isActive: true } ]
```

**Why this output:** The `User` entity has no persistence methods. All persistence logic is in `UserRepository`. The entity is a plain data structure, and the repository handles all database interactions.

#### Example 3: Eloquent (Active Record) vs. Doctrine (Data Mapper) — Conceptual Comparison

**Eloquent (Active Record) — Laravel:**
```php
// Creating a user
$user = new User();
$user->name = 'Alice';
$user->email = 'alice@example.com';
$user->save(); // The model saves itself

// Querying
$activeUsers = User::where('active', true)->get();
```

**Doctrine (Data Mapper) — Symfony:**
```php
// Creating a user
$user = new User();
$user->setName('Alice');
$user->setEmail('alice@example.com');
$entityManager->persist($user);
$entityManager->flush(); // The EntityManager writes changes

// Querying
$activeUsers = $entityManager
    ->getRepository(User::class)
    ->findBy(['active' => true]);
```

**Why this matters:** In Eloquent, the `User` model contains all the query and persistence logic. In Doctrine, the `User` entity is a plain PHP class, and the `EntityManager` and repositories handle persistence. The Data Mapper approach keeps the domain model clean and testable in isolation.

### Real-World Cases

- **Small CRUD applications:** Active Record (Eloquent, TypeORM AR mode) is ideal — fewer files, faster development.
- **Large domain models (DDD):** Data Mapper (Doctrine, Hibernate) is preferred — domain objects remain free of persistence concerns.
- **Team scaling:** Data Mapper's explicit separation makes it easier for multiple developers to work on domain logic and persistence independently.
- **Legacy codebases:** Active Record is often easier to introduce incrementally; Data Mapper requires more upfront architectural investment.

---

## Core Concept 3: Query Scopes & Criteria Objects

### Definitions

**Core Definition:** Query Scopes and Criteria Objects encapsulate reusable query filters, sorting, and pagination into isolated, composable, and testable objects, preventing repository method explosion and duplicated query logic.

**Technical Definition:** The Criteria pattern enables "the encapsulation of query logic into objects, abstracting the complexities of database operations. These objects, known as 'criteria', are passed seamlessly through application layers, serving as a blueprint for fetching data according to various conditions". The Specification pattern, a related concept, helps "split [queries] into explicit and reusable filters, improving usability and testability of your database queries". Each specification defines a set of criteria that will be automatically applied to the query builder. Specifications can be freely combined to build complex queries while remaining easily testable and maintainable separately.

**Beginner-Friendly Explanation:** Imagine you're a librarian who keeps getting asked for books in slightly different ways: "books by author X," "books published after 2020," "books by author X published after 2020," "books by author X published after 2020 sorted by title." Instead of writing a new query method for each combination, you create small, reusable filter objects — "byAuthor(X)," "publishedAfter(2020)," "sortedByTitle()" — and combine them. Each filter is a Criteria or Specification. You can test each one independently and compose them freely.

### Purposes

- To eliminate repository method explosion caused by one method per query variant.
- To encapsulate query filters, ordering, and pagination into reusable, composable objects.
- To improve testability by allowing each specification to be tested in isolation.
- To prevent duplication of similar filter combinations across the application.
- To decouple the application layer from the specifics of how queries are constructed.

### Syntax Rules and Structure

#### Criteria Object Pattern (Clean Architecture)

```typescript
// domain/Criteria.ts
export interface Filter<T> {
  field: keyof T;
  operator: '=' | '!=' | '>' | '<' | '>=' | '<=' | 'LIKE' | 'IN';
  value: unknown;
}

export interface Order<T> {
  field: keyof T;
  direction: 'ASC' | 'DESC';
}

export class Criteria<T> {
  constructor(
    public readonly filters: Filter<T>[] = [],
    public readonly orders: Order<T>[] = [],
    public readonly limit?: number,
    public readonly offset?: number,
  ) {}

  static none<T>(): Criteria<T> {
    return new Criteria<T>();
  }

  where(field: keyof T, operator: Filter<T>['operator'], value: unknown): Criteria<T> {
    return new Criteria(
      [...this.filters, { field, operator, value }],
      this.orders,
      this.limit,
      this.offset,
    );
  }

  orderBy(field: keyof T, direction: 'ASC' | 'DESC' = 'ASC'): Criteria<T> {
    return new Criteria(
      this.filters,
      [...this.orders, { field, direction }],
      this.limit,
      this.offset,
    );
  }

  take(limit: number): Criteria<T> {
    return new Criteria(this.filters, this.orders, limit, this.offset);
  }

  skip(offset: number): Criteria<T> {
    return new Criteria(this.filters, this.orders, this.limit, offset);
  }
}
```

#### Specification Pattern (Doctrine-Style)

```typescript
// domain/Specification.ts
export abstract class Specification<T> {
  abstract modifyBuilder(builder: QueryBuilder<T>): void;

  and(other: Specification<T>): Specification<T> {
    return new AndSpecification(this, other);
  }

  or(other: Specification<T>): Specification<T> {
    return new OrSpecification(this, other);
  }
}

export class PostedByUser extends Specification<Article> {
  constructor(private userId: string) { super(); }

  modifyBuilder(builder: QueryBuilder<Article>): void {
    builder.andWhere('article.userId = :userId', { userId: this.userId });
  }
}

export class Published extends Specification<Article> {
  modifyBuilder(builder: QueryBuilder<Article>): void {
    builder.andWhere('article.published = true');
  }
}

export class OrderedByDateDesc extends Specification<Article> {
  modifyBuilder(builder: QueryBuilder<Article>): void {
    builder.orderBy('article.createdAt', 'DESC');
  }
}
```

#### Syntax Rules

- Criteria objects should be **pure data structures** — they describe *what* to query, not *how*.
- The repository is responsible for translating a Criteria object into the database-specific query (SQL, MongoDB aggregation, etc.).
- Specifications should be **composable** — `specA.and(specB)` produces a new specification.
- Each specification should be **independently testable** — you can verify that a specification adds the correct filter without running a full query.
- The application layer constructs Criteria/Specifications; the infrastructure layer interprets them.

#### Constraints and Limitations

- Criteria objects add a layer of indirection; for very simple applications, direct repository methods may be simpler.
- Specifications require a translation layer (e.g., Doctrine's `DoctrineCriteria`, TypeORM's `QueryBuilder`) to be applied to the actual query.
- Overly generic Criteria objects can become a mini-language of their own, requiring documentation and tooling.
- Criteria objects should not contain database-specific logic; that belongs in the translator.

### Multiple Annotated Code Examples

#### Example 1: Criteria Pattern with TypeORM

```typescript
// domain/UserCriteria.ts
export class UserCriteria {
  public filters: Array<{ field: string; operator: string; value: unknown }> = [];
  public orders: Array<{ field: string; direction: 'ASC' | 'DESC' }> = [];
  public limit?: number;
  public offset?: number;

  static active(): UserCriteria {
    return new UserCriteria().where('status', '=', 'active');
  }

  where(field: string, operator: string, value: unknown): UserCriteria {
    this.filters.push({ field, operator, value });
    return this;
  }

  orderBy(field: string, direction: 'ASC' | 'DESC' = 'ASC'): UserCriteria {
    this.orders.push({ field, direction });
    return this;
  }

  take(limit: number): UserCriteria {
    this.limit = limit;
    return this;
  }
}
```

```typescript
// infrastructure/TypeOrmUserRepository.ts
export class TypeOrmUserRepository implements IUserRepository {
  constructor(private dataSource: DataSource) {}

  async findByCriteria(criteria: UserCriteria): Promise<User[]> {
    const qb = this.dataSource.getRepository(User).createQueryBuilder('user');

    for (const filter of criteria.filters) {
      qb.andWhere(`user.${filter.field} ${filter.operator} :${filter.field}`, {
        [filter.field]: filter.value,
      });
    }

    for (const order of criteria.orders) {
      qb.addOrderBy(`user.${order.field}`, order.direction);
    }

    if (criteria.limit) qb.take(criteria.limit);
    if (criteria.offset) qb.skip(criteria.offset);

    return qb.getMany();
  }
}
```

```typescript
// application/UserService.ts
export class UserService {
  constructor(private userRepository: IUserRepository) {}

  async listActiveUsers(): Promise<User[]> {
    const criteria = UserCriteria.active()
      .orderBy('createdAt', 'DESC')
      .take(50);
    return this.userRepository.findByCriteria(criteria);
  }

  async listActiveAdmins(): Promise<User[]> {
    const criteria = UserCriteria.active()
      .where('role', '=', 'admin')
      .orderBy('name', 'ASC');
    return this.userRepository.findByCriteria(criteria);
  }
}
```

**Expected behaviour:** `listActiveUsers()` produces `WHERE status = 'active' ORDER BY createdAt DESC LIMIT 50`. `listActiveAdmins()` produces `WHERE status = 'active' AND role = 'admin' ORDER BY name ASC`. The repository translates the Criteria object into a TypeORM query.

#### Example 2: Specification Pattern (Doctrine-Style)

```typescript
// domain/specifications/ArticleSpecifications.ts
export class ArticleSpecifications {
  static published(): Specification<Article> {
    return new class extends Specification<Article> {
      modifyBuilder(builder: QueryBuilder<Article>): void {
        builder.andWhere('article.published = true');
      }
    };
  }

  static postedByUser(userId: string): Specification<Article> {
    return new class extends Specification<Article> {
      modifyBuilder(builder: QueryBuilder<Article>): void {
        builder.andWhere('article.userId = :userId', { userId });
      }
    };
  }

  static inCategory(categoryId: string): Specification<Article> {
    return new class extends Specification<Article> {
      modifyBuilder(builder: QueryBuilder<Article>): void {
        builder.andWhere('article.categoryId = :categoryId', { categoryId });
      }
    };
  }

  static orderedByDateDesc(): Specification<Article> {
    return new class extends Specification<Article> {
      modifyBuilder(builder: QueryBuilder<Article>): void {
        builder.orderBy('article.createdAt', 'DESC');
      }
    };
  }
}

// Usage — composing specifications
const spec = ArticleSpecifications.published()
  .and(ArticleSpecifications.postedByUser(userId))
  .and(ArticleSpecifications.inCategory(categoryId))
  .and(ArticleSpecifications.orderedByDateDesc());

const articles = await articleRepository.findBySpecification(spec);
```

**Expected behaviour:** The composed specification produces `WHERE published = true AND userId = :userId AND categoryId = :categoryId ORDER BY createdAt DESC`. Each specification is independently testable and reusable.

### Real-World Cases

- **Search endpoints:** A single `GET /users` endpoint accepts filter, sort, and pagination parameters that are translated into Criteria objects.
- **Admin panels:** Different admin views (active users, pending orders, recent transactions) are built by composing specifications.
- **Reporting:** Complex report queries are assembled from reusable filter specifications.
- **CQRS read models:** Criteria objects define the "what" of a query, while the read model defines the "shape" of the result.

---

## Core Concept 4: Unit of Work & Multi-Model Transactions

### Definitions

**Core Definition:** The Unit of Work pattern maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems, ensuring that multiple repository operations either all succeed or all fail together.

**Technical Definition:** The Unit of Work pattern "maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems". A Unit of Work "keeps track of everything you do during a business transaction that can affect the database. When you're done, it figures out everything that needs to be done to alter the database as a result of your work". In practice, a Unit of Work holds a single database session or transaction context and shares it across multiple repositories, so that operations across different repositories participate in the same transaction.

**Beginner-Friendly Explanation:** Imagine you're transferring money between two bank accounts. You need to subtract from one account and add to the other. If the subtraction succeeds but the addition fails, the money vanishes. The Unit of Work pattern solves this by treating both operations as a single transaction: either both succeed, or neither does. It's like a "shopping cart" for database changes — you add all your changes to the cart, and at checkout (commit), they're all applied together, or if anything goes wrong, the cart is emptied (rollback).

### Purposes

- To ensure ACID compliance across operations spanning multiple repositories.
- To centralise transaction management in a single dependency rather than scattering transaction logic across services.
- To enable rollback of all changes when any operation in a multi-step business transaction fails.
- To avoid data inconsistencies caused by partial writes.
- To propagate the active transaction context across service layers without passing it through every function signature.

### Syntax Rules and Structure

#### TypeORM Unit of Work

```typescript
// infrastructure/UnitOfWork.ts
import { DataSource, EntityManager } from 'typeorm';

export class UnitOfWork {
  constructor(private dataSource: DataSource) {}

  async transaction<T>(work: (manager: EntityManager) => Promise<T>): Promise<T> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      const result = await work(queryRunner.manager);
      await queryRunner.commitTransaction();
      return result;
    } catch (err) {
      await queryRunner.rollbackTransaction();
      throw err;
    } finally {
      await queryRunner.release();
    }
  }
}
```

```typescript
// application/OrderService.ts
export class OrderService {
  constructor(
    private unitOfWork: UnitOfWork,
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository,
  ) {}

  async createOrder(userId: string, productId: string, quantity: number): Promise<Order> {
    return this.unitOfWork.transaction(async (manager) => {
      const product = await manager.findOne(Product, { where: { id: productId } });
      if (!product || product.stock < quantity) {
        throw new Error('Insufficient stock');
      }

      product.stock -= quantity;
      await manager.save(product);

      const order = new Order();
      order.userId = userId;
      order.productId = productId;
      order.quantity = quantity;
      await manager.save(order);

      return order;
    });
  }
}
```

#### Prisma Unit of Work

```typescript
// infrastructure/PrismaUnitOfWork.ts
import { PrismaClient } from '@prisma/client';

export class PrismaUnitOfWork {
  constructor(private prisma: PrismaClient) {}

  async transaction<T>(work: (tx: Prisma.TransactionClient) => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      return work(tx);
    });
  }
}
```

```typescript
// application/OrderService.ts
export class OrderService {
  constructor(
    private uow: PrismaUnitOfWork,
    private prisma: PrismaClient,
  ) {}

  async createOrder(userId: string, productId: string, quantity: number) {
    return this.uow.transaction(async (tx) => {
      const product = await tx.product.findUnique({ where: { id: productId } });
      if (!product || product.stock < quantity) throw new Error('Insufficient stock');

      await tx.product.update({
        where: { id: productId },
        data: { stock: product.stock - quantity },
      });

      return tx.order.create({
        data: { userId, productId, quantity },
      });
    });
  }
}
```

#### Syntax Rules

- The Unit of Work must obtain a **single transaction context** (TypeORM `QueryRunner`, Prisma `TransactionClient`) and pass it to all repositories.
- All repository operations within the transaction must use the transaction-scoped `EntityManager` or `tx` client — not the global repository.
- The `transaction()` method should handle `connect` → `startTransaction` → `commit` / `rollback` → `release` in a `try/catch/finally` block.
- Exceptions thrown inside the transaction callback trigger an automatic rollback.
- Nested transactions should be avoided or managed via savepoints.

#### Constraints and Limitations

- TypeORM repositories obtained via `dataSource.getRepository(Entity)` are **not** transaction-scoped; you must use `manager.getRepository(Entity)` inside the transaction.
- Prisma's interactive transactions have a default timeout (5 seconds); long-running transactions must configure `timeout` and `maxWait`.
- Cross-database transactions (e.g., PostgreSQL + MongoDB) require additional coordination; libraries like `@wire/uow` can orchestrate multiple transactional resources.
- The Unit of Work pattern adds complexity; for single-repository operations, a simple transaction may suffice.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: TypeORM Unit of Work with Multiple Repositories

```typescript
// infrastructure/UnitOfWork.ts
import { DataSource, EntityManager } from 'typeorm';

export class UnitOfWork {
  constructor(private dataSource: DataSource) {}

  async transaction<T>(work: (manager: EntityManager) => Promise<T>): Promise<T> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      const result = await work(queryRunner.manager);
      await queryRunner.commitTransaction();
      return result;
    } catch (err) {
      await queryRunner.rollbackTransaction();
      throw err;
    } finally {
      await queryRunner.release();
    }
  }
}
```

```typescript
// application/PurchaseService.ts
export class PurchaseService {
  constructor(
    private uow: UnitOfWork,
    private productRepository: ProductRepository,
    private purchaseRepository: PurchaseRepository,
  ) {}

  async createPurchase(dto: CreatePurchaseDto): Promise<Purchase> {
    return this.uow.transaction(async (manager) => {
      // Both repositories use the SAME transaction-scoped manager
      const product = await manager.findOne(Product, { where: { id: dto.productId } });
      if (!product || product.stock < dto.quantity) {
        throw new Error('Insufficient stock');
      }

      product.stock -= dto.quantity;
      await manager.save(product); // Part of the transaction

      const purchase = new Purchase();
      purchase.productId = product.id;
      purchase.quantity = dto.quantity;
      purchase.date = new Date();
      await manager.save(purchase); // Part of the same transaction

      return purchase;
    });
  }
}
```

**Step-by-step setup:**
1. Create `UnitOfWork` with a `DataSource`.
2. The `transaction()` method creates a `QueryRunner`, starts a transaction, and passes the `EntityManager` to the callback.
3. All repository operations inside the callback use `manager` (the transaction-scoped manager), not the global repository.
4. If any operation fails, the `catch` block rolls back the transaction.
5. The `finally` block releases the `QueryRunner`.

**Expected behaviour:** If `purchaseRepository.save()` fails, the `product.stock` update is also rolled back — no stock inconsistency. The database remains consistent.

**Why this works:** The `EntityManager` from `queryRunner.manager` is bound to the transaction. Every operation using this manager participates in the same transaction. Committing or rolling back affects all operations atomically.

#### Example 2: Prisma Unit of Work with Interactive Transactions

```typescript
// infrastructure/PrismaUnitOfWork.ts
import { PrismaClient, Prisma } from '@prisma/client';

export class PrismaUnitOfWork {
  constructor(private prisma: PrismaClient) {}

  async transaction<T>(
    work: (tx: Prisma.TransactionClient) => Promise<T>,
    options?: { timeout?: number; maxWait?: number },
  ): Promise<T> {
    return this.prisma.$transaction(
      async (tx) => work(tx),
      {
        timeout: options?.timeout ?? 10000,
        maxWait: options?.maxWait ?? 5000,
      },
    );
  }
}
```

```typescript
// application/OrderService.ts
export class OrderService {
  constructor(
    private uow: PrismaUnitOfWork,
    private prisma: PrismaClient,
  ) {}

  async createOrder(userId: string, productId: string, quantity: number): Promise<Order> {
    return this.uow.transaction(async (tx) => {
      // Check stock within the transaction
      const product = await tx.product.findUnique({
        where: { id: productId },
      });

      if (!product || product.stock < quantity) {
        throw new Error('Insufficient stock');
      }

      // Decrement stock — part of the transaction
      await tx.product.update({
        where: { id: productId },
        data: { stock: product.stock - quantity },
      });

      // Create order — part of the same transaction
      const order = await tx.order.create({
        data: {
          userId,
          productId,
          quantity,
          status: 'pending',
        },
      });

      return order;
    });
  }
}
```

**Expected Output:**
```
Order created: { id: 1, userId: 'user-1', productId: 'prod-1', quantity: 2, status: 'pending' }
```

**Why this output:** The `$transaction()` method passes a `tx` client to the callback. All operations using `tx` are part of the same transaction. If `tx.order.create()` fails, the `tx.product.update()` is rolled back automatically. The transaction commits only if all operations succeed.

#### Example 3: Cross-Database Unit of Work with `@wire/uow`

```typescript
import { UnitOfWorkScope } from 'jsr:@wire/uow';

const scope = new UnitOfWorkScope();

await scope.cast(async (uow) => {
  // Register PostgreSQL participant
  uow.set('postgres', {
    session: { txId: 'pg-1' },
    cycle: {
      commit: async () => console.log('PostgreSQL commit'),
      rollback: async () => console.log('PostgreSQL rollback'),
    },
  });

  // Register MongoDB participant
  uow.set('mongo', {
    session: { sessionId: 'mongo-1' },
    cycle: {
      commit: async () => console.log('MongoDB commit'),
      rollback: async () => console.log('MongoDB rollback'),
    },
  });

  // Domain logic — both sessions are available
  const pgSession = uow.get('postgres').session;
  const mongoSession = uow.get('mongo').session;

  // If any error is thrown, all participants roll back
});
```

**Expected Output:**
```
PostgreSQL commit
MongoDB commit
```

**Why this output:** `@wire/uow` coordinates multiple transactional resources (PostgreSQL and MongoDB) as a single logical unit. The `cast()` method provides automatic commit/rollback lifecycle. If the domain logic throws, both participants roll back.

### Real-World Cases

- **E-commerce checkout:** Creating an order, decrementing inventory, and recording a payment must all succeed or fail together.
- **Banking transfers:** Debiting one account and crediting another must be atomic.
- **User registration:** Creating a user record and sending a welcome email (via an outbox table) must be consistent.
- **Multi-tenant SaaS:** Operations across tenant-specific repositories must be transactionally consistent.
- **Event sourcing:** Writing to an event store and updating a read model must be atomic.

---

## References

- Martin Fowler — Repository Pattern (PoEAA) — https://martinfowler.com/eaaCatalog/repository.html
- Martin Fowler — Data Mapper Pattern (PoEAA) — https://martinfowler.com/eaaCatalog/dataMapper.html
- Martin Fowler — Unit of Work Pattern (PoEAA) — https://martinfowler.com/eaaCatalog/unitOfWork.html
- TypeORM Documentation — Active Record vs Data Mapper — https://typeorm.io/docs/guides/active-record-data-mapper/
- Microsoft Learn — Repository Pattern: The Purpose — https://learn.microsoft.com/en-us/archive/blogs/nilotpal/repository-pattern-the-purpose
- Microsoft Learn — The Unit Of Work Pattern And Persistence Ignorance — https://learn.microsoft.com/en-us/archive/msdn-magazine/2015/august/the-unit-of-work-pattern-and-persistence-ignorance
- Microsoft Learn — Data Mapper Application Block — https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff650156(v=pandp.10)
- OneUptime Blog — How to Implement the Repository Pattern with MongoDB — https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mongodb-repository-pattern/README.md
- tsecurity.de — Building a Unit of Work Like Pattern in NestJS and Sequelize — https://tsecurity.de/de/2345935/IT+Programmierung/Building+a+Unit+of+Work+like+pattern+in+NestJS+and+Sequelize
- JSR — @wire/uow: Unit of Work Scope — https://jsr.io/@wire/uow
- Packagist — mediagone/doctrine-specifications (Specification Pattern) — https://packagist.org/packages/mediagone/doctrine-specifications
- Stackademic — Implementing the Criteria Design Pattern with Clean Architecture in TypeScript — https://blog.stackademic.com/implementing-the-criteria-design-pattern-with-clean-architecture-in-typescript-031bb2a80ced
- GitHub — zhuravlevma/typeorm-unit-of-work: Clean Architecture for NestJS — https://github.com/zhuravlevma/typeorm-unit-of-work
- GitHub — LuanMaik/nestjs-unit-of-work: NestJS + TypeORM Unit of Work — https://github.com/LuanMaik/nestjs-unit-of-work
- Wikipedia — Specification Pattern — https://en.wikipedia.org/wiki/Specification_pattern
- Wikipedia — Active Record Pattern — https://en.wikipedia.org/wiki/Active_record_pattern
- Wikipedia — Data Mapper Pattern — https://en.wikipedia.org/wiki/Data_mapper_pattern
- Stack Overflow — TypeORM Transactions for Different Repositories — https://stackoverflow.com/questions/70140732/typeorm-transactions-for-different-repositories