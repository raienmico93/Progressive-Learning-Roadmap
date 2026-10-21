# Database Fundamentals in Express.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Database fundamentals in Express.js are the principles and practices for persisting, querying, and managing application data in a reliable, performant, and secure manner using a database system integrated with an Express backend.

**Technical Definition**
A database is a specialized software system designed to store, retrieve, and manage structured data reliably and efficiently. Unlike keeping data in memory (which vanishes when the server restarts) or flat files (which are painfully slow to query), databases provide **persistence**, **structured querying**, **concurrent access**, **integrity enforcement**, and **scalability**. Express.js is database-agnostic: it does not enforce any specific database solution, allowing developers to choose the most suitable database for their project requirements. Databases fall into two broad families—**relational (SQL)** databases, which store data in tables with predefined schemas and enforce ACID transactions, and **NoSQL** databases, which encompass document stores, key-value stores, wide-column stores, and graph databases with flexible schemas and horizontal scalability.

**Beginner-Friendly Explanation**
A database is where your application keeps its data permanently. When a user signs up, places an order, or writes a comment, that information needs to survive server restarts and be retrievable later. Without a database, your app would forget everything the moment it restarts. Express itself does not include a database—it is database-agnostic—so you choose one that fits your data: a relational database like PostgreSQL when your data is highly interconnected and needs strong consistency, or a NoSQL database like MongoDB when your data is flexible and you need to scale horizontally.

---

### Key Characteristics

- **Persistence** — Data survives server restarts, crashes, and deployments.
- **Structured querying** — Complex questions (“all orders from last month over $100”) are answered in milliseconds.
- **Concurrent access** — Thousands of users can read and write simultaneously without corruption.
- **Integrity enforcement** — Impossible states (e.g., an order without a customer) are prevented.
- **Scalability** — The database grows from hundreds to millions of records without rewriting the application.
- **Express is database-agnostic** — The framework imposes no database; connection is made through native drivers, query builders, ORMs, or ODMs.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A database server or embedded database (PostgreSQL, MySQL, SQLite, MongoDB, or Redis).
- A database driver or ORM/ODM package (e.g., `pg`, `mysql2`, `mongoose`, `prisma`, `sequelize`).
- Environment variables configured for database credentials (`dotenv`).
- Basic understanding of SQL (for relational databases) or document modeling (for NoSQL).

---

### Related Programming Areas

- **Data modeling** — Schema design, normalization, and denormalization.
- **Query optimization** — Indexing, connection pooling, and N+1 query prevention.
- **Transactions** — ACID guarantees and rollback handling.
- **Security** — SQL injection prevention, credential management, and least-privilege database users.
- **Migrations** — Version-controlled schema evolution.
- **Caching** — Redis and in-memory caching for performance.

---

## Core Concept 1: Databases

### Definitions

**Core Definition**
A database is a software system that stores, retrieves, and manages structured data with guarantees of persistence, concurrency, integrity, and query capability.

**Technical Definition**
A database management system (DBMS) provides a structured interface for defining schemas, inserting and querying data, enforcing constraints, and managing concurrent access. In the Express ecosystem, databases are accessed through **native drivers** (direct communication), **query builders** (programmatic SQL construction), **ORMs** (object-relational mappers for SQL), or **ODMs** (object-document mappers for NoSQL). The choice of access method affects productivity, type safety, and performance.

**Beginner-Friendly Explanation**
A database is the permanent filing cabinet for your application. It stores everything your users create and lets you find it later quickly and safely.

---

### Purposes

- To persist application data beyond the lifecycle of the server process.
- To enable structured querying of data at scale.
- To handle concurrent reads and writes without corruption.
- To enforce data integrity through constraints and transactions.
- To provide a foundation for scalability as the application grows.

---

### Syntax Rules and Structure

#### Complete General Syntax: Connecting to a Database in Express

```javascript
// Native driver approach
const { MongoClient } = require('mongodb');
const client = new MongoClient(process.env.MONGODB_URI);
await client.connect();
app.locals.db = client.db('my_database');

// ORM/ODM approach
const mongoose = require('mongoose');
await mongoose.connect(process.env.MONGODB_URI);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `MongoClient` | Native MongoDB driver |
| `mongoose.connect()` | ODM connection with schema support |
| `app.locals.db` | Makes the connection available to routes |

#### Syntax Rules

- Database credentials must come from environment variables, never hard-coded.
- Connection should be established once at application startup, not per request.
- Use connection pooling for SQL databases to reuse connections efficiently.
- Handle connection errors and log them at startup.

#### Constraints and Limitations

- Express does not manage database connections; the application is responsible for lifecycle and pooling.
- In-memory arrays are not databases—they lose data on restart and suffer race conditions.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Native MongoDB Driver Connection

```javascript
const express = require('express');
const { MongoClient } = require('mongodb');
const app = express();

const client = new MongoClient(process.env.MONGODB_URI);

async function start() {
  await client.connect();
  app.locals.db = client.db('my_database');
  console.log('Connected to MongoDB');
}

app.get('/users', async (req, res) => {
  const users = await app.locals.db.collection('users').find({}).toArray();
  res.json(users);
});

start().then(() => app.listen(3000));
```

**Expected Output**

```
GET /users → 200 [{ "name": "Alice" }, { "name": "Bob" }]
```

**Why This Output Occurs**

- The native driver connects once, exposes the database on `app.locals`, and the route queries the `users` collection.

---

### Real-World Cases

- **E-commerce platforms** — Persisting orders, inventory, and customer records.
- **Social applications** — Storing posts, comments, and relationships.
- **Analytics dashboards** — Querying aggregated event data.

---

## Core Concept 2: Relational Databases

### Definitions

**Core Definition**
A relational database is a collection of data items organized as a set of tables with columns and rows, where relationships between tables are defined through primary keys and foreign keys.

**Technical Definition**
Relational databases store structured data in **tables** (relations). Each **row** is a unique record identified by a **primary key**; each **column** holds a specific attribute. **Foreign keys** reference the primary key of another table, enforcing referential integrity. Integrity constraints include primary keys, foreign keys, `NOT NULL`, `UNIQUE`, `DEFAULT`, and `CHECK`. Relational databases are **ACID-compliant**: transactions are atomic (all-or-nothing), consistent (adhere to all rules), isolated (independent of concurrent transactions), and durable (permanent once committed). SQL (Structured Query Language) is the standard ANSI interface for querying and managing relational databases.

**Beginner-Friendly Explanation**
A relational database is like a set of linked spreadsheets. Each spreadsheet is a table—customers, orders, products. Each row is one record. Columns are the fields. A primary key uniquely identifies each row (like a customer ID), and a foreign key links rows across tables (an order’s `customer_id` points to the customer who placed it). This structure prevents impossible states, like an order belonging to a customer who does not exist.

---

### Purposes

- To store highly structured, interconnected data with strong consistency guarantees.
- To enforce referential integrity through primary and foreign keys.
- To support complex queries, joins, aggregations, and reporting.
- To provide ACID transactions for financial, inventory, and other critical operations.
- To use a mature, standardized query language (SQL).

---

### Syntax Rules and Structure

#### Complete General Syntax: Table Definition with Keys and Constraints

```sql
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
  total DECIMAL(10, 2) CHECK (total >= 0),
  status VARCHAR(20) DEFAULT 'pending'
);
```

**Component Breakdown**

| Constraint | Role |
|------------|------|
| `PRIMARY KEY` | Uniquely identifies each row |
| `FOREIGN KEY` | References another table’s primary key |
| `UNIQUE` | Prevents duplicate values |
| `NOT NULL` | Requires a value |
| `CHECK` | Enforces a condition |
| `DEFAULT` | Supplies a default value |

#### ACID Properties

| Property | Meaning |
|----------|---------|
| **Atomicity** | Entire transaction succeeds or fails as a unit |
| **Consistency** | Data adheres to all defined rules and constraints |
| **Isolation** | Concurrent transactions do not interfere |
| **Durability** | Committed changes are permanent |

#### Syntax Rules

- Every table should have a primary key.
- Foreign keys must reference an existing primary key (referential integrity).
- Transactions must be explicitly committed or rolled back.
- Indexes should be created on columns used in `WHERE`, `JOIN`, and `ORDER BY` clauses.

#### Constraints and Limitations

- Relational schemas are **predefined**; schema changes require migrations.
- Horizontal scaling is harder than with NoSQL; relational databases typically scale vertically and through read replicas.
- Complex joins can become expensive at very large scale.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: PostgreSQL Transaction with Connection Pool

```javascript
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL, max: 20 });

app.post('/transfer', async (req, res) => {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [req.body.amount, req.body.fromId]);
    await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [req.body.amount, req.body.toId]);
    await client.query('COMMIT');
    res.json({ success: true });
  } catch (err) {
    await client.query('ROLLBACK');
    res.status(500).json({ error: err.message });
  } finally {
    client.release();
  }
});
```

**Expected Output**

```
POST /transfer  { "fromId": 1, "toId": 2, "amount": 100 }
→ 200 { "success": true } (both balances updated atomically)

POST /transfer  (insufficient funds error)
→ 500 { "error": "..." } (both balances unchanged due to ROLLBACK)
```

**Why This Output Occurs**

- The transaction guarantees that either both updates commit or neither does, preserving financial consistency.

---

### Real-World Cases

- **Banking and payments** — ACID transactions for transfers and ledger updates.
- **Inventory management** — Preventing overselling through atomic stock decrements.
- **ERP and CRM systems** — Highly relational data with complex reporting requirements.

---

## Core Concept 3: NoSQL Databases

### Definitions

**Core Definition**
NoSQL databases are a broad family of non-relational databases that store data in flexible, schema-optional formats such as documents, key-value pairs, wide columns, or graphs, optimized for horizontal scalability and specific access patterns.

**Technical Definition**
NoSQL encompasses four major families: **document stores** (MongoDB, CouchDB) store JSON-like documents with flexible schemas; **key-value stores** (Redis, DynamoDB) provide O(1) lookups by key; **wide-column stores** (Cassandra, ScyllaDB) organize data by rows and dynamic columns optimized for write-heavy workloads; and **graph databases** (Neo4j) store entities and relationships natively for relationship-heavy queries. NoSQL databases generally favor **availability and partition tolerance** over strong consistency (AP in CAP terms), though many offer tunable consistency. They are designed for horizontal scaling across commodity hardware.

**Beginner-Friendly Explanation**
A NoSQL database is a specialized storage system. Instead of forcing everything into linked tables, it picks a shape that matches how the application actually reads and writes data. A document store keeps whole JSON objects together (like a blog post with its comments embedded). A key-value store is like a dictionary—give it a key, get the value instantly. A wide-column store is optimized for writing massive amounts of time-series data. A graph database is built for questions like “who is friends with whom.”

---

### Purposes

- To store semi-structured or rapidly evolving data without migrations.
- To scale horizontally across many servers.
- To optimize for specific access patterns (key lookups, document retrieval, relationship traversal).
- To handle very high write throughput and large volumes of data.

---

### Syntax Rules and Structure

#### Complete General Syntax: MongoDB Document Store with Mongoose

```javascript
const mongoose = require('mongoose');

const postSchema = new mongoose.Schema({
  title: { type: String, required: true },
  content: String,
  tags: [String],
  comments: [{ body: String, author: String, createdAt: { type: Date, default: Date.now } }],
  createdAt: { type: Date, default: Date.now }
});

const Post = mongoose.model('Post', postSchema);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Schema` | Defines the shape of a document (optional fields) |
| `[String]` | Array of strings (embedded tags) |
| `[{ ... }]` | Embedded subdocuments (comments) |

#### NoSQL Family Comparison

| Family | Example | Best For |
|--------|---------|----------|
| **Document** | MongoDB | Content management, catalogs, user profiles |
| **Key-Value** | Redis, DynamoDB | Session caches, shopping carts, user preferences |
| **Wide-Column** | Cassandra, ScyllaDB | Metrics, telemetry, time-series event data |
| **Graph** | Neo4j | Fraud detection, recommendation engines, social networks |

#### Syntax Rules

- Model data around **access patterns**, not around normalized entities.
- Embed data that is read together; reference data that is shared or large.
- Design for **query-first** modeling: know the queries before designing the schema.
- Use tunable consistency settings appropriate to the use case.

#### Constraints and Limitations

- NoSQL databases generally lack multi-document ACID transactions (though some now support them).
- JOINs are typically not available; relationships must be modeled through embedding or application-level joins.
- Schema flexibility can lead to inconsistent data if not governed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: MongoDB Document with Embedded Comments

```javascript
const post = new Post({
  title: 'Database Fundamentals',
  content: 'A guide to SQL and NoSQL.',
  tags: ['database', 'express'],
  comments: [
    { body: 'Great article!', author: 'Alice' },
    { body: 'Very helpful.', author: 'Bob' }
  ]
});
await post.save();

const found = await Post.findById(post._id);
console.log(found.comments.length); // 2
```

**Expected Output**

```
2
```

**Why This Output Occurs**

- Comments are embedded directly in the post document, so retrieving the post retrieves its comments in a single read—no join required.

---

### Real-World Cases

- **Content platforms** — MongoDB stores articles with embedded comments and flexible metadata.
- **Session management** — Redis stores session data for fast key lookups.
- **IoT telemetry** — Cassandra ingests high-volume sensor readings.
- **Recommendation engines** — Neo4j traverses user–item relationship graphs.

---

## Core Concept 4: Database Selection

### Definitions

**Core Definition**
Database selection is the decision process of choosing a database system—or combination of systems—based on data structure, consistency requirements, query complexity, scaling pattern, and team expertise.

**Technical Definition**
The choice between relational and NoSQL databases is governed by the **CAP theorem** (a distributed system cannot simultaneously guarantee consistency, availability, and partition tolerance) and by the application's access patterns. **Polyglot persistence**—using multiple database technologies for different services or use cases—is common in modern Express backends: PostgreSQL as the system of record, Redis for caching and sessions, Elasticsearch for search, and ClickHouse for analytics. The decision should be driven by concrete bottlenecks, not by hype: “If you cannot articulate which Postgres limit you are hitting in a number, a NoSQL migration will move the problem, not solve it.”

**Beginner-Friendly Explanation**
Choosing a database is like choosing the right tool for a job. If your data is highly connected and needs to be perfectly consistent (like money), use a relational database. If your data is flexible and you need to write massive volumes quickly, a NoSQL database may be better. Many real applications use more than one database—each doing what it does best.

---

### Purposes

- To match the database's strengths to the application's actual data and access patterns.
- To avoid the performance and consistency pitfalls of choosing the wrong database family.
- To enable **polyglot persistence**, where each service uses the most appropriate store.
- To account for operational complexity, team skills, and long-term maintenance.

---

### Syntax Rules and Structure

#### Complete General Syntax: Decision Framework

| Factor | Choose SQL | Choose NoSQL |
|--------|-----------|--------------|
| **Data structure** | Highly relational, normalized | Unstructured, evolving schema |
| **Consistency** | Strong consistency required (financial, inventory) | Eventual consistency acceptable (social feeds, analytics) |
| **Query complexity** | Complex joins, aggregations, reporting | Simple lookups, document retrieval |
| **Scaling pattern** | Vertical + read replicas | Horizontal sharding |
| **Transactions** | Multi-row ACID transactions | Single-document or best-effort |
| **Team expertise** | Strong SQL knowledge | Rapid prototyping, flexible requirements |

#### CAP Theorem Trade-offs

| Choice | Guarantees | Typical Databases |
|--------|-----------|-------------------|
| **CP** (Consistency + Partition tolerance) | Rejects requests during partitions to preserve consistency | MongoDB (default), HBase |
| **AP** (Availability + Partition tolerance) | Continues serving during partitions, accepts eventual consistency | Cassandra, DynamoDB, CouchDB |
| **CA** (Consistency + Availability) | Only possible when no partitions occur (single node) | Traditional single-node RDBMS |

#### Syntax Rules

- **Start with PostgreSQL** unless a concrete bottleneck forces another choice.
- Use **polyglot persistence** when different services have genuinely different needs.
- Measure before migrating: index, cache, and query optimization often solve perceived NoSQL needs.
- Consider operational overhead: each additional database adds deployment, backup, and monitoring complexity.

#### Constraints and Limitations

- The CAP theorem is often misread as a permanent trilemma; good systems maximize consistency and availability during normal operation and trade only during partitions.
- Polyglot persistence increases operational complexity and requires expertise across multiple systems.
- NoSQL databases are not inherently faster; they are faster for specific access patterns.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Polyglot Persistence in Express (PostgreSQL + Redis)

```javascript
const { Pool } = require('pg');
const redis = require('redis');

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const redisClient = redis.createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

async function getProductWithCache(req, res) {
  const { id } = req.params;

  // 1. Try Redis cache
  const cached = await redisClient.get(`product:${id}`);
  if (cached) return res.json(JSON.parse(cached));

  // 2. Fallback to PostgreSQL
  const result = await pool.query('SELECT * FROM products WHERE id = $1', [id]);
  if (result.rows.length === 0) return res.status(404).json({ error: 'Not found' });

  // 3. Populate cache
  await redisClient.setEx(`product:${id}`, 3600, JSON.stringify(result.rows[0]));
  res.json(result.rows[0]);
}
```

**Expected Output**

```
GET /products/1  (first request)  → 200 { product data } (from PostgreSQL, cached in Redis)
GET /products/1  (second request) → 200 { product data } (from Redis, faster)
```

**Why This Output Occurs**

- The middleware checks Redis first, falls back to PostgreSQL on a cache miss, and populates the cache for subsequent requests.

---

### Real-World Cases

- **E-commerce** — PostgreSQL for orders and inventory; Redis for cart and session caching.
- **Social platforms** — PostgreSQL for user accounts and permissions; MongoDB for flexible feed posts; Elasticsearch for search.
- **Analytics** — ClickHouse or TimescaleDB for time-series aggregation alongside PostgreSQL for transactional data.

---

## Core Concept 5: Database Best Practices

### Definitions

**Core Definition**
Database best practices in Express.js are the design, security, and operational patterns that keep database interactions reliable, performant, secure, and maintainable.

**Technical Definition**
Best practices encompass: **connection pooling** to reuse database connections; **parameterized queries** to prevent SQL injection; **environment-variable credential management**; **transaction handling** with commit and rollback; **normalization** of relational schemas with deliberate denormalization only after measurement; **indexing** of query-critical columns; **N+1 query prevention**; **migration and seeding** for version-controlled schema evolution; **least-privilege database users**; and **SSL/TLS** for remote connections.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your database from becoming a slow, insecure mess. Use a connection pool so you are not opening a new connection for every request. Use parameterized queries so attackers cannot inject SQL. Keep credentials in environment variables, not in code. Use transactions when multiple changes must succeed or fail together. Design your schema carefully, add indexes where you query, and never let user input become part of a SQL string.

---

### Purposes

- To prevent SQL injection and credential leakage.
- To optimize performance through pooling, indexing, and query design.
- To maintain data integrity through transactions and constraints.
- To keep schema changes version-controlled and reproducible.
- To enforce least privilege and encrypt connections.

---

### Syntax Rules and Structure

#### Complete General Syntax: Connection Pool and Parameterized Query

```javascript
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000
});

// Parameterized query (safe)
const result = await pool.query('SELECT * FROM users WHERE id = $1', [userId]);

// NEVER do this (SQL injection vulnerability)
// await pool.query(`SELECT * FROM users WHERE id = ${userId}`);
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Connection pooling | `new Pool({ max: 20 })` |
| Parameterized queries | `$1` placeholders with an array of values |
| Credential management | `process.env.DATABASE_URL` |
| Transaction | `BEGIN` → operations → `COMMIT` / `ROLLBACK` |
| Indexing | `CREATE INDEX ON orders (customer_id)` |

#### Syntax Rules

- **Always use parameterized queries**; never concatenate user input into SQL.
- **Always use a connection pool**; never open a new connection per request.
- **Always release connections** back to the pool in a `finally` block.
- **Always use transactions** for multi-step operations that must be atomic.
- **Store credentials in environment variables** and enable SSL/TLS for remote databases.
- **Normalize first, denormalize only after measuring** a specific performance bottleneck.

#### Constraints and Limitations

- Pool size must be tuned to the database's connection limits; too many connections degrade performance.
- Over-denormalization causes data anomalies; under-denormalization causes slow joins.
- Migrations in production require careful planning and rollback strategies.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Database Best-Practice Pipeline

```javascript
const express = require('express');
const { Pool } = require('pg');
const { body, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000
});

// Parameterized query + input validation
app.get('/users/:id',
  param('id').isInt({ min: 1 }).toInt(),
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });

    const result = await pool.query('SELECT id, email, name FROM users WHERE id = $1', [req.params.id]);
    if (result.rows.length === 0) return res.status(404).json({ error: 'User not found' });
    res.json(result.rows[0]);
  }
);

// Transaction with rollback
app.post('/transfer', async (req, res) => {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [req.body.amount, req.body.fromId]);
    await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [req.body.amount, req.body.toId]);
    await client.query('COMMIT');
    res.json({ success: true });
  } catch (err) {
    await client.query('ROLLBACK');
    res.status(500).json({ error: 'Transfer failed' });
  } finally {
    client.release();
  }
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1      → 200 { "id": 1, "email": "...", "name": "..." }
GET /users/abc    → 400 { "errors": [...] }
POST /transfer    (valid) → 200 { "success": true }
POST /transfer    (error) → 500 { "error": "Transfer failed" } (both balances unchanged)
```

**Why This Output Occurs**

- Validation rejects invalid input before the query; the parameterized query prevents injection; the transaction ensures atomicity; the connection is released in `finally`.

---

### Real-World Cases

- **Production APIs** — Connection pooling, parameterized queries, and transaction handling.
- **Financial systems** — ACID transactions with rollback and least-privilege database users.
- **High-traffic applications** — Indexed columns and N+1 query prevention.

---

## References

- Express Database Overview · Compile-N-Run — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/6-express-database-integration/0-express-database-overview.mdx
- Database Integration · srivtx/js-miden — https://github.com/srivtx/js-miden/blob/b66962fd1e3b6198342f57de8ea48b010ba6742b/docs_js_backend/03-database-integration/README.md
- What is a Relational Database? · AWS — https://aws.amazon.com/relational-database/
- NoSQL Databases: Picking the Right Non-Relational Tool · Handbook Academy — https://raw.githubusercontent.com/handbook-academy/engineering-handbook/refs/heads/main/content/hld/part-2-building-blocks/05-nosql-databases.md
- CAP Theorem · AWS Documentation — https://docs.aws.amazon.com/whitepapers/latest/availability-and-beyond-improving-resilience/cap-theorem.html
- NoSQL Databases: Critical Analysis and Comparison · IEEE — https://ieeexplore.ieee.org/document/8283999
- What are the best practices for using MySQL with Node.js and Express? · GitHub Community — https://github.com/orgs/community/discussions/177127
- Node.js Best Practices · GitHub (fil512/upship) — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/fil512/upship/nodejs-best-practices/SKILL.md
- Hybrid Data Storage Architecture · IEEE — https://ieeexplore.ieee.org/document/11282431
- Adatbázistrendek és technológiák · MTA OJS — https://ojs.mtak.hu/index.php/dk/article/view/18633