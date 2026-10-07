# SQL Database Replication: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: Database replication is the process of maintaining synchronized copies of data across multiple database servers to improve availability, scalability, and fault tolerance.

**Technical Definition**: Database replication involves propagating data changes from a primary (source) database to one or more replicas (standbys) using mechanisms including Write-Ahead Log (WAL) shipping, binary log replication, or log-based change data capture, with consistency guarantees ranging from asynchronous (best-effort) to synchronous (transactional guarantee), and topologies spanning single-primary/multi-replica to multi-primary/active-active configurations.

**Beginner-Friendly Explanation**: Replication is like having multiple copies of the same notebook in different locations. Every time you write something new in the main notebook, the system automatically copies that update to all the other notebooks—either right away (synchronous) or a little later (asynchronous).

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Consistency Model** | Synchronous (zero data loss) vs. asynchronous (eventual consistency) |
| **Topology** | Single-primary/multi-replica, multi-primary/active-active, cascading |
| **Replication Lag** | Delay between primary commit and replica apply; impacts stale reads |
| **Failover Mode** | Automatic, planned manual, or forced (with potential data loss) |
| **Conflict Resolution** | LWW, vector clocks, CRDTs for multi-primary systems |
| **Use Cases** | Read scaling, geographic distribution, disaster recovery, HA |

### Prerequisites

- **Multiple Database Nodes**: At least two servers (primary + standby) for basic replication
- **Replication Configuration**: Streaming replication (PostgreSQL), binary log replication (MySQL), or Always On Availability Groups (SQL Server)
- **Network Connectivity**: Low-latency links between nodes for synchronous replication
- **Replication User**: Dedicated credentials with `REPLICATION` privileges
- **Monitoring Infrastructure**: Tools for tracking replication lag and health
- **Failover Orchestration**: Cluster manager (Patroni, WSFC) or manual runbooks

### Related Programming Areas

- **Distributed Systems**: Consensus algorithms, CAP theorem, consistency models
- **Database Administration (DBA)** : Replication monitoring, failover drills, capacity planning
- **Cloud Engineering**: Multi-AZ deployments, managed replication services (RDS, Azure SQL)
- **Site Reliability Engineering (SRE)** : SLO/SLI definition, chaos engineering for replication
- **Application Development**: Read-after-write consistency, stale-read handling, connection routing

### Core Concepts Overview

SQL database replication comprises seven complementary techniques:

1. **Synchronous Replication**: Transaction commits wait for standby acknowledgment (including semi-synchronous)
2. **Asynchronous Replication**: Primary commits without waiting for replicas (default in most systems)
3. **Read Replicas**: Scaling read traffic and geographic distribution
4. **Replication Lag**: Monitoring, alerting, and application-level stale-read handling
5. **Failover Strategies**: Manual vs. automated failover, split-brain prevention, fencing/STONITH
6. **Replication Topologies**: Single-primary/multi-replica, multi-primary/active-active, cascading
7. **Conflict Resolution**: LWW, vector clocks, and CRDTs for multi-primary systems

---

## Core Concept 1: Synchronous Replication

### Definitions

**Core Definition**: Synchronous replication guarantees that a transaction is committed on the primary only after it has been confirmed on at least one standby.

**Technical Definition**: In synchronous replication, the primary waits for acknowledgment from standby server(s) that the transaction's WAL records have been received and persisted (or applied) before returning commit confirmation to the client. In PostgreSQL, this is controlled by `synchronous_commit` and `synchronous_standby_names`. In MySQL, semi-synchronous replication provides a middle ground where the primary waits for at least one replica to confirm receipt (not necessarily application).

**Beginner-Friendly Explanation**: Synchronous replication means the primary says "I've saved this" only after at least one backup has also said "I've saved this." Nothing is lost if the primary crashes immediately after. Semi-synchronous is a compromise: the primary waits for one backup to say "I got it" (not "I've fully applied it").

### Purposes

- **To** guarantee zero data loss for acknowledged commits
- **To** ensure that at least one replica has durable copies of all committed transactions
- **To** provide a foundation for automatic failover without data loss
- **To** offer configurable durability levels (remote_write, on, remote_apply) balancing safety and latency

### Sub-Concept 1.1: PostgreSQL Synchronous Replication

#### Syntax Rules and Structure

```conf
# postgresql.conf (on primary)
synchronous_commit = on
synchronous_standby_names = 'standby1'
wal_level = replica
max_wal_senders = 10
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `synchronous_commit` | `off` (async), `local` (local disk only), `remote_write` (standby OS), `on` (standby durable storage), `remote_apply` (standby applied) |
| `synchronous_standby_names` | List of standby servers that can acknowledge commits; supports `FIRST` (priority-based) and `ANY` (quorum-based) |

#### Syntax Rules

- `synchronous_standby_names` must be non-empty for synchronous replication to activate
- `synchronous_commit = on` is the default; `remote_apply` provides the strongest guarantee
- Individual transactions can override with `SET LOCAL synchronous_commit = off`

#### Constraints and Limitations

- Synchronous replication increases transaction latency by at least the round-trip time to the standby
- If the synchronous standby fails, the primary may block or degrade
- Cascading replication is currently asynchronous; synchronous settings have no effect on cascading standby

### Sub-Concept 1.2: MySQL Semi-Synchronous Replication

#### Syntax Rules and Structure

```sql
-- On source server
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
SET GLOBAL rpl_semi_sync_source_enabled = 1;
SET GLOBAL rpl_semi_sync_source_timeout = 1000;  -- milliseconds

-- On replica server
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
SET GLOBAL rpl_semi_sync_replica_enabled = 1;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `rpl_semi_sync_source_enabled` | Enables/disables semi-synchronous replication on source (default 0) |
| `rpl_semi_sync_source_timeout` | Milliseconds to wait for replica acknowledgment before degrading to async |
| `rpl_semi_sync_replica_enabled` | Enables semi-synchronous on replica |

#### Constraints and Limitations

- If the semi-synchronous replica becomes slow or unavailable, the primary degrades to asynchronous replication after timeout
- Only one replica needs to acknowledge (configurable in some systems)
- MySQL 8.0.26+ replaces "master/slave" with "source/replica" terminology in plugin names

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Synchronous Replication Setup

**Setup**: Two PostgreSQL 14 servers: primary (`pg-primary`) and standby (`pg-standby`).

```conf
# Step 1: On pg-primary, configure postgresql.conf
wal_level = replica
max_wal_senders = 10
synchronous_standby_names = 'pg-standby'
synchronous_commit = on
```

```bash
# Step 2: On pg-standby, create base backup
pg_basebackup -h pg-primary -D /var/lib/postgresql/14/main \
    -U replication -P -X stream -R
# Output: base backup completed
```

```sql
-- Step 3: On pg-primary, verify synchronous replication
SELECT application_name, state, sync_state 
FROM pg_stat_replication;
```

**Expected Output**:
```
 application_name |   state   | sync_state 
------------------+-----------+------------
 pg-standby       | streaming | sync
```

**Why This Output Occurs**: `sync_state = 'sync'` confirms that `pg-standby` is the synchronous standby. Commits on the primary will wait for this standby to acknowledge receipt of WAL records.

#### Example 2: MySQL Semi-Synchronous Replication

```sql
-- Step 1: On source, install and enable plugin
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
SET GLOBAL rpl_semi_sync_source_enabled = 1;
SET GLOBAL rpl_semi_sync_source_timeout = 1000;

-- Step 2: On replica, install and enable plugin
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
SET GLOBAL rpl_semi_sync_replica_enabled = 1;

-- Step 3: Verify semi-synchronous status
SHOW STATUS LIKE 'Rpl_semi_sync_source_status';
```

**Expected Output**:
```
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| Rpl_semi_sync_source_status  | ON    |
+------------------------------+-------+
```

**Why This Output Occurs**: The semi-synchronous plugin is active and waiting for at least one replica acknowledgment before returning commit success to clients.

### Real-World Cases

**Case 1: Financial Transaction Processing**: A banking application uses synchronous replication to guarantee zero data loss. Every transaction is confirmed on at least two servers before the customer sees a success message.

**Case 2: MySQL Semi-Synchronous for Balanced Durability**: A SaaS application uses semi-synchronous replication to balance durability and performance. If the replica falls behind, the system degrades gracefully to asynchronous mode.

**Case 3: PostgreSQL remote_apply for Strong Consistency**: A stock trading platform uses `synchronous_commit = remote_apply` to ensure that committed transactions are visible on the standby before the client receives confirmation.

---

## Core Concept 2: Asynchronous Replication

### Definitions

**Core Definition**: Asynchronous replication commits transactions on the primary without waiting for standby acknowledgment.

**Technical Definition**: In asynchronous replication, the primary writes to its local WAL/log and returns commit confirmation immediately; standby servers receive and apply changes independently, introducing replication lag. PostgreSQL streaming replication is asynchronous by default.

**Beginner-Friendly Explanation**: Asynchronous replication means the primary says "I've saved this" right away, without waiting to hear from backups. It's faster, but if the primary crashes before backups catch up, some recent data might be lost.

### Purposes

- **To** minimize transaction latency by eliminating standby round-trip waits
- **To** maximize availability by allowing the primary to continue even if standbys are unreachable
- **To** support geographically distributed replicas where network latency is high
- **To** enable read scaling without impacting primary write performance

### Syntax Rules and Structure

```conf
# postgresql.conf (default — asynchronous)
synchronous_commit = off
# No synchronous_standby_names required
wal_level = replica
max_wal_senders = 10
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `synchronous_commit = off` | Commits return immediately without waiting for WAL flush or standby acknowledgment |
| `wal_level = replica` | Required for streaming replication |

#### Constraints and Limitations

- Potential data loss if primary fails before standby receives changes; amount of loss is proportional to replication delay at time of failure
- Replication lag can grow under heavy write loads
- Standby may serve stale data for read queries

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Asynchronous Streaming Replication

```conf
# Step 1: On primary, configure postgresql.conf
wal_level = replica
max_wal_senders = 10
# synchronous_commit = off (default)
```

```bash
# Step 2: On standby, create base backup
pg_basebackup -h primary -D /var/lib/postgresql/14/main \
    -U replicator -P -X stream -R
```

```sql
-- Step 3: On primary, verify async replication
SELECT application_name, state, sync_state, 
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

**Expected Output**:
```
 application_name |   state   | sync_state | lag_bytes 
------------------+-----------+------------+-----------
 standby1         | streaming | async      |     10240
```

**Why This Output Occurs**: `sync_state = 'async'` confirms asynchronous replication. `lag_bytes` shows the byte gap between the primary's current WAL position and the standby's replay position.

### Real-World Cases

**Case 1: E-Commerce Read Scaling**: An e-commerce platform uses asynchronous replication to create read replicas. Product catalog queries are served from replicas, reducing primary load by 70% with acceptable staleness of a few hundred milliseconds.

**Case 2: Cross-Region Replication**: A global SaaS platform replicates data asynchronously from `us-east` to `eu-west` and `ap-southeast`, accepting higher lag in exchange for geographic read locality.

**Case 3: Analytics Offload**: A reporting database uses asynchronous replication to receive data from the primary without impacting transactional throughput.

---

## Core Concept 3: Read Replicas

### Definitions

**Core Definition**: Read replicas are standby databases that serve read-only queries to offload the primary server.

**Technical Definition**: Read replicas (also called read-only secondaries) receive replicated changes from the primary and can process SELECT queries, enabling horizontal scaling of read workloads and geographic distribution of read traffic.

**Beginner-Friendly Explanation**: Read replicas are like extra librarians who can answer questions (read queries) but can't accept new books (writes). They let the main librarian focus on accepting new books while the others handle questions.

### Purposes

- **To** scale read workloads horizontally by distributing queries across replicas
- **To** reduce primary server load by offloading reporting and analytics
- **To** enable geographic distribution for latency reduction
- **To** provide failover targets for high availability

### Sub-Concept 3.1: Read Scaling

#### Syntax Rules and Structure (AWS RDS Read Replicas)

```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-read-replica \
    --source-db-instance-identifier mydb \
    --db-instance-class db.r6g.large
```

#### Syntax Rules and Structure (Cloud SQL Read Pools)

```bash
# Google Cloud SQL read pool configuration
gcloud sql instances patch mydb \
    --read-pool-node-count=3 \
    --read-pool-auto-scaling=true
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `read-pool-node-count` | Number of read replicas in the pool |
| `read-pool-auto-scaling` | Automatically adjusts replica count based on load |
| Read endpoint | Single endpoint that load-balances across read replicas round-robin |

#### Constraints and Limitations

- Read replicas may return stale data if replication lag exceeds acceptable thresholds
- Applications must be designed to tolerate eventual consistency for read queries
- Read replicas add storage and compute costs

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Read Replica Configuration

```bash
# Step 1: On replica, set up streaming replication
pg_basebackup -h primary -D /var/lib/postgresql/14/main \
    -U replicator -P -X stream -R

# Step 2: Configure hot_standby for read queries
echo "hot_standby = on" >> /var/lib/postgresql/14/main/postgresql.conf

# Step 3: Start PostgreSQL on replica
sudo systemctl start postgresql

# Step 4: Verify read-only access
psql -h replica -c "SELECT count(*) FROM orders;"
```

**Expected Output**:
```
 count 
-------
 15000
(1 row)
```

```sql
-- Step 5: Verify replica is read-only
INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
-- Expected: ERROR: cannot execute INSERT in a read-only transaction
```

**Why This Output Occurs**: `hot_standby = on` allows the standby to accept read-only connections. Write attempts are rejected because the standby is in recovery mode.

#### Example 2: MySQL Read Replica with Connection Routing

```sql
-- Step 1: On primary, create replication user
CREATE USER 'repl'@'%' IDENTIFIED BY 'secret';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- Step 2: On replica, configure replication
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'primary',
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'secret',
    SOURCE_AUTO_POSITION = 1;
START REPLICA;

-- Step 3: Verify replication status
SHOW REPLICA STATUS\G
```

**Expected Output**:
```
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: primary
              Replica_IO_Running: Yes
             Replica_SQL_Running: Yes
```

**Why This Output Occurs**: `SHOW REPLICA STATUS` reports both IO and SQL threads running, confirming active replication.

### Real-World Cases

**Case 1: E-Commerce Read Scaling**: An e-commerce platform routes product catalog queries to three read replicas, reducing primary CPU load by 40% while maintaining sub-100ms query latency.

**Case 2: Geographic Read Distribution**: A news website uses PostgreSQL logical replication to distribute read replicas to `us-east`, `eu-west`, and `ap-southeast`, serving readers from the nearest replica.

**Case 3: Analytics Offload**: A SaaS company routes all analytics queries to a dedicated read replica, isolating the primary from expensive aggregation queries.

---

## Core Concept 4: Replication Lag

### Definitions

**Core Definition**: Replication lag is the delay between when a transaction commits on the primary and when it is applied on a replica.

**Technical Definition**: Replication lag is measured as the difference between the primary's current WAL position (or binary log position) and the standby's replay position, expressed in bytes, time, or transactions.

**Beginner-Friendly Explanation**: Replication lag is like the delay between live TV and a streaming broadcast. The primary is live; the replica is a few seconds behind. If you only watch the stream, you might miss the very latest updates.

### Purposes

- **To** quantify how far behind a replica is from the primary
- **To** trigger alerts when lag exceeds acceptable thresholds
- **To** inform application-level decisions about read routing
- **To** diagnose replication performance issues (network, disk I/O, long transactions)

### Sub-Concept 4.1: Monitoring Replication Lag

#### Syntax Rules and Structure (PostgreSQL)

```sql
-- On primary: byte lag for all replicas
SELECT 
    application_name,
    client_addr,
    state,
    pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes,
    EXTRACT(EPOCH FROM replay_lag) AS replay_lag_seconds
FROM pg_stat_replication;

-- On standby: time-based lag
SELECT 
    now() - pg_last_xact_replay_timestamp() AS time_lag;
```

#### Syntax Rules and Structure (MySQL)

```sql
-- On replica: check replication lag
SHOW REPLICA STATUS\G

-- Key fields:
-- Seconds_Behind_Source: estimated lag in seconds
-- Relay_Log_Pos: current position in relay log
-- Exec_Source_Log_Pos: current position in source log
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `pg_wal_lsn_diff` | Calculates byte difference between two WAL positions |
| `replay_lag` | Time delay for replay (PostgreSQL 10+) |
| `Seconds_Behind_Source` | Estimated lag in seconds (MySQL) |

#### Constraints and Limitations

- `Seconds_Behind_Source` can be NULL or misleading during idle periods
- Byte lag alone does not indicate time lag (varies with WAL generation rate)
- Missing rows in `pg_stat_replication` indicate disconnected standbys

### Sub-Concept 4.2: Application-Level Stale-Read Handling

#### Definitions

**Stale Read**: A read query that returns outdated data because it was served from a replica that has not yet applied recent changes.

**Read-After-Write Consistency**: A guarantee that a client's own writes are visible to its subsequent reads.

#### Syntax Rules and Structure (Application-Level Routing)

```python
# Python example: read-after-write routing
def get_user_data(user_id, last_write_time=None):
    if last_write_time and (time.time() - last_write_time) < 5:
        # Route to primary for 5 seconds after write
        return query_primary(user_id)
    else:
        # Route to replica
        return query_replica(user_id)
```

#### Constraints and Limitations

- Read-after-write routing adds complexity to application code
- Bounded staleness reads trade consistency for availability
- Monotonic read consistency ensures a client never sees older data after newer data

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Lag Monitoring Query

```sql
-- Step 1: Create a monitoring view
CREATE OR REPLACE VIEW replication_lag AS
SELECT 
    application_name,
    client_addr,
    state,
    pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn) AS send_lag_bytes,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS replay_lag_bytes,
    EXTRACT(EPOCH FROM replay_lag) AS replay_lag_seconds
FROM pg_stat_replication;

-- Step 2: Query the view
SELECT * FROM replication_lag;
```

**Expected Output**:
```
 application_name | client_addr  |   state   | send_lag_bytes | replay_lag_bytes | replay_lag_seconds
------------------+--------------+-----------+----------------+------------------+--------------------
 standby1         | 10.0.0.11    | streaming |          10240 |            20480 |               0.25
 standby2         | 10.0.0.12    | streaming |          51200 |            81920 |               1.50
```

**Why This Output Occurs**: `send_lag_bytes` measures network transfer delay; `replay_lag_bytes` measures apply delay. `replay_lag_seconds` provides a human-readable time-based lag.

#### Example 2: MySQL Lag Monitoring

```sql
-- Step 1: Execute on replica
SHOW REPLICA STATUS\G
```

**Expected Output** (excerpt):
```
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: primary
              Replica_IO_Running: Yes
             Replica_SQL_Running: Yes
           Seconds_Behind_Source: 2
                  Relay_Log_Pos: 1145
          Exec_Source_Log_Pos: 927
```

**Why This Output Occurs**: `Seconds_Behind_Source = 2` indicates the replica is 2 seconds behind the primary. `Relay_Log_Pos` and `Exec_Source_Log_Pos` show the replication progress.

### Real-World Cases

**Case 1: Alerting on Replication Lag**: A monitoring system alerts when `replay_lag_seconds > 10`, triggering investigation before lag impacts application consistency.

**Case 2: Read-After-Write Routing**: A social media application routes a user's reads to the primary for 5 seconds after they post a comment, ensuring they see their own comment immediately.

**Case 3: Bounded Staleness for Analytics**: An analytics dashboard uses bounded staleness reads, accepting up to 30 seconds of lag to distribute load across replicas.

---

## Core Concept 5: Failover Strategies

### Definitions

**Core Definition**: Failover is the process of transferring the primary role from one database server to another.

**Technical Definition**: Failover encompasses the detection of primary failure, selection of a new primary from available replicas, promotion of that replica, and redirection of client connections. SQL Server Always On supports three failover types: automatic (without data loss), planned manual (without data loss), and forced manual (with possible data loss).

**Beginner-Friendly Explanation**: Failover is like a relay race—when the current runner (primary) can't continue, the next runner (standby) picks up the baton and keeps going, so the race (application) doesn't stop.

### Purposes

- **To** restore service availability after primary server failure
- **To** enable planned maintenance on the primary with zero downtime
- **To** provide a controlled mechanism for disaster recovery
- **To** ensure that a designated standby can assume the primary role

### Sub-Concept 5.1: Manual Failover

#### Syntax Rules and Structure (PostgreSQL)

```bash
# On standby, promote to primary
pg_ctl promote -D /var/lib/postgresql/14/main
# Or:
SELECT pg_promote();
```

#### Syntax Rules and Structure (SQL Server)

```sql
-- Planned manual failover of availability group
ALTER AVAILABILITY GROUP [MyAG] FAILOVER;
```

#### Constraints and Limitations

- The failover target must be synchronized before failover for zero data loss
- Applications must be redirected (via listener, DNS, or connection string update)
- The old primary must be fenced or reconfigured as a standby after failover

### Sub-Concept 5.2: Automated Failover

#### Syntax Rules and Structure (Patroni for PostgreSQL)

```yaml
# patroni.yml
scope: postgres
namespace: /db/
name: node1
restapi:
  listen: 0.0.0.0:8008
  connect_address: node1:8008
etcd:
  host: etcd1:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
postgresql:
  listen: 0.0.0.0:5432
  connect_address: node1:5432
  data_dir: /var/lib/postgresql/14/main
```

#### Syntax Rules and Structure (SQL Server Always On)

```sql
-- Configure automatic failover
ALTER AVAILABILITY GROUP [MyAG]
MODIFY REPLICA ON N'standby1' WITH
    (FAILOVER_MODE = AUTOMATIC);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ttl` | Time-to-live for leader lease in etcd |
| `maximum_lag_on_failover` | Maximum byte lag allowed for automatic failover |
| `FAILOVER_MODE = AUTOMATIC` | Enables automatic failover for synchronous replica |

#### Constraints and Limitations

- Automatic failover requires a quorum or witness to prevent split-brain
- The standby must be synchronous for zero data loss automatic failover
- A brief service interruption occurs during failover (typically seconds)
- Patroni uses etcd for distributed consensus and leader election

### Sub-Concept 5.3: Fencing and STONITH

#### Definitions

**STONITH**: "Shoot The Other Node In The Head" — forcibly powering off or isolating a failed primary to prevent split-brain.

#### Syntax Rules and Structure (Pacemaker)

```bash
# Configure STONITH device
pcs stonith create fence_node1 fence_ipmilan \
    pcmk_host_list=node1 \
    ipaddr=10.0.0.101 \
    login=admin \
    passwd=secret \
    lanplus=1 \
    op monitor interval=60s
```

#### Constraints and Limitations

- Fencing requires out-of-band management (IPMI, iDRAC, iLO) or power switches
- Unfenced old primary may still accept writes if network isolation occurs
- Quorum requires a minimum number of nodes (typically 3 for majority voting)

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Manual Failover

```bash
# Step 1: Verify standby is synchronized
psql -h standby -c "SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn();"
# Expected: Both LSNs equal — zero lag

# Step 2: Promote standby to primary
psql -h standby -c "SELECT pg_promote();"
# Expected: pg_promote
#           -----------
#            t
#           (1 row)

# Step 3: Verify promotion
psql -h standby -c "SELECT pg_is_in_recovery();"
# Expected: f (false — no longer in recovery)

# Step 4: Redirect application connections to new primary
# Update DNS or load balancer configuration
```

**Why This Output Occurs**: `pg_promote()` triggers the standby to exit recovery mode and begin accepting writes. `pg_is_in_recovery()` returning `f` confirms the promotion succeeded.

#### Example 2: Patroni Automatic Failover

```bash
# Step 1: Verify cluster state
patronictl -c /etc/patroni.yml list
```

**Expected Output**:
```
+ Cluster: postgres (1234567890123456789) ---+----+-----------+
| Member | Host  | Role    | State     | TL | Lag in MB |
+--------+-------+---------+-----------+----+-----------+
| node1  | node1 | Leader  | running   |  1 |           |
| node2  | node2 | Replica | streaming |  1 |         0 |
| node3  | node3 | Replica | streaming |  1 |         0 |
+--------+-------+---------+-----------+----+-----------+
```

**Why This Output Occurs**: Patroni uses etcd for distributed consensus. When the leader (`node1`) fails, Patroni detects the failure via etcd lease expiration, elects the most up-to-date replica as the new leader, and promotes it automatically.

### Real-World Cases

**Case 1: Planned Maintenance with Zero Downtime**: A DBA performs OS patching on the primary server. Using manual failover, the standby is promoted, applications are redirected, and the old primary is patched and reconfigured as a standby—all without service interruption.

**Case 2: Automatic Failover in Cloud RDS**: An AWS RDS Multi-AZ deployment automatically fails over to the standby when the primary fails. The failover completes in under 35 seconds.

**Case 3: Forced Failover During Regional Outage**: A primary region becomes unreachable. The DBA performs a forced failover to a cross-region standby, accepting potential data loss to restore service.

---

## Core Concept 6: Replication Topologies

### Definitions

**Core Definition**: Replication topology describes the arrangement of primary and replica nodes and the direction of data flow between them.

**Technical Definition**: Replication topologies include single-primary/multi-replica (one writer, many readers), multi-primary/active-active (multiple writers), and cascading replication (standbys acting as relays for downstream standbys), each with distinct consistency and conflict characteristics.

**Beginner-Friendly Explanation**: Topology is like the layout of a phone tree. In a single-primary setup, one person makes calls and everyone else listens. In active-active, anyone can make calls, but you need rules to handle when two people call at once.

### Purposes

- **To** match replication architecture to workload requirements (read-heavy vs. write-heavy)
- **To** minimize network bandwidth by cascading replication through intermediate nodes
- **To** enable active-active writes for geographically distributed applications
- **To** provide flexible failover and scaling options

### Sub-Concept 6.1: Single-Primary/Multi-Replica

#### Syntax Rules and Structure (PostgreSQL Streaming Replication)

```conf
# On primary: allow multiple standbys
max_wal_senders = 10
wal_level = replica
```

#### Constraints and Limitations

- All writes must go to the single primary
- Read replicas may serve stale data
- Failover requires promoting one replica to primary

### Sub-Concept 6.2: Cascading Replication

#### Definitions

**Cascading Replication**: A standby server accepts replication connections and streams WAL records to other standbys, acting as a relay.

#### Syntax Rules and Structure (PostgreSQL)

```conf
# On cascading standby (node2): accept replication connections
max_wal_senders = 5
hot_standby = on

# On downstream standby (node3): point to cascading standby
primary_conninfo = 'host=node2 port=5432 user=replicator'
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `max_wal_senders` | Enables the cascading standby to send WAL to downstream nodes |
| `primary_conninfo` | Points downstream standby to the cascading standby |

#### Constraints and Limitations

- Cascading replication is currently asynchronous; synchronous settings have no effect
- If the upstream standby is promoted, downstream servers continue streaming if `recovery_target_timeline = 'latest'`
- Cascading reduces direct connections to the primary and minimizes inter-site bandwidth

### Sub-Concept 6.3: Multi-Primary/Active-Active

#### Definitions

**Active-Active Replication**: Multiple nodes accept writes simultaneously, requiring conflict detection and resolution.

#### Syntax Rules and Structure (MySQL Group Replication)

```sql
-- Configure group replication
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_single_primary_mode = OFF;  -- Multi-primary
SET GLOBAL group_replication_enforce_update_everywhere_checks = ON;
START GROUP_REPLICATION;
```

#### Constraints and Limitations

- Conflicts are detected at row level during certification; the transaction ordered first commits, the second aborts
- MySQL Group Replication is an eventual consistency system; as traffic slows, all members converge
- Active-active requires application-level conflict tolerance or CRDT-based data types

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Cascading Replication

```conf
# Step 1: On primary (node1), configure
wal_level = replica
max_wal_senders = 10

# Step 2: On cascading standby (node2), configure
max_wal_senders = 5
hot_standby = on
primary_conninfo = 'host=node1 port=5432 user=replicator'

# Step 3: On downstream standby (node3), configure
primary_conninfo = 'host=node2 port=5432 user=replicator'
```

```bash
# Step 4: Verify cascading replication
psql -h node1 -c "SELECT application_name, state FROM pg_stat_replication;"
# Expected: Shows node2 as the only direct standby

psql -h node2 -c "SELECT application_name, state FROM pg_stat_replication;"
# Expected: Shows node3 as the downstream standby
```

**Why This Output Occurs**: The cascading standby (`node2`) receives WAL from the primary and forwards it to `node3`. The primary only maintains one direct replication connection, reducing load.

#### Example 2: MySQL Group Replication Multi-Primary

```sql
-- Step 1: Configure multi-primary mode
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_single_primary_mode = OFF;
SET GLOBAL group_replication_enforce_update_everywhere_checks = ON;
START GROUP_REPLICATION;

-- Step 2: Verify group membership
SELECT member_id, member_host, member_state, member_role
FROM performance_schema.replication_group_members;
```

**Expected Output**:
```
 member_id | member_host | member_state | member_role
-----------+-------------+--------------+------------
 uuid-1    | node1       | ONLINE       | PRIMARY
 uuid-2    | node2       | ONLINE       | PRIMARY
 uuid-3    | node3       | ONLINE       | PRIMARY
```

**Why This Output Occurs**: In multi-primary mode, all nodes have `member_role = PRIMARY`, meaning any node can accept writes. Conflicts are resolved through certification.

### Real-World Cases

**Case 1: Cascading Replication for Bandwidth Savings**: A company with a primary in `us-east` and 20 read replicas in `eu-west` uses one cascading relay in `eu-west` to avoid 20 direct cross-Atlantic connections.

**Case 2: Active-Active for Global Writes**: A global collaboration platform uses MySQL Group Replication in multi-primary mode to allow writes from `us-east` and `eu-west`, with conflict resolution ensuring convergent state.

**Case 3: Read Replica Fan-Out**: A SaaS platform uses single-primary/multi-replica topology with 10 read replicas, routing analytics queries to a subset and transactional reads to another.

---

## Core Concept 7: Conflict Resolution

### Definitions

**Core Definition**: Conflict resolution is the set of rules and algorithms that determine how concurrent writes to the same data are reconciled in multi-primary replication systems.

**Technical Definition**: Conflict resolution encompasses last-write-wins (LWW) based on timestamps or logical clocks, vector clocks for causality tracking, and Conflict-free Replicated Data Types (CRDTs) that guarantee eventual convergence without explicit conflict resolution.

**Beginner-Friendly Explanation**: Conflict resolution is like two people editing the same document at the same time. You need a rule to decide whose changes win—either the last edit wins, or you use a special document type that automatically merges both edits.

### Purposes

- **To** determine a consistent state when concurrent writes occur in multi-primary systems
- **To** preserve data integrity without requiring manual intervention
- **To** enable active-active replication with acceptable consistency guarantees
- **To** support geographic distribution with local write latency

### Sub-Concept 7.1: Last-Write-Wins (LWW)

#### Definitions

**Core Definition**: LWW resolves conflicts by accepting the write with the latest timestamp (or logical clock value).

**Technical Definition**: LWW uses a total order (timestamp, logical clock, or sequence number) to determine which of two concurrent writes prevails; the later write is applied, and the earlier is discarded.

#### Syntax Rules and Structure (SQL Server Peer-to-Peer)

```sql
-- Configure last-writer-wins conflict resolution
EXEC sp_configure 'p2p_conflictdetection_policy', 'lastwriter';
```

#### Constraints and Limitations

- LWW can silently discard data if clocks are unsynchronized
- Clock skew between nodes can lead to incorrect winner selection
- LWW provides eventual consistency but not causal consistency

### Sub-Concept 7.2: Vector Clocks

#### Definitions

**Core Definition**: Vector clocks track causality between events in distributed systems, enabling detection of concurrent vs. causally ordered writes.

**Technical Definition**: A vector clock is a vector of integers, one entry per node, where each entry represents the highest logical sequence number received from that node. If one vector dominates another, the corresponding event is causally later.

#### Constraints and Limitations

- Vector clocks grow with the number of nodes, adding metadata overhead
- Conflicts are detected but not automatically resolved; application logic or CRDTs must decide
- Used in Redis Active-Active and ThemisDB multi-master replication

### Sub-Concept 7.3: CRDTs (Conflict-free Replicated Data Types)

#### Definitions

**Core Definition**: CRDTs are data structures designed to be replicated across multiple nodes and merged deterministically without conflicts.

**Technical Definition**: CRDTs (e.g., G-Counter, PN-Counter, OR-Set, LWW-Register) provide strong eventual consistency: once all updates are delivered, all replicas converge to the same state regardless of delivery order.

**Beginner-Friendly Explanation**: CRDTs are like special containers that automatically merge their contents when combined. A counter CRDT adds up all increments; a set CRDT unions all additions. No conflicts occur because the merge operation is designed to be associative and commutative.

#### Syntax Rules and Structure (Redis Active-Active)

```bash
# Redis Active-Active uses CRDTs automatically
redis-cli -h redis-cluster -p 6379
> INCR counter  # Uses CRDT counter semantics
```

#### Constraints and Limitations

- CRDTs are not suitable for all data types (e.g., arbitrary SQL constraints)
- Some CRDTs require tombstones for deletions, consuming storage
- Redis Active-Active databases use CRDTs with vector clocks for causality

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MySQL Group Replication Conflict Detection

```sql
-- Step 1: Create a table in multi-primary group replication
CREATE TABLE inventory (
    item_id INT PRIMARY KEY,
    quantity INT
);
INSERT INTO inventory VALUES (1, 100);
```

```sql
-- Step 2: Concurrent updates on two different nodes
-- Node1:
UPDATE inventory SET quantity = 90 WHERE item_id = 1;
-- Node2 (concurrently):
UPDATE inventory SET quantity = 80 WHERE item_id = 1;

-- Step 3: Check the outcome after certification
SELECT * FROM inventory WHERE item_id = 1;
```

**Expected Output** (one node's transaction aborts):
```
ERROR 3101 (HY000): Plugin instructed the server to rollback the current transaction.
```

**Why This Output Occurs**: Both transactions update the same row (`item_id = 1`). During certification, the transaction ordered second is aborted because it conflicts with the first.

#### Example 2: Redis CRDT Counter

```bash
# Step 1: Connect to Active-Active database
redis-cli -h redis-east.example.com

# Step 2: Increment counter from two regions
# From us-east:
INCR global_counter
# From eu-west (concurrently):
INCR global_counter

# Step 3: Read converged value
GET global_counter
```

**Expected Output**:
```
(integer) 2
```

**Why This Output Occurs**: The CRDT counter merges both increments (1 + 1 = 2) regardless of the order in which they are replicated. No conflict occurs because increments are commutative.

### Real-World Cases

**Case 1: Multi-Region Active-Active with CRDTs**: A global gaming platform uses Redis Active-Active with CRDTs for player scores and leaderboards, allowing writes from any region with automatic conflict resolution.

**Case 2: MySQL Group Replication for Fault Tolerance**: A fintech startup uses MySQL Group Replication in single-primary mode with automatic failover; conflict resolution handles the rare case of split-brain during network partitions.

**Case 3: SQL Server Peer-to-Peer with LWW**: A distributed retail chain uses SQL Server peer-to-peer replication with LWW conflict resolution for inventory updates across stores.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — Synchronous Replication | https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION |
| MySQL 8.0 Reference Manual — Semisynchronous Replication Configuration | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/replication-semisync-interface.html |
| PostgreSQL Documentation — Cascading Replication | https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.5/postgresql-9.5-A4.pdf |
| MySQL 8.0 Reference Manual — Checking Replication Status | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/replication-administration-status.html |
| MySQL 9.7 Reference Manual — Group Replication | https://docs.oracle.com/cd/E17952_01/mysql-9.7-en/group-replication-summary.html |
| Microsoft Learn — Failover Modes for Availability Groups | https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/failover-and-failover-modes-always-on-availability-groups |
| Redis Documentation — Active-Active Geo-Distributed | https://redis-io.analytics-portals.com/docs/latest/operate/rs/databases/active-active/ |
| PostgreSQL Documentation — Logical Replication Conflicts | https://www.postgresql.org/docs/current/logical-replication-conflicts.html |
| AWS RDS — Read Replicas | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html |
| Google Cloud SQL — Read Pools | https://cloud.google.com/sql/docs/mysql/read-replicas |
| CockroachDB — Follower Reads | https://www.cockroachlabs.com/docs/stable/follower-reads.html |
| Patroni Documentation | https://patroni.readthedocs.io/ |
| PostgreSQL Documentation — Monitoring Streaming Replication | https://www.postgresql.org/docs/current/monitoring-stats.html |
| ACM — Conflict-free Replicated Data Types | https://arxiv.org/abs/1805.06358 |