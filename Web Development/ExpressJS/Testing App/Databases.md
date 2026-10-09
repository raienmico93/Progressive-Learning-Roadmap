# Test Databases — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A test database is a dedicated, isolated database instance used exclusively by an automated test suite to exercise data-layer code without polluting development, staging, or production data.

**Technical Definition:** Test databases are provisioned per test run (or per test file, or per test) using in-memory engines (SQLite `:memory:`, `mongodb-memory-server`), ephemeral containers (Testcontainers, Docker), or dedicated schemas within a shared database server. Each test's data mutations are isolated from other tests through one of several strategies: transaction rollback, table truncation, schema recreation, or unique-identifier scoping. Migrations must be applied to the test database before tests run, and connection pools must be explicitly closed in teardown hooks to prevent Jest from hanging on open handles. 

**Beginner-Friendly Explanation:** When you run automated tests, you don't want them to touch your real data. A test database is a separate, throwaway database that gets set up before tests run, used during tests, and cleaned up (or thrown away entirely) afterward. It can live in memory (fast, temporary) or in a container (realistic, isolated). The goal is simple: every test starts with a clean slate and leaves no trace behind.

### Key Characteristics

- **Complete isolation:** Test data never touches development or production databases. 
- **Per-test determinism:** Each test starts from a known state, eliminating flakiness caused by shared mutable state. 
- **Speed vs. realism trade-off:** In-memory databases are fast but may not perfectly mimic production behaviour; containers are realistic but slower. 
- **Transaction-based cleanup:** Wrapping each test in a transaction and rolling it back is the fastest cleanup method, but fails when the application uses its own transactions or multiple connections. 
- **Migration parity:** Tests should run against the same migrations as production to catch schema issues early. 
- **Connection lifecycle management:** Database clients and pools must be explicitly closed in teardown hooks, or Jest will hang on open handles. 

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **A test runner:** Jest, Vitest, or Node's built-in test runner.
- **Database driver/ORM:** `pg`, `mongodb`, `mysql2`, Prisma, Drizzle, Mongoose, etc.
- **In-memory server (optional):** `mongodb-memory-server`, SQLite, or `pg-mem`.
- **Migration tool:** Knex, Umzug, Prisma Migrate, Drizzle Kit, or `node-pg-migrate`.

### Related Programming Areas

- **Unit Testing:** Testing individual functions in isolation.
- **Integration Testing:** Testing the full request–response lifecycle with a real database.
- **HTTP Testing:** Supertest assertions on API responses that reflect database state.
- **Testcontainers:** Docker-based ephemeral databases for realistic integration tests.
- **CI/CD Pipelines:** Test databases run in CI to validate every commit.

### Core Concepts

1. **Isolated Databases** — dedicated instances per test run or per test.
2. **Test Fixtures** — predefined data sets loaded before tests.
3. **Seed Data** — baseline data populated into the test database.
4. **Database Cleanup** — strategies for resetting state between tests.
5. **Transactions in Tests** — wrapping each test in a rollback transaction.
6. **In-Memory Database Wrappers** — `mongodb-memory-server`, SQLite `:memory:`.
7. **Migration Execution in Test Lifecycles** — applying migrations before tests run.
8. **Connection Pool Management** — preventing open handles from hanging Jest.

---

## Core Concept 1: Isolated Databases

### Definitions

**Core Definition:** Database isolation is the practice of ensuring that each test (or test file, or test suite) operates on its own dedicated database state, so that no test can observe or corrupt another test's data.

**Technical Definition:** Isolation can be achieved at three levels: **per-test** (each test gets a fresh state via transaction rollback or unique identifiers), **per-file** (each Jest worker process gets its own database or schema), and **per-suite** (the entire suite shares a database that is recreated once). The choice depends on the degree of parallelism required and the cost of database provisioning. 

**Beginner-Friendly Explanation:** If two tests run at the same time and both try to create a user named "Alice," they will interfere with each other. Isolation means giving each test its own space — either by running them one at a time, giving each a separate database, or making sure they use unique data that cannot collide.

### Purposes

- To prevent test data from polluting development or production databases.
- To eliminate flaky tests caused by shared mutable state. 
- To enable safe parallel test execution across multiple workers.
- To ensure each test starts from a known, deterministic state.

### Sub-Feature 1.1: Isolation Strategies Compared

| Strategy | Speed | Isolation Level | Best For |
|----------|-------|----------------|----------|
| **Transaction rollback** | Fastest | Per-test | Most tests; fails with app-owned transactions.  |
| **Table truncation** | Slow | Per-test | When transactions cannot be used. |
| **Schema recreation** | Slowest | Per-test | Complete isolation; parallel CI nodes.  |
| **Unique identifiers (UUIDs)** | Fast | Per-test (scoped) | Parallel tests; breaks on global list endpoints.  |
| **Separate database per file** | Moderate | Per-file | Jest parallel workers.  |

**Rules:**
- Prefer transaction rollback for most tests; fall back to truncation or schema recreation when the application manages its own transactions.
- Use unique UUIDs per test when parallel execution is required and global list assertions are not used.
- Never share a test database with development or production — one misconfigured `beforeEach` can wipe the wrong data. 

### Annotated Code Example

```js
// test/isolation-uuid.test.js
const { v4: uuidv4 } = require('uuid');
const request = require('supertest');
const app = require('../app');

describe('Isolation with unique identifiers', () => {
  it('creates and retrieves a user with a unique ID', async () => {
    const testId = uuidv4();
    const testName = `Test User ${uuidv4()}`;

    await request(app)
      .post('/api/users')
      .send({ id: testId, name: testName })
      .expect(201);

    const response = await request(app)
      .get(`/api/users/${testId}`)
      .expect(200);

    expect(response.body.name).toBe(testName);
  });
});
```

**Expected Output:**
```
PASS  test/isolation-uuid.test.js
  Isolation with unique identifiers
    ✓ creates and retrieves a user with a unique ID (45 ms)

Tests: 1 passed, 1 total
```

**Why this output:** Each test generates its own UUID, so even if multiple tests run in parallel, they operate on different records. No cleanup is needed because no other test can see or modify this test's data. This approach breaks down when testing global list endpoints (e.g., `GET /users`), because the response includes records from other parallel tests. 

---

## Core Concept 2: Test Fixtures

### Definitions

**Core Definition:** Test fixtures are predefined data sets — objects, records, or files — that are loaded into the database before tests run to establish a known baseline state.

**Technical Definition:** Fixtures are typically defined as plain JavaScript objects or factory-generated instances that are inserted into the test database in `beforeAll` or `beforeEach` hooks. They provide deterministic data for assertions, ensuring that tests do not depend on data created by other tests or on production data. 

**Beginner-Friendly Explanation:** A fixture is a ready-made set of data that your tests can rely on. Instead of creating a user inside every test, you define a "test user" fixture once and load it before your tests run. This makes tests faster to write and more predictable.

### Purposes

- To establish a known starting state for tests.
- To avoid duplicating data-creation code across tests.
- To provide deterministic assertions independent of test execution order.
- To separate test data concerns from test logic.

### Sub-Feature 2.1: Factory-Based Fixtures

**Factory pattern with Fishery and Faker:**
```js
const { Factory } = require('fishery');
const { faker } = require('@faker-js/faker');

const userFactory = Factory.define(({ sequence }) => ({
  id: faker.string.uuid(),
  email: `user-${sequence}@test.com`,
  name: faker.person.fullName(),
  role: 'user'
}));

const user = userFactory.build({ role: 'admin' });
const users = userFactory.buildList(5);
```

**Rules:**
- Factories produce valid objects by default; override only what the test cares about. 
- Use sequences for unique fields (email, username) to avoid collisions in parallel runs. 
- Define traits for common variations (admin user, expired subscription, pending order). 
- Seed data should be minimal — only create what tests actually need. 

### Sub-Feature 2.2: Loading Fixtures into the Database

```js
// test/fixtures/users.js
module.exports = [
  { id: '1', name: 'Alice', email: 'alice@test.com', role: 'admin' },
  { id: '2', name: 'Bob', email: 'bob@test.com', role: 'user' }
];
```

```js
// test/setup.js
const User = require('../models/User');
const userFixtures = require('./fixtures/users');

beforeEach(async () => {
  await User.deleteMany({});
  await User.insertMany(userFixtures);
});
```

**Expected Output:**
```
Fixtures loaded: 2 users
```

**Why this output:** The `beforeEach` hook clears the users collection and inserts the fixture data. Every test in the suite starts with exactly two users: Alice (admin) and Bob (user). This makes assertions deterministic — `GET /users` will always return exactly two records.

---

## Core Concept 3: Seed Data

### Definitions

**Core Definition:** Seed data is baseline data inserted into a test database to simulate a realistic starting state for integration or end-to-end tests.

**Technical Definition:** Seed scripts populate the database with a known set of records — users, products, orders, configuration — that tests can rely on. Unlike fixtures, which are typically minimal and test-specific, seeds are broader and often mimic production-like data volumes. Seed scripts should be idempotent (running twice does not create duplicates) and versioned alongside application code. 

**Beginner-Friendly Explanation:** Seed data is like stocking a store before opening day. You fill the shelves with products so that when tests run, there is something to find, update, and delete. Without seed data, every test has to create everything from scratch.

### Purposes

- To provide a realistic baseline state for integration tests.
- To avoid duplicating setup logic across test files.
- To ensure tests run against data that resembles production.
- To enable testing of list, search, and pagination endpoints.

### Syntax Rules and Structure

**Seed script pattern:**
```js
// seeds/products.js
module.exports = async (db) => {
  const products = [
    { name: 'Laptop', price: 1200, category: 'electronics' },
    { name: 'Phone', price: 800, category: 'electronics' },
    { name: 'Desk', price: 350, category: 'furniture' }
  ];

  for (const product of products) {
    await db.query(
      'INSERT INTO products (name, price, category) VALUES ($1, $2, $3) ON CONFLICT DO NOTHING',
      [product.name, product.price, product.category]
    );
  }
};
```

**Rules:**
- Seeds should be idempotent — running twice must not create duplicates. 
- Keep seed data minimal — only create what tests actually need. 
- Version seed scripts alongside application code; schema changes must update seeds. 
- Use seed scripts for integration and E2E tests; use fixtures for unit-level isolation.

### Annotated Code Example

```js
// test/integration/seeded.test.js
const request = require('supertest');
const app = require('../../app');
const seedProducts = require('../../seeds/products');

describe('Product API with seed data', () => {
  beforeAll(async () => {
    await seedProducts(app.locals.db);
  });

  it('should list all seeded products', async () => {
    const response = await request(app)
      .get('/api/products')
      .expect(200);

    expect(response.body.length).toBeGreaterThanOrEqual(3);
    expect(response.body.map(p => p.name)).toEqual(
      expect.arrayContaining(['Laptop', 'Phone', 'Desk'])
    );
  });

  it('should filter products by category', async () => {
    const response = await request(app)
      .get('/api/products?category=electronics')
      .expect(200);

    expect(response.body.every(p => p.category === 'electronics')).toBe(true);
  });
});
```

**Expected Output:**
```
PASS  test/integration/seeded.test.js
  Product API with seed data
    ✓ should list all seeded products (52 ms)
    ✓ should filter products by category (38 ms)

Tests: 2 passed, 2 total
```

**Why this output:** The `beforeAll` hook seeds the database with three products. The first test verifies that all three are returned. The second test verifies that category filtering works. Because the seed is idempotent, running the suite multiple times does not create duplicate products.

---

## Core Concept 4: Database Cleanup

### Definitions

**Core Definition:** Database cleanup is the process of resetting the test database to a known state between tests — removing data created by previous tests so that each test starts fresh.

**Technical Definition:** Cleanup strategies include **table truncation** (`TRUNCATE TABLE ... RESTART IDENTITY CASCADE`), **collection deletion** (`deleteMany({})`), **transaction rollback** (fastest; see Core Concept 5), and **schema recreation** (drop and recreate the schema). The choice depends on the database engine, the degree of isolation required, and whether the application manages its own transactions. 

**Beginner-Friendly Explanation:** After a test creates records, those records need to be removed before the next test runs. Otherwise, the next test might see data it did not create, leading to confusing failures. Cleanup is how you "wipe the table" between tests.

### Purposes

- To prevent data from one test leaking into another.
- To ensure deterministic assertions regardless of test order.
- To avoid flaky tests caused by shared mutable state. 
- To keep the test database small and fast.

### Syntax Rules and Structure

**Table truncation (PostgreSQL/MySQL):**
```js
afterEach(async () => {
  await db.query('TRUNCATE TABLE users, products, orders RESTART IDENTITY CASCADE');
});
```

**Collection deletion (MongoDB):**
```js
afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

**Cleanup ordering:**
```js
// Order matters — child tables before parent tables
afterEach(async () => {
  await db.query('TRUNCATE TABLE order_items CASCADE');
  await db.query('TRUNCATE TABLE orders CASCADE');
  await db.query('TRUNCATE TABLE users CASCADE');
});
```

**Rules:**
- **Clean before, not after:** A `beforeEach` cleanup ensures a known starting state even if a previous test failed before its `afterEach` ran. 
- **Order cleanup by foreign key dependencies** — child tables before parent tables. 
- **Never clear a shared development database** — use a dedicated test database and verify the connection string. 
- Use `RESTART IDENTITY` (PostgreSQL) or `TRUNCATE` with auto-increment reset to ensure predictable IDs.

### Annotated Code Example

```js
// test/cleanup.test.js
const mongoose = require('mongoose');
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');
const Product = require('../models/Product');

describe('Database cleanup', () => {
  beforeEach(async () => {
    // Clean collections in dependency order
    await Product.deleteMany({});
    await User.deleteMany({});
  });

  it('should start with an empty database', async () => {
    const users = await User.countDocuments();
    const products = await Product.countDocuments();
    expect(users).toBe(0);
    expect(products).toBe(0);
  });

  it('should create and clean up a user', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@test.com' })
      .expect(201);

    const count = await User.countDocuments();
    expect(count).toBe(1);
  });

  it('should still start empty in the next test', async () => {
    const count = await User.countDocuments();
    expect(count).toBe(0);  // Previous test's user was cleaned up
  });
});
```

**Expected Output:**
```
PASS  test/cleanup.test.js
  Database cleanup
    ✓ should start with an empty database (18 ms)
    ✓ should create and clean up a user (32 ms)
    ✓ should still start empty in the next test (15 ms)

Tests: 3 passed, 3 total
```

**Why this output:** The `beforeEach` hook clears the `Product` and `User` collections before every test. The first test verifies the database is empty. The second test creates a user and verifies it exists. The third test verifies that the user from the second test was cleaned up — the database is empty again.

---

## Core Concept 5: Transactions in Tests

### Definitions

**Core Definition:** Transaction-per-test is a cleanup strategy where each test runs inside a database transaction that is rolled back after the test completes, leaving the database unchanged.

**Technical Definition:** The test framework wraps each test function in `BEGIN ... ROLLBACK`, so any inserts, updates, or deletes performed during the test are discarded when the test ends. This is the fastest cleanup method because it avoids truncating tables or recreating schemas. It requires that the database driver supports transactions or savepoints, and it fails when the application under test opens its own transactions or uses multiple connections that bypass the test transaction. 

**Beginner-Friendly Explanation:** Instead of cleaning up after each test, you run the test inside a "bubble" — a transaction that never commits. When the test finishes, the bubble pops and everything inside disappears. This is much faster than deleting rows one by one.

### Purposes

- To provide the fastest possible test isolation.
- To eliminate the need for manual cleanup code.
- To ensure the database remains pristine regardless of test success or failure. 
- To enable deterministic assertions without table truncation overhead.

### Syntax Rules and Structure

**Vitest `aroundEach` pattern:**
```js
import { test as baseTest } from 'vitest';
import { createTestDatabase } from './db.js';

export const test = baseTest.extend('db', { scope: 'file' }, async ({}, { onCleanup }) => {
  const db = await createTestDatabase();
  onCleanup(() => db.close());
  return db;
});

test.aroundEach(async (runTest, { db }) => {
  await db.transaction(runTest);
});

test('insert user', async ({ db }) => {
  await db.insert({ name: 'Alice' });
  // rolled back automatically
});
```

**Jest with `pg-transactional-tests`:**
```js
import { testTransaction } from 'pg-transactional-tests';

beforeAll(testTransaction.start);
beforeEach(testTransaction.start);
afterEach(testTransaction.rollback);
afterAll(testTransaction.close);
```

| Hook | Purpose |
|------|---------|
| `beforeAll` | Start the outer transaction (only when queries are made). |
| `beforeEach` | Start a savepoint before each test. |
| `afterEach` | Roll back to the savepoint. |
| `afterAll` | Close all connections. |

**Rules:**
- Transaction rollback works with most ORMs (Sequelize, TypeORM, MikroORM, Drizzle) but **not** Prisma, whose implementation is fundamentally different. 
- If a test does not perform any query, no transaction is started — this avoids unnecessary overhead. 
- Transaction-per-test fails when the application under test uses its own transactions or multiple connections. 
- Use `scope: 'worker'` in Vitest to share a single connection across multiple files per worker. 

### Annotated Code Example

```js
// test/transactional.test.js
const { Pool } = require('pg');
const { testTransaction } = require('pg-transactional-tests');

const pool = new Pool({
  connectionString: process.env.TEST_DATABASE_URL
});

beforeAll(testTransaction.start);
beforeEach(testTransaction.start);
afterEach(testTransaction.rollback);
afterAll(async () => {
  await testTransaction.close();
  await pool.end();
});

describe('Transactional tests', () => {
  it('inserts a user that is rolled back', async () => {
    const client = await pool.connect();
    await client.query('INSERT INTO users (name) VALUES ($1)', ['Alice']);
    const { rows } = await client.query('SELECT COUNT(*) FROM users');
    expect(parseInt(rows[0].count)).toBe(1);
    client.release();
  });

  it('starts with an empty users table', async () => {
    const client = await pool.connect();
    const { rows } = await client.query('SELECT COUNT(*) FROM users');
    expect(parseInt(rows[0].count)).toBe(0);
    client.release();
  });
});
```

**Expected Output:**
```
PASS  test/transactional.test.js
  Transactional tests
    ✓ inserts a user that is rolled back (28 ms)
    ✓ starts with an empty users table (12 ms)

Tests: 2 passed, 2 total
```

**Why this output:** The first test inserts a user inside a transaction. The second test verifies that the user is gone — the transaction was rolled back. No `DELETE` or `TRUNCATE` statements were needed. The entire cleanup is handled by the database's transaction mechanism.

---

## Core Concept 6: In-Memory Database Wrappers

### Definitions

**Core Definition:** In-memory database wrappers are packages that start a real database engine in memory (or in a temporary directory) for the duration of the test run, providing realistic database behaviour without a persistent server.

**Technical Definition:** `mongodb-memory-server` downloads a real `mongod` binary and starts it on a random port against a temporary directory; data is held in memory and discarded when the process stops. SQLite `:memory:` is a real SQL engine built into Node.js that creates a database in RAM. Both provide realistic query behaviour — indexes, constraints, aggregation — without external dependencies. 

**Beginner-Friendly Explanation:** Instead of installing and running a real MongoDB or PostgreSQL server, you use a package that spins up a database in memory for your tests. It behaves like the real thing but starts instantly and leaves no trace when it stops.

### Purposes

- To provide a realistic database for tests without external infrastructure.
- To eliminate the need for Docker or a running database server during development.
- To ensure tests are fully isolated and self-contained.
- To enable fast startup and teardown.

### Sub-Feature 6.1: `mongodb-memory-server`

**Setup:**
```js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

**Rules:**
- A fresh `mongod` process uses about 7 MB of memory. 
- Boot the server once in `globalSetup` and reuse it across files for better performance. 
- Use `globalTeardown` to stop the server after the last test file. 
- For transactions, indexes, and aggregation pipelines, a real container may be more honest. 

### Sub-Feature 6.2: SQLite `:memory:`

**Setup:**
```js
const Database = require('better-sqlite3');

let db;

beforeAll(() => {
  db = new Database(':memory:');
  db.exec(`
    CREATE TABLE users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      email TEXT UNIQUE NOT NULL
    )
  `);
});

afterAll(() => {
  db.close();
});
```

**Rules:**
- SQLite `:memory:` is a real SQL engine — it supports joins, indexes, and constraints. 
- `node:sqlite` is a release candidate in Node.js 25 and 26. 
- The same repository pattern can target SQLite in tests and PostgreSQL in production. 

### Annotated Code Example

```js
// test/mongo-memory.test.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  await User.deleteMany({});
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

describe('User API with in-memory MongoDB', () => {
  it('creates a user and retrieves it', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@test.com' })
      .expect(201);

    expect(response.body.name).toBe('Alice');

    const userInDb = await User.findOne({ email: 'alice@test.com' });
    expect(userInDb).not.toBeNull();
  });

  it('starts with an empty database', async () => {
    const count = await User.countDocuments();
    expect(count).toBe(0);
  });
});
```

**Expected Output:**
```
PASS  test/mongo-memory.test.js
  User API with in-memory MongoDB
    ✓ creates a user and retrieves it (78 ms)
    ✓ starts with an empty database (22 ms)

Tests: 2 passed, 2 total
```

**Why this output:** `MongoMemoryServer` starts a real MongoDB instance in memory. The first test creates a user via the API and verifies it exists in the database. The second test verifies that the user was cleaned up by the `afterEach` hook. The entire suite is self-contained — no external MongoDB server is required.

---

## Core Concept 7: Migration Execution in Test Lifecycles

### Definitions

**Core Definition:** Migration execution in test lifecycles means applying database schema migrations to the test database before tests run, ensuring that the test schema matches the production schema.

**Technical Definition:** Migrations are executed via a `globalSetup` hook (Jest) or a `beforeAll` hook that runs the migration command programmatically. Tools like Knex, Umzug, Prisma Migrate, and `node-pg-migrate` provide programmatic APIs for running migrations. The test database should be migrated from scratch (or from a known migration state) before each test run. Using ORM `sync()` methods instead of migrations is discouraged because it bypasses migration issues that exist in production. 

**Beginner-Friendly Explanation:** Your production database has a specific structure (tables, columns, indexes) defined by migrations. Your test database needs the exact same structure, or your tests will pass against a schema that doesn't exist in production. Running migrations in the test lifecycle ensures parity.

### Purposes

- To ensure the test schema matches the production schema.
- To catch migration bugs (broken migrations, missing indexes) before deployment.
- To avoid the false confidence of ORM `sync()` methods. 
- To enable deterministic schema state across all test runs.

### Syntax Rules and Structure

**Jest `globalSetup`:**
```js
// jest.config.js
module.exports = {
  globalSetup: './test/global-setup.js'
};
```

```js
// test/global-setup.js
const { execSync } = require('child_process');

module.exports = async () => {
  // Run migrations against the test database
  execSync('npm run migrate:latest', {
    env: { ...process.env, DATABASE_URL: process.env.TEST_DATABASE_URL }
  });
};
```

**Programmatic migration with Umzug:**
```js
const { Umzug } = require('umzug');
const { migrator } = require('./migrator');

module.exports = async () => {
  const migration = migrator(sequelize);
  await migration.up();
};
```

**Rules:**
- Use `globalSetup` to run migrations once before all test files. 
- Store the last migration check locally and skip migration execution if the migration folder has not changed. 
- Never use ORM `sync()` in tests — it bypasses migration validation. 
- Test migrations against a copy of production data when possible. 

### Annotated Code Example

```js
// test/global-setup.js
const { execSync } = require('child_process');

module.exports = async () => {
  console.log('Running migrations on test database...');

  execSync('npx knex migrate:latest', {
    env: {
      ...process.env,
      DATABASE_URL: process.env.TEST_DATABASE_URL
    },
    stdio: 'inherit'
  });

  console.log('Migrations complete.');
};
```

```js
// jest.config.js
module.exports = {
  globalSetup: './test/global-setup.js',
  testEnvironment: 'node'
};
```

**Expected Output (CI log):**
```
Running migrations on test database...
Batch 1 run: 12 migrations
Migrations complete.
PASS  test/integration/users.test.js
...
```

**Why this output:** The `globalSetup` hook runs once before any test file. It executes `knex migrate:latest` against the test database, ensuring the schema matches production. The tests then run against a fully migrated schema. If a migration is broken, the test run fails before any test executes — catching the issue early.

---

## Core Concept 8: Connection Pool Management

### Definitions

**Core Definition:** Connection pool management in tests is the practice of explicitly closing database connections and pools in teardown hooks so that the test runner can exit cleanly without hanging on open handles.

**Technical Definition:** When a test suite opens a database connection pool (via `pg.Pool`, Mongoose, Prisma, or an ORM), that pool keeps the Node.js event loop alive. Jest detects these as "open handles" and refuses to exit, printing the message: `Jest did not exit one second after the test run has completed`. The fix is to call `pool.end()`, `mongoose.connection.close()`, or the equivalent close method in an `afterAll` hook, and `await` the result. 

**Beginner-Friendly Explanation:** Your database connection is like an open phone line. If you don't hang up, the test runner stays on the line forever, waiting for something that will never happen. Closing the connection tells Jest: "I'm done, you can exit now."

### Purposes

- To prevent Jest from hanging after tests complete.
- To ensure CI pipelines do not time out waiting for Jest to exit.
- To avoid resource leaks in long-running test suites.
- To make test runs faster and more reliable.

### Syntax Rules and Structure

**PostgreSQL (`pg`):**
```js
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.TEST_DATABASE_URL });

afterAll(async () => {
  await pool.end();
});
```

**MongoDB (Mongoose):**
```js
afterAll(async () => {
  await mongoose.connection.close();
});
```

**MongoDB Memory Server:**
```js
afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

**Prisma:**
```js
afterAll(async () => {
  await prisma.$disconnect();
});
```

| Handle Type | Fix |
|-------------|-----|
| `TCPWRAP` (database connection) | `await pool.end()` or `await connection.close()`.  |
| `TCPSERVERWRAP` (HTTP server) | `await app.close()`.  |
| `Timeout` (setInterval) | `clearInterval(timer)`.  |

**Rules:**
- Always `await` the close method — a non-awaited close may not complete before Jest exits. 
- Close the database connection **before** stopping the in-memory server. 
- Use `--detectOpenHandles` to identify which handle is keeping Jest alive. 
- `--forceExit` is a temporary fix that hides the underlying problem; fix the root cause instead. 

### Annotated Code Example

```js
// test/pool-management.test.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  await User.deleteMany({});
});

afterAll(async () => {
  // Close the database connection first
  await mongoose.disconnect();
  // Then stop the in-memory server
  await mongoServer.stop();
});

describe('Connection pool management', () => {
  it('should close all connections after tests', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect(200);

    expect(response.body).toEqual([]);
  });
});
```

**Expected Output:**
```
PASS  test/pool-management.test.js
  Connection pool management
    ✓ should close all connections after tests (35 ms)

Tests: 1 passed, 1 total
```

**Why this output:** The `afterAll` hook closes the Mongoose connection and then stops the in-memory MongoDB server. Jest exits immediately after the test run because there are no open handles keeping the event loop alive. Without these teardown calls, Jest would print `Jest did not exit one second after the test run has completed` and hang in CI.

---

## References

- supabase-test on npm — https://socket.dev/npm/package/supabase-test
- neon-test on npm — https://www.npmjs.com/package/neon-test
- Factory Patterns, Database Seeding, and Test Data Isolation (alpha-engineer) — https://github.com/rnavarych/alpha-engineer/blob/main/plugins/roles/role-aqa/skills/test-data-management/references/factories-seeding-isolation.md
- How to Run Jest Integration Tests in Parallel Using Isolated SQL Schemas (DEV Community) — https://dev.to/cseby92/how-to-run-jest-integration-tests-in-parallel-using-isolated-sql-schemas-1bm7
- mongodb-memory-server GitHub — https://github.com/typegoose/mongodb-memory-server
- drizzle-orm-test on npm — https://www.npmjs.com/package/drizzle-orm-test
- Testing Handlers – Node.js Built-in Test Runner — https://webcodingcenter.com/node.js/testing-with-the-built-in-test-runner--testing-http-handlers-and-data-access.html
- db-sandbox on npm — https://www.npmjs.com/package/db-sandbox
- Vitest Database Transaction per Test — https://vitest.dev/guide/recipes/db-transaction
- pg-transactional-tests GitHub — https://github.com/romeerez/pg-transactional-tests
- nodejs-integration-tests-best-practices GitHub — https://github.com/Yurishama/nodejs-integration-tests-best-practices
- The story of Jest hanging in CI (DevelopersIO) — https://dev.classmethod.jp/en/articles/jest-open-handles-localstack-floci/
- Detecting and Fixing Open Handles (bmad-labs/skills) — https://github.com/bmad-labs/skills/blob/main/skills/typescript-unit-testing/references/common/detect-open-handles.md
- How to seed databases in Node.js (CoreUI) — https://coreui.io/blog/how-to-seed-databases-in-node-js/
- DatabaseIsolationStrategy enum (pub.dev) — https://pub.dev/documentation/database_isolation/latest/
- Revisions to Error with pg in Jest-ts testing (Stack Overflow) — https://stackoverflow.com/questions/78793929
- Sequelize migration testing in CI (john-craft/stateful-test-in-ci) — https://github.com/john-craft/stateful-test-in-ci
- Knex migration lifecycle hooks (knex/knex PR #6264) — https://github.com/knex/knex/pull/6264
- tiny-fixtures on npm — https://www.npmjs.com/package/tiny-fixtures
- testdata-sweeper on npm — https://www.npmjs.com/package/testdata-sweeper