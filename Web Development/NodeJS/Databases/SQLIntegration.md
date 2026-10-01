# SQL Integration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL integration in Node.js is the practice of connecting JavaScript/TypeScript applications to relational databases, executing CRUD operations, managing transactions, optimising joins, and leveraging query builders or raw SQL for performance and type safety.

**Technical Definition:** SQL integration encompasses the full lifecycle of database interaction from Node.js: driver-level connectivity (pg, mysql2, mssql), query execution (parameterised queries, prepared statements), transaction management (ACID guarantees via BEGIN/COMMIT/ROLLBACK), result mapping (rows to JavaScript objects or TypeScript interfaces), streaming for large datasets, join optimisation (avoiding the N+1 problem), and programmatic SQL generation using query builders like Knex.js and Kysely. Raw SQL remains essential for performance-critical queries, complex aggregations, and database-specific features.

**Beginner-Friendly Explanation:** SQL integration is like being a translator between your Node.js application and your database. Your app speaks JavaScript; the database speaks SQL. You need to translate requests back and forth, make sure that groups of changes happen together (transactions), fetch related data efficiently (joins), and sometimes write the SQL directly when you need maximum control. This cheat sheet covers all the patterns you'll need to build production-grade database integration.

### Key Characteristics

- **Row mapping:** Database rows are plain objects; mapping them to typed interfaces prevents runtime errors.
- **Streaming:** Large result sets can be streamed row-by-row to avoid loading everything into memory.
- **ACID transactions:** BEGIN, COMMIT, and ROLLBACK wrap multiple statements into an atomic unit.
- **Isolation levels:** Read Committed (default), Repeatable Read, and Serializable control concurrency trade-offs.
- **N+1 problem:** Naive loops that fetch related data one query at a time; solved with joins or batch loading.
- **Query builders:** Knex.js (flexible, mature) and Kysely (type-safe, TypeScript-first) generate SQL programmatically.
- **Raw SQL:** Tagged template literals provide safe, parameterised raw queries when builders are insufficient.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **SQL fundamentals:** SELECT, INSERT, UPDATE, DELETE, JOIN, and GROUP BY.
- **Database drivers:** Familiarity with `pg`, `mysql2`, or similar.
- **TypeScript (optional):** For type-safe query builders and row mapping.
- **Asynchronous programming:** Promises, async/await, and connection pooling.

### Related Programming Areas

- **Relational Databases:** PostgreSQL, MySQL, MariaDB, SQL Server.
- **Node.js Database Connectivity:** Drivers, pooling, prepared statements.
- **ORMs:** Prisma, TypeORM, Sequelize.
- **Query Builders:** Knex.js, Kysely, Drizzle.
- **Performance:** Indexing, query plans, and streaming.

### Core Concepts

1. **CRUD from Node.js** — row mapping and stream-based fetching.
2. **Transactions** — ACID guarantees, isolation levels, and callback wrappers.
3. **Joins** — 1:N and N:M handling, N+1 detection and mitigation.
4. **Query Builders** — Knex.js and Kysely.
5. **Raw SQL** — performance optimization and safe execution.

---

## Core Concept 1: CRUD from Node.js

### Sub-Feature 1.1: Mapping Database Rows to JavaScript Objects / TypeScript Interfaces

#### Definitions

**Core Definition:** Row mapping is the process of converting database result rows (plain objects with column names as keys) into application-level JavaScript objects or TypeScript interfaces that the rest of the code can use safely.

**Technical Definition:** Database drivers return rows as plain objects (e.g., `{ id: 1, name: 'Alice', created_at: '2026-01-15' }`). Mapping involves renaming snake_case columns to camelCase, converting types (strings to dates, numeric strings to numbers), and shaping nested objects. In TypeScript, interfaces define the expected shape, and mapper functions transform raw rows into typed objects. ORMs and query builders often handle this automatically, but raw SQL requires manual mapping.

**Beginner-Friendly Explanation:** Database rows are like raw ingredients from the store — they need to be prepared before use. A row might have `created_at` as a string; your app wants a `createdAt` Date object. Mapping is the preparation step that turns database rows into the clean objects your application expects.

#### Purposes

- To provide type safety and autocomplete in TypeScript.
- To decouple database schema (snake_case) from application conventions (camelCase).
- To convert database types (strings, numeric strings) into JavaScript types (numbers, Dates).
- To shape flat rows into nested objects (e.g., user with address).

#### Syntax Rules and Structure

**TypeScript interface:**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}
```

**Raw row type:**
```typescript
interface UserRow {
  id: number;
  name: string;
  email: string;
  created_at: Date;
}
```

**Mapper function:**
```typescript
function mapUser(row: UserRow): User {
  return {
    id: row.id,
    name: row.name,
    email: row.email,
    createdAt: row.created_at,
  };
}
```

| Aspect | Raw Row | Mapped Object |
|--------|---------|---------------|
| Column names | `snake_case` | `camelCase` |
| Types | Driver-specific | JavaScript types |
| Nested data | Flat | Structured |
| Null handling | `null` | Optional or default |

**Constraints and Limitations:**
- Mapping adds a small performance overhead; avoid unnecessary transformations.
- Type assertions (`as User`) do not validate at runtime; use validation if needed.
- Nested mappings (e.g., user with orders) require aggregation or separate queries.

#### Annotated Code Example

```typescript
// row-mapping.ts
import { Pool } from 'pg';

interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface UserRow {
  id: number;
  name: string;
  email: string;
  created_at: Date;
}

function mapUser(row: UserRow): User {
  return {
    id: row.id,
    name: row.name,
    email: row.email,
    createdAt: row.created_at,
  };
}

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function getUser(id: number): Promise<User | null> {
  const result = await pool.query<UserRow>(
    'SELECT id, name, email, created_at FROM users WHERE id = $1',
    [id]
  );

  if (result.rows.length === 0) return null;
  return mapUser(result.rows[0]);
}

(async () => {
  const user = await getUser(1);
  console.log(user);
  // → { id: 1, name: 'Alice', email: 'alice@example.com', createdAt: 2026-01-15T12:00:00.000Z }
  await pool.end();
})();
```

**Expected Output:**
```
{ id: 1, name: 'Alice', email: 'alice@example.com', createdAt: 2026-01-15T12:00:00.000Z }
```

**Why this output:** The `pg` driver returns the row with `created_at` (snake_case). The `mapUser` function renames it to `createdAt` (camelCase) and returns a typed `User` object. TypeScript ensures the mapper covers all fields.

#### Real-World Cases

- **REST APIs:** Mapping database rows to DTOs before sending JSON responses.
- **GraphQL resolvers:** Mapping rows to GraphQL types.
- **Domain models:** Converting persistence models to domain entities.

---

### Sub-Feature 1.2: Stream-Based Data Fetching for Large Datasets

#### Definitions

**Core Definition:** Stream-based data fetching reads database rows one at a time (or in small batches) through a stream, avoiding loading the entire result set into memory.

**Technical Definition:** The `pg` driver provides `pg-query-stream` (a Readable stream over a query) and `Cursor` for batch fetching. `mysql2` provides `connection.query().stream()` and `connection.execute().stream()`. Knex.js exposes `.stream()` on query builders. Kysely provides `.stream()` with the `kysely-stream` plugin. Streaming is essential for exporting millions of rows, ETL pipelines, and generating large reports.

**Beginner-Friendly Explanation:** Imagine reading a 10,000-page book. Loading the whole book into memory at once would be impossible. Streaming is like reading one page at a time — you process each page and move on, never holding more than a few pages in memory.

#### Purposes

- To process large result sets without memory exhaustion.
- To stream data to files, HTTP responses, or other databases.
- To enable real-time processing of query results.
- To reduce garbage collection pressure from large arrays.

#### Syntax Rules and Structure

**pg-query-stream:**
```javascript
const QueryStream = require('pg-query-stream');
const { Pool } = require('pg');

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const client = await pool.connect();

const query = new QueryStream('SELECT * FROM large_table');
const stream = client.query(query);

stream.on('data', (row) => { /* process row */ });
stream.on('end', () => client.release());
```

**mysql2 streaming:**
```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({ /* config */ });

const query = connection.query('SELECT * FROM large_table');
query.on('result', (row) => { /* process row */ });
query.on('end', () => connection.end());
```

**Knex.js streaming:**
```javascript
const knex = require('knex')({ /* config */ });

const stream = knex('large_table').select('*').stream();
stream.on('data', (row) => { /* process row */ });
stream.on('end', () => knex.destroy());
```

| Driver | Streaming API | Install |
|--------|--------------|---------|
| `pg` | `pg-query-stream` | `npm install pg-query-stream` |
| `mysql2` | `query.stream()` | Built-in |
| `knex` | `.stream()` | Built-in |
| `kysely` | `.stream()` | `npm install kysely-stream` |

**Constraints and Limitations:**
- Streaming holds a connection until the stream ends; pool exhaustion can occur if streams are not consumed promptly.
- Backpressure must be managed; a slow consumer can cause memory buildup on the database side.
- Transactions with streaming require careful handling.

#### Annotated Code Example

```javascript
// streaming-query.js
const { Pool } = require('pg');
const QueryStream = require('pg-query-stream');
const fs = require('fs');

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function exportUsersToCSV(outputPath) {
  const client = await pool.connect();
  const writeStream = fs.createWriteStream(outputPath);

  writeStream.write('id,name,email\n');

  try {
    const query = new QueryStream('SELECT id, name, email FROM users ORDER BY id');
    const stream = client.query(query);

    for await (const row of stream) {
      writeStream.write(`${row.id},${row.name},${row.email}\n`);
    }

    writeStream.end();
    console.log(`Exported to ${outputPath}`);
  } finally {
    client.release();
  }
}

exportUsersToCSV('./users.csv').then(() => pool.end());
```

**Expected Output:**
```
Exported to ./users.csv
```

**Why this output:** The `pg-query-stream` wraps the query in a Readable stream. The `for await...of` loop processes each row individually. Memory usage remains constant regardless of the number of rows. The file is written incrementally.

#### Real-World Cases

- **Data exports:** Exporting millions of rows to CSV or JSON.
- **ETL pipelines:** Reading from one database and writing to another.
- **Report generation:** Streaming large reports to HTTP responses.
- **Analytics:** Processing aggregate data without loading it all into memory.

---

## Core Concept 2: Transactions

### Sub-Feature 2.1: ACID Compliance Guarantees in Node.js Asynchronous Code

#### Definitions

**Core Definition:** ACID (Atomicity, Consistency, Isolation, Durability) is a set of properties that guarantee database transactions are processed reliably, even in the presence of errors, crashes, or concurrent access.

**Technical Definition:** Atomicity ensures that all statements in a transaction are executed or none are. Consistency ensures that the database moves from one valid state to another. Isolation ensures that concurrent transactions do not interfere with each other. Durability ensures that committed transactions survive system failures. In Node.js, ACID is guaranteed by the database engine (InnoDB for MySQL, PostgreSQL's MVCC), not by the driver. The driver provides the `BEGIN`, `COMMIT`, and `ROLLBACK` commands.

**Beginner-Friendly Explanation:** ACID is like a bank transfer. If you transfer $100 from Account A to Account B, either both the withdrawal and deposit happen (atomicity), or neither does. The bank doesn't let another transaction see a half-completed transfer (isolation). And once the transfer is confirmed, it stays confirmed even if the bank loses power (durability).

#### Purposes

- To ensure data integrity across multiple related statements.
- To prevent partial updates from leaving the database in an inconsistent state.
- To handle concurrent access safely.
- To provide reliable recovery from failures.

#### Syntax Rules and Structure

```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [100, 1]);
  await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [100, 2]);
  await client.query('COMMIT');
} catch (err) {
  await client.query('ROLLBACK');
  throw err;
} finally {
  client.release();
}
```

| Property | Guarantee | Implementation |
|----------|-----------|----------------|
| Atomicity | All or nothing. | `BEGIN` / `COMMIT` / `ROLLBACK`. |
| Consistency | Valid state to valid state. | Constraints, triggers. |
| Isolation | Concurrent transactions don't interfere. | Isolation levels (MVCC). |
| Durability | Committed data survives crashes. | Write-ahead logging (WAL). |

**Constraints and Limitations:**
- Transactions must use the same connection; `pool.query()` may use different connections.
- Long transactions hold locks and can cause contention.
- Transactions do not automatically retry on deadlock.

#### Annotated Code Example

```javascript
// acid-transfer.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function transfer(fromId, toId, amount) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');

    // Check balance
    const { rows } = await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE',
      [fromId]
    );

    if (rows[0].balance < amount) {
      throw new Error('Insufficient funds');
    }

    // Debit
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromId]
    );

    // Credit
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toId]
    );

    await client.query('COMMIT');
    console.log(`Transferred $${amount} from ${fromId} to ${toId}`);
  } catch (err) {
    await client.query('ROLLBACK');
    console.error('Transfer failed:', err.message);
    throw err;
  } finally {
    client.release();
  }
}

transfer(1, 2, 100)
  .then(() => pool.end())
  .catch(() => pool.end());
```

**Expected Output (successful transfer):**
```
Transferred $100 from 1 to 2
```

**Expected Output (insufficient funds):**
```
Transfer failed: Insufficient funds
```

**Why this output:** The `BEGIN` starts the transaction. `FOR UPDATE` locks the account row. If the balance is insufficient, an error is thrown, `ROLLBACK` undoes any changes, and the error propagates. If successful, `COMMIT` makes the changes durable.

#### Real-World Cases

- **Banking:** Transfers, deposits, and withdrawals.
- **E-commerce:** Order placement (decrement inventory, create order, charge payment).
- **Inventory:** Stock adjustments across multiple warehouses.
- **User registration:** Create user, create profile, send welcome email.

---

### Sub-Feature 2.2: Transaction Isolation Levels and Deadlocks

#### Definitions

**Core Definition:** Transaction isolation levels control how concurrent transactions see each other's changes, trading off consistency against concurrency. Deadlocks occur when two transactions wait for each other's locks, requiring one to be rolled back.

**Technical Definition:** SQL standard isolation levels are Read Uncommitted (dirty reads allowed), Read Committed (no dirty reads; default in PostgreSQL and Oracle), Repeatable Read (no non-repeatable reads; default in MySQL InnoDB), and Serializable (no phantom reads; full isolation). Deadlocks occur when two transactions hold locks that the other needs. The database detects deadlocks and rolls back one transaction with an error (PostgreSQL: `40P01`; MySQL: `1213`). Applications must catch deadlock errors and retry.

**Beginner-Friendly Explanation:** Isolation levels are like different rules for a shared kitchen. "Read Uncommitted" lets you taste food that's still cooking (might be bad). "Read Committed" lets you taste only finished dishes. "Serializable" means only one chef works at a time — safest but slowest. A deadlock is when two chefs each hold an ingredient the other needs — someone has to give up and start over.

#### Purposes

- To balance consistency and concurrency based on application needs.
- To detect and handle deadlocks gracefully.
- To implement retry logic for transient failures.
- To avoid race conditions in concurrent code.

#### Syntax Rules and Structure

**Setting isolation level:**
```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- or
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**Node.js:**
```javascript
await client.query('BEGIN ISOLATION LEVEL SERIALIZABLE');
// ... statements ...
await client.query('COMMIT');
```

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|-----------|---------------------|--------------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed | No | Yes | Yes |
| Repeatable Read | No | No | Yes* |
| Serializable | No | No | No |

*PostgreSQL's Repeatable Read prevents phantom reads via MVCC.

**Constraints and Limitations:**
- Serializable transactions may fail with serialization errors (`40001`); retry required.
- Higher isolation reduces concurrency.
- Deadlocks cannot be prevented entirely; handle them with retries.

#### Annotated Code Example

```javascript
// isolation-deadlock.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function transferWithRetry(fromId, toId, amount, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const client = await pool.connect();
    try {
      await client.query('BEGIN ISOLATION LEVEL REPEATABLE READ');

      // Lock rows in consistent order to avoid deadlocks
      const [first, second] = [fromId, toId].sort((a, b) => a - b);

      await client.query('SELECT * FROM accounts WHERE id = $1 FOR UPDATE', [first]);
      await client.query('SELECT * FROM accounts WHERE id = $1 FOR UPDATE', [second]);

      await client.query(
        'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
        [amount, fromId]
      );
      await client.query(
        'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
        [amount, toId]
      );

      await client.query('COMMIT');
      console.log(`Transfer succeeded on attempt ${attempt}`);
      return;
    } catch (err) {
      await client.query('ROLLBACK');

      // Retry on deadlock (40P01) or serialization failure (40001)
      if ((err.code === '40P01' || err.code === '40001') && attempt < maxRetries) {
        console.log(`Retrying after ${err.code} (attempt ${attempt})`);
        await new Promise(r => setTimeout(r, 100 * attempt));
        continue;
      }
      throw err;
    } finally {
      client.release();
    }
  }
}

transferWithRetry(1, 2, 100)
  .then(() => pool.end())
  .catch((err) => { console.error(err); pool.end(); });
```

**Expected Output (successful):**
```
Transfer succeeded on attempt 1
```

**Expected Output (deadlock retry):**
```
Retrying after 40P01 (attempt 1)
Transfer succeeded on attempt 2
```

**Why this output:** The transaction locks rows in a consistent order (sorted by ID) to minimise deadlocks. If a deadlock occurs, the `catch` block checks for `40P01` (deadlock) or `40001` (serialization failure) and retries with exponential backoff.

#### Real-World Cases

- **Banking:** Serializable isolation for transfers.
- **Inventory:** Repeatable Read for stock checks.
- **Booking systems:** Serializable for seat reservation.
- **High-concurrency APIs:** Retry logic for transient deadlocks.

---

### Sub-Feature 2.3: Callback-Based Transaction Wrappers (BEGIN, COMMIT, ROLLBACK)

#### Definitions

**Core Definition:** Transaction wrappers are utility functions that encapsulate the BEGIN/COMMIT/ROLLBACK pattern, ensuring connections are properly released and errors are handled consistently.

**Technical Definition:** A transaction wrapper acquires a connection from the pool, begins a transaction, runs a callback with the client, commits on success, rolls back on error, and always releases the connection. This pattern avoids duplicating transaction boilerplate and ensures correctness. Libraries like Knex.js provide `knex.transaction(async (trx) => { ... })` and Kysely provides `db.transaction().execute(async (trx) => { ... })`.

**Beginner-Friendly Explanation:** A transaction wrapper is like a template for a recipe. You just fill in the steps (the callback), and the wrapper handles the setup (BEGIN), the cleanup (COMMIT or ROLLBACK), and the dishes (connection release).

#### Purposes

- To eliminate duplicated transaction boilerplate.
- To ensure connections are always released.
- To provide a consistent error-handling path.
- To enable nested transactions (savepoints).

#### Syntax Rules and Structure

```javascript
async function withTransaction(callback) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

// Usage
await withTransaction(async (client) => {
  await client.query('INSERT INTO orders ...');
  await client.query('UPDATE inventory ...');
});
```

**Knex.js:**
```javascript
await knex.transaction(async (trx) => {
  await trx('orders').insert({ ... });
  await trx('inventory').decrement('stock', 1);
});
```

**Kysely:**
```javascript
await db.transaction().execute(async (trx) => {
  await trx.insertInto('orders').values({ ... }).execute();
  await trx.updateTable('inventory').set('stock', 'stock - 1').execute();
});
```

**Constraints and Limitations:**
- Nested transactions require savepoints (`SAVEPOINT`, `ROLLBACK TO SAVEPOINT`).
- Callbacks must not release the client.
- Long-running transactions should be avoided.

#### Annotated Code Example

```javascript
// transaction-wrapper.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function withTransaction(callback) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

async function createOrder(userId, items) {
  return withTransaction(async (client) => {
    // Create order
    const orderResult = await client.query(
      'INSERT INTO orders (user_id, status) VALUES ($1, $2) RETURNING id',
      [userId, 'pending']
    );
    const orderId = orderResult.rows[0].id;

    // Create order items and update inventory
    for (const item of items) {
      await client.query(
        'INSERT INTO order_items (order_id, product_id, quantity) VALUES ($1, $2, $3)',
        [orderId, item.productId, item.quantity]
      );
      await client.query(
        'UPDATE inventory SET stock = stock - $1 WHERE product_id = $2',
        [item.quantity, item.productId]
      );
    }

    return { orderId, itemCount: items.length };
  });
}

createOrder(1, [
  { productId: 101, quantity: 2 },
  { productId: 102, quantity: 1 },
]).then((result) => {
  console.log('Order created:', result);
  return pool.end();
}).catch((err) => {
  console.error('Order failed:', err.message);
  return pool.end();
});
```

**Expected Output:**
```
Order created: { orderId: 1, itemCount: 2 }
```

**Why this output:** The `withTransaction` wrapper handles BEGIN, COMMIT, ROLLBACK, and release. The callback inserts the order and items and updates inventory. If any statement fails, the entire transaction is rolled back, ensuring no partial orders are created.

#### Real-World Cases

- **E-commerce:** Order creation with inventory updates.
- **Financial systems:** Multi-step payment processing.
- **SaaS applications:** Tenant provisioning (create tenant, user, subscription).
- **Data migrations:** Atomic schema and data changes.

---

## Core Concept 3: Joins

### Sub-Feature 3.1: Efficiently Handling 1:N and N:M Data Fetching

#### Definitions

**Core Definition:** Joins combine rows from two or more tables based on a related column, enabling efficient retrieval of 1:N (one-to-many) and N:M (many-to-many) relationships in a single query.

**Technical Definition:** A 1:N relationship (e.g., user → orders) can be fetched with a single `LEFT JOIN` and aggregated into nested objects. An N:M relationship (e.g., posts ↔ tags) requires a junction table and two joins, or aggregation with `JSON_AGG` (PostgreSQL) or `JSON_ARRAYAGG` (MySQL). Aggregation avoids the row multiplication problem where a single parent row is repeated for each child.

**Beginner-Friendly Explanation:** A 1:N relationship is like a parent with multiple children. A single query can fetch the parent and all children, then group them into a nested object. An N:M relationship is like students and courses — each student takes many courses, and each course has many students. You need a junction table (enrollment) to connect them.

#### Purposes

- To fetch related data in a single query instead of multiple round trips.
- To avoid the N+1 query problem.
- To aggregate nested data (e.g., user with orders and order items).
- To reduce network latency and database load.

#### Syntax Rules and Structure

**1:N with aggregation (PostgreSQL):**
```sql
SELECT
  u.id, u.name,
  COALESCE(JSON_AGG(o.*) FILTER (WHERE o.id IS NOT NULL), '[]') AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.id = $1
GROUP BY u.id;
```

**N:M with aggregation (PostgreSQL):**
```sql
SELECT
  p.id, p.title,
  COALESCE(JSON_AGG(t.name) FILTER (WHERE t.id IS NOT NULL), '[]') AS tags
FROM posts p
LEFT JOIN post_tags pt ON pt.post_id = p.id
LEFT JOIN tags t ON t.id = pt.tag_id
WHERE p.id = $1
GROUP BY p.id;
```

**MySQL equivalent:**
```sql
SELECT
  u.id, u.name,
  COALESCE(JSON_ARRAYAGG(JSON_OBJECT('id', o.id, 'total', o.total)), JSON_ARRAY()) AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.id = ?
GROUP BY u.id;
```

| Relationship | Tables | Join Strategy |
|--------------|--------|---------------|
| 1:N | users → orders | `LEFT JOIN` + aggregation |
| N:M | posts ↔ tags | Junction table + two joins |
| 1:1 | users → profiles | `LEFT JOIN` (no aggregation) |

**Constraints and Limitations:**
- Aggregation functions vary by database (JSON_AGG vs. JSON_ARRAYAGG).
- Large aggregations can be memory-intensive; consider pagination.
- Type mapping of aggregated JSON requires parsing.

#### Annotated Code Example

```javascript
// joins-1n-nm.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// 1:N — User with orders
async function getUserWithOrders(userId) {
  const result = await pool.query(`
    SELECT
      u.id, u.name, u.email,
      COALESCE(
        JSON_AGG(
          JSON_BUILD_OBJECT('id', o.id, 'total', o.total, 'status', o.status)
        ) FILTER (WHERE o.id IS NOT NULL),
        '[]'
      ) AS orders
    FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
    WHERE u.id = $1
    GROUP BY u.id
  `, [userId]);

  return result.rows[0];
}

// N:M — Post with tags
async function getPostWithTags(postId) {
  const result = await pool.query(`
    SELECT
      p.id, p.title, p.content,
      COALESCE(
        JSON_AGG(t.name) FILTER (WHERE t.id IS NOT NULL),
        '[]'
      ) AS tags
    FROM posts p
    LEFT JOIN post_tags pt ON pt.post_id = p.id
    LEFT JOIN tags t ON t.id = pt.tag_id
    WHERE p.id = $1
    GROUP BY p.id
  `, [postId]);

  return result.rows[0];
}

(async () => {
  const user = await getUserWithOrders(1);
  console.log('User with orders:', user);

  const post = await getPostWithTags(1);
  console.log('Post with tags:', post);

  await pool.end();
})();
```

**Expected Output:**
```
User with orders: {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com',
  orders: [
    { id: 1, total: 100, status: 'shipped' },
    { id: 2, total: 200, status: 'pending' }
  ]
}
Post with tags: {
  id: 1,
  title: 'Introduction to SQL',
  content: '...',
  tags: ['sql', 'database', 'tutorial']
}
```

**Why this output:** The `LEFT JOIN` with `JSON_AGG` combines the parent row with aggregated child rows in a single query. The `FILTER (WHERE ... IS NOT NULL)` clause excludes null rows when there are no children. The `COALESCE` returns an empty array instead of `[null]`.

#### Real-World Cases

- **E-commerce:** Orders with their line items.
- **Content management:** Posts with tags and comments.
- **Social media:** Users with their posts and followers.
- **Analytics:** Reports with multiple dimensions.

---

### Sub-Feature 3.2: N+1 Query Problem Detection and Mitigation Strategies

#### Definitions

**Core Definition:** The N+1 query problem occurs when an application executes one query to fetch N parent records, then N additional queries to fetch related data for each parent — totalling N+1 queries instead of one.

**Technical Definition:** The N+1 problem is a performance anti-pattern where related data is fetched in a loop. For example, fetching 100 users and then querying each user's orders individually results in 101 queries. Mitigation strategies include: (1) joins with aggregation, (2) batch loading (fetch all related data in one query using `WHERE id IN (...)`), (3) DataLoader (batching and caching), and (4) ORM eager loading (`include`/`with`).

**Beginner-Friendly Explanation:** Imagine ordering 100 pizzas and then calling the restaurant 100 times to ask about each pizza's toppings. That's N+1 queries. Instead, you could call once and ask for all 100 pizzas with their toppings. That's one query.

#### Purposes

- To detect and eliminate N+1 query patterns.
- To reduce database round trips from O(N) to O(1).
- To improve API response times and reduce database load.
- To scale applications to handle more concurrent users.

#### Syntax Rules and Structure

**N+1 (anti-pattern):**
```javascript
const users = await pool.query('SELECT * FROM users');
for (const user of users.rows) {
  // N additional queries
  const orders = await pool.query('SELECT * FROM orders WHERE user_id = $1', [user.id]);
  user.orders = orders.rows;
}
```

**Batch loading (fixed):**
```javascript
const users = await pool.query('SELECT * FROM users');
const userIds = users.rows.map(u => u.id);

// One query for all orders
const orders = await pool.query(
  'SELECT * FROM orders WHERE user_id = ANY($1::int[])',
  [userIds]
);

// Group orders by user_id in memory
const ordersByUser = orders.rows.reduce((acc, order) => {
  (acc[order.user_id] ||= []).push(order);
  return acc;
}, {});

users.rows.forEach(u => { u.orders = ordersByUser[u.id] || []; });
```

| Strategy | Queries | Best For |
|----------|---------|----------|
| N+1 (naive) | N+1 | Never. |
| Join + aggregation | 1 | 1:N and N:M with moderate N. |
| Batch loading | 2 | Large N, flexible filtering. |
| DataLoader | 2 (cached) | GraphQL, repeated access. |

**Constraints and Limitations:**
- Batch loading requires holding all parent IDs in memory.
- Joins with aggregation can produce large result sets.
- DataLoader requires per-request caching.

#### Annotated Code Example

```javascript
// n-plus-one.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// ❌ N+1 problem
async function getUsersWithOrdersN1() {
  const users = await pool.query('SELECT * FROM users LIMIT 100');

  for (const user of users.rows) {
    const orders = await pool.query(
      'SELECT * FROM orders WHERE user_id = $1',
      [user.id]
    );
    user.orders = orders.rows;
  }

  return users.rows;
}

// ✅ Batch loading
async function getUsersWithOrdersBatch() {
  const users = await pool.query('SELECT * FROM users LIMIT 100');

  if (users.rows.length === 0) return [];

  const userIds = users.rows.map(u => u.id);

  const orders = await pool.query(
    'SELECT * FROM orders WHERE user_id = ANY($1::int[])',
    [userIds]
  );

  const ordersByUser = orders.rows.reduce((acc, order) => {
    (acc[order.user_id] ||= []).push(order);
    return acc;
  }, {});

  users.rows.forEach(u => { u.orders = ordersByUser[u.id] || []; });

  return users.rows;
}

(async () => {
  console.time('N+1');
  await getUsersWithOrdersN1();
  console.timeEnd('N+1'); // ~5000ms for 100 users

  console.time('Batch');
  await getUsersWithOrdersBatch();
  console.timeEnd('Batch'); // ~50ms for 100 users

  await pool.end();
})();
```

**Expected Output:**
```
N+1: 5123ms
Batch: 47ms
```

**Why this output:** The N+1 version executes 101 queries (1 for users + 100 for orders). The batch version executes 2 queries (1 for users + 1 for all orders). The batch version is ~100x faster because it eliminates 99 round trips to the database.

#### Real-World Cases- **GraphQL APIs:** Using DataLoader to batch and cache per request.
- **REST APIs:** Batch loading related resources in list endpoints.
- **Reports:** Fetching parent and child data in a single query.
- **Dashboard:** Loading aggregated metrics efficiently.

---

## Core Concept 4: Query Builders

### Sub-Feature 4.1: Programmatic SQL Generation Using Knex.js

#### Definitions

**Core Definition:** Knex.js is a SQL query builder for Node.js that generates SQL programmatically, supporting PostgreSQL, MySQL, MariaDB, SQLite, and SQL Server.

**Technical Definition:** Knex.js provides a fluent API for building queries: `knex('users').select('*').where('id', 1)`. It handles parameterisation, schema migrations, seeds, and transactions. Knex is not an ORM — it does not map rows to objects automatically, but it integrates well with manual mapping. It supports `.stream()` for streaming, `.transaction()` for transactions, and `.toSQL()` for inspecting generated SQL.

**Beginner-Friendly Explanation:** Knex.js is like a translator that speaks SQL for you. Instead of writing SQL strings, you write JavaScript chains like `knex('users').where('id', 1)`, and Knex produces the SQL. It's safer (parameterised) and more maintainable than string concatenation.

#### Purposes

- To generate SQL programmatically without string concatenation.
- To support multiple database dialects with the same code.
- To manage schema migrations and seeds.
- To build dynamic queries with conditional clauses.

#### Syntax Rules and Structure

**Installation:**
```bash
npm install knex pg
```

**Configuration:**
```javascript
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 2, max: 10 },
});
```

**Basic queries:**
```javascript
// SELECT
const users = await knex('users').select('id', 'name').where('status', 'active');

// INSERT
const [id] = await knex('users').insert({ name: 'Alice', email: 'alice@example.com' }).returning('id');

// UPDATE
await knex('users').where('id', 1).update({ name: 'Alice Smith' });

// DELETE
await knex('users').where('id', 1).delete();
```

**Transactions:**
```javascript
await knex.transaction(async (trx) => {
  await trx('orders').insert({ user_id: 1, total: 100 });
  await trx('inventory').where('product_id', 101).decrement('stock', 1);
});
```

**Streaming:**
```javascript
const stream = knex('large_table').select('*').stream();
for await (const row of stream) {
  // process row
}
```

**Constraints and Limitations:**
- Knex is not an ORM; manual row mapping is required.
- Complex queries may be easier in raw SQL.
- Knex's TypeScript support is decent but not as strong as Kysely's.

#### Annotated Code Example

```javascript
// knex-example.js
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 2, max: 10 },
});

async function main() {
  // Create table (migration-style)
  await knex.schema.createTableIfNotExists('users', (table) => {
    table.increments('id').primary();
    table.string('name').notNullable();
    table.string('email').unique().notNullable();
    table.timestamps(true, true);
  });

  // Insert
  const [user] = await knex('users')
    .insert({ name: 'Alice', email: 'alice@example.com' })
    .returning(['id', 'name', 'email']);
  console.log('Inserted:', user);

  // Select with mapping
  const users = await knex('users')
    .select('id', 'name', 'email', 'created_at as createdAt')
    .where('name', 'like', 'A%')
    .orderBy('createdAt', 'desc');
  console.log('Users:', users);

  // Transaction
  await knex.transaction(async (trx) => {
    await trx('users').insert({ name: 'Bob', email: 'bob@example.com' });
    await trx('users').insert({ name: 'Charlie', email: 'charlie@example.com' });
  });
  console.log('Transaction committed');

  // Inspect generated SQL
  const query = knex('users').select('*').where('id', 1);
  console.log('SQL:', query.toSQL().sql);

  await knex.destroy();
}

main().catch(console.error);
```

**Expected Output:**
```
Inserted: { id: 1, name: 'Alice', email: 'alice@example.com' }
Users: [ { id: 1, name: 'Alice', email: 'alice@example.com', createdAt: 2026-01-15T12:00:00.000Z } ]
Transaction committed
SQL: select * from "users" where "id" = $1
```

**Why this output:** Knex generates parameterised SQL from the chainable API. The `returning` clause returns the inserted row. The `as createdAt` alias maps the snake_case column to camelCase. The transaction wraps two inserts in a single atomic unit. `toSQL()` reveals the generated SQL with `$1` placeholders.

#### Real-World Cases

- **Multi-database applications:** Same code for PostgreSQL and MySQL.
- **Dynamic queries:** Building filters, sorting, and pagination conditionally.
- **Migrations:** Versioned schema changes with `knex migrate`.
- **Seeds:** Populating test data with `knex seed`.

---

### Sub-Feature 4.2: Type-Safe Runtime Query Construction (Kysely)

#### Definitions

**Core Definition:** Kysely is a type-safe SQL query builder for TypeScript that generates SQL from a typed schema, providing compile-time type checking for queries and results.

**Technical Definition:** Kysely uses a `Database` interface that maps table names to row types. Queries are built with a fluent API that TypeScript validates at compile time. Kysely supports PostgreSQL, MySQL, SQLite, and MSSQL. It provides `.execute()` (returns all rows), `.executeTakeFirst()` (returns first row or undefined), and `.executeTakeFirstOrThrow()` (throws if no rows). Kysely's type system catches errors like misspelled columns, wrong types, and missing joins at compile time.

**Beginner-Friendly Explanation:** Kysely is like Knex but with TypeScript superpowers. If you try to select a column that doesn't exist, TypeScript catches it before you run the code. It's like having a spell-checker for SQL.

#### Purposes

- To catch SQL errors at compile time.
- To provide autocomplete for table and column names.
- To ensure query results match expected types.
- To enable refactoring without runtime surprises.

#### Syntax Rules and Structure

**Installation:**
```bash
npm install kysely pg
```

**Database interface:**
```typescript
import { Generated, ColumnType } from 'kysely';

interface Database {
  users: {
    id: Generated<number>;
    name: string;
    email: string;
    created_at: ColumnType<Date, string | undefined, never>;
  };
  orders: {
    id: Generated<number>;
    user_id: number;
    total: number;
  };
}
```

**Query examples:**
```typescript
// SELECT
const users = await db.selectFrom('users')
  .select(['id', 'name', 'email'])
  .where('name', 'like', 'A%')
  .execute();

// INSERT
const user = await db.insertInto('users')
  .values({ name: 'Alice', email: 'alice@example.com' })
  .returning(['id', 'name'])
  .executeTakeFirstOrThrow();

// UPDATE
await db.updateTable('users')
  .set({ name: 'Alice Smith' })
  .where('id', '=', 1)
  .execute();

// Transaction
await db.transaction().execute(async (trx) => {
  await trx.insertInto('orders').values({ user_id: 1, total: 100 }).execute();
});
```

**Constraints and Limitations:**
- Requires TypeScript (no plain JavaScript support).
- Database interface must be maintained manually (or generated).
- Some advanced PostgreSQL features require raw SQL.

#### Annotated Code Example

```typescript
// kysely-example.ts
import { Kysely, PostgresDialect, Generated } from 'kysely';
import { Pool } from 'pg';

interface Database {
  users: {
    id: Generated<number>;
    name: string;
    email: string;
    created_at: Generated<Date>;
  };
  orders: {
    id: Generated<number>;
    user_id: number;
    total: number;
    status: string;
  };
}

const db = new Kysely<Database>({
  dialect: new PostgresDialect({
    pool: new Pool({ connectionString: process.env.DATABASE_URL }),
  }),
});

async function main() {
  // Typed insert
  const user = await db.insertInto('users')
    .values({ name: 'Alice', email: 'alice@example.com' })
    .returning(['id', 'name', 'email'])
    .executeTakeFirstOrThrow();
  console.log('Inserted:', user);

  // Typed select with join
  const usersWithOrders = await db
    .selectFrom('users')
    .leftJoin('orders', 'orders.user_id', 'users.id')
    .select([
      'users.id',
      'users.name',
      'orders.id as orderId',
      'orders.total',
    ])
    .where('users.id', '=', user.id)
    .execute();
  console.log('Users with orders:', usersWithOrders);

  // Typed transaction
  await db.transaction().execute(async (trx) => {
    await trx.insertInto('orders')
      .values({ user_id: user.id, total: 100, status: 'pending' })
      .execute();
    await trx.updateTable('users')
      .set({ name: 'Alice Smith' })
      .where('id', '=', user.id)
      .execute();
  });
  console.log('Transaction committed');

  await db.destroy();
}

main().catch(console.error);
```

**Expected Output:**
```
Inserted: { id: 1, name: 'Alice', email: 'alice@example.com' }
Users with orders: [ { id: 1, name: 'Alice', orderId: 1, total: 100 } ]
Transaction committed
```

**Why this output:** Kysely's `Database` interface types every table and column. TypeScript validates that `name` is a string, `email` is a string, and `total` is a number. The join, transaction, and result types are all inferred. If you tried to select `users.nonexistent`, TypeScript would fail the build.

#### Real-World Cases

- **TypeScript-first projects:** Full-stack TypeScript applications.
- **Refactoring:** Renaming columns with compile-time safety.
- **API development:** Ensuring query results match DTOs.
- **Monorepos:** Sharing database types across packages.

---

## Core Concept 5: Raw SQL

### Sub-Feature 5.1: Writing Raw Queries for Performance Optimization and Custom Aggregations

#### Definitions

**Core Definition:** Raw SQL refers to hand-written SQL statements executed directly, used when query builders cannot express a query efficiently or when database-specific features are required.

**Technical Definition:** Raw SQL is necessary for: complex window functions, recursive CTEs, database-specific features (e.g., PostgreSQL's `LATERAL`, `DISTINCT ON`, `GENERATED ALWAYS AS`), bulk operations (`COPY`), and performance-critical queries where the query planner needs hints. Drivers expose raw query methods: `pool.query(sql, params)`, `knex.raw(sql, bindings)`, `sql` tagged templates (Kysely, postgres.js).

**Beginner-Friendly Explanation:** Raw SQL is like speaking directly to the database in its native language, without a translator. It's powerful but requires careful handling to avoid injection.

#### Purposes

- To use database-specific features not exposed by query builders.
- To optimise performance-critical queries.
- To perform bulk operations (COPY, bulk insert).
- To write complex aggregations and window functions.

#### Syntax Rules and Structure

**pg raw query:**
```javascript
const result = await pool.query(`
  SELECT
    user_id,
    COUNT(*) AS order_count,
    SUM(total) AS total_spent,
    RANK() OVER (ORDER BY SUM(total) DESC) AS rank
  FROM orders
  GROUP BY user_id
`, []);
```

**Knex raw:**
```javascript
const result = await knex.raw(`
  SELECT * FROM users WHERE id = ?
`, [1]);
```

**Kysely sql tagged template:**
```typescript
import { sql } from 'kysely';

const result = await db.selectFrom('users')
  .select([
    'id',
    'name',
    sql<number>`(SELECT COUNT(*) FROM orders WHERE user_id = users.id)`.as('orderCount'),
  ])
  .execute();
```

| Use Case | Approach |
|----------|----------|
| Window functions | Raw SQL. |
| Recursive CTEs | Raw SQL. |
| Bulk operations | `COPY` (PostgreSQL), `LOAD DATA` (MySQL). |
| Database-specific features | Raw SQL. |

**Constraints and Limitations:**
- Raw SQL bypasses type checking (unless wrapped in `sql` tagged templates).
- Must use parameterised queries to prevent injection.
- Database-specific SQL reduces portability.

#### Annotated Code Example

```javascript
// raw-sql.js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function topCustomers() {
  const result = await pool.query(`
    WITH customer_totals AS (
      SELECT
        u.id,
        u.name,
        COUNT(o.id) AS order_count,
        SUM(o.total) AS total_spent
      FROM users u
      JOIN orders o ON o.user_id = u.id
      GROUP BY u.id, u.name
    )
    SELECT
      id,
      name,
      order_count,
      total_spent,
      RANK() OVER (ORDER BY total_spent DESC) AS rank
    FROM customer_totals
    ORDER BY rank
    LIMIT 10
  `);

  return result.rows;
}

async function monthlyRevenue(year) {
  const result = await pool.query(`
    SELECT
      DATE_TRUNC('month', created_at) AS month,
      SUM(total) AS revenue,
      COUNT(*) AS order_count
    FROM orders
    WHERE EXTRACT(YEAR FROM created_at) = $1
    GROUP BY DATE_TRUNC('month', created_at)
    ORDER BY month
  `, [year]);

  return result.rows;
}

(async () => {
  const top = await topCustomers();
  console.log('Top customers:', top);

  const revenue = await monthlyRevenue(2026);
  console.log('Monthly revenue:', revenue);

  await pool.end();
})();
```

**Expected Output:**
```
Top customers: [
  { id: 1, name: 'Alice', order_count: '5', total_spent: '1500', rank: '1' },
  { id: 2, name: 'Bob', order_count: '3', total_spent: '800', rank: '2' }
]
Monthly revenue: [
  { month: 2026-01-01T00:00:00.000Z, revenue: '5000', order_count: '20' },
  { month: 2026-02-01T00:00:00.000Z, revenue: '6500', order_count: '25' }
]
```

**Why this output:** The CTE (`customer_totals`) aggregates orders per user. The `RANK()` window function ranks users by total spent. The `DATE_TRUNC` function groups orders by month. The `EXTRACT(YEAR FROM ...)` filter is parameterised (`$1`) to prevent injection.

#### Real-World Cases

- **Analytics:** Ranking, window functions, and CTEs.
- **Reporting:** Monthly, quarterly, and yearly aggregations.
- **Bulk operations:** `COPY` for fast data loading.
- **Database-specific features:** PostgreSQL's `LATERAL`, MySQL's `JSON_TABLE`.

---

### Sub-Feature 5.2: Safe Execution Using Tagged Template Literals

#### Definitions

**Core Definition:** Tagged template literals are a JavaScript feature that allows a function to receive the literal parts and interpolated values of a template string separately, enabling safe parameterised queries.

**Technical Definition:** Tagged template literals are a safer alternative to string concatenation for raw SQL. The `sql` function (in Kysely, postgres.js, and slonik) receives an array of static string parts and an array of interpolated values. It constructs a parameterised query, binding values as parameters and never concatenating them into the SQL. This prevents SQL injection even when using raw SQL. `knex.raw()` and `sql` from Kysely both support this pattern.

**Beginner-Friendly Explanation:** Tagged template literals are like a magic envelope. When you write `` sql`SELECT * FROM users WHERE id = ${userId}` ``, the `sql` function doesn't just glue the string together. Instead, it sends the SQL template and the `userId` value separately to the database, so the value can never be interpreted as SQL.

#### Purposes

- To write raw SQL safely with automatic parameterisation.
- To prevent SQL injection without manual escaping.
- To enable dynamic query construction with user input.
- To combine static SQL with dynamic values.

#### Syntax Rules and Structure

**postgres.js:**
```javascript
const postgres = require('postgres');
const sql = postgres(process.env.DATABASE_URL);

const users = await sql`
  SELECT * FROM users WHERE email = ${email} AND status = ${status}
`;
```

**Kysely:**
```typescript
import { sql } from 'kysely';

const result = await sql<{ id: number; name: string }>`
  SELECT id, name FROM users WHERE email = ${email}
`.execute(db);
```

**Slonik:**
```javascript
const { sql } = require('slonik');

const user = await connection.one(sql`
  SELECT * FROM users WHERE email = ${email}
`);
```

**Knex.raw:**
```javascript
const result = await knex.raw('SELECT * FROM users WHERE email = ?', [email]);
```

| Library | Tagged Template | Parameter Style |
|---------|----------------|-----------------|
| postgres.js | `sql\`...\`` | `${value}` |
| Kysely | `sql\`...\`` | `${value}` |
| Slonik | `sql\`...\`` | `${value}` |
| Knex | `knex.raw()` | `?` placeholders |

**Constraints and Limitations:**
- Tagged templates cannot parameterise identifiers (table names, column names); use `sql.identifier()` or whitelist.
- Some libraries require explicit type annotations for the result.
- Not all drivers support tagged templates natively.

#### Annotated Code Example

```javascript
// tagged-templates.js
const postgres = require('postgres');
const sql = postgres(process.env.DATABASE_URL);

async function searchUsers({ name, email, status }) {
  // Dynamic query with tagged template
  const conditions = [];
  if (name) conditions.push(sql`name ILIKE ${'%' + name + '%'}`);
  if (email) conditions.push(sql`email = ${email}`);
  if (status) conditions.push(sql`status = ${status}`);

  const whereClause = conditions.length > 0
    ? sql`WHERE ${sql.join(conditions, sql` AND `)}`
    : sql``;

  const users = await sql`
    SELECT id, name, email, status
    FROM users
    ${whereClause}
    ORDER BY name
    LIMIT 50
  `;

  return users;
}

// Example usage
(async () => {
  const results = await searchUsers({
    name: 'ali',
    status: 'active',
  });
  console.log('Search results:', results);

  // Demonstrate injection safety
  const maliciousInput = "' OR '1'='1";
  const safeResults = await searchUsers({ name: maliciousInput });
  console.log('Safe results (no injection):', safeResults.length, 'rows');

  await sql.end();
})();
```

**Expected Output:**
```
Search results: [ { id: 1, name: 'Alice', email: 'alice@example.com', status: 'active' } ]
Safe results (no injection): 0 rows
```

**Why this output:** The `sql` tagged template parameterises `${name}`, `${email}`, and `${status}`. Even when the input contains SQL injection syntax (`' OR '1'='1`), it is bound as a literal string value, not interpreted as SQL. The `sql.join()` helper safely combines conditions with `AND`. The malicious input matches no users because the `name ILIKE '%' OR '1'='1%'` pattern is treated as a literal string.

#### Real-World Cases

- **Search APIs:** Dynamic filtering with user-supplied search terms.
- **Admin panels:** Flexible queries with multiple optional filters.
- **Data exports:** Complex queries with parameterised date ranges.
- **Multi-tenant applications:** Queries scoped to a tenant ID.

---

## References

- node-postgres (pg) Documentation — https://node-postgres.com/
- pg-query-stream — GitHub — https://github.com/brianc/node-pg-query-stream
- mysql2 Documentation — https://github.com/sidorares/node-mysql2
- Knex.js Documentation — https://knexjs.org/
- Kysely Documentation — https://kysely.dev/
- Drizzle ORM — https://orm.drizzle.team/
- postgres.js — GitHub — https://github.com/porsager/postgres
- Slonik — GitHub — https://github.com/gajus/slonik
- PostgreSQL Documentation — Transactions — https://www.postgresql.org/docs/current/tutorial-transactions.html
- PostgreSQL Documentation — Transaction Isolation — https://www.postgresql.org/docs/current/transaction-iso.html
- PostgreSQL Documentation — Window Functions — https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation — Common Table Expressions — https://www.postgresql.org/docs/current/queries-with.html
- MySQL Documentation — Transactions — https://dev.mysql.com/doc/refman/8.0/en/commit.html
- MySQL Documentation — InnoDB Locking — https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP — Query Parameterization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- Building a Simple and Effective Error-Handling System in Node.js — https://tsecurity.de/de/2462968/
- Heroku — Best Practices in Error Handling — https://www.heroku.com/codeish-podcasts/51-best-practices-in-error-handling
- pgvector — GitHub — https://github.com/pgvector/pgvector