# SQL Horizontal and Vertical Scaling: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Horizontal and vertical scaling are the two fundamental strategies for increasing database capacity—vertical scaling adds more power to a single server, while horizontal scaling adds more servers to distribute the workload.

**Technical Definition**: Vertical scaling (scale-up) increases the compute resources (CPU, RAM, storage, I/O) of an individual database server instance. Horizontal scaling (scale-out) adds additional database nodes to a cluster, distributing data and query load across multiple machines through replication, sharding, or shared-nothing architectures. Azure SQL Database supports both models: vertical scaling via elastic database pools (scaling up individual databases) and horizontal scaling via sharding (partitioning data across multiple database nodes).

**Beginner-Friendly Explanation**: Vertical scaling is like upgrading from a small car to a bigger truck—same driver, more capacity. Horizontal scaling is like adding more cars to a delivery fleet—more drivers, each carrying part of the load. The first has limits (you can only buy so big a truck); the second is more flexible but requires coordination between drivers.

### Key Characteristics

| Characteristic | Vertical Scaling | Horizontal Scaling |
|----------------|------------------|---------------------|
| **Approach** | Bigger machine | More machines |
| **Primary Limit** | Hardware ceiling | Coordination complexity |
| **Downtime** | Often required during resize | Minimal if properly architected |
| **Cost Model** | Exponential at high end | Linear with commodity hardware |
| **State Management** | Single node, simple | Distributed, complex |
| **Use Case** | OLTP with moderate growth | Web-scale, multi-tenant, analytics |

### Prerequisites

- **Vertical Scaling**: Access to higher-spec instance types, maintenance window for restart, awareness of storage-only-increase constraints
- **Horizontal Scaling**: Distributed architecture design, shard key selection, load balancing layer, distributed transaction support
- **Read Scaling**: Read replica topology, replica lag monitoring, query routing logic
- **Write Scaling**: Sharding or consensus-based replication, distributed coordination protocol
- **Distributed SQL**: Consensus algorithm (Raft/Paxos), distributed transaction manager (2PC), CAP trade-off understanding

### Related Programming Areas

- **Distributed Systems**: CAP theorem, consensus algorithms, partition tolerance
- **Database Administration (DBA)** : Instance resizing, replica management, shard rebalancing
- **Cloud Engineering**: Managed scaling services (RDS, Azure SQL, Cloud SQL), auto-scaling policies
- **Application Development**: Stale-read handling, retry patterns, partition-aware queries
- **Site Reliability Engineering (SRE)** : Capacity planning, scaling runbooks, split-brain prevention

### Core Concepts Overview

SQL horizontal and vertical scaling comprises seven complementary techniques:

1. **Vertical Scaling**: Adding CPU, memory, and storage to a single database instance
2. **Horizontal Scaling**: Adding nodes and distributing state across the cluster
3. **Read Scaling**: Replica pools and load balancing for read-heavy workloads
4. **Write Scaling**: ACID challenges and distributed coordination for write-heavy workloads
5. **Sharding Concepts**: Shard key selection, algorithmic vs. directory-based routing, re-sharding
6. **Distributed SQL Considerations**: CAP theorem trade-offs, consensus (Raft/Paxos), distributed transactions (2PC)

---

## Core Concept 1: Vertical Scaling

### Definitions

**Core Definition**: Vertical scaling (scale-up) increases the resources of a single database server to handle greater workload.

**Technical Definition**: Vertical scaling involves provisioning a higher-spec instance—more vCPUs, memory, storage, and I/O throughput—without changing the logical topology of the database deployment. In Azure Database for PostgreSQL Flexible Server, vertical scaling allows independent changes to vCores, storage size, and backup retention period. The number of vCores can be scaled up or down, but storage size can only be increased.

**Beginner-Friendly Explanation**: Vertical scaling is like upgrading your computer's RAM and CPU to make it faster. You're not adding more computers—you're making the one you have more powerful. The downside is there's a limit to how much you can upgrade, and you often need to restart the machine.

### Purposes

- **To** increase throughput for a single-node database without architectural changes
- **To** handle short-term traffic spikes with rapid resource adjustments
- **To** avoid the complexity of distributed coordination and sharding
- **To** extend the useful life of a well-designed single-node database

### Syntax Rules and Structure

#### Complete General Syntax (Azure Database for PostgreSQL Flexible Server)

```bash
# Vertical scaling via Azure CLI
az postgres flexible-server update \
    --resource-group myResourceGroup \
    --name myserver \
    --sku-name Standard_D4s_v3 \
    --storage-size 512
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `--sku-name` | Compute tier and VM size (e.g., `Standard_D4s_v3` = 4 vCores, 16 GB RAM) |
| `--storage-size` | Storage size in GB (can only increase) |
| `--tier` | Compute tier: Burstable, General Purpose, or Memory Optimized |

#### Complete General Syntax (Azure SQL Database)

```sql
-- Change service objective (vertical scaling)
ALTER DATABASE mydatabase
MODIFY (SERVICE_OBJECTIVE = 'P4');
```

#### Syntax Rules

- Storage size can only be increased, never decreased (Azure Database for PostgreSQL)
- A server restart is required when changing vCores or compute tier; typically takes 2–10 minutes
- Near-zero downtime scaling reduces restart to under 30 seconds by creating a synchronized copy of the server

#### Constraints and Limitations

- **Hardware ceiling**: Each cloud provider has maximum instance sizes (e.g., Azure PostgreSQL up to 64 vCores)
- **Downtime**: Changing compute tier requires a restart; storage scaling is online in most cases
- **Storage asymmetry**: Storage can only be increased, not decreased (Azure PostgreSQL)
- **Cost**: Higher-spec instances cost disproportionately more per unit of compute
- **Failure domain**: A single larger server is still a single point of failure

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Azure PostgreSQL Vertical Scaling with Near-Zero Downtime

**Setup**: Azure Database for PostgreSQL Flexible Server with 2 vCores.

```bash
# Step 1: Check current configuration
az postgres flexible-server show \
    --resource-group myRG \
    --name myserver \
    --query "{sku:sku.name, storage:storage.storageSizeGb, tier:sku.tier}"
# Expected: {"sku": "Standard_D2s_v3", "storage": 128, "tier": "GeneralPurpose"}
```

```bash
# Step 2: Scale up to 8 vCores with near-zero downtime
az postgres flexible-server update \
    --resource-group myRG \
    --name myserver \
    --sku-name Standard_D8s_v3 \
    --storage-size 256

# Expected: Operation completes with <30 seconds downtime
# The server creates a new VM, synchronizes it, and switches over
```

**Expected Output**:
```json
{
  "sku": {
    "name": "Standard_D8s_v3",
    "tier": "GeneralPurpose"
  },
  "storage": {
    "storageSizeGb": 256
  },
  "state": "Ready"
}
```

**Why This Output Occurs**: Azure provisions a new VM with the higher specification, synchronizes it with the existing server, and switches to the new copy with a 30-second interruption. The old server is then retired.

### Real-World Cases

**Case 1: Rapid Traffic Spike**: An e-commerce platform experiences a flash sale. The DBA scales the database vertically from 4 to 16 vCores, handling the surge without sharding complexity.

**Case 2: Growing SaaS Application**: A SaaS startup begins with a 2-vCore PostgreSQL instance. As the customer base grows, it scales vertically to 8, then 16, then 32 vCores—deferring horizontal scaling until the application outgrows a single machine.

**Case 3: SQL Server Enterprise Edition**: A company uses SQL Server Enterprise Edition to scale vertically to 128 cores and 4 TB RAM, handling a mission-critical ERP database without sharding.

---

## Core Concept 2: Horizontal Scaling

### Definitions

**Core Definition**: Horizontal scaling (scale-out) adds more database nodes to a cluster, distributing data and workload across multiple servers.

**Technical Definition**: Horizontal scaling involves adding worker nodes to a database cluster, partitioning data across shards, and distributing query load. Stateless application instances behind a load balancer can scale horizontally without code changes; stateful database nodes require replication, sharding, or shared-nothing architectures. A common anti-pattern is horizontal scaling a stateful service without leader election—running two database primaries without coordination is split-brain and corrupts data.

**Beginner-Friendly Explanation**: Horizontal scaling is like adding more checkout lanes at a supermarket. Each lane (node) can handle customers independently, so the store serves more people. But you need a system to direct customers to the right lane and handle shared resources like the cash register.

### Purposes

- **To** scale beyond the hardware limits of a single server
- **To** distribute workload across commodity hardware for cost efficiency
- **To** achieve fault tolerance through redundancy across multiple nodes
- **To** support geographic distribution for latency reduction and compliance

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL with Citus — Sharding)

```sql
-- Add a worker node to a Citus cluster
SELECT master_add_node('worker-1.example.com', 5432);
SELECT master_add_node('worker-2.example.com', 5432);

-- Distribute a table across worker nodes
SELECT create_distributed_table('orders', 'customer_id');
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `master_add_node` | Registers a new worker node with the coordinator |
| `create_distributed_table` | Distributes a table across workers based on the shard key |

#### Complete General Syntax (Kubernetes — Stateless Application Scaling)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sync-server
spec:
  replicas: 5  # Horizontal scale-out
  template:
    spec:
      containers:
      - name: sync-server
        image: sync-server:latest
        env:
        - name: DATABASE_URL
          value: "postgres://db:5432/app"
```

#### Constraints and Limitations

- **State management**: Stateful services are hard to scale horizontally; statelessness is preferred
- **Coordination overhead**: Distributed consensus adds latency and complexity
- **Rebalancing cost**: Adding nodes requires data migration and rebalancing
- **Cross-shard queries**: Queries spanning multiple shards are expensive
- **Premature scaling**: Horizontal scaling too early is one of the most expensive mistakes; vertical scaling often suffices first

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Citus Horizontal Scaling

**Setup**: Citus extension installed on PostgreSQL coordinator and worker nodes.

```sql
-- Step 1: Register worker nodes
SELECT master_add_node('worker-1', 5432);
SELECT master_add_node('worker-2', 5432);

-- Expected Output:
--  master_add_node 
-- -----------------
--                1
--                2
```

```sql
-- Step 2: Create a distributed table
CREATE TABLE orders (
    order_id BIGINT,
    customer_id BIGINT,
    order_date DATE,
    amount NUMERIC(10,2)
);
SELECT create_distributed_table('orders', 'customer_id');

-- Expected Output:
--  create_distributed_table 
-- --------------------------
--  
-- (1 row)
```

```sql
-- Step 3: Verify distribution
SELECT * FROM citus_shards;
-- Expected Output shows shards distributed across worker-1 and worker-2
```

**Why This Output Occurs**: `master_add_node` registers workers with the coordinator. `create_distributed_table` splits the table into shards based on the `customer_id` hash and distributes them across workers. Queries filtering by `customer_id` are routed to a single shard.

### Real-World Cases

**Case 1: Multi-Tenant SaaS**: A SaaS platform partitions tenant data across shards, each shard serving a subset of tenants. Adding a new tenant is as simple as adding a shard or routing to an existing one.

**Case 2: Stateless Application Scaling**: A sync server scales from 3 to 50 instances behind a load balancer, with all state stored in PostgreSQL. Requests are independent, so any instance can handle any request.

**Case 3: Analytics Scale-Out**: A data warehouse distributes fact tables across 20 worker nodes using Citus, enabling parallel query execution across shards.

---

## Core Concept 3: Read Scaling

### Definitions

**Core Definition**: Read scaling distributes read-only query load across multiple replica databases.

**Technical Definition**: Read scaling uses read replicas (standby databases that receive replicated changes from the primary) to offload SELECT queries. Cloud SQL read pools provide a single read endpoint with an immutable IP address; connections are automatically redirected to one of the read pool nodes. Read pools support between 1 and 20 read pool nodes, scaling horizontally by modifying node count or vertically by changing machine type.

**Beginner-Friendly Explanation**: Read scaling is like having multiple librarians who can answer questions (read queries) while one librarian handles new book arrivals (writes). The main librarian isn't overwhelmed by people asking for information.

### Purposes

- **To** offload read-only queries from the primary database
- **To** scale read throughput linearly by adding replicas
- **To** reduce query latency through geographic replica placement
- **To** provide a single endpoint that transparently load-balances across replicas

### Syntax Rules and Structure

#### Complete General Syntax (Cloud SQL Read Pool)

```bash
# Create a read pool with 3 nodes
gcloud sql instances patch my-primary \
    --read-pool-node-count=3 \
    --read-pool-auto-scaling=true
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `--read-pool-node-count` | Number of read pool nodes (1–20) |
| `--read-pool-auto-scaling` | Automatically adjusts node count based on load |
| Read endpoint | Single IP/hostname that load-balances across nodes |

#### Complete General Syntax (AWS RDS Read Replicas)

```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-read-replica-1 \
    --source-db-instance-identifier mydb \
    --db-instance-class db.r6g.large
```

#### Constraints and Limitations

- Read replicas may return stale data if replication lag exceeds acceptable thresholds
- Cross-region read replicas incur data transfer costs
- Read pool nodes always reside in the same region (Cloud SQL)
- Scaling beyond 10 total read replicas requires increasing `max_wal_senders` and `max_replication_slots` on the primary

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cloud SQL Read Pool with Autoscaling

```bash
# Step 1: Enable read pool autoscaling
gcloud sql instances patch my-primary \
    --read-pool-auto-scaling=true \
    --read-pool-min-node-count=2 \
    --read-pool-max-node-count=10

# Step 2: Verify read pool status
gcloud sql instances describe my-primary \
    --format="value(readPoolNodeCount, readPoolAutoScalingEnabled)"
```

**Expected Output**:
```
2  True
```

```sql
-- Step 3: Connect to the read endpoint (applications use this)
-- The endpoint automatically routes to an available read pool node
SELECT inet_server_addr();
-- Expected: Returns the IP of one of the read pool nodes
```

**Why This Output Occurs**: The read endpoint is a single IP/hostname that distributes connections across read pool nodes. Scaling in or out (adding/removing nodes) has sub-second downtime and requires no application reconfiguration.

### Real-World Cases

**Case 1: E-Commerce Product Catalog**: A product catalog database serves millions of read queries per hour. A read pool with 10 nodes distributes the load, reducing primary CPU from 90% to 20%.

**Case 2: Global Read Distribution**: A news website places read replicas in `us-east`, `eu-west`, and `ap-southeast`, serving readers from the nearest replica with sub-50ms latency.

**Case 3: Analytics Offload**: A SaaS company routes all dashboard queries to a dedicated read pool, isolating the primary from expensive aggregation queries.

---

## Core Concept 4: Write Scaling

### Definitions

**Core Definition**: Write scaling increases the throughput of write operations (INSERT, UPDATE, DELETE) across a distributed database system.

**Technical Definition**: Write scaling is fundamentally harder than read scaling because ACID transactions require coordination across nodes. Distributed coordination (consensus protocols, distributed locks, two-phase commit) introduces latency and communication overhead. Shared-nothing systems must pay the overhead of distributed coordination and commit protocols, which may lead to scalability limits for write-intensive workloads.

**Beginner-Friendly Explanation**: Write scaling is like having multiple cashiers at a store who all need to update the same inventory system. If two cashiers sell the last item at the same time, someone has to decide who gets it. This coordination is the hard part—reads don't need it, but writes do.

### Purposes

- **To** increase write throughput beyond a single server's capacity
- **To** distribute write load across multiple shards or nodes
- **To** maintain ACID guarantees while scaling writes horizontally
- **To** reduce write latency through geographic distribution

### Syntax Rules and Structure

#### Complete General Syntax (MySQL Group Replication — Multi-Primary)

```sql
-- Configure multi-primary write scaling
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_single_primary_mode = OFF;
SET GLOBAL group_replication_enforce_update_everywhere_checks = ON;
START GROUP_REPLICATION;
```

#### Complete General Syntax (CockroachDB — Distributed Writes)

```sql
-- CockroachDB automatically distributes writes across ranges
-- Each range is a Raft group with its own leader
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id INT,
    amount DECIMAL(10,2)
);
-- Writes are automatically distributed across nodes based on key ranges
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `group_replication_single_primary_mode = OFF` | Enables multi-primary writes |
| `enforce_update_everywhere_checks` | Ensures conflict detection |
| Raft groups | Each key range has its own leader for write coordination |

#### Constraints and Limitations

- **ACID overhead**: Distributed transactions require 2PC or consensus, adding latency
- **Write skew**: Multi-primary systems may experience write skew anomalies under certain isolation levels
- **Conflict detection**: Multi-primary writes require conflict detection and resolution
- **Hotspots**: Sequential keys can create write hotspots on a single shard

### Annotated Complete Step-by-Step Code Examples

#### Example 1: CockroachDB Distributed Write Scaling

```sql
-- Step 1: Create a table with a UUID primary key (avoids hotspots)
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id INT,
    amount DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT now()
);

-- Step 2: Insert data — writes are distributed across ranges
INSERT INTO orders (customer_id, amount)
SELECT (i % 10000) + 1, random() * 1000
FROM generate_series(1, 100000) AS i;

-- Expected: 100,000 rows inserted, distributed across multiple ranges
```

```sql
-- Step 3: Verify distribution
SELECT 
    range_id,
    start_key,
    end_key,
    lease_holder
FROM crdb_internal.ranges
LIMIT 10;
-- Expected: Shows multiple ranges with different lease holders
```

**Why This Output Occurs**: CockroachDB splits data into ranges (default 64 MB each). Each range is a Raft group with its own leader (lease holder). Writes to different ranges are processed in parallel by different nodes, enabling horizontal write scaling.

### Real-World Cases

**Case 1: Multi-Region Active-Active**: A global collaboration platform uses CockroachDB with multi-region writes, allowing users in `us-east` and `eu-west` to write locally with automatic conflict resolution.

**Case 2: IoT Sensor Data Ingestion**: An IoT platform ingests millions of sensor readings per second, distributing writes across shards based on `sensor_id` hash.

**Case 3: Financial Trading**: A trading platform uses MySQL Group Replication in multi-primary mode for write scaling across three data centers, with conflict detection ensuring consistency.

---

## Core Concept 5: Sharding Concepts

### Definitions

**Core Definition**: Sharding is the horizontal partitioning of data across multiple database instances, where each shard holds a disjoint subset of rows.

**Technical Definition**: Sharding splits one logical database into many physical databases (shards), each holding a disjoint subset of the data. The sharding key determines data placement. Sharding strategies include hash-based (even distribution), range-based (good for range queries), and directory-based (flexible placement via a lookup table).

**Beginner-Friendly Explanation**: Sharding is like splitting a phone book into multiple volumes—A–F in one book, G–M in another, and so on. Each book (shard) holds part of the data, and you look up the right book based on the first letter (shard key).

### Purposes

- **To** distribute data across multiple servers for horizontal write scaling
- **To** isolate tenants or data categories into dedicated shards
- **To** reduce query latency through shard-level parallelism
- **To** enable geographic data placement for compliance

### Sub-Concept 5.1: Shard Key Selection

#### Syntax Rules and Structure

**Shard key criteria** (based on Oracle and MongoDB best practices):

| Criterion | Description |
|-----------|-------------|
| **High Cardinality** | Millions of unique values (e.g., `customer_id`, not `status`) |
| **Even Distribution** | No single value > 5% of rows |
| **Immutable** | Value should almost never change; changing key requires data migration |
| **Query Alignment** | Appears in 80%+ of WHERE clauses |
| **Stable** | Not based on volatile information or auto-incrementing fields |

#### Constraints and Limitations

- Sequential IDs with range sharding create write hotspots on the latest shard
- Timestamp as shard key overloads the recent shard
- Nullable shard keys create special hotspot handling

### Sub-Concept 5.2: Algorithmic vs. Directory-Based Sharding

| Aspect | Algorithmic (Hash/Range) | Directory-Based |
|--------|--------------------------|-----------------|
| **Routing** | Computed from key (hash or range) | Lookup table maps key to shard |
| **Rebalancing** | Consistent hashing minimizes key movement | Flexible placement, easy rebalancing |
| **Overhead** | No lookup overhead | Network hop per query; SPOF on directory |
| **Use Case** | High-volume, predictable queries | Multi-tenancy, complex routing |

**Consistent hashing** reduces the impact of adding/removing shards by organizing hash space so only a small fraction of keys move when shard count changes. With standard hash (hash(key) mod N), adding or removing a shard reassigns most keys and triggers large-scale data migration.

### Sub-Concept 5.3: Re-sharding and Rebalancing

#### Syntax Rules and Structure (PostgreSQL Citus)

```sql
-- Add a new worker and rebalance shards
SELECT master_add_node('worker-3', 5432);
SELECT rebalance_table_shards('orders');

-- Expected Output:
--  rebalance_table_shards 
-- ------------------------
--  
-- (1 row)
```

#### Constraints and Limitations

- Rebalancing moves data between shards and often causes unavailability or reduced throughput
- Virtual partitions (many logical partitions mapped to fewer physical shards) reduce rebalancing frequency
- Prefer many small shards over a few large ones: smaller shards migrate faster and balance more evenly

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Citus Shard Rebalancing

```sql
-- Step 1: Check current shard distribution
SELECT nodename, count(*) AS shard_count
FROM pg_dist_shard_placement
GROUP BY nodename;
-- Expected Output:
--  nodename  | shard_count 
-- -----------+-------------
--  worker-1  |          16
--  worker-2  |          16
```

```sql
-- Step 2: Add a new worker
SELECT master_add_node('worker-3', 5432);
-- Expected: worker-3 registered

-- Step 3: Rebalance shards across all workers
SELECT rebalance_table_shards('orders');
-- Expected Output: Shards redistributed evenly across 3 workers

-- Step 4: Verify new distribution
SELECT nodename, count(*) AS shard_count
FROM pg_dist_shard_placement
GROUP BY nodename;
-- Expected Output:
--  nodename  | shard_count 
-- -----------+-------------
--  worker-1  |          11
--  worker-2  |          11
--  worker-3  |          10
```

**Why This Output Occurs**: `rebalance_table_shards` moves shards from overloaded workers to the new worker, using consistent hashing to minimize data movement. The new distribution is roughly even across all three workers.

### Real-World Cases

**Case 1: Multi-Tenant SaaS with Directory Sharding**: A SaaS platform uses a directory table to map tenant IDs to shards, enabling flexible placement and easy tenant migration between shards.

**Case 2: Time-Series Data with Range Sharding**: A monitoring system shards metrics by date range, enabling efficient queries for recent data and simple archival of old shards.

**Case 3: High-Volume Transactions with Hash Sharding**: A payment processor shards transactions by `merchant_id` hash, distributing write load evenly across 32 shards.

---

## Core Concept 6: Distributed SQL Considerations

### Definitions

**Core Definition**: Distributed SQL considerations encompass the theoretical and practical trade-offs involved in building and using databases that span multiple nodes.

**Technical Definition**: Distributed SQL systems must navigate the CAP theorem (Consistency, Availability, Partition tolerance—pick two), implement consensus algorithms (Raft, Paxos) for leader election and log replication, and provide distributed transactions (2PC) for cross-shard atomicity. NewSQL systems employ lock-free concurrency control and shared-nothing architectures to achieve horizontal scalability while maintaining ACID guarantees.

**Beginner-Friendly Explanation**: Distributed SQL is like coordinating a team of people working on the same document. You need rules for who can edit what, how to handle conflicts when two people edit the same section, and what happens if the network between them fails. These rules are the "distributed SQL considerations."

### Purposes

- **To** provide a framework for understanding consistency vs. availability trade-offs
- **To** ensure correct leader election and log replication through consensus
- **To** guarantee atomicity across shards through distributed commit protocols
- **To** achieve horizontal scalability while maintaining SQL semantics

### Sub-Concept 6.1: CAP Theorem and PACELC

#### Definitions

**CAP Theorem**: In a distributed system, you can simultaneously guarantee at most two of: Consistency, Availability, and Partition tolerance. Systems with scalability as the primary goal typically provide high availability and partition tolerance, sacrificing strong consistency.

**PACELC**: An extension of CAP that adds: if there is a Partition (P), trade off Availability vs. Consistency (A/C); Else (E), trade off Latency vs. Consistency (L/C). This captures the latency-consistency trade-off that exists even without partitions.

#### Constraints and Limitations

- **CP systems** (e.g., CockroachDB, Spanner) choose consistency over availability during partitions
- **AP systems** (e.g., Cassandra, DynamoDB) choose availability over consistency during partitions
- **PACELC** reveals that even normal operation involves latency-consistency trade-offs

### Sub-Concept 6.2: Consensus Algorithms (Raft and Paxos)

#### Definitions

**Raft**: A consensus algorithm for managing a replicated log. It produces a result equivalent to (multi-)Paxos and is as efficient, but its structure is different—it separates leader election, log replication, and safety, making it more understandable and providing a better foundation for practical systems.

**Paxos**: A classic consensus protocol on which distributed systems with strong consistency are built. Paxos lets all nodes agree on a single decision, while Raft is designed to let nodes agree on a log of multiple entries.

#### Constraints and Limitations

- Consensus requires a majority quorum; network partitions can split a cluster if quorum is lost
- Raft requires leader election and log replication; Paxos requires more complex state management
- Both add latency proportional to round-trip time between nodes

### Sub-Concept 6.3: Distributed Transactions (2PC)

#### Definitions

**Two-Phase Commit (2PC)** : A protocol that ensures a transaction either commits at all resource managers it accessed or aborts at all of them. It avoids the undesirable outcome of committing at one resource manager and aborting at another. The protocol has two phases—voting and decision—both driven by a coordinator process. The first phase ensures all participants move into the prepared state; the second phase disseminates the global decision.

#### Syntax Rules and Structure (XA Transactions — MySQL)

```sql
-- XA transaction in MySQL
XA START 'xid';
INSERT INTO accounts VALUES (1, 500);
XA END 'xid';
XA PREPARE 'xid';  -- Phase 1: Prepare
XA COMMIT 'xid';   -- Phase 2: Commit
```

#### Constraints and Limitations

- **Blocking**: If the coordinator fails after participants vote to commit, participants remain blocked until the coordinator recovers
- **Latency**: 2PC adds 2n+2 forced log entries and 4n messages for n participants
- **Independent recovery impossible**: A failed participant cannot unilaterally decide without communicating with the coordinator

### Annotated Complete Step-by-Step Code Examples

#### Example 1: CockroachDB Raft-Based Distributed Writes

```sql
-- Step 1: Observe Raft replication in action
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL(10,2)
);
INSERT INTO accounts VALUES (1, 1000.00);

-- Step 2: Check range lease holder (Raft leader)
SELECT 
    range_id,
    start_key,
    lease_holder,
    replicas
FROM crdb_internal.ranges
WHERE start_key IS NULL OR start_key LIKE '%accounts%';
-- Expected Output: Shows the range containing accounts, with a lease_holder and replica set
```

```sql
-- Step 3: Perform a distributed transaction (automatically 2PC if cross-range)
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
COMMIT;
-- Expected: COMMIT (Raft replicates the transaction across replicas)
```

**Why This Output Occurs**: CockroachDB splits data into ranges, each managed by a Raft group. The lease holder is the Raft leader for that range. Writes are replicated to a majority of replicas before acknowledgment, ensuring durability and consistency.

### Real-World Cases

**Case 1: Global Banking with Spanner**: A multinational bank uses Google Spanner for globally distributed transactions, leveraging TrueTime and Paxos for external consistency.

**Case 2: E-Commerce with CockroachDB**: An e-commerce platform uses CockroachDB for its order management system, scaling writes across regions while maintaining ACID guarantees.

**Case 3: Financial Ledger with 2PC**: A financial institution uses MySQL XA transactions across two shards to atomically transfer funds between accounts, with 2PC ensuring both shards commit or abort together.

---

## References

| Name | Link |
|------|------|
| Microsoft Learn — Scaling resources in Azure Database for PostgreSQL Flexible Server | https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-scaling-resources |
| Microsoft Learn — Recommend a solution for database scalability | https://learn.microsoft.com/en-nz/training/modules/design-data-storage-solution-for-relational-data/5-recommend-database-scalability |
| Google Cloud — About read pools | https://cloud.google.com/sql/docs/postgres/about-read-pools |
| Google Cloud — AlloyDB overview | https://cloud.google.com/alloydb/docs/overview |
| AWS — Scaling Amazon DocumentDB clusters | https://docs.aws.amazon.com/documentdb/latest/developerguide/scaling.html |
| OceanBase — Vertical Scaling Without Database Downtime | https://en.oceanbase.com/blog/vertical-scaling-in-oceanbase |
| Oracle — Sharding Keys | https://docs.oracle.com/en/database/oracle/oracle-database/26/shard/sharding-keys.html |
| Microsoft Azure Architecture Center — Sharding pattern | https://learn.microsoft.com/en-us/azure/architecture/patterns/sharding |
| Oracle — How to Choose a Shard Key | https://help.aliyun.com/ |
| MongoDB — Choose a Shard Key | https://www.mongodb.com/docs/manual/core/sharding-choose-a-shard-key/ |
| Vitess — How do you select your sharding key for Vitess? | https://vitess.io/docs/ |
| Abadi — Consistency Tradeoffs in Modern Distributed Database System Design | https://dl.acm.org/doi/10.1109/MC.2012.33 |
| Ongaro & Ousterhout — In Search of an Understandable Consensus Algorithm (Raft) | https://dl.acm.org/doi/10.5555/2643634.2643666 |
| ScienceDirect — Two-Phase Commit Overview | https://www.sciencedirect.com/topics/computer-science/two-phase-commit |
| ScienceDirect — NewSQL Systems Overview | https://www.sciencedirect.com/topics/computer-science/entire-database |
| CMU — Distributed Transaction Management | http://www.cs.cmu.edu/~natassa/courses/15-823/F02/papers/Rstar_distrib.pdf |
| CMU — Consensus on Transaction Commit | https://www.cs.cmu.edu/~natassa/courses/15-823/F02/papers/Rstar_distrib.pdf |
| Microsoft Learn — Two-Phase Commit (Host Integration Server) | https://learn.microsoft.com/en-us/host-integration-server/core/two-phase-commit2 |