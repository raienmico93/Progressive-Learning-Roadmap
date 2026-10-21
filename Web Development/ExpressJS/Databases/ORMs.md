# Express.js ORMs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An Object-Relational Mapper (ORM) is a library that maps database tables to application-level objects, allowing developers to query and manipulate data using the host language (JavaScript/TypeScript) instead of writing SQL directly.

**Technical Definition**
ORMs implement the **Data Mapper** or **Active Record** pattern to bridge the object-oriented application domain and the relational database schema. They generate SQL from method calls, hydrate database rows into typed objects, manage relationships through associations or relations, and often provide migration tooling, transaction management, and connection pooling. In the Express.js ecosystem, the five leading ORMs are **Prisma** (schema-first, auto-generated client), **Sequelize** (mature, promise-based Active Record/Data Mapper hybrid), **TypeORM** (decorator-based, dual Active Record/Data Mapper), **Drizzle** (SQL-first, lightweight TypeScript ORM), and **MikroORM** (TypeScript-first, Data Mapper with Unit of Work and Identity Map).

**Beginner-Friendly Explanation**
An ORM is a translator between your JavaScript code and your database. Instead of writing SQL strings like `SELECT * FROM users WHERE id = 1`, you write `userRepository.findOne(1)`, and the ORM generates the SQL, runs it, and gives you back a JavaScript object. It also handles relationships (a user’s posts), migrations (changing the database schema over time), and transactions (making sure multiple changes succeed or fail together). The trade-off is abstraction: ORMs make simple things easy but can hide performance problems like N+1 queries if you are not careful.

---

### Key Characteristics

- **Productivity** — ORMs eliminate boilerplate CRUD SQL and provide a consistent API across databases.
- **Type safety varies sharply** — Prisma generates types from a schema, Drizzle infers types from TypeScript definitions, MikroORM uses typed entities and the EntityManager, TypeORM relies on decorators and generics, and Sequelize’s TypeScript support is improving but less native.
- **Abstraction leakage is real** — Every ORM eventually requires raw SQL or query-builder escape hatches for complex queries and performance tuning.
- **N+1 queries are the most common performance trap** — ORMs make it deceptively easy to trigger hundreds of queries inside loops; eager loading, joins, or batching are essential.
- **Migration tooling differs** — Prisma Migrate and Sequelize CLI are declarative and integrated; TypeORM and MikroORM generate migrations from entities; Drizzle uses Drizzle Kit for schema push and migration generation.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A database driver installed for the chosen ORM (`pg`, `mysql2`, `better-sqlite3`, or `mssql`).
- For TypeScript-first ORMs (Prisma, Drizzle, MikroORM, TypeORM): TypeScript 5.0+ with strict mode enabled.
- Environment variables configured for database credentials (`dotenv`).
- A database server or embedded database available for development.

---

### Related Programming Areas

- **Database drivers** — ORMs sit on top of `pg`, `mysql2`, `better-sqlite3`, and `mssql`.
- **Query builders** — Knex and Kysely offer a middle ground between raw SQL and full ORMs.
- **Migrations** — Schema version control is a core ORM responsibility.
- **Transactions** — ACID guarantees through ORM transaction APIs.
- **Performance optimization** — N+1 prevention, connection pooling, and query logging.

---

## Core Concept 1: Prisma

### Definitions

**Core Definition**
Prisma is a modern ORM for Node.js and TypeScript that prioritizes developer experience and type safety through a declarative, schema-first approach and powerful tooling.

**Technical Definition**
Prisma uses the **Prisma Schema Language (PSL)** to define models, relations, and data sources in a `schema.prisma` file. The Prisma CLI generates a **type-safe client** (`@prisma/client`) with strong IDE support and autocompletion. Prisma Migrate manages declarative migrations, and Prisma Studio provides a visual database browser. Since v7 (November 2025), Prisma uses a **Rust-free TypeScript/WASM engine**, resulting in faster queries and a roughly 90% smaller bundle. Prisma 7 also changed connection configuration: `url` is no longer allowed in `schema.prisma`; the CLI reads it from `prisma.config.ts`.

**Beginner-Friendly Explanation**
Prisma is the “schema-first” ORM. You write a schema file describing your tables, run a command to generate a client, and then use that client with full TypeScript autocomplete. It feels like using a typed SDK for your database.

---

### Purposes

- To provide end-to-end type safety from the database schema to the application code.
- To offer declarative migrations and a visual database browser (Prisma Studio).
- To support both SQL databases and MongoDB through a single, consistent client API.
- To reduce boilerplate through an intuitive, auto-generated client.

---

### Syntax Rules and Structure

#### Complete General Syntax: Prisma Schema and Client Usage

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int
}
```

```javascript
// usage
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// SELECT with relation
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true }
});

// Nested write (transactional)
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    posts: { create: [{ title: 'Hello' }, { title: 'World' }] }
  },
  include: { posts: true }
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `schema.prisma` | Single source of truth for models, relations, and datasource |
| `@prisma/client` | Auto-generated type-safe query client |
| `include` | Eager-loads relations in a single query |
| Nested writes | Atomic multi-table operations through Prisma’s transaction API |

#### Syntax Rules

- **Prisma 7**: `url` must be configured in `prisma.config.ts`, not in `schema.prisma`.
- Use `include` or `select` to load relations eagerly; lazy loading is not supported the way it is in some ORMs.
- Nested writes are transactional by default.
- Always create **one global `PrismaClient` instance**; do not instantiate it per request.

#### Constraints and Limitations

- Prisma’s generated queries can hide N+1 patterns; use `include` or `select` deliberately.
- Complex queries may require `$queryRaw` (TypedSQL in Prisma 7).
- Prisma is less flexible than SQL-first ORMs for highly optimized or unconventional queries.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Route with Prisma and Relation Loading

```javascript
const express = require('express');
const { PrismaClient } = require('@prisma/client');
const app = express();

const prisma = new PrismaClient();

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await prisma.user.findUnique({
      where: { id: Number(req.params.id) },
      select: { id: true, email: true, name: true, posts: { select: { title: true } } }
    });
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1 → 200 { "id": 1, "email": "alice@example.com", "name": "Alice", "posts": [{ "title": "Hello" }] }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `select` fetches only the specified fields and eagerly loads the `posts` relation in a single query.

---

### Real-World Cases

- **Rapid development** — Prisma’s generated client and migrations accelerate feature delivery.
- **TypeScript APIs** — End-to-end type safety from schema to Express handlers.
- **Multi-database applications** — Prisma supports PostgreSQL, MySQL, SQL Server, SQLite, and MongoDB.

---

## Core Concept 2: Sequelize

### Definitions

**Core Definition**
Sequelize is a mature, feature-rich, promise-based Node.js ORM that supports PostgreSQL, MySQL, MariaDB, SQLite, and SQL Server, offering models, associations, transactions, migrations, and lifecycle hooks.

**Technical Definition**
Sequelize defines models with `sequelize.define()` or ES6 classes, and associations with `belongsTo`, `hasOne`, `hasMany`, and `belongsToMany`. It provides a comprehensive query API, ACID-compliant transactions with savepoints and CLS (continuation-local storage) support, and a CLI for migrations and seeders. The `.sequelizerc` file configures CLI paths. Sequelize supports both Promise-based async/await and callback-style APIs.

**Beginner-Friendly Explanation**
Sequelize is the veteran JavaScript ORM. You define models, set up associations between them, and then use the models to query and mutate data. It has been around for years, supports almost every SQL database, and includes a full migration CLI.

---

### Purposes

- To provide a mature, battle-tested ORM with broad database support.
- To manage complex associations (1:1, 1:N, N:M) and polymorphic relationships.
- To offer ACID transactions with savepoints and CLS support.
- To provide CLI-driven migrations and seeders for schema evolution.

---

### Syntax Rules and Structure

#### Complete General Syntax: Connection, Model, and Association

```javascript
const { Sequelize, DataTypes } = require('sequelize');

const sequelize = new Sequelize(process.env.DATABASE_URL, {
  dialect: 'postgres',
  logging: false,
  pool: { max: 10, min: 0, idle: 10000 }
});

const User = sequelize.define('User', {
  email: { type: DataTypes.STRING, allowNull: false, unique: true, validate: { isEmail: true } },
  name: { type: DataTypes.STRING, allowNull: false }
}, { timestamps: true });

const Post = sequelize.define('Post', {
  title: { type: DataTypes.STRING, allowNull: false }
});

User.hasMany(Post, { foreignKey: 'authorId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'authorId', as: 'author' });
```

#### Complete General Syntax: Transaction

```javascript
await sequelize.transaction(async (t) => {
  await User.decrement('balance', { by: 100, where: { id: 1 }, transaction: t });
  await User.increment('balance', { by: 100, where: { id: 2 }, transaction: t });
});
```

#### Syntax Rules

- **Use migrations, not `sync({ alter: true })`, in production** — `sync` is for development only.
- **Pass the transaction object** to every query inside a transaction.
- **Define associations in the models index file** so they are registered before queries.
- **Use `pool` configuration** to cap connections.

#### Constraints and Limitations

- Sequelize’s TypeScript support is improving but less native than Prisma or MikroORM.
- Association aliases (`as`) are required when multiple associations exist between the same models.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Route with Sequelize Eager Loading

```javascript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findByPk(req.params.id, {
      attributes: ['id', 'email', 'name'],
      include: [{ model: Post, as: 'posts', attributes: ['title'] }]
    });
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
GET /users/1 → 200 { "id": 1, "email": "alice@example.com", "name": "Alice", "posts": [{ "title": "Hello" }] }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `include` eager-loads the associated posts, avoiding an N+1 query.

---

### Real-World Cases

- **Legacy Express applications** — Sequelize’s long history makes it a safe choice for existing codebases.
- **Multi-dialect applications** — PostgreSQL, MySQL, MariaDB, SQLite, and SQL Server support.
- **Teams needing built-in migrations** — The Sequelize CLI provides a complete migration and seeding workflow.

---

## Core Concept 3: TypeORM

### Definitions

**Core Definition**
TypeORM is a flexible and mature ORM for TypeScript and JavaScript that supports both the Active Record and Data Mapper patterns, using decorators to define entities and their relationships.

**Technical Definition**
TypeORM defines entities as TypeScript classes decorated with `@Entity()`, `@Column()`, `@PrimaryGeneratedColumn()`, and relation decorators (`@ManyToOne`, `@OneToMany`, `@ManyToMany`). A `DataSource` holds the connection configuration and manages repositories. TypeORM supports eager and lazy loading, a powerful query builder, and migration generation from entity changes. **`synchronize` must never be used in production**; migrations should always be generated and reviewed.

**Beginner-Friendly Explanation**
TypeORM is the decorator-based ORM for TypeScript. You define a class for each table, add decorators to its properties, and TypeORM maps the class to the database. It supports both the “active record” style (methods on the entity) and the “data mapper” style (repositories).

---

### Purposes

- To provide a decorator-based, class-oriented ORM for TypeScript projects.
- To support both Active Record and Data Mapper architectural patterns.
- To offer a powerful SQL-like query builder for complex queries.
- To generate migrations from entity changes.

---

### Syntax Rules and Structure

#### Complete General Syntax: Entity and DataSource

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany, DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column({ nullable: true })
  name: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

export const AppDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [User, Post],
  migrations: ['src/migrations/*.ts'],
  synchronize: false // NEVER true in production
});
```

#### Complete General Syntax: Repository Query with Relation

```typescript
const user = await AppDataSource.getRepository(User).findOne({
  where: { id: 1 },
  relations: { posts: true },
  select: { id: true, email: true, name: true, posts: { title: true } }
});
```

#### Syntax Rules

- **Never use `synchronize: true` in production**; always generate and review migrations.
- **Use `relations` or `leftJoinAndSelect`** to eager-load associations; lazy relations require Promises.
- **Initialize the `DataSource` once** at application startup.
- **Use the query builder** for complex queries; eager relations are disabled when using `QueryBuilder` unless explicitly joined.

#### Constraints and Limitations

- TypeORM’s type safety is weaker than Prisma or MikroORM; entity properties can be `undefined` before hydration.
- The dual Active Record/Data Mapper support can lead to inconsistent patterns across a codebase.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Route with TypeORM Relations

```typescript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await AppDataSource.getRepository(User).findOne({
      where: { id: Number(req.params.id) },
      relations: { posts: true },
      select: { id: true, email: true, name: true, posts: { title: true } }
    });
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
GET /users/1 → 200 { "id": 1, "email": "alice@example.com", "name": "Alice", "posts": [{ "title": "Hello" }] }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `relations` eagerly loads the posts, and `select` limits the fields returned.

---

### Real-World Cases

- **NestJS applications** — TypeORM is the default ORM in NestJS.
- **Decorator-heavy TypeScript codebases** — Entities and dependency injection integrate naturally.
- **Complex relational schemas** — The query builder handles advanced joins and subqueries.

---

## Core Concept 4: Drizzle

### Definitions

**Core Definition**
Drizzle is a lightweight, TypeScript-first, SQL-centric ORM with zero dependencies, designed for serverless environments and developers who want SQL-like control with full type safety.

**Technical Definition**
Drizzle defines schemas in TypeScript using `pgTable`, `mysqlTable`, or `sqliteTable` functions. It offers **dual query APIs**: a SQL-like query builder (`db.select().from(users).where(eq(users.id, 1))`) and a relational query API (`db.query.users.findMany({ with: { posts: true } })`). Drizzle Kit manages migrations and schema pushes. The library is approximately 7.4 kB min+gzip with zero runtime dependencies, making it well suited to edge and serverless runtimes. **Use `.with()` for relational queries** to avoid N+1 problems—it fetches related data in a single SQL query.

**Beginner-Friendly Explanation**
Drizzle is the “if you know SQL, you know Drizzle” ORM. It is tiny, fast, and fully TypeScript. You define your schema in TypeScript, and you can query it in a way that looks almost exactly like SQL—or use the relational API for a more ORM-like feel.

---

### Purposes

- To provide maximum type safety with minimal abstraction and zero dependencies.
- To support edge and serverless deployments with a tiny bundle.
- To offer both SQL-like and relational query APIs.
- To generate migrations and schema pushes through Drizzle Kit.

---

### Syntax Rules and Structure

#### Complete General Syntax: Schema Definition and Query

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { eq } from 'drizzle-orm';
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name')
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').references(() => users.id)
});

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle({ client: pool });
```

```typescript
// SQL-like query
const user = await db.select().from(users).where(eq(users.id, 1));

// Relational query (avoids N+1)
const userWithPosts = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: { posts: true }
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `pgTable()` | Defines a table schema in TypeScript |
| `db.select().from()` | SQL-like query builder |
| `db.query.*.findMany({ with: ... })` | Relational query API |
| `.with()` | Eager-loads relations in a single SQL query, avoiding N+1 |

#### Syntax Rules

- **Use `.with()` for relational queries** — it fetches related data in a single SQL query.
- **Use the `tx` parameter** inside transaction callbacks, not `db`.
- **Use Drizzle Kit** (`drizzle-kit generate`, `drizzle-kit push`) for migrations and schema changes.
- **Export the database instance once** and reuse it.

#### Constraints and Limitations

- Drizzle is newer and has a smaller ecosystem than Prisma or Sequelize.
- The SQL-like API requires SQL knowledge; the relational API is still evolving (relations v2 in 2026).
- Migration tooling is less mature than Prisma Migrate.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Route with Drizzle Relational Query

```typescript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await db.query.users.findFirst({
      where: eq(users.id, Number(req.params.id)),
      columns: { id: true, email: true, name: true },
      with: { posts: { columns: { title: true } } }
    });
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
GET /users/1 → 200 { "id": 1, "email": "alice@example.com", "name": "Alice", "posts": [{ "title": "Hello" }] }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `.with()` fetches the posts relation in the same SQL query, avoiding the N+1 pattern.

---

### Real-World Cases

- **Serverless and edge deployments** — Tiny bundle and zero dependencies.
- **TypeScript-first teams** — Full type inference from schema definitions.
- **Performance-sensitive applications** — SQL-like control without the overhead of a full ORM.

---

## Core Concept 5: MikroORM

### Definitions

**Core Definition**
MikroORM is a TypeScript-first ORM for Node.js built on the Data Mapper, Unit of Work, and Identity Map patterns, emphasizing type safety, clean architecture, and request-scoped entity management.

**Technical Definition**
MikroORM implements three Fowler patterns: **Data Mapper** (entities are plain objects with no database knowledge), **Unit of Work** (tracks all changes during a request and persists them in a single transaction), and **Identity Map** (ensures each database row is represented by exactly one entity instance within a request context). The `EntityManager` is the main facade; `em.flush()` computes change sets and commits them. Because the `EntityManager` is stateful, it **must not be shared across requests**—use `orm.em.fork()` or the `RequestContext` helper (backed by Node’s `AsyncLocalStorage`) to create a request-scoped `EntityManager`.

**Beginner-Friendly Explanation**
MikroORM is the “clean architecture” ORM. Your entities are plain TypeScript classes with no database code in them. You ask the EntityManager for objects, change them however you like, and then call `flush()`—MikroORM figures out what changed and writes it all in one transaction. Each request gets its own EntityManager so that entities from different requests do not leak into each other.

---

### Purposes

- To provide a clean separation between domain entities and persistence logic.
- To enable automatic change tracking and transactional persistence through the Unit of Work.
- To guarantee entity identity within a request through the Identity Map.
- To support request-scoped isolation through `AsyncLocalStorage`.

---

### Syntax Rules and Structure

#### Complete General Syntax: Entity and Request Context

```typescript
import { Entity, PrimaryKey, Property, ManyToOne, OneToMany, Collection } from '@mikro-orm/core';
import { MikroORM } from '@mikro-orm/postgresql';

@Entity()
export class User {
  @PrimaryKey()
  id!: number;

  @Property({ unique: true })
  email!: string;

  @Property({ nullable: true })
  name?: string;

  @OneToMany(() => Post, (post) => post.author)
  posts = new Collection<Post>(this);
}

@Entity()
export class Post {
  @PrimaryKey()
  id!: number;

  @Property()
  title!: string;

  @ManyToOne(() => User)
  author!: User;
}

const orm = await MikroORM.init({
  entities: [User, Post],
  dbName: 'my_db',
  clientUrl: process.env.DATABASE_URL
});

// Request-scoped EntityManager
app.use((req, res, next) => {
  RequestContext.create(orm.em, next);
});
```

#### Complete General Syntax: Query and Flush

```typescript
const user = await orm.em.findOne(User, { id: 1 }, { populate: ['posts'] });
user.name = 'Alice Smith'; // tracked by Unit of Work
await orm.em.flush(); // single transaction, change sets computed automatically
```

#### Syntax Rules

- **Never share the EntityManager across requests** — fork it per request (`orm.em.fork()`) or use `RequestContext.create()`.
- **Use `populate`** to eager-load relations; otherwise, related entities are loaded as references.
- **Call `em.flush()`** to persist changes; it computes change sets and commits in a single transaction.
- **Use `em.merge()`** to re-attach detached entities.

#### Constraints and Limitations

- MikroORM is stateful by design; failing to fork the EntityManager causes memory growth and stale data.
- The learning curve is steeper than Prisma or Sequelize due to the Unit of Work and Identity Map patterns.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Route with Request-Scoped EntityManager

```typescript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await req.em.findOne(User, Number(req.params.id), {
      populate: ['posts'],
      fields: ['id', 'email', 'name', 'posts.title']
    });
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
GET /users/1 → 200 { "id": 1, "email": "alice@example.com", "name": "Alice", "posts": [{ "title": "Hello" }] }
GET /users/99 → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- `populate` eagerly loads posts; the request-scoped EntityManager ensures the user and its posts share a single identity map and unit of work.

---

### Real-World Cases

- **DDD and clean architecture projects** — Entities are plain objects, persistence is isolated.
- **Complex transactional workflows** — Unit of Work batches changes into a single transaction.
- **Request-scoped isolation** — `AsyncLocalStorage` prevents entity leakage across requests.

---

## ORM Comparison

| Feature | Prisma | Sequelize | TypeORM | Drizzle | MikroORM |
|---------|--------|-----------|---------|---------|----------|
| **Paradigm** | Schema-first, generated client | Active Record / Data Mapper | Active Record / Data Mapper | SQL-first / relational | Data Mapper + UoW + Identity Map |
| **Type safety** | Excellent (generated) | Improving, less native | Moderate | Excellent (inferred) | Excellent (typed entities) |
| **Schema definition** | PSL (`schema.prisma`) | `sequelize.define()` / classes | Decorated classes | TypeScript `pgTable()` | Decorated classes |
| **Migrations** | Prisma Migrate (declarative) | CLI migrations | Generated from entities | Drizzle Kit | Schema generator |
| **Performance** | Rust-free engine (v7) | Mature, moderate | Moderate | Fast, lightweight | Full ORM with UoW overhead |
| **Edge/serverless** | Yes (v7, smaller bundle) | Node-oriented | Node-oriented | Excellent (tiny, zero deps) | Node-oriented |
| **N+1 mitigation** | `include` / `select` | `include` | `relations` / `leftJoinAndSelect` | `.with()` | `populate` |
| **Learning curve** | Low–medium | Low–medium | Medium | Medium | High |
| **Best for** | Most projects, rapid dev | Legacy/JS codebases, multi-dialect | NestJS, decorator-based TS | Edge/serverless, SQL-lovers | DDD, clean architecture |

**Decision guidance:** Prisma is the strongest default for new TypeScript projects that value developer experience and generated types. Drizzle is the best fit for serverless, edge, and teams that want SQL-level control with type safety. Sequelize remains a safe choice for mature JavaScript codebases and multi-dialect requirements. TypeORM is the natural choice for NestJS and decorator-heavy TypeScript. MikroORM shines in domain-driven design and clean-architecture projects that benefit from the Unit of Work and Identity Map.

---

## ORM Best Practices

### Definitions

**Core Definition**
ORM best practices are the design and operational patterns that keep ORM-based data access safe, performant, and maintainable in Express applications.

**Technical Definition**
Best practices include: creating a **single global ORM instance** and reusing it; **preventing N+1 queries** through eager loading, joins, or batching; **wrapping multi-step writes in transactions**; **using migrations** (never `synchronize` in production); **pooling connections** appropriately; **isolating ORM code** in a repository layer; **logging and monitoring queries**; and **using raw SQL judiciously** when the ORM’s abstraction leaks.

### Purposes

- To prevent performance degradation from N+1 queries and unbounded result sets.
- To maintain data integrity through proper transaction handling.
- To keep schema changes reproducible and auditable through migrations.
- To avoid connection leaks and pool exhaustion.

### Syntax Rules and Structure

- **Single instance** — Create one PrismaClient, Sequelize instance, DataSource, Drizzle `db`, or MikroORM instance per application.
- **Eager-load relations** — Use `include` (Prisma, Sequelize), `relations` (TypeORM), `.with()` (Drizzle), or `populate` (MikroORM).
- **Always pass the transaction object** to every query inside a transaction.
- **Use migrations** — Prisma Migrate, Sequelize CLI, TypeORM migrations, Drizzle Kit, or MikroORM’s schema generator.
- **Pool connections** — Configure pool size to respect the database’s connection limit.
- **Log slow queries** — Enable query logging in development and monitor slow queries in production.
- **Use raw SQL when needed** — Every ORM provides an escape hatch (`$queryRaw`, `sequelize.query`, `query()`, `sql`, `em.getConnection().execute`).

### Constraints and Limitations

- ORMs make N+1 queries easy to write accidentally; eager loading is essential.
- Abstraction leaks: complex queries and performance-critical paths may require raw SQL.
- Each ORM’s transaction and connection-pool APIs differ; do not mix patterns across ORMs in the same application.

---

## References

- Top TypeScript ORM 2026 · Bytebase — https://www.bytebase.com/blog/top-typescript-orm/
- Node.js ORMs, Query Builders & Database Libraries Compared (2026) · Prisma — https://www.prisma.io/dataguide/database-tools/top-nodejs-orms-query-builders-and-database-libraries
- Architecture Overview · MikroORM — https://mikro-orm.io/docs/next/architecture
- Unit of Work and Transactions · MikroORM — https://mikro-orm.io/docs/unit-of-work
- Express Sequelize ORM · Compile-N-Run — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/6-express-database-integration/4-express-sequelize-orm.mdx
- How to Use Express with TypeORM · OneUptime — https://oneuptime.com/blog/post/2026-01-26-express-typeorm/view
- Setting up Drizzle ORM for your TypeScript Project · Medium — https://medium.com/@maximilianogbuabor/setting-up-drizzle-orm-for-your-typescript-project-a-straight-forward-guide-3a0e7fbb4645
- Drizzle relational queries and transactions · SkillsAuth — https://skillsauth.com/skills/api-database-drizzle
- Prisma relation queries (nested reads, `include`, `select`) — https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries
- RequestContext · MikroORM — https://mikro-orm.io/docs/request-context
- Node.js REST API Audit: N+1 Loops and Connection Pool Starvation · Abins Chandran — https://www.abinschandran.in/blog/nodejs-rest-api-audit-checklist
- Database connections (Prisma ORM v7) — https://www.prisma.io/docs/orm/overview/databases/database-connections