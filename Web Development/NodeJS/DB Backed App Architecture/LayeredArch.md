# Architectural Evolution & CQRS — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Architectural evolution in data access refers to the progression from monolithic CRUD architectures toward specialised, performance-optimised patterns — CQRS (Command Query Responsibility Segregation) for separating read and write models, read/write database splitting for routing queries to replicas, and idempotency with retry mechanisms for guaranteeing exactly-once semantics in distributed systems.

**Technical Definition:** CQRS is an architectural pattern that separates the data mutation (command) part of a system from the query part, allowing each model to be optimised independently for throughput, latency, or consistency requirements. Read/write splitting directs read traffic to read-only replicas and write traffic to primary instances via database proxies or ORM extensions. Idempotency ensures that making the same request multiple times produces the same result, typically implemented via unique request keys stored in a database or cache with retry mechanisms using exponential backoff.

**Beginner-Friendly Explanation:** Imagine a busy restaurant. In a traditional CRUD architecture, one kitchen handles both cooking new orders and answering customer questions about the menu — it gets overwhelmed. **CQRS** is like having a dedicated kitchen for cooking (commands/writes) and a separate information desk for answering questions (queries/reads). **Read/write splitting** is like having multiple information desks (read replicas) but only one kitchen (primary). **Idempotency** is like giving each customer a numbered ticket — if they ask about the same order twice, the staff checks the ticket number and gives the same answer instead of cooking the dish again.

### Key Characteristics

- **Separation of concerns:** CQRS separates read and write models, enabling independent optimisation, scaling, and security.
- **Performance asymmetry:** Read and write operations often have different performance requirements; CQRS acknowledges and exploits this asymmetry.
- **Eventual consistency:** Read models are updated asynchronously from write models, introducing eventual consistency.
- **Query routing:** Read/write splitting automatically directs queries to replicas based on the operation type.
- **Idempotency guarantees:** Unique request keys ensure state-changing operations are processed exactly once.
- **Retry safety:** Exponential backoff with jitter prevents thundering herd problems during transient failures.
- **Database-agnostic:** These patterns work with relational databases (PostgreSQL, MySQL), NoSQL databases (DynamoDB, MongoDB), and hybrid setups.

### Prerequisites

- **Understanding of CRUD limitations:** Why a single model struggles under high read/write concurrency.
- **Database fundamentals:** Primary/replica replication, transactions, ACID properties.
- **Distributed systems concepts:** Eventual consistency, CAP theorem, network partitions.
- **Message queues:** Async processing, event-driven architecture (for CQRS event projection).
- **HTTP semantics:** Idempotency-Key header, status codes (409 Conflict, 422 Unprocessable Entity).
- **ORM/query builder familiarity:** Prisma, TypeORM, Drizzle, or raw SQL.
- **Caching:** Redis or in-memory stores for idempotency key storage.

### Related Programming Areas

- **Domain-Driven Design (DDD):** CQRS is often combined with aggregates, domain events, and repositories.
- **Event Sourcing:** Commands produce events, which are projected into read models.
- **Microservices:** CQRS enables independent scaling of read and write services.
- **API Design:** Idempotency keys are part of modern REST API design (Stripe, PayPal, etc.).
- **Resilience Engineering:** Retry mechanisms with backoff are foundational to fault-tolerant systems.
- **Database Administration:** Read replica management, replication lag monitoring, failover handling.

### Core Concepts

1. **Command Query Responsibility Segregation (CQRS)** — splitting system interactions into write-heavy mutations (Commands) and lightweight read operations (Queries).
2. **Read/Write Database Splitting** — configuring database connection routers to automatically direct queries to read replicas and commands to primary clusters.
3. **Idempotency & Retry Mechanisms** — implementing unique request keys and transient failure retries to protect state-changing database writes.

---

## Core Concept 1: Command Query Responsibility Segregation (CQRS)

### Definitions

**Core Definition:** CQRS (Command Query Responsibility Segregation) is an architectural pattern that separates the data mutation (command) part of a system from the query part, using different models for reading and writing data.

**Technical Definition:** The CQRS pattern separates the data mutation, or the command part of a system, from the query part. You can use the CQRS pattern to separate updates and queries if they have different requirements for throughput, latency, or consistency. Commands update data and should represent specific business tasks (e.g., "Book hotel room" instead of "Set ReservationStatus to Reserved"). Queries never alter data; they return Data Transfer Objects (DTOs) that present the required data in a convenient format, without any domain logic.

**Beginner-Friendly Explanation:** In a traditional application, the same model handles both reading and writing. For example, the `User` model is used to create users, update their profiles, and retrieve their data for display. CQRS says: "Use one model for writing (Commands) and a different model for reading (Queries)." The write model enforces business rules and invariants. The read model is optimised for fast retrieval — it might be a denormalised view stored in a separate database. This separation lets you scale reads and writes independently and optimise each for its specific purpose.

### Purposes

- To optimise read and write operations independently, since they have different performance and scaling requirements.
- To avoid lock contention and performance problems caused by parallel read/write operations on the same data set.
- To improve security by separating read and write concerns, preventing data exposure in unintended contexts.
- To enable asynchronous command processing, improving responsiveness and throughput.
- To support event-driven architectures where commands produce events that update read models.
- To allow different data stores for commands (e.g., relational database) and queries (e.g., NoSQL or search index).

### Syntax Rules and Structure

#### General Architecture (TypeScript/NestJS)

```typescript
// --- COMMAND SIDE ---
// Command: represents a specific business task
export class CreateOrderCommand {
  constructor(
    public readonly userId: string,
    public readonly items: ReadonlyArray<OrderItemDto>,
  ) {}
}

// Command Handler: processes the command
@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateOrderCommand): Promise<string> {
    const order = Order.create(command.userId, command.items);
    await this.orderRepository.save(order);
    this.eventBus.publish(new OrderCreatedEvent(order.id, order.total));
    return order.id;
  }
}

// --- QUERY SIDE ---
// Query: represents a read operation
export class GetOrderQuery {
  constructor(public readonly orderId: string) {}
}

// Query Handler: processes the query
@QueryHandler(GetOrderQuery)
export class GetOrderHandler implements IQueryHandler<GetOrderQuery> {
  constructor(private readonly readRepository: OrderReadRepository) {}

  async execute(query: GetOrderQuery): Promise<OrderReadDto> {
    return this.readRepository.findById(query.orderId);
  }
}

// --- CONTROLLER ---
@Controller('orders')
export class OrdersController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  @Post()
  async create(@Body() dto: CreateOrderDto): Promise<{ id: string }> {
    const command = new CreateOrderCommand(dto.userId, dto.items);
    const id = await this.commandBus.execute(command);
    return { id };
  }

  @Get(':id')
  async findById(@Param('id') id: string): Promise<OrderReadDto> {
    return this.queryBus.execute(new GetOrderQuery(id));
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `Command` | Immutable DTO representing a write intent. |
| `CommandHandler` | Processes the command, enforces business rules, persists. |
| `Query` | Immutable DTO representing a read request. |
| `QueryHandler` | Retrieves data from the read model; never mutates state. |
| `CommandBus` / `QueryBus` | Dispatch mechanisms that route to the appropriate handler. |
| `EventBus` | Publishes domain events for read-model projection. |

#### Syntax Rules

- **Commands must be imperative** — name them after business tasks (`BookHotelRoom`, `CancelOrder`), not low-level data operations (`SetStatus`).
- **Queries must never mutate state** — they return DTOs and contain no domain logic.
- **Commands return void or an ID** — not the full entity. The read side is responsible for retrieval.
- **Read and write models may share the same database initially** — CQRS does not require separate databases; it requires separate models.
- **Eventual consistency must be acknowledged** — after a command, the read model may not immediately reflect the change.
- **Handlers should be single-responsibility** — one handler per command or query.

#### Constraints and Limitations

- **Increased complexity:** CQRS introduces more classes (commands, handlers, queries) and infrastructure (buses, event projectors).
- **Eventual consistency:** Read models are updated asynchronously; clients may see stale data immediately after a write.
- **Not for simple CRUD:** For applications where read and write patterns are identical, CQRS adds unnecessary complexity.
- **Debugging difficulty:** Tracing a request across command bus, event bus, and read-model projection is harder than a single method call.
- **Duplicate data:** Read models often denormalise data, leading to storage overhead and consistency challenges.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: CQRS with NestJS — Commands, Queries, and Event Projection

**Step-by-step setup:**

1. Install the NestJS CQRS module: `npm install @nestjs/cqrs`.
2. Define the write model (`Order` entity).
3. Define commands and command handlers.
4. Define queries and query handlers.
5. Define events and event handlers (for read-model projection).
6. Wire everything in a module.

```typescript
// orders/commands/create-order.command.ts
export class CreateOrderCommand {
  constructor(
    public readonly userId: string,
    public readonly items: ReadonlyArray<{ productId: string; quantity: number }>,
  ) {}
}

// orders/commands/create-order.handler.ts
@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateOrderCommand): Promise<string> {
    // Business rules enforced here
    const order = Order.create(command.userId, command.items);
    await this.orderRepository.save(order);

    // Publish event for read-model projection
    this.eventBus.publish(new OrderCreatedEvent(order.id, order.userId, order.total));
    return order.id;
  }
}
```

```typescript
// orders/queries/get-order.query.ts
export class GetOrderQuery {
  constructor(public readonly orderId: string) {}
}

// orders/queries/get-order.handler.ts
@QueryHandler(GetOrderQuery)
export class GetOrderHandler implements IQueryHandler<GetOrderQuery> {
  constructor(
    @Inject('ORDER_READ_REPOSITORY')
    private readonly readRepository: OrderReadRepository,
  ) {}

  async execute(query: GetOrderQuery): Promise<OrderReadDto> {
    return this.readRepository.findById(query.orderId);
  }
}
```

```typescript
// orders/events/order-created.event.ts
export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
  ) {}
}

// orders/events/order-created.handler.ts — projects to read model
@EventsHandler(OrderCreatedEvent)
export class OrderCreatedHandler implements IEventHandler<OrderCreatedEvent> {
  constructor(
    @Inject('ORDER_READ_REPOSITORY')
    private readonly readRepository: OrderReadRepository,
  ) {}

  async handle(event: OrderCreatedEvent): Promise<void> {
    // Denormalised read model
    await this.readRepository.save({
      id: event.orderId,
      userId: event.userId,
      total: event.total,
      status: 'created',
      createdAt: new Date(),
    });
  }
}
```

```typescript
// orders/orders.module.ts
@Module({
  imports: [CqrsModule],
  providers: [
    CreateOrderHandler,
    GetOrderHandler,
    OrderCreatedHandler,
    OrderRepository,
    { provide: 'ORDER_READ_REPOSITORY', useClass: OrderReadRepository },
  ],
  controllers: [OrdersController],
})
export class OrdersModule {}
```

**Expected behaviour:**
- `POST /orders` dispatches a `CreateOrderCommand` → `CreateOrderHandler` creates the order, saves it to the write database, and publishes `OrderCreatedEvent`.
- `OrderCreatedHandler` asynchronously projects the event into the read database.
- `GET /orders/:id` dispatches a `GetOrderQuery` → `GetOrderHandler` retrieves the denormalised read model.
- Immediately after creation, the read model may not yet contain the order (eventual consistency).

**Why this works:** The command side enforces business rules and writes to the primary database. The query side reads from an optimised read model. The event bus decouples the two sides, allowing the read model to be updated asynchronously without blocking the command.

### Real-World Cases

- **E-commerce:** Write side handles order placement and inventory updates; read side provides product listings and order history.
- **Banking:** Write side processes transactions; read side provides account statements and analytics.
- **IoT platforms:** Write side ingests sensor data; read side provides real-time dashboards.
- **Social media:** Write side handles posts and likes; read side serves feeds and search results.

---

## Core Concept 2: Read/Write Database Splitting

### Definitions

**Core Definition:** Read/write database splitting is a technique that automatically routes read (query) operations to read-only database replicas and write (command) operations to the primary database instance, improving read scalability and reducing load on the primary.

**Technical Definition:** Read/write splitting enables you to direct all read traffic to read-only instances, and all write traffic to read-write instances. Read-write instances are primaries or sources; read-only instances are secondaries in an InnoDB Cluster or the primary or secondary instances in a Replica Cluster. Database proxies or ORM extensions classify each query as read or write and direct it to the appropriate backend. In Prisma, the `@prisma/extension-read-replicas` extension creates additional Prisma Clients for read replica connection strings and routes read queries to these clients instead of the primary Prisma Client.

**Beginner-Friendly Explanation:** Imagine a library with one main desk where you can borrow and return books, plus several reading rooms where you can only read. Read/write splitting is like a sign that says: "If you want to borrow or return a book, go to the main desk. If you just want to read, go to the reading rooms." The main desk (primary database) handles all changes. The reading rooms (replicas) handle all queries, so the main desk doesn't get overwhelmed.

### Purposes

- To scale read capacity horizontally by adding read replicas.
- To reduce load on the primary database, allowing it to focus on writes.
- To improve query performance by using replicas dedicated to reads.
- To increase availability — if the primary fails, replicas can be promoted.
- To enable geographic distribution — replicas can be placed closer to users.
- To route reads to replicas only when replication lag is within an acceptable threshold, ensuring consistency where needed.

### Syntax Rules and Structure

#### Prisma Read Replicas Extension

```typescript
import { PrismaClient } from '@prisma/client';
import { readReplicas } from '@prisma/extension-read-replicas';

const prisma = new PrismaClient().$extends(
  readReplicas({
    url: [
      'postgresql://replica-1.example.com:5432/db',
      'postgresql://replica-2.example.com:5432/db',
    ],
  }),
);

// Reads automatically go to a random replica
const users = await prisma.user.findMany();

// Writes and transactions go to the primary
await prisma.user.create({ data: { name: 'Alice' } });

// Force a read from the primary (for read-after-write consistency)
const user = await prisma.$primary().user.findUnique({ where: { id: '123' } });

// Force a raw query to go through a replica
const result = await prisma.$replica().$queryRaw`SELECT * FROM users`;
```

| Component | Breakdown |
|-----------|-----------|
| `readReplicas({ url })` | Extension that adds replica routing. |
| `$primary()` | Forces a read to the primary server. |
| `$replica()` | Forces a read to a replica. |
| Automatic routing | Non-transactional reads → replica; writes/transactions → primary. |
| `queryRaw`/`executeRaw` | Always go to primary by default (extension cannot determine read/write). |

#### TypeORM Read/Write Splitting (Manual Configuration)

```typescript
const dataSource = new DataSource({
  type: 'postgres',
  replication: {
    master: {
      host: 'primary.example.com',
      port: 5432,
      username: 'app',
      password: 'secret',
      database: 'app',
    },
    slaves: [
      { host: 'replica-1.example.com', port: 5432, username: 'app', password: 'secret', database: 'app' },
      { host: 'replica-2.example.com', port: 5432, username: 'app', password: 'secret', database: 'app' },
    ],
  },
});
```

| Component | Breakdown |
|-----------|-----------|
| `replication.master` | Primary database connection. |
| `replication.slaves` | Array of read replica connections. |
| Automatic routing | TypeORM routes reads to slaves and writes to master. |

#### Syntax Rules

- **Reads must be routed to replicas only when eventual consistency is acceptable** — after a write, a read from a replica may return stale data.
- **Use `$primary()` for read-after-write consistency** — when the client must see its own write immediately.
- **Transactions always go to the primary** — replicas are read-only.
- **Raw queries default to the primary** — the routing extension cannot determine if a raw query reads or writes.
- **Monitor replication lag** — route reads to replicas only when lag is below a threshold.
- **Handle replica failures gracefully** — fall back to the primary if all replicas are unavailable.

#### Constraints and Limitations

- **Eventual consistency:** Replicas may lag behind the primary by milliseconds to seconds.
- **Replica failure:** If all replicas are down, queries must fall back to the primary, potentially overloading it.
- **Connection overhead:** Each replica requires its own connection pool.
- **Write-after-read consistency:** A write followed immediately by a read from a replica may return stale data.
- **Raw queries:** The extension cannot determine whether a raw query reads or writes, so it defaults to the primary.
- **DNS/autoscaling:** TypeORM may not re-resolve replica endpoints after autoscaling adds new replicas.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Prisma Read Replicas with Lag-Aware Routing

```typescript
// db.ts
import { PrismaClient } from '@prisma/client';
import { readReplicas } from '@prisma/extension-read-replicas';

const PRIMARY_URL = process.env.DATABASE_URL!;
const REPLICA_URLS = process.env.REPLICA_URLS!.split(',');

export const prisma = new PrismaClient({
  datasources: { db: { url: PRIMARY_URL } },
}).$extends(
  readReplicas({
    url: REPLICA_URLS,
  }),
);

// --- Application service using the extended client ---
export class UserService {
  // Read — goes to a replica automatically
  async findById(id: string) {
    return prisma.user.findUnique({ where: { id } });
  }

  // Write — goes to the primary automatically
  async create(data: { name: string; email: string }) {
    return prisma.user.create({ data });
  }

  // Read-after-write — force primary to avoid stale data
  async findAfterCreate(id: string) {
    return prisma.$primary().user.findUnique({ where: { id } });
  }
}
```

**Step-by-step setup:**
1. Set `DATABASE_URL` (primary) and `REPLICA_URLS` (comma-separated replica URLs) as environment variables.
2. Instantiate `PrismaClient` with the primary URL.
3. Apply the `readReplicas` extension with the replica URLs.
4. Use the extended client normally — reads automatically route to replicas.
5. Use `$primary()` when read-after-write consistency is required.

**Expected behaviour:**
- `findById()` executes on a replica (random selection if multiple).
- `create()` executes on the primary.
- `findAfterCreate()` executes on the primary, guaranteeing the write is visible.

**Why this works:** The extension intercepts Prisma operations and inspects the query type. Non-transactional reads are routed to replicas; writes and transactions go to the primary. Explicit `$primary()` overrides the default routing.

#### Example 2: TypeORM Replication Configuration

```typescript
// data-source.ts
import { DataSource } from 'typeorm';

export const AppDataSource = new DataSource({
  type: 'postgres',
  replication: {
    master: {
      host: process.env.DB_PRIMARY_HOST,
      port: 5432,
      username: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
    },
    slaves: [
      {
        host: process.env.DB_REPLICA_1_HOST,
        port: 5432,
        username: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
      {
        host: process.env.DB_REPLICA_2_HOST,
        port: 5432,
        username: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
    ],
  },
  entities: [User, Order, Product],
  synchronize: false,
  logging: ['query', 'error'],
});
```

**Expected behaviour:** TypeORM automatically routes `find*` operations to the replicas and `save`/`insert`/`update`/`delete` operations to the master. Transactions always use the master.

**Why this works:** TypeORM's built-in replication configuration inspects the operation type and selects the appropriate connection. This is transparent to the application code.

### Real-World Cases

- **High-traffic APIs:** Read-heavy endpoints (product listings, user profiles) are served by replicas.
- **Analytics dashboards:** Reporting queries run against replicas to avoid impacting transactional performance.
- **Global applications:** Replicas are placed in multiple regions for low-latency reads.
- **Failover:** If the primary fails, a replica is promoted, minimising downtime.

---

## Core Concept 3: Idempotency & Retry Mechanisms

### Definitions

**Core Definition:** Idempotency ensures that making the same request multiple times produces the same result, protecting state-changing operations from duplicate execution during retries. Retry mechanisms automatically re-attempt transient failures using exponential backoff.

**Technical Definition:** Idempotency keys are built on a correlation ID to make retries safe. When a service receives a request, it derives an idempotency key from the correlation ID and a service-specific qualifier (e.g., `corr-7A3F:payment`). The service stores this key in its database before starting processing. If a retry arrives with the same correlation ID, the service reconstructs the same idempotency key, finds it in the database, and skips reprocessing. Retry mechanisms should only retry idempotent operations, and most catch-all exceptions do not guarantee that all effects from the failed invocation are undone. Exponential backoff with jitter is the recommended retry strategy.

**Beginner-Friendly Explanation:** Imagine you're paying for groceries with a credit card. If the card reader times out, you might swipe again. Idempotency ensures that even if you swipe twice, you're only charged once. The grocery store's system recognises the second swipe as the same transaction and doesn't charge you again. Retry mechanisms are like the store automatically trying your card again if the network is temporarily down — but only if the transaction is idempotent (safe to retry).

### Purposes

- To guarantee exactly-once semantics for state-changing operations in distributed systems.
- To protect against duplicate processing caused by network retries, timeouts, or client errors.
- To enable safe retries of transient failures without risking data corruption.
- To provide a consistent, predictable API contract for clients — the same request always returns the same response.
- To prevent double charges, duplicate orders, and other financial inconsistencies.
- To improve system resilience by automatically recovering from transient network or database failures.

### Syntax Rules and Structure

#### Idempotency Key Middleware (Express)

```typescript
import express from 'express';
import { idempotent } from '@idempotix/express';
import { redis } from '@idempotix/redis';

const app = express();
app.use(express.json());

app.post(
  '/orders',
  idempotent({
    storage: redis(),
    ttl: '1h',
    required: true,
    methods: ['POST', 'PUT'],
    failOpen: true,
  }),
  async (req, res) => {
    const order = await createOrder(req.body);
    res.status(201).json(order);
  },
);
```

| Component | Breakdown |
|-----------|-----------|
| `idempotent()` | Middleware that enforces idempotency. |
| `storage` | Backend for storing keys and responses (Redis, memory, database). |
| `ttl` | How long to cache responses. |
| `required` | Whether the `Idempotency-Key` header is mandatory. |
| `methods` | HTTP methods to protect. |
| `failOpen` | Whether to continue if storage fails. |
| `Idempotency-Key` | Client-provided unique key (request header). |
| `Idempotency-Replay` | Response header indicating a cached response was returned. |

#### Idempotency Key Database Schema

```sql
CREATE TABLE idempotency_keys (
  key VARCHAR(255) PRIMARY KEY,
  request_hash VARCHAR(64) NOT NULL,
  response_status INTEGER,
  response_body JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_idempotency_expires ON idempotency_keys (expires_at);
```

| Column | Purpose |
|--------|---------|
| `key` | The client-provided idempotency key (unique). |
| `request_hash` | Hash of the request body to detect key reuse with different payloads. |
| `response_status` | HTTP status code of the original response. |
| `response_body` | The original response body (for replay). |
| `expires_at` | Automatic cleanup after TTL. |

#### Exponential Backoff with Jitter

```typescript
async function withRetry<T>(
  operation: () => Promise<T>,
  options: { maxAttempts: number; baseDelayMs: number; maxDelayMs: number },
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (err) {
      lastError = err as Error;
      if (!isTransient(err)) throw err;

      const exponentialDelay = Math.min(
        options.baseDelayMs * Math.pow(2, attempt - 1),
        options.maxDelayMs,
      );
      const jitter = Math.random() * exponentialDelay * 0.1;
      const delay = exponentialDelay + jitter;

      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }

  throw lastError!;
}

function isTransient(err: unknown): boolean {
  if (err instanceof Error) {
    return ['ECONNRESET', 'ETIMEDOUT', 'ER_LOCK_DEADLOCK', '40001'].some(
      (code) => err.message.includes(code),
    );
  }
  return false;
}
```

| Component | Breakdown |
|-----------|-----------|
| `maxAttempts` | Maximum number of retry attempts. |
| `baseDelayMs` | Initial delay before the first retry. |
| `maxDelayMs` | Upper bound on the delay. |
| `isTransient()` | Classifies errors as retryable (deadlocks, timeouts) or fatal. |
| `jitter` | Random variation to prevent thundering herd. |

#### Syntax Rules

- **Idempotency keys must be client-generated** — the client is responsible for providing a unique key per logical request.
- **Keys must be stored durably** — Redis or a database table with a unique constraint.
- **The same key with a different request body must be rejected** — return `422 Unprocessable Entity` (mismatch).
- **Concurrent requests with the same key must be handled** — return `409 Conflict` if a request is already in progress.
- **Responses must be cached with the key** — so retries return the original response.
- **Retry only transient errors** — deadlocks, connection resets, and timeouts. Do not retry validation errors or business rule violations.
- **Use exponential backoff with jitter** — prevents all clients from retrying simultaneously.
- **Set a maximum retry count** — avoid infinite retry loops.

#### Constraints and Limitations

- **Storage overhead:** Idempotency keys and responses require storage and cleanup.
- **Key expiry:** Keys must expire to avoid unbounded growth; however, expired keys allow duplicate requests.
- **Distributed coordination:** In a multi-instance deployment, idempotency storage must be shared (Redis, database).
- **Request body hashing:** Detecting key reuse with different payloads requires hashing the body.
- **Retry amplification:** Aggressive retries can amplify load during outages.
- **Non-idempotent side effects:** Some operations (sending emails, charging cards) cannot be made idempotent without external coordination.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Idempotent Payment Endpoint with Redis and Database Constraint

```typescript
// payment.controller.ts
import { Controller, Post, Body, Headers, HttpException, HttpStatus } from '@nestjs/common';
import { PaymentService } from './payment.service';

@Controller('payments')
export class PaymentController {
  constructor(private readonly paymentService: PaymentService) {}

  @Post()
  async processPayment(
    @Body() dto: ProcessPaymentDto,
    @Headers('idempotency-key') idempotencyKey: string,
  ): Promise<PaymentResultDto> {
    if (!idempotencyKey) {
      throw new HttpException('Idempotency-Key header is required', HttpStatus.BAD_REQUEST);
    }

    return this.paymentService.process(dto, idempotencyKey);
  }
}
```

```typescript
// payment.service.ts
import { Injectable, ConflictException, UnprocessableEntityException } from '@nestjs/common';
import { Redis } from 'ioredis';
import { createHash } from 'node:crypto';
import { PrismaService } from './prisma.service';

@Injectable()
export class PaymentService {
  constructor(
    private readonly redis: Redis,
    private readonly prisma: PrismaService,
  ) {}

  async process(dto: ProcessPaymentDto, idempotencyKey: string): Promise<PaymentResultDto> {
    const requestHash = createHash('sha256').update(JSON.stringify(dto)).digest('hex');
    const cacheKey = `idempotency:payment:${idempotencyKey}`;

    // 1. Check for an existing in-flight or completed request
    const existing = await this.redis.get(cacheKey);

    if (existing) {
      const cached = JSON.parse(existing);
      if (cached.requestHash !== requestHash) {
        throw new UnprocessableEntityException('Idempotency key reused with different payload');
      }
      return cached.response;
    }

    // 2. Mark as in-flight (SET NX with 30s TTL)
    const lockKey = `${cacheKey}:lock`;
    const acquired = await this.redis.set(lockKey, '1', 'PX', 30000, 'NX');

    if (!acquired) {
      throw new ConflictException('A request with this idempotency key is already being processed');
    }

    try {
      // 3. Execute the payment
      const result = await this.executePayment(dto);

      // 4. Cache the response
      await this.redis.set(
        cacheKey,
        JSON.stringify({ requestHash, response: result }),
        'EX',
        86400, // 24 hours
      );

      return result;
    } finally {
      await this.redis.del(lockKey);
    }
  }

  private async executePayment(dto: ProcessPaymentDto): Promise<PaymentResultDto> {
    return this.prisma.$transaction(async (tx) => {
      // Database unique constraint on idempotency_key provides a safety net
      const existing = await tx.payment.findUnique({
        where: { idempotencyKey: dto.idempotencyKey },
      });
      if (existing) return existing;

      return tx.payment.create({
        data: {
          idempotencyKey: dto.idempotencyKey,
          amount: dto.amount,
          currency: dto.currency,
          status: 'completed',
        },
      });
    });
  }
}
```

**Step-by-step setup:**
1. Client sends `POST /payments` with an `Idempotency-Key` header and payment details.
2. Controller extracts the key and passes it to the service.
3. Service computes a SHA-256 hash of the request body.
4. Service checks Redis for an existing cached response.
5. If not cached, service acquires a Redis lock (SET NX with TTL) to prevent concurrent processing.
6. Service executes the payment inside a database transaction with a unique constraint on `idempotencyKey`.
7. Service caches the response in Redis for 24 hours.
8. On retry with the same key, the cached response is returned immediately.

**Expected behaviour:**
- First request: payment is processed, response is cached.
- Second request with same key and same body: cached response is returned, payment is not reprocessed.
- Second request with same key but different body: `422 Unprocessable Entity`.
- Concurrent request with same key: `409 Conflict`.

**Why this works:** Redis provides fast duplicate detection. The lock prevents race conditions. The database unique constraint provides a durable safety net. The request hash detects key misuse with different payloads.

#### Example 2: Retry with Exponential Backoff for Transient Database Errors

```typescript
// retry.ts
export interface RetryOptions {
  maxAttempts: number;
  baseDelayMs: number;
  maxDelayMs: number;
  onRetry?: (attempt: number, error: Error) => void;
}

export async function withRetry<T>(
  operation: () => Promise<T>,
  options: RetryOptions,
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (err) {
      lastError = err as Error;

      if (!isTransient(err) || attempt === options.maxAttempts) {
        throw err;
      }

      const exponential = Math.min(
        options.baseDelayMs * 2 ** (attempt - 1),
        options.maxDelayMs,
      );
      const jitter = Math.random() * exponential * 0.2;
      const delayMs = exponential + jitter;

      options.onRetry?.(attempt, lastError);
      await new Promise((resolve) => setTimeout(resolve, delayMs));
    }
  }

  throw lastError!;
}

function isTransient(err: unknown): boolean {
  if (!(err instanceof Error)) return false;

  const transientCodes = [
    'ECONNRESET',
    'ETIMEDOUT',
    'ECONNREFUSED',
    'ER_LOCK_DEADLOCK',
    'ER_LOCK_WAIT_TIMEOUT',
    '40001', // PostgreSQL serialization failure
    '40P01', // PostgreSQL deadlock detected
  ];

  return transientCodes.some((code) => err.message.includes(code));
}
```

```typescript
// order.service.ts — using withRetry for a state-changing write
@Injectable()
export class OrderService {
  async createOrder(dto: CreateOrderDto, idempotencyKey: string): Promise<OrderDto> {
    return withRetry(
      () => this.unitOfWork.transaction(async (manager) => {
        // Check idempotency inside the transaction
        const existing = await manager.findOne(Order, {
          where: { idempotencyKey },
        });
        if (existing) return OrderMapper.toDto(existing);

        const order = Order.create(dto);
        await manager.save(order);
        return OrderMapper.toDto(order);
      }),
      {
        maxAttempts: 3,
        baseDelayMs: 100,
        maxDelayMs: 2000,
        onRetry: (attempt, err) =>
          logger.warn(`Retry ${attempt} for order creation: ${err.message}`),
      },
    );
  }
}
```

**Expected behaviour:**
- First attempt fails with `ER_LOCK_DEADLOCK` (transient).
- `withRetry` waits ~100ms + jitter, then retries.
- Second attempt fails with `ER_LOCK_DEADLOCK`.
- `withRetry` waits ~200ms + jitter, then retries.
- Third attempt succeeds — the order is created.
- If the third attempt also fails, the error is thrown to the caller.

**Why this works:** The retry mechanism only retries transient errors (deadlocks, serialization failures). The exponential backoff with jitter prevents all clients from retrying simultaneously. The idempotency key inside the transaction ensures that even if a retry happens after a successful commit, the duplicate is detected and the existing order is returned.

### Real-World Cases

- **Payment processing:** Stripe, PayPal, and Adyen all require idempotency keys for POST requests to prevent double charges.
- **Order management:** E-commerce platforms use idempotency keys to prevent duplicate orders when customers double-click or retry.
- **Webhook delivery:** Webhook senders retry failed deliveries; receivers must be idempotent to avoid processing the same event twice.
- **Database migrations:** Retryable transactions with idempotent operations ensure migrations can be safely re-run.
- **Microservice communication:** Idempotency keys propagate through service chains, ensuring each service processes each request exactly once.

---

## References

- Microsoft Learn — CQRS Pattern (Azure Architecture Center) — https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs
- AWS Prescriptive Guidance — CQRS Pattern — https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html
- NestJS Documentation — CQRS Module — https://docs.nestjs.com/recipes/cqrs
- Oracle MySQL Router Documentation — Read/Write Splitting — https://docs.oracle.com/cd/E17952_01/mysql-router-9.7-en/router-read-write-splitting.html
- Prisma Documentation — Read Replicas Extension — https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/read-replicas
- Microsoft Learn — Idempotency Keys for Safe Retries (Azure Architecture Center) — https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/microservices/evaluation-preparation
- RFC 7231 — HTTP/1.1 Semantics and Content (Idempotent Methods) — https://www.rfc-editor.org/rfc/rfc7231
- IETF Draft — The Idempotency-Key HTTP Header Field — https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/
- Stripe API Documentation — Idempotent Requests — https://docs.stripe.com/api/idempotent_requests
- AWS Prescriptive Guidance — Retry with Exponential Backoff — https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html
- TypeORM Documentation — Replication — https://typeorm.io/docs/advanced-topics/replication/
- Martin Fowler — CQRS (Bliki) — https://martinfowler.com/bliki/CQRS.html
- Microsoft Learn — Retry Pattern (Azure Architecture Center) — https://learn.microsoft.com/en-us/azure/architecture/patterns/retry
- Redis Documentation — SET with NX and EX Options — https://redis.io/commands/set/
- PostgreSQL Documentation — Transaction Isolation and Serialization Failures — https://www.postgresql.org/docs/current/transaction-iso.html