# Database Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Database performance optimisation is the discipline of reducing the time and resources required to execute database queries, by eliminating inefficient query patterns, adding appropriate indexes, profiling slow queries, sizing connection pools correctly, offloading read traffic to caches, and distributing read/write workloads across replicas.

**Technical Definition:** Database performance in Node.js applications is governed by several factors: the **query pattern** (N+1 queries, over-fetching, missing projections), the **index strategy** (primary keys, compound indexes, covering indexes, partial indexes), the **query plan** (as revealed by `EXPLAIN ANALYZE`), the **connection pool configuration** (min/max connections, idle timeouts, queue limits), the **caching layer** (Redis or Memcached for repeated reads), and the **topology** (primary for writes, replicas for reads). These concerns are interconnected: a missing index causes slow queries, which exhausts the connection pool, which cascades into application-wide latency. 

**Beginner-Friendly Explanation:** Think of a database as a library. If the books are not organised (no indexes), finding one takes forever. If you ask for 100 books one at a time (N+1 queries), the librarian walks to the shelf 100 times. If you photocopy the same popular book for every visitor (no caching), you waste time and paper. Database performance is about organising the library well (indexes), asking for everything in one trip (query optimisation), keeping popular books at the front desk (caching), and having multiple librarians (connection pool) and multiple reading rooms (replicas). 

### Key Characteristics

- **N+1 is the most common performance killer:** One query to fetch a list, then N queries to fetch related data for each item. 
- **Indexes trade write speed for read speed:** Every index speeds up reads but slows down inserts, updates, and deletes. 
- **`EXPLAIN ANALYZE` reveals the truth:** Never guess which query is slow — measure it with the database's query planner. 
- **Connection pools are finite:** Each connection consumes memory on the database server; too many connections cause contention. 
- **Cache invalidation is the hard part:** Caching reads is easy; keeping the cache consistent with the database is the challenge. 
- **Read replicas introduce replication lag:** Reads from replicas may return stale data; design for eventual consistency. 

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **A relational database** (PostgreSQL, MySQL) or document database (MongoDB).
- **A database driver or ORM:** `pg`, `mysql2`, `mongoose`, Prisma, Drizzle, or Knex.
- **A caching server:** Redis (`ioredis`) or Memcached.
- **Database access** to run `EXPLAIN` commands and create indexes.
- **Basic understanding of SQL, indexes, and database internals.**

### Related Programming Areas

- **Express Performance:** Database latency is a primary contributor to API response time.
- **Caching:** Redis/Memcached reduce database load.
- **Observability:** Metrics and tracing reveal database bottlenecks.
- **ORM configuration:** Prisma, Mongoose, and TypeORM have specific N+1 mitigations.
- **Infrastructure:** Read replicas, connection proxies (PgBouncer), and database clusters.

### Core Concepts

1. **Query Optimisation** — eliminating N+1 queries and selecting only required fields.
2. **Indexes** — primary, compound, and partial indexes.
3. **Query Profiling** — using `EXPLAIN` to diagnose slow queries.
4. **Connection Pool Sizing** — balancing min/max connections against hardware limits.
5. **Database Caching Layers** — offloading heavy reads to Redis or Memcached.
6. **Read/Write Splitting** — routing writes to primary and reads to replicas.

---

## Core Concept 1: Query Optimisation

### Definitions

**Core Definition:** Query optimisation is the practice of writing database queries that fetch the required data in the fewest possible round trips and with the smallest possible result set, eliminating redundant queries and over-fetching.

**Technical Definition:** The two most impactful query optimisation techniques are **eliminating N+1 queries** and **projecting only required fields**. An N+1 query pattern occurs when an application fetches a list of N records with one query, then issues one additional query per record to fetch related data — resulting in N+1 total queries. The fix is to use a JOIN, an `IN` clause, or the ORM's eager-loading mechanism to fetch all related data in a single query (or a constant number of queries). Projection means using `SELECT column1, column2` (or the ORM's `select` method) instead of `SELECT *`, reducing the data transferred from the database, the memory used, and the serialisation cost. 

**Beginner-Friendly Explanation:** Imagine you are making a shopping list. An N+1 query is like going to the store once for milk, coming home, going back for eggs, coming home, going back for bread. A single optimised query is like making one trip with the full list. Projection is like buying only what you need instead of buying one of everything in the store. 

### Purposes

- To eliminate N+1 query patterns that multiply database round trips.
- To reduce the volume of data transferred from the database.
- To reduce memory usage in the application (fewer hydrated objects).
- To reduce serialisation cost for JSON responses.
- To lower database CPU and I/O load.

### Sub-Feature 1.1: Eliminating N+1 Queries

**❌ N+1 pattern (bad):**
```js
// 1 query to fetch posts
const posts = await Post.find({});

// N queries to fetch the author for each post
for (const post of posts) {
  post.author = await User.findById(post.authorId);  // N queries!
}
// Total: 1 + N queries
```

**✅ Eager loading (good):**
```js
// 1 query with JOIN
const posts = await Post.find({}).populate('author');
// Total: 2 queries (posts + authors in one IN query)
```

**✅ Explicit JOIN (good):**
```js
const posts = await db.query(`
  SELECT p.id, p.title, u.name AS author_name
  FROM posts p
  JOIN users u ON u.id = p.author_id
`);
// Total: 1 query
```

| Pattern | Queries | Use Case |
|---------|---------|----------|
| N+1 | 1 + N | ❌ Never use. |
| Eager loading (ORM) | 2 | ✅ Most ORMs. |
| JOIN | 1 | ✅ Raw SQL, query builders. |
| `IN` clause | 2 | ✅ When JOIN is impractical. |

**Rules:**
- Enable query logging in development to detect N+1 patterns. 
- Use the ORM's eager-loading mechanism (`populate`, `include`, `with`). 
- For raw SQL, use a single JOIN or an `IN` clause with a subquery. 
- Monitor query counts per request — a request should issue a constant number of queries, not one per item. 

### Sub-Feature 1.2: Selecting Only Required Fields

**❌ Over-fetching (bad):**
```js
const users = await User.find({});  // Fetches all columns
```

**✅ Projection (good):**
```js
const users = await User.find({}, 'id name email');  // Mongoose
const users = await db.query('SELECT id, name, email FROM users');  // SQL
const users = await prisma.user.findMany({ select: { id: true, name: true } });
```

| Approach | Data Transferred | Memory | Serialisation |
|----------|-----------------|--------|---------------|
| `SELECT *` | Full row | High | Slow |
| `SELECT id, name` | Minimal | Low | Fast |

**Rules:**
- Never use `SELECT *` in production queries. 
- Use the ORM's `select` or projection option. 
- Use `.lean()` in Mongoose for read-only queries to skip document hydration. 
- Exclude large text/blob columns unless explicitly needed. 

### Annotated Code Example

```js
// query-optimization.js
const express = require('express');
const app = express();

// ❌ N+1 + over-fetching
app.get('/api/posts/bad', async (req, res) => {
  const posts = await db.query('SELECT * FROM posts');  // 1 query
  for (const post of posts.rows) {
    const author = await db.query('SELECT * FROM users WHERE id = $1', [post.author_id]);  // N queries
    post.author = author.rows[0];
  }
  res.json(posts.rows);  // Includes all columns
});

// ✅ JOIN + projection
app.get('/api/posts/good', async (req, res) => {
  const result = await db.query(`
    SELECT
      p.id,
      p.title,
      p.created_at,
      u.id AS author_id,
      u.name AS author_name
    FROM posts p
    JOIN users u ON u.id = p.author_id
    ORDER BY p.created_at DESC
    LIMIT 50
  `);
  res.json(result.rows);  // Only required fields
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for the bad endpoint — 50 posts):**
```
51 queries executed
Total response time: 850ms
Payload size: 145 KB
```

**Expected Output (for the good endpoint — 50 posts):**
```
1 query executed
Total response time: 12ms
Payload size: 32 KB
```

**Why this output:** The bad endpoint issues 51 queries (1 for posts + 50 for authors) and sends every column. The good endpoint issues 1 query with a JOIN and sends only the required fields. The result is a 70× improvement in response time and a 78% reduction in payload size. 

### Real-World Cases

- **Social media feeds:** Fetching posts with author info in a single JOIN instead of N+1.
- **E-commerce:** Fetching products with categories and prices in one query.
- **Analytics dashboards:** Projecting only the columns needed for charts.

---

## Core Concept 2: Indexes

### Definitions

**Core Definition:** A database index is a data structure that speeds up data retrieval at the cost of additional storage and slower writes. Primary, compound, and partial indexes each serve different query patterns.

**Technical Definition:** A **primary index** is automatically created on the primary key and uniquely identifies each row. A **compound (composite) index** covers multiple columns and speeds up queries that filter or sort by those columns in the defined order — column order matters, as the index can only be used efficiently if the query's leading columns match the index's leading columns. A **partial index** includes only a subset of rows (defined by a `WHERE` clause) and is smaller and faster for queries that consistently filter on that subset. A **covering index** includes all columns needed by a query, allowing the database to answer the query from the index alone without touching the table.

**Beginner-Friendly Explanation:** An index is like the index at the back of a textbook. Without it, you would read every page to find a topic. With it, you jump directly to the right page. A compound index is an index for "last name, then first name" — useful if you search by last name, or by last name and first name, but not by first name alone. A partial index is an index that only covers part of the book — say, only the recipes — so it is smaller and faster for that specific section.

### Purposes

- To speed up `WHERE` clause filtering, `JOIN` conditions, and `ORDER BY` sorting.
- To enforce uniqueness constraints.
- To reduce the number of rows scanned (sequential scan → index scan).
- To enable covering indexes that avoid table lookups.

### Sub-Feature 2.1: Primary Index

**Definition:** The primary index is automatically created on the primary key column(s) and enforces uniqueness. 

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,        -- Primary index created automatically
  email VARCHAR(255) UNIQUE     -- Unique index created automatically
);
```

### Sub-Feature 2.2: Compound Index

**Definition:** A compound index covers multiple columns; the order of columns matters. 

```sql
-- Compound index on (status, created_at)
CREATE INDEX idx_orders_status_created
ON orders (status, created_at DESC);

-- ✅ Uses the index (leading column matches)
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC;

-- ❌ Does not use the index efficiently (leading column missing)
SELECT * FROM orders WHERE created_at > '2026-01-01';
```

**Rule:** The leftmost prefix rule — a compound index on `(a, b, c)` can be used for queries on `(a)`, `(a, b)`, and `(a, b, c)`, but not `(b)`, `(c)`, or `(b, c)`. 

### Sub-Feature 2.3: Partial Index

**Definition:** A partial index includes only rows matching a `WHERE` clause, making it smaller and faster for targeted queries. 

```sql
-- Partial index on active users only
CREATE INDEX idx_users_active_email
ON users (email)
WHERE is_active = true;

-- Small, fast index; queries on active users use it
SELECT * FROM users WHERE is_active = true AND email = 'alice@test.com';
```

### Sub-Feature 2.4: Covering Index

**Definition:** A covering index includes all columns a query needs, so the database can answer from the index alone. 

```sql
-- Covering index for the query below
CREATE INDEX idx_orders_covering
ON orders (user_id, status) INCLUDE (total, created_at);

-- Query satisfied entirely by the index
SELECT total, created_at FROM orders WHERE user_id = 42 AND status = 'pending';
```

### Annotated Code Example

```js
// indexes.js
const { Pool } = require('pg');
const pool = new Pool();

async function createIndexes() {
  // Primary index (automatic with PRIMARY KEY)
  await pool.query(`
    CREATE TABLE IF NOT EXISTS orders (
      id SERIAL PRIMARY KEY,
      user_id INTEGER NOT NULL,
      status VARCHAR(20) NOT NULL,
      total DECIMAL(10,2) NOT NULL,
      created_at TIMESTAMPTZ DEFAULT NOW()
    )
  `);

  // Compound index — for filtering by status and sorting by date
  await pool.query(`
    CREATE INDEX IF NOT EXISTS idx_orders_status_created
    ON orders (status, created_at DESC)
  `);

  // Partial index — only pending orders (small, fast)
  await pool.query(`
    CREATE INDEX IF NOT EXISTS idx_orders_pending
    ON orders (user_id, created_at DESC)
    WHERE status = 'pending'
  `);

  // Covering index — includes all columns for a common query
  await pool.query(`
    CREATE INDEX IF NOT EXISTS idx_orders_covering
    ON orders (user_id, status) INCLUDE (total, created_at)
  `);

  console.log('Indexes created');
}

createIndexes().then(() => process.exit(0));
```

**Expected Output (query performance comparison):**
```
Without index:  Seq Scan on orders (cost=0.00..25000.00 rows=1000000) (actual time=0.015..450.000 ms)
With compound index: Index Scan using idx_orders_status_created (cost=0.42..1250.00 rows=50000) (actual time=0.020..12.000 ms)
```

**Why this output:** Without an index, the database scans all 1 million rows (450ms). With the compound index on `(status, created_at DESC)`, it jumps directly to the matching rows (12ms) — a 37× improvement. 

### Real-World Cases

- **E-commerce:** Compound index on `(status, created_at)` for order filtering.
- **SaaS:** Partial index on active subscriptions only.
- **Analytics:** Covering indexes for dashboard queries.

---

## Core Concept 3: Query Profiling

### Definitions

**Core Definition:** Query profiling uses the database's `EXPLAIN` command to reveal the query plan — how the database intends to execute a query — and `EXPLAIN ANALYZE` to execute the query and report actual timings and row counts.

**Technical Definition:** `EXPLAIN` shows the estimated cost of each step in a query plan (sequential scan, index scan, hash join, nested loop, sort). `EXPLAIN ANALYZE` executes the query and reports actual times, rows, and loops, revealing where the time is spent. Key things to look for: **Seq Scan** on a large table (missing index), **Nested Loop** with high loop counts (N+1-like behaviour), **Sort** with high memory usage (missing index on the sort column), and **Rows Removed by Filter** (index not selective enough). 

**Beginner-Friendly Explanation:** `EXPLAIN` is like a doctor's X-ray for your query. It shows exactly how the database plans to find your data — whether it will scan the whole table, use an index, or join in an inefficient way. `EXPLAIN ANALYZE` actually runs the query and tells you how long each step took. 

### Purposes

- To identify missing indexes (Seq Scan on large tables).
- To detect inefficient joins (Nested Loop with high loop counts).
- To find sorts that spill to disk (missing index on ORDER BY column).
- To measure the actual impact of query rewrites and index additions.
- To validate that the query planner is using the indexes you created.

### Syntax Rules and Structure

```sql
-- Estimated plan (no execution)
EXPLAIN SELECT * FROM orders WHERE status = 'pending';

-- Actual plan (executes the query)
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- With buffer usage
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders WHERE status = 'pending';

-- JSON format (for programmatic analysis)
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM orders WHERE status = 'pending';
```

| Plan Node | Meaning | Action |
|-----------|---------|--------|
| `Seq Scan` | Full table scan | Add an index. |
| `Index Scan` | Uses an index | Good. |
| `Index Only Scan` | Uses a covering index | Excellent. |
| `Nested Loop` | For each row in outer, scan inner | Check for missing index on inner. |
| `Hash Join` | Builds hash table | Good for large joins. |
| `Sort` | Sorts rows | Add index on sort column. |

**Rules:**
- Run `EXPLAIN ANALYZE` on every slow query. 
- Look for `Seq Scan` on tables with more than a few thousand rows. 
- Check `Rows Removed by Filter` — high values indicate a non-selective index. 
- Use `BUFFERS` to see cache hit ratios (high shared reads = cache misses). 
- Re-run `EXPLAIN ANALYZE` after adding indexes to confirm improvement. 

### Annotated Code Example

```js
// query-profiling.js
const { Pool } = require('pg');
const pool = new Pool();

async function profileQuery(sql, params = []) {
  // Run EXPLAIN ANALYZE
  const explain = await pool.query(`EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) ${sql}`, params);
  const plan = explain.rows[0]['QUERY PLAN'][0];

  console.log('Execution time:', plan['Execution Time'], 'ms');
  console.log('Planning time:', plan['Planning Time'], 'ms');

  // Recursively log plan nodes
  function logNode(node, depth = 0) {
    const indent = '  '.repeat(depth);
    console.log(`${indent}${node['Node Type']} (cost=${node['Total Cost']}, rows=${node['Actual Rows']}, time=${node['Actual Total Time']}ms)`);
    if (node['Plans']) {
      node['Plans'].forEach(child => logNode(child, depth + 1));
    }
  }

  logNode(plan.Plan);
}

// Profile a slow query
profileQuery('SELECT * FROM orders WHERE status = $1 ORDER BY created_at DESC', ['pending'])
  .then(() => process.exit(0));
```

**Expected Output (before index):**
```
Execution time: 452.3 ms
Planning time: 0.5 ms
Seq Scan on orders (cost=0.00..25000.00, rows=50000, time=450.2ms)
  Filter: (status = 'pending')
  Rows Removed by Filter: 950000
Sort (cost=25000.00..25125.00, rows=50000, time=452.0ms)
  Sort Key: created_at DESC
  Sort Method: external merge  Disk: 8192kB
```

**Expected Output (after index):**
```
Execution time: 12.1 ms
Planning time: 0.6 ms
Index Scan using idx_orders_status_created on orders (cost=0.42..1250.00, rows=50000, time=11.8ms)
```

**Why this output:** Before the index, the query performs a sequential scan (450ms) and filters out 950,000 rows. After adding the compound index on `(status, created_at DESC)`, the query uses an index scan (12ms), avoiding the full table scan and the disk-based sort. 

### Real-World Cases

- **Slow API endpoints:** Profiling the query behind a 2-second response.
- **Index validation:** Confirming that a newly created index is being used.
- **Query rewriting:** Comparing plans before and after rewriting a complex JOIN.

---

## Core Concept 4: Connection Pool Sizing

### Definitions

**Core Definition:** Connection pool sizing is the practice of configuring the minimum and maximum number of database connections per application instance, balancing application concurrency against the database server's connection limits and memory.

**Technical Definition:** A connection pool maintains a set of open database connections that are reused across requests. The pool has a **minimum size** (connections kept open even when idle), a **maximum size** (connections opened under load), an **idle timeout** (when idle connections above the minimum are closed), and a **connection timeout** (how long a request waits for a connection before failing). Each database connection consumes memory on the database server (e.g., PostgreSQL allocates ~10 MB per connection), so the total number of connections across all application instances must not exceed the database's `max_connections`. The recommended formula: `connections = ((core_count * 2) + effective_spindle_count)`. For a 4-core database with SSD storage, this is roughly `(4 * 2) + 1 = 9` connections per instance. 

**Beginner-Friendly Explanation:** A connection pool is like a taxi stand outside a train station. The stand has a few taxis waiting (minimum), and more taxis arrive when there is demand (maximum). If too many taxis arrive, they block the road (database memory exhausted). If too few, passengers wait too long (requests queue). The right number depends on how many passengers (requests) and how big the road (database resources) is. 

### Purposes

- To reuse connections and avoid the cost of establishing a new one per request.
- To limit the number of concurrent connections to the database.
- To queue requests when the pool is exhausted rather than overwhelming the database.
- To balance application concurrency against database memory and CPU.

### Syntax Rules and Structure

**PostgreSQL (`pg`):**
```js
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                    // Maximum connections
  min: 5,                     // Minimum idle connections
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 2000,  // Fail if no connection in 2s
  maxUses: 7500               // Close connection after 7500 queries
});
```

**Mongoose (MongoDB):**
```js
mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 50,
  minPoolSize: 5,
  maxIdleTimeMS: 30000,
  waitQueueTimeoutMS: 2000
});
```

| Option | `pg` | Mongoose | Purpose |
|--------|------|----------|---------|
| Max connections | `max` | `maxPoolSize` | Upper limit. |
| Min connections | `min` | `minPoolSize` | Keep-alive connections. |
| Idle timeout | `idleTimeoutMillis` | `maxIdleTimeMS` | Close idle connections. |
| Connection timeout | `connectionTimeoutMillis` | `waitQueueTimeoutMS` | Queue wait limit. |

**Rules:**
- Calculate pool size from database hardware: `(cores * 2) + spindles`. 
- Total connections across all instances must not exceed `max_connections`. 
- Set `connectionTimeoutMillis` to fail fast rather than hang. 
- Monitor pool metrics: active, idle, waiting connections. 
- Use PgBouncer for high-concurrency deployments to multiplex connections. 

### Annotated Code Example

```js
// connection-pool.js
const { Pool } = require('pg');

// Calculate pool size for a 4-core database
const DB_CORES = 4;
const DB_SPINDLES = 1;  // SSD
const INSTANCES = 4;    // Number of app instances

const poolSizePerInstance = Math.floor(((DB_CORES * 2) + DB_SPINDLES) / INSTANCES);

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: poolSizePerInstance,      // 9 / 4 = 2 connections per instance
  min: 1,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
  maxUses: 7500
});

// Monitor pool events
pool.on('connect', () => console.log('New connection established'));
pool.on('acquire', () => console.log('Connection acquired'));
pool.on('release', () => console.log('Connection released'));
pool.on('remove', () => console.log('Connection removed'));

// Expose pool metrics
function getPoolMetrics() {
  return {
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount
  };
}

console.log('Pool size per instance:', poolSizePerInstance);
```

**Expected Output:**
```
Pool size per instance: 2
New connection established
Connection acquired
Connection released
Pool metrics: { total: 2, idle: 2, waiting: 0 }
```

**Why this output:** The pool size is calculated from the database's hardware: 4 cores × 2 + 1 spindle = 9 total connections, divided by 4 app instances = 2 connections per instance. Each instance maintains 2 connections, and the total (8) stays below the database's `max_connections` (100 by default). 

### Real-World Cases

- **Kubernetes deployments:** Each pod gets a pool size calculated from the total database capacity divided by the expected pod count.
- **Serverless functions:** Use a connection proxy (PgBouncer, RDS Proxy) because each function invocation creates a new connection.
- **High-concurrency APIs:** Monitor `waitingCount` — if it grows, increase pool size or reduce query time.

---

## Core Concept 5: Database Caching Layers

### Definitions

**Core Definition:** A database caching layer stores the results of frequent or expensive queries in a fast in-memory store (Redis or Memcached), serving subsequent requests from the cache instead of the database.

**Technical Definition:** Redis and Memcached are in-memory key-value stores that operate orders of magnitude faster than disk-based databases. The typical pattern is **cache-aside**: the application checks the cache first; on a miss, it queries the database, stores the result in the cache with a TTL, and returns it. On a hit, it returns the cached value without touching the database. **Write-through** and **write-behind** patterns update the cache alongside the database. **Cache invalidation** — removing or updating cached entries when the underlying data changes — is the hardest part of caching. 

**Beginner-Friendly Explanation:** Redis is like a whiteboard at the front of the room. Instead of walking to the filing cabinet (database) every time you need a fact, you write the most-used facts on the whiteboard. When someone asks, you read from the whiteboard (fast). When the fact changes, you erase the whiteboard and write the new version. 

### Purposes

- To offload heavy read operations from the database.
- To reduce response latency for frequently accessed data.
- To absorb traffic spikes without overwhelming the database.
- To reduce database CPU and I/O load.

### Syntax Rules and Structure

**Cache-aside with Redis:**
```js
const Redis = require('ioredis');
const redis = new Redis();

async function getProducts() {
  const cacheKey = 'products:all';

  // 1. Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // 2. Cache miss — query database
  const products = await db.query('SELECT * FROM products');

  // 3. Store in cache with TTL
  await redis.setex(cacheKey, 300, JSON.stringify(products));  // 5-minute TTL

  return products;
}

// Invalidate on write
async function createProduct(data) {
  const product = await db.query('INSERT INTO products ...');
  await redis.del('products:all');  // Invalidate cache
  return product;
}
```

| Pattern | Description | Use Case |
|---------|-------------|----------|
| Cache-aside | App checks cache, then DB. | Most common. |
| Write-through | Write to cache and DB. | Consistent reads. |
| Write-behind | Write to cache, async to DB. | High write throughput. |
| Read-through | Cache handles DB reads. | Transparent caching. |

**Rules:**
- Always set a TTL — stale data is worse than no cache. 
- Invalidate on write — delete or update the cache when data changes. 
- Use cache keys that include all query parameters. 
- Monitor cache hit ratio — aim for > 80% for cached endpoints. 
- Use Redis for distributed deployments; Memcached for simple key-value caching. 

### Annotated Code Example

```js
// database-caching.js
const express = require('express');
const Redis = require('ioredis');
const { Pool } = require('pg');

const app = express();
const redis = new Redis();
const pool = new Pool();

// Cache-aside pattern
app.get('/api/products', async (req, res) => {
  const cacheKey = `products:${req.query.category || 'all'}`;

  // 1. Try cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    res.set('X-Cache', 'HIT');
    return res.json(JSON.parse(cached));
  }

  // 2. Cache miss — query database
  const result = await pool.query(
    'SELECT id, name, price FROM products WHERE category = $1',
    [req.query.category || 'electronics']
  );

  // 3. Store in cache
  await redis.setex(cacheKey, 300, JSON.stringify(result.rows));  // 5 min

  res.set('X-Cache', 'MISS');
  res.json(result.rows);
});

// Invalidate on write
app.post('/api/products', async (req, res) => {
  const result = await pool.query(
    'INSERT INTO products (name, price, category) VALUES ($1, $2, $3) RETURNING *',
    [req.body.name, req.body.price, req.body.category]
  );

  // Invalidate all product caches
  const keys = await redis.keys('products:*');
  if (keys.length > 0) await redis.del(...keys);

  res.status(201).json(result.rows[0]);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (first request):**
```
X-Cache: MISS
Response time: 45ms
```

**Expected Output (subsequent requests within 5 minutes):**
```
X-Cache: HIT
Response time: 3ms
```

**Why this output:** The first request misses the cache, queries the database (45ms), and stores the result in Redis. Subsequent requests hit the cache (3ms) — a 15× improvement. When a new product is created, the cache is invalidated, so the next request fetches fresh data from the database. 

### Real-World Cases

- **E-commerce:** Caching product listings and category pages.
- **Social media:** Caching user profiles and feed previews.
- **Analytics dashboards:** Caching expensive aggregation queries for 1–5 minutes.
- **API rate limit counters:** Redis for distributed rate limiting.

---

## Core Concept 6: Read/Write Splitting

### Definitions

**Core Definition:** Read/write splitting routes write operations to a primary database and read operations to one or more read replicas, distributing the load and improving read scalability.

**Technical Definition:** In a primary-replica topology, the primary database handles all writes (INSERT, UPDATE, DELETE) and propagates changes to replicas via replication (streaming or logical). Replicas handle read-only queries (SELECT). The application must route queries based on their type: writes and read-after-write queries go to the primary; reads that can tolerate slight staleness go to replicas. **Replication lag** is the delay between a write on the primary and its visibility on replicas; it can range from milliseconds to seconds. The application must handle this by reading from the primary after a write (read-your-writes consistency) or by accepting eventual consistency. 

**Beginner-Friendly Explanation:** Imagine a library with one checkout desk (primary) and several reading rooms (replicas). All new books are processed at the checkout desk. Readers can read in any reading room, but a book processed at the checkout desk might take a moment to appear in the reading rooms. Read/write splitting means sending all checkouts to the desk and all reading to the rooms, so the desk is not overwhelmed by readers. 

### Purposes

- To scale read capacity horizontally by adding replicas.
- To reduce load on the primary database.
- To improve read performance by serving reads from nearby replicas.
- To increase availability — if the primary fails, a replica can be promoted.

### Syntax Rules and Structure

**Drizzle ORM with read replicas:**
```js
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const primaryPool = new Pool({ connectionString: process.env.DATABASE_URL });
const replicaPool = new Pool({ connectionString: process.env.REPLICA_URL });

const db = drizzle(primaryPool, { schema });
const replicaDb = drizzle(replicaPool, { schema });

// Writes → primary
await db.insert(users).values({ name: 'Alice' });

// Reads → replica
const users = await replicaDb.select().from(users);
```

**Prisma with read replicas:**
```js
const prisma = new PrismaClient({
  datasources: {
    db: { url: process.env.DATABASE_URL }  // Primary
  },
  replicas: [
    { url: process.env.REPLICA_URL }       // Read replica
  ]
});

// Prisma routes reads to replicas and writes to primary automatically
const users = await prisma.user.findMany();  // → replica
await prisma.user.create({ data: { name: 'Alice' } });  // → primary
```

**Manual routing:**
```js
const primaryPool = new Pool({ connectionString: process.env.DATABASE_URL });
const replicaPool = new Pool({ connectionString: process.env.REPLICA_URL });

function query(sql, params, { readOnly = false } = {}) {
  return readOnly
    ? replicaPool.query(sql, params)
    : primaryPool.query(sql, params);
}

// Usage
await query('SELECT * FROM users', [], { readOnly: true });   // → replica
await query('INSERT INTO users ...', params);                  // → primary
```

| Query Type | Target | Consistency |
|-----------|--------|-------------|
| INSERT/UPDATE/DELETE | Primary | Strong. |
| SELECT (tolerant) | Replica | Eventual. |
| SELECT after write | Primary | Strong (read-your-writes). |

**Rules:**
- Route all writes to the primary. 
- Route reads that can tolerate staleness to replicas. 
- Route read-after-write queries to the primary to avoid replication lag anomalies. 
- Monitor replication lag — if it exceeds a threshold, route reads to the primary. 
- Use a load balancer or proxy (PgBouncer, ProxySQL) to manage replica routing. 

### Annotated Code Example

```js
// read-write-splitting.js
const express = require('express');
const { Pool } = require('pg');

const app = express();

const primaryPool = new Pool({ connectionString: process.env.DATABASE_URL });
const replicaPool = new Pool({ connectionString: process.env.REPLICA_URL });

// Read from replica
app.get('/api/users', async (req, res) => {
  const result = await replicaPool.query('SELECT id, name, email FROM users');
  res.json(result.rows);
});

// Write to primary
app.post('/api/users', async (req, res) => {
  const result = await primaryPool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [req.body.name, req.body.email]
  );
  res.status(201).json(result.rows[0]);
});

// Read-after-write → primary (avoid replication lag)
app.get('/api/users/:id/after-write', async (req, res) => {
  const result = await primaryPool.query(
    'SELECT * FROM users WHERE id = $1',
    [req.params.id]
  );
  res.json(result.rows[0]);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users`):**
```
Response from replica: [{ id: 1, name: 'Alice' }]
```

**Expected Output (for `POST /api/users`):**
```
Response from primary: { id: 2, name: 'Bob' }
```

**Why this output:** Read requests are routed to `replicaPool`, which can serve them without loading the primary. Write requests are routed to `primaryPool`. The read-after-write endpoint uses the primary to ensure the newly written data is visible immediately, avoiding the replication lag anomaly. 

### Real-World Cases

- **E-commerce:** Product listings read from replicas; orders written to the primary.
- **Social media:** Feed reads from replicas; posts and likes written to the primary.
- **Analytics:** Dashboard queries read from replicas; data ingestion writes to the primary.

---

## References

- Prisma Read Replicas — https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/read-replicas
- Drizzle ORM Read Replicas — https://orm.drizzle.team/docs/read-replicas
- PostgreSQL Connection Pool Sizing — https://wiki.postgresql.org/wiki/Number_Of_Database_Connections
- PgBouncer Documentation — https://www.pgbouncer.org/
- Redis Caching Patterns — https://redis.io/docs/manual/patterns/
- PostgreSQL EXPLAIN Documentation — https://www.postgresql.org/docs/current/using-explain.html
- Use The Index, Luke — SQL Indexing Guide — https://use-the-index-luke.com/
- N+1 Query Problem — https://stackoverflow.com/questions/97197/what-is-the-n1-selects-issue
- Mongoose Connection Pooling — https://mongoosejs.com/docs/connections.html#connection-pools
- node-postgres Pool Documentation — https://node-postgres.com/apis/pool
- MySQL EXPLAIN Output — https://dev.mysql.com/doc/refman/8.0/en/explain-output.html
- Connection Pool Sizing Formula — https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing