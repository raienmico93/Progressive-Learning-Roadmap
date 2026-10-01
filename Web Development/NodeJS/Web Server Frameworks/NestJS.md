# NestJS: Enterprise Architecture & Dependency Injection — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications, built around Angular-inspired architectural patterns including modules, controllers, providers, and a powerful dependency injection container.

**Technical Definition:** NestJS is a framework for building efficient, scalable Node.js server-side applications. It uses modern JavaScript, is built with TypeScript (preserving compatibility with pure JavaScript), and combines elements of Object Oriented Programming (OOP), Functional Programming (FP), and Functional Reactive Programming (FRP). Under the hood, Nest makes use of robust HTTP Server frameworks like Express (the default) or Fastify. Nest provides a level of abstraction above these common Node.js frameworks but also exposes their APIs directly to the developer, giving developers the freedom to use the myriad of third-party modules available.

**Beginner-Friendly Explanation:** NestJS is like a well-organised factory for building backend applications. Instead of writing everything from scratch, you get a structured system with clear roles: modules group related code, controllers handle incoming requests, services contain business logic, and the dependency injection system automatically wires everything together. Think of it as a Lego set for server-side applications — each piece has a specific purpose, and they all snap together in a predictable way.

### Key Characteristics

- **Modular architecture:** Applications are composed of modules that encapsulate related controllers, providers, and imports.
- **Dependency Injection:** A powerful IoC (Inversion of Control) container manages the instantiation and injection of providers.
- **Decorator-based:** Metadata decorators (@Module, @Controller, @Injectable) declaratively define structure and behaviour.
- **Progressive disclosure:** Starts simple with Express-like handlers but scales to enterprise patterns with guards, interceptors, pipes, and filters.
- **TypeScript-first:** Built with TypeScript, providing strong typing and compile-time safety.
- **Adapter-agnostic:** Supports Express and Fastify as HTTP adapters, with WebSockets and microservices support.
- **Enterprise-ready:** Used by companies like Adidas, Autodesk, Roche, and others for large-scale applications.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended for NestJS 10+/11.
- **TypeScript knowledge:** Strong familiarity with TypeScript decorators, generics, and interfaces.
- **Basic JavaScript/Node.js:** Understanding of modules, asynchronous programming, and HTTP concepts.
- **Object-Oriented Programming concepts:** Classes, inheritance, interfaces, and dependency inversion.
- **HTTP fundamentals:** Request methods, headers, status codes, and REST conventions.

### Related Programming Areas

- **Dependency Injection:** IoC containers, provider scopes, and custom providers.
- **Aspect-Oriented Programming:** Guards, interceptors, and middleware for cross-cutting concerns.
- **Microservices:** NestJS supports message-based and event-based microservice transports.
- **GraphQL:** First-class support via @nestjs/graphql.
- **WebSockets:** Real-time communication via @nestjs/websockets.
- **Database integration:** TypeORM, Prisma, Mongoose, and other ORMs.

### Core Concepts

1. **Modular Architecture** — feature, core, shared, global, and dynamic modules.
2. **Controllers & Routing** — decorator-based REST endpoints and parameter extraction.
3. **Providers & Dependency Injection** — @Injectable(), custom providers, and injection scopes.
4. **The Request-Response Lifecycle Pipeline** — guards, interceptors, pipes, and exception filters.

---

## Core Concept 1: Modular Architecture

### Sub-Feature 1.1: Structuring Scalable Codebases with Feature, Core, Shared, and Global Modules

#### Definitions

**Core Definition:** A module is a class annotated with the `@Module()` decorator that provides metadata for organising the application structure, encapsulating related controllers, providers, and imports.

**Technical Definition:** A module is a class annotated with the `@Module()` decorator. The decorator provides metadata that Nest uses to organise and manage the application structure. Every Nest application has at least one module, the root module, which is the starting point from which Nest builds the application graph. A module encapsulates its providers by default; you can inject only providers that are part of the current module or that are explicitly exported by an imported module. The `@Module()` decorator takes a single object with properties: `providers`, `controllers`, `imports`, and `exports`. A module's exported providers form its public interface, or API.

**Beginner-Friendly Explanation:** A module is like a department in a company. Each department has its own staff (providers), its own customer-facing desk (controllers), and its own list of resources it needs from other departments (imports). The department also decides what it shares with other departments (exports). By keeping related things together, you avoid a tangled mess where everything depends on everything else.

#### Purposes

- To organise code into cohesive, reusable units based on domain or feature.
- To encapsulate providers and controllers, controlling what is shared externally.
- To define the application graph and resolve dependencies between modules.
- To enable scalable architecture as the application and team grow.

#### Syntax Rules and Structure

```typescript
@Module({
  imports: [OtherModule],
  controllers: [MyController],
  providers: [MyService],
  exports: [MyService],
})
export class MyModule {}
```

| Property | Description |
|----------|-------------|
| `imports` | Modules whose exported providers are needed in this module. |
| `controllers` | Controllers to instantiate in this module. |
| `providers` | Providers to instantiate and make available in this module. |
| `exports` | Subset of providers to make available to importing modules. |

**Module types:**

| Type | Purpose | Examples |
|------|---------|----------|
| Feature Module | Business domain features | UserModule, OrderModule |
| Shared Module | Reusable utilities | DatabaseModule, LoggerModule |
| Core Module | App-wide singletons | ConfigModule, AuthModule |
| Global Module | Providers available everywhere | @Global() DatabaseModule |

**Global modules:**
```typescript
@Global()
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}
```

**Constraints and Limitations:**
- Modules are singletons by default; every module is automatically a shared module.
- Global modules should be registered only once, generally by the root or core module.
- Importing a global module still requires the module to export the providers.
- Circular dependencies between modules should be avoided; use `forwardRef()` if unavoidable.

#### Annotated Code Example

```typescript
// cats/cats.module.ts — Feature module
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService], // Share with other modules
})
export class CatsModule {}
```

```typescript
// core/database.module.ts — Shared/Core module
import { Module, Global } from '@nestjs/common';
import { DatabaseService } from './database.service';

@Global() // Makes it available everywhere without importing
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}
```

```typescript
// app.module.ts — Root module
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';
import { DatabaseModule } from './core/database.module';

@Module({
  imports: [DatabaseModule, CatsModule],
})
export class AppModule {}
```

**Expected Output (application starts successfully):**
```
[Nest] 12345  - 01/15/2026, 12:00:00 PM     LOG [NestFactory] Starting Nest application...
[Nest] 12345  - 01/15/2026, 12:00:00 PM     LOG [InstanceLoader] DatabaseModule dependencies initialized
[Nest] 12345  - 01/15/2026, 12:00:00 PM     LOG [InstanceLoader] CatsModule dependencies initialized
[Nest] 12345  - 01/15/2026, 12:00:00 PM     LOG [RoutesResolver] CatsController {/cats}:
```

**Why this output:** The `DatabaseModule` is marked `@Global()`, so its `DatabaseService` is available everywhere without importing. The `CatsModule` exports `CatsService`, making it available to any module that imports `CatsModule`. The root `AppModule` imports both.

#### Real-World Cases

- **E-commerce:** UserModule, ProductModule, OrderModule, PaymentModule.
- **SaaS applications:** TenantModule, BillingModule, AuthModule.
- **Microservices:** Each service as a separate NestJS application with shared library modules.

---

### Sub-Feature 1.2: Dynamic Modules (`register()`, `forRoot()`, `forFeature()`) for Runtime Configurations

#### Definitions

**Core Definition:** Dynamic modules are modules created at runtime using static factory methods (conventionally `forRoot()`, `register()`, or `forFeature()`) that return a `DynamicModule` object, allowing configuration to be passed when the module is imported.

**Technical Definition:** A dynamic module is a module that is created at runtime, typically using a static method on the module class itself. The static method returns a `DynamicModule` object, which is essentially a module definition object with the same properties as `@Module()` metadata, plus a `module` property. The convention is to name the static method `forRoot()` (for global, single configuration), `register()` (for per-import configuration), or `forFeature()` (for extending a module with specific providers/entities). Dynamic modules follow the same encapsulation rules as static modules; they are just built at runtime.

**Beginner-Friendly Explanation:** A static module is like a pre-built furniture kit — it's the same for everyone. A dynamic module is like a custom furniture order — you specify the dimensions, colour, and material when you order it. `forRoot()` is the main configuration (like choosing the sofa size), `forFeature()` is for adding accessories (like throw pillows for a specific room), and `register()` is for per-instance configuration.

#### Purposes

- To allow general-purpose modules to be configured differently for each consumer.
- To pass configuration options at module import time.
- To register feature-specific providers (e.g., repositories) from a shared connection.
- To enable async configuration using `forRootAsync()` / `registerAsync()`.

#### Syntax Rules and Structure

```typescript
@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        { provide: 'CONFIG_OPTIONS', useValue: options },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

| Method | Purpose | Called |
|--------|---------|--------|
| `forRoot()` | Global configuration (DB, Config) | Once, in AppModule |
| `forRootAsync()` | Async global configuration | Once, in AppModule |
| `register()` | Per-instance configuration | Multiple times, anywhere |
| `registerAsync()` | Async per-instance configuration | Multiple times, anywhere |
| `forFeature()` | Feature-specific providers/entities | Per feature module |

**Constraints and Limitations:**
- Dynamic modules cannot be marked `@Global()` in the decorator; use the `global` property in the returned `DynamicModule` object.
- `forRoot()` should be called once (usually in the root module).
- `forFeature()` is typically used with ORMs like TypeORM to register entity repositories.

#### Annotated Code Example

```typescript
// config.module.ts — Dynamic module with forRoot
import { Module, DynamicModule } from '@nestjs/common';
import { ConfigService } from './config.service';

export interface ConfigOptions {
  env: string;
  port: number;
}

@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
      global: true, // Make it global
    };
  }
}
```

```typescript
// app.module.ts — Using forRoot
import { Module } from '@nestjs/common';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      env: process.env.NODE_ENV || 'development',
      port: 3000,
    }),
  ],
})
export class AppModule {}
```

```typescript
// users/users.module.ts — Using forFeature with TypeORM
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UsersService } from './users.service';

@Module({
  imports: [TypeOrmModule.forFeature([User])], // Register User repository
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

**Expected Output:**
```
Application starts with ConfigService available globally, and UserRepository available in UsersModule.
```

**Why this output:** `ConfigModule.forRoot()` returns a `DynamicModule` with the configuration options as a provider and `global: true`. `TypeOrmModule.forFeature([User])` creates and registers a `UserRepository` provider that can be injected into `UsersService`.

#### Real-World Cases

- **Database configuration:** `TypeOrmModule.forRoot({ host, port, ... })` for a single DB connection; `TypeOrmModule.forFeature([Entity])` for repositories.
- **Configuration management:** `ConfigModule.forRoot({ isGlobal: true })`.
- **Message queues:** `BullModule.forRoot({ redis: { ... } })`.
- **Multi-tenant:** `register()` for per-tenant configurations.

---

## Core Concept 2: Controllers & Routing

### Sub-Feature 2.1: Constructing RESTful Entrypoints Using Metadata Decorators

#### Definitions

**Core Definition:** Controllers are classes annotated with `@Controller()` that handle incoming HTTP requests and return responses, using method decorators (`@Get()`, `@Post()`, etc.) to define route handlers.

**Technical Definition:** Controllers are responsible for handling incoming requests and sending responses back to the client. The routing mechanism determines which controller handles each request. To create a basic controller, you use classes and decorators. Decorators associate classes with the required metadata, which Nest uses to build a routing map that connects requests to their corresponding controllers. The `@Controller()` decorator is required to define a basic controller, with an optional route path prefix.

**Beginner-Friendly Explanation:** A controller is like the reception desk of a building. When someone arrives (a request), the receptionist (controller) looks at what they want (the URL and HTTP method) and directs them to the right person (the handler method). The `@Controller('cats')` decorator is like putting a sign on the desk that says "Cat Department."

#### Purposes

- To define HTTP endpoints and map them to handler methods.
- To group related routes under a common path prefix.
- To handle incoming requests and return responses.
- To separate HTTP handling from business logic (which lives in services).

#### Syntax Rules and Structure

```typescript
@Controller('cats') // Path prefix
export class CatsController {
  @Get() // GET /cats
  findAll(): string { return 'All cats'; }

  @Get('breed') // GET /cats/breed
  findBreed(): string { return 'Breeds'; }

  @Post() // POST /cats
  create(): string { return 'Created'; }
}
```

| Decorator | HTTP Method | Route |
|-----------|-------------|-------|
| `@Get()` | GET | `/cats` |
| `@Post()` | POST | `/cats` |
| `@Put()` | PUT | `/cats` |
| `@Patch()` | PATCH | `/cats` |
| `@Delete()` | DELETE | `/cats` |
| `@Options()` | OPTIONS | `/cats` |
| `@Head()` | HEAD | `/cats` |
| `@All()` | All methods | `/cats` |

**Constraints and Limitations:**
- The method name is arbitrary; Nest attaches no significance to it.
- POST requests default to a 201 status code; all others default to 200.
- Use `@HttpCode()` to override the default status code.

#### Annotated Code Example

```typescript
// cats.controller.ts
import { Controller, Get, Post, Put, Delete, HttpCode, HttpStatus } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Get() // GET /cats
  findAll(): string {
    return this.catsService.findAll();
  }

  @Get(':id') // GET /cats/:id
  findOne(): string {
    return 'One cat';
  }

  @Post() // POST /cats (201 by default)
  create(): string {
    return 'Created';
  }

  @Put(':id') // PUT /cats/:id
  update(): string {
    return 'Updated';
  }

  @Delete(':id') // DELETE /cats/:id
  @HttpCode(HttpStatus.NO_CONTENT) // Override to 204
  remove(): void {}
}
```

**Expected Output (for `GET /cats`):**
```
All cats
```

**Expected Output (for `POST /cats`):**
```
201 Created
```

**Expected Output (for `DELETE /cats/42`):**
```
204 No Content
```

**Why this output:** The `@Controller('cats')` prefix combines with each method decorator to form the full route. `@Post()` defaults to 201, while `@HttpCode(HttpStatus.NO_CONTENT)` overrides `@Delete()` to 204.

#### Real-World Cases

- **REST APIs:** CRUD endpoints for resources (users, products, orders).
- **GraphQL resolvers:** Controllers for GraphQL queries and mutations.
- **Microservice message handlers:** Controllers using `@MessagePattern()`.

---

### Sub-Feature 2.2: Extracting Payloads with Execution Context Decorators

#### Definitions

**Core Definition:** NestJS provides parameter decorators (`@Body()`, `@Param()`, `@Query()`, `@Headers()`) that extract specific parts of the incoming request and inject them into handler method parameters.

**Technical Definition:** Nest provides decorators for extracting data from the request object: `@Request()` (or `@Req()`), `@Response()` (or `@Res()`), `@Next()`, `@Session()`, `@Param(key?)`, `@Body(key?)`, `@Query(key?)`, `@Headers(name?)`, `@Ip()`, `@HostParam()`. Each decorator can be used without arguments (to get the entire object) or with a string key (to get a specific property).

**Beginner-Friendly Explanation:** These decorators are like specific tools for unpacking a delivery package. `@Body()` gets the contents, `@Param()` gets the label on the box, `@Query()` gets the special instructions, and `@Headers()` gets the shipping information. Instead of manually digging through the request object, you just declare what you need.

#### Purposes

- To extract and inject specific parts of the request into handler methods.
- To avoid manual access to the raw request object.
- To enable type-safe request handling with DTOs.
- To simplify parameter extraction and validation.

#### Syntax Rules and Structure

| Decorator | Extracts | Example |
|-----------|----------|---------|
| `@Body()` | Request body | `@Body() dto: CreateUserDto` |
| `@Body('name')` | Specific body property | `@Body('name') name: string` |
| `@Param()` | Route parameters | `@Param() params: Record<string, string>` |
| `@Param('id')` | Specific route parameter | `@Param('id') id: string` |
| `@Query()` | Query parameters | `@Query() query: SearchDto` |
| `@Query('page')` | Specific query parameter | `@Query('page') page: string` |
| `@Headers()` | Request headers | `@Headers() headers: Record<string, string>` |
| `@Headers('authorization')` | Specific header | `@Headers('authorization') auth: string` |

**Constraints and Limitations:**
- Without a key, `@Param()` and `@Query()` return objects; with a key, they return strings.
- `@Body()` with a DTO class requires `ValidationPipe` for automatic validation.
- `@Res()` and `@Req()` give direct access to the underlying framework objects (Express/Fastify).

#### Annotated Code Example

```typescript
// users.controller.ts
import { Controller, Get, Post, Body, Param, Query, Headers, HttpCode, HttpStatus } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  findAll(
    @Query('page') page: string = '1',
    @Query('limit') limit: string = '10',
  ) {
    return { page: parseInt(page, 10), limit: parseInt(limit, 10) };
  }

  @Get(':id')
  findOne(
    @Param('id') id: string,
    @Headers('authorization') auth: string,
  ) {
    return { id, authorized: !!auth };
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: { name: string; email: string }) {
    return { created: true, user: createUserDto };
  }
}
```

**Expected Output (for `GET /users?page=2&limit=5`):**
```json
{"page":2,"limit":5}
```

**Expected Output (for `GET /users/42` with `Authorization: Bearer token`):**
```json
{"id":"42","authorized":true}
```

**Expected Output (for `POST /users` with JSON body):**
```json
{"created":true,"user":{"name":"Alice","email":"alice@example.com"}}
```

**Why this output:** `@Query('page')` extracts the `page` query parameter. `@Param('id')` extracts the `id` route parameter. `@Headers('authorization')` extracts the authorization header. `@Body()` injects the entire parsed request body.

#### Real-World Cases

- **CRUD APIs:** Extracting resource IDs, pagination parameters, and request bodies.
- **Authentication:** Extracting and validating tokens from headers.
- **Filtering:** Extracting query parameters for search and filter operations.

---

## Core Concept 3: Providers & Dependency Injection

### Sub-Feature 3.1: Creating Injectable Services with `@Injectable()`

#### Definitions

**Core Definition:** A provider is a class annotated with `@Injectable()` that can be injected as a dependency into other classes by Nest's IoC container.

**Technical Definition:** Providers are a fundamental concept in Nest. Many of the basic Nest classes may be treated as a provider — services, repositories, factories, helpers, and so on. The main idea of a provider is that it can be injected as a dependency; this means objects can create various relationships with each other, and the function of "wiring up" instances of objects can largely be delegated to the Nest runtime system. The `@Injectable()` decorator attaches metadata that declares the class as a provider managed by the Nest IoC container.

**Beginner-Friendly Explanation:** A provider is like a specialist in a company. The `@Injectable()` decorator says "this person can be hired by other departments." When a department needs a specialist, it doesn't hire one itself — it asks HR (the IoC container) to provide one. HR handles the hiring, training, and assignment.

#### Purposes

- To encapsulate business logic in injectable services.
- To enable dependency injection and inversion of control.
- To make code testable by allowing mock providers.
- To manage the lifecycle of service instances.

#### Syntax Rules and Structure

```typescript
@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] { return this.cats; }
  create(cat: Cat): void { this.cats.push(cat); }
}
```

```typescript
@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}
  // catsService is automatically injected
}
```

| Concept | Description |
|---------|-------------|
| `@Injectable()` | Marks a class as a provider. |
| Constructor injection | Dependencies declared in the constructor. |
| Provider registration | Listed in the `providers` array of a module. |
| Injection token | The class type itself (or a custom token). |

**Constraints and Limitations:**
- A provider must be registered in a module's `providers` array to be injectable.
- Providers are singletons by default (one instance per application).
- Circular dependencies require `forwardRef()`.

#### Annotated Code Example

```typescript
// cats.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  private cats: string[] = ['Whiskers', 'Felix'];

  findAll(): string[] {
    return this.cats;
  }

  create(name: string): void {
    this.cats.push(name);
  }
}
```

```typescript
// cats.controller.ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Get()
  findAll(): string[] {
    return this.catsService.findAll();
  }

  @Post()
  create(@Body('name') name: string): void {
    this.catsService.create(name);
  }
}
```

```typescript
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

**Expected Output (for `GET /cats`):**
```json
["Whiskers","Felix"]
```

**Expected Output (for `POST /cats` with `{"name":"Tom"}`):**
```
201 Created
```

**Why this output:** The `CatsService` is registered as a provider in `CatsModule`. When `CatsController` is instantiated, Nest's IoC container creates an instance of `CatsService` and injects it via the constructor.

#### Real-World Cases

- **Database services:** Encapsulating repository operations.
- **External API clients:** Services for calling third-party APIs.
- **Utility services:** Logging, configuration, and helper services.

---

### Sub-Feature 3.2: Control Inversion (IoC) Container Mechanics, Custom Providers, and Scoping

#### Definitions

**Core Definition:** The Nest IoC container manages the instantiation and injection of providers. Custom providers allow defining how a provider is created (`useValue`, `useClass`, `useFactory`, `useExisting`), and injection scopes control the lifetime of provider instances (`Singleton`, `Transient`, `Request`).

**Technical Definition:** The Nest IoC container resolves dependencies at application bootstrap. Providers can be registered in long-hand form using the `provide` and `use*` properties. `useValue` provides a constant value; `useClass` provides a class instance (useful for swapping implementations); `useFactory` provides a value computed by a factory function (with optional `inject` array); `useExisting` creates an alias for an existing provider. Injection scopes control the lifetime: `DEFAULT` (singleton, one instance per application), `REQUEST` (one instance per request), and `TRANSIENT` (one instance per consumer). Scope bubbles up the injection chain: a controller that depends on a request-scoped provider becomes request-scoped itself.

**Beginner-Friendly Explanation:** The IoC container is like a warehouse manager. When a worker (class) needs a tool (dependency), the manager (container) provides it. `useValue` is like handing over a pre-made tool. `useClass` is like ordering a specific brand of tool. `useFactory` is like assembling a custom tool on the spot. Scoping is about how many copies of the tool exist: singleton is one per warehouse, request is one per job, and transient is one per worker who asks.

#### Purposes

- To decouple class instantiation from usage.
- To allow different implementations to be swapped (e.g., for testing).
- To provide configuration values and pre-built objects.
- To control the lifetime of provider instances based on application needs.

#### Syntax Rules and Structure

**Custom providers:**

| Provider Type | Syntax | Use Case |
|---------------|--------|----------|
| Value | `{ provide: 'TOKEN', useValue: value }` | Constants, config objects |
| Class | `{ provide: 'TOKEN', useClass: MyClass }` | Swappable implementations |
| Factory | `{ provide: 'TOKEN', useFactory: (dep) => value, inject: [Dep] }` | Async init, conditional logic |
| Alias | `{ provide: 'TOKEN', useExisting: ExistingProvider }` | Aliasing |

**Injection scopes:**

| Scope | Lifetime | When to Use |
|-------|----------|-------------|
| `DEFAULT` (Singleton) | One instance per application | Most services (default) |
| `REQUEST` | One instance per request | Request tracking, multi-tenancy |
| `TRANSIENT` | One instance per consumer | Stateful helpers |

**Declaring scope:**
```typescript
@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

**Constraints and Limitations:**
- `Scope.REQUEST` bubbles up the injection chain; a singleton controller depending on a request-scoped service becomes request-scoped.
- Request-scoped providers cannot be retrieved with `app.get()`; use `app.resolve()`.
- Singleton scope is strongly recommended for most use cases.
- WebSocket gateways and Passport strategies should not use request-scoped providers.

#### Multiple Annotated Code Examples

#### Example 1: `useValue` — Configuration Constant

```typescript
// config.constant.ts
export const APP_NAME = {
  provide: 'APP_NAME',
  useValue: 'MyAwesomeApp',
};

// app.module.ts
@Module({
  providers: [APP_NAME],
  exports: ['APP_NAME'],
})
export class AppModule {}

// some.service.ts
@Injectable()
export class SomeService {
  constructor(@Inject('APP_NAME') private readonly name: string) {}
  whoAmI() { return `Running in ${this.name}`; }
}
```

**Expected Output:**
```
Running in MyAwesomeApp
```

**Why this output:** `useValue` provides a constant string. The `@Inject('APP_NAME')` decorator injects it into the service.

#### Example 2: `useClass` — Swappable Implementation

```typescript
// logger.interface.ts
export interface Logger { log(msg: string): void; }

// console-logger.ts
@Injectable()
export class ConsoleLogger implements Logger {
  log(msg: string) { console.log(msg); }
}

// file-logger.ts
@Injectable()
export class FileLogger implements Logger {
  log(msg: string) { /* write to file */ }
}

// app.module.ts
@Module({
  providers: [{ provide: 'Logger', useClass: FileLogger }],
})
export class AppModule {}

// any.service.ts
@Injectable()
export class AnyService {
  constructor(@Inject('Logger') private readonly logger: Logger) {}
}
```

**Expected Output:**
```
(Logs written to file instead of console)
```

**Why this output:** `useClass` allows swapping the `Logger` implementation without changing the consuming service.

#### Example 3: `useFactory` — Async Initialisation

```typescript
// database.provider.ts
export const DATABASE = {
  provide: 'DATABASE',
  useFactory: async (configService: ConfigService) => {
    const opts = configService.getDbOptions();
    const connection = await createConnection(opts);
    return connection;
  },
  inject: [ConfigService],
};

// app.module.ts
@Module({
  imports: [ConfigModule],
  providers: [DATABASE],
  exports: ['DATABASE'],
})
export class AppModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(@Inject('DATABASE') private readonly db: Connection) {}
}
```

**Expected Output:**
```
Database connection established and injected.
```

**Why this output:** `useFactory` runs the async function, injecting `ConfigService` as a dependency and returning the resolved connection.

#### Example 4: `Scope.REQUEST` — Per-Request Instance

```typescript
// request-logger.service.ts
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class RequestLoggerService {
  private readonly requestId = Math.random().toString(36).substring(7);
  log(msg: string) { console.log(`[${this.requestId}] ${msg}`); }
}
```

**Expected Output (two requests produce different IDs):**
```
[a1b2c3] Request handled
[d4e5f6] Request handled
```

**Why this output:** `Scope.REQUEST` creates a new instance for each incoming request, so `requestId` differs between requests.

#### Real-World Cases

- **Testing:** `useClass` to swap real services with mocks.
- **Configuration:** `useValue` for app constants and feature flags.
- **Database connections:** `useFactory` for async connection setup.
- **Multi-tenancy:** `Scope.REQUEST` for per-tenant context.

---

## Core Concept 4: The Request-Response Lifecycle Pipeline

### Sub-Feature 4.1: Guards — Authorization and Authentication Gates (`CanActivate`)

#### Definitions

**Core Definition:** A guard is a class annotated with `@Injectable()` that implements the `CanActivate` interface and determines whether a request should be handled by the route handler based on runtime conditions such as permissions or roles.

**Technical Definition:** A guard is a class annotated with the `@Injectable()` decorator that implements the `CanActivate` interface. Guards have a single responsibility: they determine whether a given request will be handled by the route handler, based on conditions present at runtime (such as permissions, roles, or ACLs). Every guard must implement a `canActivate()` method that returns a boolean (or a Promise/Observable of a boolean). Guards are executed after all middleware, but before any interceptor or pipe.

**Beginner-Friendly Explanation:** A guard is like a security checkpoint at the entrance of a building. Before you can enter (the route handler runs), the guard checks your ID (token) and your access level (role). If you don't have the right credentials, you're turned away.

#### Purposes

- To enforce authentication and authorization before route handlers execute.
- To check user roles, permissions, or ACLs.
- To provide fine-grained access control per route, controller, or globally.
- To keep authorization logic separate from business logic.

#### Syntax Rules and Structure

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

**Binding guards:**
```typescript
@UseGuards(AuthGuard)
@Controller('cats')
export class CatsController {}
```

| Binding Level | Scope |
|---------------|-------|
| Global (`app.useGlobalGuards()`) | All routes |
| Controller (`@UseGuards()`) | All routes in controller |
| Route (`@UseGuards()`) | Single route |

**Execution order:** Global → Controller → Route.

**Constraints and Limitations:**
- Guards run after middleware but before interceptors and pipes.
- A guard returning `false` results in a 403 Forbidden response.
- Guards can throw `UnauthorizedException` or `ForbiddenException` for specific error messages.

#### Annotated Code Example

```typescript
// auth.guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers['authorization'];

    if (!token || !token.startsWith('Bearer ')) {
      throw new UnauthorizedException('Missing or invalid token');
    }

    // Validate token (simplified)
    request.user = { id: 1, role: 'admin' };
    return true;
  }
}
```

```typescript
// cats.controller.ts
import { Controller, Get, UseGuards } from '@nestjs/common';
import { AuthGuard } from './auth.guard';

@Controller('cats')
@UseGuards(AuthGuard)
export class CatsController {
  @Get()
  findAll(): string[] {
    return ['Whiskers', 'Felix'];
  }
}
```

**Expected Output (with valid token):**
```json
["Whiskers","Felix"]
```

**Expected Output (without token):**
```json
{"statusCode":401,"message":"Missing or invalid token"}
```

**Why this output:** The `AuthGuard` checks for a valid Bearer token. If present, it attaches `request.user` and returns `true`. If missing, it throws `UnauthorizedException`, resulting in a 401 response.

#### Real-World Cases

- **JWT authentication:** Validating JWT tokens before route execution.
- **Role-based access control:** Checking user roles (admin, user, guest).
- **API key validation:** Verifying API keys for service-to-service calls.

---

### Sub-Feature 4.2: Interceptors — Binding Logic Before/After Method Execution (`NestInterceptor`)

#### Definitions

**Core Definition:** An interceptor is a class annotated with `@Injectable()` that implements the `NestInterceptor` interface, allowing logic to be executed before and after route handler execution, response transformation, and exception handling.

**Technical Definition:** An interceptor is a class annotated with the `@Injectable()` decorator that implements the `NestInterceptor` interface. Interceptors offer a set of capabilities inspired by Aspect Oriented Programming (AOP). They make it possible to: bind extra logic before or after method execution, transform the result returned from a function, transform the exception thrown from a function, extend the basic function behavior, and completely override a function depending on specific conditions. Each interceptor implements the `intercept()` method, which takes two arguments: `ExecutionContext` and `CallHandler`. The `handle()` method on `CallHandler` invokes the route handler and returns an RxJS Observable.

**Beginner-Friendly Explanation:** An interceptor is like a personal assistant who wraps around your work. Before you start a task, the assistant does some preparation (logging, timing). After you finish, the assistant reformats your output or handles errors. The assistant can even decide not to let you do the task at all (caching).

#### Purposes

- To log request/response details (timing, user activity).
- To transform response data to a consistent format.
- To implement caching by short-circuiting the handler.
- To handle timeouts and retry logic.
- To catch and transform exceptions.

#### Syntax Rules and Structure

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    const now = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`After... ${Date.now() - now}ms`)),
    );
  }
}
```

| Method | Description |
|--------|-------------|
| `intercept(context, next)` | Wraps the handler execution. |
| `next.handle()` | Invokes the route handler; returns Observable. |
| RxJS operators | Used to transform the response stream. |

**Binding interceptors:**
```typescript
@UseInterceptors(LoggingInterceptor)
@Controller('cats')
export class CatsController {}
```

**Constraints and Limitations:**
- Interceptors run after guards but before pipes.
- The response side of interceptors resolves in reverse order (route → controller → global).
- Errors thrown by pipes, controllers, or services can be caught in the interceptor's `catchError` operator.

#### Annotated Code Example

```typescript
// logging.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    console.log(`Incoming: ${request.method} ${request.url}`);

    const now = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`Completed in ${Date.now() - now}ms`)),
    );
  }
}
```

```typescript
// transform.interceptor.ts — Transform response shape
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({ success: true, data })),
    );
  }
}
```

```typescript
// cats.controller.ts
@Controller('cats')
@UseInterceptors(LoggingInterceptor, TransformInterceptor)
export class CatsController {
  @Get()
  findAll(): string[] { return ['Whiskers', 'Felix']; }
}
```

**Expected Output (console):**
```
Incoming: GET /cats
Completed in 2ms
```

**Expected Output (response body):**
```json
{"success":true,"data":["Whiskers","Felix"]}
```

**Why this output:** `LoggingInterceptor` logs before and after execution. `TransformInterceptor` maps the response to a consistent envelope `{ success, data }`.

#### Real-World Cases

- **Response formatting:** Wrapping all responses in a standard envelope.
- **Caching:** Returning cached data without invoking the handler.
- **Timeout management:** Using `timeout()` from RxJS to abort long-running handlers.
- **Metrics:** Recording execution time for performance monitoring.

---

### Sub-Feature 4.3: Pipes — Data Transformation and Validation (`ValidationPipe`)

#### Definitions

**Core Definition:** A pipe is a class annotated with `@Injectable()` that implements the `PipeTransform` interface, used to transform input data or validate it before it reaches the route handler.

**Technical Definition:** A pipe is a class annotated with the `@Injectable()` decorator that implements the `PipeTransform` interface. Pipes have two typical use cases: transformation (transform input data to the desired form) and validation (evaluate input data and, if valid, pass it through unchanged; otherwise, throw an exception). Nest provides several built-in pipes: `ValidationPipe`, `ParseIntPipe`, `ParseBoolPipe`, `ParseArrayPipe`, and `ParseUUIDPipe`. The `ValidationPipe` uses the `class-validator` and `class-transformer` libraries to validate DTOs.

**Beginner-Friendly Explanation:** A pipe is like a quality control inspector on an assembly line. Before the product (data) reaches the final station (route handler), the inspector checks it against a specification (DTO schema). If it passes, it's passed along. If not, it's rejected with an error.

#### Purposes

- To validate incoming data against DTO schemas.
- To transform data types (string to integer, etc.).
- To provide consistent error responses for invalid input.
- To strip or whitelist properties for security.

#### Syntax Rules and Structure

```typescript
@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Validate or transform value
    return value;
  }
}
```

**Binding pipes:**
```typescript
@Post()
create(@Body(ValidationPipe) dto: CreateCatDto) {}
```

| Binding Level | Scope |
|---------------|-------|
| Parameter | Single parameter |
| Method | All parameters in handler |
| Controller | All routes in controller |
| Global (`app.useGlobalPipes()`) | All routes |

**Using `ValidationPipe`:**
```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));
```

```typescript
// create-cat.dto.ts
import { IsString, IsInt, Min, Max } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  @Min(0)
  @Max(30)
  age: number;
}
```

**Constraints and Limitations:**
- `ValidationPipe` requires `class-validator` and `class-transformer`.
- Global pipes run before controller-level and route-level pipes.
- In route parameter-level, pipes run from the last parameter to the first.

#### Annotated Code Example

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,           // Strip non-whitelisted properties
    forbidNonWhitelisted: true, // Throw if non-whitelisted properties present
    transform: true,           // Auto-transform payloads to DTO instances
  }));
  await app.listen(3000);
}
bootstrap();
```

```typescript
// create-cat.dto.ts
import { IsString, IsInt, Min, Max } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  @Min(0)
  @Max(30)
  age: number;
}
```

```typescript
// cats.controller.ts
@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return { created: true, cat: createCatDto };
  }
}
```

**Expected Output (for valid body `{"name":"Tom","age":3}`):**
```json
{"created":true,"cat":{"name":"Tom","age":3}}
```

**Expected Output (for invalid body `{"name":"Tom","age":-1}`):**
```json
{"statusCode":400,"message":["age must not be less than 0"],"error":"Bad Request"}
```

**Why this output:** The `ValidationPipe` validates the body against `CreateCatDto`. Invalid data (negative age) produces a 400 response with detailed error messages. Valid data passes through and is transformed into a `CreateCatDto` instance.

#### Real-World Cases

- **Form validation:** Validating registration forms, login credentials, and profile updates.
- **Query parameter parsing:** Converting string IDs to numbers with `ParseIntPipe`.
- **UUID validation:** Ensuring resource IDs are valid UUIDs with `ParseUUIDPipe`.
- **API input sanitisation:** Stripping unknown properties with `whitelist: true`.

---

### Sub-Feature 4.4: Exception Filters — Customized Error Handling (`ExceptionFilter`)

#### Definitions

**Core Definition:** An exception filter is a class annotated with `@Catch()` that implements the `ExceptionFilter` interface, allowing custom handling and formatting of unhandled exceptions.

**Technical Definition:** Nest comes with a built-in exceptions layer that processes all unhandled exceptions across an application. When your application code does not handle an exception, this layer catches it and automatically sends an appropriate, user-friendly response. Out of the box, this is done by a built-in global exception filter, which handles exceptions of type `HttpException` (and its subclasses). To create a custom exception filter, implement the `ExceptionFilter` interface with the `@Catch()` decorator specifying which exceptions to handle. The `catch(exception, host)` method receives the exception and an `ArgumentsHost` object, which provides access to the request and response.

**Beginner-Friendly Explanation:** An exception filter is like a customer service desk for errors. When something goes wrong (an exception is thrown), the filter catches it and decides how to respond to the customer (client). Instead of letting the error crash the application or return an ugly message, the filter formats it nicely and returns a consistent response.

#### Purposes

- To catch and handle unhandled exceptions centrally.
- To format error responses consistently across the application.
- To log errors with context (request details, user info).
- To map domain-specific exceptions to appropriate HTTP status codes.

#### Syntax Rules and Structure

```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();
    response.status(status).json({
      statusCode: status,
      message: exception.message,
    });
  }
}
```

| Decorator | Purpose |
|-----------|---------|
| `@Catch()` | Specifies which exception(s) to handle. |
| `@Catch()` (empty) | Catches all exceptions. |

**Binding filters:**
```typescript
@UseFilters(HttpExceptionFilter)
@Controller('cats')
export class CatsController {}
```

**Execution order:** Filters resolve from the lowest level up: route → controller → global.

**Constraints and Limitations:**
- Exception filters only run on unhandled exceptions; caught exceptions (try/catch) do not trigger filters.
- Once an exception reaches a filter, it cannot be passed to another filter (unless via inheritance).
- Global filters can be registered via `app.useGlobalFilters()` or the `APP_FILTER` token.

#### Annotated Code Example

```typescript
// http-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.message
        : 'Internal server error';

    // Log the error with context
    console.error(`[${new Date().toISOString()}] ${request.method} ${request.url} - ${status}: ${message}`);

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}
```

```typescript
// main.ts — Register global filter
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { AllExceptionsFilter } from './http-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new AllExceptionsFilter());
  await app.listen(3000);
}
bootstrap();
```

**Expected Output (for a 404 request):**
```json
{
  "statusCode": 404,
  "timestamp": "2026-01-15T12:00:00.000Z",
  "path": "/nonexistent",
  "message": "Cannot GET /nonexistent"
}
```

**Expected Output (console):**
```
[2026-01-15T12:00:00.000Z] GET /nonexistent - 404: Cannot GET /nonexistent
```

**Why this output:** The `@Catch()` decorator with no arguments catches all exceptions. The filter determines the status code (404 for `NotFoundException`), logs the error with request context, and returns a structured JSON response with timestamp and path.

#### Real-World Cases

- **Consistent API errors:** Returning a standard error envelope for all endpoints.
- **Logging:** Recording errors with request context for debugging.
- **Domain exceptions:** Mapping `PaymentFailedException` to a 402 status code.
- **Production safety:** Hiding stack traces in production while showing them in development.

---

## References

- NestJS Documentation — Modules — https://docs.nestjs.com/modules
- NestJS Documentation — Dynamic Modules — https://docs.nestjs.com/fundamentals/dynamic-modules
- NestJS Documentation — Controllers — https://docs.nestjs.com/controllers
- NestJS Documentation — Providers — https://docs.nestjs.com/providers
- NestJS Documentation — Custom Providers — https://docs.nestjs.com/fundamentals/custom-providers
- NestJS Documentation — Injection Scopes — https://docs.nestjs.com/fundamentals/injection-scopes
- NestJS Documentation — Guards — https://docs.nestjs.com/guards
- NestJS Documentation — Interceptors — https://docs.nestjs.com/interceptors
- NestJS Documentation — Pipes — https://docs.nestjs.com/pipes
- NestJS Documentation — Validation — https://docs.nestjs.com/application/validation
- NestJS Documentation — Exception Filters — https://docs.nestjs.com/exception-filters
- NestJS Documentation — Request Lifecycle — https://docs.nestjs.com/faq/request-lifecycle
- NestJS Documentation — Execution Context — https://docs.nestjs.com/fundamentals/execution-context
- NestJS Documentation — Middleware — https://docs.nestjs.com/middleware
- The NestJS Handbook — FreeCodeCamp — https://www.freecodecamp.org/news/the-nestjs-handbook-learn-to-use-nest-with-code-examples
- Difference between forRoot and forFeature — Stack Overflow — https://stackoverflow.com/questions/66371656
- NestJS Request and Application Lifecycle — Stack Overflow — https://stackoverflow.com/questions/51691924
- NestJS Dependency Injection: Why Your Services Won't Inject — DEV Community — https://dev.to/nestjs-dependency-injection
- Building Configurable Dynamic Modules — CoddyKit — https://www.coddykit.com
- NestJS Module Encapsulation Explained — DEV Community — https://dev.to/nestjs-module-encapsulation
- NestJS Guards in NestJS — GeeksforGeeks — https://origin.geeksforgeeks.org/nestjs-guards
- NestJS Interceptors — NestJS Documentation — https://docs.nestjs.com/interceptors
- NestJS Exception Filters — NestJS Documentation — https://docs.nestjs.com/exception-filters
- NestJS Validation — NestJS Documentation — https://docs.nestjs.com/application/validation
- NestJS Injection Scopes — NestJS Documentation — https://docs.nestjs.com/fundamentals/injection-scopes