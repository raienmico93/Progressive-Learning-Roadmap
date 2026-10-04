# Enterprise Architectural Ecosystems — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Enterprise architectural ecosystems are the structural patterns and design philosophies that organise large-scale software systems around principles of separation of concerns, dependency inversion, domain modelling, and deployment boundaries — enabling maintainable, testable, and evolvable systems.

**Technical Definition:** Enterprise architectural patterns include Hexagonal Architecture (Ports and Adapters), Clean Architecture (Concentric Layers), Modular Monoliths, and Domain-Driven Design (DDD). Hexagonal Architecture isolates business logic from infrastructure via ports (interfaces) and adapters (implementations). Clean Architecture organises code into concentric layers with a strict Dependency Rule: source code dependencies can only point inward. A Modular Monolith organises a single deployable unit into loosely coupled modules with well-defined boundaries, facilitating future extraction to microservices. DDD organises sub-systems around bounded contexts, aggregate roots, value objects, and domain events, aligning software structure with business domains.

**Beginner-Friendly Explanation:** Imagine you're building a city. **Hexagonal Architecture** says: "Build the city centre (business logic) and connect it to the outside world through controlled gates (ports) with translators (adapters) — so you can replace the roads or airports without rebuilding the city centre." **Clean Architecture** says: "Build your city in concentric rings — the innermost ring is the most important (city hall, laws), and each outer ring depends only on the rings inside it. The outer rings are just details (roads, buildings) that can change without affecting the laws." **Modular Monolith** says: "Build one city hall (one deployment) but organise it into independent departments (modules) with clear walls between them — so you can later split them into separate buildings (microservices) if needed." **DDD** says: "Organise your city into districts (bounded contexts), each with its own rules and language. Within each district, define what's a 'thing' (entity), what's a 'description' (value object), and what's a 'cluster of related things that must stay consistent' (aggregate)."

### Key Characteristics

- **Dependency inversion:** All patterns enforce that business logic does not depend on infrastructure — infrastructure depends on business logic.
- **Testability:** Business rules can be tested without databases, HTTP servers, or frameworks.
- **Technology adaptability:** Databases, UIs, and external services can be swapped without modifying business logic.
- **Domain alignment:** Software structure reflects the business domain, not technical layers.
- **Explicit boundaries:** Modules, bounded contexts, and aggregates define clear ownership and consistency boundaries.
- **Deployment flexibility:** Modular monoliths can evolve into microservices; hexagonal architectures can adapt to multiple input/output mechanisms.

### Prerequisites

- **Object-oriented programming:** Classes, interfaces, inheritance, and composition.
- **Dependency injection:** Constructor injection and inversion of control.
- **Domain modelling concepts:** Entities, value objects, and business rules.
- **Layered architecture familiarity:** Understanding of presentation, application, domain, and infrastructure layers.
- **Design patterns:** Repository, Unit of Work, Factory, and Strategy patterns.
- **Testing fundamentals:** Unit testing, mocking, and dependency substitution.

### Related Programming Areas

- **Domain-Driven Design (DDD):** Hexagonal Architecture and Clean Architecture work especially well with DDD.
- **Microservices:** Modular Monoliths are often the first step toward microservices extraction.
- **Event-Driven Architecture:** Domain events are the primary coordination mechanism across bounded contexts.
- **CQRS and Event Sourcing:** These patterns complement DDD and Clean Architecture.
- **Test-Driven Development (TDD):** Hexagonal Architecture was designed to support testing in isolation.

### Core Concepts

1. **Hexagonal / Ports and Adapters** — protecting core business logic from external dependencies via abstraction adapters.
2. **Clean Architecture** — structuring codebases into strict concentric layers where code dependencies point exclusively inward.
3. **Modular Monoliths** — enforcing physical domain boundaries inside a single, unified deployment artifact.
4. **Domain-Driven Design (DDD)** — organising sub-systems around bounded contexts, aggregate roots, value objects, and domain events.

---

## Core Concept 1: Hexagonal / Ports and Adapters

### Definitions

**Core Definition:** Hexagonal Architecture, also known as Ports and Adapters, is an architectural pattern that isolates business logic from external concerns (databases, UIs, external APIs) by defining technology-agnostic interfaces (ports) and implementing them with pluggable adapters.

**Technical Definition:** The hexagonal architecture pattern, proposed by Dr. Alistair Cockburn in 2005, aims to create loosely coupled architectures where application components can be tested independently, with no dependencies on data stores or user interfaces. The application communicates with external components over interfaces called **ports**, and uses **adapters** to translate the technical exchanges with these components. Ports are technology-agnostic entry points into an application component, defining the interface that allows external actors to communicate with the application. Adapters implement these interfaces, connecting the application to external resources.

**Beginner-Friendly Explanation:** Think of your application as a hexagon. Inside the hexagon is your pure business logic — the rules that define what your application does. On the edges of the hexagon are **ports** — simple interfaces that say "if you want to talk to my application, here's the shape of the conversation." Outside the hexagon are **adapters** — the actual implementations that connect to real databases, REST APIs, message queues, or UIs. If you want to swap PostgreSQL for MongoDB, you write a new adapter for the same port. The business logic inside the hexagon never changes.

### Purposes

- To isolate business logic from infrastructure code such as database access, external APIs, and UI frameworks.
- To enable application components to be tested independently, without dependencies on data stores or user interfaces.
- To prevent technology lock-in of data stores and UIs, making it easier to change the technology stack over time with limited or no impact to business logic.
- To allow multiple types of clients to use the same domain logic.
- To support multiple input providers (HTTP, CLI, queue) and output consumers (databases, message brokers, files) without complicating application logic.

### Syntax Rules and Structure

#### General Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      OUTSIDE (Adapters)                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ HTTP     │  │ CLI      │  │ Database │  │ Message  │       │
│  │ Adapter  │  │ Adapter  │  │ Adapter  │  │ Queue    │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │              │              │              │              │
│       ▼              ▼              ▼              ▼              │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    PORTS (Interfaces)                       ││
│  │  ┌─────────────────────────────────────────────────────┐  ││
│  │  │              APPLICATION CORE                        │  ││
│  │  │  (Domain Entities, Value Objects, Domain Services)  │  ││
│  │  │                                                     │  ││
│  │  │  Business rules live here. No infrastructure.       │  ││
│  │  └─────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Port Interface (TypeScript)

```typescript
// domain/ports/UserRepositoryPort.ts — Port (interface)
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// domain/ports/NotificationPort.ts — Another port
export interface NotificationPort {
  sendWelcomeEmail(email: string): Promise<void>;
}
```

#### Adapter Implementation (TypeScript)

```typescript
// infrastructure/adapters/PostgresUserRepositoryAdapter.ts — Adapter
import { UserRepositoryPort } from '../../domain/ports/UserRepositoryPort';

export class PostgresUserRepositoryAdapter implements UserRepositoryPort {
  constructor(private readonly pool: Pool) {}

  async findById(id: string): Promise<User | null> {
    const result = await this.pool.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] ? this.toDomain(result.rows[0]) : null;
  }

  async save(user: User): Promise<void> {
    await this.pool.query(
      'INSERT INTO users (id, email, name) VALUES ($1, $2, $3) ON CONFLICT (id) DO UPDATE SET email = $2, name = $3',
      [user.id, user.email, user.name],
    );
  }
}
```

#### Syntax Rules

- **Ports belong to the domain layer** — they are interfaces defined by the application core, not by infrastructure.
- **Adapters belong to the infrastructure layer** — they implement ports and depend on external technologies.
- **The application core depends on ports, never on adapters** — this is the Dependency Inversion Principle in action.
- **Ports should be technology-agnostic** — no SQL, HTTP, or framework-specific types in port interfaces.
- **Multiple adapters can implement the same port** — e.g., `PostgresUserRepositoryAdapter` and `InMemoryUserRepositoryAdapter` for testing.

#### Constraints and Limitations

- **Additional adapter code** adds maintenance overhead; it is justified only when the application requires multiple input sources and output destinations, or when inputs and data stores change over time.
- **Latency** may increase because ports and adapters add another layer of indirection.
- **Complexity** of separating business logic from infrastructure code requires careful handling to realise benefits such as agility, test coverage, and technology adaptability.
- **Over-engineering risk:** For simple applications with a single input and output, the pattern may add unnecessary abstraction.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hexagonal Architecture with TypeScript and NestJS

**Step-by-step setup:**
1. Define the domain entity (`User`).
2. Define the port (interface) for persistence.
3. Define the port (interface) for notifications.
4. Implement the application service (use case) depending only on ports.
5. Implement PostgreSQL and in-memory adapters for the repository port.
6. Implement an email adapter for the notification port.
7. Wire everything via dependency injection.

```typescript
// domain/User.ts — Domain entity (no infrastructure)
export class User {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly name: string,
  ) {}

  static create(email: string, name: string): User {
    if (!email.includes('@')) throw new Error('Invalid email');
    return new User(crypto.randomUUID(), email, name);
  }
}

// domain/ports/UserRepositoryPort.ts — Port
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// domain/ports/NotificationPort.ts — Port
export interface NotificationPort {
  sendWelcomeEmail(email: string): Promise<void>;
}

// application/RegisterUserUseCase.ts — Application service depends only on ports
export class RegisterUserUseCase {
  constructor(
    private readonly userRepository: UserRepositoryPort,
    private readonly notificationPort: NotificationPort,
  ) {}

  async execute(email: string, name: string): Promise<User> {
    const existing = await this.userRepository.findByEmail(email);
    if (existing) throw new Error('Email already registered');

    const user = User.create(email, name);
    await this.userRepository.save(user);
    await this.notificationPort.sendWelcomeEmail(user.email);

    return user;
  }
}
```

```typescript
// infrastructure/adapters/PostgresUserRepositoryAdapter.ts — Adapter
export class PostgresUserRepositoryAdapter implements UserRepositoryPort {
  constructor(private readonly pool: Pool) {}

  async findById(id: string): Promise<User | null> {
    const result = await this.pool.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] ? new User(result.rows[0].id, result.rows[0].email, result.rows[0].name) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const result = await this.pool.query('SELECT * FROM users WHERE email = $1', [email]);
    return result.rows[0] ? new User(result.rows[0].id, result.rows[0].email, result.rows[0].name) : null;
  }

  async save(user: User): Promise<void> {
    await this.pool.query(
      'INSERT INTO users (id, email, name) VALUES ($1, $2, $3)',
      [user.id, user.email, user.name],
    );
  }
}

// infrastructure/adapters/InMemoryUserRepositoryAdapter.ts — Test adapter
export class InMemoryUserRepositoryAdapter implements UserRepositoryPort {
  private users = new Map<string, User>();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async findByEmail(email: string): Promise<User | null> {
    return [...this.users.values()].find(u => u.email === email) ?? null;
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id, user);
  }
}
```

**Expected behaviour:** The `RegisterUserUseCase` is constructed with any `UserRepositoryPort` implementation — PostgreSQL in production, in-memory in tests. The business logic (email uniqueness check, user creation) never changes.

**Why this works:** The application core depends only on the `UserRepositoryPort` and `NotificationPort` interfaces. Adapters implement these interfaces and can be swapped without modifying the core.

### Real-World Cases

- **Multi-client APIs:** The same domain logic is driven by HTTP controllers, CLI commands, and queue consumers.
- **Technology migration:** Migrating from MongoDB to PostgreSQL requires only a new adapter — the business logic stays unchanged.
- **Testing:** Unit tests use `InMemoryUserRepositoryAdapter` — no database required.
- **External service integration:** Payment gateways, email providers, and messaging systems are accessed through ports, making them replaceable.

---

## Core Concept 2: Clean Architecture

### Definitions

**Core Definition:** Clean Architecture is an architectural style that organises software into concentric layers — Entities (innermost), Use Cases, Interface Adapters, and Frameworks & Drivers (outermost) — with a strict Dependency Rule: source code dependencies can only point inward.

**Technical Definition:** Clean Architecture, proposed by Robert C. Martin (Uncle Bob), integrates several architectural ideas (Hexagonal Architecture, Onion Architecture, Screaming Architecture, DCI, and BCE) into a single actionable pattern. The architecture produces systems that are independent of frameworks, testable, independent of UI, independent of database, and independent of any external agency. The overriding rule is the **Dependency Rule**: source code dependencies can only point inwards. Nothing in an inner circle can know anything at all about something in an outer circle.

**Beginner-Friendly Explanation:** Imagine an onion. The centre of the onion is your most important business rules — the core logic that would exist even if you had no database, no web server, and no user interface. Each layer outward is less important and more "detail." Clean Architecture says: "Dependencies must always point inward. The inner layers don't know the outer layers exist. Your business logic doesn't know you're using Express or PostgreSQL — those are just details."

### Purposes

- To make business rules independent of frameworks, allowing frameworks to be used as tools rather than constraints.
- To make business rules testable without the UI, database, web server, or any other external element.
- To make the UI replaceable (e.g., web UI to console UI) without changing business rules.
- To make the database replaceable (e.g., Oracle to MongoDB) without changing business rules.
- To ensure business rules know nothing about the outside world, enabling them to be reused across applications.
- To enforce the Dependency Rule so that outer-layer changes do not impact inner layers.

### Syntax Rules and Structure

#### The Four Layers

| Layer | Contents | Dependency |
|-------|----------|------------|
| **Entities** | Enterprise-wide business rules; core business objects with methods or data structures. | Innermost — depends on nothing. |
| **Use Cases** | Application-specific business rules; orchestrates data flow to and from entities. | Depends on Entities only. |
| **Interface Adapters** | Controllers, Presenters, Gateways; converts data between use-case format and external format. | Depends on Use Cases and Entities. |
| **Frameworks & Drivers** | Database, Web framework, external tools; glue code. | Outermost — depends on all inner layers. |

#### The Dependency Rule

```
┌─────────────────────────────────────────────┐
│  Frameworks & Drivers (outermost)           │
│  ┌─────────────────────────────────────┐   │
│  │  Interface Adapters                 │   │
│  │  ┌─────────────────────────────┐   │   │
│  │  │  Use Cases                  │   │   │
│  │  │  ┌─────────────────────┐   │   │   │
│  │  │  │  Entities           │   │   │   │
│  │  │  │  (innermost)        │   │   │   │
│  │  │  └─────────────────────┘   │   │   │
│  │  └─────────────────────────────┘   │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘

Dependencies point INWARD:
Frameworks → Interface Adapters → Use Cases → Entities
```

#### Entity (TypeScript)

```typescript
// domain/entities/Order.ts — Innermost layer
export class Order {
  private items: OrderItem[] = [];
  private status: OrderStatus = 'draft';

  constructor(public readonly id: string) {}

  addItem(item: OrderItem): void {
    if (this.status !== 'draft') throw new Error('Cannot modify a non-draft order');
    this.items.push(item);
  }

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
}
```

#### Use Case (TypeScript)

```typescript
// application/use-cases/CreateOrderUseCase.ts — Use Cases layer
export class CreateOrderUseCase {
  constructor(
    private readonly orderRepository: OrderRepository, // Interface
    private readonly unitOfWork: UnitOfWork,           // Interface
  ) {}

  async execute(input: CreateOrderInput): Promise<OrderOutput> {
    return this.unitOfWork.transaction(async (manager) => {
      const order = new Order(crypto.randomUUID());
      for (const item of input.items) {
        order.addItem(new OrderItem(item.productId, item.price, item.quantity));
      }
      await this.orderRepository.save(order, manager);
      return { id: order.id, total: order.getTotal() };
    });
  }
}
```

#### Interface Adapter — Controller (TypeScript)

```typescript
// interfaces/controllers/OrderController.ts — Interface Adapters layer
@Controller('orders')
export class OrderController {
  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}

  @Post()
  async create(@Body() dto: CreateOrderDto): Promise<OrderResponseDto> {
    const output = await this.createOrderUseCase.execute({
      items: dto.items,
    });
    return { id: output.id, total: output.total };
  }
}
```

#### Frameworks & Drivers — Repository Implementation

```typescript
// infrastructure/repositories/TypeOrmOrderRepository.ts — Frameworks & Drivers
export class TypeOrmOrderRepository implements OrderRepository {
  constructor(private readonly dataSource: DataSource) {}

  async save(order: Order, manager: EntityManager): Promise<void> {
    await manager.save(this.toOrmEntity(order));
  }

  private toOrmEntity(order: Order): OrderEntity {
    // Convert domain Order to TypeORM entity
  }
}
```

#### Syntax Rules

- **The Dependency Rule is absolute:** Nothing in an inner circle can know anything about something in an outer circle.
- **Data formats used in an outer circle should not be used by an inner circle** — especially framework-generated formats.
- **Crossing boundaries:** Control flow can go outward (use case calls presenter), but dependencies always point inward. This is resolved via the Dependency Inversion Principle.
- **The circles are schematic:** You may need more than four layers, but the Dependency Rule always applies.
- **Entities encapsulate enterprise-wide business rules** — the most general and high-level rules, least likely to change.
- **Use Cases contain application-specific business rules** — they orchestrate data flow to and from entities.

#### Constraints and Limitations

- **The circles are schematic, not prescriptive** — there is no rule that says you must always have exactly four layers, but the Dependency Rule always applies.
- **Complexity:** Clean Architecture introduces significant structure and indirection; for simple CRUD applications, it may be overkill.
- **Learning curve:** The Dependency Rule and boundary crossing require understanding of the Dependency Inversion Principle.
- **Over-abstraction:** Too many layers can lead to "architecture astronaut" syndrome, where the structure becomes more complex than the problem it solves.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Clean Architecture with TypeScript

```typescript
// src/domain/entities/User.ts — Entities (innermost)
export class User {
  private constructor(
    public readonly id: string,
    private email: string,
    private name: string,
  ) {}

  static create(email: string, name: string): User {
    if (!email.includes('@')) throw new Error('Invalid email');
    return new User(crypto.randomUUID(), email, name);
  }

  changeName(name: string): void {
    if (!name.trim()) throw new Error('Name cannot be empty');
    this.name = name;
  }
}

// src/domain/repositories/UserRepository.ts — Port (interface)
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// src/application/use-cases/RegisterUserUseCase.ts — Use Cases
export class RegisterUserUseCase {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(email: string, name: string): Promise<{ id: string }> {
    const user = User.create(email, name);
    await this.userRepository.save(user);
    return { id: user.id };
  }
}

// src/interfaces/controllers/UserController.ts — Interface Adapters
@Controller('users')
export class UserController {
  constructor(private readonly registerUserUseCase: RegisterUserUseCase) {}

  @Post()
  async register(@Body() dto: RegisterUserDto): Promise<{ id: string }> {
    return this.registerUserUseCase.execute(dto.email, dto.name);
  }
}

// src/infrastructure/repositories/PrismaUserRepository.ts — Frameworks & Drivers
export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.prisma.user.findUnique({ where: { id } });
    return row ? User.create(row.email, row.name) : null;
  }

  async save(user: User): Promise<void> {
    await this.prisma.user.upsert({
      where: { id: user.id },
      update: { email: user.email, name: user.name },
      create: { id: user.id, email: user.email, name: user.name },
    });
  }
}
```

**Expected behaviour:** The `RegisterUserUseCase` depends only on the `UserRepository` interface (defined in the domain layer). The `PrismaUserRepository` (in the infrastructure layer) implements this interface. The controller (interface adapter) invokes the use case. All dependencies point inward: `PrismaUserRepository` → `UserRepository` interface ← `RegisterUserUseCase` → `User` entity.

**Why this works:** The Dependency Rule is enforced. The domain layer (`User`, `UserRepository`) has no dependencies on Prisma, NestJS, or any framework. The use case layer depends only on the domain. The infrastructure layer depends on the domain interfaces. Swapping Prisma for TypeORM requires only a new implementation of `UserRepository`.

### Real-World Cases

- **Enterprise applications:** Large systems with complex business rules that must outlast framework and database changes.
- **Multi-platform products:** The same business logic serves web, mobile, desktop, and CLI clients.
- **Long-lived systems:** Applications expected to be maintained for 5–10+ years, where framework churn is a risk.
- **Test-driven development:** Clean Architecture enables testing business rules in isolation.

---

## Core Concept 3: Modular Monoliths

### Definitions

**Core Definition:** A Modular Monolith is a software architecture pattern that organises a single deployable application into loosely coupled modules with well-defined boundaries and explicit dependencies, combining the simplicity of a monolith with the modularity of microservices.

**Technical Definition:** Modular Monolith is a software architecture pattern that strategically combines the simplicity of a monolithic structure with the advantages of microservices. The system is organised into loosely coupled modules, each delineating well-defined boundaries and explicit dependencies on other modules. The goal is to achieve independence and isolation for each module, allowing them to be worked on independently while still being deployed collectively as a single unit. Crucially, it can continue to be migrated to microservices or remain unchanged.

**Beginner-Friendly Explanation:** Imagine a large office building. A **classic monolith** is one giant open-plan office — everyone can talk to everyone, but it's chaotic and hard to change. **Microservices** are separate buildings — each team has its own building, but now you need roads, phones, and coordination between buildings. A **Modular Monolith** is one building with separate floors and locked doors. Each team has its own floor (module) with clear boundaries. They can work independently but still share the same building (deployment). If a team outgrows its floor, you can move them to their own building later.

### Purposes

- To combine the simplicity of a single deployable unit with the modularity and independence of microservices.
- To enforce well-defined boundaries and explicit dependencies between modules, preventing the "big ball of mud" anti-pattern.
- To enable independent development, testing, and deployment of modules within a single runtime.
- To facilitate future migration to microservices without a complete rewrite.
- To reduce the operational complexity of distributed systems (no network calls, no distributed tracing, no eventual consistency across services).
- To allow different teams to work on different modules with minimal coordination.

### Syntax Rules and Structure

#### Module Structure (NestJS)

```
src/
├── modules/
│   ├── orders/
│   │   ├── orders.module.ts        # Module definition
│   │   ├── orders.controller.ts    # HTTP entry point
│   │   ├── orders.service.ts       # Business logic
│   │   ├── orders.repository.ts    # Data access
│   │   ├── dto/                    # Data transfer objects
│   │   └── entities/               # Domain entities
│   ├── payments/
│   │   ├── payments.module.ts
│   │   ├── payments.controller.ts
│   │   ├── payments.service.ts
│   │   └── payments.repository.ts
│   └── users/
│       ├── users.module.ts
│       ├── users.controller.ts
│       ├── users.service.ts
│       └── users.repository.ts
├── shared/                         # Shared kernel (minimal)
│   ├── database/
│   └── common/
└── app.module.ts                   # Root module
```

#### Module Definition (NestJS)

```typescript
// modules/orders/orders.module.ts
@Module({
  imports: [PaymentsModule, UsersModule], // Explicit dependencies
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService], // Only expose what other modules need
})
export class OrdersModule {}
```

#### Cross-Module Communication via Facade

```typescript
// modules/users/users.facade.ts — Public API for the Users module
@Injectable()
export class UsersFacade {
  constructor(private readonly usersService: UsersService) {}

  async getActiveUserIds(): Promise<string[]> {
    return this.usersService.findActiveUserIds();
  }
}

// modules/orders/orders.service.ts — Orders module uses the facade
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly usersFacade: UsersFacade, // Facade, not direct service
  ) {}

  async createOrder(userId: string, items: OrderItemDto[]): Promise<Order> {
    const user = await this.usersFacade.getActiveUserIds();
    if (!user.includes(userId)) throw new Error('User is not active');
    return this.ordersRepository.create(userId, items);
  }
}
```

#### Syntax Rules

- **Each module must have well-defined boundaries** — it exposes a public API (facade) and hides its internals.
- **Modules communicate through facades or domain events** — never through direct access to another module's internals.
- **Dependencies between modules must be explicit** — declared in module imports and facade injection.
- **The shared kernel must be minimal** — only truly cross-cutting concerns (database config, common utilities).
- **The database schema may be shared** — unlike microservices, a modular monolith can use a single database schema.
- **Modules should be vertically sliced** — each module contains its own domain, application, and infrastructure layers.

#### Constraints and Limitations

- **No rigid data ownership delineation** among modules — unlike microservices, modules may share database tables.
- **Monolithic deployment structure:** All modules operate within the same VM or process; scaling is limited to scaling the entire application.
- **Module boundaries require discipline** — without enforcement, modules can become tightly coupled over time.
- **Testing across modules** requires integration tests, as modules are not independently deployable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Modular Monolith with NestJS

**Step-by-step setup:**
1. Create the root `AppModule` that imports all feature modules.
2. Create a `UsersModule` with its own controller, service, and repository.
3. Create an `OrdersModule` that imports `UsersModule` and uses its facade.
4. Create a `PaymentsModule` that listens to order events.
5. Enforce boundaries: modules cannot access each other's repositories directly.

```typescript
// app.module.ts — Root module
@Module({
  imports: [
    DatabaseModule,
    UsersModule,
    OrdersModule,
    PaymentsModule,
  ],
})
export class AppModule {}
```

```typescript
// modules/users/users.module.ts
@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository, UsersFacade],
  exports: [UsersFacade], // Only the facade is exported
})
export class UsersModule {}
```

```typescript
// modules/orders/orders.module.ts
@Module({
  imports: [UsersModule], // Explicit dependency on UsersModule
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}
```

```typescript
// modules/orders/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly usersFacade: UsersFacade, // Injected from UsersModule
    private readonly eventEmitter: EventEmitter2,
  ) {}

  async createOrder(userId: string, items: OrderItemDto[]): Promise<Order> {
    // Business rule: user must be active
    const user = await this.usersFacade.getUserById(userId);
    if (!user || !user.isActive) throw new Error('User is not active');

    const order = await this.ordersRepository.create(userId, items);

    // Notify other modules via event
    this.eventEmitter.emit('order.created', new OrderCreatedEvent(order.id, order.total));

    return order;
  }
}
```

**Expected behaviour:** The `OrdersService` can call `UsersFacade` (the public API of the Users module) but cannot access `UsersRepository` directly. The `PaymentsModule` listens for `order.created` events and processes payments asynchronously.

**Why this works:** Module boundaries are enforced by NestJS's module system and the facade pattern. The `UsersModule` hides its repository and service internals, exposing only `UsersFacade`. The `OrdersModule` depends explicitly on `UsersModule` and communicates through the facade. If the `UsersModule` needs to be extracted into a microservice, only the facade implementation changes — the `OrdersModule` remains unchanged.

### Real-World Cases

- **Shopify:** Uses a modular monolith architecture to manage its e-commerce platform.
- **Appsmith:** Migrated from microservices to a modular monolith for simplicity.
- **Gusto (Time Tracking):** Uses modular monolith for its time-tracking product.
- **PlayTech (Casino Backend):** Uses modular monolith for its casino backend.
- **Startups and scale-ups:** When teams are fewer than 50 engineers and the domain is still evolving, modular monoliths offer better returns than microservices.

---

## Core Concept 4: Domain-Driven Design (DDD)

### Definitions

**Core Definition:** Domain-Driven Design (DDD) is an approach to software development that organises sub-systems around bounded contexts, aggregate roots, value objects, and domain events, aligning software structure with the business domain and its ubiquitous language.

**Technical Definition:** Domain-driven design (DDD) rejects a single unified model for the entire system. It instead encourages you to divide the system into bounded contexts that each has its own model. Within each bounded context, tactical DDD patterns define the domain model more precisely: **Entities** (objects with unique identity), **Value Objects** (immutable objects defined by their attributes), **Aggregates** (consistency boundaries around one or more entities, with exactly one root entity), **Domain Services** (stateless objects implementing logic that spans multiple entities or aggregates), and **Domain Events** (domain-significant changes that aggregates raise after state changes).

**Beginner-Friendly Explanation:** DDD says: "Don't try to model your entire business with one giant model. Instead, divide your business into **bounded contexts** — distinct areas where specific terms and rules apply." For example, in a shipping company, "package" means something different in the warehouse context than in the billing context. Within each context, you define your **entities** (things with identity, like a `Customer` or `Order`), **value objects** (descriptions without identity, like `Address` or `Money`), **aggregates** (clusters of entities that must stay consistent together, accessed through a root), and **domain events** (things that happened, like `OrderPlaced` or `DeliveryCompleted`).

### Purposes

- To align software structure with business domains, making the code reflect how the business thinks.
- To define clear boundaries (bounded contexts) where specific terms and rules apply.
- To model complex business rules through entities, value objects, and aggregates.
- To enforce transactional invariants through aggregate consistency boundaries.
- To coordinate work across aggregates and bounded contexts via domain events.
- To prevent the anemic domain model anti-pattern by embedding behaviour in entities.
- To enable a ubiquitous language shared by developers and domain experts.

### Syntax Rules and Structure

#### Strategic Design: Bounded Contexts

```
┌─────────────────────────────────────────────────────────────┐
│                    E-COMMERCE SYSTEM                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │  Sales Context  │  │ Shipping Context│  │Billing Ctx  │ │
│  │                 │  │                 │  │             │ │
│  │  - Customer     │  │  - Delivery     │  │ - Invoice   │ │
│  │  - Order        │  │  - Package      │  │ - Payment   │ │
│  │  - LineItem     │  │  - Drone        │  │ - Account   │ │
│  │                 │  │                 │  │             │ │
│  │  "Order" =      │  │  "Order" =      │  │ "Order" =   │ │
│  │  purchase intent│  │  shipping task  │  │ billing ref │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### Tactical Patterns

| Pattern | Definition | Example |
|---------|------------|---------|
| **Entity** | Object with unique identity that persists over time. | `Customer`, `Order`, `Account` |
| **Value Object** | Immutable object defined only by its attributes; no identity. | `Address`, `Money`, `DateRange` |
| **Aggregate** | Consistency boundary around one or more entities; exactly one root. | `Order` (root) with `OrderLine` children |
| **Domain Service** | Stateless object implementing logic spanning multiple entities. | `Scheduler`, `PricingService` |
| **Domain Event** | Domain-significant change raised by an aggregate. | `OrderPlaced`, `DeliveryCompleted` |
| **Repository** | Provides collection-like access to aggregates. | `OrderRepository` |

#### Entity with Value Object and Domain Event (TypeScript)

```typescript
// domain/value-objects/Money.ts — Value Object (immutable)
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {
    if (amount < 0) throw new Error('Amount cannot be negative');
    if (currency.length !== 3) throw new Error('Currency must be a 3-letter code');
  }

  static create(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) throw new Error('Currency mismatch');
    return new Money(this.amount + other.amount, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}

// domain/events/OrderPlacedEvent.ts — Domain Event
export class OrderPlacedEvent {
  constructor(
    public readonly orderId: string,
    public readonly customerId: string,
    public readonly total: Money,
    public readonly occurredAt: Date = new Date(),
  ) {}
}

// domain/entities/Order.ts — Aggregate Root
export class Order extends AggregateRoot {
  private items: OrderItem[] = [];
  private status: OrderStatus = 'draft';
  private total: Money;

  private constructor(
    public readonly id: string,
    public readonly customerId: string,
  ) {
    super();
    this.total = Money.create(0, 'USD');
  }

  static create(customerId: string): Order {
    return new Order(crypto.randomUUID(), customerId);
  }

  addItem(productId: string, price: Money, quantity: number): void {
    if (this.status !== 'draft') throw new Error('Cannot modify a non-draft order');
    if (quantity <= 0) throw new Error('Quantity must be positive');

    const item = new OrderItem(productId, price, quantity);
    this.items.push(item);
    this.total = this.total.add(price.multiply(quantity));
  }

  place(): void {
    if (this.status !== 'draft') throw new Error('Order has already been placed');
    if (this.items.length === 0) throw new Error('Cannot place an empty order');

    this.status = 'placed';
    this.raise(new OrderPlacedEvent(this.id, this.customerId, this.total));
  }

  getTotal(): Money {
    return this.total;
  }
}
```

#### AggregateRoot Base Class (TypeScript)

```typescript
// domain/AggregateRoot.ts
export abstract class AggregateRoot {
  private domainEvents: DomainEvent[] = [];

  protected raise(event: DomainEvent): void {
    this.domainEvents.push(event);
  }

  pullDomainEvents(): DomainEvent[] {
    const events = [...this.domainEvents];
    this.domainEvents = [];
    return events;
  }
}
```

#### Domain Service (TypeScript)

```typescript
// domain/services/PricingService.ts — Domain Service
export class PricingService {
  calculateDiscount(customer: Customer, order: Order): Money {
    if (customer.isPremium()) {
      return order.getTotal().multiply(0.1);
    }
    return Money.create(0, 'USD');
  }
}
```

#### Syntax Rules

- **Bounded contexts define linguistic boundaries** — a term like "Order" may mean different things in different contexts.
- **Entities must have unique identity** — two entities with the same identity are the same, even if attributes differ.
- **Value objects must be immutable** — updating a value object means creating a new instance.
- **Aggregates must be accessed only through their root** — external code references other aggregates by identity, not direct object references.
- **Aggregates must be small** — include only data that must remain consistent within a single transaction.
- **Domain events are raised after state changes** — they coordinate work across aggregate boundaries.
- **Domain services are stateless** — they implement logic that doesn't naturally belong to an entity or value object.
- **Application services orchestrate use cases** — they coordinate domain services and repositories, manage transactions, and handle cross-cutting concerns.

#### Constraints and Limitations

- **A single unified model for the entire system is rejected** — DDD requires dividing the system into bounded contexts, which adds complexity.
- **Anemic domain models undermine DDD** — when business logic lives outside entities in service classes, the benefit of DDD is lost.
- **Small aggregates are essential** — combining unrelated aggregates forces unrelated updates to compete for the same locks.
- **Eventual consistency across aggregates** — when a business process spans multiple aggregates, domain events and eventual consistency replace single transactions.
- **DDD requires domain expertise** — without access to domain experts, the ubiquitous language and model will be incorrect.
- **DDD is not for simple CRUD** — for applications with minimal business logic, DDD adds unnecessary complexity.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: DDD with TypeScript — Order Aggregate, Value Objects, and Domain Events

**Step-by-step setup:**
1. Define value objects (`Money`, `OrderItem`).
2. Define the aggregate root (`Order`) with behaviour and domain event raising.
3. Define domain events (`OrderPlacedEvent`).
4. Define the repository interface (`OrderRepository`).
5. Implement the application service (`PlaceOrderUseCase`).
6. Wire domain event handlers.

```typescript
// domain/value-objects/Money.ts
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {
    if (amount < 0) throw new Error('Amount cannot be negative');
    if (currency.length !== 3) throw new Error('Currency must be a 3-letter code');
  }

  static create(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) throw new Error('Currency mismatch');
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }
}
```

```typescript
// domain/entities/Order.ts — Aggregate Root
export class Order extends AggregateRoot {
  private items: OrderItem[] = [];
  private status: OrderStatus = 'draft';
  private total: Money;

  private constructor(
    public readonly id: string,
    public readonly customerId: string,
  ) {
    super();
    this.total = Money.create(0, 'USD');
  }

  static create(customerId: string): Order {
    return new Order(crypto.randomUUID(), customerId);
  }

  addItem(productId: string, price: Money, quantity: number): void {
    if (this.status !== 'draft') throw new Error('Cannot modify a non-draft order');
    if (quantity <= 0) throw new Error('Quantity must be positive');

    const item = new OrderItem(productId, price, quantity);
    this.items.push(item);
    this.total = this.total.add(price.multiply(quantity));
  }

  place(): void {
    if (this.status !== 'draft') throw new Error('Order has already been placed');
    if (this.items.length === 0) throw new Error('Cannot place an empty order');

    this.status = 'placed';
    this.raise(new OrderPlacedEvent(this.id, this.customerId, this.total));
  }

  getTotal(): Money {
    return this.total;
  }
}
```

```typescript
// application/PlaceOrderUseCase.ts
export class PlaceOrderUseCase {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventDispatcher: EventDispatcher,
  ) {}

  async execute(customerId: string, items: OrderItemDto[]): Promise<{ orderId: string }> {
    const order = Order.create(customerId);

    for (const item of items) {
      order.addItem(item.productId, Money.create(item.price, item.currency), item.quantity);
    }

    order.place();

    await this.orderRepository.save(order);

    // Dispatch domain events after persistence
    const events = order.pullDomainEvents();
    await this.eventDispatcher.dispatch(events);

    return { orderId: order.id };
  }
}
```

**Expected behaviour:** Calling `placeOrderUseCase.execute()` creates an `Order` aggregate, adds items (enforcing quantity rules via value objects), places the order (enforcing non-empty rule), saves it, and dispatches an `OrderPlacedEvent`. The `PaymentsModule` listens to this event and initiates payment processing.

**Why this works:** The `Order` aggregate encapsulates all business rules for order creation and placement. Value objects (`Money`, `OrderItem`) enforce their own invariants. Domain events decouple the order placement from side effects (payment, notification, shipping). The application service orchestrates the use case without containing business logic.

### Real-World Cases

- **Banking:** Bounded contexts for accounts, payments, and loans; aggregates for `Account` and `Transaction`.
- **E-commerce:** Bounded contexts for catalog, ordering, shipping, and billing; aggregates for `Order` and `Customer`.
- **Logistics:** Bounded contexts for routing, fleet management, and tracking; aggregates for `Delivery` and `Route`.
- **Healthcare:** Bounded contexts for patient records, appointments, and billing; aggregates for `Patient` and `Prescription`.

---

## References

- Alistair Cockburn — Hexagonal Architecture — https://alistair.cockburn.us/hexagonal-architecture/
- Alistair Cockburn — Hexagonal Architecture Explained (v1.1) — https://alistaircockburn.com/hexagonal-architecture-explained
- AWS Prescriptive Guidance — Hexagonal Architecture Pattern — https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/hexagonal-architecture.html
- Robert C. Martin — The Clean Architecture (Blog) — https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- Robert C. Martin — Clean Architecture: A Craftsman's Guide to Software Structure and Design (Book) — https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/
- Microsoft Learn — Use Tactical DDD to Design Microservices — https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-domain-driven-design
- Eric Evans — Domain-Driven Design: Tackling Complexity in the Heart of Software (Book) — https://www.domainlanguage.com/ddd/
- Vlad Khononov — Learning Domain-Driven Design (Book) — https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/
- ACM — Modular Monolith: Is This the Trend in Software Architecture? — https://dl.acm.org/doi/10.1145/3643657
- SSW.Rules — Do you know the Modular Monolithic architecture? — https://www.ssw.com.au/rules/modular-monolithic-architecture/
- NestJS Documentation — Modules — https://docs.nestjs.com/modules
- Jeffrey Palermo — The Onion Architecture — https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/
- GitHub — Kashif-Kamran/clean-architecture-practice — https://github.com/Kashif-Kamran/clean-architecture-practice
- GitHub — aucun6352/project-cheat-sheet (Hexagonal Architecture TypeScript Examples) — https://github.com/aucun6352/project-cheat-sheet
- GitHub — felipfr/nestjs-nx-modular-monolith-microservices — https://github.com/felipfr/nestjs-nx-modular-monolith-microservices
- npm — @tgallacher/ddd-ts — https://www.npmjs.com/package/@tgallacher/ddd-ts
- npm — nest-hex (Hexagonal Architecture for NestJS) — https://www.npmjs.com/package/nest-hex
- npm — nodehexagen (Hexagonal Architecture Scaffolding) — https://www.npmjs.com/package/nodehexagen
- ThoughtWorks — Demystifying Software Architecture Patterns — https://www.thoughtworks.com/insights/blog/demystifying-software-architecture-patterns
- Stack Overflow — Domain Services vs Application Services — https://stackoverflow.com/questions/3839386/domain-services-vs-application-services