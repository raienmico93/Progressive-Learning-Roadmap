# SQL Scalability Engineering: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Scalability engineering is the discipline of designing, tuning, and operating database systems so that they continue to meet performance and availability requirements as workload volume, data size, and user concurrency grow.

**Technical Definition**: Scalability engineering encompasses workload characterization (OLTP vs. OLAP profiling, hotspot detection), connection management (pooling, multiplexing, threading models), query concurrency control (queuing, rate limiting, thundering-herd mitigation), caching strategy design (cache-aside, write-through, invalidation, stampede prevention), read/write traffic separation (application-level and proxy-based routing), and quantitative capacity planning (growth forecasting, IOPS/throughput modeling, network bandwidth estimation).

**Beginner-Friendly Explanation**: Scalability engineering is like designing a highway system that can handle more cars without traffic jams. You need to understand the traffic patterns (workload analysis), build enough on-ramps (connection management), control how many cars enter at once (query concurrency), add shortcuts for common trips (caching), separate express and local lanes (read/write separation), and predict how much road you'll need in five years (capacity planning).

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Workload Awareness** | OLTP systems favor small, random reads/writes; OLAP systems favor large sequential scans |
| **Connection Efficiency** | Pooling and multiplexing reduce per-connection overhead and database memory pressure |
| **Concurrency Control** | Queuing and rate limiting prevent overload; stampede prevention protects against synchronized load spikes |
| **Cache Effectiveness** | Hit ratio, staleness bounds, and invalidation strategy determine database offload |
| **Traffic Routing** | Read/write separation routes SELECTs to replicas and DML to the primary |
| **Capacity Headroom** | Compute, storage, and IOPS must be sized for peak demand, not average |

### Prerequisites

- **Observability Baseline**: Metrics for query latency, throughput, connection count, cache hit ratio, and IOPS consumption
- **Workload Classification**: Understanding of whether the application is OLTP-dominant, OLAP-dominant, or mixed
- **Connection Pooling Layer**: Application-level pools (HikariCP) or proxy-level pooling (RDS Proxy, PgBouncer)
- **Caching Infrastructure**: Redis, Memcached, or application-embedded caches with TTL and invalidation support
- **Read Replica Topology**: Asynchronous replicas with monitoring for replication lag
- **Capacity Model**: Historical growth data and projected transaction volume

### Related Programming Areas

- **Database Administration (DBA)** : Connection limits, replication lag monitoring, capacity provisioning
- **Application Architecture**: Connection pool sizing, cache strategy, read/write routing logic
- **Site Reliability Engineering (SRE)** : SLO/SLI definition, load shedding, autoscaling policies
- **Distributed Systems**: Caching consistency, backpressure, consensus for failover
- **Cloud Engineering**: Instance sizing, IOPS provisioning, autoscaling groups

### Core Concepts Overview

SQL scalability engineering comprises six complementary operational domains:

1. **Workload Analysis**: OLTP vs. OLAP profiling and hotspot identification
2. **Connection Management**: Pooling vs. multiplexing, threading models, max connection limits
3. **Query Concurrency**: Queuing mechanisms, rate limiting, thundering herd mitigation
4. **Caching Strategies**: Cache-aside, write-through, invalidation, stampede prevention
5. **Read/Write Separation**: Application-level and middleware/proxy-based routing
6. **Capacity Planning**: Growth forecasting, IOPS/throughput modeling, network bandwidth limits

---

## Core Concept 1: Workload Analysis

### Definitions

**Core Definition**: Workload analysis is the process of characterizing the types of queries, their frequency, resource consumption, and access patterns that a database system must handle.

**Technical Definition**: Workload analysis involves profiling query mixes to classify them as OLTP (short, high-frequency, single-row operations with sub-50ms latency targets) or OLAP (long-running aggregations across millions of rows), identifying hot keys and hot indexes through metrics such as CPU outliers, latch contention, and sequential I/O wait events, and determining whether a mixed workload (HTAP) requires specialized handling.

**Beginner-Friendly Explanation**: Workload analysis is like studying traffic patterns before designing a road. You want to know: Are most cars making short trips (OLTP) or long hauls (OLAP)? Is there one bridge everyone uses at rush hour (a hot key)? The answers determine how you build the road.

### Purposes

- **To** classify the application as OLTP, OLAP, or mixed to select appropriate database architecture
- **To** identify hot keys and hot indexes that cause contention and limit scalability
- **To** establish performance baselines for capacity planning and anomaly detection
- **To** guide indexing, caching, and partitioning decisions based on actual access patterns

### Key Characteristics: OLTP vs. OLAP

| Characteristic | OLTP | OLAP |
|----------------|------|------|
| **Block size** | ≤ 8K | > 8K |
| **Commit rate** | High | Low |
| **Buffer cache hit ratio** | > 99% | < 99% |
| **Prominent I/O wait events** | `db file sequential read`, `log file sync` | `db file scattered read`, `direct path read` |
| **Average I/O request size** | < 120K | > 400K |
| **Star schema** | Does not apply | Common |

AWS Prescriptive Guidance summarizes these differences: OLTP workloads benefit from smaller block sizes and high buffer cache hit ratios, while OLAP workloads use larger blocks and multi-block reads.

### Syntax Rules and Structure

#### Complete General Syntax (CockroachDB Hotspot Detection)

```sql
-- Check latch conflict wait durations per node
SELECT node_id, value
FROM crdb_internal.node_metrics
WHERE name = 'kv.concurrency.latch_conflict_wait_durations-avg';

-- Check CPU percent per node
SELECT node_id, value
FROM crdb_internal.node_metrics
WHERE name = 'sys.cpu.user.percent';
```

#### Complete General Syntax (TiDB Hotspot Analysis)

```sql
-- Identify top SQL by execution time
SELECT 
    digest,
    query,
    exec_count,
    avg_latency,
    total_keys
FROM information_schema.cluster_statements_summary
ORDER BY total_latency DESC
LIMIT 10;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `latch_conflict_wait_durations` | Time spent waiting for latch conflicts; high values indicate hot keys |
| `sys.cpu.user.percent` | CPU usage per node; outliers indicate uneven load distribution |
| `total_keys` | Number of keys scanned; high values for a single query indicate a hot index |

#### Syntax Rules

- Hotspot detection requires per-node metrics; a cluster-wide average can mask a single hot node
- CPU usage of the hottest node should be compared to the cluster average (20% or more above average indicates a hotspot)
- Latch conflict metrics track time spent waiting to acquire latches on the same key

#### Constraints and Limitations

- Metric granularity depends on the database engine (CockroachDB, TiDB, and GaussDB provide built-in hotspot detection; PostgreSQL and MySQL require manual instrumentation)
- Hot key detection is inherently approximate; Count-Min Sketch and similar algorithms provide frequency estimates with bounded error
- Workload classification is not binary; many applications exhibit HTAP characteristics requiring both row-store and columnar capabilities

### Annotated Complete Step-by-Step Code Examples

#### Example 1: CockroachDB Hotspot Detection Workflow

**Setup**: CockroachDB cluster with monitoring enabled.

```sql
-- Step 1: Check for a node outlier in latch conflict wait durations
SELECT node_id, value AS latch_wait_ms
FROM crdb_internal.node_metrics
WHERE name = 'kv.concurrency.latch_conflict_wait_durations-avg'
ORDER BY value DESC;
```

**Expected Output**:
```
 node_id | latch_wait_ms
---------+---------------
       5 |        152.34
       1 |          2.15
       2 |          1.98
       3 |          2.01
       4 |          1.87
```

```sql
-- Step 2: Check for a "popular key detected" log message
SELECT message, timestamp
FROM crdb_internal.node_log
WHERE message LIKE '%popular key detected%'
ORDER BY timestamp DESC
LIMIT 5;
-- Expected: Shows log entries identifying the hot key range
```

```sql
-- Step 3: Mitigate by using a UUID primary key instead of a sequential key
ALTER TABLE orders ALTER PRIMARY KEY USING COLUMNS (id);
-- Or use hash-sharded index:
ALTER TABLE orders ALTER PRIMARY KEY USING COLUMNS (id) USING HASH;
```

**Why This Output Occurs**: Node 5 shows a latch wait time of 152ms—far above the cluster average of ~2ms. This indicates a hot key or hot index on that node. The "popular key detected" log identifies the specific key causing contention. Using a UUID or hash-sharded index distributes the load across nodes.

#### Example 2: Azure Database for MySQL Capacity Planning Baseline

```sql
-- Step 1: Establish workload baseline metrics
SELECT 
    VARIABLE_NAME,
    VARIABLE_VALUE
FROM performance_schema.global_status
WHERE VARIABLE_NAME IN (
    'Queries', 'Com_select', 'Com_insert', 'Com_update', 'Com_delete',
    'Innodb_buffer_pool_read_requests', 'Innodb_buffer_pool_reads',
    'Threads_connected', 'Threads_running'
);

-- Step 2: Calculate buffer pool hit ratio
SELECT 
    (1 - (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
          WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') /
         (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
          WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests')) * 100 
    AS buffer_hit_ratio;
```

**Expected Output**:
```
+--------------------------+----------------+
| VARIABLE_NAME            | VARIABLE_VALUE |
+--------------------------+----------------+
| Com_select               | 1500000        |
| Com_insert               | 50000          |
| Com_update               | 75000          |
| Com_delete               | 25000          |
| Innodb_buffer_pool_reads | 8000           |
| Innodb_buffer_pool_read_requests | 25000000 |
+--------------------------+----------------+

+-------------------+
| buffer_hit_ratio  |
+-------------------+
|           99.968  |
+-------------------+
```

**Why This Output Occurs**: The buffer hit ratio of 99.97% indicates a well-sized InnoDB buffer pool for the current working set. The read/write ratio (1.5M SELECTs vs. 150K DML) classifies this as a read-heavy OLTP workload.

### Real-World Cases

**Case 1: SaaS Multi-Tenant Hotspot**: A SaaS application experiences slow queries for one large tenant. CockroachDB hotspot detection reveals that the tenant's `tenant_id` is a hot key, concentrating load on a single range. The fix is to hash-shard the index on `tenant_id`.

**Case 2: HTAP Workload Classification**: An application runs both transactional order processing and analytical reporting on the same database. Workload analysis reveals a mixed workload requiring separate OLTP and OLAP instances, with MaxScale routing queries to the appropriate tier.

**Case 3: MySQL Buffer Pool Sizing**: A MySQL database shows a buffer hit ratio of 85%—below the 99% threshold for OLTP. Increasing the InnoDB buffer pool size from 8GB to 24GB raises the hit ratio to 99.5% and reduces disk I/O by 80%.

---

## Core Concept 2: Connection Management

### Definitions

**Core Definition**: Connection management is the set of techniques for controlling how client applications establish, reuse, and pool connections to a database server.

**Technical Definition**: Connection management encompasses connection pooling (maintaining a cache of reusable connections to reduce open/close overhead), connection multiplexing (reusing a single backend database connection for multiple client transactions at the transaction level), threading models (thread-per-connection vs. event-driven vs. thread pool), and configuration of maximum connection limits to prevent database overload.

**Beginner-Friendly Explanation**: Connection management is like managing the checkout lanes at a supermarket. Connection pooling is keeping lanes open and ready so customers don't have to wait for a new lane to open. Multiplexing is letting one cashier serve multiple customers' transactions one after another without closing the register. The threading model determines whether each lane has its own cashier (thread-per-connection) or a smaller team of cashiers rotates between lanes (thread pool).

### Purposes

- **To** reduce the CPU and memory overhead of repeatedly opening and closing database connections
- **To** allow many client applications to share a smaller number of backend database connections
- **To** prevent database overload by capping concurrent connections at safe levels
- **To** enable efficient scaling for serverless and ephemeral compute workloads (Lambda, containers)

### Sub-Concept 2.1: Connection Pooling vs. Multiplexing

#### Definitions

**Connection Pooling**: Reduces the overhead associated with opening and closing connections and with keeping many connections open simultaneously.

**Connection Multiplexing**: Reuses a backend database connection after each transaction. RDS Proxy performs all operations for a transaction using one underlying database connection, then returns it to the pool.

#### Syntax Rules and Structure (AWS RDS Proxy)

```bash
# Create a proxy for connection pooling and multiplexing
aws rds create-db-proxy \
    --db-proxy-name my-proxy \
    --engine-family POSTGRESQL \
    --auth '[{"AuthScheme":"SECRETS","SecretArn":"arn:aws:secretsmanager:..."}]' \
    --role-arn arn:aws:iam::123456789012:role/rds-proxy-role \
    --vpc-subnet-ids subnet-1 subnet-2 \
    --require-tls
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `db-proxy-name` | Name of the proxy |
| `engine-family` | Database engine type |
| `auth` | Authentication configuration (Secrets Manager or IAM) |
| `require-tls` | Enforces TLS for client connections |

#### Syntax Rules

- RDS Proxy automatically determines the current writer instance for Aurora provisioned clusters
- By default, RDS Proxy reuses a connection after each transaction (multiplexing)
- When RDS Proxy cannot be sure it's safe to reuse a connection outside the current session, it keeps the session on the same connection (pinning)
- Pinning reduces the effectiveness of multiplexing; session state changes (temporary tables, prepared statements, session variables) trigger pinning

#### Constraints and Limitations

- Multiplexing works best when applications do not change session state (avoid `SET` statements, temporary tables, and advisory locks)
- Pinning can fragment the connection pool, reducing the effectiveness of multiplexing
- RDS Proxy has a cost per vCPU of the database instance

### Sub-Concept 2.2: Thread-Per-Connection vs. Event-Driven Models

#### Definitions

**Thread-Per-Connection**: Each client connection is handled by a dedicated OS thread (MySQL default) or process (PostgreSQL). As connection count rises, thread/process count rises proportionally.

**Event-Driven (Reactor) Model**: A small number of threads (typically equal to CPU cores) handle all connections using non-blocking I/O and an event loop. PostgreSQL uses a multi-process model—one OS process per client connection. A process-per-client model makes sense for under a thousand connected clients.

#### Syntax Rules and Structure (MySQL Thread Pool)

```sql
-- Enable thread pool plugin (MySQL Enterprise)
INSTALL PLUGIN thread_pool SONAME 'thread_pool.so';

-- Configure thread pool size (number of thread groups)
-- Set at server startup; cannot be changed at runtime
-- In my.cnf: thread_pool_size = 16

-- Configure max concurrent transactions
SET GLOBAL thread_pool_max_transactions_limit = 512;  -- physical cores × 32
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `thread_pool_size` | Number of thread groups; recommended = physical cores (max 512 for InnoDB) |
| `thread_pool_max_transactions_limit` | Upper limit on concurrent transactions; recommended = cores × 32 |
| `thread_pool_query_threads_per_group` | Query threads per group; recommended starting value = 2 |

#### Constraints and Limitations

- PostgreSQL uses a process-per-connection model; each process consumes 5–10MB base memory plus query memory
- MySQL Community Edition uses thread-per-connection; MySQL Enterprise offers the thread pool plugin
- Event-driven drivers (R2DBC, Reactive Mongo) eliminate thread-per-connection and achieve concurrency with a fixed, low number of threads (typically CPU core count)
- Thread pool overhead increases when using smaller thread pool sizes with larger query threads per group

### Annotated Complete Step-by-Step Code Examples

#### Example 1: RDS Proxy Connection Multiplexing

```bash
# Step 1: Create a proxy endpoint
aws rds create-db-proxy \
    --db-proxy-name orders-proxy \
    --engine-family POSTGRESQL \
    --auth '[{"AuthScheme":"SECRETS","SecretArn":"arn:aws:secretsmanager:us-east-1:123456789012:secret:rds!db-xxx"}]' \
    --role-arn arn:aws:iam::123456789012:role/rds-proxy-role \
    --vpc-subnet-ids subnet-abc123 subnet-def456 \
    --require-tls

# Step 2: Verify proxy status
aws rds describe-db-proxies \
    --db-proxy-name orders-proxy \
    --query 'DBProxies[0].{Status:Status,Endpoint:Endpoint}'
```

**Expected Output**:
```json
{
    "Status": "available",
    "Endpoint": "orders-proxy.proxy-abc123.us-east-1.rds.amazonaws.com"
}
```

```sql
-- Step 3: Application connects to the proxy endpoint instead of the database endpoint
-- Instead of:
-- postgresql://user:pass@mydb.abc123.us-east-1.rds.amazonaws.com:5432/mydb
-- Use:
-- postgresql://user:pass@orders-proxy.proxy-abc123.us-east-1.rds.amazonaws.com:5432/mydb
```

**Why This Output Occurs**: The proxy endpoint abstracts the underlying database topology. Applications connect to the proxy, which pools and multiplexes connections to the backend. The application never knows a failover occurred.

#### Example 2: MySQL Thread Pool Configuration

```sql
-- Step 1: Verify thread pool is available
SHOW PLUGINS;
-- Expected: thread_pool | ACTIVE | THREAD POOL | thread_pool.so | GPL

-- Step 2: Check current thread pool status
SHOW STATUS LIKE 'Threadpool%';
-- Expected Output:
-- Threadpool_idle_threads | 4
-- Threadpool_threads | 16
-- Threadpool_rows_fetched | 1250000
```

```ini
# Step 3: Configure thread pool in my.cnf
[mysqld]
thread_pool_size = 16                # 16 physical cores
thread_pool_max_transactions_limit = 512  # 16 × 32
thread_pool_query_threads_per_group = 2
```

**Why This Output Occurs**: The thread pool separates connections from working threads. With 16 thread groups, MySQL limits concurrent query execution to prevent CPU oversubscription. `thread_pool_max_transactions_limit = 512` caps the number of concurrent transactions, protecting the database from overload.

### Real-World Cases

**Case 1: Serverless Connection Exhaustion**: A Lambda-based application opens thousands of database connections, exhausting `max_connections`. RDS Proxy multiplexes these into a small pool of backend connections, allowing the application to scale without connection errors.

**Case 2: PostgreSQL Connection Overhead**: A PostgreSQL database supports 500 connections, each consuming 10MB, totaling 5GB of memory just for connection overhead. Implementing PgBouncer with transaction pooling reduces backend connections to 50, freeing 4.5GB for query processing.

**Case 3: MySQL Thread Pool for High Concurrency**: A MySQL database with 2,000 concurrent connections experiences thread thrashing. Enabling the thread pool plugin with 16 thread groups limits concurrent query execution and improves throughput by 40%.

---

## Core Concept 3: Query Concurrency

### Definitions

**Core Definition**: Query concurrency control manages how many queries execute simultaneously, preventing overload while maximizing throughput.

**Technical Definition**: Query concurrency encompasses queuing mechanisms (bounded queues that hold requests when concurrency limits are reached), rate limiting (token bucket or leaky bucket algorithms that cap request arrival rates), and thundering herd mitigation (preventing synchronized request bursts that overwhelm backend resources).

**Beginner-Friendly Explanation**: Query concurrency is like managing a popular restaurant. You can only seat so many tables at once (concurrency limit). A hostess keeps a waiting list (queue) when all tables are full. A bouncer at the door limits how fast people enter (rate limiting). And when the kitchen runs out of a popular dish and everyone orders it at once (thundering herd), you need a system to prevent the kitchen from being overwhelmed.

### Purposes

- **To** prevent database overload by limiting the number of concurrently executing queries
- **To** protect backend resources during traffic spikes through queuing and backpressure
- **To** enforce fair resource allocation across tenants or clients through per-tenant rate limits
- **To** prevent cache stampedes and thundering herd events from overwhelming the database

### Sub-Concept 3.1: Queuing Mechanisms

#### Definitions

**Bounded Queue**: A queue with a fixed capacity that holds pending requests when all workers are busy. When the queue is full, new requests are rejected (load shedding).

#### Syntax Rules and Structure (Java — Semaphore-Based Concurrency Limit)

```java
// Limit concurrent database operations to 10
Semaphore dbSemaphore = new Semaphore(10);

public Result queryDatabase(String sql) throws InterruptedException {
    dbSemaphore.acquire();  // Wait for a permit
    try {
        return executeQuery(sql);
    } finally {
        dbSemaphore.release();  // Return the permit
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `Semaphore(10)` | Creates a semaphore with 10 permits (max 10 concurrent operations) |
| `acquire()` | Blocks until a permit is available |
| `release()` | Returns the permit to the pool |

#### Constraints and Limitations

- The connection pool's wait queue is itself a backpressure point; tuning the worker pool to 200 threads while the connection pool has 10 connections simply moves the queue into the connection pool
- Queue capacity must be bounded; unbounded queues lead to memory exhaustion and cascading failures
- Queue utilization should be monitored; high utilization indicates the need for more capacity

### Sub-Concept 3.2: Rate Limiting

#### Definitions

**Token Bucket**: A rate-limiting algorithm where tokens are added to a bucket at a fixed rate. Each request consumes a token. If no tokens are available, the request is rejected or queued.

#### Syntax Rules and Structure (Redis Rate Limiting)

```lua
-- Redis Lua script for sliding window rate limiting
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current = redis.call('INCR', key)
if current == 1 then
    redis.call('EXPIRE', key, window)
end
if current > limit then
    return 0  -- Rate limit exceeded
end
return 1  -- Request allowed
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `KEYS[1]` | Rate limit key (e.g., `rate_limit:user:123`) |
| `ARGV[1]` | Maximum requests per window |
| `ARGV[2]` | Window duration in seconds |
| `INCR` | Atomically increments the counter |
| `EXPIRE` | Sets TTL on first request in the window |

#### Constraints and Limitations

- Rate limiting at the application layer must be coordinated across instances; Redis provides a shared counter
- Token bucket allows bursts; sliding window provides smoother limiting
- Rate limits should be per-tenant or per-client to prevent one client from consuming all capacity

### Sub-Concept 3.3: Thundering Herd Mitigation

#### Definitions

**Thundering Herd**: A failure mode where many clients or threads simultaneously request the same resource after a shared trigger (cache expiration, service restart, flash sale), overwhelming the backend. In caching systems, this is called a cache stampede.

**Single-Flight Mutex Lock**: A pattern where exactly one request recomputes a missing cache key while every other concurrent request for that same key waits for the result or serves a stale copy.

#### Syntax Rules and Structure (Single-Flight with Redis)

```python
import redis
import json

r = redis.Redis()

def get_data(key):
    # Try cache first
    cached = r.get(key)
    if cached:
        return json.loads(cached)
    
    # Acquire single-flight lock
    lock_key = f"lock:{key}"
    acquired = r.set(lock_key, "1", nx=True, ex=10)  # 10-second lock
    
    if acquired:
        try:
            # This request recomputes the value
            data = query_database(key)
            r.setex(key, 3600, json.dumps(data))
            return data
        finally:
            r.delete(lock_key)
    else:
        # Another request is recomputing; wait briefly and retry
        time.sleep(0.1)
        return get_data(key)
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `r.set(lock_key, "1", nx=True, ex=10)` | Acquires a lock only if it doesn't exist (NX), with 10-second expiry |
| `r.setex(key, 3600, data)` | Caches the result with a 1-hour TTL |
| `r.delete(lock_key)` | Releases the lock |

#### Constraints and Limitations

- Single-flight adds a small lock round-trip on cache miss
- If the lock holder crashes, the lock expires after the TTL, allowing another request to proceed
- TTL jitter (adding random variation to TTLs) prevents synchronized expiration as a complementary strategy
- Probabilistic early expiration (XFetch) refreshes keys slightly before their TTL using `delta * beta * -log(random)`, spreading recomputation over time

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Redis Single-Flight Cache Stampede Prevention

```python
import redis
import json
import time
import random

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def get_product(product_id):
    cache_key = f"product:{product_id}"
    
    # Step 1: Try cache
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Step 2: Attempt to acquire single-flight lock
    lock_key = f"lock:product:{product_id}"
    acquired = r.set(lock_key, "1", nx=True, ex=5)  # 5-second lock
    
    if acquired:
        try:
            # Step 3: This request recomputes the value
            time.sleep(2)  # Simulate expensive database query
            product = {"id": product_id, "name": f"Product {product_id}", "price": 99.99}
            
            # Step 4: Cache with TTL jitter (3600 ± 300 seconds)
            ttl = 3600 + random.randint(-300, 300)
            r.setex(cache_key, ttl, json.dumps(product))
            return product
        finally:
            r.delete(lock_key)
    else:
        # Step 5: Another request is recomputing; wait and retry
        time.sleep(0.2)
        return get_product(product_id)

# Simulate 100 concurrent requests for the same product
import threading
results = []
def worker():
    results.append(get_product(42))

threads = [threading.Thread(target=worker) for _ in range(100)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Total results: {len(results)}")
print(f"Database queries executed: 1 (only the lock holder queries)")
```

**Expected Output**:
```
Total results: 100
Database queries executed: 1 (only the lock holder queries)
```

**Why This Output Occurs**: The first request acquires the single-flight lock and performs the database query. The other 99 requests find the lock held and wait briefly, then retrieve the cached result. Only one database query is executed instead of 100.

### Real-World Cases

**Case 1: Facebook Memcache Leases**: Facebook's memcache implementation uses cache leases to cut peak database query rates during stampedes from 17,000 QPS to 1,300 QPS—a 13x reduction.

**Case 2: Razorpay Flash Sale**: Razorpay handles flash sales at 1,500 requests per second by rate limiting traffic, connection pooling, and avoiding the thundering herd through staggered reloading.

**Case 3: Redis Rate Limiting for API Protection**: An API gateway uses Redis-based sliding window rate limiting to cap each client at 100 requests per second, preventing any single client from overwhelming the database.

---

## Core Concept 4: Caching Strategies

### Definitions

**Core Definition**: Caching strategies define how application data is stored, retrieved, invalidated, and refreshed in a cache layer to reduce database load and improve read latency.

**Technical Definition**: Caching strategies encompass cache-aside (application manages cache reads and writes explicitly), write-through (cache is updated synchronously on every write), write-behind (writes are buffered and applied asynchronously), cache invalidation (deleting or updating cache entries on write), and stampede prevention (single-flight, TTL jitter, probabilistic early expiration).

**Beginner-Friendly Explanation**: Caching is like keeping frequently used items on your desk instead of walking to the filing cabinet every time. Cache-aside means you check your desk first, and if the item isn't there, you get it from the cabinet and put a copy on your desk. Write-through means every time you update the master file, you also update your desk copy. Invalidation means throwing away the desk copy when the master changes.

### Purposes

- **To** reduce database read load by serving repeated queries from a low-latency cache
- **To** improve P95 read latency for read-heavy workloads (product catalogs, user profiles)
- **To** maintain bounded staleness through TTL-based expiration and explicit invalidation
- **To** prevent cache stampedes that would otherwise overwhelm the database on key expiration

### Sub-Concept 4.1: Cache-Aside (Lazy Loading)

#### Syntax Rules and Structure

```python
def get_user(user_id):
    cache_key = f"user:{user_id}"
    
    # 1. Check cache
    cached = cache.get(cache_key)
    if cached:
        return cached
    
    # 2. Cache miss: query database
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    
    # 3. Write result to cache with TTL
    cache.setex(cache_key, 3600, user)  # 1-hour TTL
    
    return user
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `cache.get(cache_key)` | Attempts to retrieve the cached value |
| `db.query(...)` | Fallback database query on cache miss |
| `cache.setex(cache_key, ttl, value)` | Stores the result with a bounded TTL |

#### Constraints and Limitations

- The first read after a cache miss or invalidation is always slow (hits the database)
- Cache-aside works well for read-heavy workloads; write-through is better when consistency is more important
- Stale data windows are bounded by the TTL

### Sub-Concept 4.2: Write-Through Caching

#### Definitions

**Write-Through**: Every write to the database also updates the cache synchronously, keeping the cache always consistent with the database.

#### Syntax Rules and Structure

```python
def update_user(user_id, data):
    # 1. Write to database (source of truth)
    db.execute("UPDATE users SET name = %s WHERE id = %s", data['name'], user_id)
    
    # 2. Update cache synchronously
    cache.setex(f"user:{user_id}", 3600, data)
    
    # 3. Invalidate related caches if needed
    cache.delete("user_list")
```

#### Constraints and Limitations

- Write-through adds latency to every write operation (cache update is synchronous)
- Only useful if the written data is likely to be read soon
- Does not protect against stale data if the cache update fails after the database write

### Sub-Concept 4.3: Cache Invalidation

#### Definitions

**Cache Invalidation**: The process of removing or updating cache entries when the underlying data changes.

#### Syntax Rules and Structure

```python
def update_product(product_id, data):
    # 1. Update database first (source of truth)
    db.execute("UPDATE products SET price = %s WHERE id = %s", data['price'], product_id)
    
    # 2. Invalidate cache (delete the key)
    cache.delete(f"product:{product_id}")
    
    # 3. Optionally update the cache immediately
    # cache.setex(f"product:{product_id}", 3600, get_product(product_id))
```

#### Constraints and Limitations

- Invalidation must happen after the database write, not before
- If invalidation fails, the cache serves stale data until TTL expires
- Version numbers or timestamps can prevent older writes from overwriting newer data

### Sub-Concept 4.4: Cache Stampede Prevention

#### Mitigation Strategies

| Strategy | How It Works | Best For | Trade-off |
|----------|--------------|----------|-----------|
| TTL jitter | Add random spread to each TTL | Every cache write, by default | Does not help a single hyper-hot key |
| Single-flight mutex | One request holds a lock and recomputes; others wait or serve stale | Correctness-critical keys | Adds a small lock round-trip on miss |
| Probabilistic early expiration (XFetch) | Each reader may refresh just before expiry with rising probability | Very hot keys read constantly | Slightly more recomputes overall |
| Stale-while-revalidate | Serve old value, refresh in background | Reads that tolerate brief staleness | Not safe for revoked tokens |

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Redis Cache-Aside with TTL and Invalidation

```python
import redis
import json

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def get_product(product_id):
    cache_key = f"cache:product:{product_id}"
    
    # Step 1: Read from cache
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Step 2: Cache miss — query database
    product = db.query("SELECT * FROM products WHERE id = %s", product_id)
    
    # Step 3: Write to cache with TTL (3600 seconds)
    r.setex(cache_key, 3600, json.dumps(product))
    
    return product

def update_product(product_id, new_price):
    # Step 1: Update database (source of truth)
    db.execute("UPDATE products SET price = %s WHERE id = %s", new_price, product_id)
    
    # Step 2: Invalidate cache key
    r.delete(f"cache:product:{product_id}")
    # Next read will repopulate from database
```

**Why This Output Occurs**: The cache-aside pattern reads from Redis first. On miss, it queries the database and stores the result with a 1-hour TTL. On update, the database is updated first, then the cache key is deleted. The next read repopulates the cache with fresh data.

#### Example 2: TTL Jitter for Stampede Prevention

```python
import random

def set_with_jitter(key, value, base_ttl=3600, jitter_range=300):
    """Set a cache key with randomized TTL to prevent synchronized expiration."""
    ttl = base_ttl + random.randint(-jitter_range, jitter_range)
    r.setex(key, ttl, value)

# Without jitter: 1000 keys set at 10:00:00 all expire at 11:00:00
# With jitter: keys expire between 10:55:00 and 11:05:00
for i in range(1000):
    set_with_jitter(f"key:{i}", f"value:{i}")
```

**Why This Output Occurs**: TTL jitter spreads expiration times across a 10-minute window, preventing all 1,000 keys from expiring simultaneously and causing a thundering herd of cache misses.

### Real-World Cases

**Case 1: Product Catalog Cache-Aside**: An e-commerce platform caches product details with a 1-hour TTL. Cache hit ratio is 95%, reducing database read load by 95%.

**Case 2: Write-Through for User Profiles**: A social media application uses write-through caching for user profiles, ensuring the cache is always consistent with the database. Read latency for profile pages drops from 50ms to 2ms.

**Case 3: Single-Flight for JWKS Keys**: An authentication service uses single-flight locking for JWKS public key caching. When the cache expires, only one request fetches new keys while others wait, preventing a stampede on the identity provider.

---

## Core Concept 5: Read/Write Separation

### Definitions

**Core Definition**: Read/write separation routes read-only queries to replicas and write queries to the primary database, distributing workload and improving scalability.

**Technical Definition**: Read/write separation can be implemented at the application level (explicit routing logic in application code) or at the middleware/proxy level (transparent routing by tools like Vitess, MaxScale, or ProxySQL). Middleware-based routing uses statement-based analysis to determine whether a query is a read or a write, routing it to the appropriate backend.

**Beginner-Friendly Explanation**: Read/write separation is like having separate lines at a bank—one for deposits and withdrawals (writes) and one for balance inquiries (reads). The teller at the inquiry line can answer questions much faster because they don't have to handle complex transactions.

### Purposes

- **To** offload read-only queries from the primary database to replicas
- **To** scale read throughput linearly by adding replicas
- **To** isolate analytical or reporting queries from transactional workloads
- **To** enable transparent failover and routing without application changes (middleware approach)

### Sub-Concept 5.1: Application-Level Routing

#### Syntax Rules and Structure

```python
# Python application-level read/write routing
def get_connection(query_type):
    if query_type == 'write':
        return primary_pool.get_connection()
    else:
        return replica_pool.get_connection()

# Usage
def get_user(user_id):
    conn = get_connection('read')
    return conn.query("SELECT * FROM users WHERE id = %s", user_id)

def update_user(user_id, name):
    conn = get_connection('write')
    return conn.execute("UPDATE users SET name = %s WHERE id = %s", name, user_id)
```

#### Constraints and Limitations

- Applications must be aware of read-after-write consistency requirements
- Replica lag can cause stale reads; applications must tolerate eventual consistency or route reads to the primary for a bounded period after writes
- Each replica has its own connection endpoint; the application must manage multiple connection pools

### Sub-Concept 5.2: Middleware/Proxy-Based Routing

#### Vitess Routing Rules

Vitess routing rules are stored in the topo under `global/routingrules`. They contain a list of table-specific routes that direct query traffic to the right keyspaces, shards, and tablet types.

```json
{
  "rules": [
    {
      "from_table": "commerce.customer@replica",
      "to_tables": ["commerce.customer"]
    },
    {
      "from_table": "commerce.corder@rdonly",
      "to_tables": ["commerce.corder"]
    }
  ]
}
```

#### MaxScale Read/Write Split

MaxScale's Read/Write Split router is a statement-based router designed for Master/Slave replication environments. It routes writes to the primary and reads to replicas. When causal reads are enabled, MaxScale uses global transaction IDs (GTIDs) to enforce read-your-writes consistency.

#### Syntax Rules and Structure (MaxScale Configuration)

```ini
[Read-Write-Service]
type=service
router=readwritesplit
servers=server1,server2,server3
user=maxscale
password=secret
causal_reads=true

[Read-Only-Service]
type=service
router=readconnroute
router_options=slave
servers=server2,server3,server4
user=maxscale
password=secret
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `readwritesplit` | Statement-based router for read/write splitting |
| `causal_reads` | Enables GTID-based read-your-writes consistency |
| `readconnroute` | Connection-based router for read-only traffic |
| `router_options=slave` | Routes to replica servers only |

#### Constraints and Limitations

- Statement-based routing may misclassify queries (e.g., `SELECT ... FOR UPDATE` is a write)
- Causal reads add latency to reads that must wait for GTID propagation
- Middleware adds a network hop and must be highly available

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MaxScale Read/Write Splitting

```ini
# Step 1: Configure MaxScale with read/write split
# /etc/maxscale.cnf
[Read-Write-Service]
type=service
router=readwritesplit
servers=mysql-primary,mysql-replica1,mysql-replica2
user=maxscale
password=secret
causal_reads=true

[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=MariaDBClient
port=4006
```

```sql
-- Step 2: Application connects to MaxScale on port 4006
-- All queries go to MaxScale; it routes automatically
INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
-- MaxScale routes this to mysql-primary

SELECT * FROM orders WHERE customer_id = 123;
-- MaxScale routes this to mysql-replica1 or mysql-replica2
```

**Why This Output Occurs**: MaxScale analyzes each statement. INSERT, UPDATE, DELETE, and DDL go to the primary. SELECT statements go to the least-loaded replica. With `causal_reads=true`, a SELECT immediately after an INSERT waits for the replica to catch up to the primary's GTID, ensuring read-your-writes consistency.

#### Example 2: Vitess Routing Rules During Migration

```bash
# Step 1: View current routing rules
vtctldclient --server=localhost:15999 GetRoutingRules
```

**Expected Output**:
```json
{
  "rules": [
    {
      "from_table": "customer.customer",
      "to_tables": ["commerce.customer"]
    }
  ]
}
```

```bash
# Step 2: Switch reads to the new keyspace
vtctldclient --server=localhost:15999 SwitchReads \
    --keyspace=commerce \
    --tablet-types=replica,rdonly \
    customer

# Step 3: Verify routing rules updated
vtctldclient --server=localhost:15999 GetRoutingRules
# Now reads route to customer keyspace, writes still go to commerce
```

**Why This Output Occurs**: Vitess routing rules enable gradual migration. `SwitchReads` updates the rules so read queries go to the new keyspace while writes continue to the old keyspace. This allows validation before switching writes.

### Real-World Cases

**Case 1: E-Commerce Read Scaling**: An e-commerce platform uses MaxScale to route product catalog queries to three read replicas. Primary CPU drops from 80% to 25%, and read throughput triples.

**Case 2: Vitess Migration**: A company migrates from a monolithic MySQL database to a sharded Vitess cluster. Routing rules allow reads to be switched first, validated, then writes—with zero downtime.

**Case 3: Causal Reads for Consistency**: A social media application uses MaxScale causal reads to ensure users see their own posts immediately after posting, while other reads are served from replicas.

---

## Core Concept 6: Capacity Planning

### Definitions

**Core Definition**: Capacity planning is the process of forecasting future resource requirements and ensuring that compute, storage, and I/O capacity meet demand.

**Technical Definition**: Capacity planning encompasses growth forecasting (projecting data size and transaction volume based on historical trends), IOPS/throughput modeling (calculating read/write IOPS requirements from transaction volume and read/write ratios), and network bandwidth estimation (determining the bandwidth required for replication, client connections, and backup traffic).

**Beginner-Friendly Explanation**: Capacity planning is like planning a highway expansion. You look at current traffic, project how many more cars will use the road each year, and build enough lanes to handle peak rush hour—not just the average.

### Purposes

- **To** ensure that compute, storage, and IOPS capacity meet peak demand
- **To** forecast when additional resources will be required
- **To** prevent performance degradation from resource saturation
- **To** optimize cost by right-sizing instances without over-provisioning

### Sub-Concept 6.1: Growth Forecasting

#### Definitions

**Growth Rate**: The average increase in database size or transaction volume per day, week, or month.

#### Syntax Rules and Structure (SQL Server)

```sql
-- Capture weekly database size snapshots
SELECT 
    DB_NAME(database_id) AS db_name,
    SUM(size) * 8 / 1024 AS total_mb,
    GETDATE() AS captured_at
FROM sys.master_files
GROUP BY database_id;
```

#### Constraints and Limitations

- Growth is non-linear; seasonal spikes must be accounted for
- Storage can only be scaled up, not down (Azure Database for MySQL)
- Include failover capacity in planning; replicas need equal or greater resources than the source

### Sub-Concept 6.2: IOPS and Throughput Modeling

#### Definitions

**IOPS**: Input/Output Operations Per Second—the number of read/write operations the storage subsystem can handle.

**Throughput**: The amount of data transferred per second (MB/s or GB/s).

#### Calculation Formula

```
IOPS = Read I/O requests + Write I/O requests
Throughput = Average I/O request size × IOPS
```

#### Syntax Rules and Structure (AWS RDS Oracle AWR)

From an AWR report, calculate IOPS and throughput:

```
IOPS = Read I/O requests + Write I/O requests
     = 3,586.8 + 574.7 = 4,134.5 IOPS

Throughput = Average I/O request size × IOPS
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `Read I/O requests` | Number of read operations per second |
| `Write I/O requests` | Number of write operations per second |
| `Average I/O request size` | Average size of each I/O operation (KB) |

#### Constraints and Limitations

- IOPS limits scale with compute size; larger instances have higher IOPS ceilings
- Autoscale IOPS provides automatic I/O scaling based on demand
- Network bandwidth can become the bottleneck before IOPS (e.g., cross-region replication)

### Sub-Concept 6.3: Network Bandwidth Estimation

#### Definitions

**Network Bandwidth**: The maximum rate of data transfer between the database and clients, replicas, or backup storage.

#### Constraints and Limitations

- Synchronous replication bandwidth must accommodate the peak WAL generation rate
- Cross-region replication incurs WAN latency and bandwidth costs
- Backup operations consume bandwidth; schedule backups during low-traffic periods

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Azure MySQL Capacity Planning Checklist

```sql
-- Step 1: Establish baseline metrics
SELECT 
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Com_select') AS reads,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Com_insert' + 'Com_update' + 'Com_delete') AS writes,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Threads_connected') AS connections,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') AS disk_reads,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests') AS buffer_reads;
```

```sql
-- Step 2: Calculate IOPS requirements
-- From Azure Monitor metrics:
-- Storage IO Count = 5000 IOPS at peak
-- Storage IO Percent = 85%
-- Required IOPS = 5000 / 0.85 = 5882 IOPS

-- Step 3: Select appropriate compute tier
-- General Purpose: balanced compute/memory, supports HA and read replicas
-- Memory Optimized: higher memory-to-vCore ratio for cache-dependent workloads
```

**Why This Output Occurs**: Capacity planning starts with baseline metrics: read/write ratio, connection count, and buffer pool efficiency. IOPS requirements are calculated from peak demand with headroom (85% utilization target). The compute tier is selected based on whether the workload is cache-dependent (Memory Optimized) or balanced (General Purpose).

#### Example 2: Growth Forecasting with Linear Regression

```python
import numpy as np
from datetime import datetime, timedelta

# Historical database sizes (GB) from weekly snapshots
sizes = [100, 105, 112, 118, 127, 135, 148]
weeks = np.arange(len(sizes))

# Fit linear regression
slope, intercept = np.polyfit(weeks, sizes, 1)
growth_per_week = slope  # GB per week

# Forecast when storage will be exhausted
current_size = sizes[-1]
storage_limit = 500  # GB
weeks_until_full = (storage_limit - current_size) / growth_per_week

print(f"Growth rate: {growth_per_week:.2f} GB/week")
print(f"Current size: {current_size} GB")
print(f"Weeks until {storage_limit} GB: {weeks_until_full:.0f}")
print(f"Forecast date: {datetime.now() + timedelta(weeks=weeks_until_full)}")
```

**Expected Output**:
```
Growth rate: 7.82 GB/week
Current size: 148 GB
Weeks until 500 GB: 45
Forecast date: 2027-08-25
```

**Why This Output Occurs**: Linear regression on historical size data provides a growth rate. The forecast predicts that storage will reach the 500GB limit in 45 weeks, triggering a procurement or scaling action.

### Real-World Cases

**Case 1: Azure MySQL Right-Sizing**: A company runs capacity planning on Azure Database for MySQL. Baseline metrics show 5,000 IOPS at peak and a 99.5% buffer hit ratio. The DBA selects General Purpose tier with 8 vCores and 512GB storage, providing headroom for 12 months of growth.

**Case 2: IOPS Modeling from AWR**: An Oracle DBA uses AWR reports to calculate that the database requires 4,134 IOPS at peak. The storage subsystem is provisioned for 6,000 IOPS (45% headroom) to handle seasonal spikes.

**Case 3: Network Bandwidth for Cross-Region Replication**: A company replicates data from `us-east` to `eu-west`. WAL generation peaks at 50 MB/s. The network link must support at least 100 MB/s (2x headroom) for replication plus backup traffic.

---

## References

| Name | Link |
|------|------|
| AWS Prescriptive Guidance — Workload characteristics | https://docs.aws.amazon.com/prescriptive-guidance/latest/oracle-exadata-blueprint/workload-characteristics.html |
| Microsoft Learn — Determining application type (Citus) | https://learn.microsoft.com/en-us/postgresql/citus/app-type |
| CockroachDB — Detect Hotspots | https://docs.cockroachlabs.com/docs/v26.1/detect-hotspots |
| TiDB — Performance Hotspots: Fixing Issues with Top SQL | https://www.pingcap.com/blog/tidb-performance-hotspots/ |
| AWS — RDS Proxy concepts and terminology | https://docs.aws.amazon.com/en_en/AmazonRDS/latest/AuroraUserGuide/rds-proxy.howitworks.html |
| PostgreSQL — Reasoning behind process instead of thread based arch | https://www.postgresql.org/message-id/20041027170139.GA61271@winnie.fuhr.org |
| MySQL 8.0 Reference Manual — Thread Pool Tuning | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/thread-pool-tuning.html |
| Redis — How to tame the thundering herd problem | https://redis.io/blog/how-to-tame-the-thundering-herd-problem |
| Security Boulevard — Mitigating Thundering Herd Problems in Distributed Auth Caching | https://securityboulevard.com/2026/06/mitigating-thundering-herd-problems-in-distributed-auth-caching/ |
| Redis — Cache-aside pattern | https://redis.io/docs/latest/develop/use-cases/cache-aside/ |
| Vitess — Schema Routing Rules | https://vitess.io/docs/24.0/reference/features/schema-routing-rules/ |
| MariaDB — MaxScale datasheet | https://mariadb.com/wp-content/uploads/2019/03/mariadb-maxscale_datasheet_1014.pdf |
| Microsoft Learn — Architecture Best Practices for Azure Database for MySQL | https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-database-for-mysql |
| AWS — Estimate the Amazon RDS engine size for an Oracle database by using AWR reports | https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/estimate-amazon-rds-engine-size-oracle.html |
| Redis — Thundering herd problem | https://redis.io/blog/how-to-tame-the-thundering-herd-problem |
| GitHub — Distributed Systems: Backpressure | https://github.com/gw-dg/distributed-system-docs/blob/main/08-distributed-systems/backpressure.md |
| Redis — Cache stampede prevention | https://redis.io/docs/latest/develop/use-cases/cache-aside/ |