# Materialized Views: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A materialized view is a database object that physically stores the precomputed result set of a query, enabling fast access to aggregated, joined, or otherwise complex data without re-executing the underlying query.

**Technical Definition:** A materialized view is a table segment whose contents are periodically refreshed based on the results of a stored query . Unlike standard views, which are virtual and recompute their results on every access, materialized views store data persistently on disk, consuming storage space and requiring maintenance to stay current with their base tables . Materialized views are also known as "snapshots" in some database systems and "summaries" in data warehousing contexts .

**Beginner-Friendly Explanation:** A materialized view is like a photocopy of a report that you make once and then reuse. Instead of running a slow, complex query every time you need the data, the database saves the results and gives them to you instantly. When the underlying data changes, you refresh the photocopy to bring it up to date. It is faster to read but may be slightly out of date.

### Key Characteristics

- **Physical Storage:** Materialized views store actual data on disk, unlike standard views which are virtual .
- **Precomputed Results:** They contain precomputed aggregates, joins, and filters, eliminating the need to process base tables repeatedly .
- **Refresh Required:** The data becomes stale when base tables change; refresh operations update the materialized view .
- **Query Rewrite:** Database optimizers can transparently redirect queries from base tables to materialized views without changing application code .
- **Storage Cost:** Materialized views consume disk space and require maintenance overhead .

### Prerequisites

- **CREATE MATERIALIZED VIEW Privilege:** The ability to create materialized views in the target schema.
- **Base Table Access:** SELECT privileges on all tables referenced in the defining query.
- **Refresh Privilege:** MAINTAIN privilege on the materialized view to execute REFRESH operations .
- **Understanding of Query Patterns:** Materialized views are most beneficial for predictable, repeated queries .

### Related Programming Areas

- **Data Warehousing:** Materialized views pre-compute expensive joins and aggregations on fact and dimension tables .
- **Business Intelligence and Reporting:** Dashboards and reports query materialized views for consistent, fast performance .
- **Query Optimization:** Query rewrite transparently uses materialized views to improve performance without modifying SQL .
- **Application Performance:** Frequently executed queries can be redirected to materialized views to reduce latency .

### Core Concepts / Features

1. Physical Result Storage
2. Refresh Strategies
3. Manual Refresh
4. Scheduled Refresh
5. Incremental Refresh (Where Supported)
6. Reporting Workloads

---

## 1. Physical Result Storage

### Definitions

**Core Definition:** Physical result storage means that a materialized view stores its query results as actual data on disk, rather than computing them on demand.

**Technical Definition:** When a materialized view is created, the database executes the defining query and stores the result set as a table segment. Subsequent queries against the materialized view read the stored data directly, bypassing the base tables entirely . This is fundamentally different from a standard view, which executes its query every time it is referenced .

**Beginner-Friendly Explanation:** A standard view is like a recipe you follow every time you want a meal. A materialized view is like a meal you cooked ahead of time and stored in the fridge. You can eat it instantly, but it might not be as fresh as cooking it from scratch.

### Purposes

- To eliminate repeated computation of expensive queries.
- To provide consistent, fast query response times regardless of base table size.
- To reduce CPU and I/O consumption for predictable workloads.
- To enable query rewrite optimization without application changes .

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE MATERIALIZED VIEW view_name AS
SELECT column_list
FROM table_name
WHERE condition
[WITH [NO] DATA];

REFRESH MATERIALIZED VIEW view_name;
REFRESH MATERIALIZED VIEW CONCURRENTLY view_name;
```

**Complete General Syntax (Oracle):**

```sql
CREATE MATERIALIZED VIEW view_name
[BUILD IMMEDIATE | BUILD DEFERRED]
[REFRESH FAST | COMPLETE | FORCE]
[ON COMMIT | ON DEMAND]
[ENABLE QUERY REWRITE]
AS
SELECT column_list
FROM table_name
WHERE condition;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `BUILD IMMEDIATE` | Populates the view immediately at creation  |
| `BUILD DEFERRED` | Defers population until the first refresh  |
| `WITH DATA` | Populates the view at creation (PostgreSQL default)  |
| `WITH NO DATA` | Creates the view definition without data  |
| `ENABLE QUERY REWRITE` | Allows the optimizer to use the view transparently  |

**Syntax Rules:**

- PostgreSQL: `WITH NO DATA` leaves the view in an unscannable state until refreshed .
- Oracle: `BUILD IMMEDIATE` executes the defining query and stores results at creation .
- Materialized views cannot contain subqueries in the SELECT list of the defining query .

**Constraints and Limitations:**

- Materialized views consume storage space proportional to the result set size .
- PostgreSQL materialized views cannot be temporary or unlogged .
- Oracle materialized views cannot contain LONG, LONG RAW, or virtual columns .

### Annotated Complete Code Examples

**Example 1: PostgreSQL Materialized View with Physical Storage**

```sql
-- Create a base table with sales data
CREATE TABLE sales (
    sale_id     SERIAL PRIMARY KEY,
    product_id  INT,
    sale_date   DATE,
    amount      NUMERIC(10,2)
);

-- Insert sample data
INSERT INTO sales (product_id, sale_date, amount) VALUES
    (100, '2026-01-15', 1500.00),
    (100, '2026-01-20', 2500.00),
    (101, '2026-01-15', 800.00);

-- Create a materialized view that stores aggregated sales data
CREATE MATERIALIZED VIEW mv_product_sales AS
SELECT 
    product_id,
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total_sales
FROM sales
GROUP BY product_id, DATE_TRUNC('month', sale_date)
WITH DATA;

-- Query the materialized view (reads stored data, not base table)
SELECT * FROM mv_product_sales WHERE product_id = 100;

-- Expected Output:
--  product_id |        month        | total_sales
-- ------------+---------------------+-------------
--         100 | 2026-01-01 00:00:00 |     4000.00
```

**Why this output occurs:** The materialized view executes the aggregation query once at creation and stores the result (4000.00 for product 100 in January 2026). When queried, the database reads directly from the stored data segment rather than scanning the `sales` table and re-computing the sum. PostgreSQL documentation confirms that `REFRESH MATERIALIZED VIEW` "completely replaces the contents of a materialized view" .

**Example 2: Oracle Materialized View with Physical Storage**

```sql
-- Create a materialized view that stores precomputed results
CREATE MATERIALIZED VIEW sales_mv
BUILD IMMEDIATE
REFRESH COMPLETE ON DEMAND
AS
SELECT 
    product_id,
    SUM(amount) AS total_sales
FROM sales
GROUP BY product_id;

-- Query the materialized view
SELECT * FROM sales_mv WHERE product_id = 100;
```

**Why this output occurs:** Oracle's `BUILD IMMEDIATE` executes the defining query and stores the results in the materialized view's table segment . The data persists until a refresh operation replaces it. Oracle documentation states that materialized views "contain actual data and consume storage space" .

### Real-World Cases

**Case 1: Data Warehouse Aggregation**

A data warehouse has a fact table with billions of transaction rows. A materialized view pre-computes daily sales totals by product and region. Dashboards query the materialized view, returning results in milliseconds instead of minutes .

**Case 2: Cross-Database Reporting**

A reporting system needs data from a remote, slow database. A materialized view is created locally with the remote query. The view is refreshed periodically, providing fast local access to remote data .

### References

- Amazon Redshift — Materialized View Overview - https://docs.aws.eu//redshift/latest/dg/materialized-view-overview.html
- Google BigQuery — Descripción general de las vistas lógicas y materializadas - https://docs.cloud.google.com/bigquery/docs/logical-materialized-view-overview
- PostgreSQL — REFRESH MATERIALIZED VIEW(7) - https://manpages.opensuse.org/Tumbleweed/postgresql17/REFRESH_MATERIALIZED_VIEW.7pg17.en.html
- Oracle Database Data Warehousing Guide — Refreshing Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/refreshing-materialized-views.html

---

## 2. Refresh Strategies

### Definitions

**Core Definition:** Refresh strategies are the methods by which a materialized view's stored data is updated to reflect changes in its base tables.

**Technical Definition:** A refresh strategy defines when and how a materialized view is updated. The two primary dimensions are: (1) refresh method — COMPLETE (full recomputation) versus FAST (incremental, where supported) versus FORCE (attempt FAST, fall back to COMPLETE), and (2) refresh timing — ON COMMIT (automatic, synchronous) versus ON DEMAND (manual or scheduled) .

**Beginner-Friendly Explanation:** Refresh strategies are like deciding when to update your photocopied report. You can re-run the whole report from scratch (complete refresh), or just update the pages that changed (fast refresh). You can do it automatically every time the source data changes (on commit), or manually when you decide (on demand).

### Purposes

- To control data staleness versus refresh cost.
- To balance automatic freshness against system resource consumption.
- To optimize refresh performance using incremental techniques where available.
- To align refresh timing with workload patterns and SLAs.

### Syntax Rules and Structure

**Refresh Method Comparison:**

| Method | Description | Requirements |
|--------|-------------|--------------|
| `COMPLETE` | Re-executes the defining query; replaces all data  | None |
| `FAST` | Incremental update; only changed rows  | Materialized view logs on base tables  |
| `FORCE` | Attempts FAST; falls back to COMPLETE if not possible | Materialized view logs (for FAST attempt) |

**Refresh Timing Comparison:**

| Timing | Description | Use Case |
|--------|-------------|----------|
| `ON COMMIT` | Refreshes automatically when base table changes are committed  | Near-real-time dashboards |
| `ON DEMAND` | Refreshes only when explicitly requested  | Batch reporting, scheduled ETL |

**Syntax Rules:**

- PostgreSQL supports only COMPLETE refresh; FAST/INCREMENTAL is not available .
- Oracle supports COMPLETE, FAST, and FORCE refresh methods .
- `ON COMMIT` requires the materialized view to be fast-refreshable for efficient operation.

**Constraints and Limitations:**

- Fast refresh requires materialized view logs, which add write overhead to base tables .
- PostgreSQL materialized views are always completely refreshed .
- `ON COMMIT` refresh can impact transaction commit latency.

### Annotated Complete Code Examples

**Example 1: Oracle Fast Refresh with Materialized View Log**

```sql
-- Create materialized view log on base table (enables fast refresh)
CREATE MATERIALIZED VIEW LOG ON sales
WITH ROWID, PRIMARY KEY, SEQUENCE (sale_date, amount, product_id)
INCLUDING NEW VALUES;

-- Create materialized view with FAST refresh ON COMMIT
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST ON COMMIT
AS
SELECT product_id, SUM(amount) AS total_sales
FROM sales
GROUP BY product_id;
```

**Why this output occurs:** The materialized view log records changes to the `sales` table . When a transaction commits changes to `sales`, Oracle automatically applies those changes to `sales_mv` using fast refresh. This provides near-real-time data with minimal refresh overhead. Oracle documentation confirms that materialized view logs enable fast refreshes .

**Example 2: PostgreSQL Complete Refresh**

```sql
-- PostgreSQL materialized view (complete refresh only)
CREATE MATERIALIZED VIEW mv_sales AS
SELECT product_id, SUM(amount) AS total_sales
FROM sales
GROUP BY product_id
WITH DATA;

-- Manual refresh (re-executes the full query)
REFRESH MATERIALIZED VIEW mv_sales;

-- Concurrent refresh (allows reads during refresh, requires unique index)
CREATE UNIQUE INDEX idx_mv_sales ON mv_sales (product_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales;
```

**Why this output occurs:** PostgreSQL only supports complete refresh, which "completely replaces the contents" of the materialized view . The `CONCURRENTLY` option allows reads during refresh but requires a unique index . Without `CONCURRENTLY`, the refresh takes an exclusive lock, blocking reads .

### Real-World Cases

**Case 1: Real-Time Dashboard (ON COMMIT Fast Refresh)**

A financial dashboard displays current account balances. Materialized views with `REFRESH FAST ON COMMIT` update immediately when transactions are committed, providing near-real-time data with minimal refresh overhead.

**Case 2: Nightly Batch Report (ON DEMAND Complete Refresh)**

A nightly sales report uses a materialized view with `REFRESH COMPLETE ON DEMAND`. The ETL process triggers the refresh after loading all daily transactions, ensuring the report reflects complete data.

### References

- Oracle Database — Fast Refresh - https://docs.oracle.com/cd/E96517_01/admin/database-administrators-guide.pdf
- Oracle Database Data Warehousing Guide — Refreshing Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/refreshing-materialized-views.html
- Oracle Database — CREATE MATERIALIZED VIEW LOG - https://docs.oracle.com/cd/F32587_01/sqlrf/sql-language-reference.pdf
- AWS DMS — Oracle and PostgreSQL materialized views - https://docs.aws.eu/dms/latest/oracle-to-aurora-postgresql-migration-playbook/chap-oracle-aurora-pg.special.matviews.html
- PostgreSQL — REFRESH MATERIALIZED VIEW(7) - https://manpages.opensuse.org/Tumbleweed/postgresql17/REFRESH_MATERIALIZED_VIEW.7pg17.en.html

---

## 3. Manual Refresh

### Definitions

**Core Definition:** Manual refresh is the explicit execution of a refresh command by a user or application to update a materialized view's stored data.

**Technical Definition:** Manual refresh is triggered by executing `REFRESH MATERIALIZED VIEW` (PostgreSQL) or `DBMS_MVIEW.REFRESH` (Oracle) or `REFRESH MATERIALIZED VIEW` (Amazon Redshift) outside of any automatic schedule. The refresh operation re-executes the defining query or applies incremental changes, depending on the configured refresh method .

**Beginner-Friendly Explanation:** Manual refresh is like updating your photocopied report whenever you decide it needs updating. You run a command, and the database refreshes the materialized view with current data.

### Purposes

- To refresh materialized views on an ad-hoc basis when data freshness is needed.
- To integrate refresh operations into ETL pipelines and application workflows.
- To refresh after specific business events (e.g., month-end close).
- To provide a fallback when scheduled refresh fails.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
REFRESH MATERIALIZED VIEW [CONCURRENTLY] view_name;
```

**Complete General Syntax (Amazon Redshift):**

```sql
REFRESH MATERIALIZED VIEW view_name;
```

**Complete General Syntax (Oracle, via PL/SQL):**

```sql
EXECUTE DBMS_MVIEW.REFRESH('view_name', 'method');
```

**Syntax Rules:**

- PostgreSQL: `REFRESH MATERIALIZED VIEW` requires MAINTAIN privilege .
- PostgreSQL: `CONCURRENTLY` requires a unique index and allows reads during refresh .
- Amazon Redshift: The refresh operation identifies changes in base tables and applies them to the materialized view .
- Oracle: `DBMS_MVIEW.REFRESH` accepts method parameters (`'C'` for complete, `'F'` for fast) .

**Constraints and Limitations:**

- PostgreSQL: Without `CONCURRENTLY`, the refresh takes an `ACCESS EXCLUSIVE` lock, blocking reads .
- PostgreSQL: `CONCURRENTLY` can only be used when the materialized view is already populated .
- PostgreSQL: Only one refresh at a time can run against a single materialized view .

### Annotated Complete Code Examples

**Example 1: PostgreSQL Manual Refresh**

```sql
-- Create and populate materialized view
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS total
FROM sales
GROUP BY DATE_TRUNC('month', sale_date)
WITH DATA;

-- Insert new data into base table
INSERT INTO sales (product_id, sale_date, amount) VALUES (102, '2026-02-01', 1200.00);

-- Query shows stale data (no February)
SELECT * FROM mv_monthly_sales;

-- Manual refresh
REFRESH MATERIALIZED VIEW mv_monthly_sales;

-- Query now shows February data
SELECT * FROM mv_monthly_sales;
```

**Why this output occurs:** After inserting new data, the materialized view still shows old data until refreshed. The `REFRESH MATERIALIZED VIEW` command re-executes the defining query and replaces the stored data. PostgreSQL documentation states that refresh "completely replaces the contents" of the view .

**Example 2: Amazon Redshift Manual Refresh**

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW tickets_mv AS
SELECT event_id, SUM(ticket_count) AS total_tickets
FROM tickets
GROUP BY event_id;

-- Manual refresh at any time
REFRESH MATERIALIZED VIEW tickets_mv;
```

**Why this output occurs:** Amazon Redshift's `REFRESH MATERIALIZED VIEW` statement "identifies changes that have taken place in the base table or tables, and then applies those changes to the materialized view" . This can be run manually whenever updated data is needed.

### Real-World Cases

**Case 1: Post-ETL Refresh**

An ETL pipeline loads new transaction data nightly. The final step of the pipeline executes `REFRESH MATERIALIZED VIEW` to update reporting views before the morning business cycle begins.

**Case 2: Ad-Hoc Reporting**

A business analyst needs current sales figures for an urgent meeting. They execute a manual refresh on the sales materialized view to ensure the latest data is available.

### References

- Amazon Redshift — Materialized View Overview - https://docs.aws.eu//redshift/latest/dg/materialized-view-overview.html
- PostgreSQL — REFRESH MATERIALIZED VIEW(7) - https://manpages.opensuse.org/Tumbleweed/postgresql17/REFRESH_MATERIALIZED_VIEW.7pg17.en.html
- PostgreSQL — Materialized views with concurrent refresh - https://www.postgresql.org/about/featurematrix/detail/materialized-views-with-concurrent-refresh/
- Oracle Database Data Warehousing Guide — Refreshing Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/refreshing-materialized-views.html

---

## 4. Scheduled Refresh

### Definitions

**Core Definition:** Scheduled refresh is the automatic update of a materialized view at predetermined intervals or according to a configured schedule.

**Technical Definition:** Scheduled refresh is implemented through database job schedulers, cron-like mechanisms, or built-in refresh intervals. Oracle supports `START WITH` and `NEXT` clauses to define refresh schedules . Amazon Redshift provides automatic refresh when base tables change, scheduled refresh via the Redshift scheduler, and manual refresh options .

**Beginner-Friendly Explanation:** Scheduled refresh is like setting an alarm to update your photocopied report at regular intervals—every hour, every night, or every week. You don't have to remember to do it; the database does it for you.

### Purposes

- To ensure materialized view data stays current without manual intervention.
- To align refresh timing with workload patterns (e.g., off-peak hours).
- To meet service level agreements (SLAs) for data freshness .
- To automate ETL and reporting workflows.

### Syntax Rules and Structure

**Complete General Syntax (Oracle):**

```sql
CREATE MATERIALIZED VIEW view_name
REFRESH COMPLETE
START WITH SYSDATE
NEXT SYSDATE + 1
AS
SELECT ...;
```

**Complete General Syntax (Amazon Redshift):**

```sql
-- Automatic refresh (default when base tables change)
CREATE MATERIALIZED VIEW view_name
AUTO REFRESH YES
AS SELECT ...;

-- Scheduled refresh via Redshift Scheduler API
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `START WITH` | Initial refresh date/time  |
| `NEXT` | Interval until next refresh  |
| `AUTO REFRESH YES` | Enables automatic refresh on base table changes  |

**Syntax Rules:**

- Oracle: If `START WITH` or `NEXT` is omitted, the previous schedule is retained .
- Amazon Redshift: Automatic refresh runs when cluster resources are available, minimizing disruption .
- BigQuery: Materialized views refresh automatically in the background when base tables change .

**Constraints and Limitations:**

- Scheduled refresh consumes system resources; timing should avoid peak workloads.
- PostgreSQL does not have built-in scheduling; external schedulers (cron, pg_cron) are required .
- Automatic refresh may be delayed if cluster resources are constrained .

### Annotated Complete Code Examples

**Example 1: Oracle Scheduled Refresh**

```sql
-- Create materialized view with weekly refresh schedule
CREATE MATERIALIZED VIEW sales_by_month_by_state
REFRESH COMPLETE
START WITH TRUNC(SYSDATE+1) + 9/24
NEXT SYSDATE+7
AS
SELECT state, calendar_month, SUM(amount) AS total_sales
FROM sales
GROUP BY state, calendar_month;

-- Modify the schedule later
ALTER MATERIALIZED VIEW sales_by_month_by_state
REFRESH NEXT SYSDATE+7;
```

**Why this output occurs:** The `START WITH TRUNC(SYSDATE+1) + 9/24` clause sets the first refresh to 9:00 AM tomorrow . The `NEXT SYSDATE+7` clause sets subsequent refreshes to occur weekly. Oracle documentation confirms that these clauses establish the automatic refresh interval .

**Example 2: Amazon Redshift Automatic Refresh**

```sql
-- Create materialized view with automatic refresh
CREATE MATERIALIZED VIEW mv_events AS
SELECT event_type, COUNT(*) AS event_count
FROM events
GROUP BY event_type;

-- Automatic refresh is enabled by default when base tables change
-- Refresh runs at a time when cluster resources are available
```

**Why this output occurs:** Amazon Redshift automatically refreshes materialized views when base tables are updated. The refresh "runs at a time when cluster resources are available to minimize disruptions to other workloads" . This provides near-real-time data without manual intervention.

### Real-World Cases

**Case 1: Daily Sales Dashboard**

A retail company's sales dashboard uses a materialized view refreshed nightly at 2:00 AM via Oracle's scheduled refresh. The view aggregates daily sales by region and product category, providing fast morning performance.

**Case 2: Streaming Analytics**

A real-time analytics platform uses BigQuery materialized views that refresh automatically in the background as streaming data lands. Queries against the materialized view receive fresher results than querying raw streaming tables .

### References

- Amazon Redshift — Materialized View Overview - https://docs.aws.eu//redshift/latest/dg/materialized-view-overview.html
- Google BigQuery — Descripción general de las vistas lógicas y materializadas - https://docs.cloud.google.com/bigquery/docs/logical-materialized-view-overview
- Oracle Database — ALTER MATERIALIZED VIEW - https://docs.oracle.com/cd/E15817_01/server.111/e05750/statements_2.htm
- AWS DMS — Oracle and PostgreSQL materialized views - https://docs.aws.eu/dms/latest/oracle-to-aurora-postgresql-migration-playbook/chap-oracle-aurora-pg.special.matviews.html

---

## 5. Incremental Refresh (Where Supported)

### Definitions

**Core Definition:** Incremental refresh (also called fast refresh) updates only the rows that have changed since the last refresh, rather than recomputing the entire result set.

**Technical Definition:** Incremental refresh leverages materialized view logs to track changes (inserts, updates, deletes) on base tables. During refresh, only the delta changes are applied to the materialized view, significantly reducing refresh time and resource consumption compared to complete refresh . This is supported in Oracle Database; PostgreSQL does **not** support incremental refresh .

**Beginner-Friendly Explanation:** Instead of re-running the entire report from scratch, incremental refresh just updates the parts that changed. If only 10 new rows were added to a million-row table, fast refresh only processes those 10 rows, not all million.

### Purposes

- To dramatically reduce refresh time for large materialized views.
- To minimize resource consumption during refresh operations.
- To enable more frequent refreshes without excessive overhead.
- To support near-real-time data freshness with `ON COMMIT` refresh .

### Syntax Rules and Structure

**Complete General Syntax (Oracle):**

```sql
-- Step 1: Create materialized view log on base table
CREATE MATERIALIZED VIEW LOG ON sales
WITH ROWID, PRIMARY KEY, SEQUENCE (sale_date, amount)
INCLUDING NEW VALUES;

-- Step 2: Create materialized view with FAST refresh
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST ON COMMIT
AS
SELECT product_id, SUM(amount) AS total
FROM sales
GROUP BY product_id;
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `MATERIALIZED VIEW LOG` | Tracks changes to base table  |
| `WITH ROWID, PRIMARY KEY, SEQUENCE` | Specifies what the log records  |
| `INCLUDING NEW VALUES` | Records both old and new values for updates |
| `REFRESH FAST` | Instructs incremental refresh |

**Syntax Rules:**

- Fast refresh requires a materialized view log on each master table .
- The materialized view log must exist before the materialized view is created or last refreshed .
- Fast refresh fails if the materialized view is not "fast refreshable" due to unsupported constructs.
- Oracle: `EXPLAIN_MVIEW` procedure can verify fast refresh capability .

**Constraints and Limitations:**

- **PostgreSQL does not support incremental refresh** .
- Materialized view logs add write overhead to base tables .
- Not all materialized view definitions are fast refreshable (e.g., certain joins, subqueries).
- Partition Change Tracking (PCT) enables fast refresh for partitioned tables under specific conditions .

### Annotated Complete Code Examples

**Example 1: Oracle Fast Refresh with Materialized View Log**

```sql
-- Create materialized view log to track changes
CREATE MATERIALIZED VIEW LOG ON times
WITH ROWID, SEQUENCE (time_id, calendar_year)
INCLUDING NEW VALUES;

CREATE MATERIALIZED VIEW LOG ON products
WITH ROWID, SEQUENCE (prod_id)
INCLUDING NEW VALUES;

-- Create fast-refreshable materialized view
CREATE MATERIALIZED VIEW sales_mv
REFRESH FAST ON COMMIT
AS
SELECT t.calendar_year, p.prod_id, SUM(s.amount_sold) AS sum_sales
FROM times t, products p, sales s
WHERE t.time_id = s.time_id AND p.prod_id = s.prod_id
GROUP BY t.calendar_year, p.prod_id;
```

**Why this output occurs:** The materialized view logs on `times` and `products` record changes to those tables . When a transaction commits changes, Oracle applies only the delta changes to `sales_mv` using fast refresh. This is significantly faster than complete refresh because only changed rows are processed .

**Example 2: Partition Change Tracking (PCT) Fast Refresh**

```sql
-- Materialized view with PCT support
CREATE MATERIALIZED VIEW cust_mth_sales_mv
BUILD IMMEDIATE
REFRESH FAST ON DEMAND
ENABLE QUERY REWRITE AS
SELECT s.cust_id, t.calendar_month_name, 
       SUM(s.quantity_sold) AS quantity_sold,
       COUNT(s.amount_sold) AS amount_sold
FROM sales s, times t
WHERE s.time_id = t.time_id
GROUP BY s.cust_id, t.calendar_month_name;

-- After partition maintenance operation, fast refresh uses PCT
EXECUTE DBMS_MVIEW.REFRESH('CUST_MTH_SALES_MV', 'F', '', TRUE, FALSE);
```

**Why this output occurs:** Oracle documentation describes PCT as enabling fast refresh after partition maintenance operations when the materialized view has sufficient information to track which partitions are stale . The `DBMS_MVIEW.REFRESH` with `'F'` (fast) automatically uses PCT if applicable .

### Real-World Cases

**Case 1: Real-Time Financial Dashboard**

A trading platform uses Oracle materialized views with `REFRESH FAST ON COMMIT` to keep account balance summaries current. Each transaction commit triggers an incremental refresh, adding only the new transaction amount to the affected account's total.

**Case 2: Large Data Warehouse Aggregation**

A data warehouse materialized view aggregates 10 billion transaction rows. A complete refresh takes hours. With fast refresh enabled via materialized view logs, the nightly refresh processes only the day's new transactions (millions of rows), completing in minutes .

### References

- Oracle Database — Fast Refresh - https://docs.oracle.com/cd/E96517_01/admin/database-administrators-guide.pdf
- Oracle Database Data Warehousing Guide — Refreshing Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/refreshing-materialized-views.html
- Oracle Database — CREATE MATERIALIZED VIEW LOG - https://docs.oracle.com/cd/F32587_01/sqlrf/sql-language-reference.pdf
- AWS DMS — Oracle and PostgreSQL materialized views - https://docs.aws.eu/dms/latest/oracle-to-aurora-postgresql-migration-playbook/chap-oracle-aurora-pg.special.matviews.html

---

## 6. Reporting Workloads

### Definitions

**Core Definition:** Reporting workloads are read-heavy, repetitive query patterns—typically involving aggregations, joins, and filters—that benefit significantly from materialized views.

**Technical Definition:** Reporting workloads are characterized by predictable, repeated queries against large datasets, often producing dashboards, summaries, and analytical reports. Materialized views precompute these expensive operations, enabling fast response times and reducing resource consumption on base tables .

**Beginner-Friendly Explanation:** Reporting is like asking the same questions over and over: "What were total sales last month?" "Which products are top sellers?" Materialized views answer these questions ahead of time so the answers are ready instantly.

### Purposes

- To accelerate dashboard and BI tool queries .
- To reduce query costs for frequently executed aggregations .
- To provide consistent performance regardless of base table growth .
- To enable query rewrite transparency without changing application SQL .

### Syntax Rules and Structure

**Typical Reporting Materialized View Pattern:**

```sql
CREATE MATERIALIZED VIEW reporting_view
REFRESH FAST ON DEMAND
ENABLE QUERY REWRITE
AS
SELECT 
    dimension1,
    dimension2,
    SUM(measure1) AS total_measure1,
    COUNT(*) AS record_count
FROM fact_table f
JOIN dimension_table d ON f.dim_id = d.dim_id
GROUP BY dimension1, dimension2;
```

**Syntax Rules:**

- Materialized views for reporting often include aggregates (`SUM`, `COUNT`, `AVG`) and joins.
- `ENABLE QUERY REWRITE` allows the optimizer to use the view even when queries reference base tables .
- For fast refresh, the defining query must satisfy fast-refresh restrictions .

**Constraints and Limitations:**

- Query rewrite requires the materialized view to be fresh or within acceptable staleness limits.
- Complex reporting queries with subqueries in the SELECT list may not be fast refreshable.
- Reporting materialized views should be refreshed after ETL loads to ensure data accuracy.

### Annotated Complete Code Examples

**Example 1: Reporting Materialized View with Query Rewrite**

```sql
-- Create a reporting materialized view
CREATE MATERIALIZED VIEW sales_report_mv
BUILD IMMEDIATE
REFRESH FAST ON DEMAND
ENABLE QUERY REWRITE AS
SELECT 
    p.prod_name,
    t.calendar_month_name,
    SUM(s.amount_sold) AS total_sales,
    COUNT(s.quantity_sold) AS units_sold
FROM sales s, products p, times t
WHERE s.prod_id = p.prod_id AND s.time_id = t.time_id
GROUP BY p.prod_name, t.calendar_month_name;

-- Application query (does not reference materialized view)
SELECT p.prod_name, t.calendar_month_name, SUM(s.amount_sold)
FROM sales s, products p, times t
WHERE s.prod_id = p.prod_id AND s.time_id = t.time_id
  AND t.calendar_year = 2026
GROUP BY p.prod_name, t.calendar_month_name;

-- Query rewrite transparently uses sales_report_mv
```

**Why this output occurs:** Oracle's query rewrite feature "transforms a user request written in terms of master tables into a semantically equivalent request that includes materialized views" . The application query remains unchanged; the optimizer automatically uses `sales_report_mv` because it can answer the query more efficiently .

**Example 2: BigQuery Reporting Materialized View**

```sql
-- Create materialized view for dashboard acceleration
CREATE MATERIALIZED VIEW `project.dataset.daily_active_users`
AS
SELECT
  DATE(timestamp) AS activity_date,
  COUNT(DISTINCT user_id) AS daily_active_users
FROM `project.dataset.events`
GROUP BY activity_date;

-- Dashboard query (automatically rewritten)
SELECT * FROM `project.dataset.daily_active_users`
WHERE activity_date >= '2026-01-01';
```

**Why this output occurs:** BigQuery materialized views "store the results of a query" and "allow BigQuery to deliver results faster without repeatedly processing raw base data" . The query optimizer can redirect queries from base tables to the materialized view when beneficial .

### Real-World Cases

**Case 1: Executive Dashboard**

A company's executive dashboard displays KPIs like daily active users, monthly revenue, and conversion rates. Materialized views precompute these metrics, enabling sub-second dashboard load times even as the underlying data grows to billions of rows .

**Case 2: Financial Month-End Close**

A financial system runs numerous reports during month-end close: revenue by product, expenses by department, profit margins. Materialized views are refreshed after the final transactions post, then all reports query the precomputed views, completing in minutes instead of hours.

**Case 3: Query Cost Reduction**

A SaaS company's analytics platform runs thousands of identical aggregation queries daily. BigQuery materialized views reduce processing costs by serving precomputed results, with automatic refresh ensuring freshness .

### References

- Amazon Redshift — Materialized View Overview - https://docs.aws.eu//redshift/latest/dg/materialized-view-overview.html
- Google BigQuery — Descripción general de las vistas lógicas y materializadas - https://docs.cloud.google.com/bigquery/docs/logical-materialized-view-overview
- Oracle Database — Automatic Materialized Views and Query Rewrite - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/database-concepts.pdf
- Oracle Database — Query Rewrite - https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/partitions-views-and-other-schema-objects.html

---

## Summary Table

| Concept | Key Characteristic | PostgreSQL Support | Oracle Support | Amazon Redshift Support |
|---------|-------------------|-------------------|----------------|-------------------------|
| Physical Result Storage | Stores data on disk | Yes | Yes | Yes |
| Refresh Strategies | COMPLETE/FAST/FORCE | COMPLETE only | COMPLETE, FAST, FORCE | COMPLETE (incremental for Iceberg) |
| Manual Refresh | Explicit command | `REFRESH MATERIALIZED VIEW` | `DBMS_MVIEW.REFRESH` | `REFRESH MATERIALIZED VIEW` |
| Scheduled Refresh | Automatic intervals | External scheduler required | `START WITH`/`NEXT` | Auto refresh, Redshift Scheduler |
| Incremental Refresh | Only changed rows | **Not supported** | Supported (with logs) | Supported (Iceberg) |
| Reporting Workloads | Dashboard/BI acceleration | Yes | Yes (with query rewrite) | Yes (with query rewrite) |

---

## References

- Amazon Redshift — Materialized View Overview - https://docs.aws.eu//redshift/latest/dg/materialized-view-overview.html
- Google BigQuery — Descripción general de las vistas lógicas y materializadas - https://docs.cloud.google.com/bigquery/docs/logical-materialized-view-overview
- PostgreSQL — REFRESH MATERIALIZED VIEW(7) - https://manpages.opensuse.org/Tumbleweed/postgresql17/REFRESH_MATERIALIZED_VIEW.7pg17.en.html
- PostgreSQL — Materialized views with concurrent refresh - https://www.postgresql.org/about/featurematrix/detail/materialized-views-with-concurrent-refresh/
- Oracle Database — Fast Refresh - https://docs.oracle.com/cd/E96517_01/admin/database-administrators-guide.pdf
- Oracle Database Data Warehousing Guide — Refreshing Materialized Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/dwhsg/refreshing-materialized-views.html
- Oracle Database — Automatic Materialized Views and Query Rewrite - https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/database-concepts.pdf
- Oracle Database — CREATE MATERIALIZED VIEW LOG - https://docs.oracle.com/cd/F32587_01/sqlrf/sql-language-reference.pdf
- Oracle Database — ALTER MATERIALIZED VIEW - https://docs.oracle.com/cd/E15817_01/server.111/e05750/statements_2.htm
- AWS DMS — Oracle and PostgreSQL materialized views - https://docs.aws.eu/dms/latest/oracle-to-aurora-postgresql-migration-playbook/chap-oracle-aurora-pg.special.matviews.html
- Oracle Database — Query Rewrite - https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/partitions-views-and-other-schema-objects.html