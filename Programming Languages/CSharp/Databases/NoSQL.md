# NoSQL Databases — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A NoSQL database is a database management system that stores and retrieves data using models other than the relational table model, such as documents, key-value pairs, wide-column stores, or graphs. NoSQL systems are designed for horizontal scalability, flexible schemas, and high availability.

**Technical Definition**
NoSQL ("not only SQL") encompasses a family of database systems that relax one or more constraints of relational databases — fixed schemas, strict ACID transactions across tables, and vertical scalability — in exchange for flexible or schema-less data models, eventual consistency options, partition tolerance, and horizontal scale-out. The four principal data models are document (e.g., MongoDB, Couchbase), key-value (e.g., Redis), wide-column (e.g., Cassandra), and graph (e.g., Neo4j). Each system makes different CAP theorem trade-offs and offers different consistency, durability, and query guarantees.

**Beginner-Friendly Explanation**
A NoSQL database is a database that does not use tables with fixed columns like a traditional SQL database. Instead, it might store data as JSON-like documents (MongoDB), as simple key-value pairs in memory (Redis), or as rows spread across many machines (Cassandra). NoSQL databases are popular when you need to handle huge amounts of data, flexible data shapes, or very fast access, and you are willing to trade some of the strict guarantees of SQL databases.

---

### Key Characteristics

- **Flexible or schema-less data models** — Documents can have different fields; wide-column stores allow dynamic columns.
- **Horizontal scalability** — Designed to scale out across commodity servers via sharding and replication.
- **High availability** — Replication and automatic failover are core features.
- **BASE vs. ACID trade-offs** — Many NoSQL systems favor availability and partition tolerance over strict consistency, though most now offer tunable consistency.
- **Specialized query models** — Queries are often tied to access patterns and data model; some systems lack ad-hoc joins.
- **Polyglot persistence** — Different NoSQL databases are often combined with relational databases and each other.
- **Operational complexity** — Clustering, replication, and consistency tuning require operational expertise.

---

### Prerequisites

- Understanding of data modeling concepts: entities, relationships, normalization vs. denormalization.
- Basic knowledge of the CAP theorem and consistency models.
- Familiarity with at least one programming language and its database drivers.
- Understanding of JSON and key-value data structures.
- Basic Linux/command-line skills for running and administering database servers.
- Awareness of when a relational database is a better fit.

---

### Related Programming Areas

- **Distributed systems** — Sharding, replication, consensus, and failure handling.
- **Caching and performance** — Redis is widely used as a cache and data store.
- **Big data and analytics** — Wide-column stores and document databases support large-scale data processing.
- **Microservices** — NoSQL databases are often used per-service for autonomy and scale.
- **Search** — Document databases and dedicated search engines (Elasticsearch, OpenSearch) complement NoSQL stores.
- **Event sourcing and messaging** — Key-value and log-based stores support event-driven architectures.

---

### Core Concepts / Features

1. NoSQL Databases (overarching concept and data models)
2. MongoDB
3. Redis
4. Cassandra
5. Couchbase
6. NoSQL best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. NoSQL Databases (Overarching Concept and Data Models)

### Definitions

**Core Definition**
NoSQL databases are non-relational database systems built for scale, flexibility, and availability, organized into document, key-value, wide-column, and graph data models.

**Technical Definition**
NoSQL systems partition data across nodes using consistent hashing or range partitioning, replicate partitions for fault tolerance, and expose consistency models ranging from strong (single-node or quorum) to eventual. The CAP theorem states that a distributed system can guarantee at most two of consistency, availability, and partition tolerance; NoSQL systems typically choose AP or CP configurations and often allow per-operation tuning. Data models include document (JSON/BSON), key-value, wide-column (column families), and graph (nodes and edges).

**Beginner-Friendly Explanation**
NoSQL is a big family of databases, not one product. Some store documents (like JSON files), some store simple key-value pairs (like a dictionary), some store rows spread across many machines, and some store graphs of connected data. You choose the type that fits your data and access patterns.

---

### Purposes

- To scale horizontally across many servers.
- To store flexible or evolving data shapes without migrations.
- To achieve high availability through replication.
- To support very high read or write throughput.
- To model data according to access patterns rather than normalization rules.
- To complement relational databases in polyglot architectures.

---

### Syntax Rules and Structure

#### Data Model Overview

```
// Document model (MongoDB, Couchbase)
{
  "_id": "blog:1",
  "title": "NoSQL Intro",
  "tags": ["database", "nosql"],
  "author": { "name": "Alice", "email": "alice@example.com" }
}

// Key-value model (Redis)
SET user:1:name "Alice"
GET user:1:name

// Wide-column model (Cassandra)
CREATE TABLE users (
  user_id uuid PRIMARY KEY,
  email text,
  name text
);

// Graph model (Neo4j)
CREATE (alice:Person {name: 'Alice'})-[:FOLLOWS]->(bob:Person {name: 'Bob'});
```

**Component breakdown**

- Document — Self-contained JSON-like objects with nested structures.
- Key-value — Opaque values indexed by keys.
- Wide-column — Rows with dynamic column families, partitioned by key.
- Graph — Nodes and edges with properties.

#### Syntax Rules

- Data modeling is driven by access patterns, not normalization.
- Denormalization and duplication are common and expected.
- Consistency is often tunable per operation (e.g., quorum reads/writes).
- Schema validation is optional or enforced at the application level.

#### Constraints and Limitations

- Joins are limited or absent; data must be denormalized or joined in the application.
- Transactions may be limited to single documents or partitions (though support is improving).
- Consistency and durability trade-offs must be understood.
- Operational complexity increases with cluster size.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Comparing Data Models Conceptually

```csharp
// Document (MongoDB driver)
var doc = new BsonDocument
{
    { "_id", "blog:1" },
    { "title", "NoSQL Intro" },
    { "tags", new BsonArray { "database", "nosql" } }
};

// Key-value (Redis via StackExchange.Redis)
var db = redis.GetDatabase();
db.StringSet("user:1:name", "Alice");
var name = db.StringGet("user:1:name");

// Wide-column (Cassandra via CQL)
// INSERT INTO users (user_id, email, name) VALUES (uuid(), 'a@example.com', 'Alice');

// Graph (Neo4j via Cypher)
// CREATE (:Person {name: 'Alice'})-[:FOLLOWS]->(:Person {name: 'Bob'});
```

**Expected Output (conceptual)**

```
Document stored in MongoDB; Redis returns "Alice"; Cassandra stores the row; Neo4j stores the edge.
```

**Why this output occurs**

- Each database uses its native model and API.
- The same logical data is represented differently based on the data model.

---

### Real-World Cases

- **E-commerce catalogs** — Document databases store products with varying attributes.
- **Caching and sessions** — Redis stores frequently accessed data and session state.
- **IoT and time series** — Cassandra handles high write throughput and time-series data.
- **Social graphs** — Graph databases model relationships and recommendations.

---

### References

- NoSQL databases overview - https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data
- CAP theorem (IBM) - https://www.ibm.com/topics/cap-theorem
- NoSQL (Wikipedia) - https://en.wikipedia.org/wiki/NoSQL

---

## 2. MongoDB

### Definitions

**Core Definition**
MongoDB is a document-oriented NoSQL database that stores data as flexible BSON documents and provides a rich query language, secondary indexes, replication, and sharding.

**Technical Definition**
MongoDB stores documents in collections within databases. Documents are BSON (binary JSON) with a mandatory `_id` primary key. MongoDB supports CRUD operations, aggregation pipelines, secondary indexes (including compound, text, geospatial, and TTL indexes), replica sets for high availability, and sharded clusters for horizontal scaling. It offers multi-document ACID transactions on replica sets and sharded clusters (since 4.0/4.2). Consistency is tunable via read concern and write concern. The official .NET driver is `MongoDB.Driver`.

**Beginner-Friendly Explanation**
MongoDB is like a filing cabinet of JSON documents. You put documents into collections, and each document can have different fields. You can search, update, and aggregate them. MongoDB can replicate data across servers and split it across many machines for scale.

---

### Purposes

- To store flexible, nested documents without a fixed schema.
- To scale horizontally with sharding.
- To provide high availability with replica sets.
- To support rich queries and aggregation pipelines.
- To handle mixed workloads with tunable consistency.
- To support multi-document transactions when needed.

---

### Syntax Rules and Structure

#### General Syntax (MongoDB Shell)

```
// Insert
db.blogs.insertOne({ title: "NoSQL Intro", tags: ["db"], author: { name: "Alice" } })

// Query
db.blogs.find({ tags: "db" }).sort({ title: 1 }).limit(10)

// Update
db.blogs.updateOne({ _id: ObjectId("...") }, { $set: { title: "Updated" } })

// Delete
db.blogs.deleteOne({ _id: ObjectId("...") })

// Aggregation
db.blogs.aggregate([
  { $match: { tags: "db" } },
  { $group: { _id: "$author.name", count: { $sum: 1 } } }
])

// Index
db.blogs.createIndex({ title: 1 })
```

**Component breakdown**

- `insertOne` — Inserts a single document.
- `find` — Queries documents with filters, sort, and limit.
- `$set` — Updates specific fields.
- `$match`, `$group` — Aggregation pipeline stages.
- `createIndex` — Creates a secondary index.

#### Syntax Rules (C# Driver)

- Use `IMongoCollection<T>` for typed collections.
- Use `Builders<T>.Filter` and `Builders<T>.Update` for type-safe queries.
- `_id` is mapped to a property named `Id` or annotated with `[BsonId]`.
- Use `ReplaceOneAsync` with `IsUpsert = true` for upserts.
- Transactions require a session and a replica set or sharded cluster.

#### Constraints and Limitations

- Document size limit is 16 MB.
- Joins are limited to `$lookup` in aggregation; denormalization is common.
- Multi-document transactions have performance overhead.
- Sharding requires choosing a shard key carefully; bad shard keys cause hotspots.
- Schema validation is optional and must be designed deliberately.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CRUD with the .NET Driver

```csharp
using System;
using System.Threading.Tasks;
using MongoDB.Bson;
using MongoDB.Driver;

public class Blog
{
    public ObjectId Id { get; set; }
    public string Title { get; set; } = "";
    public string[] Tags { get; set; } = Array.Empty<string>();
    public Author Author { get; set; } = new();
}

public class Author { public string Name { get; set; } = ""; public string Email { get; set; } = ""; }

class Program
{
    static async Task Main()
    {
        var client = new MongoClient("mongodb://localhost:27017");
        var db = client.GetDatabase("blogging");
        var blogs = db.GetCollection<Blog>("blogs");

        var blog = new Blog { Title = "NoSQL Intro", Tags = new[] { "db", "nosql" }, Author = new Author { Name = "Alice", Email = "alice@example.com" } };
        await blogs.InsertOneAsync(blog);
        Console.WriteLine($"Inserted: {blog.Id}");

        var filter = Builders<Blog>.Filter.AnyEq(b => b.Tags, "db");
        var results = await blogs.Find(filter).SortBy(b => b.Title).ToListAsync();
        foreach (var b in results)
            Console.WriteLine($"{b.Title} by {b.Author.Name}");
    }
}
```

**Line-by-line comments**

- `MongoClient` — Connects to the MongoDB server.
- `GetCollection<Blog>("blogs")` — Gets the typed collection.
- `InsertOneAsync` — Inserts the document and populates `Id`.
- `Builders<Blog>.Filter.AnyEq` — Filters documents where the `Tags` array contains `"db"`.
- `Find(...).SortBy(...).ToListAsync()` — Executes the query.

**Expected Output**

```
Inserted: ObjectId("...")
NoSQL Intro by Alice
```

**Why this output occurs**

- The document is inserted and assigned an `_id`.
- The query matches the array element and maps the document back to `Blog`.

---

#### Example 2: Aggregation Pipeline

```csharp
var pipeline = blogs.Aggregate()
    .Match(Builders<Blog>.Filter.AnyEq(b => b.Tags, "db"))
    .Group(b => b.Author.Name, g => new { Author = g.Key, Count = g.Count() })
    .SortByDescending(x => x.Count);

var result = await pipeline.ToListAsync();
foreach (var r in result)
    Console.WriteLine($"{r.Author}: {r.Count}");
```

**Expected Output (example)**

```
Alice: 1
```

**Why this output occurs**

- `Match` filters documents with the `db` tag.
- `Group` aggregates by author name.
- `SortByDescending` orders by count.

---

### Real-World Cases

- **Content management** — Articles with varying metadata and nested comments.
- **Product catalogs** — Products with category-specific attributes.
- **IoT and telemetry** — Time-series collections (MongoDB 5.0+) for sensor data.
- **User profiles** — Nested preferences and settings.

---

### References

- MongoDB documentation - https://www.mongodb.com/docs/
- MongoDB .NET driver documentation - https://www.mongodb.com/docs/drivers/csharp/
- MongoDB manual — Data modeling - https://www.mongodb.com/docs/manual/data-modeling/
- MongoDB transactions - https://www.mongodb.com/docs/manual/core/transactions/

---

## 3. Redis

### Definitions

**Core Definition**
Redis is an in-memory key-value data store that supports strings, hashes, lists, sets, sorted sets, streams, and other data structures, with optional persistence and replication.

**Technical Definition**
Redis (Remote Dictionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. It provides rich data types, atomic operations, pub/sub, Lua scripting, transactions (`MULTI`/`EXEC`), and optional persistence via RDB snapshots and AOF logs. Redis supports replication, Sentinel for high availability, and Redis Cluster for sharding. The .NET client is `StackExchange.Redis`. Redis 7+ is licensed under RSALv2/SSPLv1; Valkey is an open-source fork.

**Beginner-Friendly Explanation**
Redis is a super-fast dictionary that lives in memory. You can store simple values, lists, sets, and more. Because it is in memory, it is extremely fast, which makes it popular for caching, session storage, and real-time features.

---

### Purposes

- To cache frequently accessed data to reduce database load.
- To store session state and user data with fast access.
- To implement rate limiting and counters.
- To provide pub/sub and streams for messaging.
- To support leaderboards and rankings with sorted sets.
- To implement distributed locks and coordination primitives.

---

### Syntax Rules and Structure

#### General Syntax (Redis CLI)

```
SET user:1:name "Alice"
GET user:1:name
EXPIRE user:1:name 3600

HSET user:1 name "Alice" email "alice@example.com"
HGETALL user:1

LPUSH queue:jobs "job1"
RPOP queue:jobs

SADD tags:db "database" "nosql"
SMEMBERS tags:db

ZADD leaderboard 100 "Alice" 200 "Bob"
ZREVRANGE leaderboard 0 9 WITHSCORES

XADD stream:events * type "login" user "Alice"
XREAD COUNT 10 STREAMS stream:events 0
```

**Component breakdown**

- `SET` / `GET` — String operations with optional expiration.
- `HSET` / `HGETALL` — Hash operations for objects.
- `LPUSH` / `RPOP` — List operations for queues.
- `SADD` / `SMEMBERS` — Set operations.
- `ZADD` / `ZREVRANGE` — Sorted set operations for rankings.
- `XADD` / `XREAD` — Stream operations for event logs.

#### Syntax Rules (C# with StackExchange.Redis)

- Create a `ConnectionMultiplexer` and reuse it.
- Get a database via `multiplexer.GetDatabase()`.
- Use async methods (`StringSetAsync`, `HashGetAllAsync`, etc.).
- Keys should be namespaced with colons (`user:1:name`).
- Use `IDistributedCache` in ASP.NET Core for abstraction.

#### Constraints and Limitations

- Data is primarily in memory; capacity is limited by RAM.
- Persistence is optional; durability depends on RDB/AOF configuration.
- Single-threaded command execution per node; avoid long-running commands.
- Cluster mode has limitations on multi-key operations.
- Licensing: Redis 7+ uses RSALv2/SSPLv1; Valkey is the open-source alternative.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic CRUD with StackExchange.Redis

```csharp
using System;
using System.Threading.Tasks;
using StackExchange.Redis;

class Program
{
    static async Task Main()
    {
        var redis = await ConnectionMultiplexer.ConnectAsync("localhost:6379");
        var db = redis.GetDatabase();

        await db.StringSetAsync("user:1:name", "Alice", TimeSpan.FromHours(1));
        var name = await db.StringGetAsync("user:1:name");
        Console.WriteLine($"name={name}");

        await db.HashSetAsync("user:1", new HashEntry[]
        {
            new("email", "alice@example.com"),
            new("role", "admin")
        });

        var entries = await db.HashGetAllAsync("user:1");
        foreach (var e in entries)
            Console.WriteLine($"{e.Name}={e.Value}");

        await db.SortedSetAddAsync("leaderboard", new[]
        {
            new SortedSetEntry("Alice", 100),
            new SortedSetEntry("Bob", 200)
        });

        var top = await db.SortedSetRangeByRankWithScoresAsync("leaderboard", 0, -1, Order.Descending);
        foreach (var item in top)
            Console.WriteLine($"{item.Element}: {item.Score}");
    }
}
```

**Expected Output**

```
name=Alice
email=alice@example.com
role=admin
Bob: 200
Alice: 100
```

**Why this output occurs**

- String operations store and retrieve the name.
- Hash operations store the user object fields.
- Sorted set operations return entries ordered by score descending.

---

#### Example 2: Distributed Lock with SET NX

```csharp
var lockKey = "lock:job:42";
var token = Guid.NewGuid().ToString();
var acquired = await db.StringSetAsync(lockKey, token, TimeSpan.FromSeconds(30), When.NotExists);

if (acquired)
{
    try
    {
        // perform critical work
        Console.WriteLine("Lock acquired");
    }
    finally
    {
        // release only if we still own the lock (Lua script recommended for atomicity)
        await db.ScriptEvaluateAsync(
            "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end",
            new RedisKey[] { lockKey }, new RedisValue[] { token });
    }
}
else
{
    Console.WriteLine("Lock not acquired");
}
```

**Expected Output (example)**

```
Lock acquired
```

**Why this output occurs**

- `SET NX` acquires the lock only if it does not exist.
- The Lua script releases the lock only if the token matches, preventing accidental release.

---

### Real-World Cases

- **Caching** — Cache database query results with expiration.
- **Session storage** — Store user sessions in ASP.NET Core.
- **Rate limiting** — Counters with expiration for API throttling.
- **Leaderboards** — Sorted sets for gaming rankings.
- **Event streams** — Redis Streams for lightweight messaging.

---

### References

- Redis documentation - https://redis.io/docs/
- StackExchange.Redis documentation - https://stackexchange.github.io/StackExchange.Redis/
- Redis data types - https://redis.io/docs/data-types/
- Redis persistence - https://redis.io/docs/management/persistence/
- Valkey (open-source Redis fork) - https://valkey.io/

---

## 4. Cassandra

### Definitions

**Core Definition**
Apache Cassandra is a distributed, wide-column NoSQL database designed for high availability, linear scalability, and high write throughput across many commodity servers.

**Technical Definition**
Cassandra uses a peer-to-peer, masterless architecture with consistent hashing (the partitioner) to distribute rows across nodes. Data is organized into keyspaces, tables, partitions, and clustering columns. The primary key consists of a partition key and optional clustering columns; the partition key determines the node, and clustering columns determine sort order within a partition. Replication is controlled per keyspace with replication factors. Consistency is tunable per operation (ONE, QUORUM, ALL, LOCAL_QUORUM, etc.). Cassandra uses a log-structured storage engine (SSTables, commit log, memtables) and supports CQL (Cassandra Query Language). The .NET driver is `CassandraCSharpDriver`.

**Beginner-Friendly Explanation**
Cassandra is built to handle massive amounts of data across many servers with no single point of failure. You design tables around your queries: the partition key decides which server holds the data, and clustering columns decide the order. It is great for writes and scale, but you must design your data model carefully.

---

### Purposes

- To handle high write throughput and large data volumes.
- To provide always-on availability with no single point of failure.
- To scale linearly by adding nodes.
- To replicate data across data centers.
- To support time-series and messaging workloads.
- To offer tunable consistency per query.

---

### Syntax Rules and Structure

#### General Syntax (CQL)

```
CREATE KEYSPACE blogging
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

CREATE TABLE blogging.posts (
  blog_id uuid,
  post_id timeuuid,
  title text,
  body text,
  PRIMARY KEY (blog_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

INSERT INTO blogging.posts (blog_id, post_id, title, body)
VALUES (uuid(), now(), 'NoSQL Intro', 'Cassandra basics');

SELECT * FROM blogging.posts WHERE blog_id = ? LIMIT 10;

UPDATE blogging.posts SET title = 'Updated' WHERE blog_id = ? AND post_id = ?;

DELETE FROM blogging.posts WHERE blog_id = ? AND post_id = ?;
```

**Component breakdown**

- `KEYSPACE` — Top-level namespace with replication settings.
- `PRIMARY KEY (partition_key, clustering_column)` — Defines distribution and sort order.
- `CLUSTERING ORDER BY` — Controls order within a partition.
- `INSERT`, `SELECT`, `UPDATE`, `DELETE` — CQL data manipulation.
- `?` — Prepared statement parameter placeholder.

#### Syntax Rules (C# Driver)

- Use `Cluster.Builder().AddContactPoint(...).Build()` to connect.
- Use `ISession` for queries; sessions are thread-safe and long-lived.
- Use prepared statements (`session.Prepare`) for repeated queries.
- Use `SimpleStatement` for ad-hoc queries; prefer bound statements.
- Set consistency level per statement (`QueryOptions.SetConsistencyLevel`).

#### Constraints and Limitations

- Queries must be designed around the partition key; filtering on non-key columns is restricted.
- `ALLOW FILTERING` is expensive and should be avoided in production.
- No joins; denormalization and duplicate tables per query are standard.
- Lightweight transactions (LWT) using `IF` conditions have performance costs.
- Tombstones from deletes can impact performance if not managed.
- Collection columns have size and performance considerations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CRUD with the .NET Driver

```csharp
using System;
using System.Threading.Tasks;
using Cassandra;

class Program
{
    static async Task Main()
    {
        var cluster = Cluster.Builder()
            .AddContactPoint("127.0.0.1")
            .WithPort(9042)
            .Build();

        var session = await cluster.ConnectAsync("blogging");

        var insert = await session.PrepareAsync(
            "INSERT INTO posts (blog_id, post_id, title, body) VALUES (?, now(), ?, ?)");
        var blogId = Guid.NewGuid();
        await session.ExecuteAsync(insert.Bind(blogId, "NoSQL Intro", "Cassandra basics"));

        var select = await session.PrepareAsync(
            "SELECT post_id, title FROM posts WHERE blog_id = ? LIMIT 10");
        var rows = await session.ExecuteAsync(select.Bind(blogId));

        foreach (var row in rows)
            Console.WriteLine($"{row.GetValue<Guid>("post_id")}: {row.GetValue<string>("title")}");
    }
}
```

**Line-by-line comments**

- `Cluster.Builder()` — Configures contact points and port.
- `ConnectAsync("blogging")` — Connects to the keyspace.
- `PrepareAsync` — Prepares the insert statement.
- `Bind` — Binds parameters to the prepared statement.
- `ExecuteAsync` — Executes the statement.
- The select uses the partition key `blog_id`, which is required for efficient queries.

**Expected Output (example)**

```
<guid>: NoSQL Intro
```

**Why this output occurs**

- The row is inserted into the partition for `blog_id`.
- The select queries that partition and returns the row.
- `now()` generates a timeuuid for the clustering column.

---

#### Example 2: Tunable Consistency

```csharp
var statement = new SimpleStatement("SELECT * FROM posts WHERE blog_id = ?", blogId)
    .SetConsistencyLevel(ConsistencyLevel.LocalQuorum);

var rows = await session.ExecuteAsync(statement);
Console.WriteLine($"rows={rows.Count()}");
```

**Expected Output (example)**

```
rows=1
```

**Why this output occurs**

- `LocalQuorum` requires a quorum of replicas in the local data center to respond.
- The query returns the row if the consistency level is satisfied.

---

### Real-World Cases

- **Messaging** — Cassandra is used by Discord and Netflix for message storage.
- **IoT telemetry** — High write throughput for sensor data.
- **Time series** — Partition by device and cluster by time.
- **Recommendation feeds** — Denormalized tables per access pattern.

---

### References

- Apache Cassandra documentation - https://cassandra.apache.org/doc/latest/
- Cassandra CQL reference - https://cassandra.apache.org/doc/latest/cassandra/cql/
- DataStax C# driver documentation - https://docs.datastax.com/en/developer/csharp-driver/latest/
- Cassandra data modeling - https://cassandra.apache.org/doc/latest/cassandra/data_modeling/

---

## 5. Couchbase

### Definitions

**Core Definition**
Couchbase is a distributed document database with a memory-first architecture, SQL++ (N1QL) query language, and integrated caching, search, and analytics services.

**Technical Definition**
Couchbase Server stores JSON documents in buckets, with keys mapped to vBuckets distributed across nodes. It provides memory-first access with persistence to disk, replication, and automatic failover. Couchbase supports SQL++ (formerly N1QL), a SQL-like query language for JSON, plus full-text search, analytics, and eventing services. The .NET SDK is `Couchbase.NetClient` (SDK 3.x). Couchbase Capella is the managed cloud offering.

**Beginner-Friendly Explanation**
Couchbase is a document database that keeps data in memory for speed and on disk for durability. You store JSON documents and query them with a SQL-like language called SQL++. It also includes search and analytics features.

---

### Purposes

- To store and query JSON documents with SQL-like syntax.
- To provide low-latency access with memory-first architecture.
- To scale horizontally with automatic data distribution.
- To support full-text search and analytics alongside operational data.
- To offer flexible deployment (self-managed or Capella cloud).
- To support mobile and edge sync via Couchbase Lite.

---

### Syntax Rules and Structure

#### General Syntax (SQL++ / N1QL)

```
// Insert
INSERT INTO `blogging` (KEY, VALUE)
VALUES ("blog:1", { "title": "NoSQL Intro", "tags": ["db"] });

// Query
SELECT b.title, b.tags
FROM `blogging` b
WHERE "db" IN b.tags
ORDER BY b.title
LIMIT 10;

// Update
UPDATE `blogging` b
SET b.title = "Updated"
WHERE META(b).id = "blog:1";

// Delete
DELETE FROM `blogging` b
WHERE META(b).id = "blog:1";

// Index
CREATE INDEX idx_tags ON `blogging` (DISTINCT ARRAY t FOR t IN tags END);
```

**Component breakdown**

- `INSERT INTO ... (KEY, VALUE)` — Inserts a document with a key.
- `SELECT ... FROM bucket` — Queries documents.
- `META(b).id` — Accesses the document key.
- `CREATE INDEX` — Creates a secondary index, including array indexes.

#### Syntax Rules (C# SDK)

- Use `Cluster.ConnectAsync` and `BucketAsync` to access buckets.
- Use `Collection.InsertAsync`, `GetAsync`, `UpsertAsync`, `RemoveAsync`.
- Use `Cluster.QueryAsync<T>` for SQL++ queries.
- Use parameterized queries (`$param`) to prevent injection.
- Use `IScope` and `ICollection` for multi-tenancy (collections).

#### Constraints and Limitations

- SQL++ queries require appropriate indexes; otherwise they may be slow or rejected.
- Document size limit is 20 MB.
- Cross-document transactions are supported but have overhead.
- Memory-first architecture requires sizing RAM for the working set.
- Some features (analytics, search) are separate services.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CRUD with the .NET SDK

```csharp
using System;
using System.Threading.Tasks;
using Couchbase;
using Couchbase.KeyValue;

public class Blog
{
    public string Title { get; set; } = "";
    public string[] Tags { get; set; } = Array.Empty<string>();
}

class Program
{
    static async Task Main()
    {
        await using var cluster = await Cluster.ConnectAsync(
            "couchbases://localhost", "Administrator", "password");

        var bucket = await cluster.BucketAsync("blogging");
        var collection = bucket.DefaultCollection();

        var blog = new Blog { Title = "NoSQL Intro", Tags = new[] { "db", "nosql" } };
        await collection.UpsertAsync("blog:1", blog);

        var result = await collection.GetAsync("blog:1");
        var loaded = result.ContentAs<Blog>();
        Console.WriteLine($"title={loaded.Title}, tags={string.Join(",", loaded.Tags)}");

        var query = await cluster.QueryAsync<Blog>(
            "SELECT b.title, b.tags FROM `blogging` b WHERE $tag IN b.tags",
            options => options.Parameter("tag", "db"));

        await foreach (var row in query.Rows)
            Console.WriteLine($"{row.Title}");
    }
}
```

**Line-by-line comments**

- `Cluster.ConnectAsync` — Connects to the cluster.
- `BucketAsync("blogging")` — Gets the bucket.
- `DefaultCollection()` — Uses the default collection.
- `UpsertAsync` — Inserts or replaces the document.
- `ContentAs<Blog>()` — Deserializes the document.
- `cluster.QueryAsync<Blog>` — Executes SQL++ with a parameter.

**Expected Output**

```
title=NoSQL Intro, tags=db,nosql
NoSQL Intro
```

**Why this output occurs**

- The document is upserted and retrieved.
- The SQL++ query filters documents whose `tags` array contains `"db"`.

---

#### Example 2: Parameterized SQL++ Query

```csharp
var query = await cluster.QueryAsync<dynamic>(
    "SELECT META(b).id AS id, b.title FROM `blogging` b WHERE b.title LIKE $pattern",
    options => options.Parameter("pattern", "NoSQL%"));

await foreach (var row in query.Rows)
    Console.WriteLine($"{row.id}: {row.title}");
```

**Expected Output (example)**

```
blog:1: NoSQL Intro
```

**Why this output occurs**

- The parameter `$pattern` is bound safely.
- The query returns matching documents.

---

### Real-World Cases

- **User profiles and sessions** — Low-latency document access.
- **Product catalogs** — Flexible documents with SQL++ queries.
- **Mobile sync** — Couchbase Lite and Sync Gateway for offline-first apps.
- **Operational analytics** — Analytics service over operational data.

---

### References

- Couchbase documentation - https://docs.couchbase.com/home/index.html
- Couchbase .NET SDK documentation - https://docs.couchbase.com/dotnet-sdk/current/hello-world/start-using-sdk.html
- SQL++ reference - https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/index.html
- Couchbase data modeling - https://docs.couchbase.com/server/current/data-modeling/data-modeling.html

---

## 6. NoSQL Best Practices

### Definitions

**Core Definition**
NoSQL best practices are guidelines for choosing, modeling, operating, and querying NoSQL databases to achieve correctness, performance, and reliability.

**Technical Definition**
Best practices span data modeling (query-driven design, denormalization, key design), consistency tuning, index management, operational concerns (backup, monitoring, capacity planning), security, and polyglot persistence. They differ by database family but share principles: understand access patterns, avoid unbounded partitions, design keys carefully, and plan for failure.

**Beginner-Friendly Explanation**
Best practices are "the habits that keep a NoSQL database fast, reliable, and safe." Because NoSQL databases make different trade-offs than SQL databases, the rules for modeling and operating them are different too.

---

### Purposes

- To choose the right database for the workload.
- To model data around access patterns.
- To avoid hotspots and unbounded partitions.
- To tune consistency and durability appropriately.
- To operate clusters reliably and securely.
- To integrate NoSQL with relational systems safely.

---

### Guidelines and Structure

#### Choosing a Database

```
Relational (PostgreSQL, SQL Server) -> complex joins, strong ACID, reporting
Document (MongoDB, Couchbase) -> flexible schemas, nested data, content
Key-value (Redis) -> caching, sessions, counters, pub/sub
Wide-column (Cassandra) -> high write throughput, time series, multi-DC
Graph (Neo4j) -> relationship-heavy queries, recommendations
```

- Match the data model to the access patterns.
- Do not choose NoSQL solely for "scale"; relational databases scale far.
- Consider operational expertise and ecosystem.

#### Data Modeling

```
// Cassandra: partition by access pattern
PRIMARY KEY ((tenant_id, device_id), event_time)

// MongoDB: embed for one-to-few, reference for one-to-many/many-to-many
{ "_id": "...", "author": { "name": "Alice" }, "comments": [ ... ] }

// Redis: namespace keys and set TTLs
SET session:abc "..." EX 3600
```

- Design keys around queries; avoid unbounded partitions.
- Embed vs. reference based on cardinality and access.
- Denormalize for read performance; accept duplication.
- Version schemas and handle migrations in application code.

#### Consistency and Durability

```
// Cassandra
CONSISTENCY LOCAL_QUORUM;

// MongoDB
db.blogs.find().readConcern("majority").writeConcern({ w: "majority" });

// Redis
CONFIG SET appendonly yes
```

- Understand read/write concerns and consistency levels.
- Use quorum-level consistency when correctness matters.
- Configure persistence (AOF/RDB for Redis; commit log for Cassandra; journaling for MongoDB).

#### Performance

```
// MongoDB: create indexes matching query patterns
db.blogs.createIndex({ tags: 1, title: 1 })

// Cassandra: avoid ALLOW FILTERING; design tables per query
SELECT * FROM posts WHERE blog_id = ?;

// Redis: pipelining and batching
var batch = db.CreateBatch();
batch.StringSetAsync("a", "1");
batch.StringSetAsync("b", "2");
batch.Execute();
```

- Index for queries; monitor slow queries.
- Avoid full scans and unbounded reads.
- Use batching and pipelining for throughput.
- Monitor cache hit ratios and eviction.

#### Operations

```
// MongoDB: replica set with backups
mongodump --uri="mongodb://..." --out=/backup

// Cassandra: nodetool repairs and snapshots
nodetool repair
nodetool snapshot

// Redis: Redis Sentinel/Cluster
sentinel monitor mymaster 127.0.0.1 6379 2
```

- Use replication and automatic failover.
- Back up regularly and test restores.
- Monitor disk, memory, latency, and replication lag.
- Plan capacity for growth and failure scenarios.

#### Security

- Enable authentication and TLS.
- Apply least-privilege roles.
- Encrypt data at rest and in transit.
- Audit access and rotate credentials.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: MongoDB Index for Query Pattern

```csharp
var indexKeys = Builders<Blog>.IndexKeys
    .Ascending(b => b.Tags)
    .Ascending(b => b.Title);

await blogs.Indexes.CreateOneAsync(new CreateIndexModel<Blog>(indexKeys));
Console.WriteLine("Index created");
```

**Expected Output**

```
Index created
```

**Why this output occurs**

- The compound index supports queries filtering by `Tags` and sorting by `Title`.
- Index creation returns a result when successful.

---

#### Example 2: Redis Batch for Throughput

```csharp
var batch = db.CreateBatch();
var tasks = new List<Task<bool>>();
for (int i = 0; i < 100; i++)
    tasks.Add(batch.StringSetAsync($"key:{i}", $"value:{i}"));

batch.Execute();
await Task.WhenAll(tasks);
Console.WriteLine("Batch written");
```

**Expected Output**

```
Batch written
```

**Why this output occurs**

- The batch groups commands and reduces round trips.
- `Execute` sends the batch; tasks complete when responses arrive.

---

#### Example 3: Cassandra Query-Driven Table

```csharp
var statement = await session.PrepareAsync(
    "SELECT post_id, title FROM posts_by_blog WHERE blog_id = ? LIMIT 20");
var rows = await session.ExecuteAsync(statement.Bind(blogId));
foreach (var row in rows)
    Console.WriteLine(row.GetValue<string>("title"));
```

**Expected Output (example)**

```
NoSQL Intro
```

**Why this output occurs**

- The table `posts_by_blog` is designed with `blog_id` as the partition key.
- The query uses the partition key, avoiding `ALLOW FILTERING`.

---

### Real-World Cases

- **Polyglot persistence** — Relational for transactions, Redis for caching, MongoDB for content.
- **Multi-region deployments** — Cassandra and Couchbase for geo-distribution.
- **Event-driven systems** — Redis Streams and Kafka for messaging.
- **Caching layers** — Redis in front of relational databases.

---

### References

- MongoDB data modeling best practices - https://www.mongodb.com/docs/manual/data-modeling/
- Redis best practices - https://redis.io/docs/management/optimization/
- Cassandra data modeling best practices - https://cassandra.apache.org/doc/latest/cassandra/data_modeling/
- Couchbase best practices - https://docs.couchbase.com/server/current/manage/manage-servers.html
- CAP theorem (IBM) - https://www.ibm.com/topics/cap-theorem
- NoSQL databases overview - https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Redis 7+ license | RSALv2/SSPLv1; Valkey is the open-source fork. |
| Redis single-threaded commands | Long-running commands block the server; avoid `KEYS` in production. |
| Cassandra `ALLOW FILTERING` | Expensive; avoid in production queries. |
| Cassandra lightweight transactions | Performance cost; use sparingly. |
| MongoDB 16 MB document limit | Design documents within the limit. |
| MongoDB multi-document transactions | Require replica set or sharded cluster; overhead. |
| Couchbase document size limit | 20 MB per document. |
| Couchbase SQL++ indexes | Queries may fail or be slow without appropriate indexes. |
| Cassandra tombstones | Excessive deletes can degrade performance; manage compaction. |
| Redis Cluster multi-key operations | Cross-slot operations are restricted. |
| MongoDB `$lookup` | Limited join support; denormalization preferred. |
| Wide-column schema changes | Some changes require careful migration planning. |

---

## References (Consolidated)

- NoSQL databases overview - https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data
- CAP theorem (IBM) - https://www.ibm.com/topics/cap-theorem
- NoSQL (Wikipedia) - https://en.wikipedia.org/wiki/NoSQL
- MongoDB documentation - https://www.mongodb.com/docs/
- MongoDB .NET driver documentation - https://www.mongodb.com/docs/drivers/csharp/
- MongoDB manual — Data modeling - https://www.mongodb.com/docs/manual/data-modeling/
- MongoDB transactions - https://www.mongodb.com/docs/manual/core/transactions/
- Redis documentation - https://redis.io/docs/
- StackExchange.Redis documentation - https://stackexchange.github.io/StackExchange.Redis/
- Redis data types - https://redis.io/docs/data-types/
- Redis persistence - https://redis.io/docs/management/persistence/
- Valkey (open-source Redis fork) - https://valkey.io/
- Apache Cassandra documentation - https://cassandra.apache.org/doc/latest/
- Cassandra CQL reference - https://cassandra.apache.org/doc/latest/cassandra/cql/
- DataStax C# driver documentation - https://docs.datastax.com/en/developer/csharp-driver/latest/
- Cassandra data modeling - https://cassandra.apache.org/doc/latest/cassandra/data_modeling/
- Couchbase documentation - https://docs.couchbase.com/home/index.html
- Couchbase .NET SDK documentation - https://docs.couchbase.com/dotnet-sdk/current/hello-world/start-using-sdk.html
- SQL++ reference - https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/index.html
- Couchbase data modeling - https://docs.couchbase.com/server/current/data-modeling/data-modeling.html
- Redis best practices - https://redis.io/docs/management/optimization/
- Cassandra data modeling best practices - https://cassandra.apache.org/doc/latest/cassandra/data_modeling/
- Couchbase best practices - https://docs.couchbase.com/server/current/manage/manage-servers.html