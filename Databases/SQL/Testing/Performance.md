# SQL Performance Testing: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL performance testing is the systematic process of measuring, validating, and predicting database behavior under controlled conditions—baseline, typical load, extreme stress, high concurrency, schema changes, and large data volumes—to ensure the database meets performance requirements before production deployment.

**Technical Definition**: SQL performance testing encompasses benchmarking (establishing baseline query execution metrics under static, controlled conditions), load testing (measuring throughput and resource utilization under expected peak-hour traffic), stress testing (pushing the database to extreme transaction thresholds to evaluate graceful degradation or failure), concurrency testing (simulating thousands of parallel transactions to detect lock contention and deadlocks), execution-plan comparison (comparing query plans before and after schema, index, or statistics changes), and data volume scale testing (seeding millions of records to validate query performance at production scale).

**Beginner-Friendly Explanation**: Performance testing is like test-driving a car before a long trip. Benchmarking is measuring how fast it goes on a flat road. Load testing is driving with a full load of passengers. Stress testing is pushing the engine to its limit. Concurrency testing is seeing how it handles heavy traffic. Execution-plan comparison is checking the engine after a tune-up. Data volume testing is loading the trunk with luggage and seeing if it still performs.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Controlled Environment** | Tests run on isolated, reproducible infrastructure |
| **Measurable Metrics** | Latency (P50/P95/P99), throughput (TPS/QPS), resource utilization (CPU, I/O, memory) |
| **Realistic Data** | Data volume and distribution mirror production |
| **Repeatable** | Tests can be re-run with the same conditions for comparison |
| **Gradual Scale** | Load increases stepwise to find breaking points |
| **Plan-Aware** | Execution plans are captured and compared before/after changes |

### Prerequisites

- **Isolated Test Environment**: Hardware matching production (or scaled proportionally)
- **Production-Like Data**: Same schema, indexes, and data distribution; volume may be scaled
- **Benchmarking Tool**: pgbench, sysbench, HammerDB, JMeter, or custom scripts
- **Monitoring Stack**: CPU, memory, disk I/O, and database-specific metrics
- **Baseline Metrics**: Known performance numbers before changes
- **Version Control**: Schema and query changes tracked for comparison

### Related Programming Areas

- **Database Administration**: Index tuning, statistics maintenance, parameter configuration
- **Application Development**: Query optimization, connection pooling, caching
- **Capacity Planning**: Forecasting resource needs from load test results
- **Site Reliability Engineering**: SLO/SLI definition, incident prevention
- **DevOps**: CI/CD integration, performance regression gates

### Core Concepts Overview

SQL performance testing comprises six complementary categories:

1. **Benchmarking**: Establishing baseline query execution metrics
2. **Load Testing**: Measuring throughput and resource spikes under typical traffic
3. **Stress Testing**: Pushing to extreme thresholds to evaluate degradation
4. **Concurrency Testing**: Simulating thousands of parallel transactions
5. **Execution-Plan Comparison**: Comparing plans before and after modifications
6. **Data Volume Scale Testing**: Seeding millions of records to test scaling

---

## Core Concept 1: Benchmarking

### Definitions

**Core Definition**: Benchmarking establishes baseline query execution metrics under static, controlled conditions, providing a reference point for detecting regressions and measuring improvements.

**Technical Definition**: Benchmarking measures specific queries or workloads (e.g., TPC-C, TPC-H, or custom OLTP/OLAP mixes) under controlled conditions: fixed data volume, fixed concurrency, fixed hardware, and no competing workloads. Metrics include latency percentiles (P50, P95, P99), throughput (transactions per second), rows scanned, buffer cache hit ratio, and I/O operations. The baseline is stored and compared against future runs.

**Beginner-Friendly Explanation**: Benchmarking is like timing a runner on a treadmill at a fixed speed. You record the time, and next month you run the same test to see if the runner improved or regressed. The controlled environment ensures the comparison is fair.

### Purposes

- **To** establish a reference point for detecting performance regressions
- **To** measure the impact of schema changes, index additions, or parameter tuning
- **To** compare hardware or configuration options objectively
- **To** validate that a system meets minimum performance requirements

### Syntax Rules and Structure

#### PostgreSQL: pgbench Benchmark

```bash
# Initialize the benchmark database (scale factor 100 = ~1.5 GB)
pgbench -i -s 100 mybenchmark

# Run a read-only benchmark for 60 seconds with 10 clients
pgbench -S -c 10 -j 2 -T 60 mybenchmark

# Run a read-write benchmark (TPC-B-like)
pgbench -c 10 -j 2 -T 60 mybenchmark

# Run a custom script
pgbench -c 10 -j 2 -T 60 -f my_script.sql mybenchmark
```

#### MySQL: sysbench Benchmark

```bash
# Prepare the benchmark (10 tables, 1M rows each)
sysbench oltp_read_write \
    --mysql-host=localhost \
    --mysql-user=root \
    --mysql-password=secret \
    --tables=10 \
    --table-size=1000000 \
    prepare

# Run the benchmark for 60 seconds with 10 threads
sysbench oltp_read_write \
    --mysql-host=localhost \
    --mysql-user=root \
    --mysql-password=secret \
    --tables=10 \
    --table-size=1000000 \
    --threads=10 \
    --time=60 \
    run

# Cleanup
sysbench oltp_read_write ... cleanup
```

#### SQL Server: Query Store Baseline

```sql
-- Capture baseline metrics for a query
SELECT
    q.query_id,
    qt.query_sql_text,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_ms,
    rs.avg_logical_io_reads,
    rs.count_executions
FROM sys.query_store_query q
JOIN sys.query_store_query_text qt ON q.query_text_id = qt.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
WHERE qt.query_sql_text LIKE '%orders%'
ORDER BY rs.avg_duration DESC;
```

#### Component Breakdown

| Metric | Description | Tool |
|--------|-------------|------|
| TPS (Transactions Per Second) | Throughput | pgbench, sysbench |
| Latency (ms) | Time per transaction | pgbench (`--latency-limit`) |
| P95/P99 Latency | Tail latency | Custom scripts, JMeter |
| Buffer Cache Hit Ratio | Memory efficiency | `pg_stat_database`, `InnoDB_buffer_pool_read_requests` |
| Rows Scanned | Query efficiency | EXPLAIN ANALYZE |
| I/O Operations | Disk pressure | `pg_stat_statements`, `iostat` |

#### Syntax Rules

- Benchmarks must run on an isolated environment with no competing workloads.
- Data volume and distribution must be identical between baseline and comparison runs.
- Warm up the cache before measuring (run the benchmark once, discard results, then measure).
- Run each benchmark multiple times and average the results.
- Store baseline metrics in version control for historical comparison.

#### Constraints and Limitations

- Benchmarks measure the database in isolation, not the full application stack.
- Synthetic benchmarks (pgbench, sysbench) may not reflect real query mixes.
- Hardware differences make benchmarks non-comparable across environments.
- Benchmark results are sensitive to configuration parameters (`shared_buffers`, `work_mem`).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PostgreSQL pgbench Baseline

```bash
# Step 1: Initialize the benchmark database with scale factor 50
pgbench -i -s 50 mybenchmark
# Expected output:
# creating tables...
# 5000000 tuples done.
# set primary key...
# vacuum... done.

# Step 2: Warm up the cache (run once, discard results)
pgbench -S -c 10 -j 2 -T 30 mybenchmark > /dev/null

# Step 3: Run the baseline benchmark
pgbench -S -c 10 -j 2 -T 60 --latency-limit=100 mybenchmark
```

**Expected Output**:
```
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
duration: 60 s
number of transactions actually processed: 1250000
latency average = 0.480 ms
latency stddev = 0.120 ms
tps = 20833.333333 (including connections establishing)
tps = 20835.123456 (excluding connections establishing)
```

```bash
# Step 4: Save the baseline
pgbench -S -c 10 -j 2 -T 60 mybenchmark > baseline_$(date +%Y%m%d).txt

# Step 5: After a schema change, re-run and compare
pgbench -S -c 10 -j 2 -T 60 mybenchmark > after_change_$(date +%Y%m%d).txt
diff baseline_*.txt after_change_*.txt
```

**Why This Output Occurs**: The `-S` flag runs a read-only (SELECT) benchmark. `-c 10` uses 10 concurrent clients, `-j 2` uses 2 threads, `-T 60` runs for 60 seconds. The output shows 1,250,000 transactions processed at 20,833 TPS with an average latency of 0.480ms. Comparing baseline and after-change outputs reveals regressions or improvements.

### Real-World Cases

**Case 1: Index Addition Benchmark**: A team adds an index on `orders(customer_id)`. They benchmark the `SELECT * FROM orders WHERE customer_id = ?` query before and after. TPS increases from 500 to 20,000.

**Case 2: Configuration Tuning**: A DBA increases `shared_buffers` from 1GB to 8GB. The benchmark shows a 40% TPS improvement and a 95% buffer cache hit ratio.

**Case 3: Hardware Comparison**: A company compares on-premises hardware against a cloud instance. pgbench results show the cloud instance provides 30% higher TPS at 40% lower cost.

---

## Core Concept 2: Load Testing

### Definitions

**Core Definition**: Load testing measures database throughput, latency, and resource utilization under expected peak-hour traffic frequencies.

**Technical Definition**: Load testing simulates the production workload at expected peak levels (e.g., 1,000 concurrent users, 5,000 TPS) and measures response time, throughput, error rates, and resource consumption (CPU, memory, disk I/O, network). The goal is to verify that the database meets performance SLOs under normal peak conditions, not to find the breaking point.

**Beginner-Friendly Explanation**: Load testing is like a restaurant preparing for a Friday night rush. You want to know: can the kitchen handle 100 orders per hour? How long do customers wait? Does the kitchen run out of ingredients? Load testing answers these questions before the rush happens.

### Purposes

- **To** verify that the database meets performance SLOs under peak traffic
- **To** identify resource bottlenecks (CPU, I/O, memory) before production
- **To** measure latency percentiles (P95, P99) under realistic load
- **To** validate that connection pooling and caching are correctly sized

### Syntax Rules and Structure

#### pgbench Load Test (Read-Write)

```bash
# Simulate 50 concurrent clients for 5 minutes
pgbench -c 50 -j 4 -T 300 -P 10 mybenchmark
# -P 10: print progress every 10 seconds
```

#### JMeter Load Test Plan (Conceptual)

```xml
<ThreadGroup>
    <num_threads>100</num_threads>       <!-- 100 concurrent users -->
    <ramp_time>60</ramp_time>             <!-- Ramp up over 60 seconds -->
    <duration>300</duration>              <!-- Run for 5 minutes -->
    <JDBCSampler>
        <query>SELECT * FROM orders WHERE customer_id = ?</query>
    </JDBCSampler>
</ThreadGroup>
```

#### MySQL: sysbench Load Test

```bash
sysbench oltp_read_write \
    --mysql-host=localhost \
    --mysql-user=root \
    --mysql-password=secret \
    --tables=10 \
    --table-size=1000000 \
    --threads=50 \
    --time=300 \
    --report-interval=10 \
    run
```

#### Component Breakdown

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| `-c` / `--threads` | Concurrent clients | Peak user count |
| `-j` | Worker threads | CPU cores |
| `-T` / `--time` | Duration (seconds) | 300–3600 |
| `-P` / `--report-interval` | Progress interval | 10–60 seconds |
| Ramp-up | Time to reach full load | 60–300 seconds |

#### Syntax Rules

- Load tests should use production-like data volume and distribution.
- Ramp up gradually to avoid artificial spikes at test start.
- Run for at least 5–10 minutes to reach steady state.
- Monitor database and system metrics during the test (CPU, I/O, memory, connections).
- Compare P95 and P99 latency, not just average.

#### Constraints and Limitations

- Load testing on non-production hardware may not reflect production performance.
- The test client itself may become the bottleneck (network, CPU).
- Connection pool sizing must match the load test's concurrency.
- Caching (application, database) may mask underlying performance issues.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: pgbench Load Test with Progress Reporting

```bash
# Step 1: Initialize with production-like data
pgbench -i -s 100 mybenchmark

# Step 2: Warm up
pgbench -c 50 -j 4 -T 60 mybenchmark > /dev/null

# Step 3: Run the load test with progress
pgbench -c 50 -j 4 -T 300 -P 30 mybenchmark
```

**Expected Output**:
```
starting vacuum...end.
progress: 30.0 s, 125000.0 tps, lat 0.395 ms stddev 0.120
progress: 60.0 s, 126000.0 tps, lat 0.392 ms stddev 0.118
progress: 90.0 s, 125500.0 tps, lat 0.394 ms stddev 0.119
...
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 100
query mode: simple
number of clients: 50
number of threads: 4
duration: 300 s
number of transactions actually processed: 37500000
latency average = 0.393 ms
latency stddev = 0.119 ms
tps = 125000.000000 (including connections establishing)
tps = 125123.456789 (excluding connections establishing)
```

```bash
# Step 4: Monitor system resources during the test
# In another terminal:
iostat -x 5
vmstat 5
```

**Expected Output (iostat excerpt)**:
```
Device            r/s     w/s     rkB/s     wkB/s   await  %util
sda             500.00  200.00   8000.00   4000.00   2.50  45.00
```

**Why This Output Occurs**: The `-P 30` flag prints progress every 30 seconds, showing TPS and latency. The final summary shows 37.5 million transactions at 125,000 TPS with 0.393ms average latency. `iostat` shows disk utilization at 45%, indicating the disk is not the bottleneck.

### Real-World Cases

**Case 1: E-Commerce Peak Hour**: A load test simulates 5,000 concurrent users browsing products and placing orders. Results show P95 latency of 50ms and 2,000 TPS, meeting the SLO of 100ms P95.

**Case 2: Connection Pool Sizing**: A load test with 100 concurrent clients shows connection wait times increasing. The DBA increases the pool size from 20 to 50, reducing wait times to zero.

**Case 3: Read Replica Offload**: A load test compares primary-only vs. primary+replica read distribution. Routing 70% of reads to replicas reduces primary CPU from 90% to 40%.

---

## Core Concept 3: Stress Testing

### Definitions

**Core Definition**: Stress testing pushes the database beyond normal operating limits to evaluate how it degrades—gracefully or catastrophically—and to find the breaking point.

**Technical Definition**: Stress testing increases load stepwise (e.g., 100, 200, 400, 800, 1600 concurrent clients) until the database fails, saturates, or exceeds acceptable error rates. The goal is to identify the maximum sustainable load, observe failure modes (timeouts, connection errors, deadlocks, OOM kills), and verify that the system recovers gracefully when load is reduced.

**Beginner-Friendly Explanation**: Stress testing is like overloading a bridge with trucks until it shows signs of strain. You want to know: at what weight does it start to bend? Does it recover when the trucks leave? Or does it collapse?

### Purposes

- **To** find the maximum sustainable load before degradation
- **To** observe failure modes (timeouts, errors, crashes)
- **To** verify that the system recovers after the stress is removed
- **To** validate autoscaling and failover behavior under extreme load

### Syntax Rules and Structure

#### pgbench Stepwise Stress Test

```bash
# Step 1: Start at low concurrency and increase
for clients in 10 25 50 100 200 400 800; do
    echo "=== Testing with $clients clients ==="
    pgbench -c $clients -j 4 -T 60 mybenchmark 2>&1 | \
        grep -E "tps|latency average"
    sleep 10
done
```

#### MySQL: sysbench Stress Test

```bash
for threads in 10 25 50 100 200 400 800; do
    echo "=== Testing with $threads threads ==="
    sysbench oltp_read_write \
        --mysql-host=localhost \
        --mysql-user=root \
        --mysql-password=secret \
        --tables=10 \
        --table-size=1000000 \
        --threads=$threads \
        --time=60 \
        run 2>&1 | grep -E "transactions|queries|latency"
    sleep 10
done
```

#### Component Breakdown

| Concurrency | Expected Behavior |
|-------------|-------------------|
| Low (10) | Low latency, low throughput |
| Medium (50) | Increasing throughput, stable latency |
| High (200) | Throughput plateaus, latency increases |
| Very High (800) | Throughput drops, errors appear |
| Beyond Limit | Connection errors, timeouts, OOM |

#### Syntax Rules

- Increase concurrency in steps (e.g., doubling) to find the knee in the curve.
- Monitor error rates at each step; errors indicate the breaking point.
- Allow the system to cool down between steps (sleep 10–30 seconds).
- Capture system metrics (CPU, memory, I/O) at each step.
- After the test, verify that the database recovers (connections drop, latency returns to baseline).

#### Constraints and Limitations

- Stress testing can cause data corruption if the database crashes mid-write; use a disposable test database.
- Hardware may be permanently damaged by extreme stress (thermal, disk wear).
- Cloud instances may be throttled or terminated by the provider under sustained stress.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Stepwise Stress Test with pgbench

```bash
# Step 1: Run stepwise stress test
for clients in 10 25 50 100 200 400; do
    echo "=== $clients clients ==="
    pgbench -c $clients -j 4 -T 60 mybenchmark 2>&1 | \
        grep -E "tps|latency average|number of failed"
    sleep 15
done
```

**Expected Output**:
```
=== 10 clients ===
number of transactions actually processed: 1250000
latency average = 0.480 ms
tps = 20833.333333 (including connections establishing)

=== 25 clients ===
number of transactions actually processed: 2800000
latency average = 0.535 ms
tps = 46666.666667 (including connections establishing)

=== 50 clients ===
number of transactions actually processed: 3750000
latency average = 0.800 ms
tps = 62500.000000 (including connections establishing)

=== 100 clients ===
number of transactions actually processed: 4000000
latency average = 1.500 ms
tps = 66666.666667 (including connections establishing)

=== 200 clients ===
number of transactions actually processed: 3500000
latency average = 3.428 ms
tps = 58333.333333 (including connections establishing)
number of failed transactions: 500

=== 400 clients ===
number of transactions actually processed: 2000000
latency average = 12.000 ms
tps = 33333.333333 (including connections establishing)
number of failed transactions: 5000
```

**Why This Output Occurs**: Throughput increases from 10 to 100 clients (20,833 → 66,666 TPS), then drops at 200 clients (58,333 TPS) and 400 clients (33,333 TPS). Latency increases sharply at 200+ clients (3.4ms → 12ms). Failed transactions appear at 200 clients. The breaking point is around 100–200 clients.

#### Example 2: Observing Recovery After Stress

```bash
# Step 1: Stress the database
pgbench -c 400 -j 8 -T 120 mybenchmark

# Step 2: Immediately run a low-concurrency benchmark to check recovery
sleep 30
pgbench -c 10 -j 2 -T 60 mybenchmark
```

**Expected Output (after recovery)**:
```
number of transactions actually processed: 1200000
latency average = 0.500 ms
tps = 20000.000000 (including connections establishing)
```

**Why This Output Occurs**: After the stress test ends, connections close, locks release, and the database returns to baseline performance (20,000 TPS, 0.500ms latency). If performance does not recover, the stress test revealed a resource leak or unreleased lock.

### Real-World Cases

**Case 1: Black Friday Preparation**: A stress test finds that the database handles up to 3,000 TPS before latency exceeds 100ms. The team provisions additional read replicas to handle the expected 5,000 TPS peak.

**Case 2: Connection Limit Discovery**: A stress test with 500 clients exhausts `max_connections` (100), causing connection errors. The fix increases `max_connections` and adds PgBouncer.

**Case 3: Memory Exhaustion**: A stress test with large sorts causes the database to be OOM-killed. The fix increases `work_mem` limits or adds memory.

---

## Core Concept 4: Concurrency Testing

### Definitions

**Core Definition**: Concurrency testing simulates many parallel user transactions to detect lock contention, deadlocks, and serialization failures that only appear under concurrent access.

**Technical Definition**: Concurrency testing runs multiple transactions simultaneously, accessing overlapping rows and tables, to measure lock wait times, deadlock frequency, and throughput degradation. It validates that isolation levels behave correctly, that lock ordering is consistent, and that the application handles deadlock retries. Metrics include lock wait time, deadlock count, and transaction abort rate.

**Beginner-Friendly Explanation**: Concurrency testing is like seeing how a bank handles 100 people trying to withdraw from the same account at the same time. Does the system serialize them correctly? Do some transactions fail? How long do people wait?

### Purposes

- **To** detect lock contention under high concurrency
- **To** measure deadlock frequency and identify the root cause
- **To** verify that isolation levels prevent anomalies
- **To** validate that the application retries deadlock victims correctly

### Syntax Rules and Structure

#### PostgreSQL: Concurrent Updates with pgbench Custom Script

```sql
-- custom_script.sql: update the same account repeatedly
\set account_id random(1, 100)
BEGIN;
UPDATE accounts SET balance = balance - 10 WHERE account_id = :account_id;
UPDATE accounts SET balance = balance + 10 WHERE account_id = :account_id;
COMMIT;
```

```bash
pgbench -c 100 -j 4 -T 60 -f custom_script.sql mybenchmark
```

#### PostgreSQL: Monitoring Lock Contention During the Test

```sql
-- In another session, monitor lock waits
SELECT
    blocked.pid AS blocked_pid,
    blocking.pid AS blocking_pid,
    blocked.wait_event_type,
    blocked.wait_event,
    now() - blocked.query_start AS wait_duration
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.wait_event_type = 'Lock';
```

#### MySQL: Concurrency Test with sysbench

```bash
sysbench oltp_read_write \
    --mysql-host=localhost \
    --mysql-user=root \
    --mysql-password=secret \
    --tables=10 \
    --table-size=1000000 \
    --threads=200 \
    --time=120 \
    --rand-type=uniform \
    run
```

#### Component Breakdown

| Metric | Description | Tool |
|--------|-------------|------|
| Lock wait time | Time spent waiting for locks | `pg_stat_activity`, `sys.innodb_lock_waits` |
| Deadlock count | Number of deadlocks | `pg_stat_database.deadlocks`, `Innodb_deadlocks` |
| Transaction abort rate | Failed transactions | pgbench "number of failed" |
| Serialization failures | SQLSTATE 40001 count | Application logs |
| Throughput | TPS under concurrency | pgbench, sysbench |

#### Syntax Rules

- Concurrency tests should use random access patterns to maximize lock conflicts.
- Monitor lock waits and deadlocks during the test, not just at the end.
- Test with the production isolation level (READ COMMITTED, REPEATABLE READ, SERIALIZABLE).
- Verify that the application retries deadlock victims; measure retry success rate.

#### Constraints and Limitations

- High concurrency tests may cause data inconsistencies if the application does not handle retries.
- Lock contention is data-dependent; skewed data causes more contention.
- Deadlocks are unavoidable; the goal is to minimize frequency and handle them gracefully.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Concurrency Test with Lock Monitoring

```bash
# Step 1: Create a custom script that updates hot rows
cat > concurrency_test.sql << 'EOF'
\set account_id random(1, 10)
BEGIN;
UPDATE accounts SET balance = balance - 10 WHERE account_id = :account_id;
SELECT pg_sleep(0.001);
UPDATE accounts SET balance = balance + 10 WHERE account_id = :account_id;
COMMIT;
EOF

# Step 2: Run with 100 concurrent clients
pgbench -c 100 -j 4 -T 60 -f concurrency_test.sql mybenchmark
```

**Expected Output**:
```
transaction type: concurrency_test.sql
scaling factor: 100
query mode: simple
number of clients: 100
number of threads: 4
duration: 60 s
number of transactions actually processed: 450000
latency average = 13.333 ms
latency stddev = 5.678 ms
tps = 7500.000000 (including connections establishing)
number of failed transactions: 1200
```

```sql
-- Step 3: Check deadlock count after the test
SELECT deadlocks FROM pg_stat_database WHERE datname = 'mybenchmark';
-- Expected: 1200
```

**Why This Output Occurs**: The custom script updates the same 10 accounts repeatedly, causing lock contention. 100 concurrent clients compete for locks, producing 1,200 failed transactions (deadlock victims). Average latency is 13.3ms—much higher than the 0.4ms baseline—due to lock waits. The deadlock count matches the failed transaction count.

#### Example 2: Verifying Deadlock Retry Logic

```python
import psycopg2
from psycopg2 import errors

def transfer_with_retry(from_acct, to_acct, amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            conn = psycopg2.connect("dbname=mybenchmark")
            cur = conn.cursor()
            cur.execute("UPDATE accounts SET balance = balance - %s WHERE account_id = %s",
                        (amount, from_acct))
            cur.execute("UPDATE accounts SET balance = balance + %s WHERE account_id = %s",
                        (amount, to_acct))
            conn.commit()
            return True
        except errors.DeadlockDetected:
            conn.rollback()
            if attempt == max_retries - 1:
                raise
            time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
        finally:
            conn.close()
    return False
```

**Why This Output Occurs**: The retry function catches `DeadlockDetected` (SQLSTATE 40P01), rolls back, and retries with exponential backoff. After the concurrency test, the retry success rate should be high, indicating that the application handles deadlocks gracefully.

### Real-World Cases

**Case 1: Inventory Contention**: During a flash sale, 1,000 users try to buy the same product simultaneously. Concurrency testing reveals lock waits up to 5 seconds. The fix uses `SELECT ... FOR UPDATE NOWAIT` or an application-level queue.

**Case 2: Deadlock in Batch Processing**: Two batch jobs update the same tables in different orders, causing deadlocks. The fix enforces consistent lock ordering.

**Case 3: Serialization Failures**: A SERIALIZABLE transaction test shows a 5% serialization failure rate. The application retries, and the effective failure rate drops to 0.1%.

---

## Core Concept 5: Execution-Plan Comparison

### Definitions

**Core Definition**: Execution-plan comparison is the process of capturing and comparing query execution plans before and after schema, index, statistics, or configuration changes to verify that performance improvements (or regressions) are real.

**Technical Definition**: Execution-plan comparison uses `EXPLAIN` (estimated) and `EXPLAIN ANALYZE` (actual) to capture plan structure, access methods (Seq Scan, Index Scan, Hash Join), join order, estimated vs. actual rows, and timing. Plans are compared textually or via JSON diff to identify changes in access paths, join strategies, and costs. The goal is to confirm that a change improved the plan and to detect unintended regressions.

**Beginner-Friendly Explanation**: Execution-plan comparison is like comparing a GPS route before and after a road change. The old route went through downtown (Seq Scan); the new route uses the highway (Index Scan). The comparison shows whether the change actually improved the route.

### Purposes

- **To** verify that an index addition changes a Seq Scan to an Index Scan
- **To** detect plan regressions after statistics updates or schema changes
- **To** understand why a query became slower or faster
- **To** validate that the optimizer chooses the intended plan

### Syntax Rules and Structure

#### PostgreSQL: Capture Plan Before Change

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (Before Index)**:
```json
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Relation Name": "orders",
      "Actual Rows": 100,
      "Actual Total Time": 2500.000,
      "Filter": "(customer_id = 12345)",
      "Rows Removed by Filter": 999900
    }
  }
]
```

#### PostgreSQL: Capture Plan After Change

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM orders WHERE customer_id = 12345;
```

**Expected Output (After Index)**:
```json
[
  {
    "Plan": {
      "Node Type": "Index Scan",
      "Index Name": "idx_orders_customer_id",
      "Actual Rows": 100,
      "Actual Total Time": 0.550,
      "Index Cond": "(customer_id = 12345)"
    }
  }
]
```

#### Component Breakdown

| Plan Element | Before | After |
|--------------|--------|-------|
| Node Type | Seq Scan | Index Scan |
| Index Name | — | idx_orders_customer_id |
| Actual Total Time | 2500.000 ms | 0.550 ms |
| Rows Removed by Filter | 999900 | — |

#### Syntax Rules

- Use `FORMAT JSON` for programmatic comparison and diffing.
- Capture plans before and after the change, with the same query and data.
- Compare `Actual Total Time`, `Actual Rows`, `Node Type`, and `Index Name`.
- Use `EXPLAIN (ANALYZE, BUFFERS)` to see memory vs. disk reads.
- Store plans in version control for historical comparison.

#### Constraints and Limitations

- Plans can change due to statistics updates, not just the intended change.
- `EXPLAIN ANALYZE` executes the query; use `EXPLAIN` (without ANALYZE) for read-only comparison.
- Plan comparison is sensitive to data distribution; test with production-like data.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Plans Before and After Index Addition

```sql
-- Step 1: Capture the "before" plan
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345;
-- Save output to before_plan.txt
```

**Before Plan**:
```
 Seq Scan on orders  (actual time=0.500..2500.000 rows=100 loops=1)
   Filter: (customer_id = 12345)
   Rows Removed by Filter: 999900
   Buffers: shared hit=10000 read=5000
 Execution Time: 2500.500 ms
```

```sql
-- Step 2: Add the index
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Step 3: Capture the "after" plan
EXPLAIN (ANALYZE, BUFFERS, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345;
-- Save output to after_plan.txt
```

**After Plan**:
```
 Index Scan using idx_orders_customer_id on orders
   (actual time=0.050..0.500 rows=100 loops=1)
   Index Cond: (customer_id = 12345)
   Buffers: shared hit=10 read=2
 Execution Time: 0.550 ms
```

```bash
# Step 4: Compare plans
diff before_plan.txt after_plan.txt
```

**Expected Diff**:
```
<  Seq Scan on orders  (actual time=0.500..2500.000 rows=100 loops=1)
<    Filter: (customer_id = 12345)
<    Rows Removed by Filter: 999900
<    Buffers: shared hit=10000 read=5000
<  Execution Time: 2500.500 ms
---
>  Index Scan using idx_orders_customer_id on orders
>    (actual time=0.050..0.500 rows=100 loops=1)
>    Index Cond: (customer_id = 12345)
>    Buffers: shared hit=10 read=2
>  Execution Time: 0.550 ms
```

**Why This Output Occurs**: The diff shows that the `Seq Scan` (2,500ms, 999,900 rows removed) was replaced by an `Index Scan` (0.550ms, 10 buffer hits). The comparison confirms the index improved performance by 4,500x.

#### Example 2: Detecting a Plan Regression

```sql
-- Step 1: Capture plan before statistics update
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345 AND order_date > '2025-06-01';
-- Before: Index Scan using idx_orders_customer_date

-- Step 2: Update statistics
ANALYZE orders;

-- Step 3: Capture plan after statistics update
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM orders WHERE customer_id = 12345 AND order_date > '2025-06-01';
-- After: Seq Scan (regression!)
```

**Expected Diff**:
```
<  Index Scan using idx_orders_customer_date on orders
<    Index Cond: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
<  Execution Time: 0.550 ms
---
>  Seq Scan on orders
>    Filter: ((customer_id = 12345) AND (order_date > '2025-06-01'::date))
>    Rows Removed by Filter: 999900
>  Execution Time: 2500.500 ms
```

**Why This Output Occurs**: The statistics update changed the optimizer's cardinality estimate, causing it to choose a Seq Scan instead of the Index Scan. This is a plan regression—the change made performance worse. The fix might be to increase the statistics target, create extended statistics, or use a query hint.

### Real-World Cases

**Case 1: Index Addition Validation**: A DBA adds an index and compares plans before and after. The plan changes from Seq Scan to Index Scan, confirming the index is used.

**Case 2: PostgreSQL Upgrade Regression**: After upgrading PostgreSQL, a query plan changes from Hash Join to Nested Loop, causing a 10x slowdown. Plan comparison identifies the regression, and the fix is to adjust `enable_nestloop` or update statistics.

**Case 3: Schema Change Impact**: Adding a column changes the plan for a critical query. Plan comparison reveals the regression before deployment.

---

## Core Concept 6: Data Volume Scale Testing

### Definitions

**Core Definition**: Data volume scale testing populates mock tables with millions of records to validate that queries perform acceptably at production scale before code is deployed.

**Technical Definition**: Data volume scale testing seeds tables with realistic row counts, data distributions, and cardinalities, then benchmarks queries against that data. It detects performance issues that only appear at scale: index degradation, statistics staleness, memory pressure, and plan changes. Seeding methods include `generate_series` (PostgreSQL), recursive CTEs, stored procedures, and external data generators (pgbench, sysbench, custom scripts).

**Beginner-Friendly Explanation**: Data volume testing is like testing a bridge with the actual weight of cars it will carry, not just a few toy cars. A query that's fast on 1,000 rows may be slow on 10 million rows. You need to test at production scale before going live.

### Purposes

- **To** validate query performance at production-scale data volume
- **To** detect plan changes that occur only at large row counts
- **To** verify that indexes remain effective as data grows
- **To** measure the impact of data distribution (skew, cardinality) on query performance

### Syntax Rules and Structure

#### PostgreSQL: Seed with generate_series

```sql
-- Seed 10 million orders
INSERT INTO orders (customer_id, order_date, total)
SELECT
    (random() * 100000)::int,
    '2020-01-01'::date + (random() * 2000)::int,
    random() * 10000
FROM generate_series(1, 10000000);

-- Update statistics after seeding
ANALYZE orders;
```

#### MySQL: Seed with Recursive CTE

```sql
-- Seed 1 million products
INSERT INTO products (product_name, price, category_id)
WITH RECURSIVE seq AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM seq WHERE n < 1000000
)
SELECT
    CONCAT('Product ', n),
    ROUND(RAND() * 1000, 2),
    (n % 100) + 1
FROM seq;
```

#### SQL Server: Seed with Tally Table

```sql
-- Seed 5 million rows
WITH tally AS (
    SELECT TOP (5000000)
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
    FROM sys.all_columns a
    CROSS JOIN sys.all_columns b
)
INSERT INTO orders (customer_id, order_date, total)
SELECT
    ABS(CHECKSUM(NEWID())) % 100000,
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 2000, '2020-01-01'),
    ABS(CHECKSUM(NEWID())) % 10000
FROM tally;
```

#### Component Breakdown

| Method | Database | Rows Generated |
|--------|----------|----------------|
| `generate_series` | PostgreSQL | Any number |
| Recursive CTE | MySQL, PostgreSQL | Up to recursion limit |
| Tally table | SQL Server | Millions |
| pgbench `-i -s N` | PostgreSQL | N × 100,000 |
| sysbench `--table-size` | MySQL | Configurable |

#### Syntax Rules

- Seed data must match production cardinality and distribution (e.g., skewed customer IDs).
- Update statistics (`ANALYZE`) after seeding; the optimizer needs accurate stats.
- Test with multiple data volumes (e.g., 1M, 10M, 100M) to observe scaling behavior.
- Use `EXPLAIN ANALYZE` to measure performance at each volume.
- Clean up seed data after testing to reclaim storage.

#### Constraints and Limitations

- Seeding millions of rows takes time and storage.
- Synthetic data may not match production skew; use realistic distributions.
- Statistics need to be updated after seeding; otherwise, plans are based on stale estimates.
- Very large data volumes may require partitioning or sharding.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Seeding and Testing at Multiple Volumes

```sql
-- Step 1: Create the table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total NUMERIC(10,2)
);

-- Step 2: Seed 1 million rows
INSERT INTO orders (customer_id, order_date, total)
SELECT
    (random() * 10000)::int,
    '2020-01-01'::date + (random() * 2000)::int,
    random() * 10000
FROM generate_series(1, 1000000);
ANALYZE orders;

-- Step 3: Test query at 1M rows
EXPLAIN (ANALYZE, COSTS OFF)
SELECT COUNT(*) FROM orders WHERE customer_id = 5000;
-- Expected: ~0.5ms

-- Step 4: Seed additional 9 million rows (total 10M)
INSERT INTO orders (customer_id, order_date, total)
SELECT
    (random() * 10000)::int,
    '2020-01-01'::date + (random() * 2000)::int,
    random() * 10000
FROM generate_series(1, 9000000);
ANALYZE orders;

-- Step 5: Test same query at 10M rows
EXPLAIN (ANALYZE, COSTS OFF)
SELECT COUNT(*) FROM orders WHERE customer_id = 5000;
-- Expected: ~5ms (10x slower due to 10x data)
```

**Expected Output (1M rows)**:
```
 Aggregate  (actual time=0.500..0.501 rows=1 loops=1)
   ->  Seq Scan on orders  (actual time=0.200..0.450 rows=100 loops=1)
         Filter: (customer_id = 5000)
         Rows Removed by Filter: 999900
 Execution Time: 0.550 ms
```

**Expected Output (10M rows)**:
```
 Aggregate  (actual time=5.000..5.001 rows=1 loops=1)
   ->  Seq Scan on orders  (actual time=0.200..4.500 rows=100 loops=1)
         Filter: (customer_id = 5000)
         Rows Removed by Filter: 9999900
 Execution Time: 5.050 ms
```

```sql
-- Step 6: Add index and re-test at 10M
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
ANALYZE orders;

EXPLAIN (ANALYZE, COSTS OFF)
SELECT COUNT(*) FROM orders WHERE customer_id = 5000;
-- Expected: ~0.5ms (index used)
```

**Expected Output (After Index)**:
```
 Aggregate  (actual time=0.500..0.501 rows=1 loops=1)
   ->  Index Scan using idx_orders_customer_id on orders
         (actual time=0.050..0.450 rows=100 loops=1)
         Index Cond: (customer_id = 5000)
 Execution Time: 0.550 ms
```

**Why This Output Occurs**: At 1M rows, the Seq Scan takes 0.55ms. At 10M rows, it takes 5.05ms (10x slower, as expected). Adding the index reduces it to 0.55ms, matching the 1M performance. This demonstrates that the index scales logarithmically, while the Seq Scan scales linearly.

#### Example 2: Testing with Skewed Data Distribution

```sql
-- Step 1: Seed with skewed customer_id (90% of orders from customer 1)
INSERT INTO orders (customer_id, order_date, total)
SELECT
    CASE WHEN random() < 0.9 THEN 1 ELSE (random() * 10000)::int + 2 END,
    '2020-01-01'::date + (random() * 2000)::int,
    random() * 10000
FROM generate_series(1, 10000000);
ANALYZE orders;

-- Step 2: Test query for the hot customer
EXPLAIN (ANALYZE, COSTS OFF)
SELECT COUNT(*) FROM orders WHERE customer_id = 1;
-- Expected: Index scan returns 9M rows (slow due to volume)

-- Step 3: Test query for a cold customer
EXPLAIN (ANALYZE, COSTS OFF)
SELECT COUNT(*) FROM orders WHERE customer_id = 5000;
-- Expected: Index scan returns few rows (fast)
```

**Why This Output Occurs**: Skewed data means the hot customer (ID 1) has 9 million rows, so even an index scan must read many rows. The cold customer (ID 5000) has few rows, so the index scan is fast. Skewed data testing reveals that the optimizer may choose different plans for different values of the same column.

### Real-World Cases

**Case 1: Pre-Deployment Volume Test**: A team tests a new reporting query against 50 million rows before deploying. The query takes 30 seconds—unacceptable for the SLO. The fix adds a covering index and reduces the time to 2 seconds.

**Case 2: Statistics Staleness at Scale**: After seeding 100 million rows, the optimizer's statistics are stale, causing a Seq Scan. Running `ANALYZE` fixes the estimate and the plan.

**Case 3: Partitioning Decision**: Data volume testing at 500 million rows shows that a single table is too large for maintenance. The team partitions the table by date, improving query performance and maintenance.

---

## References

| Name | Link |
|------|------|
| PostgreSQL Documentation — pgbench | https://www.postgresql.org/docs/current/pgbench.html |
| PostgreSQL Documentation — EXPLAIN | https://www.postgresql.org/docs/current/using-explain.html |
| PostgreSQL Documentation — pg_stat_statements | https://www.postgresql.org/docs/current/pgstatstatements.html |
| MySQL 8.0 Reference Manual — sysbench | https://dev.mysql.com/doc/refman/8.0/en/ |
| MySQL 8.0 Reference Manual — Performance Schema | https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html |
| Microsoft Learn — Query Store | https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store |
| Microsoft Learn — Execution Plans | https://learn.microsoft.com/en-us/sql/relational-databases/performance/execution-plans |
| sysbench Documentation | https://github.com/akopytov/sysbench |
| HammerDB Documentation | https://www.hammerdb.com/documentation/ |
| Apache JMeter Documentation | https://jmeter.apache.org/usermanual/ |
| TPC Benchmarks | https://www.tpc.org/tpc_documents_current_versions/current_specifications5.asp |
| Use The Index, Luke — Performance Testing | https://use-the-index-luke.com/ |
| Percona — Benchmarking MySQL | https://www.percona.com/blog/ |
| PostgreSQL Wiki — Performance Testing | https://wiki.postgresql.org/wiki/Performance_Testing |
| Redgate — SQL Server Stress Testing | https://www.red-gate.com/simple-talk/databases/sql-server/ |