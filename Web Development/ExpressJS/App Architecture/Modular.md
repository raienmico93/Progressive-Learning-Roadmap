# Modular Architecture (Feature-Driven Design) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Modular architecture (also called feature-driven or feature-first design) is an organisational pattern that structures code by business features instead of technical layers. Each feature is a self-contained unit that includes all the logic, steps, and tasks needed to handle a specific business capability .

**Technical Definition:** Feature-First Architecture structures the codebase around vertical slices of business functionality rather than horizontal technical layers. Instead of grouping files by their type (controllers, services, models, routes), files are grouped by feature, so that each feature module contains its own routes, controllers, services, repositories, validations, and tests. This approach applies the principle of locality of behaviour: related code lives together, making the system easier to reason about, test, and modify .

**Beginner-Friendly Explanation:** Imagine organising a toolbox by task rather than by tool type. Instead of having one drawer for all screwdrivers and another for all hammers, you have a "hanging a picture" box that contains the screwdriver, the hammer, the nails, and the picture hook — everything you need for that one job. Feature-driven design does the same for code: everything related to "users" lives in the `/users` folder, and everything related to "orders" lives in the `/orders` folder.

### Key Characteristics

- **Vertical slicing:** Code is organised by business feature, not by technical layer .
- **Self-contained modules:** Each feature module owns its routes, controllers, services, validations, and tests .
- **Locality of behaviour:** Related code lives together, reducing the need to jump between directories .
- **Independent testability:** Each feature can be tested in isolation .
- **Team scalability:** Different developers can work on different features with minimal merge conflicts .
- **Framework independence:** Domain modules encapsulate enterprise business rules independent of Express or any other framework .

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, modules, and asynchronous programming.
- **Understanding of Express routing and middleware.**
- **Familiarity with a validation library** (Zod, Joi) for configuration validation.

### Related Programming Areas

- **Domain-Driven Design (DDD):** Feature modules map to bounded contexts and domain modules encapsulate enterprise business rules .
- **Layered Architecture:** Feature-driven design is an evolution of layered architecture, organising the same components in a more modular way .
- **Modular Monolith:** A single deployable application organised into independent, loosely coupled feature modules .
- **Microservices readiness:** Feature modules can be extracted into standalone microservices with minimal refactoring.
- **Dependency Injection:** Services and repositories within modules receive dependencies through DI, enabling testability .

### Core Concepts

1. **Feature Modules** — encapsulating files by domain context rather than technical roles.
2. **Domain Modules** — managing core enterprise business rules independent of framework details.
3. **Shared Modules & Common Utilities** — structuring cross-cutting tools (error classes, token processors, crypto helpers).
4. **Configuration Modules** — centralised environment variable orchestration and strict schema validation.

---

## Core Concept 1: Feature Modules

### Definitions

**Core Definition:** A feature module is a self-contained directory that encapsulates everything related to a single business capability — routes, controllers, services, validations, and tests — organised by domain context rather than technical role.

**Technical Definition:** Feature modules structure the codebase around vertical slices of business functionality. Each module folder contains its own `*.routes.ts` (API endpoints), `*.controller.ts` (HTTP request handling), `*.service.ts` (business logic), `*.validation.ts` (Zod/Joi schemas), and `__tests__/` directory. The module exports a router that is mounted on the main application at a feature-specific prefix .

**Beginner-Friendly Explanation:** A feature module is like a dedicated department in a company. The "Users" department has its own manager (controller), its own workers (services), its own filing system (repository), and its own quality checks (tests). When you need to change something about users, you go to one place — the `/users` folder — and everything you need is right there.

### Purposes

- To encapsulate files by domain context rather than technical roles.
- To ensure locality of behaviour — related code lives together .
- To enable independent testing of each feature .
- To reduce merge conflicts when multiple developers work on different features .
- To make the codebase easier to navigate and scale .

### Syntax Rules and Structure

#### Recommended Feature Module Structure

```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.routes.ts        # API endpoints
│   │   ├── auth.controller.ts    # HTTP handling
│   │   ├── auth.service.ts       # Business logic
│   │   ├── auth.validation.ts    # Zod schemas
│   │   └── __tests__/
│   │       └── auth.test.ts
│   ├── users/
│   │   ├── users.routes.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.validation.ts
│   │   └── __tests__/
│   └── orders/
│       ├── orders.routes.ts
│       ├── orders.controller.ts
│       ├── orders.service.ts
│       ├── orders.validation.ts
│       └── __tests__/
├── middleware/
├── config/
├── utils/
├── app.ts
└── index.ts
```

#### Module Component Breakdown

| File | Responsibility |
|------|---------------|
| `*.routes.ts` | Defines API endpoints and maps them to controller functions. |
| `*.controller.ts` | Handles HTTP requests, validates input, calls services. |
| `*.service.ts` | Contains core business logic; interacts with the database. |
| `*.validation.ts` | Defines Zod/Joi schemas for request validation. |
| `__tests__/` | Unit and integration tests for the feature. |

#### Syntax Rules

- Each feature should have its own module directory under `src/modules/`.
- The module must export a router for mounting on the main application.
- Routes within a module should be relative to the module's mount path (e.g., `/` becomes `/users` when mounted at `/users`).
- Tests should live inside the feature module's `__tests__/` directory, keeping them close to the code they test .

#### Constraints and Limitations

- Cross-feature dependencies should be minimised — if module A needs data from module B, use a shared service or event bus.
- Over-modularisation (too many tiny modules) can increase complexity without benefit.

### Annotated Code Example
`modules/users/users.routes.ts`
```typescript
import { Router } from 'express';
import { UserController } from './users.controller';
import { validate } from '../../middleware/validate';
import { createUserSchema } from './users.validation';

const router = Router();
const controller = new UserController();

router.get('/', controller.getAll);
router.get('/:id', controller.getById);
router.post('/', validate(createUserSchema), controller.create);

export default router;
```
\
`modules/users/users.controller.ts`
```typescript
import { Request, Response, NextFunction } from 'express';
import { UserService } from './users.service';

export class UserController {
  private userService = new UserService();

  getAll = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const users = await this.userService.findAll();
      res.json({ data: users });
    } catch (error) {
      next(error);
    }
  };

  getById = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const user = await this.userService.findById(req.params.id);
      if (!user) return res.status(404).json({ error: 'User not found' });
      res.json({ data: user });
    } catch (error) {
      next(error);
    }
  };

  create = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const user = await this.userService.create(req.body);
      res.status(201).json({ data: user });
    } catch (error) {
      next(error);
    }
  };
}
```
\
`modules/users/users.service.ts`
```typescript
import { UserRepository } from './users.repository';
import { hashPassword } from '../../shared/crypto';

export class UserService {
  private userRepository = new UserRepository();

  async findAll() {
    return this.userRepository.findAll();
  }

  async findById(id: string) {
    return this.userRepository.findById(id);
  }

  async create(data: CreateUserDto) {
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) throw new AppError('Email already exists', 409);

    const hashedPassword = await hashPassword(data.password);
    return this.userRepository.create({ ...data, password: hashedPassword });
  }
}
```

**Expected Output (for `GET /api/users`):**
```json
{
  "data": [
    { "id": "1", "name": "Alice", "email": "alice@example.com" }
  ]
}
```

**Why this output:** The feature module encapsulates all user-related logic. The route file defines the HTTP surface, the controller handles request/response, and the service contains business logic (email uniqueness check, password hashing). Everything is co-located in the `/users` directory.

### Real-World Cases

- **SaaS platforms:** `modules/auth/`, `modules/billing/`, `modules/reports/` — each feature is self-contained .
- **E-commerce:** `modules/products/`, `modules/orders/`, `modules/payments/`.
- **Content platforms:** `modules/posts/`, `modules/comments/`, `modules/media/`.

---

## Core Concept 2: Domain Modules

### Definitions

**Core Definition:** Domain modules encapsulate the core enterprise business rules and domain logic of the application, independent of any framework, database, or delivery mechanism.

**Technical Definition:** Domain modules correspond to the entity layer of Clean Architecture and the domain modules of Domain-Driven Design (DDD). They contain domain entities (objects with identity), value objects (immutable descriptive objects), aggregates (clusters of entities treated as a unit), and domain exceptions (e.g., `BusinessRuleViolationException`, `EntityNotFoundException`). Domain modules are framework-agnostic — they do not import Express, ORM libraries, or any infrastructure concerns .

**Beginner-Friendly Explanation:** Domain modules are the "brain" of your application — they contain the rules that make your business unique. For example, a banking app's domain module knows that "a withdrawal cannot exceed the account balance" — that rule has nothing to do with Express, databases, or HTTP. It's pure business logic that would be the same whether you're building a web app, a mobile app, or a CLI tool.

### Purposes

- To manage core enterprise business rules independent of framework details.
- To encapsulate domain entities, value objects, and aggregates.
- To enforce domain invariants and business rule violations.
- To provide a framework-agnostic core that can be tested without infrastructure.

### Syntax Rules and Structure

```
src/
├── domain/
│   ├── entities/
│   │   ├── User.ts           # Entity with identity
│   │   └── Order.ts
│   ├── value-objects/
│   │   ├── Email.ts          # Immutable descriptive object
│   │   └── Money.ts
│   ├── exceptions/
│   │   ├── DomainException.ts
│   │   ├── BusinessRuleViolationException.ts
│   │   └── EntityNotFoundException.ts
│   └── services/
│       └── PricingService.ts  # Domain service (stateless)
```

| Component | Description |
|-----------|-------------|
| Entities | Objects with a distinct identity (e.g., `User`, `Order`). |
| Value Objects | Immutable objects defined by attributes (e.g., `Email`, `Money`). |
| Domain Exceptions | Typed errors for business rule violations. |
| Domain Services | Stateless operations that don't belong to a single entity. |

#### Syntax Rules

- Domain modules must **not** import Express, ORMs, or any infrastructure library.
- Domain entities should enforce their own invariants in constructors or factory methods.
- Domain exceptions should be typed and carry business meaning .
- Domain services should be stateless and receive their dependencies via method arguments.

#### Constraints and Limitations

- Domain modules should not know about HTTP status codes — those are translated by controllers.
- Overly complex domain models may indicate a need for further decomposition.

### Annotated Code Example

```typescript
// domain/entities/Order.ts
import { BusinessRuleViolationException } from '../exceptions/BusinessRuleViolationException';
import { Money } from '../value-objects/Money';

export class Order {
  private items: OrderItem[] = [];
  private status: 'pending' | 'paid' | 'shipped' = 'pending';

  addItem(item: OrderItem): void {
    if (this.status !== 'pending') {
      throw new BusinessRuleViolationException(
        'Cannot add items to a non-pending order'
      );
    }
    this.items.push(item);
  }

  markAsPaid(): void {
    if (this.items.length === 0) {
      throw new BusinessRuleViolationException(
        'Cannot pay for an empty order'
      );
    }
    this.status = 'paid';
  }

  getTotal(): Money {
    return this.items.reduce(
      (sum, item) => sum.add(item.price.multiply(item.quantity)),
      Money.zero()
    );
  }
}
```

**Expected Output (when attempting to pay for an empty order):**
```
BusinessRuleViolationException: Cannot pay for an empty order
```

**Why this output:** The `Order` entity enforces its own business rule — you cannot pay for an order with no items. This rule is enforced in the domain layer, independent of any controller or database. The exception carries business meaning (`BusinessRuleViolationException`) that the controller can translate to an HTTP 409 Conflict.

### Real-World Cases

- **Banking:** `Account` entity enforcing overdraft rules, `Money` value object preventing negative amounts.
- **E-commerce:** `Order` aggregate enforcing checkout rules, `Discount` value object with validation.
- **Healthcare:** `Patient` entity with medical record rules, `Prescription` value object with dosage validation.

---

## Core Concept 3: Shared Modules & Common Utilities

### Definitions

**Core Definition:** Shared modules contain cross-cutting tools and utilities that are used by multiple feature modules but do not belong to any single feature — such as custom error classes, JWT token processors, cryptographic helpers, and logging utilities.

**Technical Definition:** The shared modules directory (often called `shared/`, `common/`, or `utils/`) contains reusable code that is framework-agnostic and feature-independent. This includes custom error classes (`AppError`, `NotFoundError`, `ValidationError`), token processors for JWT signing and verification, cryptographic helpers (password hashing, encryption), and utility functions (date formatting, string manipulation). Shared modules should have no dependencies on feature modules .

**Beginner-Friendly Explanation:** Shared modules are like the office supplies cupboard that everyone in the company uses — pens, staplers, printer paper. They're not specific to any one department, but every department needs them. In code, the shared folder contains things like the error class that every feature uses, the password hashing function, and the logger.

### Purposes

- To structure cross-cutting tools (error classes, token processors, cryptographic helpers).
- To avoid duplicating utility code across feature modules.
- To provide a single source of truth for shared logic.
- To enable consistent error handling across all features.

### Syntax Rules and Structure

```
src/
├── shared/
│   ├── errors/
│   │   ├── AppError.ts           # Base error class
│   │   ├── NotFoundError.ts
│   │   └── ValidationError.ts
│   ├── crypto/
│   │   ├── hash.ts               # bcrypt/argon2 wrappers
│   │   └── encrypt.ts            # AES-256 encryption
│   ├── token/
│   │   └── jwt.ts                # JWT sign/verify/parse
│   ├── logger.ts                 # Winston/Pino logger
│   └── utils/
│       └── date.ts               # Date formatting helpers
```

#### Component Breakdown

| Component | Responsibility |
|-----------|---------------|
| `AppError` | Base error class with `statusCode` and `isOperational`. |
| `hash.ts` | Password hashing and verification. |
| `jwt.ts` | Token generation, verification, and parsing. |
| `logger.ts` | Structured logging for the application. |

#### Syntax Rules

- Shared modules must **not** import from feature modules — dependencies flow one way.
- Each utility should have a single, clear responsibility.
- Error classes should extend a common base (`AppError`) with `statusCode` and `isOperational` properties .
- Token processors should never expose secrets in logs or error messages.

#### Constraints and Limitations

- Over-stuffing shared modules with feature-specific logic defeats the purpose of modularity.
- Shared modules should be stable — frequent changes ripple across all features.

### Annotated Code Example

```typescript
// shared/errors/AppError.ts
export class AppError extends Error {
  public readonly statusCode: number;
  public readonly isOperational: boolean;

  constructor(message: string, statusCode: number, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public readonly details?: unknown) {
    super(message, 422);
  }
}
```

```typescript
// shared/token/jwt.ts
import jwt from 'jsonwebtoken';

const ACCESS_SECRET = process.env.JWT_SECRET!;
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;

export function signAccessToken(payload: object): string {
  return jwt.sign(payload, ACCESS_SECRET, { expiresIn: '15m' });
}

export function signRefreshToken(payload: object): string {
  return jwt.sign(payload, REFRESH_SECRET, { expiresIn: '7d' });
}

export function verifyAccessToken(token: string): object {
  return jwt.verify(token, ACCESS_SECRET) as object;
}
```

```typescript
// shared/crypto/hash.ts
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

export async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return bcrypt.compare(password, hash);
}
```

**Expected Output (when a feature module throws a `NotFoundError`):**
```json
{
  "error": "User not found",
  "statusCode": 404
}
```

**Why this output:** The shared `NotFoundError` class extends `AppError` with a 404 status code. The error-handling middleware checks `err.statusCode` and `err.isOperational` to format the response consistently across all features.

### Real-World Cases

- **All features:** Every feature module imports `AppError` for consistent error handling .
- **Auth module:** Imports `hashPassword` and `signAccessToken` from shared.
- **Users module:** Imports `verifyPassword` for password verification.
- **All modules:** Import `logger` for structured logging.

---

## Core Concept 4: Configuration Modules

### Definitions

**Core Definition:** A configuration module centralises environment variable loading, validates all required variables against a strict schema at startup, and exports a single typed configuration object for use throughout the application.

**Technical Definition:** The configuration module uses `dotenv` to load environment variables from `.env` files and Zod to validate them against a schema. If any required variable is missing or invalid, the application crashes at startup with a descriptive error — this is the "fail fast" principle. The validated configuration is exported as a single typed object, and all other modules import from this object instead of accessing `process.env` directly .

**Beginner-Friendly Explanation:** The configuration module is like a checklist at the start of a flight. Before the plane takes off (the app starts), the pilot (the config module) checks that all required systems are working — fuel, engine, navigation. If something is missing, the flight doesn't take off. This prevents "mid-flight" failures caused by missing configuration.

### Purposes

- To centralise environment variable orchestration.
- To validate all required variables at startup (fail fast).
- To provide a single typed configuration object.
- To support multi-environment management (development, staging, production).

### Syntax Rules and Structure

```
src/
├── config/
│   ├── index.ts          # Main config export
│   ├── schema.ts         # Zod validation schema
│   └── .env.example      # Template for required variables
```

#### Zod Schema Example

```typescript
// config/schema.ts
import { z } from 'zod';

export const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.string().pipe(z.coerce.number().int().min(1).max(65535)).default('3000'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
  JWT_REFRESH_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
  CORS_ORIGINS: z.string()
    .default('http://localhost:3000')
    .transform((s) => s.split(',').map((o) => o.trim()))
});

export type EnvConfig = z.infer<typeof envSchema>;
```

#### Config Export

```typescript
// config/index.ts
import 'dotenv/config';
import { envSchema } from './schema';

const parseResult = envSchema.safeParse(process.env);

if (!parseResult.success) {
  console.error('Invalid configuration:');
  console.error(parseResult.error.flatten().fieldErrors);
  process.exit(1); // Fail fast
}

export const config = parseResult.data;
```

#### Syntax Rules

- The config module must be imported at the top of the application entry point.
- Validation must use `safeParse` (not `parse`) to handle errors gracefully.
- On validation failure, log descriptive errors and exit with a non-zero code.
- All other modules must import from `config/` — never access `process.env` directly .
- `.env.example` should list every required variable with comments.

#### Constraints and Limitations

- Environment variables are always strings; Zod coercion (`z.coerce.number()`) is required for numeric values .
- Boolean coercion must be explicit — `"false"` is truthy in JavaScript .
- Secrets should never be logged or committed to version control.

### Annotated Code Example

```typescript
// config/schema.ts
import { z } from 'zod';

export const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url().optional(),
  JWT_SECRET: z.string().min(32),
  JWT_ACCESS_EXPIRE: z.string().default('15m'),
  JWT_REFRESH_SECRET: z.string().min(32),
  JWT_REFRESH_EXPIRE: z.string().default('7d'),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
  CORS_ORIGINS: z.string()
    .default('http://localhost:3000')
    .transform((s) => s.split(',').map((o) => o.trim())),
  RATE_LIMIT_WINDOW_MS: z.coerce.number().int().positive().default(60000),
  RATE_LIMIT_MAX: z.coerce.number().int().positive().default(100)
});
```

```typescript
// config/index.ts
import 'dotenv/config';
import { envSchema } from './schema';

const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('ERROR: Invalid environment configuration');
  console.error(JSON.stringify(parsed.error.flatten().fieldErrors, null, 2));
  process.exit(1);
}

export const config = parsed.data;
```

```typescript
// app.ts — import config at the top
import { config } from './config';
import express from 'express';

const app = express();

app.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT} in ${config.NODE_ENV} mode`);
});
```

**Expected Output (when `JWT_SECRET` is missing):**
```
ERROR: Invalid environment configuration
{
  "JWT_SECRET": ["Required"],
  "JWT_REFRESH_SECRET": ["Required"]
}
(process exits with code 1)
```

**Expected Output (when all variables are valid):**
```
Server running on port 3000 in development mode
```

**Why this output:** The config module validates all environment variables at startup using Zod. If any are missing or invalid, it prints descriptive errors and exits with code 1 — the application never starts with an invalid configuration. If all variables are valid, the typed config object is exported and used throughout the application.

### Real-World Cases

- **All Node.js applications:** Centralised config with Zod validation is a best practice for any production application .
- **Multi-environment deployments:** Separate `.env` files for development, staging, and production.
- **Secrets management:** Integration with AWS Secrets Manager or HashiCorp Vault for production secrets .

---

## References

- express-numflow — Feature-First Architecture Guide — https://github.com/gazerkr/express-numflow/blob/HEAD/docs/feature-first-architecture.md
- Prism AI — API Backend Architecture (Feature-Based) — https://raw.githubusercontent.com/precious112/prism-ai-deep-research/refs/heads/main/docs/02-architecture/02-api-backend.md
- Configuration Validation Utilities — https://raw.githubusercontent.com/vasilyu1983/AI-Agents-public/3424d6f5e94010409da012eeb1cb84aaecec88b3/frameworks/shared-skills/skills/software-clean-code-standard/references/config-validation.md
- How to Build a Configuration System in Node.js — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-configuration-system-nodejs/README.md
- Abdullah Niaz — Layer-Based vs Feature-Based Architecture — https://www.linkedin.com/posts/abdullah-niaz_nodejs-expressjs-backend-activity-7464242464715956225-9zHo
- DDD Boilerplate — Domain Modules (Clean Architecture) — https://github.com/guilhermelim/ddd-boilerplate
- node-clean-domain — Independent Domain Layer — https://github.com/joschonarth/node-clean-domain
- Global Error Types in Express — Steve Kinney — https://stevekinney.com
- Typed-Error Middleware with Express — https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run
- express-modularity — npm — https://www.npmjs.com/package/express-modularity
- @nestjs/config — Configuration Module — https://docs.nestjs.com/techniques/configuration
- NestJS 中文文档 — ConfigModule — https://docs.nestjs.cn
- Zod Documentation — https://zod.dev
- dotenv Documentation — https://github.com/motdotla/dotenv