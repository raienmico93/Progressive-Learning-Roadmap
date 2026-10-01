# NoSQL Databases — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NoSQL databases are non-relational data stores designed for flexible schemas, horizontal scalability, and high-performance access patterns, categorised into document stores (MongoDB), key-value stores (Redis), and others.

**Technical Definition:** NoSQL (Not Only SQL) databases encompass a broad class of data management systems that diverge from the traditional relational model. They prioritise availability, partition tolerance, and horizontal scaling over strict ACID guarantees in some configurations. In the Node.js ecosystem, MongoDB (document store) and Redis (in-memory key-value store) dominate. MongoDB uses BSON documents with flexible schemas, while Redis provides sub-millisecond access to strings, hashes, lists, sets, sorted sets, streams, and more.

**Beginner-Friendly Explanation:** A relational database is like a strict filing cabinet with predefined folders and forms. A NoSQL database is like a flexible backpack — you can put in whatever shape of data you need, change it whenever you want, and it's designed to scale across many servers. MongoDB stores data as flexible documents (like JSON), while Redis stores everything in memory for lightning-fast access.

### Key Characteristics

- **MongoDB:** Document-oriented, schema-flexible, supports rich queries, aggregation pipelines, and change streams.
- **Redis:** In-memory, key-value store with rich data structures (Hashes, Sorted Sets, Pub/Sub), sub-millisecond latency, and TTL support.
- **Drivers vs. ODMs:** The native MongoDB driver provides direct access; Mongoose adds schema validation, middleware hooks, and population.
- **Embedding vs. Referencing:** MongoDB schema design follows access patterns — embed data accessed together, reference data accessed independently.
- **Caching patterns:** Cache-aside (lazy loading) and write-through are the two dominant strategies for integrating Redis with databases.
- **TTL and eviction:** Redis keys can have Time-To-Live (TTL) for automatic expiry; eviction policies (volatile-lru, allkeys-lru) manage memory when full.
- **Distributed locks:** The Redlock algorithm enables fault-tolerant mutual exclusion across multiple Redis instances.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **MongoDB and Redis installed:** Local instances or cloud services (MongoDB Atlas, Redis Cloud).
- **Basic JavaScript/TypeScript:** Promises, async/await, and event handling.
- **Understanding of JSON:** For document modelling.

### Related Programming Areas

- **Relational Databases:** SQL databases and ORMs.
- **Caching:** Application-level caching strategies.
- **Real-time Applications:** WebSockets, Pub/Sub, and change streams.
- **Session Management:** Distributed session storage.
- **Event-Driven Architecture:** Change streams and Redis Pub/Sub.

### Core Concepts

1. **MongoDB** — driver connectivity, Mongoose ODM, schema validation, middleware hooks, change streams.
2. **Redis** — connection management, complex data structures (Hashes, Sorted Sets, Pub/Sub).
3. **Document-Oriented Modeling** — embedded vs. referenced documents, aggregation pipelines.
4. **Key-Value Modeling** — TTL management, cache eviction, session store integration.
5. **Caching** — cache-aside, write-through, distributed locks (Redlock).

---

## Core Concept 1: MongoDB

### Sub-Feature 1.1: Driver Connectivity (`mongodb`) vs. Object Data Modeling (Mongoose)

#### Definitions

**Core Definition:** The native MongoDB Node.js driver provides direct, unwrapped access to MongoDB, while Mongoose is an Object Data Modeling (ODM) library built on top of the driver that adds schemas, validation, middleware, and relationships.

**Technical Definition:** The `mongodb` package is the official MongoDB driver for Node.js. It mirrors the MongoDB shell API and gives direct access to aggregation, transactions, and change streams. Mongoose is an ODM that enforces application-level schema validation, provides type coercion, middleware (pre/post hooks), default values, and `populate()` for resolving references. Mongoose is necessary to leverage features like model middleware, hooks, and automated schema relationships.

**Beginner-Friendly Explanation:** The native driver is like talking directly to the database — fast and flexible but you have to manage everything yourself. Mongoose is like having a personal assistant — it checks your data before saving, runs automatic tasks (like hashing passwords), and helps you connect related data. The assistant adds overhead but saves you from mistakes.

#### Purposes

- To provide flexible, performant access to MongoDB (native driver).
- To enforce data consistency and structure (Mongoose).
- To enable middleware for cross-cutting concerns like password hashing (Mongoose).
- To simplify relationship handling with `populate()` (Mongoose).

#### Syntax Rules and Structure

**Native driver:**
```javascript
const { MongoClient } = require('mongodb');
const client = new MongoClient(process.env.MONGO_URI);
await client.connect();
const db = client.db('shop');
await db.collection('products').insertOne({ name: 'Widget', price: 9.99 });
```

**Mongoose:**
```javascript
const mongoose = require('mongoose');
const productSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  price: { type: Number, required: true, min: 0 },
  slug: { type: String },
  createdAt: { type: Date, default: Date.now },
});
productSchema.pre('save', function (next) {
  this.slug = this.name.toLowerCase().replace(/\s+/g, '-');
  next();
});
const Product = mongoose.model('Product', productSchema);
```

| Aspect | Native Driver | Mongoose |
|--------|--------------|----------|
| Schema | Schemaless | Strict schema |
| Validation | Manual | Built-in |
| Middleware | None | Pre/post hooks |
| Relationships | `$lookup` aggregation | `populate()` |
| Performance | Faster | Overhead from hydration |
| Use case | Complex pipelines, bulk writes | Structured applications |

**Constraints and Limitations:**
- Mongoose adds overhead; use `.lean()` to return plain objects and skip document hydration, which is 30–50% faster.
- `pre` and `post` hooks are not called for update operations executed directly on the database (e.g., `updateOne`).
- The native driver is recommended for bulk writes, aggregation pipelines, and change streams.

#### Annotated Code Example

```javascript
// mongodb-vs-mongoose.js
// Native driver
const { MongoClient } = require('mongodb');
const client = new MongoClient(process.env.MONGO_URI);

async function nativeExample() {
  await client.connect();
  const db = client.db('shop');

  // No validation — any shape accepted
  await db.collection('products').insertOne({ name: 'Widget' });
  await db.collection('products').insertOne({ title: 'Gadget', price: 'ten' }); // inconsistent!
  console.log('Native: Documents inserted without validation.');
  await client.close();
}

// Mongoose
const mongoose = require('mongoose');
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true, min: 0 },
});
const Product = mongoose.model('Product', productSchema);

async function mongooseExample() {
  await mongoose.connect(process.env.MONGO_URI);

  try {
    const p = new Product({ name: 'Widget', price: 9.99 });
    await p.save(); // Validation runs
    console.log('Mongoose: Product saved with validation.');

    // This will fail validation
    const invalid = new Product({ name: 'Gadget', price: -5 });
    await invalid.save();
  } catch (err) {
    console.error('Mongoose validation error:', err.message);
  }

  await mongoose.disconnect();
}

nativeExample().then(() => mongooseExample());
```

**Expected Output:**
```
Native: Documents inserted without validation.
Mongoose: Product saved with validation.
Mongoose validation error: Product validation failed: price: Path `price` (-5) is less than minimum allowed value (0).
```

**Why this output:** The native driver inserts any shape without complaint. Mongoose validates the schema, rejecting the negative price. This demonstrates the core trade-off: flexibility vs. consistency.

#### Real-World Cases

- **Native driver:** High-throughput data ingestion, complex aggregation pipelines, change stream consumers.
- **Mongoose:** User management, content management systems, applications requiring data integrity and middleware.

---

### Sub-Feature 1.2: Schema Validation, Middleware Hooks, and Change Streams

#### Definitions

**Core Definition:** Mongoose provides application-level schema validation and middleware (pre/post hooks) for document lifecycle events, while MongoDB change streams enable real-time notification of data changes at the collection, database, or deployment level.

**Technical Definition:** Mongoose middleware (also called pre and post hooks) are functions passed control during execution of asynchronous functions. All middleware types support pre and post hooks. Change streams are a MongoDB feature that allows applications to subscribe to data changes in real time. They use the aggregation framework's `$changeStream` stage and return a cursor that emits change events for inserts, updates, replacements, and deletes.

**Beginner-Friendly Explanation:** Middleware hooks are like automatic actions that run before or after you save a document — for example, hashing a password before storing it. Change streams are like a live notification system: whenever data changes in the database, your application gets an event immediately, without polling.

#### Purposes

- To automate data transformations and validations (middleware hooks).
- To enforce business rules consistently across the application.
- To react to data changes in real time (change streams).
- To build event-driven architectures and live dashboards.

#### Syntax Rules and Structure

**Mongoose middleware:**
```javascript
userSchema.pre('save', async function (next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 12);
  }
  next();
});

userSchema.post('save', function (doc) {
  console.log('User saved:', doc._id);
});
```

**Change streams (native driver):**
```javascript
const changeStream = db.collection('orders').watch([
  { $match: { 'fullDocument.status': 'shipped' } },
]);

changeStream.on('change', (event) => {
  console.log('Order changed:', event.fullDocument);
});
```

| Middleware Type | Trigger | Use Case |
|-----------------|---------|----------|
| `pre('save')` | Before save | Password hashing, slug generation |
| `post('save')` | After save | Notifications, logging |
| `pre('remove')` | Before delete | Cleanup related documents |
| `pre(/^find/)` | Before find | Soft-delete filtering |

**Constraints and Limitations:**
- Change streams require a replica set or sharded cluster (not standalone MongoDB).
- Middleware hooks run in the Node.js process, not the database; direct database updates bypass them.
- `pre('save')` hooks fire for `create()` but not for `insertMany()` by default.

#### Annotated Code Example

```javascript
// mongoose-middleware-change-streams.js
const mongoose = require('mongoose');
const { MongoClient } = require('mongodb');

// Mongoose middleware
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  slug: String,
});

userSchema.pre('save', function (next) {
  this.slug = this.name.toLowerCase().replace(/\s+/g, '-');
  console.log('Pre-save: slug generated as', this.slug);
  next();
});

userSchema.post('save', function (doc) {
  console.log('Post-save: user saved with ID', doc._id);
});

const User = mongoose.model('User', userSchema);

// Change stream (native driver)
async function watchChanges() {
  const client = new MongoClient(process.env.MONGO_URI);
  await client.connect();
  const db = client.db('test');
  const collection = db.collection('users');

  const changeStream = collection.watch();
  changeStream.on('change', (event) => {
    console.log('Change detected:', event.operationType, event.fullDocument?.name);
  });

  // Trigger a change via Mongoose
  await mongoose.connect(process.env.MONGO_URI);
  await User.create({ name: 'Alice', email: 'alice@example.com', password: 'secret' });

  setTimeout(() => {
    changeStream.close();
    client.close();
    mongoose.disconnect();
  }, 2000);
}

watchChanges();
```

**Expected Output:**
```
Pre-save: slug generated as alice
Post-save: user saved with ID 67890abcdef1234567890abcd
Change detected: insert Alice
```

**Why this output:** The `pre('save')` hook generates a slug from the name. The `post('save')` hook logs the saved document. The change stream (listening on the native driver) detects the insert performed by Mongoose, demonstrating interoperability between the two.

#### Real-World Cases

- **User registration:** Pre-save hook for password hashing; post-save hook for welcome emails.
- **Order processing:** Change stream watching for order status changes to trigger fulfilment.
- **Real-time dashboards:** Change streams feeding live metrics to WebSocket clients.

---

## Core Concept 2: Redis

### Sub-Feature 2.1: Multi-Tenant Connection Management via `ioredis` or `redis` Client

#### Definitions

**Core Definition:** Multi-tenant connection management in Redis involves maintaining separate connections for different tenants, purposes (publisher/subscriber), or configurations (cluster vs. standalone) to ensure isolation and correctness.

**Technical Definition:** `ioredis` is a robust, full-featured Redis client that supports Cluster, Sentinel, Streams, Pipelining, Lua scripting, Pub/Sub, and transparent key prefixing. The `redis` (node-redis) client is the officially recommended library for new projects, supporting hash-field expiration and Redis 8 features. A critical constraint: a client in subscribe mode cannot issue other commands, so a separate connection is required for subscribing versus publishing. For multi-tenant applications, key prefixing (`keyPrefix` option) isolates tenant data within a single Redis instance.

**Beginner-Friendly Explanation:** Redis connections are like phone lines. If you're listening on a call (subscribed to a channel), you can't also make outgoing calls on the same line. So you need a second line for publishing. For multi-tenant applications, you add a tenant prefix to every key so each tenant's data is separate.

#### Purposes

- To isolate tenant data in a shared Redis instance.
- To maintain correct Pub/Sub behaviour with separate publisher and subscriber connections.
- To support cluster and sentinel deployments with automatic failover.
- To manage connection lifecycle in serverless and containerised environments.

#### Syntax Rules and Structure

**ioredis basic connection:**
```javascript
const Redis = require('ioredis');
const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  password: process.env.REDIS_PASSWORD,
  keyPrefix: `tenant:${tenantId}:`, // Multi-tenant isolation
});
```

**Separate Pub/Sub connections:**
```javascript
const pub = new Redis();
const sub = new Redis();

sub.subscribe('notifications', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});

sub.on('message', (channel, message) => {
  console.log(`Received on ${channel}: ${message}`);
});

pub.publish('notifications', 'Hello subscribers!');
```

| Feature | `ioredis` | `node-redis` (redis) |
|---------|-----------|---------------------|
| Cluster | Yes | Yes |
| Sentinel | Yes | Yes |
| Pub/Sub | Yes (binary support) | Yes |
| Lua Scripting | Yes | Yes |
| Key Prefixing | Built-in | Manual |
| Recommendation | Stable, best-effort | New projects |

**Constraints and Limitations:**
- A subscriber connection cannot run other commands; create separate connections for pub/sub and regular operations.
- `ioredis` is in best-effort maintenance; `node-redis` is recommended for new projects.
- Key prefixing in `ioredis` is transparent for most commands but not all (e.g., `KEYS`).

#### Annotated Code Example

```javascript
// redis-multi-tenant.js
const Redis = require('ioredis');

// Tenant-specific connections with prefix isolation
const tenantA = new Redis({ keyPrefix: 'tenant:a:' });
const tenantB = new Redis({ keyPrefix: 'tenant:b:' });

// Separate Pub/Sub connections
const publisher = new Redis();
const subscriber = new Redis();

async function main() {
  // Set values with automatic prefixing
  await tenantA.set('config', JSON.stringify({ theme: 'dark' }));
  await tenantB.set('config', JSON.stringify({ theme: 'light' }));

  const configA = JSON.parse(await tenantA.get('config'));
  const configB = JSON.parse(await tenantB.get('config'));
  console.log('Tenant A config:', configA);
  console.log('Tenant B config:', configB);

  // Pub/Sub with separate connections
  await subscriber.subscribe('events');
  subscriber.on('message', (channel, message) => {
    console.log(`Received on ${channel}: ${message}`);
  });

  await publisher.publish('events', 'Order shipped');

  setTimeout(() => {
    tenantA.disconnect();
    tenantB.disconnect();
    publisher.disconnect();
    subscriber.disconnect();
  }, 1000);
}

main().catch(console.error);
```

**Expected Output:**
```
Tenant A config: { theme: 'dark' }
Tenant B config: { theme: 'light' }
Received on events: Order shipped
```

**Why this output:** The `keyPrefix` option transparently isolates tenant data. `tenantA` writes to `tenant:a:config`; `tenantB` writes to `tenant:b:config`. The separate subscriber connection receives the message published by the publisher. Without the separate subscriber connection, the subscriber could not receive messages while also issuing commands.

#### Real-World Cases

- **Multi-tenant SaaS:** Isolating cache and session data per tenant.
- **Microservices:** Each service uses a prefix for its keys.
- **Real-time applications:** Pub/Sub for WebSocket broadcasts.

---

### Sub-Feature 2.2: Complex Data Structures (Hashes, Sorted Sets, Pub/Sub)

#### Definitions

**Core Definition:** Redis provides rich data structures beyond simple strings: Hashes (field-value maps), Sorted Sets (elements ordered by score), and Pub/Sub (publish-subscribe messaging), each enabling specific application patterns.

**Technical Definition:** Redis Hashes (`HSET`, `HGET`, `HGETALL`) store objects as field-value pairs, ideal for representing entities. Sorted Sets (`ZADD`, `ZRANGE`, `ZRANK`) store elements with a score, enabling leaderboards and priority queues. Pub/Sub (`PUBLISH`, `SUBSCRIBE`) enables real-time messaging between processes. Lists (`LPUSH`, `LRANGE`) provide queues. Sets (`SADD`, `SMEMBERS`) provide unique collections.

**Beginner-Friendly Explanation:** Hashes are like a folder for an object — all the fields of a user in one key. Sorted Sets are like a leaderboard — each player has a score and you can quickly get the top 10. Pub/Sub is like a radio station — publishers broadcast on channels, subscribers tune in.

#### Purposes

- To store object-like data efficiently (Hashes).
- To implement leaderboards, rankings, and priority queues (Sorted Sets).
- To enable real-time messaging and event broadcasting (Pub/Sub).
- To build queues and unique collections (Lists and Sets).

#### Syntax Rules and Structure

**Hashes:**
```javascript
await redis.hset('user:1', 'name', 'Alice', 'email', 'alice@example.com');
const user = await redis.hgetall('user:1');
// { name: 'Alice', email: 'alice@example.com' }
```

**Sorted Sets:**
```javascript
await redis.zadd('leaderboard', 100, 'Alice', 85, 'Bob', 92, 'Charlie');
const top2 = await redis.zrange('leaderboard', 0, 1, 'WITHSCORES');
// ['Charlie', '92', 'Alice', '100'] (descending? No, ascending by default)
const rank = await redis.zrevrank('leaderboard', 'Alice'); // 0 (highest)
```

**Pub/Sub:**
```javascript
await subscriber.subscribe('room:1');
subscriber.on('message', (channel, message) => {
  console.log(`Room ${channel}: ${message}`);
});
await publisher.publish('room:1', 'Hello room!');
```

| Data Structure | Commands | Use Case |
|---------------|----------|----------|
| Hash | `HSET`, `HGET`, `HGETALL` | User profiles, product attributes |
| Sorted Set | `ZADD`, `ZRANGE`, `ZREVRANK` | Leaderboards, priority queues |
| List | `LPUSH`, `LRANGE`, `BRPOP` | Queues, activity feeds |
| Set | `SADD`, `SMEMBERS`, `SISMEMBER` | Unique visitors, tags |
| Pub/Sub | `PUBLISH`, `SUBSCRIBE` | Real-time notifications, chat |

**Constraints and Limitations:**
- Pub/Sub messages are fire-and-forget; if a subscriber is offline, messages are lost. Use Streams for persistence.
- Sorted Set scores are 64-bit floats; precision limits apply.
- Hashes are efficient for small objects; very large hashes (thousands of fields) may impact performance.

#### Annotated Code Example

```javascript
// redis-data-structures.js
const Redis = require('ioredis');
const redis = new Redis();

async function main() {
  // Hash: user profile
  await redis.hset('user:1',
    'name', 'Alice',
    'email', 'alice@example.com',
    'role', 'admin'
  );
  const user = await redis.hgetall('user:1');
  console.log('User:', user);

  // Sorted Set: leaderboard (highest score first)
  await redis.zadd('leaderboard',
    100, 'Alice',
    85, 'Bob',
    92, 'Charlie'
  );
  const top3 = await redis.zrevrange('leaderboard', 0, 2, 'WITHSCORES');
  console.log('Top 3:', top3);

  // Pub/Sub
  const subscriber = new Redis();
  await subscriber.subscribe('notifications');
  subscriber.on('message', (channel, message) => {
    console.log(`[${channel}] ${message}`);
  });

  await redis.publish('notifications', 'Deployment complete');

  setTimeout(() => {
    subscriber.disconnect();
    redis.disconnect();
  }, 1000);
}

main().catch(console.error);
```

**Expected Output:**
```
User: { name: 'Alice', email: 'alice@example.com', role: 'admin' }
Top 3: [ 'Alice', '100', 'Charlie', '92', 'Bob', '85' ]
[notifications] Deployment complete
```

**Why this output:** The Hash stores the user object as field-value pairs. The Sorted Set stores scores; `zrevrange` returns elements in descending order (highest score first). Pub/Sub delivers the message to the subscriber.

#### Real-World Cases

- **Gaming:** Leaderboards with Sorted Sets; player profiles with Hashes.
- **Chat applications:** Pub/Sub for message broadcasting; Lists for message history.
- **E-commerce:** Product recommendations with Sets; shopping carts with Hashes.
- **Real-time analytics:** Sorted Sets for trending topics; Pub/Sub for live dashboards.

---

## Core Concept 3: Document-Oriented Modeling

### Sub-Feature 3.1: Embedded Subdocuments vs. References (Normalisation vs. Denormalisation)

#### Definitions

**Core Definition:** Embedded subdocuments store related data within a single document (denormalisation), while references store related data in separate documents linked by ID (normalisation).

**Technical Definition:** MongoDB schema design follows the principle: "Embed what is read together and is bounded; reference what is accessed independently or is unbounded." Embedding provides atomic writes and single-read access but risks hitting the 16MB document limit. Referencing avoids duplication and supports independent lifecycles but requires `$lookup` (aggregation) or `populate()` (Mongoose), which adds latency.

**Beginner-Friendly Explanation:** Embedding is like putting all the chapters of a book in one file — you get the whole book in one read, but the file can get huge. Referencing is like having separate files for each chapter and an index — smaller files, but you need to look up multiple files to read the whole book.

#### Purposes

- To optimise for common access patterns (embed data read together).
- To avoid duplication and maintain consistency (reference shared data).
- To stay within document size limits (reference unbounded arrays).
- To enable atomic updates on related data (embed).

#### Syntax Rules and Structure

**Embed (one-to-few):**
```javascript
// User with addresses — bounded, always fetched together
const userSchema = new mongoose.Schema({
  name: String,
  addresses: [{
    street: String,
    city: String,
    zipCode: String,
    isDefault: { type: Boolean, default: false },
  }],
});
```

**Reference (one-to-many):**
```javascript
// Product with parts — parts accessed independently, shared across products
const productSchema = new mongoose.Schema({
  name: String,
  parts: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Part' }], // bounded ~10-50
});

// Log messages — unbounded, query from child side
const logSchema = new mongoose.Schema({
  message: String,
  host: { type: mongoose.Schema.Types.ObjectId, ref: 'Host', required: true, index: true },
});
```

| Relationship | Strategy | When to Use |
|--------------|----------|-------------|
| One-to-few | Embed | Bounded, accessed together |
| One-to-many | Reference array | Bounded, shared, independent access |
| One-to-squillions | Parent reference | Unbounded, query from child side |
| Denormalise | Copy field | Read-heavy, rarely updated |

**Constraints and Limitations:**
- Embedded arrays are limited by the 16MB document size limit.
- `$lookup` in aggregation pipelines does not work on embedded structures directly; use `$unwind` first.
- Referencing requires N+1 query avoidance; use `$lookup` or batch `populate()`.

#### Annotated Code Example

```javascript
// mongo-schema-design.js
const mongoose = require('mongoose');

// Embed: User with addresses
const userSchema = new mongoose.Schema({
  name: String,
  addresses: [{
    street: String,
    city: String,
    zipCode: String,
  }],
});

// Reference: Order with product references
const orderSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  items: [{
    productId: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
    quantity: Number,
  }],
  total: Number,
});

const User = mongoose.model('User', userSchema);
const Order = mongoose.model('Order', orderSchema);

async function main() {
  await mongoose.connect(process.env.MONGO_URI);

  // Embedded document — single read
  const user = await User.create({
    name: 'Alice',
    addresses: [
      { street: '123 Main St', city: 'London', zipCode: 'SW1A 1AA' },
      { street: '456 High St', city: 'Paris', zipCode: '75001' },
    ],
  });
  console.log('User with embedded addresses:', user.addresses.length);

  // Reference with populate — multiple reads
  const order = await Order.create({
    userId: user._id,
    items: [{ productId: new mongoose.Types.ObjectId(), quantity: 2 }],
    total: 100,
  });
  const populated = await Order.findById(order._id).populate('userId', 'name');
  console.log('Order populated with user:', populated.userId.name);

  await mongoose.disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
User with embedded addresses: 2
Order populated with user: Alice
```

**Why this output:** The embedded addresses are stored inside the user document and retrieved in a single read. The order references the user by ID; `populate()` performs a second query to resolve the reference, demonstrating the trade-off between single-read embedding and multi-read referencing.

#### Real-World Cases

- **E-commerce:** Embed order line items (bounded); reference products (shared, independent).
- **Social media:** Embed comments (bounded, accessed with post); reference users (shared).
- **IoT:** Reference sensor readings from device (unbounded); embed device metadata (bounded).

---

### Sub-Feature 3.2: Aggregation Pipelines for Complex Data Processing in Node.js

#### Definitions

**Core Definition:** The MongoDB aggregation pipeline is a framework for data processing that passes documents through a sequence of stages (`$match`, `$group`, `$lookup`, `$project`, etc.), each transforming the data.

**Technical Definition:** The aggregation pipeline is MongoDB's equivalent of SQL's `GROUP BY`, `JOIN`, and window functions. Stages are executed in order: `$match` filters documents early (use indexes), `$lookup` performs left outer joins with other collections, `$group` aggregates, `$project` shapes output, and `$sort`/`$limit` finalise results. The pipeline runs on the server, minimising data transfer to Node.js.

**Beginner-Friendly Explanation:** An aggregation pipeline is like an assembly line for data. Raw documents go in one end; each stage does something (filter, group, join, reshape); the final result comes out. It's much faster than fetching all documents and processing them in JavaScript because the heavy lifting happens on the database server.

#### Purposes

- To perform complex data transformations on the server.
- To join collections without multiple round trips.
- To aggregate, group, and reshape data for reports and dashboards.
- To optimise performance by filtering early with `$match`.

#### Syntax Rules and Structure

```javascript
const result = await db.collection('orders').aggregate([
  // Stage 1: Filter early (uses index)
  { $match: { status: 'completed', createdAt: { $gte: new Date('2026-01-01') } } },

  // Stage 2: Join with users
  { $lookup: {
    from: 'users',
    localField: 'userId',
    foreignField: '_id',
    as: 'user',
  }},

  // Stage 3: Unwind the joined array
  { $unwind: '$user' },

  // Stage 4: Group by user and sum totals
  { $group: {
    _id: '$user._id',
    userName: { $first: '$user.name' },
    totalSpent: { $sum: '$total' },
    orderCount: { $sum: 1 },
  }},

  // Stage 5: Sort and limit
  { $sort: { totalSpent: -1 } },
  { $limit: 10 },

  // Stage 6: Shape output
  { $project: {
    _id: 0,
    userId: '$_id',
    userName: 1,
    totalSpent: 1,
    orderCount: 1,
  }},
]).toArray();
```

| Stage | Purpose | When to Use |
|-------|---------|-------------|
| `$match` | Filter documents | First stage (uses indexes) |
| `$lookup` | Join collections | When referencing data |
| `$group` | Aggregate | Sum, average, count |
| `$project` | Shape output | Select/rename fields |
| `$sort` | Order results | After grouping |
| `$limit` | Limit results | After sorting |

**Constraints and Limitations:**
- `$match` should be the first stage to leverage indexes and reduce pipeline size.
- `$lookup` on large collections can be slow; ensure the foreign field is indexed.
- Aggregation pipelines cannot use Mongoose middleware hooks.

#### Annotated Code Example

```javascript
// mongo-aggregation.js
const { MongoClient } = require('mongodb');

async function main() {
  const client = new MongoClient(process.env.MONGO_URI);
  await client.connect();
  const db = client.db('shop');

  const result = await db.collection('orders').aggregate([
    // Filter completed orders
    { $match: { status: 'completed' } },

    // Join with users
    { $lookup: {
      from: 'users',
      localField: 'userId',
      foreignField: '_id',
      as: 'user',
    }},

    // Unwind user array
    { $unwind: '$user' },

    // Group by user
    { $group: {
      _id: '$user._id',
      userName: { $first: '$user.name' },
      totalSpent: { $sum: '$total' },
      orderCount: { $sum: 1 },
    }},

    // Sort by total spent descending
    { $sort: { totalSpent: -1 } },

    // Limit to top 5
    { $limit: 5 },

    // Shape output
    { $project: {
      _id: 0,
      userId: '$_id',
      userName: 1,
      totalSpent: 1,
      orderCount: 1,
    }},
  ]).toArray();

  console.log('Top customers:', result);
  await client.close();
}

main().catch(console.error);
```

**Expected Output:**
```
Top customers: [
  { userId: ObjectId("..."), userName: 'Alice', totalSpent: 1500, orderCount: 5 },
  { userId: ObjectId("..."), userName: 'Bob', totalSpent: 800, orderCount: 3 }
]
```

**Why this output:** The pipeline filters completed orders, joins with users, groups by user to calculate total spent and order count, sorts by total spent, and limits to the top 5. The entire computation happens on the MongoDB server, returning only the aggregated results.

#### Real-World Cases

- **E-commerce:** Top customers, monthly revenue, product popularity.
- **Analytics:** User engagement metrics, funnel analysis, cohort retention.
- **Reporting:** Daily/weekly/monthly summaries with `$group` and date operators.

---

## Core Concept 4: Key-Value Modeling

### Sub-Feature 4.1: TTL (Time-To-Live) Management and Cache Eviction Policies

#### Definitions

**Core Definition:** TTL (Time-To-Live) is a Redis feature that automatically deletes a key after a specified duration, while eviction policies determine which keys Redis removes when memory is full.

**Technical Definition:** Redis keys can have an expiration time set via `EXPIRE`, `SETEX`, or `SET` with the `EX` option. When the TTL expires, Redis automatically deletes the key. Eviction policies are configured via `maxmemory-policy`: `volatile-lru` evicts the least recently used keys **with a TTL set**; `allkeys-lru` evicts the least recently used keys **regardless of TTL**; `noeviction` returns errors when memory is full. The default policy is `volatile-lru`, meaning only keys with TTLs are eligible for eviction.

**Beginner-Friendly Explanation:** TTL is like a self-destruct timer on a message — after the set time, it's automatically deleted. Eviction policies are like a bouncer who decides who leaves when the club is full. "volatile-lru" only kicks out people who have a return ticket (TTL); "allkeys-lru" kicks out anyone, but always the least recently active.

#### Purposes

- To automatically clean up expired data (sessions, caches, tokens).
- To prevent Redis memory from growing unbounded.
- To prioritise eviction of temporary data over persistent data (volatile-lru).
- To ensure cache keys are recycled efficiently.

#### Syntax Rules and Structure

```javascript
// Set with TTL (seconds)
await redis.setex('session:abc', 3600, JSON.stringify(sessionData));

// Set with TTL (milliseconds)
await redis.psetex('lock:resource', 5000, 'locked');

// Set TTL on existing key
await redis.expire('user:1', 86400);

// Check TTL
const ttl = await redis.ttl('user:1'); // -1 = no TTL, -2 = key doesn't exist
```

| Eviction Policy | Description | Use Case |
|-----------------|-------------|----------|
| `volatile-lru` | Evict LRU keys with TTL | Mixed persistent + cache data |
| `allkeys-lru` | Evict LRU keys regardless of TTL | Pure cache |
| `volatile-ttl` | Evict keys with nearest expiry | Cache with varying TTLs |
| `noeviction` | Return errors when full | Critical data (no eviction) |

**Constraints and Limitations:**
- `volatile-lru` does not evict keys without TTL; if no keys have TTLs, Redis returns errors.
- `allkeys-lru` may evict persistent keys if they are least recently used.
- TTLs are not persisted across Redis restarts unless RDB or AOF persistence is enabled.

#### Annotated Code Example

```javascript
// redis-ttl.js
const Redis = require('ioredis');
const redis = new Redis({ maxRetriesPerRequest: null });

async function main() {
  // Session with 24-hour TTL
  await redis.setex('session:user:1', 86400, JSON.stringify({
    userId: 1,
    name: 'Alice',
  }));
  console.log('Session TTL:', await redis.ttl('session:user:1')); // 86400

  // Cache with 5-minute TTL
  await redis.setex('cache:products', 300, JSON.stringify([{ id: 1, name: 'Widget' }]));
  console.log('Cache TTL:', await redis.ttl('cache:products')); // 300

  // Persistent key (no TTL) — not evicted under volatile-lru
  await redis.set('config:app', JSON.stringify({ theme: 'dark' }));
  console.log('Config TTL:', await redis.ttl('config:app')); // -1

  // Check eviction policy
  const policy = await redis.config('GET', 'maxmemory-policy');
  console.log('Eviction policy:', policy);

  redis.disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
Session TTL: 86400
Cache TTL: 300
Config TTL: -1
Eviction policy: [ 'maxmemory-policy', 'volatile-lru' ]
```

**Why this output:** `setex` sets the key with a TTL. `ttl` returns the remaining time (`-1` means no TTL). The config key has no TTL, so it would not be evicted under `volatile-lru`. The eviction policy confirms that only keys with TTLs are eligible for eviction.

#### Real-World Cases

- **Session management:** Session keys with 24-hour TTL; refreshed on each request.
- **API caching:** Cache keys with 5-minute TTL; evicted when memory is full.
- **Rate limiting:** Rate limit keys with 1-minute TTL; automatically reset.

---

### Sub-Feature 4.2: Session Store Integration for Scalable Web Backends

#### Definitions

**Core Definition:** A Redis session store persists user session data in Redis, enabling stateless web servers that can scale horizontally across multiple instances.

**Technical Definition:** `express-session` with `connect-redis` stores session data in Redis with automatic TTL management. The session ID is stored in a cookie; the session data (user ID, role, preferences) is stored in Redis. This enables multiple Node.js instances to share session state, as any instance can look up the session from Redis.

**Beginner-Friendly Explanation:** Without a shared session store, each server has its own memory of who's logged in — so if a user's next request goes to a different server, they're logged out. Redis solves this by storing session data centrally, so every server can see it.

#### Purposes

- To enable horizontal scaling of web servers without sticky sessions.
- To provide sub-millisecond session lookups.
- To automatically expire sessions via TTL.
- To invalidate sessions by deleting the Redis key.

#### Syntax Rules and Structure

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const redis = require('redis');

const redisClient = redis.createClient({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  password: process.env.REDIS_PASSWORD,
});
redisClient.connect();

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 1000 * 60 * 60 * 24, // 24 hours
  },
}));
```

**Custom session operations:**
```javascript
class SessionService {
  constructor(client) { this.client = client; }

  async setSession(sessionId, data, ttl = 86400) {
    await this.client.setEx(`sess:${sessionId}`, ttl, JSON.stringify(data));
  }

  async getSession(sessionId) {
    const data = await this.client.get(`sess:${sessionId}`);
    return data ? JSON.parse(data) : null;
  }

  async deleteSession(sessionId) {
    await this.client.del(`sess:${sessionId}`);
  }
}
```

**Constraints and Limitations:**
- `saveUninitialized: false` prevents empty sessions from being stored, reducing memory usage.
- The session secret must be stored in environment variables.
- `secure: true` requires HTTPS; set to `false` in development.

#### Annotated Code Example

```javascript
// redis-session.js
const express = require('express');
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const redis = require('redis');

const app = express();
const redisClient = redis.createClient({
  host: process.env.REDIS_HOST || 'localhost',
  port: 6379,
});
redisClient.connect();

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: 'session-secret',
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, maxAge: 86400000 },
}));

app.post('/login', (req, res) => {
  req.session.userId = 42;
  req.session.username = 'alice';
  res.json({ message: 'Logged in' });
});

app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ message: 'Not authenticated' });
  }
  res.json({ userId: req.session.userId, username: req.session.username });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (login then profile):**
```
POST /login → { "message": "Logged in" }
GET /profile → { "userId": 42, "username": "alice" }
```

**Why this output:** The session data is stored in Redis with a 24-hour TTL. The `/login` route sets the session; the `/profile` route reads it. Any server instance connected to the same Redis can serve the profile request, enabling horizontal scaling.

#### Real-World Cases

- **Multi-server deployments:** Load-balanced Node.js applications sharing sessions.
- **Microservices:** Auth services storing sessions in Redis for other services.
- **Serverless:** Lambda functions sharing sessions via Redis (with ElastiCache or Upstash).

---

## Core Concept 5: Caching

### Sub-Feature 5.1: Cache-Aside (Lazy Loading) and Write-Through Caching Patterns

#### Definitions

**Core Definition:** Cache-aside (lazy loading) loads data into the cache only when requested, while write-through writes to the cache and database simultaneously on every write.

**Technical Definition:** In cache-aside, the application checks the cache first; on a miss, it queries the database and populates the cache. In write-through, every write goes to both the cache and the database, ensuring the cache is always fresh. Cache-aside is simpler but risks stale data; write-through adds write latency but guarantees consistency. A common hybrid is cache-aside for reads with write-through for critical mutations.

**Beginner-Friendly Explanation:** Cache-aside is like checking the fridge before going to the store — if the milk is there, you don't buy more. Write-through is like buying milk and immediately putting it in the fridge, so it's always there when you need it.

#### Purposes

- To reduce database load by serving frequent reads from cache.
- To improve response times for read-heavy workloads.
- To ensure cache consistency for critical data (write-through).
- To avoid cache stampedes with lock-based single-flight.

#### Syntax Rules and Structure

**Cache-aside:**
```javascript
async function getProduct(id) {
  // Check cache
  const cached = await redis.get(`product:${id}`);
  if (cached) return JSON.parse(cached);

  // Cache miss — fetch from DB
  const product = await db.query('SELECT * FROM products WHERE id = $1', [id]);

  // Populate cache with TTL
  await redis.setex(`product:${id}`, 300, JSON.stringify(product));
  return product;
}
```

**Write-through:**
```javascript
async function updateProduct(id, data) {
  // Write to database
  await db.query('UPDATE products SET name = $1 WHERE id = $2', [data.name, id]);

  // Write to cache
  await redis.setex(`product:${id}`, 300, JSON.stringify(data));
}
```

| Pattern | Read | Write | Consistency |
|---------|------|-------|-------------|
| Cache-aside | Cache → DB → Cache | DB only (invalidate cache) | Eventual |
| Write-through | Cache → DB | Cache + DB | Strong |

**Constraints and Limitations:**
- Cache-aside can suffer from stale data if the database is updated without invalidating the cache.
- Write-through doubles write latency.
- Neither pattern handles cache stampedes; use locking or probabilistic early expiration.

#### Annotated Code Example

```javascript
// caching-patterns.js
const Redis = require('ioredis');
const redis = new Redis();

// Simulated database
const db = {
  query: async (sql) => {
    await new Promise(r => setTimeout(r, 50)); // Simulate latency
    return { id: 1, name: 'Widget', price: 9.99 };
  },
  update: async (id, data) => {
    await new Promise(r => setTimeout(r, 50));
    return { id, ...data };
  },
};

// Cache-aside (lazy loading)
async function getProductCacheAside(id) {
  const cached = await redis.get(`product:${id}`);
  if (cached) {
    console.log('Cache hit');
    return JSON.parse(cached);
  }

  console.log('Cache miss — fetching from DB');
  const product = await db.query('SELECT ...');
  await redis.setex(`product:${id}`, 300, JSON.stringify(product));
  return product;
}

// Write-through
async function updateProductWriteThrough(id, data) {
  const updated = await db.update(id, data);
  await redis.setex(`product:${id}`, 300, JSON.stringify(updated));
  console.log('Written to DB and cache');
  return updated;
}

async function main() {
  console.log('--- Cache-Aside ---');
  await getProductCacheAside(1); // Miss
  await getProductCacheAside(1); // Hit

  console.log('--- Write-Through ---');
  await updateProductWriteThrough(1, { name: 'Widget Pro' });

  redis.disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
--- Cache-Aside ---
Cache miss — fetching from DB
Cache hit
--- Write-Through ---
Written to DB and cache
```

**Why this output:** The first `getProductCacheAside` call misses the cache and fetches from the database, populating the cache. The second call hits the cache. The write-through update writes to both the database and the cache.

#### Real-World Cases

- **Product catalogues:** Cache-aside for read-heavy product pages.
- **User profiles:** Write-through for profile updates that must be immediately visible.
- **Configuration:** Cache-aside with long TTLs; write-through for critical settings.

---

### Sub-Feature 5.2: Distributed Locks (Redlock Algorithm) to Prevent Race Conditions

#### Definitions

**Core Definition:** A distributed lock ensures that only one process across multiple servers can access a shared resource at a time, preventing race conditions in distributed systems.

**Technical Definition:** The Redlock algorithm acquires locks on N independent Redis instances. A lock is granted when a majority (N/2 + 1) of instances agree within a bounded time. The `redlock` npm package implements this algorithm. For most applications, a single Redis primary with a finite lease is sufficient; Redlock provides stronger guarantees for critical sections.

**Beginner-Friendly Explanation:** A distributed lock is like a single key to a shared bathroom. Only one person can have the key at a time. In a distributed system with multiple servers, Redlock ensures that even if one Redis server fails, the lock is still granted correctly by the majority of servers.

#### Purposes

- To prevent race conditions in distributed systems.
- To ensure exactly-once processing of critical operations.
- To coordinate access to shared resources (cache rebuild, inventory decrement).
- To prevent duplicate job execution.

#### Syntax Rules and Structure

```javascript
const Redlock = require('redlock');
const Redis = require('ioredis');

const redis = new Redis();
const redlock = new Redlock([redis], {
  retryCount: 3,
  retryDelay: 200, // ms
  retryJitter: 100, // ms
});

async function criticalSection() {
  const lock = await redlock.acquire(['lock:resource'], 5000); // 5s TTL

  try {
    // Only one process executes this
    console.log('Lock acquired — performing critical work');
    await doWork();
  } finally {
    await lock.release();
    console.log('Lock released');
  }
}
```

| Parameter | Description |
|-----------|-------------|
| `resource` | Array of resource names to lock. |
| `ttl` | Lock time-to-live in milliseconds. |
| `retryCount` | Number of retry attempts. |
| `retryDelay` | Base delay between retries (ms). |
| `retryJitter` | Random jitter added to retry delay. |

**Constraints and Limitations:**
- Redlock is contentious; Martin Kleppmann argues it requires strong assumptions about clock synchronisation. For most applications, a single Redis with `SET NX` and TTL is sufficient.
- Always release locks in a `finally` block.
- Lock TTL must be longer than the critical section; otherwise, the lock expires and another process enters.

#### Annotated Code Example

```javascript
// redlock-example.js
const Redis = require('ioredis');
const Redlock = require('redlock');

const redis = new Redis();
const redlock = new Redlock([redis], {
  retryCount: 5,
  retryDelay: 200,
  retryJitter: 100,
});

async function rebuildCache(key, fetchFn, ttl) {
  const lock = await redlock.acquire([`lock:${key}`], 10000);

  try {
    // Double-check cache after acquiring lock
    const cached = await redis.get(key);
    if (cached) return JSON.parse(cached);

    console.log('Rebuilding cache for', key);
    const data = await fetchFn();
    await redis.setex(key, ttl, JSON.stringify(data));
    return data;
  } finally {
    await lock.release();
  }
}

async function fetchFromDB() {
  await new Promise(r => setTimeout(r, 500)); // Simulate slow DB
  return { data: 'fresh', timestamp: Date.now() };
}

async function main() {
  // Simulate 10 concurrent requests
  const results = await Promise.all(
    Array.from({ length: 10 }, () => rebuildCache('popular', fetchFromDB, 60))
  );

  // Only one should have rebuilt the cache
  console.log('All requests completed. Cache rebuilt once.');
  redis.disconnect();
}

main().catch(console.error);
```

**Expected Output:**
```
Rebuilding cache for popular
All requests completed. Cache rebuilt once.
```

**Why this output:** Ten concurrent requests attempt to rebuild the cache. The Redlock ensures only one acquires the lock and fetches from the database. The others wait for the lock, then find the cache already populated and return the cached data. This prevents the cache stampede (thundering herd) problem.

#### Real-World Cases

- **Cache rebuild:** Preventing multiple processes from hitting the database simultaneously when a popular cache key expires.
- **Inventory decrement:** Ensuring stock is decremented exactly once per order.
- **Scheduled jobs:** Preventing duplicate job execution across multiple workers.
- **Payment processing:** Ensuring idempotent payment operations.

---

## References

- MongoDB Node.js Driver Documentation — https://www.mongodb.com/docs/drivers/node/
- Mongoose Documentation — https://mongoosejs.com/docs/
- Mongoose Middleware — https://mongoosejs.com/docs/middleware.html
- MongoDB Change Streams — https://www.mongodb.com/docs/manual/changeStreams/
- MongoDB Aggregation Pipeline — https://www.mongodb.com/docs/manual/core/aggregation-pipeline/
- MongoDB Schema Design: Embed vs Reference — https://www.mongodb.com/docs/manual/data-modeling/
- ioredis GitHub — https://github.com/redis/ioredis
- node-redis Documentation — https://redis.io/docs/latest/develop/clients/nodejs/
- Redis Hashes — https://redis.io/docs/latest/develop/data-types/hashes/
- Redis Sorted Sets — https://redis.io/docs/latest/develop/data-types/sorted-sets/
- Redis Pub/Sub — https://redis.io/docs/latest/develop/interact/pubsub/
- Redis TTL and Expiration — https://redis.io/docs/latest/commands/expire/
- Redis Eviction Policies — https://redis.io/docs/latest/develop/reference/eviction/
- express-session + connect-redis — https://www.npmjs.com/package/connect-redis
- redlock npm — https://www.npmjs.com/package/redlock
- Redis Distributed Locks (Redlock) — https://redis.io/docs/latest/develop/clients/patterns/distributed-locks/
- Cache-Aside Pattern — https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside
- Cache Stampede Prevention — https://en.wikipedia.org/wiki/Cache_stampede
- Mongoose vs Native Driver — https://oneuptime.com/blog/post/2026-03-31-mongodb-mongoose-vs-native-driver-nodejs/view
- Redis Caching Patterns — https://github.com/omer-metin/skills-for-antigravity/blob/main/skills/caching-patterns/references/sharp_edges.md