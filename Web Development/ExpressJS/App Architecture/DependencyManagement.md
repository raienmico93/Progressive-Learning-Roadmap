# Dependency Management & Inversion of Control (IoC) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dependency management is the practice of organising how the components of an application obtain the other components they depend on, while Inversion of Control (IoC) is the principle of transferring control over object creation and lifecycle management from the application code to an external entity.

**Technical Definition:** Dependency injection is a design pattern that implements the principle of Inversion of Control. Under this model, a component never imports or instantiates its own dependencies directly. Instead, dependencies are passed into the component from the outside (injected) during configuration, shifting the responsibility of providing dependencies to external code . The primary forms of injection are **constructor injection** (dependencies passed into a class's constructor), **property/setter injection** (assigned after creation), and **parameter injection** (passed into individual method calls). An **IoC container** is an automated dependency resolution system that manages object creation and lifetimes based on registration metadata .

**Beginner-Friendly Explanation:** Imagine you're assembling a piece of furniture. Without DI, you'd have to forge your own screws, cut your own wood, and build your own tools before you could even start. With DI, someone hands you the screws, the wood, and the tools — you just assemble. Inversion of Control takes this further: a warehouse manager (the container) keeps track of everything you need and delivers it to your workbench automatically. You never worry about where things come from; you just do your job.

### Key Characteristics

- **Inversion of control:** The responsibility for creating dependencies is inverted — moved from the consumer to an external assembler or container.
- **Loose coupling:** Components depend on abstractions (interfaces) rather than concrete implementations, making them easier to replace.
- **Testability:** Dependencies can be swapped with mocks or stubs during unit testing without modifying the component under test.
- **Explicit dependencies:** Constructor injection makes all dependencies visible at the point of instantiation.
- **Lifecycle management:** Containers manage object lifetimes (singleton, scoped, transient) automatically.
- **Composition root:** All wiring happens in one place, separate from business logic.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript or TypeScript knowledge:** Classes, constructors, and modules.
- **Understanding of Express routing and middleware.**
- **Familiarity with unit testing concepts:** Mocks, stubs, and fakes.

### Related Programming Areas

- **SOLID Principles:** Dependency Inversion Principle (DIP) is the theoretical foundation; DI is the implementation mechanism.
- **Separation of Concerns:** DI enables clean layer separation by decoupling components.
- **Testing:** DI is essential for isolated unit testing with mocks.
- **Modular Architecture:** Feature modules benefit from DI for cross-module dependencies.
- **Framework ecosystems:** NestJS, Angular, and InversifyJS provide built-in DI containers.

### Core Concepts

1. **Dependency Injection (DI) Concepts** — passing dependencies via constructors rather than hardcoding.
2. **Service Dependencies** — structuring cross-dependencies (e.g., EmailService into AuthService).
3. **Configuration Injection** — passing tailored configuration into database handlers and SDK providers.
4. **Testing Dependencies** — mocking, stubbing, and swapping real services during unit testing.
5. **Inversion of Control Containers** — automated resolution vs. manual composition root.

---

## Core Concept 1: Dependency Injection (DI) Concepts

### Definitions

**Core Definition:** Dependency injection is a technique where an object receives other objects it depends on, rather than creating them internally, shifting the responsibility of providing dependencies to external code .

**Technical Definition:** Dependency injection is a design pattern that implements the principle of Inversion of Control. Under this model, a component never imports or instantiates its own dependencies directly. Instead, the dependencies are passed into the component from the outside (injected) during configuration . The three operational forms are constructor injection (preferred, most explicit), property/setter injection (flexible but riskier), and parameter injection (method-level) .

**Beginner-Friendly Explanation:** Without DI, a class is like a chef who has to grow their own vegetables, raise their own chickens, and forge their own knives before cooking. With DI, the chef simply says "I need vegetables, chicken, and a knife" — and someone hands them over. The chef focuses on cooking, not on sourcing ingredients.

### Purposes

- To pass dependencies dynamically into classes and functions via constructors or arguments rather than hardcoding `require()` or `import` links inside files.
- To reduce tight coupling between components, making the codebase more maintainable and modular.
- To enable swapping implementations (e.g., switching databases or services) without touching the consuming code .
- To make unit testing feasible by allowing dependencies to be replaced with mocks.
- To make all dependencies explicit at the point of instantiation.

### Syntax Rules and Structure

#### Constructor Injection (Preferred Pattern)

```typescript
// WITHOUT DI — tightly coupled
class OrderService {
  private db = new PostgresDatabase();   // hardcoded
  private mailer = new SmtpMailer();     // hardcoded
}

// WITH DI — loosely coupled
class OrderService {
  constructor(
    private readonly db: Database,       // interface, not implementation
    private readonly mailer: Mailer      // interface, not implementation
  ) {}
}
```


| Component | Breakdown |
|-----------|-----------|
| `constructor(db, mailer)` | Dependencies are declared as constructor parameters. |
| `private readonly db: Database` | The dependency is typed as an interface, not a concrete class. |
| Instantiation | The consumer (`new OrderService(db, mailer)`) provides the implementations. |

#### Three Forms of Injection

| Form | Mechanism | Pros | Cons |
|------|-----------|------|------|
| **Constructor** | Passed into constructor | Explicit, guarantees dependencies at creation | Requires all deps upfront |
| **Property/Setter** | Assigned after creation | Flexible, optional deps | Runtime errors if not set |
| **Parameter** | Passed into method calls | Method-level granularity | Verbose for repeated deps |



#### Syntax Rules

- Dependencies should be typed as **interfaces or abstractions**, not concrete classes .
- Constructor injection is the **preferred** pattern — it guarantees the component is fully initialised before use .
- The consumer should **never** call `new` on a dependency inside the class — that defeats the purpose.
- All wiring should happen in a **composition root** — a single place where the object graph is assembled .

#### Constraints and Limitations

- Constructor injection can lead to long parameter lists in classes with many dependencies.
- Over-injection (too many dependencies) may indicate a class has too many responsibilities (violates SRP).
- DI adds indirection — debugging requires tracing through the container or composition root.

### Annotated Code Example

```typescript
// ❌ BAD: Hardcoded dependency
class UserService {
  private db = new PostgresDatabase();   // Tightly coupled
  async getUser(id: string) {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
  }
}

// ✅ GOOD: Constructor injection
interface Database {
  query(sql: string, params: unknown[]): Promise<unknown>;
}

class UserService {
  constructor(private readonly db: Database) {}   // Injected
  async getUser(id: string) {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
  }
}

// Composition root — wiring happens here, once
const db = new PostgresDatabase();
const userService = new UserService(db);
```

**Expected Output (when testing `UserService` with a mock database):**
```typescript
const mockDb: Database = {
  query: jest.fn().mockResolvedValue([{ id: '1', name: 'Alice' }])
};
const service = new UserService(mockDb);
await service.getUser('1');
// mockDb.query was called with the correct SQL and parameters
```

**Why this output:** The `UserService` depends on the `Database` interface, not a concrete implementation. In production, `PostgresDatabase` is injected; in tests, `mockDb` is injected. The service code never changes .

### Real-World Cases

- **E-commerce:** `OrderService` receives a `PaymentGateway` and `InventoryRepository` via constructor injection.
- **SaaS platforms:** `SubscriptionService` receives a `BillingProvider` (Stripe, Paddle) that can be swapped.
- **Multi-database apps:** `UserRepository` receives a `DatabaseConnection` that can be PostgreSQL in production and SQLite in tests.

---

## Core Concept 2: Service Dependencies

### Definitions

**Core Definition:** Service dependencies are the cross-dependencies between services — for example, an `AuthService` depending on an `EmailService` to send verification emails.

**Technical Definition:** In a layered architecture, services often depend on other services to fulfil their responsibilities. Rather than instantiating these dependencies internally, they are injected via the constructor. This allows the dependency graph to be managed centrally and ensures that services remain testable in isolation. The injected dependency is typically typed as an interface, allowing different implementations to be swapped .

**Beginner-Friendly Explanation:** Services are like specialists in a company. The authentication department needs the email department to send verification emails. Instead of the auth department hiring its own email person, the company (the container) provides an email specialist to the auth department. If the company switches email providers, only the email department changes — the auth department never notices.

### Purposes

- To structure cross-dependencies between services cleanly.
- To inject an `EmailService` into an `AuthService` without hardcoding the dependency.
- To allow different email implementations (SendGrid, AWS SES, console logger) to be swapped without changing `AuthService`.
- To enable testing `AuthService` with a mock `EmailService`.

### Syntax Rules and Structure

```typescript
// services/email.service.ts
interface EmailService {
  sendVerificationEmail(to: string, token: string): Promise<void>;
}

class SmtpEmailService implements EmailService {
  async sendVerificationEmail(to: string, token: string): Promise<void> {
    // SMTP implementation
  }
}

// services/auth.service.ts
class AuthService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService   // Injected
  ) {}

  async register(email: string, password: string): Promise<User> {
    const user = await this.userRepository.create({ email, password });
    const token = generateToken(user.id);
    await this.emailService.sendVerificationEmail(email, token);  // Uses injected service
    return user;
  }
}

// Composition root
const emailService = new SmtpEmailService();
const authService = new AuthService(userRepository, emailService);
```

| Component | Breakdown |
|-----------|-----------|
| `EmailService` (interface) | The abstraction that `AuthService` depends on. |
| `SmtpEmailService` | A concrete implementation. |
| `constructor(..., emailService)` | The dependency is injected. |
| `emailService.sendVerificationEmail()` | The injected service is used. |

#### Syntax Rules

- Services should depend on **interfaces**, not concrete implementations.
- Cross-service dependencies should be injected via the constructor.
- The composition root wires all services together.
- Circular dependencies (A depends on B, B depends on A) should be resolved using lazy injection or a proxy .

#### Constraints and Limitations

- Circular service dependencies can cause startup failures; use a container with proxy support or restructure the services.
- Too many cross-service dependencies may indicate a need to extract a shared service or use an event bus.

### Annotated Code Example

```typescript
// services/auth.service.ts
import { UserRepository } from '../repositories/user.repository';
import { EmailService } from './email.service';
import { TokenService } from './token.service';

export class AuthService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly tokenService: TokenService
  ) {}

  async register(email: string, password: string): Promise<void> {
    const user = await this.userRepository.create({ email, password });
    const token = this.tokenService.generate(user.id);
    await this.emailService.sendVerificationEmail(email, token);
  }

  async login(email: string, password: string): Promise<string> {
    const user = await this.userRepository.findByEmail(email);
    if (!user) throw new Error('User not found');
    return this.tokenService.generate(user.id);
  }
}
```

**Expected Output (when testing `AuthService` with a mock `EmailService`):**
```typescript
const mockEmailService: EmailService = {
  sendVerificationEmail: jest.fn().mockResolvedValue(undefined)
};
const authService = new AuthService(userRepository, mockEmailService, tokenService);
await authService.register('alice@example.com', 'password123');
expect(mockEmailService.sendVerificationEmail).toHaveBeenCalledWith('alice@example.com', expect.any(String));
```

**Why this output:** The `AuthService` uses the injected `EmailService` to send a verification email. In tests, a mock `EmailService` is injected, allowing the test to verify that the email was sent without actually sending it. The `AuthService` code is identical in both cases .

### Real-World Cases

- **Authentication flows:** `AuthService` → `EmailService` for verification and password-reset emails.
- **Order processing:** `OrderService` → `PaymentService` → `NotificationService` for order confirmations.
- **Content platforms:** `PostService` → `ModerationService` → `NotificationService` for flagged content alerts.

---

## Core Concept 3: Configuration Injection

### Definitions

**Core Definition:** Configuration injection is the practice of passing configuration values (database URLs, API keys, SDK settings) into components via constructors or dedicated configuration objects, rather than having components read environment variables directly.

**Technical Definition:** Configuration injection separates the concern of reading and validating environment variables from the components that use those values. A centralised configuration module loads environment variables, validates them against a schema (e.g., Zod), and exports a typed configuration object. This object is then injected into database handlers, SDK providers, and services that need it . The component remains unaware of `process.env` — it only receives the configuration it needs.

**Beginner-Friendly Explanation:** Configuration injection is like giving a chef a recipe card with the exact ingredients and quantities, rather than asking them to go find the ingredients themselves in a warehouse. The chef focuses on cooking; the warehouse manager (config module) handles sourcing and validating the ingredients.

### Purposes

- To pass tailored configuration options cleanly into database handlers or external SDK providers.
- To centralise environment variable loading and validation in one module.
- To make configuration testable by allowing different config values to be injected.
- To prevent configuration values from being scattered across the codebase as `process.env` calls.

### Syntax Rules and Structure

```typescript
// config/schema.ts
import { z } from 'zod';

export const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  STRIPE_API_KEY: z.string().startsWith('sk_'),
  EMAIL_FROM: z.string().email()
});

export type Config = z.infer<typeof envSchema>;
```

```typescript
// config/index.ts
import 'dotenv/config';
import { envSchema } from './schema';

const parsed = envSchema.safeParse(process.env);
if (!parsed.success) {
  console.error(parsed.error.flatten().fieldErrors);
  process.exit(1);
}
export const config = parsed.data;
```

```typescript
// repositories/user.repository.ts
import { Pool } from 'pg';
import { Config } from '../config/schema';

export class UserRepository {
  private pool: Pool;

  constructor(config: Pick<Config, 'DATABASE_URL'>) {
    this.pool = new Pool({ connectionString: config.DATABASE_URL });
  }

  async findById(id: string) {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE id = $1', [id]
    );
    return result.rows[0];
  }
}

// Composition root — inject the relevant config slice
const userRepository = new UserRepository({ DATABASE_URL: config.DATABASE_URL });
```

| Component | Breakdown |
|-----------|-----------|
| `envSchema` | Zod schema validating all required environment variables. |
| `config` | The validated, typed configuration object. |
| `Pick<Config, 'DATABASE_URL'>` | The repository only receives the config it needs. |
| `connectionString` | The injected config value is used to create the pool. |

#### Syntax Rules

- Configuration should be validated **at startup** — fail fast if variables are missing or invalid.
- Components should receive only the configuration slices they need, not the entire config object.
- Configuration values should never be accessed via `process.env` inside services or repositories.
- Use TypeScript's `Pick<>` or destructuring to narrow the config surface.

#### Constraints and Limitations

- Environment variables are always strings; use Zod coercion (`z.coerce.number()`) for numeric values.
- Boolean coercion must be explicit — `"false"` is truthy in JavaScript.
- Secrets should never be logged or committed to version control.

### Annotated Code Example

```typescript
// services/stripe.service.ts
import Stripe from 'stripe';
import { Config } from '../config/schema';

export class StripeService {
  private stripe: Stripe;

  constructor(config: Pick<Config, 'STRIPE_API_KEY'>) {
    this.stripe = new Stripe(config.STRIPE_API_KEY, {
      apiVersion: '2024-06-20'
    });
  }

  async createPayment(amount: number, currency: string) {
    return this.stripe.paymentIntents.create({ amount, currency });
  }
}

// Composition root
const stripeService = new StripeService({
  STRIPE_API_KEY: config.STRIPE_API_KEY
});
```

**Expected Output (when `STRIPE_API_KEY` is missing):**
```
{ STRIPE_API_KEY: [ 'Required' ] }
(process exits with code 1)
```

**Why this output:** The config module validates `STRIPE_API_KEY` at startup. If missing, the application fails fast with a descriptive error. The `StripeService` receives the validated key via constructor injection — it never touches `process.env`.

### Real-World Cases

- **Database connections:** `PostgresRepository` receives `DATABASE_URL` via config injection.
- **Payment SDKs:** `StripeService` receives `STRIPE_API_KEY` via config injection.
- **Email providers:** `SendGridService` receives `SENDGRID_API_KEY` and `EMAIL_FROM`.
- **Cloud storage:** `S3Service` receives `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `S3_BUCKET`.

---

## Core Concept 4: Testing Dependencies

### Definitions

**Core Definition:** Testing dependencies is the practice of replacing real services or database clients with mock or stub instances during unit testing, made possible by dependency injection.

**Technical Definition:** Because dependencies are injected rather than hardcoded, tests can substitute real implementations with test doubles — mocks (verify interactions), stubs (return canned data), and fakes (simplified working implementations) . This allows the class under test to be exercised in isolation, without touching real databases, networks, or external services. In NestJS, the testing module provides the DI system in the test environment for easily mocking components .

**Beginner-Friendly Explanation:** Testing with DI is like rehearsing a play with understudies. The main actors (real services) are replaced by understudies (mocks) who deliver the same lines but in a controlled environment. The director (the test) can verify that every actor said the right thing at the right time, without worrying about the real actors' availability.

### Purposes

- To mock, stub, and swap real services or database clients with mock instances during unit testing.
- To isolate the class under test from external dependencies (databases, APIs, file systems).
- To verify that the correct dependencies were called with the correct arguments.
- To run tests quickly and reliably without real infrastructure.

### Syntax Rules and Structure

```typescript
// services/user.service.test.ts
import { UserService } from './user.service';
import { UserRepository } from '../repositories/user.repository';

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    // Create a mock repository
    mockUserRepository = {
      findById: jest.fn(),
      findByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      findAll: jest.fn()
    } as unknown as jest.Mocked<UserRepository>;

    // Inject the mock into the service
    userService = new UserService(mockUserRepository);
  });

  it('should create a user and return it', async () => {
    const newUser = { id: '1', name: 'Alice', email: 'alice@example.com' };
    mockUserRepository.create.mockResolvedValue(newUser);

    const result = await userService.create({ name: 'Alice', email: 'alice@example.com' });

    expect(result).toEqual(newUser);
    expect(mockUserRepository.create).toHaveBeenCalledWith({
      name: 'Alice',
      email: 'alice@example.com'
    });
  });

  it('should throw if email already exists', async () => {
    mockUserRepository.findByEmail.mockResolvedValue({ id: '1', email: 'alice@example.com' });

    await expect(
      userService.create({ name: 'Alice', email: 'alice@example.com' })
    ).rejects.toThrow('Email already exists');
  });
});
```

| Component | Breakdown |
|-----------|-----------|
| `jest.Mocked<UserRepository>` | A typed mock of the repository. |
| `mockResolvedValue()` | Configures the mock to return a value. |
| `mockUserRepository.create` | The mock method that was called. |
| `expect().toHaveBeenCalledWith()` | Verifies the mock was called with specific arguments. |

#### Syntax Rules

- Tests should **inject mocks** into the class under test, not use `new` on real dependencies.
- Mocks should be typed using the same interface as the real dependency.
- Each test should set up its own mock behaviour in `beforeEach` or within the test.
- Tests should verify both the return value and the interactions with dependencies.

#### Constraints and Limitations

- Over-mocking can lead to tests that pass even when the real integration is broken.
- Mocks must be kept in sync with the real interface — changes to the interface break tests.
- Integration tests (with real dependencies) are still needed to verify end-to-end behaviour.

### Annotated Code Example

```typescript
// repositories/user.repository.ts (interface)
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  create(data: CreateUserDto): Promise<User>;
}

// services/user.service.ts
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return user;
  }
}

// __tests__/user.service.test.ts
describe('UserService', () => {
  it('should return a user when found', async () => {
    const mockRepo: UserRepository = {
      findById: jest.fn().mockResolvedValue({ id: '1', name: 'Alice' }),
      create: jest.fn()
    };
    const service = new UserService(mockRepo);

    const user = await service.getUser('1');

    expect(user.name).toBe('Alice');
    expect(mockRepo.findById).toHaveBeenCalledWith('1');
  });
});
```

**Expected Output (test passes):**
```
PASS  __tests__/user.service.test.ts
  UserService
    ✓ should return a user when found
```

**Why this output:** The test injects a mock `UserRepository` into `UserService`. The mock returns a canned user. The test verifies both the returned value and that `findById` was called with the correct ID. No real database is involved .

### Real-World Cases

- **CI/CD pipelines:** Tests run quickly without needing database or network access.
- **TDD workflows:** Developers write tests before implementation, using mocks to define expected behaviour.
- **Regression testing:** Mocks ensure that refactoring services does not break tests.
- **Isolation testing:** Testing a service in complete isolation from its dependencies.

---

## Core Concept 5: Inversion of Control Containers

### Definitions

**Core Definition:** An IoC container is an automated dependency resolution system that manages object creation, wiring, and lifetimes based on registration metadata — eliminating the need for manual composition root wiring.

**Technical Definition:** Libraries like InversifyJS or Awilix add lifecycle management and lazy resolution when the dependency graph gets large enough that manual wiring becomes unwieldy . Awilix takes a functional approach — no decorators, no reflect-metadata, just a container that auto-wires by naming convention. InversifyJS is decorator-based (`@injectable`, `@inject`), familiar to developers from NestJS or Spring . NestJS provides a built-in DI container with `@Injectable()` decorators and module-based registration .

**Beginner-Friendly Explanation:** A container is like a warehouse manager who knows every item in stock, who needs what, and how long each item should last. Instead of you running around collecting parts, you just say "I need a UserService" and the warehouse manager assembles it, including all its dependencies, and hands it to you fully ready.

### Purposes

- To automate dependency resolution for large applications where manual wiring becomes complex.
- To manage object lifetimes (singleton, scoped, transient) automatically.
- To provide a single place for registering and resolving all dependencies.
- To reduce boilerplate compared to manual composition root configuration.

### Sub-Feature 5.1: Awilix (Functional Container)

#### Syntax Rules and Structure

```typescript
// container.ts — composition root
import { createContainer, asClass, asValue, InjectionMode } from 'awilix';

const container = createContainer({
  injectionMode: InjectionMode.PROXY   // Auto-wire by constructor param name
});

container.register({
  userRepository: asClass(UserRepository).singleton(),
  userService: asClass(UserService).scoped(),
  config: asValue(config)
});

// Resolve
const userService = container.resolve<UserService>('userService');
```


| Component | Breakdown |
|-----------|-----------|
| `InjectionMode.PROXY` | Auto-wires by matching constructor parameter names to registered tokens. |
| `asClass(UserRepository)` | Registers a class as a dependency. |
| `.singleton()` | One instance for the entire application. |
| `.scoped()` | One instance per scope. |
| `asValue(config)` | Registers a pre-built value. |

#### Syntax Rules

- One container should be built once at a single composition root.
- Class constructors declare dependencies as a destructured object: `constructor({ userRepository, logger }) {}`.
- Awilix matches parameter names to registered token names.
- A dependency must live at least as long as its consumer (`singleton ≥ scoped`) .

---

### Sub-Feature 5.2: InversifyJS (Decorator-Based Container)

#### Syntax Rules and Structure

```typescript
// inversify.config.ts
import 'reflect-metadata';
import { Container } from 'inversify';
import { TYPES } from './types';

const container = new Container();

container.bind<UserRepository>(TYPES.UserRepository).to(PostgresUserRepository);
container.bind<UserService>(TYPES.UserService).to(UserService);

// services/user.service.ts
import { injectable, inject } from 'inversify';

@injectable()
export class UserService {
  constructor(
    @inject(TYPES.UserRepository) private readonly userRepository: UserRepository
  ) {}
}
```


| Component | Breakdown |
|-----------|-----------|
| `@injectable()` | Marks a class as available for injection. |
| `@inject(TYPES.UserRepository)` | Specifies which binding to inject. |
| `container.bind().to()` | Registers a binding. |
| `TYPES` | Symbol-based identifiers (recommended over strings). |

#### Syntax Rules

- `reflect-metadata` must be imported **before** any decorators.
- `experimentalDecorators` and `emitDecoratorMetadata` must be enabled in `tsconfig.json` .
- Use `Symbol.for()` tokens instead of string tokens for type safety.
- Bindings can be `inSingletonScope()`, `inTransientScope()`, or `inRequestScope()`.

---

### Sub-Feature 5.3: NestJS (Framework-Integrated Container)

#### Syntax Rules and Structure

```typescript
// cats/cats.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  findAll(): Cat[] { return this.cats; }
}

// cats/cats.controller.ts
@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}
  @Get()
  async findAll(): Promise<Cat[]> { return this.catsService.findAll(); }
}

// app.module.ts
@Module({
  controllers: [CatsController],
  providers: [CatsService]
})
export class AppModule {}
```


| Component | Breakdown |
|-----------|-----------|
| `@Injectable()` | Marks a class as a provider that the Nest IoC container can manage. |
| `constructor(private catsService: CatsService)` | Constructor injection declares the dependency. |
| `providers: [CatsService]` | Associates the token with the class. |
| Default scope | Singleton (one instance per application). |

#### Syntax Rules

- `@Injectable()` must be applied to any class that will be injected.
- Providers are registered in the `providers` array of a module.
- Default scope is **SINGLETON**; use `@Injectable({ scope: Scope.REQUEST })` for request-scoped providers.
- The Nest IoC container instantiates the controller and resolves its dependencies automatically .

### Annotated Code Example (Manual Composition Root vs. Container)

```typescript
// ❌ MANUAL WIRING (composition root) — fine for small apps
const userRepository = new PostgresUserRepository(config.DATABASE_URL);
const emailService = new SmtpEmailService(config.SMTP_HOST);
const userService = new UserService(userRepository, emailService);
const authService = new AuthService(userRepository, emailService, tokenService);
// As the graph grows, this becomes unwieldy

// ✅ CONTAINER-BASED (Awilix) — scales for large apps
import { createContainer, asClass, asValue, InjectionMode } from 'awilix';

const container = createContainer({ injectionMode: InjectionMode.PROXY });

container.register({
  config: asValue(config),
  userRepository: asClass(PostgresUserRepository).singleton(),
  emailService: asClass(SmtpEmailService).singleton(),
  userService: asClass(UserService).scoped(),
  authService: asClass(AuthService).scoped()
});

// Resolve — Awilix injects all dependencies automatically
const authService = container.resolve<AuthService>('authService');
```

**Expected Output (when resolving `authService`):**
```
AuthService {
  userRepository: PostgresUserRepository { ... },
  emailService: SmtpEmailService { ... },
  tokenService: TokenService { ... }
}
```

**Why this output:** The container automatically resolves `AuthService` and all its dependencies (`UserRepository`, `EmailService`, `TokenService`) by matching constructor parameter names to registered tokens. No manual wiring is needed — the container handles the entire object graph .

### Real-World Cases

- **Large Express APIs:** Awilix for functional DI without decorators.
- **TypeScript-first teams:** InversifyJS for decorator-based DI with strong typing.
- **Full framework adoption:** NestJS provides DI out of the box, eliminating container setup.
- **Microservices:** Each service uses a container to manage its own dependency graph.

---

## References

- Compile-N-Run — Express Dependency Injection — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/15-express-advanced-patterns/3-express-dependency-injection.mdx
- Dependency Injection Patterns in Express.js — CosmicLearn — https://mail.cosmiclearn.com/expressjs/dependency-injection.php
- Skriva solid, underhållbar, löst kopplad kod (SOLID, IoC, DI) — https://gitlab.lnu.se/1dv027/content/coursesite/-/raw/ad8edd490ed6a66749e336fb2701911948229d80/content/webbteknik/resurser/SOLID.pdf
- InversifyJS vs Awilix vs TSyringe 2026 — PkgPulse — https://www.pkgpulse.com/guides/inversifyjs-vs-awilix-vs-tsyringe-dependency-injection-2026
- Dependency Injection Agent Skill — InugamiDev/ultrathink-oss — https://skillsmp.com/creators/inugamidev/ultrathink-oss/claude-skills-dependency-injection
- CitrineOS — Dependency Injection (Awilix) — https://raw.githubusercontent.com/citrineos/citrineos-core/362adf1195349267fbbc98263f9cd14854ace499/apps/ocpp-server/DEPENDENCY_INJECTION.md
- NestJS — Custom Providers — https://docs.nestjs.com/fundamentals/custom-providers
- NestJS — Providers — https://docs.nestjs.com/providers
- InversifyJS — Official Framework — http://inversify.io/framework/
- Awilix — npm — https://www.npmjs.com/package/awilix
- InversifyJS — npm — https://www.npmjs.com/package/inversify
- TSyringe — npm — https://www.npmjs.com/package/tsyringe
- NestJS — Unit Testing — https://docs.nestjs.com/fundamentals/unit-testing
- The Art of Unit Testing, Third Edition (JavaScript) — https://books.google.com.sg
- Dependency Injection in JavaScript: Security Guide — safeguard.sh — https://safeguard.sh
- Inversion of Control in JavaScript: Security Guide — safeguard.sh — https://safeguard.sh