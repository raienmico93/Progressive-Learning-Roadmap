# SQL Recovery: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL recovery is the process of restoring a database to a functional, consistent state after data loss, corruption, or a disaster event.

**Technical Definition**: SQL recovery encompasses the set of operations—RESTORE commands, WAL replay, binary log application, and integrity verification—that reconstruct a database from backup artifacts and transaction logs, bringing it to a transactionally consistent state at a chosen recovery point .

**Beginner-Friendly Explanation**: Think of SQL recovery as the "undo" button for your entire database. When something goes wrong—a crash, a mistaken deletion, or a cyberattack—recovery uses the backup copies you made earlier to bring your database back to how it was before the problem happened.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Recovery Scope** | Full, partial, file/filegroup, page, or object-level restoration |
| **Recovery Point** | Specific point in time (PITR) or last available backup |
| **Recovery Chain** | Sequence of restore operations (full + differential + log) |
| **Recovery Model** | SQL Server: FULL, BULK_LOGGED, or SIMPLE; affects log backup availability |
| **Verification** | Checksums, RESTORE VERIFYONLY, and sandbox restore testing |
| **RPO/RTO** | Business-defined targets for data loss and downtime tolerance |

### Prerequisites

- **Valid Backup Artifacts**: Full backup, differential backups, and/or transaction log backups
- **Backup Chain Integrity**: Unbroken sequence of log backups for point-in-time recovery
- **Sufficient Storage**: Space for restored database files and temporary work areas
- **Appropriate Permissions**: `RESTORE` privileges or `sysadmin`/`dbcreator` roles
- **Recovery Model Configuration**: FULL or BULK_LOGGED for log-based PITR 
- **Network Access**: For restores from remote or cloud storage locations

### Related Programming Areas

- **Database Administration (DBA)**: Daily restore operations, recovery testing, troubleshooting
- **Disaster Recovery Engineering**: Failover orchestration, runbook automation, split-brain prevention
- **Site Reliability Engineering (SRE)**: RTO/RPO monitoring, chaos engineering for recovery validation
- **Compliance and Audit**: Recovery evidence collection, retention verification
- **Security Operations**: Ransomware recovery, forensic restoration from clean backups

### Core Concepts Overview

SQL recovery comprises multiple complementary techniques:

1. **Restore Operations**: Full, partial, and object-level restoration from backup artifacts
2. **Point-in-Time Recovery (PITR)** : Transaction log/WAL replay to a specific moment
3. **Recovery Objectives (RPO & RTO)** : Business-driven targets for data loss and downtime
4. **Disaster Recovery Planning**: Runbooks, multi-site strategies, and split-brain mitigation
5. **Backup Validation & Verification**: Integrity checks and non-disruptive sandbox testing
6. **Data Corruption Handling**: Block-level repair and page checksum management

---

## Core Concept 1: Restore Operations

### Definitions

**Core Definition**: Restore operations reconstruct a database from backup artifacts, ranging from full database restoration to selective object-level recovery.

**Technical Definition**: SQL Server RESTORE statements perform full database restoration (`RESTORE DATABASE`), partial restoration (`RESTORE DATABASE ... WITH PARTIAL`), file/filegroup restoration, page restoration, and transaction log restoration, each with distinct syntax and recovery point semantics .

**Beginner-Friendly Explanation**: Restore operations are the actual "putting back" process. You can restore everything (full), just some tables (partial), or even a single damaged page (page-level restore), depending on what went wrong.

### Purposes

- **To** reconstruct an entire database from a full backup
- **To** restore only specific files, filegroups, or pages when full recovery is unnecessary
- **To** recover individual objects (tables) from logical backups
- **To** bring a database to a consistent state after media failure

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server Full Restore)

```sql
RESTORE DATABASE database_name
FROM DISK = 'path_to_backup.bak'
WITH 
    MOVE 'logical_data_file' TO 'new_physical_path.mdf',
    MOVE 'logical_log_file' TO 'new_physical_path.ldf',
    RECOVERY | NORECOVERY,
    REPLACE | NO_REPLACE,
    STATS = percentage
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `RESTORE DATABASE` | Target database name |
| `FROM DISK` | Source backup device or URL |
| `MOVE` | Relocates logical files to new physical paths |
| `RECOVERY` | Brings database online after final restore step |
| `NORECOVERY` | Leaves database in restoring state for additional restores |
| `REPLACE` | Overwrites existing database without safety check |

#### Complete General Syntax (SQL Server Partial Restore)

```sql
RESTORE DATABASE database_name
FILEGROUP = 'filegroup_name'
FROM DISK = 'path_to_partial_backup.bak'
WITH 
    PARTIAL,
    NORECOVERY,
    STATS = percentage
```

#### Complete General Syntax (SQL Server Page Restore)

```sql
RESTORE DATABASE database_name
PAGE = 'file_id:page_id'
FROM DISK = 'path_to_backup.bak'
WITH 
    NORECOVERY | RECOVERY,
    STATS = percentage
```

#### Complete General Syntax (MySQL Table-Level Recovery)

```bash
mysqlbackup --defaults-file=/etc/my.cnf \
    --backup-dir=/backup \
    --include-tables='db_name.table_name' \
    copy-back-and-apply-log
```

#### Complete General Syntax (Azure SQL Point-in-Time Restore)

```bash
az sql db restore \
    --resource-group myResourceGroup \
    --server myserver \
    --name mydb \
    --dest-name mydb-restored \
    --time "2026-10-07T14:00:00Z"
```

#### Syntax Rules

- SQL Server: A restore sequence may include multiple `RESTORE` statements; intermediate restores use `NORECOVERY`, final restore uses `RECOVERY` 
- Partial restores require a partial backup or a full backup with `WITH PARTIAL` 
- Page restores require a full backup and transaction log backups covering the page changes 
- MySQL: Partial restores require `innodb_file_per_table` enabled and same page size 
- Azure SQL: Point-in-time restore creates a **new** database; you cannot overwrite an existing database 

#### Constraints and Limitations

- SQL Server partial restore: unrestored filegroups become read-only or defunct 
- MySQL partial restore: individual partitions cannot be selectively restored; encrypted InnoDB tables cannot be included; binary/relay/undo logs are not restored 
- Azure SQL: Restoring between Hyperscale and other service tiers is not supported 
- Page restore requires the page ID and file ID, obtainable from `msdb..suspect_pages` 
- Object-level restore from logical backups requires the original dump file and careful extraction

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Full Database Restore

**Setup**: A full backup file `TestFullBackup_Full.bak` exists.

```sql
-- Step 1: Verify the backup before restoring
RESTORE VERIFYONLY
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH CHECKSUM;
-- Expected: The backup set on file 1 is valid.
```

```sql
-- Step 2: Restore the database, relocating files to a new path
RESTORE DATABASE TestFullBackup
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH 
    MOVE 'TestFullBackup' TO 'C:\Restored\TestFullBackup.mdf',
    MOVE 'TestFullBackup_log' TO 'C:\Restored\TestFullBackup_log.ldf',
    RECOVERY,
    REPLACE,
    STATS = 10;
GO
```

**Expected Output**:
```
10 percent processed.
20 percent processed.
...
100 percent processed.
Processed 248 pages for database 'TestFullBackup', file 'TestFullBackup' on file 1.
Processed 2 pages for database 'TestFullBackup', file 'TestFullBackup_log' on file 1.
RESTORE DATABASE successfully processed 250 pages in 0.156 seconds (12.531 MB/sec).
```

**Why This Output Occurs**: The `MOVE` clauses redirect the logical file names to new physical paths, allowing restore to a different directory. `RECOVERY` brings the database online immediately after this single restore step. `REPLACE` allows overwriting an existing database of the same name.

#### Example 2: SQL Server Page-Level Restore

**Setup**: A page corruption is detected in file 1, page 177.

```sql
-- Step 1: Identify the suspect page
SELECT * FROM msdb.dbo.suspect_pages
WHERE database_id = DB_ID('TestFullBackup');
-- Expected: Shows page_id = 177, file_id = 1, event_type = 2 (checksum error)
```

```sql
-- Step 2: Restore the full backup with NORECOVERY (if database is offline)
RESTORE DATABASE TestFullBackup
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH NORECOVERY;
-- Then restore subsequent log backups with NORECOVERY
-- Finally, restore the specific page
RESTORE DATABASE TestFullBackup
PAGE = '1:177'
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH RECOVERY;
GO
```

**Expected Output**:
```
Processed 1 pages for database 'TestFullBackup', file 'TestFullBackup' on file 1.
RESTORE DATABASE ... PAGE successfully processed 1 pages in 0.031 seconds (0.250 MB/sec).
```

**Why This Output Occurs**: Page restore targets only the damaged page (file 1, page 177), restoring it from the backup. This is far faster than a full database restore and minimizes downtime .

#### Example 3: MySQL Table-Level Recovery

**Setup**: MySQL Enterprise Backup with a full backup containing an accidentally deleted `orders` table.

```bash
# Step 1: List available tables in the backup
mysqlbackup --backup-dir=/backup list-tables
# Expected: Shows orders table

# Step 2: Restore only the orders table
mysqlbackup --defaults-file=/etc/my.cnf \
    --backup-dir=/backup \
    --include-tables='inventory.orders' \
    copy-back-and-apply-log

# Expected Output:
# MySQL Enterprise Backup successfully restored table inventory.orders
```

**Why This Output Occurs**: `--include-tables` restricts the restore to the specified table. The destination server must already have the table defined (for non-TTS backups) or must not have it (for TTS backups) .

### Real-World Cases

**Case 1: Accidental Table Drop**
A developer accidentally drops the `customers` table. Using MySQL Enterprise Backup's table-level recovery, only that table is restored from the most recent backup, avoiding a full database restore and minimizing downtime .

**Case 2: Single Page Corruption**
A disk error corrupts one page in a large SQL Server database. Page-level restore recovers just that page from a backup, taking seconds instead of hours .

**Case 3: Azure SQL Point-in-Time Restore**
An application bug corrupts data at 2:00 PM. The DBA uses Azure SQL point-in-time restore to create a new database as of 1:59 PM, preserving the corrupted database for forensic analysis while restoring service .

---

## Core Concept 2: Point-in-Time Recovery (PITR)

### Definitions

**Core Definition**: Point-in-Time Recovery restores a database to a specific moment using transaction log or WAL archives.

**Technical Definition**: PITR combines a base backup with a continuous sequence of transaction log (SQL Server) or Write-Ahead Log (PostgreSQL) archives, replaying log records until a specified timestamp, LSN, or transaction ID is reached .

**Beginner-Friendly Explanation**: PITR lets you "rewind" your database to an exact moment—like 10:59 AM, one minute before someone accidentally deleted the wrong data. It uses the database's own change log to replay everything up to that point.

### Purposes

- **To** recover from logical errors (accidental DELETE, UPDATE, or DROP)
- **To** achieve near-zero RPO with continuous log archiving
- **To** restore to any point within the retention period
- **To** create a consistent snapshot for testing or reporting

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server PITR)

```sql
-- Restore full backup (NORECOVERY)
RESTORE DATABASE database_name
FROM DISK = 'full_backup.bak'
WITH NORECOVERY, REPLACE;

-- Restore differential (NORECOVERY) — optional
RESTORE DATABASE database_name
FROM DISK = 'differential.bak'
WITH NORECOVERY;

-- Restore log backups (NORECOVERY) until the target time
RESTORE LOG database_name
FROM DISK = 'log_backup.trn'
WITH 
    NORECOVERY,
    STOPAT = '2026-10-07T14:04:00';

-- Final restore (RECOVERY)
RESTORE DATABASE database_name WITH RECOVERY;
```

#### Complete General Syntax (PostgreSQL PITR)

```conf
# postgresql.conf
restore_command = 'cp /mnt/archive/%f %p'
recovery_target_time = '2026-10-07 14:04:00'
recovery_target_action = 'promote'
```

#### Component Breakdown (SQL Server)

| Component | Description |
|-----------|-------------|
| `STOPAT` | Target timestamp for log replay |
| `STOPATMARK` | Stop at a marked transaction (BEGIN LOG MARK) |
| `STOPBEFOREMARK` | Stop before a marked transaction |
| `NORECOVERY` | Required for intermediate restore steps |
| `RECOVERY` | Brings database online after final step |

#### Component Breakdown (PostgreSQL)

| Component | Description |
|-----------|-------------|
| `restore_command` | Shell command to retrieve WAL files (`%f` = filename, `%p` = path) |
| `recovery_target_time` | Timestamp to recover to |
| `recovery_target_action` | Action after recovery: `pause`, `promote`, `shutdown` |

#### Syntax Rules

- SQL Server: All log backups in the chain must be applied in sequence 
- SQL Server: `STOPAT` requires FULL or BULK_LOGGED recovery model 
- PostgreSQL: `recovery_target_time` is inclusive; set to one second before the disaster for safety
- PostgreSQL: A `recovery.signal` file must exist to trigger recovery mode
- MySQL: Binary logs must be enabled (`log-bin`) for PITR; use `mysqlbinlog` to extract and apply events 

#### Constraints and Limitations

- SQL Server: A broken log chain (missing log backup) prevents PITR beyond the break 
- PostgreSQL: `pg_dump` logical backups cannot be used for PITR; only physical base backups work 
- MySQL: Binary logs do not capture DDL in all configurations; statement-based logging may miss certain operations 
- PITR requires continuous log archiving; if archiving fails, the recoverable window shrinks
- Timeline issues (PostgreSQL): After PITR, a new timeline is created; subsequent WAL archives use the new timeline

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server PITR

**Setup**: Full backup, differential backup, and log backups exist for `TestFullBackup`.

```sql
-- Step 1: Restore full backup with NORECOVERY
RESTORE DATABASE TestFullBackup
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH NORECOVERY, REPLACE;
-- Expected: RESTORE DATABASE successfully processed 250 pages

-- Step 2: Restore differential with NORECOVERY
RESTORE DATABASE TestFullBackup
FROM DISK = 'C:\Backups\TestFullBackup_Diff2.bak'
WITH NORECOVERY;
-- Expected: RESTORE DATABASE WITH DIFFERENTIAL successfully processed 12 pages

-- Step 3: Restore log backup(s) with STOPAT
RESTORE LOG TestFullBackup
FROM DISK = 'C:\Backups\TestFullBackup_Log.trn'
WITH 
    NORECOVERY,
    STOPAT = '2026-10-07T14:04:00';
-- Expected: RESTORE LOG successfully processed 5 pages

-- Step 4: Bring database online
RESTORE DATABASE TestFullBackup WITH RECOVERY;
-- Expected: RESTORE DATABASE successfully processed 0 pages
```

```sql
-- Step 5: Verify recovery
SELECT * FROM Customers;
-- Expected: Shows all rows as they existed at 14:04, before the accidental DELETE
```

**Why This Output Occurs**: The restore sequence applies the full backup, then the differential (which contains all changes since the full), then the log backup replayed up to `STOPAT`. The database is brought online at the target time, excluding changes made after that point .

#### Example 2: PostgreSQL PITR

**Setup**: A base backup and archived WAL segments exist.

```bash
# Step 1: Stop PostgreSQL and clear the data directory
sudo systemctl stop postgresql
rm -rf /var/lib/postgresql/14/main/*

# Step 2: Restore the base backup
tar -xzf /backups/base/base.tar.gz -C /var/lib/postgresql/14/main/
tar -xzf /backups/base/pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal/

# Step 3: Configure recovery
cat > /var/lib/postgresql/14/main/postgresql.auto.conf << EOF
restore_command = 'cp /mnt/archive/%f %p'
recovery_target_time = '2026-10-07 14:04:00'
recovery_target_action = 'promote'
EOF

touch /var/lib/postgresql/14/main/recovery.signal
chown -R postgres:postgres /var/lib/postgresql/14/main/

# Step 4: Start PostgreSQL (it will enter recovery mode)
sudo systemctl start postgresql

# Step 5: Verify recovery
sudo -u postgres psql -c "SELECT * FROM critical_data;"
```

**Expected Output**:
```
 id |   value   
----+-----------
  1 | important
  2 | valuable
(2 rows)
```

**Why This Output Occurs**: PostgreSQL reads `recovery.signal`, enters recovery mode, and replays WAL segments from `/mnt/archive/` until `recovery_target_time`. The database is then promoted to normal operation, containing the data as it existed at 14:04—before the accidental DELETE at 14:05 .

### Real-World Cases

**Case 1: SQL Server Log Chain Recovery**
A DBA accidentally runs `DELETE FROM orders` without a WHERE clause at 2:15 PM. Using PITR with `STOPAT '2026-10-07T14:14:00'`, the database is recovered to one minute before the DELETE, losing zero orders .

**Case 2: PostgreSQL WAL Archiving**
A SaaS platform archives WAL continuously to S3-compatible storage. When a bad deployment corrupts data, PITR restores to the pre-deployment state in minutes .

**Case 3: MySQL Binary Log Recovery**
A developer drops the wrong table. The DBA restores the last full backup and applies binary logs up to (but not including) the DROP statement using `mysqlbinlog --stop-datetime` .

---

## Core Concept 3: Recovery Objectives (RPO & RTO)

### Definitions

**Core Definition**: RPO and RTO are business-defined targets that specify acceptable data loss and downtime after a disaster.

**Technical Definition**: Recovery Point Objective (RPO) is the maximum targeted period in which data might be lost; Recovery Time Objective (RTO) is the maximum acceptable time to restore service after an outage .

**Beginner-Friendly Explanation**: RPO answers "How much data can we afford to lose?" (e.g., 15 minutes' worth). RTO answers "How long can we be down?" (e.g., 1 hour). These targets drive every recovery decision.

### Purposes

- **To** quantify acceptable data loss and downtime in business terms
- **To** guide selection of backup frequency and replication strategy
- **To** establish measurable targets for recovery testing and validation
- **To** align technical recovery capabilities with business requirements

### Key Definitions and Alignment

| Objective | Definition | Example | Drives |
|-----------|-----------|---------|--------|
| **RPO** | Maximum acceptable data loss (measured backward from failure) | 15 minutes | Backup frequency, log archiving interval |
| **RTO** | Maximum acceptable downtime (measured forward from failure) | 1 hour | Restore speed, failover automation, warm standby |

#### RPO/RTO Alignment Rules

- RPO is measured backward from the failure point; RTO is measured forward 
- The overall RTO is determined by the slowest component to recover (e.g., if SQL Server recovers in 5 minutes but the application takes 20 minutes, RTO = 20 minutes) 
- Backup frequency must be at least as frequent as the RPO (e.g., RPO of 15 minutes requires log backups every 15 minutes or more frequently) 
- If RTO is 2 hours but a full backup takes 3 hours to copy, the RTO is unachievable and must be adjusted 

#### Constraints and Limitations

- RPO and RTO are business requirements, not technical capabilities; they must be achievable with available technology
- Zero RPO and zero RTO are often unrealistic; cost grows exponentially as targets approach zero 
- RTO must be defined at both component and application levels 
- RPO determines what backup features are required (e.g., log shipping for 15-minute RPO)

### Real-World Cases

**Case 1: Financial Trading Platform**
A trading system requires RPO = 0 (zero data loss) and RTO = 5 minutes. This drives investment in synchronous replication with automatic failover across two data centers.

**Case 2: E-Commerce Order Database**
An e-commerce platform tolerates RPO = 1 hour (one hour of orders may be lost) and RTO = 4 hours. This allows nightly full backups plus hourly transaction log backups, with restore from the most recent log backup.

**Case 3: Internal Reporting Database**
A reporting database tolerates RPO = 24 hours and RTO = 24 hours. A daily full backup suffices; no log archiving is needed.

---

## Core Concept 4: Disaster Recovery Planning

### Definitions

**Core Definition**: Disaster recovery planning is the documented set of procedures, runbooks, and infrastructure strategies for recovering from site-level failures.

**Technical Definition**: DR planning encompasses runbook automation, multi-site replication topologies (active-passive, active-active), failover orchestration, and split-brain prevention mechanisms including quorum-based fencing and epoch management .

**Beginner-Friendly Explanation**: A DR plan is like a fire drill for your database. It says exactly who does what, in what order, if the primary data center goes down—and how to prevent both sites from thinking they're in charge at the same time.

### Purposes

- **To** provide step-by-step recovery procedures during disasters
- **To** prevent split-brain scenarios where both sites accept writes
- **To** define multi-site strategies (active-passive, active-active) aligned with RTO/RPO
- **To** enable tested, repeatable failover with predictable outcomes

### Syntax Rules and Structure

#### Runbook Structure (General)

```markdown
# Disaster Recovery Runbook

## 1. RTO/RPO Targets
| Metric | Target |
|--------|--------|
| RTO    | 1 hour |
| RPO    | 5 minutes |

## 2. Failover Procedures

### Phase 1: Prevent Split-Brain
- FENCE old primary site (disable network, revoke write authority)
- Verify old primary is down or isolated

### Phase 2: Promote Standby
- Verify standby replication lag < RPO
- Promote standby to primary
- Update DNS/GSLB to point to new primary

### Phase 3: Verify
- Check application connectivity
- Validate data integrity
- Monitor for split-brain indicators

## 3. Failback Procedures
...
```

#### Split-Brain Prevention Mechanisms

| Mechanism | Description |
|-----------|-------------|
| **Quorum** | Majority of nodes must agree before promotion |
| **Fencing** | Forcibly isolate old primary (STONITH) |
| **Epoch** | Monotonically increasing generation number; stale epochs rejected |
| **GSLB** | Global load balancer redirects traffic only to healthy site |
| **Manual Verification** | Telephone/chat confirmation with remote site admins |

#### Syntax Rules

- Fencing must occur **before** promotion to prevent split-brain 
- GSLB TTL should be under 60 seconds for fast client redirection 
- Runbooks must include GO/NO-GO decision points before each critical step
- All steps must be executable by on-call staff without original authors

#### Constraints and Limitations

- Split-brain can occur if network partitioning makes the old primary unreachable from standby but still accessible to some clients 
- Routing changes alone are not fencing; write authority must be revoked at the storage/database level
- Unrehearsed failovers are not DR plans; regular testing is mandatory

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL Failover Runbook Steps

**Setup**: Primary and standby PostgreSQL servers with streaming replication.

```bash
# Step 1: Verify standby is current (GO/NO-GO)
psql -h standby -c "SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn();"
# If receive_lsn = replay_lsn, standby is current — PROCEED

# Step 2: FENCE the old primary (prevent split-brain)
# Option A: Shut down old primary
ssh old_primary "sudo systemctl stop postgresql"
# Option B: Revoke network access
ssh firewall "block all traffic from old_primary subnet"

# Step 3: Promote standby to primary
ssh standby "pg_ctl promote -D /var/lib/postgresql/14/main"
# Expected: server promoting

# Step 4: Verify promotion
psql -h standby -c "SELECT pg_is_in_recovery();"
# Expected: f (false — not in recovery, now primary)

# Step 5: Redirect traffic via GSLB/DNS
# Update DNS record to point to new primary IP
# Verify with: nslookup db.example.com
```

**Why This Output Occurs**: Fencing ensures the old primary cannot accept writes. Promotion transitions the standby to primary mode. `pg_is_in_recovery()` returning `f` confirms the standby is now the primary and accepting writes.

#### Example 2: Split-Brain Reconciliation

**Setup**: Split-brain has occurred; both sites have diverged data.

```bash
# Step 1: Detect divergence via WAL analysis
pg_waldump /var/lib/postgresql/14/main/pg_wal/000000010000000000000005 | grep "COMMIT"
# Compare LSN ranges on both sites

# Step 2: Elect authoritative WAL
# Use the site with the longest valid WAL sequence as authoritative

# Step 3: Rebuild the stale site
# On the stale site:
pg_basebackup -h new_primary -D /var/lib/postgresql/14/main -Ft -z -X stream -P
# This creates a fresh physical copy of the authoritative database

# Step 4: Verify consistency
pg_checksums --check -D /var/lib/postgresql/14/main
# Expected: Checksum operation completed
# All files have valid checksums
```

**Why This Output Occurs**: `pg_waldump` allows forensic analysis of which site has more valid data. `pg_basebackup` rebuilds the stale site from the authoritative primary, eliminating divergence. `pg_checksums --check` verifies physical integrity after rebuild.

### Real-World Cases

**Case 1: Multi-Region Active-Passive Failover**
A SaaS company runs an active-passive PostgreSQL setup across two regions. When the primary region fails, the DR runbook fences the old primary, promotes the standby, and redirects traffic via GSLB—all within the 1-hour RTO .

**Case 2: Split-Brain During Network Partition**
A network partition isolates the primary from the standby but not from some clients. The DR team detects split-brain indicators (diverging dedup state) and follows reconciliation procedures to elect an authoritative WAL and rebuild the stale site .

**Case 3: SQL Server Always On Failover**
A SQL Server Always On availability group automatically fails over when the primary replica becomes unavailable. Manual failover follows a runbook with quorum verification and listener redirection.

---

## Core Concept 5: Backup Validation and Verification

### Definitions

**Core Definition**: Backup validation confirms that backups are complete, uncorrupted, and restorable.

**Technical Definition**: Validation includes `RESTORE VERIFYONLY` checks (backup set completeness, checksum verification, media readability) and automated sandbox restore testing where backups are restored into isolated environments and data integrity is programmatically verified .

**Beginner-Friendly Explanation**: A backup that has never been tested is not a real backup. Validation is like test-driving a spare tire—you only know it works when you actually put it on and drive.

### Purposes

- **To** confirm backup completeness and media readability
- **To** detect corruption before a real recovery is needed
- **To** measure actual RTO and RPO through restore drills
- **To** provide audit-grade evidence of recoverability

### Syntax Rules and Structure

#### Complete General Syntax (RESTORE VERIFYONLY)

```sql
RESTORE VERIFYONLY
FROM DISK = 'path_to_backup.bak'
WITH 
    CHECKSUM | NO_CHECKSUM,
    FILE = backup_set_number,
    STATS = percentage
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `CHECKSUM` | Verifies backup checksums and page checksums  |
| `NO_CHECKSUM` | Skips checksum validation |
| `FILE` | Specifies which backup set on the media to verify |

#### Complete General Syntax (Sandbox Restore Testing - Script)

```bash
#!/bin/bash
# Automated sandbox restore test
SANDBOX_DIR="/tmp/restore-test-$(date +%s)"
BACKUP_FILE="/backups/latest.bak"

# Step 1: Provision sandbox
mkdir -p "$SANDBOX_DIR"

# Step 2: Restore backup into sandbox
pg_restore -d sandbox_db "$BACKUP_FILE" --clean --if-exists

# Step 3: Run verification queries
psql -d sandbox_db -c "SELECT COUNT(*) FROM critical_table;"
psql -d sandbox_db -c "CHECKSUM TABLE critical_table;"

# Step 4: Compare against expected values
# Step 5: Destroy sandbox
rm -rf "$SANDBOX_DIR"
```

#### Syntax Rules

- `RESTORE VERIFYONLY` does **not** restore the database; it only reads and validates the backup 
- `WITH CHECKSUM` verifies both backup checksums and page checksums if present 
- Sandbox tests should use isolated network environments to avoid impacting production 
- Validation scripts should capture logs, checksums, and KPIs for audit evidence 

#### Constraints and Limitations

- `RESTORE VERIFYONLY` does **not** guarantee that all data in the backup is correct, only that the backup is structurally valid 
- If the backup lacks checksums, `WITH CHECKSUM` fails rather than manufacturing one 
- Sandbox testing requires additional storage and compute resources
- Restore drills should be performed at the workload level (file, application, full-system) 

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server RESTORE VERIFYONLY

```sql
-- Step 1: Verify a backup with checksum validation
RESTORE VERIFYONLY
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH CHECKSUM, STATS = 10;
```

**Expected Output**:
```
10 percent processed.
20 percent processed.
...
100 percent processed.
The backup set on file 1 is valid.
```

**Why This Output Occurs**: `RESTORE VERIFYONLY` reads the backup media, validates the backup set header, checks the checksum (if present), and confirms that all volumes are readable. The "backup set is valid" message indicates structural integrity .

#### Example 2: Automated Sandbox Restore Test (PostgreSQL)

**Setup**: A CI/CD pipeline with Docker and a PostgreSQL backup file.

```bash
#!/bin/bash
set -e

# Step 1: Start an isolated PostgreSQL sandbox container
docker run -d --name pg-sandbox \
    -e POSTGRES_PASSWORD=test \
    -p 5433:5432 \
    postgres:14

# Step 2: Wait for PostgreSQL to be ready
sleep 5

# Step 3: Restore the backup into the sandbox
pg_restore -h localhost -p 5433 -U postgres \
    -d postgres --clean --if-exists \
    /backups/testdb_custom.dump

# Step 4: Run verification queries
psql -h localhost -p 5433 -U postgres -d testdb -c "
    SELECT COUNT(*) AS row_count FROM products;
    SELECT md5(string_agg(name, ',' ORDER BY id)) AS data_checksum FROM products;
"
```

**Expected Output**:
```
 row_count 
-----------
         3
(1 row)

          data_checksum           
----------------------------------
 d41d8cd98f00b204e9800998ecf8427e
(1 row)
```

```bash
# Step 5: Compare against expected values from production
# If counts and checksums match, restore is validated

# Step 6: Destroy the sandbox
docker stop pg-sandbox && docker rm pg-sandbox
```

**Why This Output Occurs**: The sandbox container provides an isolated environment where the backup can be restored without affecting production. The `COUNT(*)` and `md5` checksum queries verify that the restored data matches expected values. The sandbox is then destroyed, leaving no residual state .

### Real-World Cases

**Case 1: Daily CI/CD Restore Pipeline**
An engineering team runs a nightly CI/CD job that restores the latest backup into a sandbox VPC, checks record counts and CRUD operations, and emits audit-grade evidence .

**Case 2: Veeam SureBackup Validation**
An IT team uses Veeam SureBackup to automatically verify VM backups with checksums, file integrity, and network connectivity checks—all without impacting production .

**Case 3: Compliance Audit Evidence**
A financial institution must prove to regulators that backups are recoverable. Automated sandbox tests produce signed, timestamped evidence of successful restores with data integrity verification .

---

## Core Concept 6: Data Corruption Handling

### Definitions

**Core Definition**: Data corruption handling encompasses the detection, diagnosis, and repair of corrupted database pages or blocks.

**Technical Definition**: Data corruption handling uses page checksums, torn-page detection, `DBCC CHECKDB`, `pg_checksums`, and `ignore_checksum_failure` to detect and mitigate physical data corruption at the block/page level.

**Beginner-Friendly Explanation**: Data corruption is like a page in a book being torn or smudged. Corruption handling uses checksums (like a fingerprint for each page) to detect damage, then repairs it by restoring that page from a backup or marking it as unusable.

### Purposes

- **To** detect corruption early through checksums and consistency checks
- **To** repair damaged pages without full database restoration
- **To** prevent corruption from spreading to clean pages
- **To** identify the root cause (disk error, memory error, firmware bug)

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server PAGE_VERIFY)

```sql
ALTER DATABASE database_name
SET PAGE_VERIFY { CHECKSUM | TORN_PAGE_DETECTION | NONE };
```

#### Complete General Syntax (SQL Server DBCC CHECKDB)

```sql
DBCC CHECKDB ('database_name')
WITH 
    NO_INFOMSGS,
    REPAIR_ALLOW_DATA_LOSS | REPAIR_REBUILD
```

#### Complete General Syntax (SQL Server Backup with Checksum)

```sql
BACKUP DATABASE database_name
TO DISK = 'backup.bak'
WITH CHECKSUM;
```

#### Complete General Syntax (PostgreSQL pg_checksums)

```bash
pg_checksums --check -D /var/lib/postgresql/14/main
pg_checksums --enable -D /var/lib/postgresql/14/main
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `PAGE_VERIFY CHECKSUM` | Enables page-level checksums (default in SQL Server 2005+) |
| `PAGE_VERIFY TORN_PAGE_DETECTION` | Detects torn pages via bit-flip method (legacy) |
| `REPAIR_ALLOW_DATA_LOSS` | Repairs corruption with potential data loss |
| `REPAIR_REBUILD` | Repairs index corruption without data loss |
| `pg_checksums --check` | Verifies checksums on all data pages |
| `ignore_checksum_failure` | Bypasses checksum verification during recovery |

#### Syntax Rules

- `PAGE_VERIFY CHECKSUM` should be the default; it detects corruption when pages are read into memory 
- `DBCC CHECKDB` should be run regularly to detect corruption before it spreads 
- `REPAIR_ALLOW_DATA_LOSS` de-allocates corrupt pages, resulting in data loss 
- `pg_checksums --enable` must be run while the server is offline 
- `ignore_checksum_failure` is a recovery-only setting; do not leave it enabled 

#### Constraints and Limitations

- Page checksums detect corruption only when the page is read into memory; dormant pages may remain corrupt 
- `REPAIR_ALLOW_DATA_LOSS` is the minimum repair level for some errors and may de-allocate pages 
- `pg_checksums` is offline-only; it cannot run while PostgreSQL is accepting connections 
- `ignore_checksum_failure` allows recovery to proceed but may propagate corruption to indexes 

### Annotated Complete Step-by-Step Code Examples

#### Example 1: SQL Server Corruption Detection and Repair

**Setup**: A database with `PAGE_VERIFY CHECKSUM` enabled.

```sql
-- Step 1: Enable page checksums (if not already enabled)
ALTER DATABASE TestFullBackup
SET PAGE_VERIFY CHECKSUM;
-- Expected: Commands completed successfully.

-- Step 2: Create a backup with checksums for validation
BACKUP DATABASE TestFullBackup
TO DISK = 'C:\Backups\TestFullBackup_Checksum.bak'
WITH CHECKSUM, STATS = 25;
-- Expected: BACKUP DATABASE successfully processed 250 pages

-- Step 3: Simulate corruption (in a test environment only)
-- In production, corruption occurs due to disk/memory errors

-- Step 4: Detect corruption via DBCC CHECKDB
DBCC CHECKDB ('TestFullBackup') WITH NO_INFOMSGS;
-- Expected: DBCC results for 'TestFullBackup'.
--          There are 0 allocation errors and 0 consistency errors
```

```sql
-- Step 5: If corruption is found, restore the page from backup
-- First, identify the corrupt page from msdb..suspect_pages
SELECT * FROM msdb.dbo.suspect_pages WHERE database_id = DB_ID('TestFullBackup');

-- Step 6: Perform page-level restore
RESTORE DATABASE TestFullBackup
PAGE = '1:177'
FROM DISK = 'C:\Backups\TestFullBackup_Full.bak'
WITH RECOVERY;
```

**Why This Output Occurs**: `PAGE_VERIFY CHECKSUM` writes a checksum to each page header. When a page is read, SQL Server recomputes the checksum; a mismatch indicates corruption. `DBCC CHECKDB` scans all pages for consistency. If corruption is detected, page-level restore recovers just the damaged page from backup, avoiding a full restore .

#### Example 2: PostgreSQL Checksum Verification

```bash
# Step 1: Stop PostgreSQL
sudo systemctl stop postgresql

# Step 2: Enable checksums (only if not already enabled)
pg_checksums --enable -D /var/lib/postgresql/14/main
# Expected: Checksum operation completed
#           Files scanned: 1234, blocks scanned: 56789
#           Bad checksums: 0, Data blocks: 56789

# Step 3: Start PostgreSQL
sudo systemctl start postgresql
```

```bash
# Later, after a suspected corruption:
# Step 4: Check checksums
pg_checksums --check -D /var/lib/postgresql/14/main
# Expected: Checksum operation completed
#           Bad checksums: 0
```

```bash
# Step 5: If corruption is detected, recover with ignore_checksum_failure
# In postgresql.conf:
# ignore_checksum_failure = on  # TEMPORARY, for recovery only
```

**Why This Output Occurs**: `pg_checksums --enable` writes checksums to every data page while the server is offline. `--check` verifies all pages and reports any bad checksums. If corruption is found, `ignore_checksum_failure` allows recovery to proceed by ignoring checksum mismatches, though this may leave indexes inconsistent .

### Real-World Cases

**Case 1: Disk Firmware Bug Causing Checksum Errors**
A storage vendor's firmware bug causes intermittent checksum errors. `DBCC CHECKDB` identifies affected pages, and page-level restores repair them while the vendor applies a firmware patch .

**Case 2: PostgreSQL Checksum Mismatch After Hardware Failure**
A memory error corrupts a data page. `pg_checksums --check` detects the bad checksum, and the DBA restores the page from a base backup, avoiding full database restoration .

**Case 3: Ransomware Detection via Checksum Monitoring**
A ransomware attack modifies database files. Checksum monitoring detects the tampering before the attack is complete, triggering recovery from immutable backups.

---

## References

| Name | Link |
|------|------|
| Microsoft Learn - RESTORE (Transact-SQL) | https://learn.microsoft.com/en-us/sql/t-sql/statements/restore-statements-transact-sql |
| PostgreSQL Documentation - Continuous Archiving and PITR | https://www.postgresql.org/docs/14/continuous-archiving.html |
| Microsoft Learn - Describe RTO and RPO | https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/2-describe-recovery-time-objective-recovery-point-objective |
| Broadcom Tanzu Hub - Manual Failover and Split-Brain Prevention | https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-hub/10-4/tnz-hub/install-disaster-recovery-perform-failover.html |
| NinjaOne - How to Test Backups and Prove Restores | https://www.ninjaone.com/blog/how-to-test-backups-and-prove-restores/ |
| MySQL Enterprise Backup - Table-Level Recovery | https://docs.oracle.com/cd/E17952_01/mysql-enterprise-backup-9.7-en/restore.partial.html |
| Azure SQL Database - Restore from Backup | https://learn.microsoft.com/en-us/azure/azure-sql/database/recovery-using-backups |
| Microsoft Learn - RESTORE VERIFYONLY | https://learn.microsoft.com/en-us/sql/t-sql/statements/restore-statements-verifyonly-transact-sql |
| MySQL 8.0 Reference Manual - Point-in-Time Recovery Using Binary Log | https://dev.mysql.com/doc/refman/8.0/en/point-in-time-recovery-binlog.html |
| PostgreSQL Documentation - Data Checksums | https://www.postgresql.org/docs/14/checksums.html |
| Microsoft Learn - DBCC CHECKDB | https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql |
| Microsoft Learn - PAGE_VERIFY | https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options |