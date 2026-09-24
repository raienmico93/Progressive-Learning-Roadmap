# SQL Window Function Optimization & Execution: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Window function optimization is the practice of structuring queries, indexes, and database configurations so that window functions execute efficiently, avoiding excessive sorting, memory spills, and redundant computations.

**Technical Definition:** Database engines execute window functions through a combination of physical operators (Segment, Sequence Project in SQL Server; WindowAgg in PostgreSQL) that partition, sort, and compute values across row sets. Optimization involves aligning index structures with partition/order requirements, minimizing sort operations, managing memory grants to prevent tempdb spills, and consolidating window specifications to reduce redundant passes over data.

**Beginner-Friendly Explanation:** Window functions are powerful but can be slow if the database has to sort huge amounts of data or run out of memory. Optimization means giving the database the right indexes so it doesn't need to sort, keeping the data small so it fits in memory, and avoiding unnecessary repetition.

### Key Characteristics

- **Sort-dependent:** Window functions often require sorted input; sorting is frequently the bottleneck .
- **Operator pipeline:** Execution uses specialized operators (Segment, Sequence Project, WindowAgg) that process rows in sequence .
- **Index-sensitive:** A well-designed index can eliminate sort operations entirely .
- **Memory-intensive:** Large sorts may spill to tempdb, causing severe performance degradation .
- **Combinatorial risk:** Multiple distinct `OVER` clauses can cause repeated sorting unless carefully designed .

### Prerequisites

- **Window function syntax:** `OVER`, `PARTITION BY`, `ORDER BY`, frame clauses.
- **Execution plan basics:** Understanding of Sort, Segment, Sequence Project, and WindowAgg operators.
- **Indexing fundamentals:** Composite indexes, covering indexes, and key order.

### Related Programming Areas

- **Database Administration:** Memory configuration, index maintenance.
- **Query Tuning:** Execution plan analysis, statistics management.
- **Application Development:** Writing efficient analytical queries.

### Core Concepts / Features

1. Under the Hood: Segment and Sequence Operators
2. Indexing for Window Functions: The POC Rule
3. Performance Bottlenecks: Memory Spills
4. Combinatorial Windows

---

## 1. Under the Hood: Segment and Sequence Operators

### Definitions

**Core Definition:** Database engines execute window functions through a pipeline of physical operators that partition, order, and compute values row by row. In SQL Server, these are **Segment** and **Sequence Project**; in PostgreSQL, they are **WindowAgg** nodes.

**Technical Definition:** The **Segment** operator detects partition boundaries by comparing the `PARTITION BY` values of consecutive rows; it outputs a "segment flag" indicating when a new partition begins . The **Sequence Project** operator uses this flag (and optional ordering-change flags) to compute window function values sequentially. For `ROW_NUMBER`, a single Segment operator suffices. For `RANK` and `DENSE_RANK`, a second Segment operator grouped by both partition and order columns is required to detect ties . PostgreSQL uses a stack of `WindowAgg` nodes, each processing one window specification; shared partition/order definitions allow multiple functions to share a single `WindowAgg` node .

**Beginner-Friendly Explanation:** The database reads rows in sorted order. The Segment operator marks "new group starts here." The Sequence Project operator then counts or ranks rows as it moves through each group. It's like a conveyor belt with a sensor that says "new box of items" and a counter that numbers each item.

### Purposes

- To understand why window functions require sorted input.
- To interpret execution plans and identify the operators involved.
- To recognize that Segment and Sequence Project are lightweight; the real cost is in Sort .
- To optimize by eliminating or reducing Sort operations.

### Syntax Rules and Structure

**SQL Server Execution Plan Operators:**

| Operator | Purpose | Appears When |
|----------|---------|--------------|
| `Segment` | Detects partition/order boundaries | `PARTITION BY` or `ORDER BY` present  |
| `Sequence Project` | Computes window function values | Any window function  |
| `Window Spool` | Caches frame data for sliding windows | Frame clause with `PRECEDING`/`FOLLOWING`  |
| `Sort` | Orders rows for partitioning | No suitable index  |

**PostgreSQL Execution Plan Operators:**

| Operator | Purpose |
|----------|---------|
| `WindowAgg` | Processes one window specification |
| `Sort` | Orders rows before WindowAgg |

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| Segment flag | Indicates whether current row starts a new partition |
| Order-change flag | Indicates whether the `ORDER BY` value changed (for RANK/DENSE_RANK) |
| Sequence counter | Tracks the running value for the function |

**Syntax Rules:**

- `ROW_NUMBER` requires one Segment operator (partition only) .
- `RANK` and `DENSE_RANK` require two Segment operators (partition + order) .
- PostgreSQL deduplicates `WindowAgg` nodes when multiple functions share identical `PARTITION BY` and `ORDER BY` clauses .
- The absence of a frame clause and an explicit equivalent frame are considered different for deduplication purposes in PostgreSQL .

**Constraints and Limitations:**

- Segment and Sequence Project are not the bottleneck; Sort is .
- `Window Spool` (for sliding frames) can consume significant memory and spill to disk .

### Annotated Complete Code Examples

**Example 1: SQL Server Execution Plan for ROW_NUMBER**

```sql
-- Query with ROW_NUMBER
SELECT CustomerID, SalesOrderID,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RowNumber
FROM Sales.SalesOrderHeader;
```

**Execution Plan (simplified):**
```
Sequence Project (Compute Scalar)
  -> Segment [Group By: CustomerID]
    -> Index Scan (NonClustered)
       [Index: IX_SalesOrderHeader_Customer_OrderDate]
```

**Why this plan occurs:** The index provides rows pre-sorted by `CustomerID, OrderDate`. No Sort operator is needed. The Segment operator flags new `CustomerID` partitions. Sequence Project increments the row number within each partition .

**Example 2: PostgreSQL Execution Plan for Multiple Windows**

```sql
SELECT count(*) OVER (PARTITION BY j) AS count_j,
       sum(i)   OVER (PARTITION BY j) AS sum_j,
       count(*) OVER (PARTITION BY i) AS count_i
FROM values_table;
```

**Execution Plan (simplified):**
```
WindowAgg [PARTITION BY i]
  -> Sort [i]
    -> WindowAgg [PARTITION BY j]
      -> Sort [j]
        -> Values Scan
```

**Why this plan occurs:** The `count_j` and `sum_j` functions share the same partition (`j`), so they use one `WindowAgg` node. The `count_i` function has a different partition (`i`), requiring a separate `WindowAgg` and a second Sort. The data is sorted twice .

### Real-World Cases

**Case 1: Top-N Per Group**

A query finds the top 3 orders per customer. `ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate)` with an index on `(CustomerID, OrderDate)` avoids sorting entirely, executing in milliseconds.

**Case 2: Multiple Rankings**

A query computes both `ROW_NUMBER` and `RANK` with the same `PARTITION BY` and `ORDER BY`. Both share the same Segment and Sequence Project pipeline, so the overhead of the second function is minimal .

### References

- Microsoft Learn — OVER Clause (Performance Benefits, Indexing) - https://learn.microsoft.com/pl-pl/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-linux-2017#2
- PostgreSQL Documentation — Window Function Processing (WindowAgg deduplication) - https://www.postgresql.org/message-id/attachment/173563/v0-draft-0001-doc-more-thoroughly-explain-window-function-processing.patch
- T-SQL Window Functions (Execution Plan Operators) - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/pilimi-zlib/ia20241105/annas_archive_data__aacid__ia2_acsmpdf_files__20240823T234744Z--20240823T234745Z/aacid__ia2_acsmpdf_files__20240823T234744Z__BqSKjLdpo2KsSyanJXBy7G#50#31

---

## 2. Indexing for Window Functions: The POC Rule

### Definitions

**Core Definition:** The POC rule (Partition, Order, Cover) states that an optimal index for a window function query should include the `PARTITION BY` columns first, then the `ORDER BY` columns, and finally `INCLUDE` the columns needed for the calculation or output.

**Technical Definition:** For a query with `OVER (PARTITION BY c ORDER BY o)` that accesses columns `v`, an index on `(c, o) INCLUDE (v)` allows the database to scan rows in the exact order required, eliminating the Sort operator entirely . The partition columns must lead the index because they define the segment boundaries; the order columns follow because they define the sequence within each partition . Covering columns avoid key lookups .

**Beginner-Friendly Explanation:** Think of a filing cabinet. If you want to process files by department (partition) and then by date (order), you want the files already sorted that way. The POC rule tells you how to organise the cabinet so the database doesn't have to re-sort everything.

### Purposes

- To eliminate Sort operators from window function execution plans.
- To reduce I/O by avoiding key lookups (covering index).
- To enable batch-mode execution in SQL Server .
- To improve performance from seconds to milliseconds on large datasets.

### Syntax Rules and Structure

**Optimal Index Pattern:**

```sql
CREATE NONCLUSTERED INDEX IX_Window_Optimal
ON TableName (PartitionColumn, OrderColumn)
INCLUDE (ValueColumn1, ValueColumn2);
```

**Component Breakdown:**

| Index Component | Purpose |
|-----------------|---------|
| Partition columns (first) | Defines segment boundaries; enables partition detection |
| Order columns (second) | Defines sequence within partitions |
| INCLUDE columns | Covers query columns; avoids key lookups |

**Syntax Rules:**

- Partition columns must come before order columns in the index key .
- `INCLUDE` columns are not part of the sort order but are available in the leaf level.
- The index must be usable by the optimizer (statistics up to date, no parameter sniffing issues).
- For multiple window functions with different orderings, multiple indexes may be needed.

**Constraints and Limitations:**

- An index can only support one sort order; queries with different `ORDER BY` directions may still require sorting.
- Covering indexes increase storage and write overhead.
- High-cardinality partition columns can make the index large .

### Annotated Complete Code Examples

**Example 1: Index Eliminating Sort**

```sql
-- Query: Running total by customer ordered by date
SELECT CustomerID, OrderDate, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate
                           ROWS UNBOUNDED PRECEDING) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Optimal index
CREATE NONCLUSTERED INDEX IX_SalesOrderHeader_Customer_OrderDate
ON Sales.SalesOrderHeader (CustomerID, OrderDate)
INCLUDE (TotalDue);
```

**Why this works:** The index provides rows in `(CustomerID, OrderDate)` order. The optimizer can scan the index directly, avoiding a Sort operator. The `INCLUDE (TotalDue)` makes it a covering index, avoiding key lookups for `TotalDue` .

**Example 2: Index for Multiple Window Functions**

```sql
-- Query with two different orderings
SELECT CustomerID,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS rn,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY SalesOrderID) AS total
FROM Sales.SalesOrderHeader;
```

**Why two indexes may be needed:** The first function orders by `OrderDate`, the second by `SalesOrderID`. A single index cannot satisfy both orderings simultaneously. The optimizer may choose one index and sort for the other, or use two separate scans .

### Real-World Cases

**Case 1: Financial Running Balance**

A bank statement query with `(AccountID, TransactionDate)` index and `INCLUDE (Amount)` runs in milliseconds on millions of rows because the index provides the required order without sorting.

**Case 2: Top-N Dashboard**

An e-commerce dashboard query with `(CategoryID, SalesDate)` index and `INCLUDE (ProductID, Revenue)` enables instant top-N per category without sorting or lookups.

### References

- Microsoft Learn — OVER Clause (Index Recommendations) - https://learn.microsoft.com/pl-pl/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-linux-2017#2
- 华为云社区 — SQL Server中OVER子句的4个性能陷阱 - https://bbs.huaweicloud.com/blogs/487448#1
- Microsoft Learn — OVER-satsen (Batch Mode and Indexing) - https://learn.microsoft.com/sv-se/SQL/t-sql/queries/select-over-clause-transact-sql?view=aps-pdw-2016#2

---

## 3. Performance Bottlenecks: Memory Spills (TempDB/Worktable Spilling)

### Definitions

**Core Definition:** Memory spills occur when the database runs out of allocated memory for sorting or window frame caching and writes intermediate data to disk (tempdb or worktable), causing severe performance degradation.

**Technical Definition:** When a Sort or Window Spool operator needs more memory than its Memory Grant allows, SQL Server spills rows to tempdb. This occurs when cardinality estimates are too low or when the window frame requires caching large amounts of data . PostgreSQL and other databases use `work_mem` for similar purposes; exceeding it triggers multi-pass merge sorts with disk I/O .

**Beginner-Friendly Explanation:** It's like trying to organize a huge pile of papers on a small desk. When the desk (memory) is full, you have to put some papers on the floor (disk) and keep shuffling between desk and floor. That's much slower than having a big enough desk.

### Purposes

- To identify when window functions are spilling to disk.
- To resolve spills by adjusting memory settings or query design.
- To reduce the risk of tempdb contention and I/O bottlenecks.
- To improve query response time from minutes to seconds.

### Syntax Rules and Structure

**Detecting Spills (SQL Server Extended Events):**

```sql
CREATE EVENT SESSION [Spill_Count] ON SERVER
ADD EVENT sqlserver.sort_warning,
ADD EVENT sqlserver.hash_warning,
ADD EVENT sqlserver.exchange_spill
ADD TARGET package0.event_counter;
```

**Memory Grant Feedback (SQL Server 2017+):**

```sql
-- Database compatibility level 140 or higher enables Memory Grant Feedback
ALTER DATABASE YourDatabase SET COMPATIBILITY_LEVEL = 150;
```

**PostgreSQL Work Memory:**

```sql
-- Increase work_mem for window function queries
SET work_mem = '256MB';
```

**Component Breakdown:**

| Setting/Tool | Purpose |
|--------------|---------|
| `sort_warning` | Extended Event fired when Sort spills  |
| `exchange_spill` | Extended Event fired when parallel exchange spills  |
| Memory Grant Feedback | Automatically adjusts memory grants on subsequent executions  |
| `work_mem` | PostgreSQL per-sort memory limit  |

**Syntax Rules:**

- Spills are detected via Extended Events (`sort_warning`, `hash_warning`, `exchange_spill`) .
- Memory Grant Feedback adjusts grants automatically when compatibility level is 140+ .
- PostgreSQL `work_mem` is per-operation, not total; concurrent sorts each get the limit.
- Statistics must be up to date for accurate cardinality estimates .

**Constraints and Limitations:**

- Increasing `work_mem` globally can cause memory pressure with concurrent queries .
- Memory Grant Feedback requires multiple executions to learn.
- Spills are more likely with wide rows (many columns) or large frames.

### Annotated Complete Code Examples

**Example 1: Detecting Spills with Extended Events**

```sql
CREATE EVENT SESSION [Spill_Count] ON SERVER
ADD EVENT sqlserver.sort_warning,
ADD EVENT sqlserver.hash_warning,
ADD EVENT sqlserver.exchange_spill
ADD TARGET package0.event_counter;

ALTER EVENT SESSION [Spill_Count] ON SERVER STATE = START;

-- Run the problematic query
SELECT CustomerID, OrderDate, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- Check spill counts
SELECT t.e.value('@name','sysname') AS [Event],
       t.e.value('@count','bigint') AS [Count]
FROM sys.dm_xe_sessions s
JOIN sys.dm_xe_session_targets st ON s.address = st.event_session_address
CROSS APPLY (SELECT CAST(st.target_data AS XML) AS TargetData) td
CROSS APPLY td.TargetData.nodes('//Event') AS t(e)
WHERE s.name = 'Spill_Count';
```

**Why this works:** The Extended Events session captures `sort_warning` events when the Sort operator spills. The `event_counter` target aggregates counts. If counts are non-zero, the query is spilling .

**Example 2: Resolving Spills with Index and Memory Grant Feedback**

```sql
-- Step 1: Create covering index to reduce sort data
CREATE NONCLUSTERED INDEX IX_Sales_Customer_OrderDate
ON Sales.SalesOrderHeader (CustomerID, OrderDate)
INCLUDE (TotalDue);

-- Step 2: Ensure compatibility level 150 for Memory Grant Feedback
ALTER DATABASE SalesDB SET COMPATIBILITY_LEVEL = 150;

-- Step 3: Run query multiple times; Memory Grant Feedback adjusts memory
SELECT CustomerID, OrderDate, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate
                           ROWS UNBOUNDED PRECEDING) AS RunningTotal
FROM Sales.SalesOrderHeader;
```

**Why this works:** The index eliminates the Sort for partitioning/ordering. Memory Grant Feedback learns from spill events and increases the grant on subsequent executions. Using `ROWS` instead of the default `RANGE` frame reduces window spool size .

### Real-World Cases

**Case 1: Monthly Financial Report**

A monthly report query spilled to tempdb, taking 15 minutes. After adding a covering index on `(AccountID, TransactionDate) INCLUDE (Amount)` and enabling Memory Grant Feedback, the query runs in 30 seconds.

**Case 2: Dashboard Timeout**

A dashboard query with `PARTITION BY` on a high-cardinality column caused massive sorting and spilling. Replacing the window function with a `GROUP BY` + `JOIN` approach eliminated the spill and improved response time .

### References

- SQL Server Extended Events — Spill Counting - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/4021000/e63b841cb330f0989282df2af21a6b6f#84#51
- Microsoft Learn — OVER Clause (Avoid Sort Spills, Memory Grant Feedback) - https://learn.microsoft.com/nl-nl/SQL/t-sql/queries/select-over-clause-transact-sql?view=aps-pdw-2016#2
- 华为云社区 — SQL Server中OVER子句的4个性能陷阱 - https://bbs.huaweicloud.com/blogs/487448#1
- 金仓数据库 — 窗口函数参数调优实战指南 - https://www.kingbase.com.cn/explore/tech-blog/金仓数据库窗口函数参数调优实战指南/#1

---

## 4. Combinatorial Windows: Multiple Distinct OVER Clauses

### Definitions

**Core Definition:** Combinatorial windows occur when a query contains multiple window functions with different `PARTITION BY` and/or `ORDER BY` specifications, potentially causing the database to sort and process the data multiple times.

**Technical Definition:** Each distinct window specification (`PARTITION BY` + `ORDER BY` combination) requires its own sort and `WindowAgg`/`Sequence Project` pipeline. PostgreSQL deduplicates `WindowAgg` nodes only when the window definitions are syntactically equivalent . MySQL processes windows with the same ordering requirements sequentially, skipping sort for subsequent windows . Using a `WINDOW` clause to name and reuse specifications reduces duplication and improves maintainability .

**Beginner-Friendly Explanation:** If you ask the database to sort your data by department for one calculation, then by year for another, it has to sort twice. Combinatorial windows are when you write multiple `OVER` clauses that force multiple sorts. The fix is to make the windows as similar as possible, or use named windows.

### Purposes

- To understand why multiple window functions with different orderings are expensive.
- To consolidate window specifications using the `WINDOW` clause.
- To reduce the number of sorts and passes over the data.
- To write maintainable queries with shared window definitions.

### Syntax Rules and Structure

**Problematic Query (Multiple Sorts):**

```sql
SELECT sum(salary) OVER (PARTITION BY dept ORDER BY salary DESC) AS sum_salary,
       avg(salary) OVER (PARTITION BY dept ORDER BY salary ASC) AS avg_salary
FROM empsalary;
```

**Optimized Query (Named Window):**

```sql
SELECT sum(salary) OVER w AS sum_salary,
       avg(salary) OVER w AS avg_salary
FROM empsalary
WINDOW w AS (PARTITION BY dept ORDER BY salary DESC);
```

**Component Breakdown:**

| Aspect | Multiple OVER Clauses | Named WINDOW Clause |
|--------|----------------------|---------------------|
| Sort operations | One per distinct ordering | Shared where possible |
| Maintainability | Duplicated definitions | Centralized definition |
| Optimizer deduplication | May deduplicate identical specs | Explicitly shared |

**Syntax Rules:**

- The `WINDOW` clause defines named windows after `FROM` and before `SELECT` (or after `GROUP BY`/`HAVING`) .
- Multiple `OVER` clauses can reference the same named window.
- PostgreSQL deduplicates `WindowAgg` nodes for syntactically equivalent specifications .
- MySQL processes windows with the same ordering requirements in sequence, skipping sort for later windows .
- SQL Server does not deduplicate automatically; the optimizer may or may not share sorts.

**Constraints and Limitations:**

- Different `ORDER BY` directions (`ASC` vs. `DESC`) require separate sorts.
- Different partition columns require separate sorts.
- A named window can only be referenced by functions with compatible frame requirements.
- MySQL does not merge windows in a single step; it processes them sequentially .

### Annotated Complete Code Examples

**Example 1: Multiple Sorts vs. Shared Sort**

```sql
-- Expensive: two different orderings
SELECT CustomerID, OrderDate, TotalDue,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS rn,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY TotalDue) AS running_total
FROM Sales.SalesOrderHeader;

-- Better: same ordering for both functions
SELECT CustomerID, OrderDate, TotalDue,
       ROW_NUMBER() OVER w AS rn,
       SUM(TotalDue) OVER w AS running_total
FROM Sales.SalesOrderHeader
WINDOW w AS (PARTITION BY CustomerID ORDER BY OrderDate);
```

**Why the second is better:** The first query orders by `OrderDate` for `ROW_NUMBER` and by `TotalDue` for `SUM`, requiring two sorts. The second uses a single ordering for both, so one sort suffices .

**Example 2: PostgreSQL WindowAgg Deduplication**

```sql
-- These two share the same partition and order
SELECT count(*) OVER (PARTITION BY j) AS count_j,
       sum(i) OVER (PARTITION BY j) AS sum_j
FROM values_table;
```

**Execution Plan:** A single `WindowAgg` node handles both functions because the window specifications are identical. PostgreSQL deduplicates during planning .

### Real-World Cases

**Case 1: Sales Dashboard**

A dashboard computes `ROW_NUMBER`, `RANK`, and `SUM` over the same `(Region, SalesDate)` window. Using a named `WINDOW` clause ensures one sort and one WindowAgg pipeline.

**Case 2: ETL Ranking**

An ETL pipeline ranks products by revenue and by quantity sold. Two different orderings require two sorts; the developer adds indexes for each ordering to avoid spilling.

### References

- PostgreSQL Documentation — Window Function Processing (WindowAgg deduplication) - https://www.postgresql.org/message-id/attachment/173563/v0-draft-0001-doc-more-thoroughly-explain-window-function-processing.patch
- MySQL Reference Manual — Window Function Optimization - https://downloads.mysql.com/docs/refman-8.0-ja.a4.pdf#1239#344
- AWS — Reducing Window Function Temporary Files - https://docs.aws.amazon.com/zh_tw/AmazonRDS/latest/UserGuide/rds-ug.pdf#604#507

---

## Summary Table: Window Function Optimization

| Optimization | Technique | Impact |
|-------------|-----------|--------|
| Index Design | POC rule: Partition, Order, Cover | Eliminates Sort  |
| Memory Management | Memory Grant Feedback, `work_mem` | Reduces spills  |
| Spill Detection | Extended Events (`sort_warning`, `exchange_spill`) | Identifies problem queries  |
| Window Consolidation | Named `WINDOW` clause | Reduces sorts  |
| Frame Choice | `ROWS` instead of default `RANGE` | Smaller window spool  |
| Batch Mode | Columnstore index or compatibility 150+ | Faster Window Aggregate  |

---

## References

- Microsoft Learn — OVER Clause (Performance, Indexing, Batch Mode) - https://learn.microsoft.com/pl-pl/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-linux-2017#2
- PostgreSQL Documentation — Window Function Processing - https://www.postgresql.org/message-id/attachment/173563/v0-draft-0001-doc-more-thoroughly-explain-window-function-processing.patch
- T-SQL Window Functions (Execution Plan Operators) - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/pilimi-zlib/ia20241105/annas_archive_data__aacid__ia2_acsmpdf_files__20240823T234744Z--20240823T234745Z/aacid__ia2_acsmpdf_files__20240823T234744Z__BqSKjLdpo2KsSyanJXBy7G#50#31
- 华为云社区 — SQL Server中OVER子句的4个性能陷阱 - https://bbs.huaweicloud.com/blogs/487448#1
- Microsoft Learn — OVER-satsen (Batch Mode and Indexing) - https://learn.microsoft.com/sv-se/SQL/t-sql/queries/select-over-clause-transact-sql?view=aps-pdw-2016#2
- SQL Server Extended Events — Spill Counting - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/4021000/e63b841cb330f0989282df2af21a6b6f#84#51
- Microsoft Learn — OVER Clause (Avoid Sort Spills, Memory Grant Feedback) - https://learn.microsoft.com/nl-nl/SQL/t-sql/queries/select-over-clause-transact-sql?view=aps-pdw-2016#2
- 金仓数据库 — 窗口函数参数调优实战指南 - https://www.kingbase.com.cn/explore/tech-blog/金仓数据库窗口函数参数调优实战指南/#1
- MySQL Reference Manual — Window Function Optimization - https://downloads.mysql.com/docs/refman-8.0-ja.a4.pdf#1239#344
- AWS — Reducing Window Function Temporary Files - https://docs.aws.amazon.com/zh_tw/AmazonRDS/latest/UserGuide/rds-ug.pdf#604#507
- Patent US20070078826A1 — Column Vector for Window Function Evaluation - https://patentimages.storage.googleapis.com/ba/0c/62/03e732186b65c2/US20070078826A1.pdf#6#3
- PostgreSQL Documentation — Window Functions (WINDOW clause) - https://git.postgresql.org/cgit/pgweb-static.git/plain/documentation/pdf/9.0/postgresql-9.0-US.pdf?id=7717fa21b41965b14f773d59bd002e47cbc5f34d#309#14
- SQL Server Execution Plans — Sequence Project and Segment - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2525000/5ecf30ddb442cc4d3d606c372cec8c38#56#17