# Application & Domain Service Layers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The application and domain service layers are the architectural tiers responsible for orchestrating use cases, encapsulating business rules, and coordinating domain objects and infrastructure — with the application layer acting as a thin facade for external consumers and the domain layer containing pure, framework-agnostic business logic.

**Technical Definition:** The Service Layer defines an application's boundary with a set of coarse-grained application services that orchestrate domain logic, coordinate transactions, and enforce security in one authoritative place. Application Services implement the use cases of the application (user interactions in a typical web application), while Domain Services implement the core, use-case-independent domain logic. Application Services get and return Data Transfer Objects (DTOs), while Domain Service methods typically get and return domain objects (entities, value objects). Domain Services are typically used by Application Services or other Domain Services, while Application Services are used by the Presentation Layer or client applications.

**Beginner-Friendly Explanation:** Imagine a restaurant. The **Application Service** is the waiter — they take your order (a use case), coordinate with the kitchen, handle the payment, and bring you your food. They don't actually cook anything; they just make sure everything happens in the right order. The **Domain Service** is the head chef — they know the recipes (business rules) and decide how to combine ingredients (domain objects) to create a dish. The **domain entities** are the ingredients themselves — the chef manipulates them according to the rules. The **DTO** is the order slip — a structured piece of paper that carries your request from the waiter to the kitchen. Keeping these roles separate means you can change the menu (business rules) without retraining the waiter, and change the waitstaff (API) without changing the recipes.

### Key Characteristics

- **Application layer orchestration:** Application services are thin orchestrators that sequence work, manage transactions, and handle cross-cutting concerns like authorization and logging.
- **Domain layer purity:** Domain services and entities contain only business rules and are completely unaware of infrastructure, databases, or frameworks.
- **DTO boundaries:** Application services accept and return DTOs, never exposing domain entities to external consumers.
- **Transaction ownership:** The application service owns the transaction boundary (Unit of Work), committing or rolling back all operations atomically.
- **Stateless design:** Both application and domain services are stateless — they receive all necessary data as method parameters.
- **Rich vs. anemic models:** A rich domain model embeds behaviour in entities; an anemic model separates data from behaviour into service classes.

### Prerequisites

- **Understanding of layered architecture:** Familiarity with presentation, application, domain, and infrastructure layers.
- **Domain-Driven Design (DDD) concepts:** Entities, value objects, aggregates, and repositories.
- **Dependency injection:** Constructor injection and inversion of control (common in NestJS, Spring, and .NET).
- **Repository pattern:** How persistence is abstracted behind interfaces.
- **DTO fundamentals:** Understanding of immutable data transfer objects and validation.
- **Transaction management:** ACID properties and Unit of Work pattern.

### Related Programming Areas

- **Domain-Driven Design (DDD):** Application services, domain services, and entities are core tactical DDD patterns.
- **Clean Architecture / Hexagonal Architecture:** Application services are the "use case" layer; domain services are the "domain" layer.
- **CQRS (Command Query Responsibility Segregation):** Application services often implement command handlers.
- **Repository Pattern:** Application services coordinate repositories; domain services may use them for cross-aggregate logic.
- **Unit of Work:** Application services own the transaction boundary, ensuring ACID compliance across repositories.
- **Validation and DTOs:** Cross-layer data contracts ensure type safety from controllers to services.

### Core Concepts

1. **Business Logic Encapsulation** — restricting services to orchestrating business rules, managing domain invariants, and interacting with the persistence layer.
2. **Anemic vs. Rich Domain Models** — designing thin services with logic embedded in entities versus heavy services with dumb data structures.
3. **Application Services vs. Domain Services** — distinguishing between framework-specific orchestration and pure, database-agnostic business rules.
4. **Cross-Layer Data Contracts (DTOs)** — utilizing immutable Data Transfer Objects to strictly type-bind data moving from controllers down into services.

---

## Core Concept 1: Business Logic Encapsulation

### Definitions

**Core Definition:** Business logic encapsulation is the practice of restricting application services to orchestrating business rules, managing domain invariants, and interacting with the persistence layer — without embedding actual business rules inside the service itself.

**Technical Definition:** The Service Layer gives each use case a single method — `registerUser()`, `transferFunds()`, `publishArticle()` — that scripts the work: validate input, load domain objects, invoke their behaviour, commit the transaction, dispatch side effects. The service is deliberately thin. It does not contain business rules so much as sequence them, delegating the actual decisions to domain entities and delegating persistence to repositories. Application services may contain transaction management (Unit of Work), application validations (validate state of objects retrieved from database / input saved to database), security validations, and cross-cutting concerns such as logging and caching.

**Beginner-Friendly Explanation:** Think of an application service as a film director. The director doesn't act, write the script, or operate the camera — they coordinate all these specialists to produce a scene. Similarly, an application service doesn't contain business rules; it loads the right domain objects, tells them to do their job, saves the results, and handles all the logistical concerns (transactions, logging, security). The business rules live in the domain objects and domain services, where they belong.

### Purposes

- To centralise use-case orchestration in a single, authoritative place per operation.
- To prevent business rules from leaking into controllers, HTTP handlers, or CLI entry points.
- To ensure that transaction boundaries, authorization checks, and cross-cutting concerns are handled consistently.
- To keep the domain model free of infrastructure concerns such as databases, HTTP, and logging.
- To enable multiple delivery mechanisms (HTTP, CLI, queue workers) to reuse the same use-case logic.

### Syntax Rules and Structure

#### General Syntax (TypeScript)

```typescript
// Application Service — one method per use case
export class TransferMoneyService {
  constructor(
    private readonly unitOfWork: UnitOfWork,
    private readonly accountRepository: AccountRepository,
    private readonly authorizationService: AuthorizationService,
    private readonly logger: Logger,
  ) {}

  async transfer(command: TransferMoneyCommand): Promise<void> {
    // 1. Application validation
    if (command.amountCents <= 0) {
      throw new InvalidArgumentException('Transfer amount must be positive.');
    }

    // 2. Authorization
    await this.authorizationService.assertCanTransfer(
      command.actorId,
      command.fromAccountId,
    );

    // 3. Transaction boundary
    await this.unitOfWork.transaction(async (manager) => {
      // 4. Load domain objects
      const from = await this.accountRepository.findById(command.fromAccountId, manager);
      const to = await this.accountRepository.findById(command.toAccountId, manager);

      if (!from || !to) throw new AccountNotFoundException();

      // 5. Delegate to domain logic (business rules live here)
      from.withdraw(command.amountCents);
      to.deposit(command.amountCents);

      // 6. Persist
      await this.accountRepository.save(from, manager);
      await this.accountRepository.save(to, manager);
    });

    // 7. Side effects (after commit)
    this.logger.info(`Transfer completed: ${command.fromAccountId} -> ${command.toAccountId}`);
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `TransferMoneyService` | Application service — one per use case. |
| `TransferMoneyCommand` | Immutable DTO carrying validated input. |
| `unitOfWork.transaction()` | Owns the transaction boundary. |
| `authorizationService.assertCanTransfer()` | Cross-cutting security concern. |
| `from.withdraw()` / `to.deposit()` | Domain logic delegated to entities. |
| `accountRepository.save()` | Persistence delegated to repository. |

#### Syntax Rules

- Each application service method should correspond to exactly one use case.
- The service must accept a DTO or Command object as its input, never raw parameters scattered across the method signature.
- Transaction boundaries must be explicit and owned by the application service.
- Business rules must be delegated to domain entities or domain services — the application service only sequences them.
- Cross-cutting concerns (authorization, logging, caching) belong in the application service or in decorators/middleware.

#### Constraints and Limitations

- Application services must not contain business rules that domain experts would want to change — those belong in the domain layer.
- Application services must not leak domain entities to external consumers; always map to DTOs.
- Overly thin services that only forward calls to repositories (without any orchestration) may indicate an anemic domain model.
- Application services are stateless; they must not hold mutable state between calls.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Application Service with Domain Delegation (TypeScript)

```typescript
// domain/Account.ts — Rich domain entity with business rules
export class Account {
  constructor(
    public readonly id: string,
    private balance: number,
  ) {}

  withdraw(amount: number): void {
    if (amount > this.balance) {
      throw new InsufficientFundsError(
        `Cannot withdraw ${amount}; balance is ${this.balance}`,
      );
    }
    this.balance -= amount;
  }

  deposit(amount: number): void {
    if (amount <= 0) {
      throw new InvalidAmountError('Deposit amount must be positive');
    }
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

```typescript
// application/dto/TransferMoneyCommand.ts
export class TransferMoneyCommand {
  constructor(
    public readonly actorId: string,
    public readonly fromAccountId: string,
    public readonly toAccountId: string,
    public readonly amountCents: number,
  ) {}
}
```

```typescript
// application/TransferMoneyService.ts
import { UnitOfWork } from '../infrastructure/UnitOfWork';
import { AccountRepository } from '../domain/AccountRepository';
import { AuthorizationService } from '../infrastructure/AuthorizationService';
import { TransferMoneyCommand } from './dto/TransferMoneyCommand';

export class TransferMoneyService {
  constructor(
    private readonly unitOfWork: UnitOfWork,
    private readonly accountRepository: AccountRepository,
    private readonly auth: AuthorizationService,
    private readonly logger: Logger,
  ) {}

  async transfer(command: TransferMoneyCommand): Promise<void> {
    // Application-level validation
    if (command.amountCents <= 0) {
      throw new Error('Transfer amount must be positive.');
    }

    // Authorization (cross-cutting concern)
    await this.auth.assertCanTransfer(command.actorId, command.fromAccountId);

    // Transaction boundary
    await this.unitOfWork.transaction(async (manager) => {
      const from = await this.accountRepository.findById(command.fromAccountId, manager);
      const to = await this.accountRepository.findById(command.toAccountId, manager);

      if (!from || !to) throw new Error('Account not found');

      // Business rules live in the domain entity
      from.withdraw(command.amountCents);
      to.deposit(command.amountCents);

      await this.accountRepository.save(from, manager);
      await this.accountRepository.save(to, manager);
    });

    this.logger.info(`Transfer: ${command.fromAccountId} -> ${command.toAccountId}`);
  }
}
```

**Step-by-step setup:**
1. Define a rich `Account` entity with `withdraw()` and `deposit()` methods containing business rules.
2. Create an immutable `TransferMoneyCommand` DTO.
3. Implement `TransferMoneyService` with constructor injection of `UnitOfWork`, `AccountRepository`, `AuthorizationService`, and `Logger`.
4. The service validates input, authorizes the actor, opens a transaction, loads entities, delegates to domain methods, persists, and logs after commit.

**Expected behaviour:** Calling `transfer()` with a valid command withdraws from one account and deposits into another within a single transaction. If the source account has insufficient funds, the domain entity throws, the transaction rolls back, and no changes are persisted.

**Why this works:** The application service orchestrates but does not decide. The business rule ("cannot withdraw more than the balance") lives in `Account.withdraw()`. The application service handles the logistics: transaction, authorization, logging, and repository coordination.

#### Example 2: NestJS Application Service with DTO and Domain Service

```typescript
// domain/book/BookManager.ts — Domain Service (pure business logic)
@Injectable()
export class BookManager {
  async createAsync(title: string, author: Author, price: number): Promise<Book> {
    if (price <= 0) throw new Error('Price must be positive');
    if (!author.isActive) throw new Error('Author is not active');
    return new Book(title, author, price);
  }
}
```

```typescript
// application/BookAppService.ts — Application Service (orchestration)
@Injectable()
export class BookAppService {
  constructor(
    private readonly bookRepository: BookRepository,
    private readonly authorRepository: AuthorRepository,
    private readonly bookManager: BookManager,
    private readonly objectMapper: ObjectMapper,
  ) {}

  @Authorize(BookPermissions.Create)
  async createBookAsync(input: CreateBookDto): Promise<BookDto> {
    // Load related domain objects
    const author = await this.authorRepository.getAsync(input.authorId);

    // Delegate business logic to domain service
    const book = await this.bookManager.createAsync(input.title, author, input.price);

    // Persist
    await this.bookRepository.insertAsync(book);

    // Return DTO (never the entity)
    return this.objectMapper.map<Book, BookDto>(book);
  }
}
```

**Expected behaviour:** `createBookAsync()` accepts a `CreateBookDto`, loads the `Author` entity, delegates to `BookManager` (domain service) for business rules, persists the `Book` entity, and returns a `BookDto`. The application service never contains the rule "price must be positive" — that lives in the domain service.

### Real-World Cases

- **Banking transfers:** The application service orchestrates the transaction; the domain entity enforces the balance invariant.
- **E-commerce checkout:** The application service coordinates inventory, payment, and order creation; domain entities enforce stock and pricing rules.
- **User registration:** The application service validates input, checks authorization, and persists; a domain service enforces email uniqueness.
- **Multi-client APIs:** The same application service is called by HTTP, CLI, and queue workers, ensuring consistent use-case logic.

---

## Core Concept 2: Anemic vs. Rich Domain Models

### Definitions

**Core Definition:** An anemic domain model contains domain objects with only data (attributes) and no business logic, with behaviour transferred to separate service classes; a rich domain model encapsulates both data and behaviour inside domain objects.

**Technical Definition:** According to Fowler, the anemic domain model is where domain objects (usually referred to as entities or model objects) contain only data (attributes) and no relevant business logic or behaviour. In a rich model, data and behaviour are encapsulated inside domain objects; in an anemic model, only data is encapsulated in the objects of the domain model, and the behaviour is transferred to a separate layer of services. Fowler and Evans consider the anemic model to be an antipattern because it breaks object-oriented encapsulation — the logic is not where the data is.

**Beginner-Friendly Explanation:** Imagine a car. In a rich domain model, the car knows how to start itself, accelerate, brake, and honk — all the behaviour is inside the car object. In an anemic domain model, the car is just a shell with wheels and an engine, and a separate "CarService" class knows how to start it, accelerate it, and brake it. The anemic approach may seem simpler at first, but it leads to "procedural programming disguised as object orientation" — you have classes, but all the logic is outside them.

### Purposes

- To understand the architectural trade-offs between embedding logic in entities versus services.
- To identify and refactor anemic domains into rich domains for better encapsulation and maintainability.
- To make informed decisions about where business rules should live based on domain complexity.
- To align persistence architecture with domain-driven design principles.
- To prevent the duplication and scattering of business logic across service classes.

### Syntax Rules and Structure

#### Anemic Domain Model

```typescript
// Anemic entity — only data, no behaviour
export class Subscription {
  id: string;
  status: 'active' | 'cancelled' | 'expired';
  expiresAt: Date;

  // No behaviour — just getters and setters
}

// Anemic service — all behaviour lives here
export class SubscriptionService {
  cancel(subscription: Subscription): void {
    if (subscription.status !== 'active') {
      throw new Error('Only active subscriptions can be cancelled');
    }
    subscription.status = 'cancelled';
  }

  isExpired(subscription: Subscription): boolean {
    return subscription.expiresAt < new Date();
  }
}
```

#### Rich Domain Model

```typescript
// Rich entity — data and behaviour together
export class Subscription {
  private constructor(
    public readonly id: string,
    private status: SubscriptionStatus,
    private expiresAt: Date,
  ) {}

  cancel(): void {
    if (this.status !== 'active') {
      throw new Error('Only active subscriptions can be cancelled');
    }
    this.status = 'cancelled';
  }

  isExpired(): boolean {
    return this.expiresAt < new Date();
  }

  renew(additionalDays: number): void {
    if (this.status === 'cancelled') {
      throw new Error('Cannot renew a cancelled subscription');
    }
    this.expiresAt = new Date(this.expiresAt.getTime() + additionalDays * 86400000);
    this.status = 'active';
  }
}
```

#### Comparison Table

| Aspect | Anemic Domain Model | Rich Domain Model |
|--------|---------------------|-------------------|
| **Data location** | In entities | In entities |
| **Behaviour location** | In service classes | In entities |
| **Encapsulation** | Broken — data exposed via getters/setters | Strong — internal state protected |
| **Business rule consistency** | Scattered across services | Centralised in entities |
| **Testability of rules** | Requires service instantiation | Can test entity in isolation |
| **Expressiveness** | Lost — procedural style | High — ubiquitous language |
| **Fowler's verdict** | Antipattern | Recommended for complex domains |
| **Best for** | Simple CRUD, prototypes | Complex domains, DDD |

**Constraints and Limitations:**
- Anemic models are not always harmful — for simple CRUD applications, they may be pragmatic.
- Refactoring from anemic to rich requires careful migration, as services may be shared across use cases.
- Rich models require discipline: entities must not become bloated with responsibilities that belong in domain services (cross-aggregate logic).
- Rich models can be harder to persist with some ORMs that expect public setters.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Anemic Model — The Problem

```typescript
// Anemic entity
export class Order {
  id: string;
  items: OrderItem[];
  status: string;
  total: number;
}

// Anemic service — logic scattered
export class OrderService {
  calculateTotal(order: Order): number {
    return order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  cancel(order: Order): void {
    if (order.status === 'shipped') {
      throw new Error('Cannot cancel shipped orders');
    }
    order.status = 'cancelled';
  }

  addItem(order: Order, item: OrderItem): void {
    if (order.status !== 'draft') {
      throw new Error('Cannot modify a non-draft order');
    }
    order.items.push(item);
  }
}
```

**Expected behaviour:** Every operation requires the service. The `Order` entity has no protection — any code can set `order.status = 'shipped'` directly, bypassing the rule.

**Why this is problematic:** The business rule "cannot cancel shipped orders" lives in `OrderService`, not in `Order`. Another service could accidentally bypass it. The entity is just a data bag.

#### Example 2: Rich Model — The Solution

```typescript
// Rich entity — behaviour encapsulated
export class Order {
  private items: OrderItem[] = [];
  private status: OrderStatus = 'draft';

  constructor(public readonly id: string) {}

  addItem(item: OrderItem): void {
    if (this.status !== 'draft') {
      throw new Error('Cannot modify a non-draft order');
    }
    this.items.push(item);
  }

  cancel(): void {
    if (this.status === 'shipped') {
      throw new Error('Cannot cancel shipped orders');
    }
    this.status = 'cancelled';
  }

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  getStatus(): OrderStatus {
    return this.status;
  }
}
```

```typescript
// Thin application service — just orchestrates
export class CancelOrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly unitOfWork: UnitOfWork,
  ) {}

  async cancelOrder(orderId: string): Promise<void> {
    await this.unitOfWork.transaction(async (manager) => {
      const order = await this.orderRepository.findById(orderId, manager);
      if (!order) throw new Error('Order not found');

      order.cancel(); // Business rule lives in the entity
      await this.orderRepository.save(order, manager);
    });
  }
}
```

**Expected behaviour:** The `Order` entity enforces its own invariants. The application service simply loads the order, calls `cancel()`, and saves it. No business rule is duplicated in the service.

**Why this works:** The rule "cannot cancel shipped orders" is in `Order.cancel()`. Any code path that cancels an order — whether from an API, a batch job, or a test — goes through the same rule. The entity is the single source of truth for its own behaviour.

### Real-World Cases

- **Complex domains (DDD):** Rich models are preferred for domains with intricate business rules (banking, insurance, logistics).
- **Simple CRUD applications:** Anemic models are pragmatic — a "User" with just name and email doesn't need behaviour.
- **Legacy migration:** Refactoring anemic services into rich entities is a common modernization effort.
- **Team expertise:** Rich models require domain expertise; teams without it may struggle and default to anemic models.

---

## Core Concept 3: Application Services vs. Domain Services

### Definitions

**Core Definition:** Application services orchestrate use cases and coordinate infrastructure, while domain services encapsulate core business logic that doesn't naturally fit within a single entity or value object.

**Technical Definition:** Application Services implement the use cases of the application (user interactions in a typical web application), while Domain Services implement the core, use-case-independent domain logic. Application Services get and return Data Transfer Objects (DTOs); Domain Service methods typically get and return the domain objects (entities, value objects). Domain services are typically used by the Application Services or other Domain Services, while Application Services are used by the Presentation Layer or client applications. Application services coordinate application flow and infrastructure but do not execute business logic rules or invariants. Domain services are unaware of infrastructure or overall application flow — they exclusively encapsulate business logic rules.

**Beginner-Friendly Explanation:** The **Application Service** is the "use case orchestrator." It knows what steps to take to complete a user's request: validate input, check permissions, start a transaction, call the right domain objects, save changes, and log the result. The **Domain Service** is the "business rule expert." It knows the rules that span multiple entities — like "an author's name must be unique across all authors" or "assigning an issue to a user cannot exceed a certain limit." The application service asks the domain service to make a decision; the domain service applies the rule. Neither replaces the other — they collaborate.

### Purposes

- To clearly separate use-case orchestration (application) from pure business logic (domain).
- To ensure that business rules remain independent of frameworks, databases, and delivery mechanisms.
- To prevent application services from becoming bloated with domain logic.
- To enable domain logic to be reused across multiple use cases.
- To keep the domain model clean and focused on the ubiquitous language.

### Syntax Rules and Structure

#### Application Service

```typescript
// Application Service — orchestrates use case
@Injectable()
export class BookAppService {
  constructor(
    private readonly bookRepository: BookRepository,
    private readonly bookManager: BookManager, // Domain service
    private readonly objectMapper: ObjectMapper,
  ) {}

  async createBookAsync(input: CreateBookDto): Promise<BookDto> {
    const author = await this.authorRepository.getAsync(input.authorId);
    const book = await this.bookManager.createAsync(input.title, author, input.price);
    await this.bookRepository.insertAsync(book);
    return this.objectMapper.map<Book, BookDto>(book);
  }
}
```

#### Domain Service

```typescript
// Domain Service — pure business logic
@Injectable()
export class BookManager {
  async createAsync(title: string, author: Author, price: number): Promise<Book> {
    if (price <= 0) throw new Error('Price must be positive');
    if (!author.isActive) throw new Error('Author is not active');
    return new Book(title, author, price);
  }
}
```

#### Comparison Table

| Aspect | Application Service | Domain Service |
|--------|---------------------|----------------|
| **Layer** | Application layer | Domain layer |
| **Responsibility** | Use-case orchestration | Core business logic |
| **Input/Output** | DTOs / Commands | Domain entities / value objects |
| **Transaction ownership** | Yes | No |
| **Authorization** | Yes | No |
| **Infrastructure awareness** | Yes (repositories, UoW, logging) | No |
| **Stateless** | Yes | Yes |
| **Used by** | Presentation layer | Application services, other domain services |
| **Typical method names** | `createBookAsync`, `transferFunds` | `createAsync`, `isNameUnique` |

**Constraints and Limitations:**
- Domain services should not perform authorization checks or depend on the current user context.
- Domain services should not contain query methods — only state-changing operations.
- Application services must not contain business rules that domain experts would want to change.
- Not every use case needs a domain service — if the logic fits naturally in an entity, put it there.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Application Service vs. Domain Service (NestJS)

```typescript
// domain/AuthorManager.ts — Domain Service
@Injectable()
export class AuthorManager {
  constructor(private readonly authorRepository: AuthorRepository) {}

  async createAsync(name: string): Promise<Author> {
    const existing = await this.authorRepository.findByNameAsync(name);
    if (existing) {
      throw new BusinessException('Author name already exists');
    }
    return new Author(name);
  }
}
```

```typescript
// application/AuthorAppService.ts — Application Service
@Injectable()
export class AuthorAppService {
  constructor(
    private readonly authorManager: AuthorManager,
    private readonly authorRepository: AuthorRepository,
    private readonly objectMapper: ObjectMapper,
  ) {}

  async createAuthorAsync(input: CreateAuthorDto): Promise<AuthorDto> {
    // Delegate business rule to domain service
    const author = await this.authorManager.createAsync(input.name);

    // Persist (application service decides when to insert)
    await this.authorRepository.insertAsync(author);

    // Map to DTO
    return this.objectMapper.map<Author, AuthorDto>(author);
  }
}
```

**Expected behaviour:** `createAuthorAsync()` accepts a DTO, calls `AuthorManager.createAsync()` (domain service) to enforce the "name must be unique" rule, inserts the author, and returns a DTO.

**Why this works:** The domain service enforces the business rule. The application service decides when to persist — maybe it needs to do additional work on the entity before inserting, as the ABP documentation suggests.

#### Example 2: When to Use a Domain Service (Cross-Aggregate Logic)

```typescript
// Domain Service — logic spans multiple aggregates
@Injectable()
export class IssueAssignmentService {
  constructor(
    private readonly issueRepository: IssueRepository,
    private readonly userRepository: UserRepository,
  ) {}

  async assignIssueAsync(issueId: string, userId: string): Promise<void> {
    const user = await this.userRepository.findByIdAsync(userId);
    const assignedCount = await this.issueRepository.countByAssigneeAsync(userId);

    if (assignedCount >= 5) {
      throw new BusinessException('User cannot be assigned more than 5 issues');
    }

    const issue = await this.issueRepository.findByIdAsync(issueId);
    issue.assignTo(user);
  }
}
```

**Expected behaviour:** The domain service enforces the rule "a user cannot have more than 5 assigned issues" — a rule that spans the `Issue` and `User` aggregates and therefore doesn't belong in either entity alone.

**Why this works:** Neither `Issue` nor `User` can enforce this rule alone. The domain service coordinates both aggregates and applies the business rule.

### Real-World Cases

- **E-commerce:** Application service orchestrates checkout; domain service enforces "maximum 3 items per customer per day."
- **Banking:** Application service handles transfer; domain service enforces "daily transfer limit across accounts."
- **Content management:** Application service creates an article; domain service enforces "author name must be unique."
- **SaaS multi-tenancy:** Application service handles tenant onboarding; domain service enforces "maximum users per plan."

---

## Core Concept 4: Cross-Layer Data Contracts (DTOs)

### Definitions

**Core Definition:** A Data Transfer Object (DTO) is an immutable, framework-agnostic object that carries data between layers (controller → service → repository) with strict type binding and no business logic.

**Technical Definition:** The DTO / Command is a typed input object carrying already-validated request data into the service. Application service methods should only accept and return DTOs, never directly expose domain entities. Immutable properties should be used whenever possible to ensure that data is not accidentally changed after being created. DTOs are stored under `dto/domain/` — they are part of the application layer.

**Beginner-Friendly Explanation:** A DTO is like a sealed envelope. The controller puts a request inside, seals it, and hands it to the application service. The service can read the contents but cannot modify them — and it never sees the raw internal domain objects. This keeps the API contract stable and prevents accidental leaks of internal data structures. Using DTOs also means the controller can be validated (e.g., with `class-validator`) before the service is even called.

### Purposes

- To strictly type-bind data moving from controllers down into services.
- To prevent domain entities from leaking to external consumers.
- To ensure immutability and prevent accidental mutation of request data.
- To enable validation at the boundary (controller) before business logic executes.
- To decouple the API contract from the internal domain model.

### Syntax Rules and Structure

#### DTO Pattern (TypeScript with class-validator)

```typescript
import { IsString, IsNumber, IsPositive, IsOptional, MaxLength } from 'class-validator';

export class CreateProductDto {
  @IsString()
  @MaxLength(200)
  readonly name: string;

  @IsOptional()
  @IsString()
  @MaxLength(2000)
  readonly description?: string;

  @IsNumber({ maxDecimalPlaces: 2 })
  @IsPositive()
  readonly price: number;
}
```

#### Command Pattern (Immutable, constructor-injected)

```typescript
export class TransferMoneyCommand {
  constructor(
    public readonly actorId: string,
    public readonly fromAccountId: string,
    public readonly toAccountId: string,
    public readonly amountCents: number,
  ) {}
}
```

| Component | Breakdown |
|-----------|-----------|
| `readonly` properties | Ensures immutability after construction. |
| `@IsString()` / `@IsNumber()` | Validation decorators (class-validator). |
| `constructor` injection | Used for Commands, ensuring all fields are set once. |
| No methods | DTOs contain no business logic — just data. |

#### Syntax Rules

- DTOs must be immutable — all properties should be `readonly`.
- DTOs must not contain business logic or persistence logic.
- DTOs should be named after the use case: `CreateProductDto`, `TransferMoneyCommand`, `QueryProductsDto`.
- Application services must accept and return DTOs, never domain entities.
- DTOs may have validation decorators but no other framework dependencies.
- Domain DTOs may depend on entities, but never the other way around.

#### Constraints and Limitations

- DTOs add boilerplate — for very simple applications, they may be overkill.
- Mapping between DTOs and entities requires a mapper (e.g., `ObjectMapper`, `class-transformer`).
- DTOs must be validated at the boundary; relying on the service to validate defeats the purpose.
- Commands (for write operations) and Queries (for read operations) are different DTO types and should be named accordingly.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: DTO with Validation (NestJS)

```typescript
// products/dto/create-product.dto.ts
import { IsString, IsNumber, IsPositive, IsOptional, MaxLength, Min } from 'class-validator';
import { Type } from 'class-transformer';

const MAX_NAME_LENGTH = 200;
const MAX_DESCRIPTION_LENGTH = 2000;
const MIN_PRICE = 0.01;

export class CreateProductDto {
  @IsString()
  @MaxLength(MAX_NAME_LENGTH)
  readonly name: string;

  @IsOptional()
  @IsString()
  @MaxLength(MAX_DESCRIPTION_LENGTH)
  readonly description?: string;

  @IsNumber({ maxDecimalPlaces: 2 })
  @IsPositive()
  @Min(MIN_PRICE)
  readonly price: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  @Type(() => Number)
  readonly stock?: number;
}
```

```typescript
// products/products.controller.ts
@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Post()
  async create(@Body() createProductDto: CreateProductDto): Promise<ProductDto> {
    // DTO is automatically validated by NestJS ValidationPipe
    return this.productsService.create(createProductDto);
  }
}
```

```typescript
// products/products.service.ts — Application Service
@Injectable()
export class ProductsService {
  async create(dto: CreateProductDto): Promise<ProductDto> {
    // Business logic delegates to domain
    const product = Product.create(dto.name, dto.price, dto.description);
    await this.productRepository.save(product);

    // Map entity to DTO — never return the entity
    return ProductMapper.toDto(product);
  }
}
```

**Expected behaviour:** The controller receives a validated `CreateProductDto`. The service uses the DTO's data to create a domain entity, persists it, and returns a `ProductDto`. The entity never leaves the service layer.

**Why this works:** Validation happens at the boundary (controller). The DTO is immutable. The service works with domain entities internally but exposes only DTOs externally.

#### Example 2: Command Pattern for Write Operations

```typescript
// application/commands/CreateOrderCommand.ts
export class CreateOrderCommand {
  constructor(
    public readonly userId: string,
    public readonly items: ReadonlyArray<OrderItemCommand>,
  ) {}
}

export class OrderItemCommand {
  constructor(
    public readonly productId: string,
    public readonly quantity: number,
  ) {}
}
```

```typescript
// application/OrderService.ts
export class OrderService {
  async createOrder(command: CreateOrderCommand): Promise<OrderDto> {
    // Validate command
    if (command.items.length === 0) {
      throw new Error('Order must contain at least one item');
    }

    // Load products, apply domain logic, persist
    const order = await this.orderFactory.createFromCommand(command);
    await this.orderRepository.save(order);

    return OrderMapper.toDto(order);
  }
}
```

**Expected behaviour:** The `CreateOrderCommand` carries all data needed for the use case. The service validates the command, delegates to a factory (domain service), persists, and returns a DTO.

**Why this works:** The Command is a special type of DTO specifically used for write operations. It is immutable and carries only data — no behaviour. The service reads the command and orchestrates the domain logic.

### Real-World Cases

- **REST APIs:** `CreateUserDto`, `UpdateProductDto`, `QueryOrdersDto` — each maps to a specific endpoint and use case.
- **GraphQL resolvers:** Input types serve as DTOs, validated before reaching services.
- **CQRS:** Commands and Queries are DTOs with distinct purposes — Commands mutate state, Queries read state.
- **Microservices:** DTOs define the contract between services, ensuring stable APIs.

---

## References

- Martin Fowler — Service Layer (PoEAA) — https://martinfowler.com/eaaCatalog/serviceLayer.html
- Martin Fowler — Anemic Domain Model — https://martinfowler.com/bliki/AnemicDomainModel.html
- Martin Fowler — Data Transfer Object (PoEAA) — https://martinfowler.com/eaaCatalog/dataTransferObject.html
- Eric Evans — Domain-Driven Design (Domain Services) — https://www.domainlanguage.com/ddd/
- ABP Framework — App Services vs Domain Services: Deep Dive — https://abp.io/community/articles/app-services-vs-domain-services-deep-dive-into-two-core-service-types-in-abp-framework-4dvau41u
- ABP Framework — Domain Services Documentation — https://docs.abp.io/en/abp/latest/Domain-Services
- Microsoft Learn — Implementing the Microservice Application Layer Using the Web API — https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-application-layer-implementation-web-api
- Telerik — Rich Domains: How to Use DDD for More Sustainable Systems — https://www.telerik.com/blogs/rich-domains-how-use-ddd-create-more-sustainable-systems
- DDD Europe 2024 — Are Anaemic Domain Models really considered harmful? — https://2024.dddeurope.com/program/are-anaemic-domain-models-really-considered-harmful/
- GitHub — Secure-PHP-Development/Service-Layer.md — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Architecture/Service-Layer.md
- Stack Overflow — Domain services vs Application services — https://stackoverflow.com/questions/3839386/domain-services-vs-application-services
- Stack Overflow — Is this domain or application service — https://stackoverflow.com/questions/38981882/is-this-domain-or-application-service
- NestJS Documentation — DTOs and Validation — https://docs.nestjs.com/techniques/validation