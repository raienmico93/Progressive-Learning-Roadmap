# SQL High Availability: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL High Availability (HA) is the set of architectural patterns, protocols, and operational practices that ensure a database system remains accessible and operational despite component failures.

**Technical Definition**: High Availability in database systems encompasses replication topologies (synchronous, asynchronous, semi-synchronous), failover orchestration (manual, automatic, forced), redundancy configurations (multi-AZ, geographic distribution), consensus-based automatic recovery (Raft, Paxos, quorum voting), and connection-level resilience (pooling, load balancing, seamless reconnection) that collectively minimize downtime and data loss.

**Beginner-Friendly Explanation**: High Availability is like having a backup generator for your database. If the main power (primary server) fails, the backup (standby replica) takes over automatically, so your applications keep running without interruption.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Availability Target** | Expressed as "nines" (99.9%, 99.99%, 99.999%) |
| **Failover Time** | Time to detect failure and promote a standby (seconds to minutes) |
| **Data Consistency** | Synchronous guarantees zero data loss; asynchronous allows some loss |
| **Topology** | Primary-replica, multi-primary, shared-nothing clusters |
| **Split-Brain Prevention** | Quorum voting, fencing, STONITH, epoch management |
| **Client Transparency** | Applications reconnect seamlessly via load balancers, proxies, or DNS |

### Prerequisites

- **Multiple Database Nodes**: At least two servers (primary + standby) for basic HA
- **Replication Configuration**: Streaming replication (PostgreSQL), binary log replication (MySQL), or Always On Availability Groups (SQL Server)
- **Network Infrastructure**: Low-latency connectivity between nodes; redundant network paths
- **Cluster Manager**: Pacemaker, Corosync, WSFC, or cloud-native orchestration
- **Monitoring and Health Checks**: Heartbeat mechanisms, failure detectors, and alerting
- **Connection Routing Layer**: Load balancer, proxy (HAProxy, PgBouncer, MySQL Router), or DNS-based failover

### Related Programming Areas

- **Distributed Systems**: Consensus algorithms, CAP theorem, replication protocols
- **Database Administration (DBA)** : Replication monitoring, failover drills, capacity planning
- **Cloud Engineering**: Multi-AZ deployments, managed HA services (RDS Multi-AZ, Azure SQL)
- **Site Reliability Engineering (SRE)** : SLO/SLI definition, chaos engineering, incident response
- **Network Engineering**: Load balancing, DNS failover, firewall rules for replication traffic

### Core Concepts Overview

SQL High Availability comprises multiple complementary techniques:

1. **Replication**: Synchronous, asynchronous, and semi-synchronous data propagation
2. **Primary-Replica Architectures**: Multi-replica topologies and read-replica scaling
3. **Failover**: Manual, automatic, and forced failover mechanisms
4. **Redundancy**: Multi-AZ deployment, geographic distribution, power/network redundancy
5. **Automatic Recovery**: Self-healing nodes, split-brain prevention, quorum/consensus protocols
6. **Connection Pooling & Load Balancing**: Traffic routing and seamless client reconnection

---

## Core Concept 1: Replication

### Definitions

**Core Definition**: Replication is the process of copying and maintaining database objects across multiple database servers.

**Technical Definition**: Database replication involves propagating data changes from a primary (source) database to one or more replicas (standbys) using mechanisms including Write-Ahead Log (WAL) shipping, binary log replication, or log-based change data capture, with consistency guarantees ranging from asynchronous (best-effort) to synchronous (transactional guarantee) .

**Beginner-Friendly Explanation**: Replication is like making photocopies of your database as it changes. Every time you write something new, the copy machine (replication process) makes sure all the copies get the same update—either right away (synchronous) or a little later (asynchronous).

### Purposes

- **To** maintain identical copies of data across multiple servers for redundancy
- **To** offload read-only workloads to replicas, improving primary performance
- **To** enable failover by ensuring a standby has current data
- **To** support geographic distribution for latency reduction and disaster recovery

### Sub-Concept 1.1: Synchronous Replication

#### Definitions

**Core Definition**: Synchronous replication guarantees that a transaction is committed on the primary only after it has been confirmed on at least one standby.

**Technical Definition**: In synchronous replication, the primary waits for acknowledgment from standby server(s) that the transaction's WAL records or binary log events have been received and persisted before returning commit confirmation to the client .

**Beginner-Friendly Explanation**: Synchronous replication means the primary says "I've saved this" only after at least one backup has also said "I've saved this." Nothing is lost if the primary crashes immediately after.

#### Syntax Rules and Structure (PostgreSQL)

```conf
# postgresql.conf (on primary and standby)
synchronous_standby_names = 'standby1,standby2'
synchronous_commit = on
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `synchronous_standby_names` | List of standby servers that can acknowledge commits |
| `synchronous_commit` | Controls when commit returns: `on`, `remote_apply`, `remote_write`, `local`, `off` |

#### Syntax Rules (MySQL Semisynchronous)

```sql
-- On source server
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
SET GLOBAL rpl_semi_sync_source_enabled = 1;
SET GLOBAL rpl_semi_sync_source_timeout = 1000;  -- milliseconds

-- On replica server
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
SET GLOBAL rpl_semi_sync_replica_enabled = 1;
```

#### Constraints and Limitations

- Synchronous replication increases transaction latency by at least the round-trip time to the standby
- If the synchronous standby fails, the primary may block or degrade (configurable behavior) 
- Minimum wait time is the round-trip time from primary to standby 

### Sub-Concept 1.2: Asynchronous Replication

#### Definitions

**Core Definition**: Asynchronous replication commits transactions on the primary without waiting for standby acknowledgment.

**Technical Definition**: In asynchronous replication, the primary writes to its local WAL/log and returns commit confirmation immediately; standby servers receive and apply changes independently, introducing replication lag .

**Beginner-Friendly Explanation**: Asynchronous replication means the primary says "I've saved this" right away, without waiting to hear from backups. It's faster, but if the primary crashes before backups catch up, some recent data might be lost.

#### Syntax Rules and Structure (PostgreSQL Default)

```conf
# postgresql.conf (default)
synchronous_commit = off
# No synchronous_standby_names required
```

#### Constraints and Limitations

- Potential data loss if primary fails before standby receives changes 
- Amount of data loss is proportional to replication delay at time of failure 
- Replication lag can grow under heavy write loads

### Sub-Concept 1.3: Semi-Synchronous Replication

#### Definitions

**Core Definition**: Semi-synchronous replication is a middle ground where at least one replica must acknowledge receipt, but full synchronization is not required.

**Technical Definition**: In semi-synchronous replication, the primary waits for at least one replica to confirm receipt of the transaction (not necessarily application), providing improved data integrity compared to asynchronous while maintaining better performance than fully synchronous .

**Beginner-Friendly Explanation**: Semi-synchronous is like a compromise: the primary waits for one backup to say "I got it" (not "I've fully applied it"), then commits. It's safer than async but faster than full sync.

#### Syntax Rules and Structure (MySQL)

```sql
-- Configure semi-synchronous replication
SET GLOBAL rpl_semi_sync_source_enabled = 1;
SET GLOBAL rpl_semi_sync_source_timeout = 1000;
-- After timeout, degrades to asynchronous replication
```

#### Constraints and Limitations

- If the semi-synchronous replica becomes slow or unavailable, the primary degrades to asynchronous replication 
- If timeout expires before acknowledgment, replication falls back to asynchronous 
- Only one replica needs to acknowledge (configurable in some systems)

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

**Why This Output Occurs**: `sync_state = 'sync'` confirms that `pg-standby` is the synchronous standby. Commits on the primary will wait for this standby to acknowledge receipt of WAL records .

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

**Why This Output Occurs**: The semi-synchronous plugin is active and waiting for at least one replica acknowledgment before returning commit success to clients .

### Real-World Cases

**Case 1: Financial Transaction Processing**
A banking application uses synchronous replication to guarantee zero data loss. Every transaction is confirmed on at least two servers before the customer sees a success message.

**Case 2: E-Commerce Read Scaling**
An e-commerce platform uses asynchronous replication to create read replicas. Product catalog queries are served from replicas, reducing primary load by 70%.

**Case 3: MySQL Semi-Synchronous for Balanced Durability**
A SaaS application uses semi-synchronous replication to balance durability and performance. If the replica falls behind, the system degrades gracefully to asynchronous mode .

---

## Core Concept 2: Primary-Replica Architectures

### Definitions

**Core Definition**: Primary-replica architecture designates one database server as the primary (read-write) and one or more servers as replicas (read-only or standby).

**Technical Definition**: In primary-replica (leader-follower) replication, the primary processes all write operations and propagates changes to replicas; replicas can serve read-only queries, provide failover targets, and enable geographic distribution .

**Beginner-Friendly Explanation**: One server is the "boss" (primary) that handles all changes. The other servers are "helpers" (replicas) that copy the boss's work and can answer questions (read queries) so the boss isn't overloaded.

### Purposes

- **To** scale read workloads horizontally by distributing queries across replicas
- **To** provide failover targets for high availability
- **To** reduce primary server load by offloading reporting and analytics
- **To** enable geographic distribution for latency reduction

### Sub-Concept 2.1: Multi-Replica Topologies

#### Syntax Rules and Structure (SQL Server Always On)

```sql
-- Read-scale availability group (no cluster required, SQL Server 2017+)
CREATE AVAILABILITY GROUP [ReadScaleAG]
WITH (CLUSTER_TYPE = NONE)
FOR REPLICA ON
    N'primary' WITH (
        ENDPOINT_URL = 'TCP://primary:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = MANUAL,
        SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY)
    ),
    N'replica1' WITH (
        ENDPOINT_URL = 'TCP://replica1:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = MANUAL,
        SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY)
    ),
    N'replica2' WITH (
        ENDPOINT_URL = 'TCP://replica2:5022',
        AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,
        FAILOVER_MODE = MANUAL,
        SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY)
    );
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `CLUSTER_TYPE = NONE` | No cluster manager required (read-scale only, no automatic failover) |
| `AVAILABILITY_MODE` | `SYNCHRONOUS_COMMIT` (zero data loss) or `ASYNCHRONOUS_COMMIT` |
| `FAILOVER_MODE` | `MANUAL` (for read-scale AGs) |
| `SECONDARY_ROLE` | `ALLOW_CONNECTIONS = READ_ONLY` enables read access |

#### Syntax Rules (MySQL Group Replication)

```sql
-- Start Group Replication on each node
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_start_on_boot = ON;
SET GLOBAL group_replication_local_address = 'node1:33061';
SET GLOBAL group_replication_group_seeds = 'node1:33061,node2:33061,node3:33061';
START GROUP_REPLICATION;
```

#### Constraints and Limitations

- SQL Server read-scale AGs without cluster managers provide **read-scale only**, not high availability (no automatic failover) 
- MySQL Group Replication requires a connector, load balancer, or router to redirect clients on failure 
- Replica lag can cause stale reads if applications are not designed for eventual consistency

### Sub-Concept 2.2: Read-Replica Scaling

#### Definitions

**Core Definition**: Read-replica scaling distributes read-only workloads across multiple replicas to improve throughput.

**Technical Definition**: Read-scale out involves routing SELECT queries to secondary replicas while directing all writes to the primary, using load balancers or connection routers to distribute read traffic .

**Beginner-Friendly Explanation**: Instead of making one server answer all questions, you have several servers that each answer some questions. This way, no single server gets overwhelmed.

#### Syntax Rules and Structure (PostgreSQL with PgBouncer)

```ini
; pgbouncer.ini
[databases]
mydb = host=primary port=5432
mydb_read = host=replica1 port=5432

[pgbouncer]
listen_port = 6432
pool_mode = transaction
```

#### Constraints and Limitations

- Applications must be designed to tolerate replication lag (eventual consistency)
- Read replicas may return stale data if replication lag exceeds acceptable thresholds
- Some queries (those requiring read-your-writes consistency) must be routed to the primary

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Read-Scale Availability Group

```sql
-- Step 1: Enable Always On on all replicas
ALTER SERVER CONFIGURATION SET HADR CLUSTER CONTEXT = 'OFF';
-- For read-scale AG (no cluster):
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'hadr enabled', 1;
RECONFIGURE;

-- Step 2: Create the read-scale availability group (see syntax above)

-- Step 3: Configure read-only routing
ALTER AVAILABILITY GROUP [ReadScaleAG]
MODIFY REPLICA ON N'primary' WITH
    (SECONDARY_ROLE (READ_ONLY_ROUTING_URL = 'TCP://primary:1433'));

ALTER AVAILABILITY GROUP [ReadScaleAG]
MODIFY REPLICA ON N'replica1' WITH
    (SECONDARY_ROLE (READ_ONLY_ROUTING_URL = 'TCP://replica1:1433'));

-- Step 4: Create listener and routing list
ALTER AVAILABILITY GROUP [ReadScaleAG]
ADD LISTENER 'readscale-listener' (
    WITH IP (('10.0.0.100', '255.255.255.0')),
    PORT = 1433
);

ALTER AVAILABILITY GROUP [ReadScaleAG]
MODIFY REPLICA ON N'primary' WITH
    (PRIMARY_ROLE (READ_ONLY_ROUTING_LIST = ('replica1', 'replica2')));
```

**Expected Output**:
```
Commands completed successfully.
```

**Why This Output Occurs**: The read-only routing list directs read-intent connections from the primary to `replica1` and `replica2` in round-robin fashion. Applications connect with `ApplicationIntent=ReadOnly` to use this routing .

#### Example 2: PostgreSQL Read Replica with PgBouncer

```bash
# Step 1: Set up streaming replication
# On primary: create replication user
CREATE USER replicator WITH REPLICATION PASSWORD 'secret';

# Step 2: On replica: create base backup
pg_basebackup -h primary -D /var/lib/postgresql/14/main -U replicator -P -X stream -R

# Step 3: Configure PgBouncer
cat > /etc/pgbouncer/pgbouncer.ini << EOF
[databases]
mydb = host=primary port=5432 dbname=mydb
mydb_read = host=replica port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 50
EOF

# Step 4: Start PgBouncer
sudo systemctl start pgbouncer
```

```sql
-- Step 5: Verify read routing
-- Application connects to port 6432, database mydb_read for reads
SELECT inet_server_addr();  -- Should return replica IP
```

**Why This Output Occurs**: PgBouncer routes connections to `mydb` to the primary and `mydb_read` to the replica. Applications use different connection strings for read and write operations .

### Real-World Cases

**Case 1: SQL Server Reporting Offload**
A retail company configures a read-scale availability group with three secondary replicas. Daily sales reports run against replicas, reducing primary CPU load by 40% .

**Case 2: MySQL InnoDB Cluster for Read Scaling**
An e-commerce platform uses MySQL InnoDB Cluster with MySQL Router. Read queries are distributed across three replicas, while writes go to the primary .

**Case 3: PostgreSQL Analytics Replica**
A SaaS company routes all analytics queries to a dedicated read replica, isolating the primary from expensive aggregation queries and maintaining low-latency transaction processing.

---

## Core Concept 3: Failover

### Definitions

**Core Definition**: Failover is the process of transferring the primary role from one database server to another.

**Technical Definition**: Failover encompasses the detection of primary failure, selection of a new primary from available replicas, promotion of that replica, and redirection of client connections, with three variants: automatic (no data loss), planned manual (no data loss), and forced manual (potential data loss) .

**Beginner-Friendly Explanation**: Failover is like a relay race—when the current runner (primary) can't continue, the next runner (standby) picks up the baton and keeps going, so the race (application) doesn't stop.

### Purposes

- **To** restore service availability after primary server failure
- **To** enable planned maintenance on the primary with zero downtime
- **To** provide a controlled mechanism for disaster recovery
- **To** ensure that a designated standby can assume the primary role

### Sub-Concept 3.1: Manual Failover

#### Definitions

**Core Definition**: Manual failover is initiated by an administrator or operator, typically during planned maintenance.

**Technical Definition**: Planned manual failover involves synchronizing the standby with the primary, verifying no data loss will occur, then promoting the standby and redirecting clients .

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

### Sub-Concept 3.2: Automatic Failover

#### Definitions

**Core Definition**: Automatic failover occurs without human intervention when the cluster detects primary failure.

**Technical Definition**: Automatic failover is triggered by a cluster manager (WSFC, Pacemaker, Patroni) that detects primary unavailability via health checks, verifies quorum, promotes the most up-to-date standby, and redirects connections .

#### Syntax Rules and Structure (SQL Server Always On)

```sql
-- Configure automatic failover
ALTER AVAILABILITY GROUP [MyAG]
MODIFY REPLICA ON N'standby1' WITH
    (FAILOVER_MODE = AUTOMATIC);
```

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

#### Constraints and Limitations

- Automatic failover requires a quorum or witness to prevent split-brain 
- The standby must be synchronous for zero data loss automatic failover
- A brief service interruption occurs during failover (typically seconds) 
- Automatic failover is not available in read-scale-only configurations 

### Sub-Concept 3.3: Forced Failover

#### Definitions

**Core Definition**: Forced failover promotes a standby even when synchronization is not confirmed, risking data loss.

**Technical Definition**: Forced manual failover (also called forced failover) is used in disaster recovery scenarios when the primary is unreachable and the standby may not have all transactions, resulting in potential data loss .

#### Syntax Rules and Structure (SQL Server)

```sql
-- Forced failover (potential data loss)
ALTER AVAILABILITY GROUP [MyAG] FORCE_FAILOVER_ALLOW_DATA_LOSS;
```

#### Constraints and Limitations

- Data loss is guaranteed if the standby is not fully synchronized
- Should only be used in disaster recovery scenarios
- The old primary must be prevented from rejoining (fencing or rebuild)

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

**Why This Output Occurs**: `pg_promote()` triggers the standby to exit recovery mode and begin accepting writes. `pg_is_in_recovery()` returning `f` confirms the promotion succeeded .

#### Example 2: SQL Server Automatic Failover

```sql
-- Step 1: Configure automatic failover on a synchronous replica
ALTER AVAILABILITY GROUP [MyAG]
MODIFY REPLICA ON N'standby1' WITH
    (AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
     FAILOVER_MODE = AUTOMATIC);

-- Step 2: Verify failover readiness
SELECT 
    ag.name AS ag_name,
    ar.replica_server_name,
    ar.availability_mode_desc,
    ar.failover_mode_desc,
    ars.role_desc,
    ars.synchronization_health_desc
FROM sys.dm_hadr_availability_replica_states ars
JOIN sys.availability_replicas ar 
    ON ars.replica_id = ar.replica_id
JOIN sys.availability_groups ag 
    ON ar.group_id = ag.group_id;
```

**Expected Output**:
```
ag_name | replica_server_name | availability_mode_desc | failover_mode_desc | role_desc | synchronization_health_desc
--------|---------------------|-----------------------|-------------------|-----------|---------------------------
MyAG    | primary             | SYNCHRONOUS_COMMIT    | AUTOMATIC         | PRIMARY   | HEALTHY
MyAG    | standby1            | SYNCHRONOUS_COMMIT    | AUTOMATIC         | SECONDARY | HEALTHY
```

**Why This Output Occurs**: Both replicas are configured for synchronous commit and automatic failover. `synchronization_health_desc = HEALTHY` confirms the standby is synchronized and ready to take over .

### Real-World Cases

**Case 1: Planned Maintenance with Zero Downtime**
A DBA performs OS patching on the primary server. Using manual failover, the standby is promoted, applications are redirected, and the old primary is patched and reconfigured as a standby—all without service interruption .

**Case 2: Automatic Failover in Cloud RDS**
An AWS RDS Multi-AZ deployment automatically fails over to the standby when the primary fails. The failover completes in under 35 seconds .

**Case 3: Forced Failover During Regional Outage**
A primary region becomes unreachable. The DBA performs a forced failover to a cross-region standby, accepting potential data loss to restore service.

---

## Core Concept 4: Redundancy

### Definitions

**Core Definition**: Redundancy is the duplication of critical components to eliminate single points of failure.

**Technical Definition**: Database redundancy encompasses multi-AZ deployment (physically separate data centers within a region), geographic distribution (cross-region replication), and infrastructure redundancy (independent power, cooling, and networking) .

**Beginner-Friendly Explanation**: Redundancy means having extra copies of everything—servers, power supplies, network connections—so that if one fails, another is already there to take over.

### Purposes

- **To** eliminate single points of failure across infrastructure layers
- **To** enable automatic failover during availability zone outages
- **To** provide disaster recovery across geographic regions
- **To** satisfy compliance requirements for data durability

### Sub-Concept 4.1: Multi-AZ Deployment

#### Definitions

**Core Definition**: Multi-AZ deployment places database replicas in physically separate availability zones within a cloud region.

**Technical Definition**: Availability Zones (AZs) are physically separate data centers with independent power, cooling, and networking; multi-AZ database deployments maintain synchronous replicas across AZs for automatic failover .

**Beginner-Friendly Explanation**: Multi-AZ means your database copies live in different buildings that have their own power and internet. If one building loses power, the other keeps running.

#### Syntax Rules and Structure (AWS RDS Multi-AZ)

```bash
aws rds create-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.r6g.xlarge \
    --engine postgres \
    --multi-az \
    --availability-zone us-east-1a \
    --backup-retention-period 7
```

#### Constraints and Limitations

- Multi-AZ provides high availability within a region but not cross-region disaster recovery 
- A large regional incident can affect all AZs within the region
- Cross-region DR requires additional configuration (see geographic distribution)

### Sub-Concept 4.2: Geographic Distribution

#### Definitions

**Core Definition**: Geographic distribution replicates databases across distant regions for disaster recovery.

**Technical Definition**: Geographic distribution (cross-region replication) maintains replicas in geographically separated regions, typically using asynchronous replication to tolerate high network latency .

#### Syntax Rules and Structure (Azure SQL Active Geo-Replication)

```sql
-- Configure active geo-replication
ALTER DATABASE mydb
ADD SECONDARY ON SERVER 'mydb-secondary' 
WITH (ALLOW_CONNECTIONS = READ_ONLY);

-- Configure failover group
CREATE FAILOVER GROUP myfailovergroup
WITH (
    PARTNER_SERVER = 'mydb-secondary',
    DATABASES = (mydb)
);
```

#### Constraints and Limitations

- Cross-region replication is typically asynchronous (latency makes synchronous impractical)
- Network latency between regions determines replication lag
- Cross-region data transfer incurs additional costs
- Failover groups automate connection routing for cross-region failover 

### Annotated Complete Step-by-Step Code Examples

#### Example 1: AWS RDS Multi-AZ Configuration

```bash
# Step 1: Create Multi-AZ RDS instance
aws rds create-db-instance \
    --db-instance-identifier prod-db \
    --db-instance-class db.r6g.2xlarge \
    --engine postgres \
    --multi-az \
    --allocated-storage 100 \
    --master-username admin \
    --master-user-password 'SecurePass123!' \
    --backup-retention-period 30

# Step 2: Verify Multi-AZ configuration
aws rds describe-db-instances \
    --db-instance-identifier prod-db \
    --query 'DBInstances[0].{MultiAZ:MultiAZ,AZ:AvailabilityZone,SecondaryAZ:SecondaryAvailabilityZone}'
```

**Expected Output**:
```json
{
    "MultiAZ": true,
    "AZ": "us-east-1a",
    "SecondaryAZ": "us-east-1b"
}
```

**Why This Output Occurs**: AWS RDS automatically provisions a synchronous standby in a different AZ (`us-east-1b`). If the primary AZ (`us-east-1a`) fails, RDS automatically fails over to the standby.

#### Example 2: Azure SQL Failover Group

```bash
# Step 1: Create failover group (PowerShell)
New-AzSqlDatabaseFailoverGroup `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -FailoverGroupName "myfailovergroup" `
    -PartnerServerName "myserver-secondary" `
    -FailoverPolicy Automatic `
    -GracePeriodInMinutes 60

# Step 2: Verify failover group
Get-AzSqlDatabaseFailoverGroup `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -FailoverGroupName "myfailovergroup"
```

**Expected Output**:
```
FailoverGroupName     : myfailovergroup
PartnerServerName     : myserver-secondary
FailoverPolicy        : Automatic
GracePeriodInMinutes  : 60
ReadWriteListener     : myfailovergroup.database.windows.net
```

**Why This Output Occurs**: The failover group provides a read-write listener endpoint that automatically redirects connections to the current primary. `FailoverPolicy = Automatic` enables automatic failover on primary unavailability .

### Real-World Cases

**Case 1: Multi-AZ for HA Within a Region**
A SaaS application deploys RDS Multi-AZ for its production database. When the primary AZ experiences a power outage, RDS automatically fails over to the standby AZ, maintaining service with minimal disruption .

**Case 2: Cross-Region DR with Azure Failover Groups**
A multinational company uses Azure SQL failover groups to replicate databases from `East US` to `West Europe`. If the US region fails, the failover group automatically redirects applications to the European replica.

**Case 3: Geographic Read Distribution**
A global e-commerce platform uses PostgreSQL logical replication to distribute read replicas to `us-east`, `eu-west`, and `ap-southeast` regions, reducing query latency for users worldwide.

---

## Core Concept 5: Automatic Recovery

### Definitions

**Core Definition**: Automatic recovery encompasses self-healing mechanisms that restore database service without human intervention.

**Technical Definition**: Automatic recovery includes self-healing node restart, split-brain prevention through quorum and fencing, and consensus-based leader election (Raft, Paxos) that ensures at most one primary is active in a partition .

**Beginner-Friendly Explanation**: Automatic recovery is the database's "immune system." If a node fails, the system automatically restarts it, elects a new leader, and ensures that no two nodes claim to be in charge at the same time.

### Purposes

- **To** restore service automatically after transient failures
- **To** prevent split-brain scenarios where two nodes accept writes
- **To** elect a new primary through consensus when the current leader fails
- **To** heal failed nodes by rejoining them as standbys

### Sub-Concept 5.1: Self-Healing Nodes

#### Definitions

**Core Definition**: Self-healing nodes automatically restart and rejoin the cluster after failure.

**Technical Definition**: Self-healing involves process monitoring (systemd, supervisord), automatic restart on crash, and automatic rejoin as a standby after restart, often using tools like `pg_rewind` (PostgreSQL) or `Clone Plugin` (MySQL) .

**Beginner-Friendly Explanation**: If a database node crashes, the system automatically restarts it and brings it back into the cluster as a standby, without needing an administrator.

#### Syntax Rules and Structure (systemd)

```ini
# /etc/systemd/system/postgresql.service
[Service]
Restart=on-failure
RestartSec=5s
ExecStart=/usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main
```

#### Constraints and Limitations

- Self-healing requires reliable process supervision and monitoring
- After restart, a node may need to catch up on replication before rejoining
- Automatic restart may not resolve underlying issues (e.g., disk failure)

### Sub-Concept 5.2: Split-Brain Prevention

#### Definitions

**Core Definition**: Split-brain prevention ensures that only one node can act as primary at any time.

**Technical Definition**: Split-brain prevention uses quorum voting (majority of nodes must agree), fencing (STONITH — Shoot The Other Node In The Head), and epoch/generation numbers to prevent two nodes from simultaneously accepting writes .

**Beginner-Friendly Explanation**: Split-brain is when two servers both think they're the boss. Prevention mechanisms—like voting and forcing one to shut down—ensure only one boss exists.

#### Syntax Rules and Structure (Pacemaker Fencing)

```bash
# Configure STONITH device
pcs stonith create fence_node1 fence_ipmilan \
    pcmk_host_list=node1 \
    ipaddr=10.0.0.101 \
    login=admin \
    passwd=secret \
    lanplus=1 \
    op monitor interval=60s

# Verify fencing configuration
pcs stonith show
```

#### Constraints and Limitations

- Fencing requires out-of-band management (IPMI, iDRAC, iLO) or power switches
- Unfenced old primary may still accept writes if network isolation occurs
- Quorum requires a minimum number of nodes (typically 3 for majority voting)

### Sub-Concept 5.3: Quorum and Consensus Protocols

#### Definitions

**Core Definition**: Consensus protocols like Raft and Paxos ensure that distributed nodes agree on a single leader and log state.

**Technical Definition**: Raft and Paxos are consensus algorithms that maintain a replicated log across nodes, elect a leader through majority quorum, and ensure that only the leader appends new entries, which are committed once a majority has persisted them .

**Beginner-Friendly Explanation**: Raft and Paxos are like democratic voting systems for servers. The majority (quorum) elects a leader, and the leader's decisions are only final when the majority agrees.

#### Syntax Rules and Structure (MySQL Group Replication)

```sql
-- Configure group replication with consensus
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_single_primary_mode = ON;
SET GLOBAL group_replication_enforce_update_everywhere_checks = OFF;
START GROUP_REPLICATION;

-- Check group status
SELECT * FROM performance_schema.replication_group_members;
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `group_replication_group_name` | UUID identifying the replication group |
| `group_replication_single_primary_mode` | `ON` = single-primary with automatic election |
| `replication_group_members` | Shows member state, role, and version |

#### Constraints and Limitations

- Raft and Paxos require a majority quorum to make progress
- Network partitions can split a cluster if quorum is lost
- MySQL Group Replication requires a connector (MySQL Router) for client redirection 

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MySQL Group Replication with Automatic Failover

```sql
-- Step 1: On each node, configure group replication
SET GLOBAL group_replication_group_name = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee';
SET GLOBAL group_replication_start_on_boot = ON;
SET GLOBAL group_replication_local_address = 'node1:33061';
SET GLOBAL group_replication_group_seeds = 'node1:33061,node2:33061,node3:33061';
SET GLOBAL group_replication_single_primary_mode = ON;
SET GLOBAL group_replication_enforce_update_everywhere_checks = OFF;

-- Step 2: Start group replication
START GROUP_REPLICATION;

-- Step 3: Verify group membership
SELECT 
    member_id,
    member_host,
    member_port,
    member_state,
    member_role
FROM performance_schema.replication_group_members;
```

**Expected Output**:
```
member_id | member_host | member_port | member_state | member_role
----------|-------------|-------------|--------------|------------
uuid-1    | node1       | 3306        | ONLINE       | PRIMARY
uuid-2    | node2       | 3306        | ONLINE       | SECONDARY
uuid-3    | node3       | 3306        | ONLINE       | SECONDARY
```

**Why This Output Occurs**: MySQL Group Replication uses a consensus protocol (similar to Raft) to elect a primary. When `node1` fails, the group automatically elects a new primary from the remaining nodes .

#### Example 2: Patroni Automatic Failover (PostgreSQL)

```yaml
# Step 1: Configure Patroni on all nodes
# patroni.yml (node1)
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

```bash
# Step 2: Start Patroni
sudo systemctl start patroni

# Step 3: Verify cluster state
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

**Case 1: MySQL Group Replication for Fault Tolerance**
A fintech startup uses MySQL Group Replication with three nodes. When the primary node fails, the group automatically elects a new primary within seconds, maintaining service availability .

**Case 2: Patroni for PostgreSQL HA**
A SaaS company runs Patroni across three PostgreSQL nodes with etcd. Automatic failover occurs in under 30 seconds when the primary fails, with no data loss (synchronous replication).

**Case 3: Raft-Based Distributed SQL**
CockroachDB and TiDB use Raft consensus for automatic failover and self-healing. When a node fails, the Raft group elects a new leader and rebalances data automatically .

---

## Core Concept 6: Connection Pooling & Load Balancing

### Definitions

**Core Definition**: Connection pooling and load balancing manage database connections to optimize resource usage and route traffic during failover.

**Technical Definition**: Connection pooling maintains a cache of reusable database connections, reducing connection overhead; load balancing distributes connections across replicas and redirects them during failover using proxies, DNS, or virtual IPs .

**Beginner-Friendly Explanation**: Connection pooling is like a shared taxi service—instead of creating a new car (connection) for every ride (query), you reuse cars from a pool. Load balancing is like a traffic officer directing cars to different lanes so no single lane is congested.

### Purposes

- **To** reduce connection establishment overhead and resource consumption
- **To** distribute read queries across multiple replicas
- **To** route traffic to the new primary during failover
- **To** enable seamless client reconnection without application changes

### Sub-Concept 6.1: Connection Pooling

#### Syntax Rules and Structure (PgBouncer)

```ini
; pgbouncer.ini
[databases]
mydb = host=primary port=5432

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 50
min_pool_size = 5
reserve_pool_size = 10
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `pool_mode` | `session` (connection per session), `transaction` (connection per transaction), `statement` |
| `max_client_conn` | Maximum client connections |
| `default_pool_size` | Connections per database/user pair |
| `reserve_pool_size` | Additional connections when pool is exhausted |

#### Constraints and Limitations

- Transaction pooling is incompatible with session-level features (prepared statements, advisory locks)
- Pooled connections may return stale data if replicas lag
- Connection lifetime management is essential to avoid stale connections 

### Sub-Concept 6.2: Load Balancing and Failover Routing

#### Syntax Rules and Structure (HAProxy)

```ini
# haproxy.cfg
frontend pgsql_frontend
    bind *:5432
    mode tcp
    default_backend pgsql_backend

backend pgsql_backend
    mode tcp
    option tcp-check
    tcp-check connect
    tcp-check send "SELECT 1\n"
    tcp-check expect string "1"
    server primary primary:5432 check
    server replica1 replica1:5432 check
    server replica2 replica2:5432 check
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `tcp-check` | Health check via TCP |
| `option tcp-check` | Enable TCP-level health checks |
| `server` | Backend server with health check enabled |

#### Syntax Rules and Structure (MySQL Router)

```ini
# mysqlrouter.conf
[routing:primary]
bind_address = 0.0.0.0
bind_port = 6446
destinations = metadata-cache://mycluster/default?role=PRIMARY
routing_strategy = first-available

[routing:secondary]
bind_address = 0.0.0.0
bind_port = 6447
destinations = metadata-cache://mycluster/default?role=SECONDARY
routing_strategy = round-robin
```

#### Constraints and Limitations

- Load balancers must be updated during failover (or use dynamic discovery)
- DNS-based failover may be limited by TTL caching
- Proxy layers add a single point of failure unless themselves redundant 

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PgBouncer with Read-Write Splitting

```ini
; Step 1: Configure PgBouncer
; /etc/pgbouncer/pgbouncer.ini
[databases]
mydb_rw = host=primary port=5432
mydb_ro = host=replica1 port=5432

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 500
default_pool_size = 25
```

```ini
; Step 2: Create user list
; /etc/pgbouncer/userlist.txt
"app_user" "md5passwordhash"
```

```bash
# Step 3: Start PgBouncer
sudo systemctl start pgbouncer

# Step 4: Verify routing
psql -h localhost -p 6432 -U app_user -d mydb_rw -c "SELECT inet_server_addr();"
# Expected: Returns primary IP

psql -h localhost -p 6432 -U app_user -d mydb_ro -c "SELECT inet_server_addr();"
# Expected: Returns replica IP
```

**Why This Output Occurs**: PgBouncer routes connections based on the database name. `mydb_rw` connects to the primary; `mydb_ro` connects to the replica. Applications use different connection strings for read and write operations .

#### Example 2: MySQL Router for Failover

```ini
# Step 1: Bootstrap MySQL Router
mysqlrouter --bootstrap root@primary:3306 --user=mysqlrouter

# Step 2: Start MySQL Router
systemctl start mysqlrouter

# Step 3: Verify routing
# Application connects to port 6446 for reads/writes
mysql -h router -P 6446 -u app_user -p

# Step 4: Verify connection routing
SELECT @@hostname;
```

**Expected Output**: Returns the hostname of the current primary.

**Why This Output Occurs**: MySQL Router reads the InnoDB Cluster metadata and automatically routes connections to the current primary (port 6446) or secondary (port 6447). When failover occurs, Router updates its routing tables automatically .

### Real-World Cases

**Case 1: PgBouncer for Connection Multiplexing**
A SaaS company with 10,000 application instances uses PgBouncer to multiplex connections to 50 database connections, reducing PostgreSQL memory usage by 90% .

**Case 2: HAProxy for PostgreSQL Failover**
A DBA uses HAProxy with health checks to detect primary failure and route connections to the promoted standby within seconds, without application changes.

**Case 3: MySQL Router for InnoDB Cluster**
An e-commerce platform uses MySQL Router to automatically redirect client connections during failover, providing seamless high availability without custom application logic .

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation - Synchronous Replication | https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION |
| PostgreSQL Documentation - Failover | https://www.postgresql.org/docs/current/warm-standby-failover.html |
| MySQL 8.0 Reference Manual - Group Replication | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/group-replication.html |
| MySQL 8.0 Reference Manual - Semisynchronous Replication | https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/replication-semisync.html |
| Microsoft Learn - SQL Server Always On Failover | https://learn.microsoft.com/en-us/sql/linux/business-continuity/availability-groups/failover-high-availability |
| Microsoft Learn - Read-Scale Availability Groups | https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/read-scale-availability-groups |
| Microsoft Learn - Azure SQL Global Availability | https://learn.microsoft.com/en-us/azure/azure-sql/database/designing-cloud-solutions-for-disaster-recovery |
| Oracle Docs - ODP.NET Fast Connection Failover | https://docs.oracle.com/en/database/oracle/oracle-database/21/admin/configuring-automatic-restart-of-an-oracle-database.html |
| Pacemaker Documentation - Quorum and Fencing | https://clusterlabs.org/pacemaker/doc/ |
| Patroni Documentation | https://patroni.readthedocs.io/ |
| HAProxy Configuration Manual | https://www.haproxy.org/download/2.8/doc/configuration.txt |
| PgBouncer Documentation | https://www.pgbouncer.org/config.html |