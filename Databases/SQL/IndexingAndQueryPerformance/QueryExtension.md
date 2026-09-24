# SQL Query Execution & Optimization Engine: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The SQL query execution and optimization engine is the database subsystem that transforms a declarative SQL statement into an efficient execution plan, executes that plan, and returns the result set.

**Technical Definition:** The query processing engine consists of several cooperating components: a parser that validates syntax and builds an abstract syntax tree (AST); a semantic analyzer that resolves object references and checks permissions; an optimizer that enumerates alternative execution plans and selects the one with the lowest estimated cost; and an execution engine that runs the chosen plan using row-source iterators. The optimizer's cost model incorporates disk I/O, CPU usage, and memory consumption, with cardinality estimation as a critical input.

**Beginner-Friendly Explanation:** When you type a SQL query, the database doesn't just run it blindly. It first checks that your query makes sense (parsing and semantic analysis). Then it brainstorms multiple ways to get your answer (query planning) and picks the one it thinks will be fastest (optimization). Finally, it executes that plan and hands you the results. Think of it like a GPS: you give it a destination (your query), and it calculates several routes (plans), picks the fastest one based on current conditions (cost estimation), and guides you along that route (execution).

### Key Characteristics

- **Cost-based:** The optimizer assigns numeric costs to plans and chooses the lowest-cost option.
- **Statistics-driven:** Cardinality estimation uses table and index statistics to predict row counts at each step.
- **Rule-based transformations:** The query transformer rewrites queries into semantically equivalent forms that may be cheaper to execute (e.g., converting OR to UNION ALL, unnesting subqueries).
- **Adaptive:** Some modern optimizers can change plans mid-execution based on runtime statistics.

### Prerequisites

- **SQL proficiency:** Understanding of SELECT, JOIN, WHERE, and subqueries.
- **Basic database concepts:** Tables, indexes, statistics, and storage structures.
- **Familiarity with execution plans:** Ability to read EXPLAIN output.

### Related Programming Areas

- **Database Administration:** Performance tuning and plan analysis.
- **Application Development:** Writing queries that the optimizer can execute efficiently.
- **Data Engineering:** Designing schemas and indexes that support optimal plans.

### Core Concepts / Features

1. Query Parsing & Semantic Analysis
2. Query Planning & Optimization
3. Cost Estimation
4. Cardinality Estimation
5. Join Algorithms
6. Parallel Query Execution

---

## 1. Query Parsing & Semantic Analysis

### Definitions

**Core Definition:** Query parsing and semantic analysis is the first phase of query processing, where the SQL text is validated for correct syntax, and all referenced objects and permissions are verified.

**Technical Definition:** The parser constructs an abstract syntax tree (AST) from the SQL text, checking syntax according to the grammar of the SQL dialect. The semantic analyzer then resolves all identifiers (table names, column names) against the database catalog, infers data types, checks user privileges, and produces a validated, typed representation of the query. SAP HANA documentation states that "if the SQL user does not have the correct permission to access the object, the object will not be found".

**Beginner-Friendly Explanation:** Parsing is like a grammar check for your SQL. The database reads your query and asks: "Is this valid SQL? Do the tables and columns you mentioned actually exist? Do you have permission to see them?" If any of these checks fail, the query stops immediately with an error.

### Purposes

- To validate that the SQL statement conforms to the dialect's grammar.
- To resolve all object references against the database schema.
- To verify that the user has the necessary privileges on referenced objects.
- To produce a semantically correct internal representation for optimization.

### Syntax Rules and Structure

This phase is internal to the database; users do not write syntax for it. However, errors surface as SQL error codes:

| Error Type | Example | Cause |
|------------|---------|-------|
| Syntax Error | `SELECT * FORM users` | Misspelled keyword (`FORM` instead of `FROM`) |
| Object Not Found | `SELECT * FROM nonexistent_table` | Table does not exist in schema |
| Column Not Found | `SELECT bad_column FROM users` | Column does not exist in referenced table |
| Permission Denied | `SELECT * FROM restricted_table` | User lacks SELECT privilege |

**Syntax Rules:**

- Keywords must be spelled correctly and in valid clause order.
- Object names must be qualified if ambiguous across schemas.
- Column references must be resolvable to exactly one table in the FROM clause.

**Constraints and Limitations:**

- Semantic analysis requires catalog access; very large schemas may slow parsing slightly.
- Some databases defer certain checks (e.g., permission checks on views) until execution.

### Annotated Complete Code Examples

**Example 1: Syntax Error**

```sql
-- Incorrect syntax: FORM instead of FROM
SELECT * FORM users;

-- Expected Error:
-- ERROR: syntax error at or near "FORM"
-- LINE 1: SELECT * FORM users;
```

**Why this error occurs:** The parser expects the keyword `FROM` after the select list. The token `FORM` is not a recognized keyword and does not match any expected grammar rule, so parsing fails immediately. No semantic analysis or optimization is attempted.

**Example 2: Semantic Error (Column Not Found)**

```sql
-- Assume table 'users' exists but has no column 'email'
SELECT user_id, email FROM users;

-- Expected Error:
-- ERROR: column "email" does not exist
-- LINE 1: SELECT user_id, email FROM users;
```

**Why this error occurs:** The parser accepts the syntax. The semantic analyzer then queries the catalog for the `users` table's columns. Since `email` is not among them, the analyzer raises an error. SAP HANA documentation confirms this phase "checks the catalog to verify whether the objects called by the SQL statement are present".

### Real-World Cases

**Case 1: Application Development**

A developer writes a query joining `orders` and `customers`. Semantic analysis verifies both tables exist, the join columns are of compatible types, and the application's database user has SELECT permission on both tables. If the developer misspells a column name, the error is caught before any data is accessed.

**Case 2: Security Enforcement**

A restricted database user attempts to query a table containing sensitive data. Semantic analysis checks the user's privileges and denies access before the optimizer even begins, preventing any information leakage through query behavior.

### References

- Query Optimization: Processing Example - https://resources.mpi-inf.mpg.de/departments/d5/teaching/ws07_08/queryoptimization/Lecture1.pdf
- SAP HANA Performance Guide for Developers - https://help.sap.com/docs/SAP_HANA_PLATFORM/9de0171a6027400bb3b9bee385222eff/27875edb31d7436ea62031d8b167052c.html

---

## 2. Query Planning & Optimization

### Definitions

**Core Definition:** Query planning and optimization is the process where the optimizer explores alternative execution strategies for a validated query and selects the plan with the lowest estimated cost.

**Technical Definition:** The optimizer performs logical rewriting (transformations that preserve semantics while potentially improving performance), enumerates alternative logical and physical plans, estimates the cost of each, and passes the lowest-cost plan to the row-source generator. The plan generator considers different join orders, join algorithms, access paths (index vs. table scan), and parallelism options.

**Beginner-Friendly Explanation:** The optimizer is like a travel agent planning your route. You tell it where you want to go (your query). It considers flying, driving, or taking the train (different algorithms). It checks traffic, distance, and cost (statistics and cost model). Then it books the option that gets you there fastest for the least cost.

### Purposes

- To transform a declarative SQL statement into a procedural execution plan.
- To explore alternative join orders, access methods, and algorithms.
- To apply logical rewrites (e.g., subquery unnesting, OR expansion) that simplify execution.
- To select the plan with the lowest estimated cost based on available statistics.

### Syntax Rules and Structure

**Query Transformation Examples:**

| Original Construct | Transformed To | Benefit |
|--------------------|----------------|---------|
| `WHERE a = 1 OR a = 2` | `UNION ALL` of two queries | Enables index use |
| `WHERE x IN (SELECT y ...)` | Join with subquery result | Enables hash/merge join |
| Correlated subquery | Unnested join | Eliminates repeated execution |

**Cost-Based Enumeration:**

The optimizer compares plans from two perspectives:

- **Logical enumerators:** Different tree shapes and operator orderings (e.g., which table is joined first).
- **Physical enumerators:** Different algorithm choices for each operator (e.g., Hash Join vs. Nested Loop Join).

**Syntax Rules:**

- The optimizer has limited time for plan exploration; a limit is applied to cost enumerations to reduce compilation time.
- The optimizer does not rewrite queries if the rewritten plan has a higher cost.

**Constraints and Limitations:**

- Optimizer decisions are only as good as the available statistics; stale or missing statistics lead to poor plans.
- Some transformations are only valid under certain semantic guarantees (e.g., subquery unnesting requires no aggregates in the subquery).
- The optimizer cannot always find the globally optimal plan in the available time.

### Annotated Complete Code Examples

**Example 1: OR Expansion**

```sql
-- Original query with OR
SELECT employee_id, department_id
FROM employees
WHERE department_id = 50 OR department_id = 60;

-- Optimizer may transform to UNION ALL
SELECT employee_id, department_id FROM employees WHERE department_id = 50
UNION ALL
SELECT employee_id, department_id FROM employees WHERE department_id = 60;
```

**Why this transformation occurs:** The `OR` predicate may prevent efficient index usage in some databases. Splitting into two `UNION ALL` branches allows each branch to use an index range scan on `department_id`. Oracle documentation describes this as the query transformer determining "whether it is advantageous to rewrite the original SQL statement into a semantically equivalent SQL statement with a lower cost".

**Example 2: Subquery Unnesting**

```sql
-- Original: Correlated IN subquery
SELECT sales.*
FROM sales
WHERE cust_id IN (SELECT cust_id FROM customers);

-- Optimizer transforms to join (customers.cust_id is PK)
SELECT sales.*
FROM sales, customers
WHERE sales.cust_id = customers.cust_id;
```

**Why this transformation occurs:** Oracle documentation states that "in subquery unnesting, the optimizer transforms a nested query into an equivalent join statement, and then optimizes the join". The join form enables the optimizer to use hash or merge join algorithms, which may be far more efficient than evaluating the subquery for each row of `sales`.

### Real-World Cases

**Case 1: Data Warehouse Query Optimization**

A complex analytical query joins five tables with multiple subqueries. The optimizer unnest subqueries, reorders joins to filter early, and selects hash joins for large unindexed datasets. The resulting plan reduces execution time from minutes to seconds.

**Case 2: OLTP Query Rewriting**

A frequently executed OLTP query uses a correlated subquery. The optimizer unnest it into a join, enabling the use of a covering index and eliminating per-row subquery execution.

### References

- Oracle Database SQL Tuning Guide (Query Transformer) - https://docs.oracle.com/cd/F82042_01/tgsql/sql-tuning-guide.pdf
- Oracle Database Performance Tuning Guide (Subquery Unnesting) - https://docs.oracle.com/cd/E11882%5F01/server.112/e41573/optimops.htm
- SAP HANA Performance Guide (Query Optimizer) - https://help.sap.com/docs/SAP_HANA_PLATFORM/9de0171a6027400bb3b9bee385222eff/27875edb31d7436ea62031d8b167052c.html

---

## 3. Cost Estimation

### Definitions

**Core Definition:** Cost estimation is the optimizer's process of assigning a numeric value to each candidate execution plan, representing the estimated resources required to execute that plan.

**Technical Definition:** The cost model combines disk I/O, CPU usage, and memory consumption into a single cost metric, sometimes normalized to execution time. Oracle documentation states that "the query optimizer uses disk I/O, CPU usage, and memory usage as units of work". The cost of an access path depends on factors such as the number of blocks scanned (for table scans) or B-tree levels and leaf blocks (for index scans).

**Beginner-Friendly Explanation:** The cost is like a bill the optimizer estimates for each possible plan. A plan that reads 1,000 blocks costs more than one that reads 10 blocks. The optimizer adds up all the estimated costs and picks the cheapest plan.

### Purposes

- To provide a comparable numeric value for comparing alternative execution plans.
- To account for different resource types (I/O, CPU, memory) in a unified metric.
- To enable cost-based selection of access paths, join algorithms, and join orders.

### Syntax Rules and Structure

**Cost Components:**

| Resource | Measurement | Example |
|----------|-------------|---------|
| Disk I/O | Number of block reads/writes | Single-block reads, multi-block reads |
| CPU | Estimated CPU cycles or execution time | Comparison operations, hash computations |
| Memory | Buffers required | Hash table size, sort area |

**Cost Formula (Conceptual):**

```
Total Cost = (I/O Cost) + (CPU Cost) + (Memory Cost)
```

**Normalization:** Costs are normalized to a common unit, often execution time, so that I/O and CPU costs can be combined.

**Syntax Rules:**

- The cost of a table scan depends on the number of blocks under the high water mark and the multiblock read count.
- The cost of an index scan depends on B-tree levels, leaf blocks scanned, and the clustering factor (how well row order matches index order).
- Join cost combines the access costs of both inputs plus the join operation cost.

**Constraints and Limitations:**

- The cost model is an approximation; actual resource consumption may differ.
- Cost values are not portable across database systems; each RDBMS has its own cost model.
- The cost unit is internal and not directly interpretable as seconds or milliseconds.

### Annotated Complete Code Examples

**Example 1: Access Path Cost Comparison**

```sql
-- Option A: Full table scan (assuming 1000 blocks, multiblock read count 8)
-- Cost ≈ 1000 blocks / 8 blocks-per-read ≈ 125 I/O operations

-- Option B: Index range scan (assuming 10 leaf blocks, 50 rows fetched)
-- Cost ≈ 10 leaf blocks + 50 rowid fetches ≈ 60 I/O operations

-- Optimizer chooses Option B (lower cost)
```

**Why this comparison matters:** Oracle documentation explains that the cost of a scan "depends on the number of blocks to be scanned and the multiblock read count value". For a query accessing a small fraction of a table, an index scan (Option B) is cheaper. For a query accessing most of the table, a full table scan (Option A) is cheaper because multiblock reads are more efficient than many random rowid fetches.

### Real-World Cases

**Case 1: Index vs. Full Table Scan Decision**

A query filters on a column where 80% of rows match. The optimizer estimates that an index scan would require 800,000 rowid fetches (random I/O), while a full table scan reads 100,000 blocks sequentially (multiblock I/O). Despite scanning more data, the full table scan has lower cost and is chosen.

**Case 2: Join Order Selection**

For a three-table join, the optimizer estimates the cost of joining A-B-C versus C-B-A. If filtering on C's indexed column reduces rows to 10 before joining, starting with C produces a smaller intermediate result and lower overall cost.

### References

- Oracle Database SQL Tuning Guide (Estimator and Cost) - https://docs.oracle.com/cd/F82042_01/tgsql/sql-tuning-guide.pdf
- Oracle Database Performance Tuning Guide (Cost) - https://docs.oracle.com/cd/E28271_01/server.1111/e16638/optimops.htm
- Optimizer Cost Function (Patent) - http://patentimages.storage.googleapis.com/pdfs/US6957211.pdf

---

## 4. Cardinality Estimation

### Definitions

**Core Definition:** Cardinality estimation is the optimizer's process of predicting the number of rows that will be returned by each operation in an execution plan.

**Technical Definition:** Cardinality is derived from table statistics (row counts, distinct values, histograms) combined with predicate selectivity estimates. Selectivity is the fraction of rows passing a predicate, ranging from 0.0 (no rows) to 1.0 (all rows). The estimator computes cardinality for base tables, joins, filters, DISTINCT operations, and GROUP BY operations.

**Beginner-Friendly Explanation:** Cardinality estimation is the optimizer's guess at how many rows each step will produce. If it thinks a filter will reduce 1 million rows to 100 rows, it might choose an index lookup. If it thinks the filter will leave 800,000 rows, it might choose a table scan instead.

### Purposes

- To provide the row count input needed for cost estimation.
- To determine the optimal join order and join algorithm.
- To decide whether an index is selective enough to be useful.
- To estimate memory requirements for hash joins and sorts.

### Syntax Rules and Structure

**Selectivity Formula:**

```
Selectivity = (Rows passing predicate) / (Total rows)
```

**Cardinality Computation:**

```
Estimated Cardinality = Total Rows × Selectivity
```

**Statistics Used:**

| Statistic | Description | Source |
|-----------|-------------|--------|
| Row count | Total rows in table | Catalog statistics |
| Distinct values | Number of unique values in column | Histograms |
| Histogram | Distribution of values across ranges | DBMS_STATS or equivalent |
| Null count | Number of NULL values | Column statistics |

**Syntax Rules:**

- The estimator uses statistics if available; otherwise it uses dynamic sampling or internal defaults.
- For a predicate `last_name = 'Smith'`, selectivity = 1 / distinct values in `last_name` (assuming uniform distribution).
- For a join, cardinality depends on the selectivity of the join predicate and the input cardinalities.

**Constraints and Limitations:**

- Cardinality estimates can be wrong, especially for correlated predicates or stale statistics.
- The "ascending key problem" causes underestimates for recently inserted values beyond the statistics range.
- Uniform distribution assumptions fail for skewed data.

### Annotated Complete Code Examples

**Example 1: Cardinality Estimate in Execution Plan**

```sql
-- Query with cardinality estimates shown in plan
EXPLAIN SELECT * FROM employees WHERE department_id = 50;

-- Plan output (conceptual):
-- | Id | Operation         | Name        | Rows | Cost |
-- |----|-------------------|-------------|------|------|
-- |  0 | SELECT STATEMENT  |             |   10 |    3 |
-- |  1 | TABLE ACCESS FULL | EMPLOYEES   |   10 |    3 |
```

**Why this output occurs:** Oracle documentation states that "the cardinality is the number of rows returned by each operation in an execution plan" and that "the Rows column in an execution plan shows the estimated cardinality". Here, the optimizer estimates 10 rows will match `department_id = 50` based on statistics showing 10 employees in that department.

**Example 2: Cardinality Misestimate Leading to Bad Plan**

```sql
-- Query on a column with skewed distribution
SELECT * FROM orders WHERE status = 'pending';

-- If 90% of orders are 'pending' but statistics say otherwise,
-- the optimizer might choose an index scan when a table scan is better.
```

**Why this is a problem:** Oracle documentation notes that "the optimizer occasionally picks a suboptimal default plan because of a cardinality misestimate". If statistics show a uniform distribution but the actual data is heavily skewed toward 'pending', the optimizer underestimates the row count and chooses an index scan, which then performs poorly.

### Real-World Cases

**Case 1: SQL Server 2014 Cardinality Estimator**

Microsoft redesigned the cardinality estimator in SQL Server 2014 to handle modern workloads better. The new CE "incorporates assumptions and algorithms that work well on modern OLTP and data warehousing workloads". For SAP NetWeaver applications, the new CE changed estimates for out-of-range values from 1 row to approximately 1,000 rows in a 1-million-row table with 100 distinct values.

**Case 2: Adaptive Query Plans**

Oracle Database can use adaptive plans that collect runtime statistics and switch between predetermined subplans if the original cardinality estimate proves incorrect. A statistics collector is inserted at key points to gather actual cardinality and decide which subplan to use.

### References

- Oracle Database SQL Tuning Guide (Cardinality) - https://docs.oracle.com/cd/F82042_01/tgsql/sql-tuning-guide.pdf
- SQL Server 2014 Cardinality Estimation - https://learn.microsoft.com/it-ch/previous-versions/sql/2014/relational-databases/performance/cardinality-estimation-sql-server
- New Functionality in SQL Server 2014: Cardinality Estimation - https://learn.microsoft.com/es-es/archive/blogs/saponsqlserver/new-functionality-in-sql-server-2014-part-2-new-cardinality-estimation

---

## 5. Join Algorithms

### Definitions

**Core Definition:** Join algorithms are the physical techniques the execution engine uses to combine rows from two or more tables based on a join condition.

**Technical Definition:** The three primary join algorithms are Nested Loop Join (for each row in the outer table, scan the inner table for matches), Hash Join (build a hash table from one input, probe with the other), and Merge Join (sort both inputs on the join key, then merge in a single pass). The optimizer selects the algorithm based on input sizes, indexing, sort order, and join predicate type.

**Beginner-Friendly Explanation:** Joins are like matching two lists. Nested Loop Join is like checking every name on List A against every name on List B. Hash Join is like building a phone book from List B, then looking up each name from List A. Merge Join is like having both lists already sorted alphabetically, then walking through them together.

### Purposes

- To combine rows from multiple tables based on a join predicate.
- To minimize the cost of matching rows when inputs vary in size and indexing.
- To leverage sorted order or hash structures for efficient matching.

### Syntax Rules and Structure

**Nested Loop Join:**

```
For each row r in Outer:
    For each row s in Inner:
        If r.join_key = s.join_key: output (r, s)
```

**Hash Join:**

```
Phase 1 (Build): Hash all rows of smaller input into buckets
Phase 2 (Probe): For each row in larger input, hash join key and probe bucket for matches
```

**Merge Join:**

```
Sort both inputs by join key (if not already sorted)
Maintain one cursor per input
While both cursors have rows:
    If keys match: output pair, advance both
    If left key < right key: advance left
    If left key > right key: advance right
```

**Syntax Rules:**

- Hash and Merge joins require an equijoin predicate (`=`); Nested Loop Join can handle inequality joins.
- Merge Join requires both inputs sorted on the join key; if not, explicit sorts are added.
- Hash Join is memory-intensive; large inputs may spill to disk (Grace hash).

**Constraints and Limitations:**

- Nested Loop Join has O(N×M) complexity in the worst case.
- Hash Join requires sufficient memory for the build input's hash table.
- Merge Join has the overhead of sorting if inputs are not already sorted.

### Annotated Complete Code Examples

**Example 1: Nested Loop Join Execution Plan**

```sql
-- Query joining employees to departments
SELECT e.last_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id;

-- Plan showing Nested Loop Join (Oracle):
-- | Id | Operation                     | Name        | Rows |
-- |----|-------------------------------|-------------|------|
-- |  0 | SELECT STATEMENT              |             |   10 |
-- |  1 |  NESTED LOOPS                 |             |   10 |
-- |  2 |   TABLE ACCESS FULL           | EMPLOYEES   |   10 |
-- |  3 |   TABLE ACCESS BY INDEX ROWID | DEPARTMENTS |    1 |
-- |  4 |    INDEX UNIQUE SCAN          | DEPT_ID_PK  |    1 |
```

**Why this plan occurs:** For each row in `employees` (outer), the database uses the unique index on `departments.department_id` (inner) to find the matching department. Oracle documentation shows this pattern where the inner side uses "TABLE ACCESS BY INDEX ROWID" and "INDEX UNIQUE SCAN". Nested Loop is efficient here because the inner side is highly indexed and returns few rows.

**Example 2: Hash Join Execution Plan**

```sql
-- Query joining two large unindexed tables
SELECT o.order_id, c.customer_name
FROM orders o, customers c
WHERE o.customer_id = c.customer_id;

-- Plan showing Hash Join (conceptual):
-- | Id | Operation          | Name      | Rows  |
-- |----|--------------------|-----------|-------|
-- |  0 | SELECT STATEMENT   |           | 10000 |
-- |  1 |  HASH JOIN         |           | 10000 |
-- |  2 |   TABLE ACCESS FULL| CUSTOMERS |  1000 |
-- |  3 |   TABLE ACCESS FULL| ORDERS    | 10000 |
```

**Why this plan occurs:** Neither table has an index on the join column, and both are large. The optimizer chooses Hash Join: the smaller table (`customers`, 1,000 rows) is hashed into memory, then the larger table (`orders`) is scanned and probed against the hash table. Amazon Redshift documentation notes that "a hash join is typically faster than a nested loop join" for such cases.

### Real-World Cases

**Case 1: OLTP Point Lookup Join**

An application retrieves an order and its customer details. The `orders` table has 10 million rows, `customers` has 1 million. The query joins on `customer_id` with a unique index on `customers.customer_id`. The optimizer chooses Nested Loop Join, using the index to find one customer per order.

**Case 2: Data Warehouse Fact-Dimension Join**

A fact table with 100 million rows joins to a dimension table with 10,000 rows. Neither is sorted on the join key. The optimizer chooses Hash Join, building a hash table from the dimension (smaller) and probing with the fact table (larger).

### References

- CS143: Query Processing and Join Algorithms - http://oak.cs.ucla.edu/classes/cs143/notes/join.pdf
- Optimizing Query Performance (Microsoft) - https://learn.microsoft.com/en-us/previous-versions/cc917719(v=technet.10)
- Amazon Redshift EXPLAIN Joins - https://docs.aws.amazon.com/prescriptive-guidance/latest/query-lifecycle-redshift/explain-joins.html

---

## 6. Parallel Query Execution

### Definitions

**Core Definition:** Parallel query execution is the technique of dividing a single query's work across multiple CPU cores or worker threads to reduce response time for large queries.

**Technical Definition:** The execution engine horizontally partitions input data into approximately equal sets, assigns one set to each CPU/worker, and performs the same operation (scan, join, aggregate) on each set independently. A parallelism (or exchange) iterator handles data partitioning and movement between threads. The query coordinator (QC) manages the parallel plan and aggregates results.

**Beginner-Friendly Explanation:** Parallel execution is like having multiple people work on one big task. Instead of one person reading a whole book, ten people each read one chapter, then combine their notes. The book gets read ten times faster—if you have ten people and the task can be divided.

### Purposes

- To reduce response time for large queries by using multiple CPUs.
- To scale up (process more data in the same time) by adding CPUs.
- To keep many CPUs busy on servers running a small number of concurrent queries.

### Syntax Rules and Structure

**Parallel Execution Plan Elements (Oracle):**

| Element | Purpose |
|---------|---------|
| PX COORDINATOR | Query Coordinator that controls parallel execution |
| PX SEND | Sends rows to another set of parallel processes |
| PX RECEIVE | Receives rows from another set of parallel processes |
| PX BLOCK ITERATOR | Divides table scan into pieces for parallel workers |

**Syntax Rules:**

- Parallelism is cost-based; the optimizer decides whether to parallelize based on estimated query cost.
- Degree of Parallelism (DOP) is determined at execution time, considering available CPUs and configuration.
- Data is partitioned by hash, range, or broadcast depending on the operation.
- Hybrid hash distribution adaptively chooses between hash and broadcast at runtime based on actual row counts.

**Constraints and Limitations:**

- Parallelism adds overhead; it is inappropriate for small queries where overhead dominates.
- On servers with many concurrent queries (OLTP), parallelism may reduce overall throughput.
- Memory pressure can limit hash and merge joins in parallel plans.

### Annotated Complete Code Examples

**Example 1: Parallel Query Plan (Oracle)**

```sql
-- Enable parallelism on a table
ALTER TABLE emp2 PARALLEL 2;

-- Query with GROUP BY
EXPLAIN PLAN FOR
SELECT department_id, COUNT(*) FROM emp2 GROUP BY department_id;

-- Parallel plan output:
-- | Id | Operation               | Name   | TQ     | IN-OUT |
-- |----|-------------------------|--------|--------|--------|
-- |  0 | SELECT STATEMENT        |        |        |        |
-- |  1 | PX COORDINATOR          |        |        |        |
-- |  2 | PX SEND QC (RANDOM)     | :TQ10001| Q1,01 | P->S   |
-- |  3 | HASH GROUP BY           |        | Q1,01 | PCWP   |
-- |  4 | PX RECEIVE              |        | Q1,01 | PCWP   |
-- |  5 | PX SEND HASH            | :TQ10000| Q1,00 | P->P   |
-- |  6 | HASH GROUP BY           |        | Q1,00 | PCWP   |
-- |  7 | PX BLOCK ITERATOR       |        | Q1,00 | PCWP   |
-- |  8 | TABLE ACCESS FULL       | EMP2   | Q1,00 | PCWP   |
```

**Why this plan occurs:** Oracle documentation explains that "one set of parallel execution servers scans EMP2 in parallel, while the second set performs the aggregation for the GROUP BY operation". The `PX BLOCK ITERATOR` splits the table scan among workers. `PX SEND` and `PX RECEIVE` move rows between the scan workers and aggregation workers. `PX SEND QC` returns results to the coordinator.

**Example 2: Adaptive Parallel Distribution (Oracle)**

```sql
-- Query joining a small departments table to a large employees table in parallel
-- The optimizer uses hybrid hash distribution

-- If actual rows from departments < 2 × DOP:
--   Broadcast departments to all parallel servers
-- If actual rows from departments >= 2 × DOP:
--   Hash-distribute departments to parallel servers
```

**Why this adaptivity matters:** Oracle documentation states that "the hybrid hash distribution technique is an adaptive parallel data distribution that does not decide the final data distribution method until execution time". A statistics collector counts actual rows; if the row count is below a threshold (twice the DOP), it switches from hash to broadcast. This avoids the overhead of hashing when broadcasting is more efficient.

### Real-World Cases

**Case 1: Large Analytical Query**

A data warehouse query scans 500 million rows and aggregates by region. With a DOP of 16, the scan is divided into 16 pieces, and aggregation is performed in parallel. The query completes in 1/16th the time of a serial execution.

**Case 2: Partitioned Table Query**

A table is partitioned by date. A query filtering on `WHERE date IN (13, 17, 25)` uses a parallel plan that allocates workers to the three specific partitions, rather than scanning all partitions.

### References

- Oracle Database SQL Tuning Guide (Parallel Execution) - https://docs.oracle.com/cd/F82042_01/tgsql/sql-tuning-guide.pdf
- Introduction to Parallel Query Execution (Microsoft) - https://learn.microsoft.com/en-us/archive/blogs/craigfr/introduction-to-parallel-query-execution
- SQL Server Query Processing Architecture Guide - https://learn.microsoft.com/it-ch/sql/relational-databases/query-processing-architecture-guide

---

## Summary Table

| Concept | Key Purpose | Critical Input | Primary Output |
|---------|-------------|----------------|----------------|
| Parsing & Semantic Analysis | Validate SQL syntax and objects | SQL text | Validated AST |
| Query Planning & Optimization | Find lowest-cost execution strategy | Validated query, statistics | Execution plan |
| Cost Estimation | Assign numeric cost to plans | I/O, CPU, memory estimates | Comparable cost values |
| Cardinality Estimation | Predict row counts per operation | Statistics, selectivity | Estimated rows |
| Join Algorithms | Combine rows from multiple tables | Input sizes, indexes, sort order | Joined result set |
| Parallel Query Execution | Reduce response time via multiple CPUs | Available CPUs, query cost | Parallel plan |

---

## References

- Query Optimization: Processing Example - https://resources.mpi-inf.mpg.de/departments/d5/teaching/ws07_08/queryoptimization/Lecture1.pdf
- Oracle Database SQL Tuning Guide - https://docs.oracle.com/cd/F82042_01/tgsql/sql-tuning-guide.pdf
- Optimizer Cost Function (Patent) - http://patentimages.storage.googleapis.com/pdfs/US6957211.pdf
- SAP HANA Performance Guide for Developers - https://help.sap.com/docs/SAP_HANA_PLATFORM/9de0171a6027400bb3b9bee385222eff/27875edb31d7436ea62031d8b167052c.html
- Oracle Database Concepts (Row Source Generation) - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/database-concepts.pdf
- Oracle Database Performance Tuning Guide (Cost) - https://docs.oracle.com/cd/E28271_01/server.1111/e16638/optimops.htm
- SQL Server 2014 Cardinality Estimation - https://learn.microsoft.com/it-ch/previous-versions/sql/2014/relational-databases/performance/cardinality-estimation-sql-server
- New Functionality in SQL Server 2014: Cardinality Estimation - https://learn.microsoft.com/es-es/archive/blogs/saponsqlserver/new-functionality-in-sql-server-2014-part-2-new-cardinality-estimation
- Amazon Redshift EXPLAIN Joins - https://docs.aws.amazon.com/prescriptive-guidance/latest/query-lifecycle-redshift/explain-joins.html
- CS143: Query Processing and Join Algorithms - http://oak.cs.ucla.edu/classes/cs143/notes/join.pdf
- Introduction to Parallel Query Execution (Microsoft) - https://learn.microsoft.com/en-us/archive/blogs/craigfr/introduction-to-parallel-query-execution
- SQL Server Query Processing Architecture Guide - https://learn.microsoft.com/it-ch/sql/relational-databases/query-processing-architecture-guide
- Optimizing Query Performance (Microsoft) - https://learn.microsoft.com/en-us/previous-versions/cc917719(v=technet.10)
- Oracle Database Performance Tuning Guide (Subquery Unnesting) - https://docs.oracle.com/cd/E11882%5F01/server.112/e41573/optimops.htm