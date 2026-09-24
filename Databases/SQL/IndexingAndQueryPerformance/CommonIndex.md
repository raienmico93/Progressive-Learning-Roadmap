# SQL Common Index Structures: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL index structures are specialised data structures that organise table data to enable fast retrieval of rows matching a query predicate, with each structure optimised for different data types, query patterns, and workload characteristics.

**Technical Definition:** An index structure is a disk-resident or memory-resident auxiliary data structure that maintains a mapping from key values to row identifiers, supporting efficient search, insertion, and deletion operations. Common structures include B-trees (balanced tree structures for ordered data), hash tables (direct-address structures for equality predicates), bitmaps (bit-vector representations for low-cardinality data), LSM trees (log-structured merge trees for write-optimised workloads), inverted indexes (term-to-document mappings for full-text search), R-trees (hierarchical bounding-box structures for spatial data), and expression indexes (indexes on computed values).

**Beginner-Friendly Explanation:** Think of index structures as different types of filing systems. A B-tree is like a well-organised filing cabinet with labelled drawers and folders. A hash index is like a coat check—you give a specific ticket (the key) and get your item back instantly. A bitmap index is like a checklist where each box represents whether a row has a certain value. An LSM tree is like a inbox where you write new entries quickly and sort them later. Different situations call for different filing systems.

### Key Characteristics

- **Algorithm-specific:** Each structure uses a distinct algorithm optimised for specific operations and data characteristics .
- **Workload-sensitive:** Some structures favour read-heavy workloads (B-tree, bitmap), others write-heavy workloads (LSM tree) .
- **Data-type-specific:** Spatial indexes handle geometric data, full-text indexes handle natural language, JSON indexes handle semi-structured documents.
- **RDBMS-specific:** Not all index structures are available in every database system; PostgreSQL, MySQL, SQL Server, and Oracle each support different subsets .

### Prerequisites

- **Basic SQL knowledge:** Understanding of `SELECT`, `WHERE`, and `JOIN` operations.
- **Data type concepts:** Familiarity with scalar, spatial, textual, and semi-structured data types.
- **Storage concepts:** Awareness of disk I/O versus memory access, and read versus write workloads.

### Related Programming Areas

- **Database Administration:** Index selection and tuning are core DBA responsibilities.
- **Query Optimization:** The optimizer chooses index structures based on cost estimates.
- **Geographic Information Systems (GIS):** Spatial indexes are fundamental to GIS applications.
- **Search Engineering:** Full-text and inverted indexes power search engines and document retrieval systems.
- **Big Data and NoSQL:** LSM trees are the storage backbone of many distributed databases .

### Core Concepts / Features

The following core concepts are covered:

1. B-tree / B+tree Indexes
2. Hash Indexes
3. Bitmap Indexes
4. LSM Trees (Log-Structured Merge)
5. Specialised Indexes
   - Full-Text Indexes
   - Spatial Indexes (R-trees)
   - JSON / Inverted Indexes
   - Expression-Based / Functional Indexes

---

## 1. B-tree / B+tree Indexes

### Definitions

**Core Definition:** A B-tree (Balanced Tree) index is a self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in logarithmic time.

**Technical Definition:** A B-tree of order *m* is a tree where each node contains at most *m* children and *m*−1 keys. All leaves are at the same depth (balanced). B+trees, the variant used by most database systems, store all data pointers in leaf nodes and link leaves sequentially, making range scans efficient . SQL Server documentation notes that while the term "B-tree" is used generically, the Database Engine actually implements a B+ tree for rowstore indexes .

**Beginner-Friendly Explanation:** A B-tree is like a perfectly organised library catalogue. Every book is exactly the same number of steps away from the entrance, and the catalogue is sorted alphabetically. Whether you want one specific book or every book from A to C, the system is equally efficient.

### Purposes

- To accelerate equality and range queries on sortable data.
- To retrieve rows in sorted order without a separate sort operation.
- To serve as the default index structure for most relational database systems .
- To enforce uniqueness constraints through unique B-tree indexes.

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name ON table_name USING btree (column_name [ASC | DESC] [NULLS { FIRST | LAST }]);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `USING btree` | Specifies B-tree index method (default in most systems) |
| `column_name` | The column(s) to index |
| `ASC \| DESC` | Sort order of key values |
| `NULLS FIRST \| NULLS LAST` | Position of NULL values in the index |

**Syntax Rules:**

- B-tree indexes support the operators `=`, `<`, `<=`, `>`, `>=`, and `BETWEEN` .
- Pattern-matching operators `LIKE` and `~` can use B-tree indexes if the pattern is anchored to the beginning of the string (e.g., `LIKE 'foo%'`) .
- Composite B-tree indexes are ordered by the first column, then the second, and so on; the leading column is most critical for query matching .

**Constraints and Limitations:**

- B-tree indexes are less effective for columns with very low cardinality (few distinct values) .
- Index maintenance (page splits) occurs on every `INSERT`, `UPDATE`, or `DELETE`, adding write overhead.
- Very wide indexes (many columns or large data types) increase storage requirements.

### Annotated Complete Code Examples

**Example 1: Creating and Using a B-tree Index**

```sql
-- Create a table with an indexed column
CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    category     VARCHAR(50),
    price        NUMERIC(10, 2)
);

-- Insert sample data
INSERT INTO products (product_name, category, price) VALUES
    ('Laptop', 'Electronics', 999.99),
    ('Mouse', 'Electronics', 29.99),
    ('Desk', 'Furniture', 299.99),
    ('Chair', 'Furniture', 199.99);

-- Create a B-tree index on price
CREATE INDEX idx_products_price ON products USING btree (price);

-- Range query using the index
SELECT product_name, price
FROM products
WHERE price BETWEEN 100 AND 500;

-- Expected Output:
--  product_name | price
-- --------------+--------
--  Chair        | 199.99
--  Desk         | 299.99
```

**Why this output occurs:** The B-tree index on `price` stores values in sorted order. The `BETWEEN` operator triggers a range scan that reads only the leaf nodes containing values from 100 to 500, skipping Laptop (999.99) and Mouse (29.99). The result is returned in sorted order without an additional sort operation.

**Example 2: B-tree for Ordered Retrieval**

```sql
-- Retrieve products sorted by name using the index
CREATE INDEX idx_products_name ON products (product_name);

SELECT product_name, category
FROM products
ORDER BY product_name;

-- Expected Output:
--  product_name | category
-- --------------+------------
--  Chair        | Furniture
--  Desk         | Furniture
--  Laptop       | Electronics
--  Mouse        | Electronics
```

**Why this output occurs:** The B-tree index on `product_name` maintains sorted order. The `ORDER BY` clause can be satisfied by scanning the index in order, avoiding a sort operation. PostgreSQL documentation notes that B-tree indexes "can also be used to retrieve data in sorted order" and that this "is often helpful" .

### Real-World Cases

**Case 1: Primary Key Lookups**

Every relational database uses a B-tree index on primary key columns. A query like `SELECT * FROM users WHERE user_id = 12345` traverses the B-tree in logarithmic time, retrieving the row in milliseconds even in tables with billions of rows.

**Case 2: Date Range Reporting**

A financial system indexes the `transaction_date` column. Month-end reports query `WHERE transaction_date BETWEEN '2026-03-01' AND '2026-03-31'`, using a B-tree range scan to read only one month of data from a multi-year table.

### References

- PostgreSQL Documentation — Index Types - https://www.postgresql.org/docs/10/indexes-types.html
- Microsoft SQL Server — Indexes Overview - https://learn.microsoft.com/fil-ph/sql/relational-databases/indexes/indexes?view=azuresqldb-current
- Oracle Database — Managing Indexes - https://docs.oracle.com/cd/B16341_01/doc/server.102/b14196/schema003.htm

---

## 2. Hash Indexes

### Definitions

**Core Definition:** A hash index uses a hash function to map key values to bucket locations, providing O(1) average-case lookup time for equality comparisons.

**Technical Definition:** A hash index applies a hash function to the indexed key, storing the resulting hash value and a pointer to the row. Lookups compute the hash of the search key and examine the corresponding bucket. Hash indexes support only equality predicates (`=`); they cannot support range queries because hash values are not ordered .

**Beginner-Friendly Explanation:** A hash index is like a coat check system. When you check your coat, you get a ticket with a number. To retrieve it, you give the number back and they find it instantly. But you cannot ask for "all coats between number 100 and 200"—you need a specific ticket.

### Purposes

- To provide extremely fast equality lookups on high-cardinality columns.
- To reduce index size compared to B-tree indexes for equality-only workloads.
- To support memory-optimised tables where hash indexes are the native structure .

### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name ON table_name USING hash (column_name);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `USING hash` | Specifies hash index method |
| `column_name` | The column to index (single column only) |

**Syntax Rules:**

- Hash indexes support only the `=` operator .
- Hash indexes cannot be used for `ORDER BY`, range queries, or multi-column indexes.
- In SQL Server, hash indexes are available only on memory-optimised tables .

**Constraints and Limitations:**

- Hash indexes are not crash-safe in some older implementations (PostgreSQL hash indexes were WAL-logged starting in version 10).
- Hash indexes cannot enforce uniqueness or support `NULL` searches in the same way as B-trees.
- The PostgreSQL documentation historically discouraged hash indexes, though they are now fully supported .

### Annotated Complete Code Examples

**Example 1: Creating and Using a Hash Index**

```sql
-- Create a hash index on the product name column
CREATE INDEX idx_products_name_hash ON products USING hash (product_name);

-- Equality lookup using the hash index
SELECT product_name, price
FROM products
WHERE product_name = 'Laptop';

-- Expected Output:
--  product_name | price
-- --------------+--------
--  Laptop       | 999.99
```

**Why this output occurs:** The hash index computes the hash of `'Laptop'` and directly locates the bucket containing the row pointer. The lookup does not compare multiple key values (as a B-tree would); it computes one hash and retrieves the result. This is why hash indexes can be faster than B-trees for exact-match queries on high-cardinality columns.

### Real-World Cases

**Case 1: Session Token Lookup**

A web application stores session tokens in a table and indexes the `session_token` column with a hash index. Authentication middleware performs `SELECT user_id FROM sessions WHERE session_token = ?` on every request, achieving sub-millisecond response times.

**Case 2: Memory-Optimised Tables**

SQL Server memory-optimised tables use hash indexes for primary key lookups, benefiting from in-memory bucket arrays and lock-free concurrency .

### References

- PostgreSQL Documentation — Index Types (Hash) - https://www.postgresql.org/docs/10/indexes-types.html
- Microsoft SQL Server — Indexes Overview (Hash) - https://learn.microsoft.com/fil-ph/sql/relational-databases/indexes/indexes?view=azuresqldb-current

---

## 3. Bitmap Indexes

### Definitions

**Core Definition:** A bitmap index stores a bit vector for each distinct value in a column, where each bit indicates whether a particular row contains that value.

**Technical Definition:** A bitmap index maintains one bitmap (bit string) per distinct key value. For a table with *n* rows and *k* distinct values, the index consists of *k* bitmaps of length *n*. Bit position *i* is set to 1 in the bitmap for value *v* if row *i* contains value *v*. Bitwise operations (AND, OR, NOT) efficiently combine multiple bitmap indexes for complex predicates .

**Beginner-Friendly Explanation:** A bitmap index is like a set of checklists. For a "Status" column with values Active, Inactive, and Pending, there are three checklists. If row 5 is Active, the Active checklist has a checkmark in position 5. To find all Active rows, you just read the Active checklist. To find Active rows in a specific region, you AND the Active checklist with the Region checklist.

### Purposes

- To efficiently index low-cardinality columns (few distinct values).
- To accelerate complex queries involving multiple low-cardinality predicates.
- To reduce storage for columns where B-tree indexes would be large and inefficient .
- To support data warehouse and analytical workloads where reads dominate .

### Syntax Rules and Structure

**Complete General Syntax (Oracle):**

```sql
CREATE BITMAP INDEX index_name ON table_name (column_name);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `BITMAP` | Specifies bitmap index type |
| `column_name` | The column(s) to index (typically low cardinality) |

**Syntax Rules:**

- Bitmap indexes are designed for columns with low cardinality (e.g., gender, status, region) .
- Bitmap indexes can be combined with bitwise operations to satisfy complex predicates.
- Oracle documentation states that bitmap indexes are "more compact than a B-tree" and "can perform some types of retrieval more efficiently" .

**Constraints and Limitations:**

- Bitmap indexes require significant overhead during `INSERT`, `UPDATE`, and `DELETE` operations, making them unsuitable for OLTP systems .
- Oracle documentation explicitly recommends bitmap indexes "primarily for data warehouse environments" .
- Concurrent DML on bitmap-indexed tables can cause locking contention.

### Annotated Complete Code Examples

**Example 1: Creating a Bitmap Index**

```sql
-- Create a table with low-cardinality columns
CREATE TABLE sales (
    sale_id     NUMBER PRIMARY KEY,
    region      VARCHAR2(20),
    product     VARCHAR2(20),
    status      VARCHAR2(10)
);

-- Create bitmap indexes on region and status
CREATE BITMAP INDEX idx_sales_region ON sales (region);
CREATE BITMAP INDEX idx_sales_status ON sales (status);

-- Query using multiple bitmap indexes
SELECT COUNT(*)
FROM sales
WHERE region = 'North' AND status = 'Completed';
```

**Why this output occurs:** The optimizer combines the bitmap for `region = 'North'` with the bitmap for `status = 'Completed'` using a bitwise AND operation. Only rows where both bits are set are counted. This is far more efficient than scanning the table or using multiple B-tree lookups for low-cardinality predicates.

### Real-World Cases

**Case 1: Data Warehouse Star Schema**

A retail data warehouse has a fact table with foreign keys to dimension tables. Bitmap indexes on low-cardinality dimension columns (e.g., `store_type`, `promotion_type`) enable fast slice-and-dice analytical queries.

**Case 2: Boolean Column Filtering**

A table with a boolean `is_active` column (only two values) is indexed with a bitmap index. Queries filtering on `is_active = TRUE` use the bitmap efficiently, whereas a B-tree index on such a low-cardinality column would be largely ineffective.

### References

- Oracle Database — Managing Indexes (Bitmap) - https://docs.oracle.com/cd/B16341_01/doc/server.102/b14196/schema003.htm

---

## 4. LSM Trees (Log-Structured Merge)

### Definitions

**Core Definition:** An LSM tree (Log-Structured Merge tree) is a write-optimised data structure that buffers writes in memory and periodically merges them into sorted, immutable files on disk.

**Technical Definition:** An LSM tree consists of a memory-resident component (memtable) and one or more disk-resident sorted string tables (SSTables). Writes are first recorded in a write-ahead log (WAL) for durability, then inserted into the memtable. When the memtable reaches a threshold, it is flushed to disk as a new SSTable. Background compaction processes merge SSTables to maintain read efficiency .

**Beginner-Friendly Explanation:** An LSM tree is like an inbox tray. New papers go on top (memory) for fast access. When the tray fills up, you sort the papers and file them in a drawer (disk). Periodically, you merge and reorganise the drawers to keep everything tidy. This makes adding new papers very fast, though finding an old paper might require checking the tray and several drawers.

### Purposes

- To provide high write throughput by batching and sequentially writing data.
- To optimise for SSD storage, where sequential writes are much faster than random writes.
- To support distributed databases that require high ingestion rates .
- To reduce write amplification compared to B-tree page splits.

### Syntax Rules and Structure

LSM trees are an internal storage engine structure and are not typically created via SQL DDL. They are configured at the database or table level.

**Conceptual Structure:**

| Component | Purpose |
|-----------|---------|
| Memtable | In-memory sorted buffer for recent writes |
| WAL | Write-ahead log for crash recovery |
| SSTable | Immutable sorted file on disk |
| Compaction | Background process merging SSTables |

**Constraints and Limitations:**

- LSM trees optimise for writes at the cost of read amplification (a read may need to check multiple SSTables).
- Compaction consumes background I/O and CPU resources.
- Range scans may be less efficient than B-trees due to data being spread across multiple levels .

### Annotated Complete Code Examples

LSM trees are not exposed via user-facing SQL syntax. However, their behavior can be illustrated conceptually:

**Conceptual Write Path:**

```sql
-- When you execute an INSERT on an LSM-backed table:
INSERT INTO users (user_id, name) VALUES (1001, 'Alice');

-- The storage engine:
-- 1. Appends the change to the WAL (for durability)
-- 2. Inserts (1001, 'Alice') into the in-memory memtable (sorted)
-- 3. When memtable is full, flushes it to disk as an SSTable
-- 4. Background compaction merges SSTables over time
```

**Why this matters:** The `INSERT` returns quickly because it only writes to memory and the WAL (sequential append). The data is not immediately sorted into its final on-disk position; that happens during compaction. This design is why LSM trees achieve high write throughput .

### Real-World Cases

**Case 1: Google Spanner**

Google Spanner uses an LSM tree called Ressi for its storage layer, designed for "handling SQL queries over large-scale distributed databases with a mix of OLTP and OLAP workloads" .

**Case 2: NoSQL Databases**

Apache HBase, Apache Cassandra, and RocksDB (used by MySQL and MongoDB) all use LSM trees as their storage engine for high write throughput.

### References

- Google Spanner Ressi LSM Tree - https://pdfs.semanticscholar.org/fc57/e5d5790ac461bfabe89975cb1ea10e2ec30f.pdf
- Distributed Databases Using Many-Core Processors (LSM Tree) - https://FreePatentsOnline.com/y2014/0280375.html

---

## 5. Specialised Indexes

### 5.1 Full-Text Indexes

#### Definitions

**Core Definition:** A full-text index is an inverted index that maps keywords (tokens) to the documents containing them, enabling efficient natural language searches.

**Technical Definition:** A full-text index is a "token-based functional index" that inverts the document-term relationship: instead of storing terms per document, it stores a mapping from each term to the list of documents (and positions) where it occurs . The index is built by a word breaker that extracts tokens, a stemmer that normalises words, and a filter that removes stopwords (common words like "and", "the") .

**Beginner-Friendly Explanation:** A full-text index is like the index at the back of a textbook—it tells you every page where a particular word appears. Instead of searching every page for "database", you look up "database" in the index and immediately see all the pages that mention it.

#### Purposes

- To enable fast keyword and phrase searches in large text columns.
- To support linguistic features like stemming, stopword removal, and relevance ranking.
- To power search functionality in content management systems, document repositories, and e-commerce sites.

#### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE FULLTEXT INDEX ON table_name (column_name [LANGUAGE language_term])
KEY INDEX index_name
[WITH (CHANGE_TRACKING = { AUTO | MANUAL | OFF })];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ON table_name (column_name)` | The table and text column to index |
| `KEY INDEX index_name` | The unique index that identifies rows |
| `CHANGE_TRACKING` | How the index stays updated as data changes |

**Syntax Rules:**

- A full-text index requires a unique key index on the table to map index entries to rows .
- The index is maintained in fragments; reorganising merges fragments and removes obsolete entries .
- Queries use `CONTAINS` and `FREETEXT` predicates rather than standard comparison operators.

**Constraints and Limitations:**

- Full-text indexes have a unique update model; too many fragments degrade query performance .
- Stopword removal can cause unexpected results (e.g., searching for "and" returns nothing).
- Language-specific stemming may not suit all use cases.

#### Annotated Complete Code Examples

**Example 1: Creating and Using a Full-Text Index (SQL Server)**

```sql
-- Assume a Documents table with a unique index on DocumentID
-- and a Title column containing text

-- Create a full-text catalog
CREATE FULLTEXT CATALOG ft_catalog AS DEFAULT;

-- Create a full-text index on the Title column
CREATE FULLTEXT INDEX ON Documents (Title)
KEY INDEX PK_Documents
WITH (CHANGE_TRACKING = AUTO);

-- Query using CONTAINS
SELECT DocumentID, Title
FROM Documents
WHERE CONTAINS(Title, 'reflector');

-- Expected Output (conceptual):
--  DocumentID | Title
-- ------------+----------------------
--  3          | Rear Reflector Assembly
--  7          | Front Reflector Kit
```

**Why this output occurs:** The full-text index has inverted the Title column: the term "reflector" maps to DocumentIDs 3 and 7 (and any others containing that word). The `CONTAINS` predicate uses this inverted index to find matching documents without scanning the full text of every row .

#### Real-World Cases

**Case 1: E-Commerce Product Search**

An online store uses full-text indexes on product names and descriptions to power site search. A query for "wireless headphones" matches products containing those keywords, with relevance ranking based on term frequency.

**Case 2: Document Management Systems**

A legal document repository indexes the full text of contracts and briefs. Attorneys search for phrases like "force majeure" and receive all relevant documents ranked by relevance.

#### References

- Microsoft SQL Server — Full-Text Index Structure - https://learn.microsoft.com/hi-in/previous-versions/sql/sql-server-2008-r2/ms142505(v=sql.105)
- Microsoft SQL Server — Indexes Overview (Full-text) - https://learn.microsoft.com/fil-ph/sql/relational-databases/indexes/indexes?view=azuresqldb-current

---

### 5.2 Spatial Indexes (R-trees)

#### Definitions

**Core Definition:** A spatial index (typically an R-tree) organises multidimensional geometric data using hierarchical bounding boxes, enabling efficient spatial queries such as intersection, containment, and proximity searches.

**Technical Definition:** An R-tree is a height-balanced tree where each node contains a minimum bounding rectangle (MBR) that encloses all child nodes. Leaf nodes contain entries for spatial objects and their MBRs. Queries traverse the tree, pruning branches whose MBRs do not intersect the search region .

**Beginner-Friendly Explanation:** A spatial index is like a map with progressively finer grid overlays. To find all restaurants in a city, you first look at the city-level grid, then zoom into the relevant neighbourhoods, then into individual blocks. You never examine areas that cannot contain restaurants.

#### Purposes

- To accelerate spatial queries on geographic and geometric data.
- To support nearest-neighbour searches and spatial joins.
- To reduce I/O for location-based queries in GIS, CAD, and mapping applications .

#### Syntax Rules and Structure

**Complete General Syntax (SQL Server):**

```sql
CREATE SPATIAL INDEX index_name ON table_name (geometry_column)
USING GEOMETRY_GRID
WITH (BOUNDING_BOX = (xmin, ymin, xmax, ymax));
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `SPATIAL` | Specifies spatial index type |
| `geometry_column` | The spatial data column |
| `USING GEOMETRY_GRID` | Tiling scheme for the index |
| `BOUNDING_BOX` | Extent of the indexed space |

**Syntax Rules:**

- Spatial indexes require a bounding box that encloses all indexed geometries.
- The index is built on a column of a spatial data type (`geometry`, `geography`, or `SDO_GEOMETRY`).
- Spatial predicates like `STIntersects`, `STContains`, and `STDistance` trigger index usage.

**Constraints and Limitations:**

- Spatial indexes require periodic maintenance for optimal performance .
- The bounding box must be set correctly at creation; geometries outside it are not indexed.
- R-tree performance degrades with high-dimensional data.

#### Annotated Complete Code Examples

**Example 1: Creating and Using a Spatial Index (SQL Server)**

```sql
-- Create a table with a geometry column
CREATE TABLE Places (
    PlaceID   INT PRIMARY KEY,
    Name      VARCHAR(100),
    Location  GEOMETRY
);

-- Create a spatial index
CREATE SPATIAL INDEX idx_places_location
ON Places (Location)
USING GEOMETRY_GRID
WITH (
    BOUNDING_BOX = (-180, -90, 180, 90)
);

-- Query for places within a bounding box
SELECT Name
FROM Places
WHERE Location.STIntersects(
    geometry::STGeomFromText('POLYGON((0 0, 10 0, 10 10, 0 10, 0 0))', 4326)
) = 1;
```

**Why this output occurs:** The spatial index on `Location` allows the query to prune the search space. The `STIntersects` predicate checks which geometries overlap with the specified polygon. The R-tree index quickly eliminates places whose bounding rectangles do not intersect, avoiding a full table scan .

#### Real-World Cases

**Case 1: Ride-Sharing Applications**

A ride-sharing app uses spatial indexes to find available drivers within a radius of a rider's location. The query `WHERE STDistance(driver_location, rider_location) < 1000` uses the spatial index to check only nearby drivers.

**Case 2: Real Estate Listings**

A property website indexes the locations of all listings. A user drawing a polygon on a map triggers a spatial query that returns all properties within the selected area, powered by an R-tree index.

#### References

- Oracle Spatial — R-tree Indexing - https://docs.oracle.com/cd/B19306_01/appdev.102/b14255/sdo_intro.htm
- R-trees in Handbook of Data Structures and Applications - https://www.taylorfrancis.com/chapters/mono/10.1201/9781420035179-32/trees-dinesh-mehta-dinesh-mehta-sartaj-sahni
- Microsoft SQL Server — Indexes Overview (Spatial) - https://learn.microsoft.com/fil-ph/sql/relational-databases/indexes/indexes?view=azuresqldb-current

---

### 5.3 JSON / Inverted Indexes

#### Definitions

**Core Definition:** A JSON index (often implemented as an inverted index) indexes specific paths or attributes within semi-structured JSON documents, enabling efficient queries on document properties.

**Technical Definition:** JSON indexes in databases like MySQL 8.0.13+ are implemented as functional indexes on expression values extracted from JSON documents (e.g., `CAST(data->>'$.country' AS CHAR(100))`). PostgreSQL uses GIN (Generalized Inverted Index) to index JSONB documents, creating a separate index entry for each key-value pair .

**Beginner-Friendly Explanation:** A JSON index is like creating a separate index card for each important field in a stack of forms. If every form has a "Country" field, you make a card for each country and note which forms contain it. When you need all forms from "US", you just read the US card.

#### Purposes

- To accelerate queries on specific paths within JSON or JSONB columns.
- To avoid full document scans when filtering on document attributes.
- To support flexible schema designs without sacrificing query performance.

#### Syntax Rules and Structure

**Complete General Syntax (MySQL 8.0.13+):**

```sql
CREATE INDEX index_name ON table_name ((CAST(json_column->>'$.path' AS type)));
```

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name ON table_name USING gin (jsonb_column);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `->>'$.path'` | JSON path extraction (MySQL) |
| `CAST(... AS type)` | Type conversion for the indexed expression |
| `USING gin` | GIN index method for JSONB (PostgreSQL) |

**Syntax Rules:**

- MySQL functional indexes on JSON fields must wrap expressions in parentheses .
- PostgreSQL GIN indexes support operators like `@>`, `?`, and `?|` for JSONB containment and key existence .
- The extracted expression must be deterministic and cannot reference other generated columns .

**Constraints and Limitations:**

- Functional indexes on JSON are limited to specific paths; queries on other paths cannot use the index.
- GIN indexes in PostgreSQL are larger and slower to update than B-tree indexes.
- JSON path expressions must match exactly between the index definition and the query.

#### Annotated Complete Code Examples

**Example 1: MySQL JSON Functional Index**

```sql
-- Create a table with a JSON column
CREATE TABLE profiles (
    id   INT AUTO_INCREMENT PRIMARY KEY,
    data JSON NOT NULL,
    INDEX idx_country ((CAST(data->>'$.country' AS CHAR(100))))
);

-- Insert sample data
INSERT INTO profiles (data) VALUES
    ('{"name": "Alice", "country": "US"}'),
    ('{"name": "Bob", "country": "UK"}');

-- Query using the functional index
SELECT id, data->>'$.name' AS name
FROM profiles
WHERE CAST(data->>'$.country' AS CHAR(100)) = 'US';

-- Expected Output:
--  id | name
-- ----+-------
--   1 | Alice
```

**Why this output occurs:** The functional index stores the computed value of `CAST(data->>'$.country' AS CHAR(100))` for each row. The query's `WHERE` clause uses the identical expression, allowing the optimizer to use the index to find rows where the country is "US" without scanning the JSON documents .

**Example 2: PostgreSQL GIN Index on JSONB**

```sql
-- Create a table with a JSONB column
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    payload  JSONB
);

-- Create a GIN index
CREATE INDEX idx_events_payload ON events USING gin (payload);

-- Query using containment operator
SELECT event_id
FROM events
WHERE payload @> '{"type": "click"}';

-- Expected Output (conceptual):
--  event_id
-- ----------
--        42
--        57
```

**Why this output occurs:** The GIN index creates an entry for each key-value pair in the JSONB document. The `@>` containment operator checks whether the document contains the specified key-value pair. The GIN index finds matching entries without scanning every document .

#### Real-World Cases

**Case 1: Event Tracking**

An analytics platform stores user events as JSON documents. A GIN index on the `payload` column allows queries like "find all events where `type = 'purchase'`" to run efficiently across millions of events.

**Case 2: Configuration Management**

A SaaS application stores per-tenant configuration as JSON. A functional index on `(config->>'$.plan_tier')` accelerates billing queries that filter tenants by their subscription level.

#### References

- PostgreSQL Documentation — Index Types (GIN) - https://www.postgresql.org/docs/10/indexes-types.html
- MySQL — What Is a Functional Index - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-what-is-a-functional-index-in-mysql/README.md
- PostgreSQL — JSONB Index Projection - https://www.postgresql.org/message-id/attachment/52885/projection.patch

---

### 5.4 Expression-Based / Functional Indexes

#### Definitions

**Core Definition:** An expression-based (or functional) index indexes the result of a function or expression applied to one or more columns, rather than the raw column values.

**Technical Definition:** A functional index stores precomputed values of an expression in the index structure. When a query uses the same expression in its `WHERE` clause, the optimizer can use the index to satisfy the predicate. Oracle documentation notes that function-based indexes are treated specially: Oracle does not use them until the index and table are analyzed . MySQL 8.0.13+ implements functional indexes as hidden virtual generated columns .

**Beginner-Friendly Explanation:** A functional index is like creating a new column that contains the result of a calculation (e.g., `UPPER(email)`), then indexing that new column. Queries that use the same calculation can then use the index, even though the original column has no index.

#### Purposes

- To enable index usage for queries that apply functions to columns (which normally prevents index use) .
- To support case-insensitive searches without modifying the underlying data.
- To index computed values such as date parts, string prefixes, or JSON attributes.
- To avoid creating and maintaining explicit generated columns .

#### Syntax Rules and Structure

**Complete General Syntax (PostgreSQL):**

```sql
CREATE INDEX index_name ON table_name ((function_name(column_name)));
```

**Complete General Syntax (MySQL 8.0.13+):**

```sql
CREATE INDEX index_name ON table_name ((expression));
```

**Complete General Syntax (Oracle):**

```sql
CREATE INDEX index_name ON table_name (function_name(column_name));
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `function_name(column_name)` | The expression to index |
| Parentheses | Required in PostgreSQL and MySQL to distinguish from column list |

**Syntax Rules:**

- The expression must be deterministic (no `RAND()`, `NOW()`, etc.) .
- The query must use the exact same expression for the index to be used .
- In Oracle, the index must be analyzed before the optimizer will consider it .
- If the function becomes invalid or is dropped, Oracle marks the index as `DISABLED` .

**Constraints and Limitations:**

- Primary keys cannot be functional indexes .
- The expression cannot reference other generated columns .
- Oracle's function-based indexes are affected by session parameters like `NLS_SORT` and `NLS_COMP` .

#### Annotated Complete Code Examples

**Example 1: Case-Insensitive Email Search (MySQL)**

```sql
-- Create a table with an email column
CREATE TABLE accounts (
    id    INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL
);

-- Create a functional index on LOWER(email)
ALTER TABLE accounts ADD INDEX idx_email_lower ((LOWER(email)));

-- Insert sample data
INSERT INTO accounts (email) VALUES ('Alice@EXAMPLE.COM');

-- Query using the functional index
SELECT id, email
FROM accounts
WHERE LOWER(email) = 'alice@example.com';

-- Expected Output:
--  id | email
-- ----+------------------
--   1 | Alice@EXAMPLE.COM
```

**Why this output occurs:** Without the functional index, the query would perform a full table scan because `LOWER(email)` is not indexed. The functional index precomputes and stores `LOWER(email)` values. The query's `WHERE` clause uses the same expression, so the optimizer uses the index to find the matching row .

**Example 2: Indexing a String Prefix (PostgreSQL)**

```sql
-- Create a functional index on the first 10 characters of a URL
CREATE INDEX idx_pages_url_prefix ON pages ((LEFT(url, 10)));

-- Query using the prefix expression
SELECT url
FROM pages
WHERE LEFT(url, 10) = 'https://ex';

-- Expected Output (conceptual):
--       url
-- -----------------
--  https://example.com
--  https://example.org
```

**Why this output occurs:** The functional index stores the first 10 characters of each URL. Queries that filter on the same expression can use the index to quickly find matching rows. This is useful when URLs are very long and indexing the full URL would be wasteful .

#### Real-World Cases

**Case 1: Case-Insensitive Login**

A web application stores emails in mixed case but needs case-insensitive login. A functional index on `LOWER(email)` allows `WHERE LOWER(email) = ?` to use an index, avoiding full scans.

**Case 2: Date Part Reporting**

A reporting system frequently queries orders by month: `WHERE MONTH(created_at) = 3`. A functional index on `MONTH(created_at)` accelerates these queries, which would otherwise require full table scans .

#### References

- MySQL — What Is a Functional Index in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-what-is-a-functional-index-in-mysql/README.md
- Oracle Database — Function-Based Indexes - https://docs.oracle.com/cd/B10500%5F01/server.920/a96540/statements_510a.htm
- PostgreSQL — Functional Indexes - https://www.postgresql.org/message-id/20071207174441.GE2878%40svana.org

---

## Summary Table

| Index Structure | Best For | Key Limitation | Typical Use Case |
|-----------------|----------|----------------|------------------|
| B-tree / B+tree | Range and equality queries | Write overhead from page splits | Primary keys, date ranges |
| Hash | Equality-only lookups | No range queries | Session tokens, cache keys |
| Bitmap | Low-cardinality columns | Poor for OLTP writes | Data warehouse filtering |
| LSM Tree | Write-heavy workloads | Read amplification | NoSQL, distributed databases |
| Full-Text | Natural language search | Stopword removal, fragments | Site search, document retrieval |
| Spatial (R-tree) | Geographic queries | Bounding box setup | GIS, ride-sharing, real estate |
| JSON / GIN | Semi-structured documents | Size and update cost | Event tracking, configuration |
| Functional / Expression | Function-wrapped predicates | Deterministic expressions only | Case-insensitive search, date parts |

---

## References

- PostgreSQL Documentation — Index Types - https://www.postgresql.org/docs/10/indexes-types.html
- Microsoft SQL Server — Indexes Overview - https://learn.microsoft.com/fil-ph/sql/relational-databases/indexes/indexes?view=azuresqldb-current
- Oracle Database — Managing Indexes - https://docs.oracle.com/cd/B16341_01/doc/server.102/b14196/schema003.htm
- Microsoft SQL Server — Full-Text Index Structure - https://learn.microsoft.com/hi-in/previous-versions/sql/sql-server-2008-r2/ms142505(v=sql.105)
- Oracle Spatial — R-tree Indexing - https://docs.oracle.com/cd/B19306_01/appdev.102/b14255/sdo_intro.htm
- Google Spanner Ressi LSM Tree - https://pdfs.semanticscholar.org/fc57/e5d5790ac461bfabe89975cb1ea10e2ec30f.pdf
- MySQL — What Is a Functional Index in MySQL - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mysql-what-is-a-functional-index-in-mysql/README.md
- Oracle Database — Function-Based Indexes - https://docs.oracle.com/cd/B10500%5F01/server.920/a96540/statements_510a.htm
- R-trees in Handbook of Data Structures and Applications - https://www.taylorfrancis.com/chapters/mono/10.1201/9781420035179-32/trees-dinesh-mehta-dinesh-mehta-sartaj-sahni
- Distributed Databases Using Many-Core Processors (LSM Tree) - https://FreePatentsOnline.com/y2014/0280375.html
- PostgreSQL — Functional Indexes - https://www.postgresql.org/message-id/20071207174441.GE2878%40svana.org
- PostgreSQL — JSONB Index Projection - https://www.postgresql.org/message-id/attachment/52885/projection.patch