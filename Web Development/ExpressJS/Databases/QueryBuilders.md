# Express.js Query Builders — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Query builders are programmatic libraries that construct SQL queries through a fluent, chainable JavaScript/TypeScript API, providing a middle ground between raw SQL strings and full object-relational mappers (ORMs).

**Technical Definition**
A query builder exposes methods such as `.select()`, `.where()`, `.join()`, and `.insert()` that compose into an abstract syntax tree, which is then compiled into dialect-specific SQL. In the Express.js ecosystem, **Knex.js** is the veteran JavaScript-first query builder with built-in migrations and connection pooling, while **Kysely** is a TypeScript-first query builder that derives compile-time type safety from a manually defined `Database` interface. Both support PostgreSQL, MySQL, SQLite, and SQL Server, and both allow raw SQL escape hatches when the builder's abstraction is insufficient.

**Beginner-Friendly Explanation**
A query builder lets you write SQL-like code in JavaScript instead of typing SQL strings. You chain methods together—`.select('name').where('age', '>', 18)`—and the library turns that into the correct SQL for your database. It protects you from SQL injection by parameterizing values automatically, and it gives you autocomplete and (with Kysely) compile-time type checking. It sits between raw SQL (maximum control, no safety) and a full ORM (less control, more magic).

---

### Key Characteristics

- **SQL-first abstraction** — Both Knex and Kysely model SQL directly; you write what looks like SQL, and the library compiles it 1:1 without hidden queries.
- **Dialect portability** — The same query builder code compiles to PostgreSQL, MySQL, SQLite, and SQL Server syntax.
- **Automatic parameterization** — Values passed to builder methods are bound as parameters, preventing SQL injection.
- **Type safety spectrum** — Knex offers optional TypeScript generics; Kysely provides native, field-level type inference from a `Database` interface.
- **Ecosystem depth** — Knex includes built-in migrations, seeds, and schema builder; Kysely is a pure query builder that leaves migrations and schema management to separate tools.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A database driver installed (`pg`, `mysql2`, `better-sqlite3`, or `mssql`).
- The query builder package installed (`knex` or `kysely`).
- For Kysely: TypeScript 5.0+ with strict mode, and a `Database` interface describing the schema.
- For Knex: optional TypeScript generics for row typing.

---

### Related Programming Areas

- **Database drivers** — Query builders sit on top of drivers such as `pg` and `mysql2`.
- **Migrations** — Knex provides a built-in migration CLI; Kysely relies on external tools like `kysely-codegen` and migration frameworks.
- **Transactions** — Both builders provide callback-style and controlled transaction APIs.
- **Raw SQL** — Both offer escape hatches (`knex.raw()`, Kysely's `sql` template tag) for expressions the builder does not cover.
- **Connection pooling** — Knex manages its own pool via `tarn.js`; Kysely uses the underlying driver's pool.

---

## Core Concept 1: Knex.js

### Definitions

**Core Definition**
Knex.js is a “batteries-included” SQL query builder for JavaScript that provides a fluent, chainable API for building and executing standard SQL queries, with built-in connection pooling, schema building, migrations, and seeding.

**Technical Definition**
Knex exposes a `knex(tableName)` entry point that returns a `QueryBuilder` instance. The builder supports `select`, `insert`, `update`, `delete`, joins, subqueries, raw expressions, and schema operations. It abstracts dialect-specific SQL syntax differences, compiling the same builder chain to PostgreSQL, MySQL, SQLite, MSSQL, Oracle, and Redshift. Knex manages a connection pool internally via `tarn.js`, and provides a `transaction()` method for ACID-compliant operations. TypeScript support is available via optional generic type parameters.

**Beginner-Friendly Explanation**
Knex is the long-standing query builder for Node.js. You start with `knex('users')` and chain methods like `.select()`, `.where()`, and `.join()`. It handles the SQL syntax differences between databases and gives you a connection pool out of the box. It also includes tools for creating tables and running migrations.

---

### Purposes

- To construct SQL queries programmatically with a fluent API that is portable across multiple database dialects.
- To prevent SQL injection through automatic parameter binding.
- To provide built-in connection pooling, migrations, and schema building without additional dependencies.
- To offer a mature, battle-tested alternative to raw SQL strings and heavy ORMs.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Query Chain

```javascript
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 0, max: 20 }
});

// SELECT
const users = await knex('users')
  .select('id', 'name', 'email')
  .where('age', '>', 18)
  .orderBy('name', 'asc');

// INSERT with returning
const [newUser] = await knex('users')
  .insert({ name: 'Alice', email: 'alice@example.com' })
  .returning(['id', 'name', 'email']);

// UPDATE
await knex('users').where({ id: 1 }).update({ name: 'Alice Smith' });

// DELETE
await knex('users').where({ id: 1 }).del();
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `knex('users')` | Starts a query against the `users` table |
| `.select(...)` | Specifies columns to return |
| `.where(...)` | Adds a `WHERE` clause with parameter binding |
| `.returning(...)` | Returns inserted/updated rows (PostgreSQL/MSSQL) |
| `.insert()`, `.update()`, `.del()` | Mutations |

#### Complete General Syntax: Transaction

```javascript
await knex.transaction(async (trx) => {
  await trx('accounts').where({ id: 1 }).decrement('balance', 100);
  await trx('accounts').where({ id: 2 }).increment('balance', 100);
  // If any statement throws, Knex auto-rolls back
});
```

#### Complete General Syntax: Raw Query with Bindings

```javascript
const result = await knex.raw('SELECT * FROM users WHERE email = ?', [email]);
// Use ?? for identifiers, ? for values
```

#### Syntax Rules

- **Use parameterized bindings** — `?` for values and `??` for identifiers in `knex.raw()`; never interpolate user input.
- **Return or await the transaction promise** — failing to do so causes the transaction connection to hang indefinitely.
- **Initialize Knex once** per application; creating multiple instances leaks connection pools.
- **Call `knex.destroy()`** on graceful shutdown to drain the pool.
- **Use `.returning()`** on PostgreSQL/MSSQL to retrieve inserted or updated rows; it is a no-op on MySQL/SQLite.

#### Constraints and Limitations

- Knex's TypeScript support is **optional and bolted on**; type inference is weaker than Kysely's native design.
- The query builder API can be less ergonomic than Kysely's for complex, deeply nested queries.
- Built-in migrations are convenient but less flexible than dedicated migration frameworks.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Knex with Express and PostgreSQL

```javascript
const express = require('express');
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 0, max: 20 }
});

const app = express();

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await knex('users')
      .select('id', 'name', 'email')
      .where('id', req.params.id)
      .first();
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.json(user);
  } catch (err) {
    next(err);
  }
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Initialization** — Knex is created once with a PostgreSQL client and a pool.
2. **Query construction** — `knex('users').select(...).where(...)` builds a parameterized `SELECT`.
3. **Execution** — `.first()` executes and returns the first row or `undefined`.

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "name": "Alice", "email": "alice@example.com" }
GET /users/99  → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- The `where('id', req.params.id)` binds the ID as a parameter, preventing injection; `.first()` returns a single row.

---

#### Example 2: Knex Transaction for a Transfer

```javascript
app.post('/transfer', async (req, res, next) => {
  try {
    await knex.transaction(async (trx) => {
      await trx('accounts').where({ id: req.body.fromId }).decrement('balance', req.body.amount);
      await trx('accounts').where({ id: req.body.toId }).increment('balance', req.body.amount);
    });
    res.json({ success: true });
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
POST /transfer  (valid) → 200 { "success": true }
POST /transfer  (error) → 500 (both balances unchanged due to automatic rollback)
```

**Why This Output Occurs**

- Knex automatically commits on promise resolution and rolls back on rejection; returning or awaiting the transaction promise is essential.

---

### Real-World Cases

- **Existing JavaScript Express codebases** — Knex is a low-friction upgrade from raw SQL strings.
- **Multi-dialect applications** — Applications that must support PostgreSQL, MySQL, and SQLite with the same query code.
- **Teams that need built-in migrations** — Knex's `migrate:make` / `migrate:latest` toolchain is mature and widely documented.
- **Oracle and Redshift support** — Knex covers databases that Kysely does not natively support.

---

## Core Concept 2: Kysely

### Definitions

**Core Definition**
Kysely (pronounced “Key-Seh-Lee”) is a type-safe, autocompletion-friendly TypeScript SQL query builder that models SQL directly in TypeScript, deriving full compile-time type safety from a manually defined database schema interface.

**Technical Definition**
Kysely requires a `Database` interface that describes every table, column, and nullability. Queries are written with a fluent API (`db.selectFrom('person').select('first_name').where('person.age', '>', 18)`), and the TypeScript compiler validates table names, column references, and result types. When a column is renamed or a type changes, the compiler surfaces every query that needs updating. Kysely is a **pure query builder**—it has no schema management, no migrations, and no relation abstractions. Transactions use callback-style `.transaction().execute(async (trx) => ...)` or controlled `db.startTransaction()`.

**Beginner-Friendly Explanation**
Kysely is the TypeScript-native query builder. You define your database schema as a TypeScript interface, and then every query you write is checked by the compiler. If you misspell a column or use the wrong type, your editor shows a red squiggle before you even run the code. It does not manage your schema or migrations—it just builds queries with maximum type safety.

---

### Purposes

- To catch SQL errors—misspelled columns, wrong table names, type mismatches—at compile time rather than runtime.
- To provide full autocompletion and type inference from a single `Database` interface.
- To offer a thin, predictable 1:1 abstraction over SQL for developers who want maximum control without ORM magic.
- To support complex queries (joins, CTEs, window functions, subqueries) with typed result shapes.

---

### Syntax Rules and Structure

#### Complete General Syntax: Schema Interface and Setup

```typescript
import { Kysely, PostgresDialect, Generated, ColumnType } from 'kysely';
import { Pool } from 'pg';

interface UsersTable {
  id: Generated<string>;
  email: string;
  name: string | null;
  role: 'admin' | 'user';
  created_at: Generated<Date>;
}

interface Database {
  users: UsersTable;
}

const db = new Kysely<Database>({
  dialect: new PostgresDialect({
    pool: new Pool({ connectionString: process.env.DATABASE_URL })
  })
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Database` interface | Single source of truth for all query types |
| `Generated<T>` | Database-generated values (IDs, timestamps) |
| `ColumnType<Select, Insert, Update>` | Different types per operation |
| `db.selectFrom()` | Entry point for `SELECT` queries |

#### Complete General Syntax: Typed Query

```typescript
const adults = await db
  .selectFrom('users')
  .select(['id', 'name', 'email'])
  .where('role', '=', 'admin')
  .where('created_at', '>', new Date('2025-01-01'))
  .execute();
// Type: { id: string; name: string | null; email: string }[]
```

#### Complete General Syntax: Transaction

```typescript
await db.transaction().execute(async (trx) => {
  await trx.insertInto('users').values({ email, name }).execute();
  await trx.updateTable('audit').set({ updated: true }).execute();
  // If any statement throws, Kysely auto-rolls back
});
```

#### Syntax Rules

- **Define the `Database` interface** before writing queries; it is the foundation of all type safety.
- **Use `Generated<T>`** for auto-increment IDs, default timestamps, and other database-generated values.
- **Use `ColumnType<Select, Insert, Update>`** when a column has different types for reads and writes.
- **Use the `sql` template tag** for raw expressions; never interpolate unchecked input.
- **Use `executeTakeFirstOrThrow()`** when a row is expected and its absence should be an error.

#### Constraints and Limitations

- **No built-in migrations** — schema management must use `kysely-codegen`, a separate migration tool, or manual SQL.
- **Manual schema interface** — boilerplate upfront; `kysely-codegen` can generate it from an existing database.
- **Steeper learning curve** for developers new to TypeScript or advanced type inference.
- **No relation abstractions** — you write the `JOIN`; there is no `user.posts` magic.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Typed Query with Express

```typescript
import express from 'express';
import { db } from './db';

const app = express();

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await db
      .selectFrom('users')
      .select(['id', 'name', 'email'])
      .where('id', '=', req.params.id)
      .executeTakeFirst();
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
GET /users/1   → 200 { "id": "1", "name": "Alice", "email": "alice@example.com" }
GET /users/99  → 404 { "error": "User not found" }
```

**Why This Output Occurs**

- Kysely compiles the typed builder chain to parameterized SQL; `executeTakeFirst()` returns the first row or `undefined`.

---

#### Example 2: Type-Safe Join with Inferred Result

```typescript
const posts = await db
  .selectFrom('posts')
  .innerJoin('users', 'users.id', 'posts.author_id')
  .select(['posts.title', 'users.name as author_name', 'posts.created_at'])
  .where('posts.published', '=', true)
  .execute();
// Type: { title: string; author_name: string; created_at: Date }[]
```

**Expected Output**

```
[{ "title": "Hello World", "author_name": "Alice", "created_at": "2025-01-15T00:00:00.000Z" }]
```

**Why This Output Occurs**

- Kysely infers the result type from the selected columns and their aliases; misspelling `users.nam` would be a compile-time error.

---

#### Example 3: Kysely Transaction

```typescript
app.post('/transfer', async (req, res, next) => {
  try {
    await db.transaction().execute(async (trx) => {
      await trx.updateTable('accounts').set({ balance: sql`balance - ${req.body.amount}` }).where('id', '=', req.body.fromId).execute();
      await trx.updateTable('accounts').set({ balance: sql`balance + ${req.body.amount}` }).where('id', '=', req.body.toId).execute();
    });
    res.json({ success: true });
  } catch (err) {
    next(err);
  }
});
```

**Expected Output**

```
POST /transfer  (valid) → 200 { "success": true }
POST /transfer  (error) → 500 (both balances unchanged due to automatic rollback)
```

**Why This Output Occurs**

- All operations on `trx` are inside the transaction; if any throws, the entire transaction rolls back.

---

### Real-World Cases

- **New TypeScript Express projects** — Kysely is recommended for teams that want maximum SQL control with full type safety.
- **Complex queries** — Joins, CTEs, window functions, and subqueries with typed result shapes.
- **Serverless and edge deployments** — Kysely is a pure TypeScript library with no native dependencies and fast cold starts.

---

## Core Concept 3: Query Builder Best Practices

### Definitions

**Core Definition**
Query builder best practices are the design and operational patterns that keep query construction safe, performant, and maintainable across Knex and Kysely applications.

**Technical Definition**
Best practices encompass: **always using parameterized bindings** to prevent SQL injection; **initializing the query builder once** and reusing the connection pool; **wrapping multi-step writes in transactions**; **using migrations** for version-controlled schema changes; **avoiding N+1 queries** through joins or batching; **using `.returning()`** where supported; **calling `destroy()`** on shutdown; and **isolating query code** in a repository or data-access layer rather than embedding it in route handlers.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your query builder from becoming a security hole or a performance bottleneck. Always let the builder parameterize your values. Create the connection once and reuse it. Use transactions for anything that must succeed or fail together. Use migrations so your schema changes are tracked. And keep your queries in a separate layer so your routes stay clean.

---

### Purposes

- To prevent SQL injection through consistent parameterization.
- To avoid connection leaks and pool exhaustion through singleton initialization and proper shutdown.
- To ensure data integrity through transaction handling.
- To keep schema changes reproducible through migrations.
- To maintain readable, testable query code through a repository layer.

---

### Syntax Rules and Structure

#### Complete General Syntax: Singleton Initialization and Shutdown

```javascript
// Knex
const knex = require('knex')(config);
process.on('SIGTERM', () => knex.destroy());

// Kysely
const db = new Kysely({ dialect });
process.on('SIGTERM', () => db.destroy());
```

#### Complete General Syntax: Parameterized Raw Queries

```javascript
// Knex: ? for values, ?? for identifiers
await knex.raw('SELECT * FROM users WHERE email = ?', [email]);

// Kysely: sql template tag with bound parameters
await sql`SELECT * FROM users WHERE email = ${email}`.execute(db);
```

#### Complete General Syntax: Repository Pattern

```javascript
// repositories/userRepository.js
class UserRepository {
  constructor(db) { this.db = db; }
  findById(id) {
    return this.db('users').where({ id }).first();
  }
  create(data) {
    return this.db('users').insert(data).returning('*');
  }
}
```

#### Syntax Rules

- **Always parameterize** — never concatenate user input into queries, even with `raw()`.
- **Initialize once** — Knex and Kysely should each have a single instance per application.
- **Always return or await transactions** — an un-awaited transaction promise hangs the connection.
- **Use migrations** — Knex has built-in migrations; Kysely applications should use `kysely-codegen` and a migration tool.
- **Use `.returning()`** on PostgreSQL/MSSQL for inserted/updated rows.
- **Call `destroy()` on graceful shutdown** to drain the pool.

#### Constraints and Limitations

- Knex's TypeScript types are optional and weaker than Kysely's native inference.
- Kysely requires an upfront `Database` interface; keeping it in sync with the database is manual unless `kysely-codegen` is used.
- Neither builder eliminates the need to understand the SQL it generates; use `.toSQL()` (Knex) or `.compile()` (Kysely) to inspect queries.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Knex Repository with Parameterized Queries and Transactions

```javascript
const express = require('express');
const knex = require('knex')({ client: 'pg', connection: process.env.DATABASE_URL });
const app = express();

// Repository layer
const userRepo = {
  findById: (id) => knex('users').select('id', 'name', 'email').where({ id }).first(),
  create: (data) => knex('users').insert(data).returning(['id', 'name', 'email']),
};

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userRepo.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (err) { next(err); }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1   → 200 { "id": 1, "name": "Alice", "email": "alice@example.com" }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- The repository isolates query construction; `.where({ id })` binds the ID as a parameter.

---

#### Example 2: Kysely Repository with Type-Safe Queries

```typescript
import { db } from './db';

const userRepo = {
  findById: (id: string) =>
    db.selectFrom('users').select(['id', 'name', 'email']).where('id', '=', id).executeTakeFirst(),
  create: (data: { email: string; name: string }) =>
    db.insertInto('users').values(data).returningAll().executeTakeFirstOrThrow(),
};

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userRepo.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (err) { next(err); }
});
```

**Expected Output**

```
GET /users/1   → 200 { "id": "1", "name": "Alice", "email": "alice@example.com" }
GET /users/99  → 404 { "error": "Not found" }
```

**Why This Output Occurs**

- The repository uses Kysely's typed builder; `executeTakeFirstOrThrow()` enforces that a row is returned for inserts.

---

### Real-World Cases

- **Production APIs** — Repository layers with singleton query builders and parameterized queries.
- **Financial systems** — Transactions with automatic rollback and explicit commit.
- **Multi-tenant SaaS** — Tenant-scoped queries with parameterized tenant IDs.
- **Migration-driven teams** — Knex migrations for schema evolution; Kysely with `kysely-codegen` for type generation.

---

## Knex vs Kysely Comparison

| Feature | Knex.js | Kysely |
|---------|---------|--------|
| **Language** | JavaScript-first (+ optional TypeScript generics) | TypeScript-first |
| **Type safety** | Moderate — types are add-ons | Excellent — compile-time column and table validation |
| **Schema definition** | Migrations (`knex migrate:make`) | Manual `Database` interface or `kysely-codegen` |
| **Migrations** | Built-in CLI | No built-in; external tools |
| **Connection pooling** | Built-in via `tarn.js` | Uses driver's pool (`pg.Pool`, etc.) |
| **Dialect support** | PostgreSQL, MySQL, SQLite, MSSQL, Oracle, Redshift, CockroachDB | PostgreSQL, MySQL, SQLite, MSSQL (native dialects) |
| **Performance** | ~458,940 ops/sec (benchmark) | ~584,682 ops/sec (benchmark) |
| **Learning curve** | Lower for JavaScript developers | Steeper for TypeScript newcomers |
| **Best for** | Existing JS codebases, multi-dialect support, built-in migrations | New TypeScript projects, maximum type safety, complex queries with typed results |

**Decision guidance:** For new TypeScript projects, Kysely is the recommended choice for field-level type safety and compile-time error detection. For existing JavaScript codebases, teams without strict TypeScript requirements, or projects needing Oracle/Redshift support, Knex remains a mature and capable choice. Kysely was explicitly inspired by Knex and can even reuse an existing Knex setup via the `kysely-knex` toolkit.

---

## References

- Knex Query Builder — https://knexjs.org/guide/query-builder.html
- Knex.js Patterns (singleton, pooling, raw bindings, transactions) — https://lobehub.com/skills/neversight-learn-skills.dev-api-database-knex
- Kysely Official Documentation — https://kysely.dev/
- Kysely: Type-Safe SQL Query Builder for Node.js (2026) — https://dev.to/stacknotice/kysely-type-safe-sql-query-builder-for-nodejs-2026-2kmj
- Kysely Transactions (controlled and callback) — https://kysely.dev/docs/transactions
- Knex vs Kysely Comparison (Chinese) — https://www.guance.com/learn/articles/kysely-vs-knexjs
- Drizzle ORM vs Kysely vs Knex.js (2026) — https://aidev.fit/en/compare/drizzle-vs-kysely-vs-knex.html
- js-query-builder-benchmark (Knex vs Kysely performance) — https://github.com/endel/js-query-builder-benchmark
- Kysely vs. Knex.js: The JavaScript Query Builder Showdown (Better Stack) — https://betterstack.com/community/comparisons/kysely-vs-knex/
- kysely-knex (reuse existing Knex setup with Kysely) — https://github.com/kysely-org/kysely-knex