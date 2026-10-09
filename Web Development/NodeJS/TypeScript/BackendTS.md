# TypeScript for Backend Development — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** TypeScript for backend development applies TypeScript's static type system to server-side Node.js code — typing HTTP requests, database models, DTOs, service interfaces, errors, and runtime validation — to catch bugs at compile time and improve maintainability.

**Technical Definition:** TypeScript for backend development integrates static types with Node.js frameworks (Express, Fastify, NestJS), ORMs/ODMs (Prisma, TypeORM, Mongoose, Drizzle), and runtime validators (Zod, TypeBox, ArkType, class-validator, Valibot). Key patterns include module augmentation for extending framework types (e.g., `req.user`), inferred types from ORM schemas (`Prisma.UserGetPayload`), rigid DTO contracts, interface-based dependency injection, typed error hierarchies, and runtime-to-compile-time type generation (schema-first development).

**Beginner-Friendly Explanation:** Backend development has many moving parts — HTTP requests, database queries, validation, errors, and services. Without types, a typo in a property name or a mismatch between what the database returns and what the API sends can cause runtime crashes. TypeScript for backend catches these mistakes at compile time. It's like having a blueprint that ensures every part of your server fits together correctly before you turn it on.

### Key Characteristics

- **End-to-end type safety:** Types flow from the database through services to the HTTP response.
- **Framework integration:** Typed requests, responses, and middleware.
- **ORM inference:** Types generated from database schemas.
- **DTO contracts:** Rigid structures for data crossing layer boundaries.
- **Dependency injection:** Interfaces enabling testable, swappable services.
- **Runtime validation:** Zod, TypeBox, and class-validator bridge runtime and compile-time.
- **Error hierarchies:** Typed custom exceptions for structured error handling.

### Prerequisites

- **TypeScript fundamentals:** Types, interfaces, generics, unions, strict mode.
- **Node.js fundamentals:** Express, Fastify, or NestJS.
- **ORM/ODM basics:** Prisma, TypeORM, Mongoose, or Drizzle.
- **Async programming:** Promises, `async`/`await`, error handling.
- **REST API concepts:** Routes, methods, status codes, DTOs.
- **Validation libraries:** Zod, TypeBox, class-validator, Joi.

### Related Programming Areas

- **Web security:** Input validation, output encoding, authentication.
- **Database access:** ORMs, query builders, migrations.
- **API design:** REST, GraphQL, gRPC, tRPC.
- **Testing:** Unit, integration, and end-to-end tests.
- **DevOps:** Build pipelines, Docker, CI/CD.

### Core Concepts

1. **Typed Request Objects** — extending Express or Fastify request definitions with custom properties like `req.user`.
2. **Typed Database Models** — typing schemas for ORMs/ODMs like Prisma, TypeORM, or Mongoose.
3. **DTOs** — Data Transfer Objects for defining rigid structural data contracts.
4. **Service Interfaces** — abstracting application domain logic structures for dependency injection.
5. **Error Types** — creating robust, typed custom application exceptions.
6. **Runtime Schema Validation Integration** — generating compile-time TypeScript types from runtime validators like Zod, TypeBox, or ArkType.

---

## Core Concept 1: Typed Request Objects

### Definitions

**Core Definition:** Typed request objects extend the framework's request type with custom properties (e.g., `req.user`, `req.tenantId`) so that middleware and controllers can access them with full type safety.

**Technical Definition:** Express and Fastify define `Request` interfaces that can be augmented via module augmentation (TypeScript declaration merging). For Express, `declare global { namespace Express { interface Request { user?: AuthUser; tenantId?: string; } } }` adds custom properties. For Fastify, `declare module 'fastify' { interface FastifyRequest { user?: AuthUser; } }` augments the request. The alternative is to use generic request types (`Request<Params, ResBody, ReqBody, ReqQuery>`) in Express, which types the standard fields (params, body, query) but not custom properties.

**Beginner-Friendly Explanation:** When a user logs in, you attach their info to the request object (`req.user`). But plain TypeScript doesn't know that `req.user` exists — it would show an error. Typed request objects tell TypeScript "yes, this request has a `user` property, and it's of type `AuthUser`." This way, your code knows exactly what's on the request.

### Purposes

- To give middleware and controllers type-safe access to custom request properties.
- To type standard request fields (params, body, query) in Express.
- To enable autocomplete for request properties.
- To catch typos and type mismatches at compile time.
- To document the shape of authenticated requests.

### Syntax Rules and Structure

#### Express Module Augmentation

```typescript
// types/express.d.ts
import { AuthUser } from '../auth/types';

declare global {
  namespace Express {
    interface Request {
      user?: AuthUser;
      tenantId?: string;
      requestId: string;
    }
  }
}

export {};
```

#### Fastify Module Augmentation

```typescript
// types/fastify.d.ts
import { AuthUser } from '../auth/types';

declare module 'fastify' {
  interface FastifyRequest {
    user?: AuthUser;
    tenantId?: string;
  }

  interface FastifyInstance {
    authenticate: (request: FastifyRequest) => Promise<void>;
  }
}

export {};
```

#### Typed Express Request with Generics

```typescript
import { Request, Response } from 'express';

interface CreateUserParams {
  id: string;
}

interface CreateUserBody {
  name: string;
  email: string;
}

interface CreateUserQuery {
  notify?: string;
}

type CreateUserRequest = Request<CreateUserParams, unknown, CreateUserBody, CreateUserQuery>;

app.post('/users/:id', (req: CreateUserRequest, res: Response) => {
  const { id } = req.params;         // string
  const { name, email } = req.body;  // string
  const { notify } = req.query;      // string | undefined
});
```

#### Syntax Rules

- **Use `declare global` for Express** — augment the `Express.Request` interface.
- **Use `declare module 'fastify'` for Fastify** — augment `FastifyRequest`.
- **Mark custom properties as optional (`?`)** — they may not be set on all requests.
- **Use generics for standard fields** — `Request<Params, ResBody, ReqBody, ReqQuery>`.
- **Export `{}`** — to make the file a module.
- **Place type declarations in a `types/` directory** — include in `tsconfig.json`.
- **Use a shared `AuthUser` type** — consistent across the codebase.
- **Avoid `any`** — type all request properties.

#### Constraints and Limitations

- **Module augmentation is global** — it affects all requests.
- **Custom properties must be optional** — otherwise TypeScript errors on non-authenticated routes.
- **Augmentation files must be included** — in `tsconfig.json` `include`.
- **Express generics only type standard fields** — custom properties require augmentation.
- **Fastify has built-in request typing** — use `FastifyRequest<{ Params, Body, Query }>`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Typed Express Request with Authentication Middleware

```typescript
// types/auth.ts
export interface AuthUser {
  id: string;
  email: string;
  roles: string[];
  tenantId: string;
}
```

```typescript
// types/express.d.ts
import { AuthUser } from './auth';

declare global {
  namespace Express {
    interface Request {
      user?: AuthUser;
      tenantId?: string;
      requestId: string;
    }
  }
}

export {};
```

```typescript
// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';

export function requireAuth(req: Request, res: Response, next: NextFunction): void {
  const token = req.headers.authorization?.slice(7);
  if (!token) {
    res.status(401).json({ error: 'Unauthorized' });
    return;
  }

  try {
    const payload = verifyToken(token);
    req.user = {
      id: payload.sub,
      email: payload.email,
      roles: payload.roles,
      tenantId: payload.tenantId,
    };
    req.tenantId = payload.tenantId;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}
```

```typescript
// controllers/user.controller.ts
import { Request, Response } from 'express';

interface GetUserParams {
  id: string;
}

export async function getUser(
  req: Request<GetUserParams>,
  res: Response,
): Promise<void> {
  // req.user is typed as AuthUser | undefined
  if (!req.user) {
    res.status(401).json({ error: 'Unauthorized' });
    return;
  }

  // req.user.id is string (not any)
  const userId = req.user.id;
  const tenantId = req.user.tenantId;

  // req.params.id is string
  const targetId = req.params.id;

  // Authorization check
  if (userId !== targetId && !req.user.roles.includes('admin')) {
    res.status(403).json({ error: 'Forbidden' });
    return;
  }

  const user = await userService.findById(targetId, tenantId);
  if (!user) {
    res.status(404).json({ error: 'User not found' });
    return;
  }

  res.json(user);
}
```

**Expected behaviour:** TypeScript knows that `req.user` exists and is of type `AuthUser | undefined`. After the null check, `req.user.id` is `string`. The `req.params.id` is typed as `string`. Typos in property names are compile errors.

**Why this works:** Module augmentation extends the Express `Request` interface with custom properties. The `AuthUser` type ensures consistency. Generics type the standard fields. TypeScript catches errors at compile time.

### Real-World Cases

- **Authentication:** Attaching `req.user` after JWT verification.
- **Multi-tenancy:** Attaching `req.tenantId` for tenant isolation.
- **Request tracing:** Attaching `req.requestId` for logging.
- **Localisation:** Attaching `req.locale` for i18n.

---

## Core Concept 2: Typed Database Models

### Definitions

**Core Definition:** Typed database models are TypeScript types inferred from ORM/ODM schemas, ensuring that database queries and results are fully typed.

**Technical Definition:** Modern ORMs (Prisma, Drizzle) generate TypeScript types from the schema, providing end-to-end type safety from the database to the application. Prisma generates a `PrismaClient` with typed models, `Prisma.UserGetPayload` for custom selections, and `Prisma.UserCreateInput` for input types. TypeORM uses decorators and entity classes. Mongoose requires manual interface definitions plus a `Schema` and `Model`. Drizzle infers types from the schema definition. The key benefit: typos in field names, missing fields, and type mismatches are caught at compile time.

**Beginner-Friendly Explanation:** Your database has a schema — columns with types. Typed database models tell TypeScript what those columns are. So when you write `user.emial` (typo), TypeScript catches it. When you try to save a number in a string field, TypeScript catches it. It's like having a blueprint of your database that TypeScript can check against your code.

### Purposes

- To catch database-related errors at compile time.
- To provide autocomplete for database fields.
- To ensure that queries and mutations are type-safe.
- To generate types from the schema automatically.
- To enable safe refactoring of database schemas.

### Syntax Rules and Structure

#### Prisma (Generated Types)

```prisma
// schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  USER
  EDITOR
  ADMIN
}

model Post {
  id       String @id @default(uuid())
  title    String
  body     String
  authorId String
  author   User   @relation(fields: [authorId], references: [id])
}
```

```typescript
// Generated types (simplified)
import { PrismaClient, User, Post, Role } from '@prisma/client';

const prisma = new PrismaClient();

// Fully typed
const user: User = await prisma.user.findUniqueOrThrow({ where: { id: '1' } });
console.log(user.email); // string
console.log(user.role);  // Role enum

// Custom selection
const userWithPosts = await prisma.user.findUnique({
  where: { id: '1' },
  include: { posts: true },
});
// userWithPosts.posts is Post[]

// Input types
type CreateUserInput = Prisma.UserCreateInput;
const input: CreateUserInput = {
  email: 'alice@example.com',
  name: 'Alice',
  role: 'ADMIN',
};
```

#### TypeORM (Entity Classes)

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ type: 'enum', enum: Role, default: Role.USER })
  role: Role;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @Column({ type: 'timestamptz', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;
}

export enum Role {
  USER = 'USER',
  EDITOR = 'EDITOR',
  ADMIN = 'ADMIN',
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  @Column('text')
  body: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;

  @Column()
  authorId: string;
}
```

#### Mongoose (Manual Interfaces)

```typescript
import { Schema, model, Document, Types } from 'mongoose';

interface IUser {
  email: string;
  name: string;
  role: 'user' | 'editor' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

interface IUserDocument extends IUser, Document {
  _id: Types.ObjectId;
  fullName: string; // Virtual
}

const UserSchema = new Schema<IUserDocument>({
  email: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  role: { type: String, enum: ['user', 'editor', 'admin'], default: 'user' },
}, { timestamps: true });

UserSchema.virtual('fullName').get(function () {
  return this.name;
});

export const UserModel = model<IUserDocument>('User', UserSchema);

// Usage — fully typed
const user = await UserModel.findOne({ email: 'alice@example.com' });
if (user) {
  console.log(user.email);    // string
  console.log(user.fullName); // string (virtual)
}
```

#### Drizzle (Schema Inference)

```typescript
import { pgTable, uuid, text, timestamp, pgEnum } from 'drizzle-orm/pg-core';
import { InferSelectModel, InferInsertModel } from 'drizzle-orm';

export const roleEnum = pgEnum('role', ['user', 'editor', 'admin']);

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  role: roleEnum('role').default('user').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export type User = InferSelectModel<typeof users>;
export type NewUser = InferInsertModel<typeof users>;
```

#### Syntax Rules

- **Use Prisma for generated types** — run `prisma generate` after schema changes.
- **Use TypeORM entities with decorators** — enable `emitDecoratorMetadata`.
- **Use Mongoose with manual interfaces** — define `IUser` and `IUserDocument`.
- **Use Drizzle with schema inference** — `InferSelectModel`, `InferInsertModel`.
- **Prefer schema-first ORMs** — Prisma, Drizzle generate types.
- **Use `Prisma.UserGetPayload`** — for custom selections.
- **Use `Prisma.UserCreateInput`** — for input types.
- **Keep database types separate from DTOs** — do not leak ORM types to the API.

#### Constraints and Limitations

- **Generated types must be regenerated** — after schema changes.
- **TypeORM requires decorators** — `experimentalDecorators`, `emitDecoratorMetadata`.
- **Mongoose requires manual interfaces** — types are not inferred from the schema.
- **Drizzle requires schema definition** — types are inferred from the Drizzle schema.
- **ORM types may differ from runtime** — always validate input.
- **Prisma types are large** — compilation may be slower.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Prisma with Typed Repository and Service

```typescript
// prisma/schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
}

enum Role {
  USER
  EDITOR
  ADMIN
}
```

```typescript
// types/user.ts
import { User, Role, Prisma } from '@prisma/client';

export type { User, Role };

// Custom selection type
export type UserWithPostCount = Prisma.UserGetPayload<{
  include: { _count: { select: { posts: true } } };
}>;

// Input types
export type CreateUserInput = Prisma.UserCreateInput;
export type UpdateUserInput = Prisma.UserUpdateInput;
```

```typescript
// repositories/user.repository.ts
import { PrismaClient, User, Prisma } from '@prisma/client';

export class UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async findWithPostCount(id: string): Promise<UserWithPostCount | null> {
    return this.prisma.user.findUnique({
      where: { id },
      include: { _count: { select: { posts: true } } },
    });
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    return this.prisma.user.create({ data });
  }

  async update(id: string, data: Prisma.UserUpdateInput): Promise<User> {
    return this.prisma.user.update({ where: { id }, data });
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }
}
```

**Expected behaviour:** All database operations are fully typed. `findWithPostCount` returns a type that includes `_count.posts`. `create` accepts only valid `UserCreateInput`. Typos in field names are compile errors.

**Why this works:** Prisma generates types from the schema. The repository wraps Prisma with a typed interface. `Prisma.UserGetPayload` extracts the exact return type for custom selections.

### Real-World Cases

- **REST APIs:** Typed database access with Prisma or Drizzle.
- **GraphQL:** Typed resolvers with database types.
- **Migrations:** Type-safe schema changes.
- **Testing:** Typed in-memory repositories.

---

## Core Concept 3: DTOs (Data Transfer Objects)

### Definitions

**Core Definition:** A DTO is an object that defines the shape of data crossing a boundary (e.g., HTTP request/response), providing a rigid contract between layers.

**Technical Definition:** DTOs in TypeScript are typically defined with interfaces, type aliases, or classes. They are separate from database models and domain entities — DTOs are designed for transport. DTOs should be immutable (`readonly` properties), validated (Zod, class-validator), and mapped to/from domain entities. Common patterns include input DTOs (request bodies), output DTOs (responses), and query DTOs (filters/pagination). Mapping between DTOs and entities is done with mappers (`UserMapper.toDto(user)`).

**Beginner-Friendly Explanation:** A DTO is like a shipping label. It describes exactly what data is being sent and how it's structured. The database has its own model (the package), but when it crosses the API boundary, it's packaged into a DTO (the shipping label). This keeps the API contract stable even if the database schema changes.

### Purposes

- To define rigid data contracts between layers.
- To decouple the API from the database schema.
- To enable validation at the boundary.
- To prevent over-posting and data leakage.
- To document the API contract.

### Syntax Rules and Structure

#### Input DTO

```typescript
// dto/create-user.dto.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase().trim(),
  name: z.string().min(1).max(100).trim(),
  password: z.string().min(12).max(128),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
}).strict();

export type CreateUserDto = z.infer<typeof CreateUserSchema>;
```

#### Output DTO

```typescript
// dto/user.dto.ts
export interface UserDto {
  readonly id: string;
  readonly email: string;
  readonly name: string;
  readonly role: 'user' | 'editor' | 'admin';
  readonly createdAt: string; // ISO string
}
```

#### Query DTO

```typescript
// dto/list-users.dto.ts
import { z } from 'zod';

export const ListUsersSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  role: z.enum(['user', 'editor', 'admin']).optional(),
  search: z.string().max(100).optional(),
});

export type ListUsersDto = z.infer<typeof ListUsersSchema>;
```

#### Mapper

```typescript
// mappers/user.mapper.ts
import { User } from '@prisma/client';
import { UserDto } from '../dto/user.dto';

export class UserMapper {
  static toDto(user: User): UserDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
      createdAt: user.createdAt.toISOString(),
    };
  }

  static toDtoList(users: User[]): UserDto[] {
    return users.map(UserMapper.toDto);
  }
}
```

#### Syntax Rules

- **Define DTOs as interfaces or Zod schemas.**
- **Use `readonly` properties** — DTOs should be immutable.
- **Use `.strict()`** — reject unknown properties.
- **Validate at the boundary** — controller or middleware.
- **Map between DTOs and entities** — never expose entities directly.
- **Use separate input and output DTOs** — different shapes.
- **Use query DTOs for filters** — pagination, sorting, search.
- **Document DTOs** — with OpenAPI/Swagger decorators.
- **Keep DTOs flat** — avoid deeply nested structures.

#### Constraints and Limitations

- **DTOs add boilerplate** — mapping between DTOs and entities.
- **DTOs can drift from entities** — keep them in sync.
- **Validation is not automatic** — must be applied at the boundary.
- **DTOs should not contain business logic** — pure data.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full DTO Flow with Zod and Mapping

```typescript
// dto/create-user.dto.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase().trim(),
  name: z.string().min(1).max(100).trim(),
  password: z.string().min(12).max(128),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
}).strict();

export type CreateUserDto = z.infer<typeof CreateUserSchema>;
```

```typescript
// dto/user.dto.ts
export interface UserDto {
  readonly id: string;
  readonly email: string;
  readonly name: string;
  readonly role: 'user' | 'editor' | 'admin';
  readonly createdAt: string;
}
```

```typescript
// mappers/user.mapper.ts
import { User } from '@prisma/client';
import { UserDto } from '../dto/user.dto';

export class UserMapper {
  static toDto(user: User): UserDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
      createdAt: user.createdAt.toISOString(),
    };
  }
}
```

```typescript
// services/user.service.ts
import { PrismaClient, User } from '@prisma/client';
import { CreateUserDto } from '../dto/create-user.dto';
import { UserDto } from '../dto/user.dto';
import { UserMapper } from '../mappers/user.mapper';
import bcrypt from 'bcrypt';

export class UserService {
  constructor(private readonly prisma: PrismaClient) {}

  async create(dto: CreateUserDto): Promise<UserDto> {
    const passwordHash = await bcrypt.hash(dto.password, 12);

    const user = await this.prisma.user.create({
      data: {
        email: dto.email,
        name: dto.name,
        passwordHash,
        role: dto.role,
      },
    });

    return UserMapper.toDto(user);
  }

  async findById(id: string): Promise<UserDto | null> {
    const user = await this.prisma.user.findUnique({ where: { id } });
    return user ? UserMapper.toDto(user) : null;
  }
}
```

```typescript
// controllers/user.controller.ts
import { Request, Response } from 'express';
import { CreateUserSchema } from '../dto/create-user.dto';

export async function createUser(req: Request, res: Response): Promise<void> {
  const result = CreateUserSchema.safeParse(req.body);
  if (!result.success) {
    res.status(400).json({
      error: 'ValidationError',
      details: result.error.issues,
    });
    return;
  }

  const user = await userService.create(result.data);
  res.status(201).json(user);
}
```

**Expected behaviour:** The controller validates the request body with Zod. The service receives a typed `CreateUserDto`. The mapper converts the Prisma `User` to a `UserDto`. The response never leaks `passwordHash`.

**Why this works:** DTOs define rigid contracts. Zod validates at the boundary. The mapper decouples the API from the database. The response only includes safe fields.

### Real-World Cases

- **REST APIs:** Input/output DTOs for every endpoint.
- **GraphQL:** Input types and output types.
- **gRPC:** Protobuf messages.
- **Webhooks:** Payload DTOs with validation.

---

## Core Concept 4: Service Interfaces

### Definitions

**Core Definition:** Service interfaces define the contract for application domain logic, enabling dependency injection, testing, and swappable implementations.

**Technical Definition:** Service interfaces in TypeScript are abstract contracts (`interface IUserService`) that concrete services implement (`class UserService implements IUserService`). They enable dependency injection (constructor injection), testability (mock implementations), and separation of concerns. Interfaces should be focused (single responsibility), technology-agnostic (no ORM types), and stable (changes require updating all implementations). Services orchestrate business logic, coordinate repositories, and manage transactions.

**Beginner-Friendly Explanation:** A service interface is like a job description. It says "anyone in this role must be able to do these things." The concrete service is the actual employee. By coding against the interface, you can swap employees (implementations) without changing the rest of the company (the application). For testing, you can hire a temporary worker (mock) that does the same things but with predictable behaviour.

### Purposes

- To define contracts for business logic.
- To enable dependency injection.
- To make services testable with mocks.
- To decouple the application from specific implementations.
- To enforce single responsibility.

### Syntax Rules and Structure

#### Service Interface

```typescript
// services/user.service.interface.ts
export interface IUserService {
  findById(id: string): Promise<UserDto | null>;
  findByEmail(email: string): Promise<UserDto | null>;
  create(dto: CreateUserDto): Promise<UserDto>;
  update(id: string, dto: UpdateUserDto): Promise<UserDto>;
  delete(id: string): Promise<void>;
}
```

#### Concrete Implementation

```typescript
// services/user.service.ts
export class UserService implements IUserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly passwordService: PasswordService,
    private readonly logger: Logger,
  ) {}

  async findById(id: string): Promise<UserDto | null> {
    const user = await this.userRepository.findById(id);
    return user ? UserMapper.toDto(user) : null;
  }

  async create(dto: CreateUserDto): Promise<UserDto> {
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) throw new ConflictError('Email already registered');

    const passwordHash = await this.passwordService.hash(dto.password);
    const user = await this.userRepository.create({
      ...dto,
      passwordHash,
    });

    this.logger.info(`User created: ${user.id}`);
    return UserMapper.toDto(user);
  }

  // ... other methods
}
```

#### Dependency Injection

```typescript
// container.ts
import { PrismaClient } from '@prisma/client';
import { IUserService } from './services/user.service.interface';
import { UserService } from './services/user.service';
import { UserRepository } from './repositories/user.repository';
import { PasswordService } from './services/password.service';

const prisma = new PrismaClient();
const userRepository = new UserRepository(prisma);
const passwordService = new PasswordService();

export const userService: IUserService = new UserService(
  userRepository,
  passwordService,
  logger,
);
```

#### Syntax Rules

- **Use `I` prefix or `Service` suffix** — `IUserService`, `UserService`.
- **Keep interfaces focused** — one responsibility per interface.
- **Use constructor injection** — dependencies are passed in.
- **Program to interfaces** — depend on abstractions, not concretions.
- **Return DTOs from interfaces** — not ORM entities.
- **Throw typed errors** — `ConflictError`, `NotFoundError`.
- **Use `readonly` dependencies** — prevent reassignment.
- **Document interfaces** — with JSDoc.

#### Constraints and Limitations

- **Interfaces add boilerplate** — one interface per service.
- **Over-abstraction** — small services may not need interfaces.
- **Dependency injection requires a container** — manual or framework-based.
- **Interface changes require updating implementations** — use IDE refactoring.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Service Interface with Testing

```typescript
// services/user.service.interface.ts
export interface IUserService {
  findById(id: string): Promise<UserDto | null>;
  create(dto: CreateUserDto): Promise<UserDto>;
}
```

```typescript
// services/user.service.ts
export class UserService implements IUserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly passwordService: IPasswordService,
  ) {}

  async findById(id: string): Promise<UserDto | null> {
    const user = await this.userRepository.findById(id);
    return user ? UserMapper.toDto(user) : null;
  }

  async create(dto: CreateUserDto): Promise<UserDto> {
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) throw new ConflictError('Email already registered');

    const passwordHash = await this.passwordService.hash(dto.password);
    const user = await this.userRepository.create({ ...dto, passwordHash });

    return UserMapper.toDto(user);
  }
}
```

```typescript
// services/user.service.test.ts
import { describe, it, expect, vi } from 'vitest';
import { UserService } from './user.service';
import { IUserService } from './user.service.interface';

describe('UserService', () => {
  it('creates a user with a hashed password', async () => {
    const mockRepository = {
      findById: vi.fn(),
      findByEmail: vi.fn().mockResolvedValue(null),
      create: vi.fn().mockImplementation((data) => Promise.resolve({
        id: 'user-1',
        email: data.email,
        name: data.name,
        role: 'USER',
        createdAt: new Date(),
      })),
    };

    const mockPasswordService = {
      hash: vi.fn().mockResolvedValue('hashed-password'),
      verify: vi.fn(),
    };

    const service: IUserService = new UserService(
      mockRepository as any,
      mockPasswordService,
    );

    const result = await service.create({
      email: 'alice@example.com',
      name: 'Alice',
      password: 'secure-password-123',
      role: 'user',
    });

    expect(result.email).toBe('alice@example.com');
    expect(mockPasswordService.hash).toHaveBeenCalledWith('secure-password-123');
    expect(mockRepository.create).toHaveBeenCalledWith(
      expect.objectContaining({ passwordHash: 'hashed-password' }),
    );
  });

  it('throws ConflictError if email exists', async () => {
    const mockRepository = {
      findByEmail: vi.fn().mockResolvedValue({ id: 'existing' }),
    };

    const service: IUserService = new UserService(
      mockRepository as any,
      {} as any,
    );

    await expect(
      service.create({
        email: 'alice@example.com',
        name: 'Alice',
        password: 'secure-password-123',
        role: 'user',
      }),
    ).rejects.toThrow(ConflictError);
  });
});
```

**Expected behaviour:** Tests use mock implementations of the repository and password service. The service is tested in isolation. The interface ensures the mock satisfies the contract.

**Why this works:** The `IUserService` interface defines the contract. The test provides mock dependencies. The service logic is tested without a database. TypeScript ensures the mocks satisfy the interface.

### Real-World Cases

- **Testing:** Mock services for unit tests.
- **Multiple implementations:** `PostgresUserRepository` and `InMemoryUserRepository`.
- **Dependency injection:** NestJS providers, tsyringe, InversifyJS.
- **Microservices:** Service interfaces for inter-service contracts.

---

## Core Concept 5: Error Types

### Definitions

**Core Definition:** Typed error types are custom error classes that extend `Error` and represent specific failure modes, enabling structured error handling and appropriate HTTP status mapping.

**Technical Definition:** TypeScript custom errors extend the built-in `Error` class, set `name` to the class name, and may add typed properties (e.g., `statusCode`, `code`, `details`). They are used with `instanceof` checks, exception filters, and centralized error handlers. Best practices: one error class per failure mode, a base `AppError` class, error codes for programmatic handling, and no sensitive information in messages. In Node.js, custom errors must set `Object.setPrototypeOf(this, new.target.prototype)` for `instanceof` to work correctly after transpilation.

**Beginner-Friendly Explanation:** A typed error is like a labelled box for a specific type of problem. Instead of throwing a generic `Error('Something went wrong')`, you throw a `NotFoundError('User not found')`. The error handler can then check "is this a NotFoundError? Then return 404." It's structured, predictable, and easy to handle.

### Purposes

- To represent specific failure modes.
- To enable `instanceof` checks for error handling.
- To map errors to HTTP status codes.
- To provide structured error responses.
- To avoid generic, unhelpful error messages.

### Syntax Rules and Structure

#### Base Error Class

```typescript
// errors/app.error.ts
export abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly code: string;

  constructor(message: string, public readonly details?: unknown) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}
```

#### Specific Error Classes

```typescript
// errors/not-found.error.ts
export class NotFoundError extends AppError {
  readonly statusCode = 404;
  readonly code = 'NOT_FOUND';

  constructor(resource: string, id?: string) {
    super(`${resource}${id ? ` with id ${id}` : ''} not found`);
  }
}

// errors/validation.error.ts
export class ValidationError extends AppError {
  readonly statusCode = 422;
  readonly code = 'VALIDATION_ERROR';

  constructor(
    message: string,
    public readonly issues: Array<{ field: string; message: string }>,
  ) {
    super(message);
  }
}

// errors/conflict.error.ts
export class ConflictError extends AppError {
  readonly statusCode = 409;
  readonly code = 'CONFLICT';

  constructor(message: string) {
    super(message);
  }
}

// errors/unauthorized.error.ts
export class UnauthorizedError extends AppError {
  readonly statusCode = 401;
  readonly code = 'UNAUTHORIZED';

  constructor(message = 'Authentication required') {
    super(message);
  }
}

// errors/forbidden.error.ts
export class ForbiddenError extends AppError {
  readonly statusCode = 403;
  readonly code = 'FORBIDDEN';

  constructor(message = 'Access denied') {
    super(message);
  }
}
```

#### Error Handler

```typescript
// middleware/error-handler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../errors/app.error';
import { logger } from '../logger';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction,
): void {
  if (res.headersSent) return next(err);

  if (err instanceof AppError) {
    logger.warn({ err, path: req.path }, 'Application error');
    res.status(err.statusCode).json({
      error: err.code,
      message: err.message,
      details: err.details,
    });
    return;
  }

  logger.error({ err, path: req.path }, 'Unhandled error');
  res.status(500).json({
    error: 'INTERNAL_SERVER_ERROR',
    message: process.env.NODE_ENV === 'production'
      ? 'An unexpected error occurred'
      : err.message,
  });
}
```

#### Syntax Rules

- **Extend `Error`** — always.
- **Set `name`** — `this.name = this.constructor.name`.
- **Call `Error.captureStackTrace`** — for clean stack traces.
- **Call `Object.setPrototypeOf`** — for `instanceof` to work after transpilation.
- **Define `statusCode` and `code`** — for HTTP mapping.
- **Use one class per failure mode** — `NotFoundError`, `ConflictError`.
- **Never include sensitive info in messages.**
- **Use `instanceof` for handling** — or check `code`.
- **Log with structured data** — `{ err, path }`.
- **Return generic messages in production** — for 5xx errors.

#### Constraints and Limitations

- **`instanceof` may fail after transpilation** — `Object.setPrototypeOf` fixes this.
- **Custom errors add boilerplate** — but improve clarity.
- **Error classes must be imported** — for `instanceof` checks.
- **Serialization** — `JSON.stringify(error)` does not include custom properties; define `toJSON`.
- **Stack traces may be lost** — if not captured correctly.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Error Hierarchy with Handler

```typescript
// errors/app.error.ts
export abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly code: string;

  constructor(message: string, public readonly details?: unknown) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
    Object.setPrototypeOf(this, new.target.prototype);
  }

  toJSON(): Record<string, unknown> {
    return {
      error: this.code,
      message: this.message,
      details: this.details,
    };
  }
}
```

```typescript
// errors/index.ts
export class NotFoundError extends AppError {
  readonly statusCode = 404;
  readonly code = 'NOT_FOUND';
  constructor(resource: string, id?: string) {
    super(`${resource}${id ? ` with id ${id}` : ''} not found`);
  }
}

export class ValidationError extends AppError {
  readonly statusCode = 422;
  readonly code = 'VALIDATION_ERROR';
  constructor(message: string, public readonly issues: Array<{ field: string; message: string }>) {
    super(message);
  }
}

export class ConflictError extends AppError {
  readonly statusCode = 409;
  readonly code = 'CONFLICT';
  constructor(message: string) {
    super(message);
  }
}

export class UnauthorizedError extends AppError {
  readonly statusCode = 401;
  readonly code = 'UNAUTHORIZED';
  constructor(message = 'Authentication required') {
    super(message);
  }
}

export class ForbiddenError extends AppError {
  readonly statusCode = 403;
  readonly code = 'FORBIDDEN';
  constructor(message = 'Access denied') {
    super(message);
  }
}
```

```typescript
// services/user.service.ts
import { ConflictError, NotFoundError } from '../errors';

export class UserService {
  async create(dto: CreateUserDto): Promise<UserDto> {
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) {
      throw new ConflictError('Email already registered');
    }
    // ...
  }

  async findById(id: string): Promise<UserDto> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new NotFoundError('User', id);
    }
    return UserMapper.toDto(user);
  }
}
```

```typescript
// middleware/error-handler.ts
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction): void {
  if (res.headersSent) return next(err);

  if (err instanceof AppError) {
    res.status(err.statusCode).json(err.toJSON());
    return;
  }

  logger.error({ err, path: req.path }, 'Unhandled error');
  res.status(500).json({
    error: 'INTERNAL_SERVER_ERROR',
    message: process.env.NODE_ENV === 'production' ? 'An unexpected error occurred' : err.message,
  });
}
```

**Expected behaviour:** `NotFoundError` returns `404` with `{ error: 'NOT_FOUND', message: 'User with id X not found' }`. `ConflictError` returns `409`. `ValidationError` returns `422` with details. Unhandled errors return `500` with a sanitised message in production.

**Why this works:** The error hierarchy provides typed failure modes. The error handler maps `AppError` subclasses to HTTP responses. `instanceof` works because of `Object.setPrototypeOf`. `toJSON` provides structured serialisation.

### Real-World Cases

- **REST APIs:** Mapping domain errors to HTTP status codes.
- **GraphQL:** Mapping errors to GraphQL error extensions.
- **gRPC:** Mapping errors to gRPC status codes.
- **Webhooks:** Structured error responses.

---

## Core Concept 6: Runtime Schema Validation Integration

### Definitions

**Core Definition:** Runtime schema validation integration uses libraries like Zod, TypeBox, or ArkType to define schemas that validate data at runtime while automatically inferring TypeScript types at compile time.

**Technical Definition:** Schema-first validation libraries (Zod, TypeBox, ArkType, Valibot) define schemas as values, from which TypeScript types are inferred (`z.infer<typeof Schema>`). This provides a single source of truth: the schema validates at runtime and types at compile time. TypeBox goes further by generating JSON Schema from TypeScript types, enabling OpenAPI documentation. ArkType uses a TypeScript-like syntax for schema definition. These libraries integrate with Express, Fastify, NestJS, and tRPC.

**Beginner-Friendly Explanation:** Normally, you write a TypeScript type AND a separate validation schema — and keep them in sync manually. Runtime schema validation integration says: "Write the schema once, and get both the runtime validation and the TypeScript type for free." The schema is the source of truth — if you change it, both the validation and the types update automatically.

### Purposes

- To eliminate duplication between types and validation.
- To ensure runtime data matches compile-time types.
- To generate OpenAPI documentation from schemas.
- To provide clear, structured validation errors.
- To enable type-safe parsing and serialisation.

### Syntax Rules and Structure

#### Zod

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email().max(255),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
  createdAt: z.coerce.date(),
});

// Infer TypeScript type
type User = z.infer<typeof UserSchema>;

// Parse and validate
const result = UserSchema.safeParse(data);
if (result.success) {
  const user: User = result.data; // Typed
} else {
  console.error(result.error.issues); // Structured errors
}
```

#### TypeBox (JSON Schema)

```typescript
import { Type, Static } from '@sinclair/typebox';

const UserSchema = Type.Object({
  id: Type.String({ format: 'uuid' }),
  email: Type.String({ format: 'email', maxLength: 255 }),
  name: Type.String({ minLength: 1, maxLength: 100 }),
  age: Type.Optional(Type.Integer({ minimum: 0, maximum: 150 })),
  role: Type.Union([Type.Literal('user'), Type.Literal('editor'), Type.Literal('admin')]),
});

// Infer TypeScript type
type User = Static<typeof UserSchema>;

// JSON Schema is available
console.log(JSON.stringify(UserSchema, null, 2));
```

#### ArkType

```typescript
import { type } from 'arktype';

const User = type({
  id: 'string.uuid',
  email: 'string.email <= 255',
  name: 'string >= 1 <= 100',
  'age?': 'number.integer >= 0 <= 150',
  role: "'user' | 'editor' | 'admin' = 'user'",
});

// Infer TypeScript type
type User = typeof User.infer;

// Parse and validate
const result = User(data);
if (result instanceof type.errors) {
  console.error(result.summary);
} else {
  const user: User = result; // Typed
}
```

#### Fastify Integration (TypeBox)

```typescript
import Fastify from 'fastify';
import { Type, Static } from '@sinclair/typebox';

const CreateUserSchema = Type.Object({
  email: Type.String({ format: 'email' }),
  name: Type.String({ minLength: 1, maxLength: 100 }),
  password: Type.String({ minLength: 12 }),
});

type CreateUserBody = Static<typeof CreateUserSchema>;

const app = Fastify();

app.post<{ Body: CreateUserBody }>('/users', {
  schema: { body: CreateUserSchema },
}, async (request, reply) => {
  // request.body is typed and validated
  const user = await userService.create(request.body);
  return reply.status(201).send(user);
});
```

#### Syntax Rules

- **Define the schema once** — use it for both validation and types.
- **Use `z.infer<typeof Schema>`** — for Zod types.
- **Use `Static<typeof Schema>`** — for TypeBox types.
- **Use `typeof Schema.infer`** — for ArkType types.
- **Use `safeParse`** — to avoid exceptions.
- **Use `.strict()`** — to reject unknown properties.
- **Use `coerce`** — for type coercion (numbers, dates, booleans).
- **Use `.transform()`** — for post-validation transformations.
- **Integrate with the framework** — Fastify `schema`, NestJS `ZodValidationPipe`.
- **Generate OpenAPI** — from JSON Schema (TypeBox, Zod).

#### Constraints and Limitations

- **Runtime overhead** — validation adds latency.
- **Schema complexity** — complex schemas are hard to read.
- **Library lock-in** — swapping libraries requires rewriting schemas.
- **JSON Schema limitations** — some Zod features are not expressible.
- **Type inference edge cases** — some transformations lose type information.
- **Bundle size** — Zod is ~50 KB; TypeBox is smaller.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Zod Integration (Express)

```typescript
// schemas/user.schema.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase().trim(),
  name: z.string().min(1).max(100).trim(),
  password: z.string().min(12).max(128),
  age: z.coerce.number().int().min(0).max(150).optional(),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
}).strict();

export const UpdateUserSchema = CreateUserSchema.partial().omit({ password: true });

export const ListUsersSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  role: z.enum(['user', 'editor', 'admin']).optional(),
  search: z.string().max(100).optional(),
});

export type CreateUserDto = z.infer<typeof CreateUserSchema>;
export type UpdateUserDto = z.infer<typeof UpdateUserSchema>;
export type ListUsersDto = z.infer<typeof ListUsersSchema>;
```

```typescript
// middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema } from 'zod';

export function validate(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction): void => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      res.status(400).json({
        error: 'ValidationError',
        details: result.error.issues.map((issue) => ({
          field: issue.path.join('.'),
          message: issue.message,
        })),
      });
      return;
    }
    req.body = result.data; // Replace with validated, coerced data
    next();
  };
}
```

```typescript
// routes/user.routes.ts
import { Router } from 'express';
import { CreateUserSchema, UpdateUserSchema, ListUsersSchema } from '../schemas/user.schema';
import { validate } from '../middleware/validate';

const router = Router();

router.post('/users', validate(CreateUserSchema), async (req, res, next) => {
  try {
    // req.body is typed as CreateUserDto
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
});

router.patch('/users/:id', validate(UpdateUserSchema), async (req, res, next) => {
  try {
    const user = await userService.update(req.params.id, req.body);
    res.json(user);
  } catch (err) {
    next(err);
  }
});

export { router };
```

**Expected behaviour:** The middleware validates and coerces the request body. `req.body` is typed as `CreateUserDto` after validation. Invalid input returns `400` with field-level errors. The service receives validated, typed data.

**Why this works:** Zod is the single source of truth for validation and types. The middleware applies the schema at the boundary. `safeParse` returns structured errors. The service receives validated data.

### Real-World Cases

- **REST APIs:** Request/response validation.
- **GraphQL:** Input validation with Zod.
- **tRPC:** Input/output validation with Zod.
- **Fastify:** JSON Schema validation with TypeBox.
- **OpenAPI:** Generate specs from Zod or TypeBox schemas.

---

## References

- TypeScript Documentation — https://www.typescriptlang.org/docs/
- TypeScript Documentation — Declaration Merging — https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Documentation — Module Augmentation — https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- Express Documentation — TypeScript — https://expressjs.com/en/guide/typescript.html
- Fastify Documentation — TypeScript — https://fastify.dev/docs/latest/Reference/TypeScript/
- NestJS Documentation — TypeScript — https://docs.nestjs.com/
- Prisma Documentation — TypeScript — https://www.prisma.io/docs/concepts/overview/why-prisma
- Prisma Documentation — `Prisma.UserGetPayload` — https://www.prisma.io/docs/orm/prisma-client/type-safety/operating-against-partial-structures-of-model-types
- TypeORM Documentation — Entities — https://typeorm.io/entities
- Mongoose Documentation — TypeScript — https://mongoosejs.com/docs/typescript.html
- Drizzle Documentation — Type Inference — https://orm.drizzle.team/docs/type-inference
- Zod Documentation — https://zod.dev/
- TypeBox Documentation — https://github.com/sinclairzx81/typebox
- ArkType Documentation — https://arktype.io/
- Valibot Documentation — https://valibot.dev/
- class-validator Documentation — https://github.com/typestack/class-validator
- Vitest Documentation — Mocking — https://vitest.dev/guide/mocking.html
- tsyringe — Dependency Injection — https://github.com/microsoft/tsyringe
- InversifyJS — Dependency Injection — https://inversify.io/
- Node.js Documentation — Errors — https://nodejs.org/api/errors.html
- MDN Web Docs — `Error` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
- Total TypeScript — https://www.totaltypescript.com/
- Type Challenges — https://github.com/type-challenges/type-challenges
- DefinitelyTyped — https://github.com/DefinitelyTyped/DefinitelyTyped