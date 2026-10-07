# Application-Level Data Access: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Application-level data access is the architectural discipline of structuring how an application's code interacts with persistent storage, encompassing the patterns, layers, boundaries, and resilience mechanisms that mediate between business logic and database systems.

**Technical Definition**: Application-level data access comprises the repository pattern (abstracting persistence behind domain-facing interfaces), data access layer (DAL) boundaries (separating data retrieval from business logic), CQRS (Command Query Responsibility Segregation, separating read and write models), service layers (orchestrating business operations across domain objects), transaction boundaries (declarative propagation and distributed transaction handling), connection lifecycle management (scoped sessions and OSIV avoidance), error translation (mapping vendor SQL states to application exceptions), and resilience patterns (circuit breakers, retry with backoff, and fail-fast architectures).

**Beginner-Friendly Explanation**: Think of your application as a restaurant. The kitchen (business logic) doesn't go to the farm (database) to get ingredients. Instead, a supplier (repository) delivers what's needed. A head chef (service layer) coordinates multiple cooks (domain objects). The restaurant manager (transaction boundary) ensures that either everything for an order is ready or nothing is served. And when the supplier is having problems, the restaurant has backup plans (resilience patterns) to keep serving customers.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Abstraction** | Domain layer depends on interfaces, not database implementations |
| **Separation** | Read and write paths can be optimized independently (CQRS) |
| **Orchestration** | Service layer coordinates multi-entity operations and side effects |
| **Transactionality** | Declarative boundaries with propagation semantics |
| **Resilience** | Circuit breakers, retries, and fail-fast prevent cascading failures |
| **Testability** | Repository interfaces enable in-memory fakes for unit testing |

### Prerequisites

- **Domain Model**: Entities and value objects that represent business concepts
- **Persistence Framework**: JPA/Hibernate, Entity Framework, SQLAlchemy, or equivalent
- **Dependency Injection Container**: Spring, .NET DI, or equivalent for wiring repositories
- **Transaction Manager**: JTA, Spring `PlatformTransactionManager`, or framework-managed transactions
- **Resilience Library**: Resilience4j, Polly, pybreaker, or equivalent

### Related Programming Areas

- **Domain-Driven Design**: Aggregates, repositories, domain services, bounded contexts
- **Microservices Architecture**: Saga pattern, distributed transactions, service choreography
- **Performance Engineering**: Connection pooling, query optimization, caching
- **Security Engineering**: Transaction isolation, SQL injection prevention, credential management
- **Site Reliability Engineering**: Circuit breakers, retry budgets, failure isolation

### Core Concepts Overview

Application-level data access comprises seven complementary domains:

1. **Repository Patterns**: Abstraction, interface segregation, query objects
2. **Data-Access Layers**: DAL boundaries, CQRS separation of read/write paths
3. **Service Layers**: Business logic encapsulation and orchestration
4. **Transaction Boundaries**: Declarative propagation and Saga pattern foundations
5. **Connection Lifecycle**: Open-session-in-view pitfalls and scoped management
6. **Error Handling**: Vendor SQL state translation and deadlock handling
7. **Application Resilience**: Circuit breakers, exponential backoff with jitter, fail-fast

---

## Core Concept 1: Repository Patterns

### Definitions

**Core Definition**: The repository pattern mediates between the domain and data mapping layers, acting like an in-memory domain object collection.

**Technical Definition**: A Repository encapsulates the data store and the operations performed over it, providing a more object-oriented view of the persistence layer. It supports achieving a clean separation and one-way dependency between the domain and data mapping layers. The pattern puts one object in charge of persistence for an entity, providing a domain layer that depends on an interface rather than on SQLAlchemy or any specific ORM. Query logic resides in one place, and implementations can be swapped (change ORM, add caching, split reads and writes) without touching callers.

**Beginner-Friendly Explanation**: A repository is like a librarian for your data. Your application code (the reader) doesn't go into the stacks (database) directly. Instead, the librarian (repository) knows exactly where everything is and fetches what you need. If the library reorganizes its shelves (changes the ORM), the reader never notices because the librarian's interface stays the same.

### Purposes

- **To** abstract data persistence behind a domain-focused interface
- **To** enable unit testing without a database by swapping in in-memory fakes
- **To** centralize query logic and eliminate query duplication across services
- **To** allow swapping ORM implementations, adding caching, or splitting read/write paths without modifying callers

### Syntax Rules and Structure

#### Repository Interface (Domain Layer)

```java
// Domain layer depends on this interface — never on the ORM
public interface UserRepository {
    Optional<User> findById(Long id);
    List<User> findByEmailDomain(String domain);
    User save(User user);
    void delete(User user);
}
```

#### Repository Implementation (Infrastructure Layer)

```java
// Infrastructure layer implements the interface using JPA
@Repository
public class JpaUserRepository implements UserRepository {
    @PersistenceContext
    private EntityManager em;

    @Override
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(em.find(User.class, id));
    }

    @Override
    public List<User> findByEmailDomain(String domain) {
        return em.createQuery(
            "SELECT u FROM User u WHERE u.email LIKE :domain", User.class)
            .setParameter("domain", "%@" + domain)
            .getResultList();
    }

    @Override
    public User save(User user) {
        if (user.getId() == null) {
            em.persist(user);
            return user;
        }
        return em.merge(user);
    }

    @Override
    public void delete(User user) {
        em.remove(em.contains(user) ? user : em.merge(user));
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `UserRepository` | Domain-facing interface defining persistence operations |
| `JpaUserRepository` | Infrastructure implementation using JPA |
| `Optional<User>` | Null-safe return type for single-entity queries |
| `findByEmailDomain()` | Query method encapsulating filtering logic |

#### Syntax Rules

- The domain layer must define the repository interface; the infrastructure layer implements it.
- Repository methods return domain objects, not database rows or ORM-specific types.
- Query objects encapsulate complex query logic and can be passed to repository methods.
- Interface segregation: keep repository interfaces narrow and focused on domain needs.

#### Constraints and Limitations

- The repository pattern is not meant to be an abstraction of the ORM framework itself; it provides a collection-like interface to any kind of data.
- Over-abstraction can lead to generic repositories with `findAll()` methods that expose database internals.
- Repositories should not contain business logic — that belongs in the service layer.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Repository with Query Object

```java
// Step 1: Define a query object for complex filtering
public class UserQuery {
    private String namePattern;
    private Integer minAge;
    private String status;
    // getters and builder methods
}

// Step 2: Define repository interface with query object
public interface UserRepository {
    List<User> find(UserQuery query);
    Optional<User> findById(Long id);
    User save(User user);
}

// Step 3: Implement with JPA Criteria API
@Repository
public class JpaUserRepository implements UserRepository {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<User> find(UserQuery query) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> cq = cb.createQuery(User.class);
        Root<User> root = cq.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        if (query.getNamePattern() != null) {
            predicates.add(cb.like(root.get("name"), query.getNamePattern()));
        }
        if (query.getMinAge() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("age"), query.getMinAge()));
        }
        if (query.getStatus() != null) {
            predicates.add(cb.equal(root.get("status"), query.getStatus()));
        }
        
        cq.where(predicates.toArray(new Predicate[0]));
        return em.createQuery(cq).getResultList();
    }
    // other methods...
}
```

**Expected Output**: The repository returns a filtered list of `User` entities matching the query object's criteria, with all filtering logic encapsulated in the repository implementation.

**Why This Output Occurs**: The query object separates the *what* (filter criteria) from the *how* (JPA Criteria API implementation). Callers construct a `UserQuery` and pass it to the repository, which translates it into database-specific query logic. This keeps query logic in one place and testable in isolation.

### Real-World Cases

**Case 1: Swapping ORM Without Caller Changes**: A team migrates from Hibernate to jOOQ. Because the domain layer depends on the `UserRepository` interface, only the `JpaUserRepository` implementation is replaced — service and controller code remain unchanged.

**Case 2: In-Memory Testing**: Unit tests for `UserService` use `InMemoryUserRepository` (a `HashMap`-backed fake) instead of a real database, running in milliseconds without database setup.

**Case 3: Read/Write Splitting**: A repository interface has separate implementations for read (routing to replicas) and write (routing to primary) operations, transparent to the service layer.

---

## Core Concept 2: Data-Access Layers (DAL) and CQRS

### Definitions

**Core Definition**: A data-access layer (DAL) is a logical boundary that isolates data retrieval and persistence operations from business logic. CQRS extends this by segregating read and write models into separate data stores or schemas.

**Technical Definition**: CQRS (Command Query Responsibility Segregation) is a pattern that segregates the operations that read data (Queries) from the operations that update data (Commands) by using separate interfaces. CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores. The pattern addresses the asymmetry between read and write workloads: data mismatch, lock contention, performance problems, and security challenges that arise when a single model serves both purposes.

**Beginner-Friendly Explanation**: Imagine a library with separate desks for borrowing books (writes) and reading books (reads). The borrowing desk needs to track inventory and update records. The reading desk only needs comfortable chairs and good lighting. CQRS says: don't make one desk do both jobs — give each its own optimized setup.

### Purposes

- **To** optimize read and write models independently for their specific access patterns
- **To** eliminate lock contention between read-heavy and write-heavy workloads
- **To** enable independent scaling of read and write paths (e.g., read replicas for queries)
- **To** simplify security by applying different access controls to commands and queries

### Syntax Rules and Structure

#### CQRS with Separate Read and Write Models

```java
// Command side: write model with domain logic
public class OrderCommandService {
    private final OrderWriteRepository writeRepo;

    @Transactional
    public Long placeOrder(PlaceOrderCommand cmd) {
        Order order = Order.create(cmd);
        writeRepo.save(order);
        return order.getId();
    }
}

// Query side: read model optimized for display
public class OrderQueryService {
    private final OrderReadRepository readRepo;

    public List<OrderSummary> getOrderSummaries(Long customerId) {
        return readRepo.findSummariesByCustomer(customerId);
    }
}
```

#### Read Model (Denormalized for Queries)

```sql
-- Read model: optimized for query performance
CREATE TABLE order_summaries (
    order_id BIGINT PRIMARY KEY,
    customer_name VARCHAR(255),
    total_amount DECIMAL(10,2),
    item_count INT,
    status VARCHAR(50),
    created_at TIMESTAMP
);
-- Maintained via events or triggers, not primary writes
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `OrderCommandService` | Handles writes, enforces business rules |
| `OrderQueryService` | Handles reads, returns DTOs |
| `order_summaries` | Denormalized read model |
| `PlaceOrderCommand` | Intent-capturing command object |

#### Syntax Rules

- Commands should represent specific business tasks, not low-level data updates. Use "Book hotel room" instead of "Set ReservationStatus to Reserved".
- Queries return DTOs, not domain entities, and never alter data.
- The read model is maintained via events, triggers, or batch synchronization.
- CQRS is not a top-level architecture; it applies to specific bounded contexts where the read/write asymmetry is significant.

#### Constraints and Limitations

- CQRS introduces eventual consistency between the write and read models.
- Maintaining two models increases complexity and requires synchronization logic.
- Not suitable for simple CRUD applications where a single model suffices.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: CQRS with Event-Driven Read Model Sync

```java
// Step 1: Write side — command handler publishes event
@Service
public class OrderCommandHandler {
    @Autowired private OrderWriteRepository writeRepo;
    @Autowired private ApplicationEventPublisher publisher;

    @Transactional
    public void handle(PlaceOrderCommand cmd) {
        Order order = new Order(cmd.getCustomerId(), cmd.getItems());
        writeRepo.save(order);
        publisher.publishEvent(new OrderPlacedEvent(order.getId(), order.getTotal()));
    }
}

// Step 2: Read side — event listener updates denormalized view
@Component
public class OrderReadModelUpdater {
    @Autowired private OrderSummaryRepository summaryRepo;

    @EventListener
    public void on(OrderPlacedEvent event) {
        OrderSummary summary = new OrderSummary();
        summary.setOrderId(event.getOrderId());
        summary.setTotal(event.getTotal());
        summaryRepo.save(summary);
    }
}

// Step 3: Query handler reads from denormalized model
@Service
public class OrderQueryHandler {
    @Autowired private OrderSummaryRepository summaryRepo;

    public List<OrderSummary> getCustomerOrders(Long customerId) {
        return summaryRepo.findByCustomerId(customerId);
    }
}
```

**Expected Output**: When `PlaceOrderCommand` is handled, the write model is updated and an event is published. The read model is updated asynchronously via the event listener. Queries for order summaries hit the denormalized `order_summaries` table, which is optimized for reads.

**Why This Output Occurs**: Commands modify the write model and publish domain events. The read model subscribes to these events and maintains a denormalized view. Queries never touch the write model, eliminating lock contention and enabling independent optimization.

### Real-World Cases

**Case 1: E-Commerce Order Dashboard**: The write model uses normalized tables for order processing. The read model uses a denormalized `order_summaries` table for the customer dashboard, eliminating complex joins and improving load time.

**Case 2: Financial Reporting**: A trading platform uses CQRS: the write model processes trades with strict ACID guarantees; the read model maintains a materialized view for reporting, updated asynchronously.

**Case 3: Multi-Tenant SaaS**: CQRS enables per-tenant read models (custom views) while maintaining a unified write model, balancing consistency and query flexibility.

---

## Core Concept 3: Service Layers

### Definitions

**Core Definition**: The service layer encapsulates business logic, coordinates operations across multiple domain objects, and provides a well-defined interface for application operations.

**Technical Definition**: The service layer creates a boundary between presentation and domain logic while managing transactions and orchestrating complex workflows. It handles workflow orchestration, error handling, compensating transactions, and external integration coordination. The service layer separates "orchestration" logic (what happens and in what order) from "domain" logic (specific business rules) and "transport" logic (HTTP/gRPC details).

**Beginner-Friendly Explanation**: The service layer is like a project manager. The domain objects (workers) know how to do specific tasks. The service layer (manager) decides the order of tasks, coordinates between workers, handles external vendors, and ensures the project completes successfully — but doesn't do the actual work itself.

### Purposes

- **To** encapsulate business rules and workflows in a single, testable location
- **To** orchestrate operations across multiple domain entities and external services
- **To** control transaction boundaries and ensure data consistency
- **To** provide a stable interface that abstracts implementation complexity from presentation layers

### Syntax Rules and Structure

#### Domain Service

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final PasswordService passwordService;

    public UserService(UserRepository userRepository,
                       EmailService emailService,
                       PasswordService passwordService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.passwordService = passwordService;
    }

    @Transactional
    public User registerUser(RegisterUserRequest request) {
        // Business rule: email must be unique
        userRepository.findByEmail(request.getEmail())
            .ifPresent(u -> { throw new BusinessError("Email already registered"); });

        // Business logic: hash password, create entity
        String hashedPassword = passwordService.hash(request.getPassword());
        User user = User.create(request, hashedPassword);

        // Persist
        User savedUser = userRepository.save(user);

        // Coordinate side effects
        emailService.sendVerificationEmail(savedUser);

        return savedUser;
    }
}
```

#### Application Service (Orchestrator)

```java
@Service
public class OrderProcessingService {
    private final OrderService orderService;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final NotificationService notificationService;

    @Transactional
    public OrderResult processOrder(CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        try {
            inventoryService.reserveItems(order.getItems());
            PaymentResult payment = paymentService.processPayment(order);
            orderService.confirmOrder(order.getId(), payment.getTransactionId());
            notificationService.sendOrderConfirmation(order);
            return new OrderResult(order.getId(), "CONFIRMED");
        } catch (PaymentFailedException e) {
            // Compensating action
            inventoryService.releaseItems(order.getItems());
            orderService.cancelOrder(order.getId(), "Payment failed");
            throw e;
        }
    }
}
```

#### Component Breakdown

| Component | Responsibility |
|-----------|----------------|
| `UserService` | Domain service: business rules for user operations |
| `OrderProcessingService` | Application service: orchestrates multi-service workflow |
| `@Transactional` | Declares transaction boundary |
| Compensating action | Rolls back partial work on failure |

#### Syntax Rules

- Services should be stateless.
- The service layer is the only layer that knows the *sequence* of steps a business operation requires.
- Transaction boundaries should be declared on service methods, not on repository methods.
- Application services orchestrate; domain services contain business rules.

#### Constraints and Limitations

- Services should not contain HTTP/presentation logic — that belongs in controllers.
- Avoid the "anemic domain model" anti-pattern where services contain all logic and entities are just data holders.
- Service-to-service dependencies can create tight coupling; prefer event-driven communication for cross-domain workflows.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Service Layer with Orchestration and Compensation

```java
// Step 1: Define service interface
public interface OrderProcessingService {
    OrderResult processOrder(CreateOrderRequest request);
    void cancelOrder(Long orderId, String reason);
}

// Step 2: Implementation with orchestration
@Service
public class OrderProcessingServiceImpl implements OrderProcessingService {
    private final OrderService orderService;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    @Transactional
    public OrderResult processOrder(CreateOrderRequest request) {
        // Step 1: Create order in PENDING state
        Order order = orderService.createOrder(request);

        try {
            // Step 2: Reserve inventory
            inventoryService.reserveItems(order.getItems());

            // Step 3: Process payment
            PaymentResult payment = paymentService.processPayment(order);

            // Step 4: Confirm order
            orderService.confirmOrder(order.getId(), payment.getTransactionId());

            return new OrderResult(order.getId(), "CONFIRMED", payment.getTransactionId());
        } catch (Exception e) {
            // Compensating actions in reverse order
            inventoryService.releaseItems(order.getItems());
            orderService.cancelOrder(order.getId(), e.getMessage());
            throw new OrderProcessingException("Order failed", e);
        }
    }
}
```

**Expected Output**: On success, the order is confirmed with a transaction ID. On failure, inventory is released and the order is cancelled, leaving no partial state.

**Why This Output Occurs**: The service layer coordinates multiple domain services within a transaction boundary. Compensating actions in the catch block ensure that partial work is undone, maintaining consistency even when individual steps fail.

### Real-World Cases

**Case 1: Banking Transfer Service**: A `TransferService` orchestrates debit, credit, and audit operations across multiple repositories within a single transaction.

**Case 2: E-Commerce Checkout**: An `OrderProcessingService` coordinates inventory reservation, payment processing, and notification sending, with compensating actions for each failure scenario.

**Case 3: User Registration Workflow**: A `UserRegistrationService` validates uniqueness, hashes the password, creates the user, sends a verification email, and logs the event — all within one transaction.

---

## Core Concept 4: Transaction Boundaries and Saga Pattern

### Definitions

**Core Definition**: Transaction boundaries define the scope within which database operations are atomic. The Saga pattern manages distributed transactions across microservices through a sequence of local transactions with compensating actions.

**Technical Definition**: The Saga pattern manages transactions by breaking them into a sequence of local transactions. Each local transaction performs its work atomically within a single service, updates the service's database, and triggers the next step through events or messages. If a step fails, a series of compensating transactions undoes the changes made by previous steps. There are two implementation approaches: **choreography** (no central coordinator; each service listens for and responds to events) and **orchestration** (a central orchestrator manages the flow).

**Beginner-Friendly Explanation**: A regular transaction is like a single bank transfer — it either happens or it doesn't. A Saga is like a multi-step trip: you book a flight, then a hotel, then a car. If the car rental fails, you cancel the hotel and then the flight. Each step is its own transaction, and compensation undoes previous steps.

### Purposes

- **To** maintain data consistency across multiple services without distributed locks
- **To** provide ACID-like guarantees in microservices architectures where 2PC is impractical
- **To** enable long-running business processes that span multiple services and timeframes
- **To** support both orchestration (central coordinator) and choreography (event-driven) models

### Syntax Rules and Structure

#### Choreography-Based Saga (Event-Driven)

```java
// Order service publishes event
@Service
public class OrderService {
    @Autowired private ApplicationEventPublisher publisher;

    @Transactional
    public void createOrder(OrderRequest request) {
        Order order = orderRepo.save(new Order(request));
        publisher.publishEvent(new OrderCreatedEvent(order.getId(), order.getItems()));
    }
}

// Inventory service listens and reacts
@Service
public class InventoryService {
    @EventListener
    @Transactional
    public void on(OrderCreatedEvent event) {
        try {
            reserveItems(event.getItems());
            publisher.publishEvent(new InventoryReservedEvent(event.getOrderId()));
        } catch (Exception e) {
            publisher.publishEvent(new InventoryReservationFailedEvent(event.getOrderId()));
        }
    }
}

// Order service compensates on failure
@Service
public class OrderCompensationHandler {
    @EventListener
    @Transactional
    public void on(InventoryReservationFailedEvent event) {
        orderRepo.cancelOrder(event.getOrderId(), "Inventory unavailable");
    }
}
```

#### Orchestration-Based Saga

```java
@Service
public class OrderSagaOrchestrator {
    @Transactional
    public void execute(CreateOrderRequest request) {
        Long orderId = orderService.createOrder(request);
        
        try {
            inventoryService.reserveItems(orderId, request.getItems());
            paymentService.processPayment(orderId, request.getPaymentDetails());
            orderService.confirmOrder(orderId);
        } catch (InventoryException e) {
            orderService.cancelOrder(orderId, "Inventory failed");
        } catch (PaymentException e) {
            inventoryService.releaseItems(orderId, request.getItems());
            orderService.cancelOrder(orderId, "Payment failed");
        }
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| **Choreography** | No central coordinator; services react to events |
| **Orchestration** | Central orchestrator manages the flow |
| **Compensating transaction** | Undoes a previously committed local transaction |
| **Saga log** | Records saga state for recovery |

#### Syntax Rules

- Each local transaction must be idempotent to handle retries safely.
- Compensating transactions must be defined for every step that can fail.
- The Saga pattern is better than 2PC for microservices because locks are placed only for the duration of the local transaction.
- Choreography is simpler for few services; orchestration is better for complex workflows.

#### Constraints and Limitations

- Sagas provide eventual consistency, not immediate consistency.
- Compensating transactions may not always be possible (e.g., sending an email cannot be "unsent").
- The Saga log must be durable and recoverable for crash recovery.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Orchestration-Based Saga for Order Processing

```java
// Step 1: Define saga state
public class OrderSagaState {
    private Long orderId;
    private SagaStatus status; // STARTED, INVENTORY_RESERVED, PAYMENT_PROCESSED, COMPLETED, COMPENSATING
    private List<CompensationStep> compensations = new ArrayList<>();
}

// Step 2: Orchestrator executes steps with compensation tracking
@Service
public class OrderSagaOrchestrator {
    @Autowired private OrderService orderService;
    @Autowired private InventoryService inventoryService;
    @Autowired private PaymentService paymentService;
    @Autowired private SagaLogRepository sagaLog;

    @Transactional
    public void execute(CreateOrderRequest request) {
        OrderSagaState state = new OrderSagaState();
        
        try {
            // Step 1: Create order
            Long orderId = orderService.createOrder(request);
            state.setOrderId(orderId);
            state.setStatus(SagaStatus.STARTED);
            sagaLog.save(state);

            // Step 2: Reserve inventory
            inventoryService.reserveItems(orderId, request.getItems());
            state.setStatus(SagaStatus.INVENTORY_RESERVED);
            state.getCompensations().add(() -> inventoryService.releaseItems(orderId));
            sagaLog.save(state);

            // Step 3: Process payment
            paymentService.processPayment(orderId, request.getPaymentDetails());
            state.setStatus(SagaStatus.PAYMENT_PROCESSED);
            state.getCompensations().add(() -> paymentService.refund(orderId));
            sagaLog.save(state);

            // Step 4: Confirm order
            orderService.confirmOrder(orderId);
            state.setStatus(SagaStatus.COMPLETED);
            sagaLog.save(state);

        } catch (Exception e) {
            // Compensate in reverse order
            state.setStatus(SagaStatus.COMPENSATING);
            for (int i = state.getCompensations().size() - 1; i >= 0; i--) {
                state.getCompensations().get(i).run();
            }
            sagaLog.save(state);
            throw new SagaFailedException("Order saga failed", e);
        }
    }
}
```

**Expected Output**: On success, the saga completes with all steps executed. On failure at any step, compensations run in reverse order, restoring the system to its pre-saga state.

**Why This Output Occurs**: The orchestrator explicitly tracks each step and its corresponding compensation. When a step fails, the orchestrator executes all previously registered compensations in reverse order, ensuring eventual consistency across services.

### Real-World Cases

**Case 1: Travel Booking Saga**: A travel platform uses a Saga to book flight, hotel, and car rental. If the car rental fails, the hotel and flight bookings are compensated.

**Case 2: E-Commerce Order Fulfillment**: Order creation, inventory reservation, payment processing, and shipping are coordinated via a Saga, with compensating actions for each failure point.

**Case 3: Financial Transfer Across Services**: A Saga coordinates debit from one account, credit to another, and audit logging, with compensation if any step fails.

---

## Core Concept 5: Connection Lifecycle Management

### Definitions

**Core Definition**: Connection lifecycle management controls how database sessions and connections are opened, scoped, and closed throughout an application's request lifecycle.

**Technical Definition**: The Open Session in View (OSIV) anti-pattern keeps a Hibernate session open for the entire lifetime of an HTTP request — not only during the service layer and transaction, but also during view rendering or JSON serialization, after the transaction has already completed. OSIV does not keep a database connection open the whole time, but it does allow database queries to be executed at unpredictable moments, often very late in the request lifecycle.

**Beginner-Friendly Explanation**: OSIV is like keeping the library open all night just in case someone wants to browse books while walking home. Instead of deciding upfront which books are needed, you leave the doors open and let people wander in at random times. This wastes resources and creates unpredictable behavior.

### Purposes

- **To** define clear boundaries for when database sessions are open and closed
- **To** prevent connection pool exhaustion from held-open connections
- **To** avoid unpredictable query execution during view rendering
- **To** ensure transactions are properly scoped and committed

### Syntax Rules and Structure

#### Disabling OSIV (Spring Boot)

```properties
# application.properties
spring.jpa.open-in-view=false
```

#### Explicit Connection Scope

```java
// BAD: OSIV allows lazy loading during view rendering
@GetMapping("/orders")
public List<Order> getOrders() {
    return orderService.findAll();  // Returns entities with lazy associations
    // View rendering triggers lazy loads — unpredictable queries!
}

// GOOD: Service layer explicitly loads what's needed
@GetMapping("/orders")
public List<OrderDto> getOrders() {
    return orderService.findAllWithDetails();  // Returns DTOs with all data loaded
    // No lazy loading during rendering — predictable, efficient
}
```

#### Component Breakdown

| Approach | Session Scope | Query Timing | Connection Held |
|----------|---------------|--------------|-----------------|
| **OSIV** | Entire HTTP request | Unpredictable (during rendering) | Potentially long |
| **Scoped** | Service/transaction only | Predictable (during service call) | Short |

#### Syntax Rules

- Disable OSIV in production (`spring.jpa.open-in-view=false`).
- Service methods should return DTOs, not entities with lazy associations.
- If returning entities, use `JOIN FETCH` or `@EntityGraph` to load all needed associations.
- The session should be scoped to the transaction, not the request.

#### Constraints and Limitations

- OSIV keeps the database connection held throughout UI rendering, increasing connection lease time and limiting overall transaction throughput due to congestion on the connection pool.
- Every additional statement issued from the UI rendering phase is executed in auto-commit mode, causing significant I/O pressure.
- OSIV creates no separation of concerns because statements are generated both by the service layer and by the UI rendering process.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: OSIV Problem and Fix

```java
// PROBLEM: OSIV enabled — lazy loading during view rendering
@RestController
public class OrderController {
    @Autowired private OrderRepository orderRepo;

    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        return orderRepo.findById(id).orElseThrow();
        // When Jackson serializes this, it triggers lazy loads
        // for every association — N+1 queries during rendering!
    }
}

// FIX: Disable OSIV and use DTO projection
@RestController
public class FixedOrderController {
    @Autowired private OrderQueryService queryService;

    @GetMapping("/orders/{id}")
    public OrderDto getOrder(@PathVariable Long id) {
        return queryService.getOrderDto(id);
    }
}

@Service
public class OrderQueryService {
    @Autowired private OrderRepository orderRepo;

    @Transactional(readOnly = true)
    public OrderDto getOrderDto(Long id) {
        Order order = orderRepo.findByIdWithAssociations(id);  // JOIN FETCH
        return new OrderDto(order);  // DTO built within transaction
    }
}
```

**Expected Output** (OSIV enabled):
```
Hibernate: select o.* from orders o where o.id = ?
Hibernate: select c.* from customers c where c.id = ?    -- Lazy load during serialization
Hibernate: select i.* from order_items i where i.order_id = ?  -- Lazy load during serialization
Hibernate: select p.* from products p where p.id = ?  -- Per item, N+1!
```

**Expected Output** (OSIV disabled, DTO):
```
Hibernate: select o.*, c.*, i.*, p.* from orders o 
           join customers c on o.customer_id = c.id
           join order_items i on i.order_id = o.id
           join products p on i.product_id = p.id
           where o.id = ?
```

**Why This Output Occurs**: With OSIV enabled, the Hibernate session remains open during Jackson serialization, triggering lazy loads for every association. With OSIV disabled and a DTO projection, all data is loaded in the service layer with a single JOIN FETCH query, and the DTO contains only the fields needed for serialization.

### Real-World Cases

**Case 1: Connection Pool Exhaustion**: A high-traffic API with OSIV enabled holds connections throughout serialization, exhausting the pool under load. Disabling OSIV and using DTOs reduces connection hold time by 80%.

**Case 2: Unpredictable Query Patterns**: An application with OSIV triggers queries during Thymeleaf template rendering, making it impossible to predict query patterns. Disabling OSIV forces explicit loading in the service layer.

**Case 3: N+1 During Rendering**: A REST API with OSIV serializes a list of entities with lazy associations, triggering N+1 queries. Using `@EntityGraph` in the repository query loads all associations eagerly.

---

## Core Concept 6: Error Handling and SQL State Translation

### Definitions

**Core Definition**: Error handling in data access involves translating vendor-specific SQL error codes into application-level exceptions that business logic can interpret and respond to appropriately.

**Technical Definition**: SQLSTATE is a five-character alphanumeric code that indicates the outcome of a SQL statement. Class 40 represents transaction rollback (deadlock, serialization failure), with specific codes including 40001 (deadlock timeout) and 40P01 (deadlock detected in PostgreSQL). Applications should test for SQLExceptions with SQLStates of 40001 (deadlock timeout) or 40XL1/40XL2 (lockwait timeout) and retry the transaction accordingly. Spring's `SQLErrorCodeSQLExceptionTranslator` maps 40001 to `PessimisticLockingFailureException` or more specifically `CannotAcquireLockException`.

**Beginner-Friendly Explanation**: When the database says "something went wrong," it speaks in codes, not plain English. A deadlock is code 40001, a lock timeout is 40XL1. Your application needs a translator that turns these codes into meaningful exceptions like `DeadlockException` or `LockTimeoutException`, so your code can decide whether to retry or fail.

### Purposes

- **To** translate vendor-specific SQL error codes into meaningful application exceptions
- **To** enable retry logic for transient failures (deadlocks, lock timeouts)
- **To** distinguish between retryable and non-retryable errors
- **To** provide consistent error handling across different database vendors

### Syntax Rules and Structure

#### SQL State Codes for Deadlock and Lock Timeout

| SQLSTATE | Meaning | Retryable? |
|----------|---------|------------|
| 40001 | Deadlock timeout | ✅ Yes (retry immediately) |
| 40P01 | Deadlock detected (PostgreSQL) | ✅ Yes |
| 40XL1 | Lock wait timeout | ❌ No (do not retry immediately) |
| 40XL2 | Lock wait timeout | ❌ No |

#### Java Exception Handling

```java
import java.sql.SQLException;

public class OrderService {
    private static final int MAX_RETRIES = 3;

    public void processOrderWithRetry(Order order) {
        for (int attempt = 1; attempt <= MAX_RETRIES; attempt++) {
            try {
                updateInventory(order);
                updateOrderStatus(order);
                return;  // Success
            } catch (SQLException se) {
                if (se.getSQLState().equals("40001")) {
                    // Deadlock victim — retry
                    System.out.println("Deadlock detected, retrying (attempt " + attempt + ")");
                    if (attempt == MAX_RETRIES) {
                        throw new RetryExhaustedException("Deadlock persisted", se);
                    }
                    // Exponential backoff
                    try {
                        Thread.sleep(100L * attempt);
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                        throw new RuntimeException(ie);
                    }
                } else {
                    throw new DataAccessException("Non-retryable error", se);
                }
            }
        }
    }
}
```

#### Spring Exception Translation

```java
// Spring automatically translates SQLException to DataAccessException hierarchy
// 40001 -> CannotAcquireLockException (retryable)
// 23505 (unique violation) -> DuplicateKeyException (non-retryable)
// 40XL1 -> CannotAcquireLockException (not retryable)

@Repository
public class UserRepository {
    @Autowired private JdbcTemplate jdbc;

    public void createUser(User user) {
        try {
            jdbc.update("INSERT INTO users (email) VALUES (?)", user.getEmail());
        } catch (DuplicateKeyException e) {
            throw new BusinessError("Email already exists", e);
        }
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `SQLState 40001` | Deadlock victim — transaction was chosen for rollback |
| `SQLState 40XL1` | Lock wait timeout — waited too long for a lock |
| `DuplicateKeyException` | Spring translation of unique constraint violation |
| `CannotAcquireLockException` | Spring translation of deadlock/lock timeout |

#### Syntax Rules

- Catch deadlock exceptions only at the outermost level of application code, not in database-side methods.
- Retry deadlocks at least once; do not retry lock wait timeouts immediately.
- Use SQLSTATE class 40 for broad logic (all deadlock conditions share this class).
- Spring's `SQLErrorCodeSQLExceptionTranslator` uses vendor-specific error codes for precise translation.

#### Constraints and Limitations

- Not all vendors use the same SQLSTATE codes; always test with the production database.
- Retrying non-idempotent operations (e.g., payments) can cause double-processing.
- Deadlock retries should be bounded to prevent infinite loops.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Deadlock Retry with Exponential Backoff

```java
// Step 1: Define retryable exception
public class RetryableDataAccessException extends RuntimeException {
    public RetryableDataAccessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Step 2: Retry template with deadlock detection
public class DeadlockRetryTemplate {
    private static final int MAX_ATTEMPTS = 3;

    public <T> T execute(Supplier<T> operation) {
        for (int attempt = 1; attempt <= MAX_ATTEMPTS; attempt++) {
            try {
                return operation.get();
            } catch (DataAccessException e) {
                if (isDeadlock(e) && attempt < MAX_ATTEMPTS) {
                    long delay = (long) (100 * Math.pow(2, attempt) 
                        + Math.random() * 100);
                    System.out.println("Deadlock detected, retrying in " 
                        + delay + "ms (attempt " + attempt + ")");
                    sleep(delay);
                } else if (isDeadlock(e)) {
                    throw new RetryExhaustedException("Deadlock persisted", e);
                } else {
                    throw e;  // Non-retryable
                }
            }
        }
        throw new IllegalStateException("Unreachable");
    }

    private boolean isDeadlock(DataAccessException e) {
        Throwable root = e.getRootCause();
        if (root instanceof SQLException) {
            String state = ((SQLException) root).getSQLState();
            return "40001".equals(state) || "40P01".equals(state);
        }
        return false;
    }

    private void sleep(long ms) {
        try { Thread.sleep(ms); } 
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}
```

**Expected Output**:
```
Deadlock detected, retrying in 234ms (attempt 1)
Deadlock detected, retrying in 512ms (attempt 2)
Transfer completed successfully
```

**Why This Output Occurs**: The retry template catches `DataAccessException`, extracts the root SQLException, and checks the SQLSTATE. For 40001 (deadlock), it retries with exponential backoff and jitter. After the deadlock clears, the operation succeeds.

### Real-World Cases

**Case 1: E-Commerce Inventory Updates**: Multiple concurrent orders cause deadlocks on inventory rows. The application retries deadlock victims with exponential backoff, ensuring orders eventually succeed.

**Case 2: Banking Transfers**: Transfer operations acquire locks in a consistent order to minimize deadlocks. When deadlocks occur, the retry template handles them transparently.

**Case 3: Spring Data JPA Deadlock Handling**: Spring's exception translation converts SQLSTATE 40001 to `CannotAcquireLockException`, which the service layer catches and retries.

---

## Core Concept 7: Application Resilience Patterns

### Definitions

**Core Definition**: Application resilience patterns are mechanisms that prevent, detect, and recover from failures in distributed systems, ensuring that transient faults do not cascade into complete outages.

**Technical Definition**: Resilience patterns include the **circuit breaker** (stops requests to a failing dependency after a threshold, with CLOSED, OPEN, and HALF-OPEN states), **retry with exponential backoff and jitter** (delay = base × 2^attempt + random(0, base)), and **fail-fast** (immediately returning errors when a dependency is known to be unavailable). A circuit breaker transitions: CLOSED → OPEN when failures exceed threshold → HALF-OPEN after timeout → CLOSED on successful probe. Retries should only apply to idempotent operations; retrying a `POST /payment` double-charges.

**Beginner-Friendly Explanation**: A circuit breaker is like an electrical breaker in your house. When there's a short circuit (repeated failures), the breaker trips and stops the flow of electricity (requests). After a while, you test if the problem is fixed by flipping the breaker (HALF-OPEN). If it works, normal operation resumes. Exponential backoff with jitter is like knocking on a door: you wait 1 second, then 2, then 4, and you add random variation so not everyone knocks at the same time.

### Purposes

- **To** prevent cascading failures by stopping requests to a failing dependency
- **To** allow failing dependencies time to recover by reducing load
- **To** spread retry attempts over time to avoid thundering herd problems
- **To** fail fast when a dependency is known to be unavailable, avoiding wasted resources

### Syntax Rules and Structure

#### Circuit Breaker Configuration (Resilience4j)

```java
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)                // Open after 50% failures
    .waitDurationInOpenState(Duration.ofSeconds(30))  // Wait 30s before HALF-OPEN
    .slidingWindowSize(10)                   // Evaluate last 10 calls
    .minimumNumberOfCalls(5)                 // Need 5 calls before evaluating
    .permittedNumberOfCallsInHalfOpenState(3) // Allow 3 probe calls
    .build();

CircuitBreaker circuitBreaker = CircuitBreaker.of("database", config);

// Wrap the call
Supplier<Result> decorated = CircuitBreaker.decorateSupplier(
    circuitBreaker, () -> database.query(sql));

try {
    Result result = decorated.get();
} catch (CallNotPermittedException e) {
    // Circuit is OPEN — fail fast
    return fallbackResult();
}
```

#### Exponential Backoff with Jitter

```java
public <T> T withRetry(Supplier<T> operation, int maxAttempts) {
    for (int attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            return operation.get();
        } catch (TransientException e) {
            if (attempt == maxAttempts) throw e;
            
            // Exponential backoff with full jitter
            long baseDelay = 100;  // 100ms
            long maxDelay = 3000;  // 3 seconds
            long exponential = baseDelay * (1L << attempt);  // 200, 400, 800...
            long capped = Math.min(exponential, maxDelay);
            long delay = (long) (Math.random() * capped);  // Full jitter
            
            System.out.println("Retry " + attempt + " in " + delay + "ms");
            sleep(delay);
        }
    }
    throw new IllegalStateException("Unreachable");
}
```

#### Component Breakdown

| State | Behavior | Transition |
|-------|----------|------------|
| **CLOSED** | Normal operation, track failures | → OPEN when failures exceed threshold |
| **OPEN** | Fail fast, no calls to dependency | → HALF-OPEN after timeout |
| **HALF-OPEN** | Allow limited probe requests | → CLOSED on success, → OPEN on failure |

#### Retryable vs Non-Retryable Errors

| Error Type | Retry? |
|------------|--------|
| Network timeout | ✅ Yes |
| 429 Too Many Requests | ✅ Yes (respect Retry-After) |
| 503 Service Unavailable | ✅ Yes |
| 400 Bad Request | ❌ No (client bug) |
| 401 Unauthorized | ❌ No |
| 500 (non-idempotent) | ❌ No (risk of double-processing) |

#### Syntax Rules

- Retry only idempotent operations.
- Max retries: 3 attempts is standard; never infinite retry loops.
- Use full jitter: `delay = uniform(0, min(max_delay, base_delay * 2^attempt))`.
- Circuit breaker failure threshold: 50% of requests in a 10-second window.
- Track retry rate: if > 10% of requests are retries, the underlying system is failing — alert instead of retrying.

#### Constraints and Limitations

- Circuit breaker adds state management complexity.
- Retries can amplify load during outages if not bounded.
- Compensating for non-idempotent operations requires idempotency keys or deduplication.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Complete Resilience Stack (Circuit Breaker + Retry + Jitter)

```java
// Step 1: Configure resilience4j
@Configuration
public class ResilienceConfig {
    @Bean
    public CircuitBreaker databaseCircuitBreaker() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofSeconds(30))
            .slidingWindowSize(10)
            .minimumNumberOfCalls(5)
            .build();
        return CircuitBreaker.of("database", config);
    }

    @Bean
    public Retry databaseRetry() {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .intervalFunction(IntervalFunction.ofExponentialRandomBackoff(
                100, 2.0, 0.5))  // 100ms base, 2x multiplier, 50% jitter
            .retryExceptions(SQLException.class, TimeoutException.class)
            .ignoreExceptions(BusinessException.class)
            .build();
        return Retry.of("database", config);
    }
}

// Step 2: Compose decorators
@Service
public class ResilientOrderService {
    private final CircuitBreaker circuitBreaker;
    private final Retry retry;
    private final OrderRepository repository;

    public ResilientOrderService(CircuitBreaker cb, Retry retry,
                                  OrderRepository repo) {
        this.circuitBreaker = cb;
        this.retry = retry;
        this.repository = repo;
    }

    public Order findOrder(Long id) {
        // Compose: CircuitBreaker -> Retry -> actual call
        Supplier<Order> decorated = Decorators.ofSupplier(
                () -> repository.findById(id).orElseThrow())
            .withCircuitBreaker(circuitBreaker)
            .withRetry(retry)
            .withFallback(Arrays.asList(CallNotPermittedException.class),
                e -> Order.fallback(id))
            .decorate();

        return decorated.get();
    }
}
```

**Expected Output** (normal operation):
```
Order found: ORD-12345
```

**Expected Output** (circuit open):
```
Circuit is OPEN — returning fallback order
```

**Why This Output Occurs**: The circuit breaker wraps the retry-wrapped repository call. Under normal conditions, the call succeeds. When failures exceed the threshold (50% in a 10-call window), the circuit opens and returns a fallback immediately without calling the database. After 30 seconds, it enters HALF-OPEN and allows probe calls to test recovery.

### Real-World Cases

**Case 1: Database Outage Fallback**: An e-commerce application uses a circuit breaker around database calls. When the database is unavailable, the circuit opens and serves cached product data from Redis, keeping the site partially functional.

**Case 2: Payment Gateway Resilience**: A payment service retries transient gateway failures with exponential backoff and jitter. After 3 failed attempts, it falls back to an alternative gateway.

**Case 3: Thundering Herd Prevention**: A social media API uses full jitter retry to prevent synchronized retry storms after a brief outage. The retry delay is `uniform(0, min(3000, 100 * 2^attempt))` milliseconds.

---

## References

| Name | Link |
|------|------|
| Martin Fowler — Repository Pattern | https://martinfowler.com/eaaCatalog/repository.html |
| Microsoft Learn — Repository Pattern Purpose | https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10) |
| fast-repository — Interface-First Repository Pattern | https://pypi.org/project/fast-repository/ |
| Azure Architecture Center — CQRS Pattern | https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs |
| AWS — CQRS Pattern | https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/cqrs.html |
| Service Layer Pattern — GitHub | https://github.com/foomakers/pair/blob/main/.pair/knowledge/guidelines/code-design/framework-patterns/service-layer.md |
| Azure Architecture Center — Saga Design Pattern | https://learn.microsoft.com/en-us/azure/architecture/patterns/saga |
| AWS — Saga Choreography Pattern | https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/saga-choreography.html |
| Open Session In View Anti-Pattern — Stack Overflow | https://stackoverflow.com/revisions/85a02fb8-ce60-40f5-9113-49c06d03b87e/view-source |
| Martinelli — Open Session in View in Spring Boot | https://martinelli.ch/the-hidden-performance-killer-understanding-open-session-in-view-in-spring-boot/ |
| Oracle — Programming Applications to Handle Deadlocks | https://docs.oracle.com/javadb/10.6.2.1/devguide/cdevconcepts32861.html |
| Spring Framework — SQL Error Code Translation | https://github.com/spring-projects/spring-framework/issues/36499 |
| Resilience Patterns — Circuit Breaker, Retry, Bulkhead | https://github.com/HoangNguyen0403/agent-skills-standard/blob/main/.github/skills/common/common-system-design/references/resilience-patterns.md |
| AWS Builder's Library — Exponential Backoff and Jitter | https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/ |
| SQL Return Codes — SQLSTATE Best Practices | https://www.cleverence.com/blog/sql-return-codes-sqlstate-vendor-errors/ |