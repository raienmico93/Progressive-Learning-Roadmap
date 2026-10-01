# ORMs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An Object-Relational Mapper (ORM) is a library that maps database tables to application-level objects, enabling developers to interact with relational databases using their programming language's idioms instead of writing raw SQL.

**Technical Definition:** An ORM is a software layer that abstracts the relational database model into an object-oriented domain model. It translates between the two paradigms: tables become classes, rows become instances, columns become properties, and foreign key relationships become object references. In the Node.js ecosystem, ORMs such as Prisma, Sequelize, TypeORM, and Drizzle provide query builders, migration tools, type safety (in varying degrees), and lifecycle hooks.

**Beginner-Friendly Explanation:** An ORM is like a translator between your JavaScript application and your database. Your app speaks JavaScript (objects, classes, arrays), and the database speaks SQL (tables, rows, joins). The ORM translates back and forth so you can write `user.posts` instead of `SELECT * FROM posts WHERE user_id = 1`.

### Key Characteristics

- **Prisma:** Schema-first ORM with its own DSL (PSL), generates a fully type-safe client, and offers the highest npm download volume (55.3M/month).
- **Sequelize:** Promise-based, mature ORM with a large historical install base (11.8M downloads/month), suitable for legacy systems.
- **TypeORM:** Decorator-based ORM supporting both Active Record and Data Mapper patterns, widely used in NestJS applications.
- **Drizzle:** Lightweight, headless, TypeScript-first ORM with zero dependencies, SQL-like syntax, and no codegen step.
- **Schema synchronisation:** `synchronize: true` auto-creates schema from entities (development only); migrations are the production standard.
- **Lifecycle hooks:** All ORMs support hooks (beforeCreate, afterUpdate, etc.) for cross-cutting concerns.
- **Relations:** Eager and lazy loading, cascade deletes, and referential integrity are configurable per relation.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **SQL fundamentals:** SELECT, INSERT, UPDATE, DELETE, JOIN, and transactions.
- **TypeScript (recommended):** Most modern ORMs are TypeScript-first.
- **Database driver:** `pg` (PostgreSQL), `mysql2` (MySQL), or `better-sqlite3` (SQLite).

### Related Programming Areas

- **Relational Databases:** PostgreSQL, MySQL, MariaDB, SQL Server.
- **Node.js Database Connectivity:** Drivers, pooling, prepared statements.
- **SQL Integration:** Transactions, joins, query builders, raw SQL.
- **API Development:** DTOs, validation, and data access layers.

### Core Concepts

1. **Prisma** — PSL, client generation, type safety limits, performance tuning.
2. **Sequelize** — promise-based MVC patterns and legacy systems.
3. **TypeORM** — Data Mapper vs. Active Record with decorators.
4. **Drizzle** — headless schema definition, SQL-like syntax, compile-time type safety.
5. **Entity/Model Concepts** — synchronisation, code-first mapping, hooks, and timestamps.
6. **Relations** — eager vs. lazy loading, cascade deletes, referential integrity.
7. **Migrations** — declarative vs. imperative, zero-downtime, seed scripts, and version control.

---

## Core Concept 1: Prisma

### Sub-Feature 1.1: Prisma Schema Language (PSL) and Automated Client Generation

#### Definitions

**Core Definition:** Prisma Schema Language (PSL) is a declarative domain-specific language for defining database schemas, from which Prisma generates a fully type-safe client and database migrations.

**Technical Definition:** PSL is a domain-specific language designed for defining database schemas. Its syntax is concise, readable, and focuses exclusively on modelling database entities and relationships. The Prisma schema (`schema.prisma`) consists of three parts: data sources (database connection), generators (what clients to generate), and the data model definition (models and relations). Whenever a Prisma command is invoked, the CLI reads the schema to generate client code or create migrations.

**Beginner-Friendly Explanation:** PSL is like a blueprint for your database. Instead of writing SQL `CREATE TABLE` statements, you write a simple model definition, and Prisma generates everything else — the database tables, the TypeScript types, and the query client.

#### Purposes

- To provide a single, readable source of truth for the database schema.
- To automatically generate a type-safe client for database queries.
- To generate migrations from schema changes.
- To enable AI tooling and code generation from a declarative contract.

#### Syntax Rules and Structure

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int
}
```

| Block | Purpose |
|-------|---------|
| `datasource` | Database connection configuration. |
| `generator` | What clients to generate (Prisma Client). |
| `model` | Data model definitions with fields and relations. |
| `@id` | Primary key. |
| `@default` | Default value (e.g., `autoincrement()`). |
| `@unique` | Unique constraint. |
| `@relation` | Relationship definition. |

**Client generation:**
```bash
npx prisma generate  # Generate Prisma Client
npx prisma migrate dev --name init  # Create and apply migration
```

**Usage:**
```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

const users = await prisma.user.findMany({ include: { posts: true } });
```

**Constraints and Limitations:**
- PSL is a proprietary DSL; it is not standard SQL.
- Client generation is a required build step; forgetting to run `prisma generate` after schema changes causes type mismatches.
- Prisma v7 introduced significant TypeScript type-checking performance regressions in large codebases (406 models, 9,300-line schema): `tsc --noEmit` ran for over 2 minutes and `tsgo` timed out at 3 minutes, whereas v6 completed in ~9 seconds. The regression was traced to a single line change in the `OmitOpts` generic default.

#### Annotated Code Example

```typescript
// prisma-example.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create with nested relation
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@example.com',
      posts: {
        create: [{ title: 'First Post' }, { title: 'Second Post' }],
      },
    },
    include: { posts: true },
  });
  console.log('Created:', user);

  // Query with eager loading
  const usersWithPosts = await prisma.user.findMany({
    include: { posts: true },
  });
  console.log('Users with posts:', usersWithPosts);

  await prisma.$disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
Created: { id: 1, name: 'Alice', email: 'alice@example.com', posts: [ { id: 1, title: 'First Post', authorId: 1 }, { id: 2, title: 'Second Post', authorId: 1 } ] }
Users with posts: [ { id: 1, name: 'Alice', email: 'alice@example.com', posts: [ ... ] } ]
```

**Why this output:** The PSL model defines `User` and `Post` with a one-to-many relation. The generated Prisma Client provides `prisma.user.create` with nested `posts` creation. The `include` option eagerly loads the related posts in a single query, avoiding the N+1 problem.

#### Real-World Cases

- **SaaS applications:** Multi-tenant data models with complex relations.
- **Rapid prototyping:** Schema-first development with instant client generation.
- **Type-safe APIs:** Full-stack TypeScript with Prisma-generated types shared between backend and frontend.

---

### Sub-Feature 1.2: Type Safety Limits and Performance-Tuning Patterns

#### Definitions

**Core Definition:** Prisma's type safety is generated from the schema, but TypeScript compilation performance degrades on large schemas, and certain query patterns require raw SQL or Prisma Client extensions for optimal performance.

**Technical Definition:** Prisma achieves type safety through a code-generation step: when you modify `schema.prisma` and run `prisma generate`, the CLI writes custom TypeScript types into `node_modules/.prisma/client`. Prisma v7 introduced a significant type-checking performance regression on large schemas (406 models, ~9,300 lines): type instantiations grew from ~8M (v6) to ~32.8M (v7), causing `tsc --noEmit` to run for over 2 minutes and `tsgo` to time out at 3 minutes. The issue was traced to a single line change in the `OmitOpts` generic default. Prisma 6 is recommended for large TypeScript codebases until the regression is addressed.

**Beginner-Friendly Explanation:** Prisma's type safety is powerful, but on very large projects it can slow down your editor and build tools. The fix in Prisma v7 accidentally made TypeScript work much harder than it needed to. If your project has hundreds of models, you may want to stay on Prisma 6 until the performance is fixed.

#### Purposes

- To understand the trade-offs of Prisma's code-generation approach.
- To mitigate TypeScript compilation performance issues on large schemas.
- To use raw SQL and query extensions for performance-critical operations.

#### Syntax Rules and Structure

**Performance mitigation strategies:**
| Strategy | Description |
|----------|-------------|
| Stay on Prisma 6 | Avoid the v7 type-checking regression. |
| Use `prisma.$queryRaw` | Bypass Prisma Client for performance-critical queries. |
| Use Prisma Client Extensions | Add custom query methods with type safety. |
| Limit `include` depth | Deeply nested includes generate complex types. |
| Split schema into multiple files | Improves organisation but not type-checking. |

**Raw SQL:**
```typescript
const users = await prisma.$queryRaw`
  SELECT id, name FROM users WHERE email = ${email}
`;
```

**Prisma Client Extension:**
```typescript
const extendedClient = prisma.$extends({
  query: {
    user: {
      async findMany({ args, query }) {
        args.where = { ...args.where, deletedAt: null };
        return query(args);
      },
    },
  },
});
```

**Constraints and Limitations:**
- `$queryRaw` returns `unknown[]`; manual type casting is required.
- Prisma Client Extensions replace the deprecated `$use` middleware.
- Very large schemas may require splitting into multiple Prisma schemas (not natively supported).

#### Annotated Code Example

```typescript
// prisma-performance.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function performanceComparison() {
  // ❌ N+1 pattern
  console.time('N+1');
  const users = await prisma.user.findMany();
  for (const user of users) {
    const posts = await prisma.post.findMany({ where: { authorId: user.id } });
    user.posts = posts;
  }
  console.timeEnd('N+1');

  // ✅ Single query with include
  console.time('Include');
  const usersWithPosts = await prisma.user.findMany({
    include: { posts: true },
  });
  console.timeEnd('Include');

  // ✅ Raw SQL for complex aggregation
  const result = await prisma.$queryRaw`
    SELECT u.id, u.name, COUNT(p.id) as post_count
    FROM users u
    LEFT JOIN posts p ON p.author_id = u.id
    GROUP BY u.id, u.name
    ORDER BY post_count DESC
    LIMIT 10
  `;
  console.log('Top users:', result);
}

performanceComparison();
```

**Expected Output:**
```
N+1: 5200ms
Include: 45ms
Top users: [ { id: 1, name: 'Alice', post_count: 5n }, ... ]
```

**Why this output:** The N+1 pattern executes 1 query for users plus N queries for posts (101 queries for 100 users). The `include` option loads everything in a single query with a JOIN. Raw SQL handles complex aggregations that are difficult to express in the Prisma Client API. The `post_count` is returned as a BigInt (`5n`) because PostgreSQL's `COUNT` returns `bigint`.

#### Real-World Cases

- **Large enterprise schemas:** Staying on Prisma 6 until v7 performance is resolved.
- **Performance-critical queries:** Using `$queryRaw` for complex aggregations.
- **Soft deletes:** Using Prisma Client Extensions to automatically filter deleted records.

---

## Core Concept 2: Sequelize

### Sub-Feature 2.1: Traditional Promise-Based MVC Model Patterns and Legacy Systems

#### Definitions

**Core Definition:** Sequelize is a promise-based Node.js ORM that follows the traditional MVC (Model-View-Controller) pattern, with models defined via `sequelize.define()` and associations established through methods like `hasMany` and `belongsTo`.

**Technical Definition:** Sequelize is a promise-based Node.js ORM for PostgreSQL, MySQL, MariaDB, SQLite, and Microsoft SQL Server. It features solid transaction support, relations, eager and lazy loading, read replication, and more. Models are defined either via `sequelize.define('ModelName', attributes)` or by extending `Model` in TypeScript. Associations are defined using `Model.belongsTo()`, `Model.hasMany()`, `Model.hasOne()`, and `Model.belongsToMany()`. Sequelize automatically adds `createdAt` and `updatedAt` timestamp columns unless `timestamps: false` is set.

**Beginner-Friendly Explanation:** Sequelize is the old reliable of Node.js ORMs. It's been around since 2014, has a huge community, and works with almost every database. If you're maintaining a legacy Node.js application, there's a good chance it uses Sequelize. It follows the classic MVC pattern: models define your data, controllers handle requests, and views render responses.

#### Purposes

- To provide a mature, battle-tested ORM for legacy and traditional applications.
- To define models and associations in a declarative, promise-based API.
- To support transactions, read replication, and migrations.
- To integrate with Express and other MVC frameworks.

#### Syntax Rules and Structure

```javascript
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize(process.env.DATABASE_URL);

// Define model
const User = sequelize.define('User', {
  name: { type: DataTypes.STRING, allowNull: false },
  email: { type: DataTypes.STRING, unique: true, allowNull: false },
}, {
  timestamps: true,        // Auto-adds createdAt, updatedAt
  createdAt: 'created_at',  // Custom column names
  updatedAt: 'updated_at',
  hooks: {
    beforeCreate: (user) => {
      user.name = user.name.trim();
    },
  },
});

// Define associations
User.hasMany(Post, { foreignKey: 'user_id', onDelete: 'CASCADE' });
Post.belongsTo(User, { foreignKey: 'user_id' });

// Usage
const user = await User.create({ name: 'Alice', email: 'alice@example.com' });
const users = await User.findAll({ include: [{ model: Post }] });
```

| Method | Description |
|--------|-------------|
| `sequelize.define()` | Define a model. |
| `Model.create()` | Insert a row. |
| `Model.findAll()` | Query rows. |
| `Model.findByPk()` | Query by primary key. |
| `Model.update()` | Update rows. |
| `Model.destroy()` | Delete rows. |
| `Model.hasMany()` | One-to-many association. |
| `Model.belongsTo()` | Many-to-one association. |
| `Model.belongsToMany()` | Many-to-many association. |

**Constraints and Limitations:**
- Sequelize's TypeScript support is less mature than Prisma or Drizzle.
- Models defined with `sequelize.define()` lack TypeScript inference; `Model.init()` or class extension is required for TypeScript.
- Seeders are not tracked; running a seeder twice can result in duplicate rows or constraint violations.
- The default `save()` method always executes a SELECT before UPDATE; use `Model.update()` when the operation type is known.

#### Annotated Code Example

```javascript
// sequelize-example.js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize(process.env.DATABASE_URL);

const User = sequelize.define('User', {
  name: { type: DataTypes.STRING, allowNull: false },
  email: { type: DataTypes.STRING, unique: true, allowNull: false },
});

const Post = sequelize.define('Post', {
  title: { type: DataTypes.STRING, allowNull: false },
});

User.hasMany(Post, { foreignKey: 'userId', onDelete: 'CASCADE' });
Post.belongsTo(User, { foreignKey: 'userId' });

async function main() {
  await sequelize.sync({ force: true });

  const user = await User.create({
    name: 'Alice',
    email: 'alice@example.com',
    Posts: [{ title: 'First Post' }, { title: 'Second Post' }],
  }, { include: [Post] });

  const usersWithPosts = await User.findAll({
    include: [{ model: Post }],
  });

  console.log(JSON.stringify(usersWithPosts, null, 2));
  await sequelize.close();
}

main().catch(console.error);
```

**Expected Output:**
```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "createdAt": "2026-01-15T12:00:00.000Z",
    "updatedAt": "2026-01-15T12:00:00.000Z",
    "Posts": [
      { "id": 1, "title": "First Post", "userId": 1 },
      { "id": 2, "title": "Second Post", "userId": 1 }
    ]
  }
]
```

**Why this output:** `User.hasMany(Post)` defines the one-to-many relationship. The `include` option in `findAll` eager-loads the related posts. Sequelize automatically adds `createdAt` and `updatedAt` timestamps. The `onDelete: 'CASCADE'` ensures posts are deleted when their user is deleted.

#### Real-World Cases

- **Legacy applications:** Maintaining existing Express + Sequelize codebases.
- **Rapid prototyping:** Quick model definition with `sequelize.define()`.
- **MVC frameworks:** Integration with traditional Express MVC structure.

---

## Core Concept 3: TypeORM

### Sub-Feature 3.1: Data Mapper vs. Active Record Patterns Using TypeScript Decorators

#### Definitions

**Core Definition:** TypeORM supports both the Active Record pattern (entities manage their own persistence) and the Data Mapper pattern (separate repository classes handle persistence), using TypeScript decorators for entity definition.

**Technical Definition:** In TypeORM, entities are defined using decorators: `@Entity()`, `@Column()`, `@PrimaryGeneratedColumn()`, and relation decorators (`@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@OneToOne`). The Active Record pattern extends `BaseEntity`, allowing entities to call `User.find()`, `user.save()`, and `user.remove()` directly. The Data Mapper pattern uses repositories: `userRepository.find()`, `userRepository.save(user)`. The Data Mapper approach is recommended for larger applications because it separates business logic from persistence. TypeORM supports `synchronize: true` for development (never use in production), migrations for production, and `QueryBuilder` for complex queries.

**Beginner-Friendly Explanation:** TypeORM gives you two ways to work. Active Record is like a self-driving car — the entity knows how to save itself. Data Mapper is like having a chauffeur — a separate repository class handles the driving. For small projects, Active Record is simpler. For large projects, Data Mapper keeps things organised.

#### Purposes

- To define entities with TypeScript decorators for full type safety.
- To choose between Active Record (simplicity) and Data Mapper (maintainability).
- To use `QueryBuilder` for complex queries with joins and subqueries.
- To manage transactions with `QueryRunner` or `EntityManager`.

#### Syntax Rules and Structure

**Active Record:**
```typescript
@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

// Usage
const user = await User.findOne({ where: { id: 1 } });
user.name = 'Alice Smith';
await user.save();
```

**Data Mapper:**
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

// Usage
const userRepository = dataSource.getRepository(User);
const user = await userRepository.findOne({ where: { id: 1 } });
user.name = 'Alice Smith';
await userRepository.save(user);
```

| Pattern | Persistence | Best For |
|---------|-------------|----------|
| Active Record | `entity.save()` | Small applications. |
| Data Mapper | `repository.save(entity)` | Large applications. |

**Relation options:**
| Option | Description |
|--------|-------------|
| `eager: true` | Always load relation with `find*`. |
| `cascade: true` | Insert/update related entities. |
| `onDelete: 'CASCADE'` | Database-level cascade delete. |
| `nullable: false` | INNER JOIN instead of LEFT JOIN. |

**Constraints and Limitations:**
- Never use `synchronize: true` in production — it can drop columns and lose data.
- `save()` always runs an extra SELECT query; use `insert()`/`update()` when the operation type is known.
- Decorator metadata (`reflect-metadata`) adds weight to edge/serverless cold starts.
- Entity listeners should not make database calls.

#### Annotated Code Example

```typescript
// typeorm-example.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne, DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, (post) => post.author, { cascade: true })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts, { onDelete: 'CASCADE' })
  author: User;
}

const dataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [User, Post],
  synchronize: false, // Use migrations in production
});

async function main() {
  await dataSource.initialize();

  const userRepo = dataSource.getRepository(User);

  const user = userRepo.create({
    name: 'Alice',
    posts: [{ title: 'First Post' }, { title: 'Second Post' }],
  });
  await userRepo.save(user); // Cascade saves posts

  const users = await userRepo.find({ relations: { posts: true } });
  console.log(JSON.stringify(users, null, 2));

  await dataSource.destroy();
}

main().catch(console.error);
```

**Expected Output:**
```json
[
  {
    "id": 1,
    "name": "Alice",
    "posts": [
      { "id": 1, "title": "First Post" },
      { "id": 2, "title": "Second Post" }
    ]
  }
]
```

**Why this output:** The `@OneToMany` with `cascade: true` automatically saves the related posts when the user is saved. The `relations: { posts: true }` option in `find` eager-loads the posts. The Data Mapper pattern uses `userRepo.save()` instead of `user.save()`.

#### Real-World Cases

- **NestJS applications:** TypeORM is the default ORM for NestJS.
- **Enterprise applications:** Data Mapper pattern for maintainability.
- **Complex queries:** QueryBuilder for joins, subqueries, and pagination.

---

## Core Concept 4: Drizzle

### Sub-Feature 4.1: Headless, Lightweight Schema Definition with Zero-Overhead Performance

#### Definitions

**Core Definition:** Drizzle is a headless TypeScript ORM that defines schemas in TypeScript using plain functions, generates SQL-like queries with full type inference, and has zero runtime dependencies.

**Technical Definition:** Drizzle is a modern TypeScript ORM that is lightweight (~7.4kb minified+gzipped) and tree-shakeable with exactly 0 dependencies. It supports every PostgreSQL, MySQL, and SQLite database, including serverless ones like Turso, Neon, Xata, PlanetScale, and Cloudflare D1. Drizzle lets you declare SQL schemas in TypeScript and build both relational and SQL-like queries. Type safety is achieved through TypeScript inference, not code generation — there is no `drizzle generate` step for types. Drizzle Kit is the companion CLI for migrations.

**Beginner-Friendly Explanation:** Drizzle is like writing SQL in TypeScript. You define your tables with `pgTable()`, and Drizzle infers every type from your schema. There's no code generation step, no heavy runtime, and no magic. It's fast, lightweight, and works everywhere — including edge runtimes.

#### Purposes

- To provide a lightweight, zero-dependency ORM for modern TypeScript projects.
- To declare schemas in TypeScript with full type inference.
- To write SQL-like queries with compile-time type safety.
- To work in edge/serverless environments (Cloudflare Workers, Deno, Bun).

#### Syntax Rules and Structure

```typescript
// schema.ts
import { pgTable, serial, text, timestamp, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').references(() => users.id, { onDelete: 'cascade' }),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));
```

**Queries:**
```typescript
// SQL-like query
const allUsers = await db.select().from(users).where(eq(users.name, 'Alice'));

// Relational query (eager loading)
const usersWithPosts = await db.query.users.findMany({
  with: { posts: true },
});
```

| Feature | Description |
|---------|-------------|
| Zero dependencies | No runtime dependencies. |
| Type inference | Types inferred from schema, no codegen. |
| SQL-like | `db.select().from(users)` mirrors SQL. |
| Relational API | `db.query.users.findMany({ with: { posts: true } })`. |
| Edge-ready | Works on Node, Bun, Deno, Cloudflare Workers. |

**Constraints and Limitations:**
- No entity classes; Drizzle uses plain table objects and functions.
- `db.query` requires the schema to be passed at initialization.
- Relational queries v2 require all relations to be defined in one place.

#### Annotated Code Example

```typescript
// drizzle-example.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';
import { eq } from 'drizzle-orm';

const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
});

const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').references(() => users.id, { onDelete: 'cascade' }),
});

const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle(pool, { schema: { users, posts, usersRelations, postsRelations } });

async function main() {
  // Insert
  const [user] = await db.insert(users).values({
    name: 'Alice',
    email: 'alice@example.com',
  }).returning();

  await db.insert(posts).values([
    { title: 'First Post', authorId: user.id },
    { title: 'Second Post', authorId: user.id },
  ]);

  // Relational query (eager loading)
  const usersWithPosts = await db.query.users.findMany({
    with: { posts: true },
  });
  console.log(JSON.stringify(usersWithPosts, null, 2));

  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "posts": [
      { "id": 1, "title": "First Post", "authorId": 1 },
      { "id": 2, "title": "Second Post", "authorId": 1 }
    ]
  }
]
```

**Why this output:** Drizzle infers the types of `usersWithPosts` from the schema and relations. The `with: { posts: true }` option eager-loads the posts in a single query. No code generation is required — the types are inferred directly from the TypeScript schema.

#### Real-World Cases

- **Edge/serverless:** Cloudflare Workers, Vercel Edge, Deno Deploy.
- **Lightweight APIs:** Minimal bundle size, fast cold starts.
- **SQL-comfortable teams:** Developers who prefer SQL-like syntax over abstract query builders.

---

### Sub-Feature 4.2: SQL-Like Syntax with Compile-Time Type Safety

#### Definitions

**Core Definition:** Drizzle's SQL-like query builder mirrors SQL syntax (`select`, `from`, `where`, `join`) while providing full TypeScript type inference for every column and result.

**Technical Definition:** Drizzle's SQL-like query builder is the most direct way to write SQL in TypeScript. Queries are built with chainable methods: `db.select().from(users).where(eq(users.id, 1))`. Every method is fully typed; selecting a column that doesn't exist produces a compile-time error. The result type is inferred from the select list. Drizzle also supports `insert`, `update`, `delete`, and `$count`.

**Beginner-Friendly Explanation:** Drizzle's SQL-like syntax is like writing SQL but with TypeScript's autocomplete and type checking. If you type `users.nam` (typo), TypeScript catches it immediately. The result type is exactly what you selected — no guessing.

#### Purposes

- To write SQL-like queries with compile-time type safety.
- To catch column typos and type mismatches at build time.
- To use a familiar SQL syntax without sacrificing type safety.

#### Syntax Rules and Structure

```typescript
// Select
const users = await db.select({ id: users.id, name: users.name }).from(users);

// Join
const result = await db.select()
  .from(users)
  .leftJoin(posts, eq(posts.authorId, users.id))
  .where(eq(users.name, 'Alice'));

// Insert
await db.insert(users).values({ name: 'Alice', email: 'alice@example.com' });

// Update
await db.update(users).set({ name: 'Alice Smith' }).where(eq(users.id, 1));

// Delete
await db.delete(users).where(eq(users.id, 1));

// Aggregation
import { count, sum } from 'drizzle-orm';
const result = await db.select({
  userId: posts.authorId,
  postCount: count(posts.id),
}).from(posts).groupBy(posts.authorId);
```

| Method | SQL Equivalent |
|--------|---------------|
| `db.select()` | `SELECT` |
| `.from(users)` | `FROM users` |
| `.where(eq(...))` | `WHERE ...` |
| `.leftJoin(...)` | `LEFT JOIN ...` |
| `.groupBy(...)` | `GROUP BY ...` |
| `.orderBy(...)` | `ORDER BY ...` |

**Constraints and Limitations:**
- The SQL-like API is more verbose than the relational API for nested reads.
- Dynamic filters require composing conditions with `and()`, `or()`, and `eq()`.
- Raw SQL is still available via `sql` tagged template literals.

#### Annotated Code Example

```typescript
// drizzle-sql-like.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { eq, count, sql } from 'drizzle-orm';

const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').references(() => users.id),
});

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle(pool);

async function main() {
  // Join with type-safe result
  const usersWithPostCount = await db
    .select({
      userId: users.id,
      userName: users.name,
      postCount: count(posts.id),
    })
    .from(users)
    .leftJoin(posts, eq(posts.authorId, users.id))
    .groupBy(users.id, users.name)
    .orderBy(sql`count(${posts.id}) DESC`);

  console.log(usersWithPostCount);
  await pool.end();
}

main().catch(console.error);
```

**Expected Output:**
```
[ { userId: 1, userName: 'Alice', postCount: 2 }, { userId: 2, userName: 'Bob', postCount: 0 } ]
```

**Why this output:** The join combines users and posts. The `count()` aggregation is typed. The result type is inferred as `{ userId: number; userName: string; postCount: number }[]`. TypeScript would error if `users.nam` were used instead of `users.name`.

#### Real-World Cases

- **Analytics dashboards:** Complex aggregations with type safety.
- **Reporting:** SQL-like queries with compile-time validation.
- **Data migrations:** Bulk operations with typed inserts.

---

## Core Concept 5: Entity/Model Concepts

### Sub-Feature 5.1: Database Schema Synchronisation vs. Strict Code-First Mapping

#### Definitions

**Core Definition:** Schema synchronisation auto-generates database tables from entity/model definitions (development only), while strict code-first mapping uses migrations to apply schema changes in a controlled, versioned manner.

**Technical Definition:** TypeORM and Sequelize support `synchronize: true` (TypeORM) or `sequelize.sync()` (Sequelize), which automatically creates or alters tables to match entity definitions. This is convenient for development but dangerous in production — TypeORM's `synchronize: true` can drop columns and lose data when entities change. Prisma and Drizzle use migrations as the production standard: Prisma generates SQL from the PSL schema; Drizzle generates SQL from TypeScript schema definitions. Migrations are versioned, reviewable, and reversible.

**Beginner-Friendly Explanation:** Schema synchronisation is like letting the ORM redecorate your house whenever you change your mind about the furniture — convenient but risky. Migrations are like a renovation plan: you review the changes, approve them, and apply them step by step. For production, always use migrations.

#### Purposes

- To enable rapid development with automatic schema updates.
- To ensure production safety with versioned migrations.
- To track schema changes in version control.
- To enable rollback and audit trails.

#### Syntax Rules and Structure

| ORM | Synchronisation | Migrations |
|-----|----------------|------------|
| TypeORM | `synchronize: true` | `migration:generate`, `migration:run` |
| Sequelize | `sequelize.sync()` | `db:migrate`, `db:migrate:undo` |
| Prisma | `prisma db push` | `prisma migrate dev`, `prisma migrate deploy` |
| Drizzle | `drizzle-kit push` | `drizzle-kit generate`, `drizzle-kit migrate` |

**Constraints and Limitations:**
- `synchronize: true` in production can drop columns and lose data.
- `prisma db push` should only be used for prototyping; use `prisma migrate` for production.
- `drizzle-kit push` does not produce a migration artifact; use `drizzle-kit generate` for production.

#### Annotated Code Example

```typescript
// typeorm-sync-vs-migrate.ts
import { DataSource } from 'typeorm';

// ❌ Development only — never in production
const devDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [User, Post],
  synchronize: true, // Auto-creates tables
});

// ✅ Production — use migrations
const prodDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [User, Post],
  synchronize: false,
  migrations: ['src/migrations/*.ts'],
});
```

**Expected Output:**
```
Development: Tables auto-created from entities.
Production: Migrations applied via `typeorm migration:run`.
```

**Why this output:** `synchronize: true` inspects entity metadata and generates `CREATE TABLE` or `ALTER TABLE` statements. In production, migrations are versioned files that are reviewed and applied deliberately.

#### Real-World Cases

- **Development:** Rapid iteration with `synchronize: true`.
- **Production:** Versioned migrations applied via CI/CD.
- **Compliance:** Audit trails of schema changes for regulatory requirements.

---

### Sub-Feature 5.2: Hooks, Lifecycles, and Automated Timestamp Generation

#### Definitions

**Core Definition:** Hooks (also called lifecycle events) are functions that run before or after specific ORM operations (create, update, delete), while timestamps (`createdAt`, `updatedAt`) are automatically managed columns that record when records were created and modified.

**Technical Definition:** All major ORMs provide hooks: Sequelize has `beforeCreate`, `afterUpdate`, `beforeDestroy`, etc.; TypeORM has `@BeforeInsert`, `@AfterLoad`, `@BeforeUpdate`, etc.; Prisma uses Client Extensions with query hooks; Drizzle uses `$onUpdate()` for automatic timestamp updates. Timestamps are auto-managed: Sequelize adds `createdAt` and `updatedAt` by default; TypeORM has `@CreateDateColumn()` and `@UpdateDateColumn()`; Prisma uses `@default(now())` and `@updatedAt`; Drizzle uses `.defaultNow()` and `.$onUpdate(() => new Date())`.

**Beginner-Friendly Explanation:** Hooks are like automatic actions that happen when you save or delete something. For example, you can automatically set a `slug` field before saving, or update a `lastModified` timestamp. Timestamps are the most common hook — they record when something was created and last changed.

#### Purposes

- To automate cross-cutting concerns (timestamps, slugs, audit fields).
- To execute logic before or after persistence.
- To ensure data consistency without duplicating code.

#### Syntax Rules and Structure

**Sequelize:**
```javascript
const User = sequelize.define('User', {
  name: DataTypes.STRING,
}, {
  hooks: {
    beforeCreate: (user) => {
      user.name = user.name.trim();
    },
    beforeUpdate: (user) => {
      user.updatedAt = new Date();
    },
  },
});
```

**TypeORM:**
```typescript
@Entity()
export class User {
  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeInsert()
  normalizeName() {
    this.name = this.name.trim();
  }
}
```

**Prisma:**
```prisma
model User {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Drizzle:**
```typescript
const users = pgTable('users', {
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().$onUpdate(() => new Date()).notNull(),
});
```

| ORM | Timestamp Columns | Hook Mechanism |
|-----|------------------|----------------|
| Sequelize | `createdAt`, `updatedAt` (auto) | `hooks` option or `Model.addHook()`. |
| TypeORM | `@CreateDateColumn`, `@UpdateDateColumn` | `@BeforeInsert`, `@AfterLoad`, etc. |
| Prisma | `@default(now())`, `@updatedAt` | Client Extensions (query hooks). |
| Drizzle | `.defaultNow()`, `.$onUpdate()` | `$onUpdate()` lifecycle hook. |

**Constraints and Limitations:**
- Prisma removed `$use` middleware; use Client Extensions instead.
- TypeORM entity listeners should not make database calls.
- Drizzle `$onUpdate` runs on every update; ensure it is applied consistently.

#### Annotated Code Example

```typescript
// drizzle-timestamps.ts
import { pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at')
    .defaultNow()
    .$onUpdate(() => new Date())
    .notNull(),
});
```

**Expected Output:**
```sql
-- Generated migration
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() NOT NULL
);
```

**Why this output:** `defaultNow()` sets the timestamp on insert. `$onUpdate(() => new Date())` automatically updates the `updatedAt` column on every UPDATE. Drizzle generates the SQL migration from the TypeScript schema.

#### Real-World Cases

- **Audit trails:** Tracking when records were created and modified.
- **Soft deletes:** Setting `deletedAt` instead of deleting rows.
- **Slug generation:** Automatically generating URL-friendly slugs from titles.

---

## Core Concept 6: Relations

### Sub-Feature 6.1: Eager Loading vs. Lazy Loading in Asynchronous Environments

#### Definitions

**Core Definition:** Eager loading fetches related data in the same query as the parent record, while lazy loading fetches related data only when it is accessed, requiring an additional query.

**Technical Definition:** In TypeORM, `eager: true` on a relation loads it automatically with `find*` methods. Lazy relations return a Promise that must be awaited (`user.posts` returns `Promise<Post[]>`). In Prisma, `include` eager-loads relations. In Drizzle, `db.query.users.findMany({ with: { posts: true } })` eager-loads. In Sequelize, `include` eager-loads; lazy loading is not directly supported (associations must be included explicitly). Lazy loading in asynchronous environments can cause the N+1 problem and is generally discouraged.

**Beginner-Friendly Explanation:** Eager loading is like ordering a burger with fries — you get everything in one order. Lazy loading is like ordering a burger and then, after it arrives, calling the restaurant again to ask for fries. Eager loading is almost always better for performance.

#### Purposes

- To fetch related data efficiently in a single query.
- To avoid the N+1 query problem.
- To control when related data is loaded (lazy) when it's not always needed.

#### Syntax Rules and Structure

| ORM | Eager Loading | Lazy Loading |
|-----|--------------|--------------|
| Prisma | `include: { posts: true }` | Not supported. |
| TypeORM | `relations: { posts: true }` or `eager: true` | `user.posts` (returns Promise). |
| Sequelize | `include: [{ model: Post }]` | Not directly supported. |
| Drizzle | `with: { posts: true }` | Not supported. |

**Constraints and Limitations:**
- TypeORM's `eager: true` only works with `find*` methods, not `QueryBuilder`.
- Lazy loading in TypeORM uses Promises, which can cause N+1 if not batched.
- Prisma, Sequelize, and Drizzle do not support true lazy loading.

#### Annotated Code Example

```typescript
// prisma-eager-loading.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  // ❌ N+1 pattern
  const users = await prisma.user.findMany();
  for (const user of users) {
    user.posts = await prisma.post.findMany({ where: { authorId: user.id } });
  }

  // ✅ Eager loading
  const usersWithPosts = await prisma.user.findMany({
    include: { posts: true },
  });

  console.log('Users with posts:', usersWithPosts);
  await prisma.$disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
Users with posts: [ { id: 1, name: 'Alice', posts: [ [Object], [Object] ] } ]
```

**Why this output:** The `include` option generates a single SQL query with a JOIN, loading all users and their posts in one round trip. Without `include`, each user would require a separate query for posts (N+1).

#### Real-World Cases

- **GraphQL APIs:** Using `include` to resolve nested fields efficiently.
- **REST APIs:** Eager-loading relations for list endpoints.
- **Dashboard:** Loading aggregated data with relations in a single query.

---

### Sub-Feature 6.2: Cascade Deletes, Updates, and Referential Integrity Constraints

#### Definitions

**Core Definition:** Cascade deletes automatically delete related records when a parent record is deleted; cascade updates propagate changes to related records; referential integrity constraints ensure foreign keys always point to valid records.

**Technical Definition:** In TypeORM, `onDelete: 'CASCADE'` specifies database-level cascade deletion, while `cascade: true` enables application-level cascade for insert/update/remove. In Prisma, `onDelete: Cascade` is defined in the `@relation` attribute. In Sequelize, `onDelete: 'CASCADE'` is set in the association options. In Drizzle, `.references(() => users.id, { onDelete: 'cascade' })` sets the foreign key action. TypeORM's `cascade` option can be a boolean or an array of cascade options: `("insert" | "update" | "remove" | "soft-remove" | "recover")[]`.

**Beginner-Friendly Explanation:** Cascade deletes are like deleting a folder on your computer — all the files inside are deleted too. Referential integrity ensures you can't have an order that points to a user who doesn't exist.

#### Purposes

- To maintain referential integrity automatically.
- To avoid orphaned records.
- To simplify deletion of complex object graphs.
- To enforce database-level constraints.

#### Syntax Rules and Structure

**TypeORM:**
```typescript
@ManyToOne(() => User, (user) => user.posts, { onDelete: 'CASCADE' })
author: User;
```

**Prisma:**
```prisma
model Post {
  author   User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId Int
}
```

**Sequelize:**
```javascript
User.hasMany(Post, { foreignKey: 'userId', onDelete: 'CASCADE' });
```

**Drizzle:**
```typescript
authorId: integer('author_id').references(() => users.id, { onDelete: 'cascade' }),
```

| Option | Description | Level |
|--------|-------------|-------|
| `onDelete: 'CASCADE'` | Database-level cascade. | All ORMs. |
| `cascade: true` | Application-level cascade. | TypeORM, Sequelize. |
| `onDelete: 'RESTRICT'` | Prevent deletion if children exist. | TypeORM, Prisma. |
| `onDelete: 'SET NULL'` | Set foreign key to null. | TypeORM, Prisma. |

**Constraints and Limitations:**
- Database-level cascade (`onDelete: 'CASCADE'`) is more reliable than application-level cascade.
- TypeORM only traverses relations that are populated on the object — if a relation is not loaded, its children will not be cascade-removed.
- Prisma's `onDelete: Cascade` is defined in the `@relation` attribute.

#### Annotated Code Example

```typescript
// prisma-cascade.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  // Create user with posts
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      posts: { create: [{ title: 'Post 1' }, { title: 'Post 2' }] },
    },
    include: { posts: true },
  });
  console.log('Created user with posts:', user.posts.length);

  // Delete user — posts are cascade-deleted
  await prisma.user.delete({ where: { id: user.id } });

  const remainingPosts = await prisma.post.findMany();
  console.log('Remaining posts:', remainingPosts.length); // 0

  await prisma.$disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
Created user with posts: 2
Remaining posts: 0
```

**Why this output:** The `onDelete: Cascade` in the Prisma schema causes the database to automatically delete the posts when the user is deleted. The `remainingPosts` count confirms the cascade delete.

#### Real-World Cases

- **User deletion:** Cascade-deleting posts, comments, and likes.
- **Order cancellation:** Cascade-deleting order items.
- **Data cleanup:** Using `SET NULL` to preserve historical records.

---

## Core Concept 7: Migrations

### Sub-Feature 7.1: Declarative vs. Imperative Migration Paradigms

#### Definitions

**Core Definition:** Declarative migrations describe the desired end state of the schema, and the tool generates the migration SQL; imperative migrations are hand-written SQL scripts that explicitly define each change.

**Technical Definition:** Prisma and Drizzle use a declarative approach: you modify the schema, and the CLI generates a migration SQL file based on the diff. Sequelize and TypeORM can auto-generate migrations from entity changes (`sequelize-cli db:migrate:generate`, `typeorm migration:generate`), but also support hand-written imperative migrations. Prisma migrations can be edited before applying — for example, to rename a column instead of dropping and re-creating it.

**Beginner-Friendly Explanation:** Declarative migrations are like telling a contractor "I want a wall here" and letting them figure out the steps. Imperative migrations are like writing the step-by-step instructions yourself. Declarative is easier; imperative gives more control.

#### Purposes

- To version-control database schema changes.
- To apply consistent changes across development, staging, and production.
- To enable rollback and audit trails.
- To support zero-downtime deployments.

#### Syntax Rules and Structure

| ORM | Declarative | Imperative |
|-----|------------|------------|
| Prisma | Schema changes → `prisma migrate dev` | Edit generated SQL. |
| Drizzle | Schema changes → `drizzle-kit generate` | Hand-write SQL. |
| Sequelize | `db:migrate:generate` (from models) | Hand-write `up`/`down`. |
| TypeORM | `migration:generate` (from entities) | Hand-write `up`/`down`. |

**Constraints and Limitations:**
- Auto-generated migrations may not preserve data (e.g., renaming a column drops and re-creates it).
- Hand-written migrations require careful testing.
- Rollback scripts must be maintained for each migration.

#### Annotated Code Example

```bash
# Prisma — declarative migration
npx prisma migrate dev --name add_bio_field
# Generates: migrations/20260115120000_add_bio_field/migration.sql

# Edit the generated SQL to preserve data:
# ALTER TABLE users ADD COLUMN bio TEXT;
# UPDATE users SET bio = biography;  -- Copy data
# ALTER TABLE users DROP COLUMN biography;
```

```javascript
// Sequelize — imperative migration
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.addColumn('users', 'bio', {
      type: Sequelize.TEXT,
      allowNull: true,
    });
    await queryInterface.sequelize.query('UPDATE users SET bio = biography');
    await queryInterface.removeColumn('users', 'biography');
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.addColumn('users', 'biography', {
      type: Sequelize.TEXT,
    });
    await queryInterface.sequelize.query('UPDATE users SET biography = bio');
    await queryInterface.removeColumn('users', 'bio');
  },
};
```

**Expected Output:**
```
Migration applied: 20260115120000_add_bio_field
```

**Why this output:** The declarative migration is generated by Prisma from the schema change. The imperative migration is hand-written for Sequelize, with explicit `up` and `down` functions. Both achieve the same result: adding a `bio` column and migrating data from `biography`.

#### Real-World Cases

- **Schema evolution:** Adding columns, creating indexes, and altering constraints.
- **Data migrations:** Copying data between columns during schema changes.
- **Rollback:** Reverting the last migration if deployment fails.

---

### Sub-Feature 7.2: Zero-Downtime Deployment Strategies (Expand/Contract Pattern)

#### Definitions

**Core Definition:** The Expand/Contract pattern enables zero-downtime schema changes by splitting each change into two phases: **Expand** (additive, backward-compatible changes) and **Contract** (removing old columns after the application has migrated).

**Technical Definition:** The expand and contract pattern splits database changes into two phases: **Pre-deploy (Expand)** — additive, non-breaking changes that are backwards compatible with the current API version; **Post-deploy (Contract)** — removing old columns or constraints after the application has migrated. For renaming a column, the pattern involves: (1) add the new column, (2) write to both old and new columns, (3) migrate data, (4) read from the new column, (5) stop writing to the old column, (6) drop the old column. This requires three production deployments to fully implement.

**Beginner-Friendly Explanation:** Imagine changing a road sign while traffic is flowing. Instead of ripping out the old sign and putting up a new one (which causes a traffic jam), you put up the new sign next to the old one, wait for everyone to start using the new sign, and then remove the old one. That's the expand/contract pattern.

#### Purposes

- To deploy schema changes without downtime.
- To support rolling deployments where old and new code coexist.
- To avoid data loss during column renames.
- To enable safe rollback at each phase.

#### Syntax Rules and Structure

**Expand phase (pre-deploy):**
```sql
-- Add new column (nullable)
ALTER TABLE users ADD COLUMN biography TEXT;
```

**Application code (dual write):**
```typescript
await prisma.user.update({
  where: { id },
  data: { bio: req.body.bio, biography: req.body.bio },
});
```

**Contract phase (post-deploy):**
```sql
-- Copy data
UPDATE users SET biography = bio WHERE biography IS NULL;
-- Drop old column
ALTER TABLE users DROP COLUMN bio;
```

**Prisma expand/contract:**
```bash
# 1. Expand: add new field
npx prisma migrate dev --name add_biography

# 2. Update application code to write to both fields

# 3. Create empty migration and copy data
npx prisma migrate dev --create-only --name migrate_bio_data
# Edit SQL: UPDATE users SET biography = bio;

# 4. Update application code to read from new field

# 5. Contract: drop old field
npx prisma migrate dev --name drop_bio
```

**Constraints and Limitations:**
- Requires three production deployments for a column rename.
- The transitional phase must support both old and new code.
- Rollback is possible at any phase.

#### Annotated Code Example

```typescript
// prisma-expand-contract.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

// Phase 1: Expand — write to both columns
async function createUserExpand(name: string, bio: string) {
  return prisma.user.create({
    data: { name, bio, biography: bio }, // Dual write
  });
}

// Phase 2: Migrate data
async function migrateData() {
  await prisma.$executeRaw`
    UPDATE users SET biography = bio WHERE biography IS NULL
  `;
}

// Phase 3: Contract — read/write only new column
async function createUserContract(name: string, biography: string) {
  return prisma.user.create({
    data: { name, biography },
  });
}
```

**Expected Output:**
```
Phase 1: Users created with both bio and biography.
Phase 2: Data migrated from bio to biography.
Phase 3: Users created with biography only; bio column dropped.
```

**Why this output:** The expand phase adds the new column and writes to both. The data migration copies existing data. The contract phase removes the old column. At no point does the application stop working.

#### Real-World Cases

- **Renaming columns:** The most common expand/contract use case.
- **Changing column types:** Add new column, migrate data, drop old column.
- **Splitting tables:** Create new tables, dual-write, migrate, drop old tables.

---

### Sub-Feature 7.3: Seed Scripts and Version Control Management for Schemas

#### Definitions

**Core Definition:** Seed scripts populate the database with initial or test data, while version control management tracks schema changes as migration files committed to Git.

**Technical Definition:** Seed scripts are idempotent scripts that insert reference data (roles, feature flags, admin users) or test data. Prisma uses `prisma db seed` with a `seed.ts` file; Sequelize uses seeders in `seeders/` directory; TypeORM uses a custom seed script or the `typeorm-extension` package. Migrations are stored in version control (Git) alongside application code, ensuring every environment can be brought to the same schema state. The `migrations/` directory contains timestamped migration files; the `seeders/` directory contains seed files.

**Beginner-Friendly Explanation:** Seed scripts are like the initial furniture you put in a new house — they make it usable. Version control for schemas means every time you change the database structure, you save a record of that change, so you can track who changed what and when.

#### Purposes

- To populate the database with required reference data.
- To provide consistent test data for development and testing.
- To track schema changes in version control.
- To enable CI/CD pipelines to apply migrations automatically.

#### Syntax Rules and Structure

**Prisma seed:**
```json
// package.json
{
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  await prisma.role.upsert({
    where: { name: 'admin' },
    update: {},
    create: { name: 'admin', permissions: ['read', 'write', 'delete'] },
  });
}

main().catch(console.error).finally(() => prisma.$disconnect());
```

**Sequelize seeders:**
```bash
npx sequelize-cli seed:generate --name demo-user
```

```javascript
// seeders/20260115120000-demo-user.js
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.bulkInsert('Users', [{
      name: 'Alice',
      email: 'alice@example.com',
      createdAt: new Date(),
      updatedAt: new Date(),
    }]);
  },
  async down(queryInterface) {
    await queryInterface.bulkDelete('Users', { email: 'alice@example.com' });
  },
};
```

| ORM | Seed Command | Seed Location |
|-----|-------------|---------------|
| Prisma | `prisma db seed` | `prisma/seed.ts` |
| Sequelize | `sequelize-cli db:seed:all` | `seeders/` |
| TypeORM | Custom script | `src/seeds/` |
| Drizzle | Custom script | `src/seed.ts` |

**Constraints and Limitations:**
- Sequelize does not track which seeders have run; running a seeder twice can result in duplicate rows or constraint violations.
- Prisma seed scripts must be idempotent (use `upsert`).
- Seed data should be separate from migration data.

#### Annotated Code Example

```typescript
// prisma-seed.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  // Idempotent seed using upsert
  const adminRole = await prisma.role.upsert({
    where: { name: 'admin' },
    update: {},
    create: {
      name: 'admin',
      permissions: ['read', 'write', 'delete', 'manage_users'],
    },
  });
  console.log('Admin role seeded:', adminRole);

  // Feature flags
  await prisma.featureFlag.upsert({
    where: { key: 'new_dashboard' },
    update: {},
    create: { key: 'new_dashboard', enabled: false },
  });

  console.log('Seeding complete.');
}

main()
  .catch((e) => { console.error(e); process.exit(1); })
  .finally(() => prisma.$disconnect());
```

**Expected Output:**
```
Admin role seeded: { id: 1, name: 'admin', permissions: [ 'read', 'write', 'delete', 'manage_users' ] }
Seeding complete.
```

**Why this output:** `upsert` ensures idempotency — running the seed script multiple times does not create duplicate roles. The `where` clause checks for an existing role by name; if found, it does nothing; if not found, it creates it.

#### Real-World Cases

- **Reference data:** Roles, permissions, feature flags, and configuration.
- **Test data:** Sample users, products, and orders for development.
- **CI/CD:** Running seeds automatically after migrations in test environments.

---

## References

- Prisma Documentation — https://www.prisma.io/docs
- Prisma Schema Language — https://www.prisma.io/docs/orm/prisma-schema/overview
- Prisma Migrate — https://www.prisma.io/docs/orm/prisma-migrate
- Prisma Expand/Contract — https://www.prisma.io/docs/orm/prisma-migrate/workflows/customizing-migrations
- Sequelize Documentation — https://sequelize.org/docs/v6/
- Sequelize Migrations — https://sequelize.org/docs/v6/other-topics/migrations/
- Sequelize Hooks — https://sequelize.org/docs/v6/other-topics/hooks/
- TypeORM Documentation — https://typeorm.io/docs/
- TypeORM Relations — https://typeorm.io/docs/relations/relations/
- TypeORM Active Record vs Data Mapper — https://typeorm.io/docs/guides/active-record-data-mapper/
- TypeORM Entity Listeners — https://typeorm.io/docs/listeners-and-subscribers/
- Drizzle ORM Documentation — https://orm.drizzle.team/docs/overview
- Drizzle Kit Migrations — https://orm.drizzle.team/docs/migrations
- Drizzle Relations — https://orm.drizzle.team/docs/rqb
- Drizzle `$onUpdate` — https://orm.drizzle.team/docs/indexes-constraints#on-update
- Node.js ORM Comparison 2026 — https://www.pkgpulse.com/guides/nodejs-orm-comparison-2026
- Prisma v7 TypeScript Performance Regression — https://github.com/prisma/prisma/issues/29011
- The Expand and Contract Pattern — https://martinfowler.com/articles/expand-contract.html
- Zero-Downtime Schema Migrations — https://www.prisma.io/docs/orm/prisma-migrate/workflows/customizing-migrations