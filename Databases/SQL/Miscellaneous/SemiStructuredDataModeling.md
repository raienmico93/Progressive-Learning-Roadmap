# Semi-Structured Data Modeling: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Semi-structured data modeling is the practice of designing database schemas that accommodate data with flexible, variable, or evolving structures—combining elements of rigid relational tables with the flexibility of document-oriented storage.

**Technical Definition**
Semi-structured modeling encompasses hybrid schema patterns that store predictable attributes in strongly-typed relational columns while relegating variable or sparse attributes to JSON documents, EAV tables, or auxiliary key-value structures. It navigates the trade-off between schema-on-write (enforced at ingestion) and schema-on-read (interpreted at query time).

**Beginner-Friendly Explanation**
Traditional tables require every row to have the same columns. But real-world data—product catalogs, user preferences, event logs—often varies. Semi-structured modeling gives you the best of both worlds: keep the common stuff in regular columns and put the variable stuff in a flexible "box" (JSON, EAV, etc.) that can hold anything.

### Key Characteristics

- **Hybrid Storage**: Combines relational columns for stable attributes with JSON/EAV for variable attributes .
- **Schema Flexibility**: Accommodates polymorphic data patterns where different records have different fields .
- **Trade-off Spectrum**: Balances query performance, storage efficiency, and schema evolution ease .
- **Schema-on-Read vs. Schema-on-Write**: Determines when structure is enforced—at ingestion or at query time .

### Prerequisites

- Relational database fundamentals (tables, columns, indexes)
- JSON syntax and querying basics
- Understanding of normalization principles
- Familiarity with indexing concepts

### Related Programming Areas

- **NoSQL Integration**: Document stores, key-value databases
- **Data Lakes**: Schema-on-read analytics
- **Multi-tenant SaaS**: Per-customer custom fields
- **E-commerce Catalogs**: Product attributes that vary by category

### Core Concepts

| Concept | Description |
|---------|-------------|
| Relational vs. Document Storage | Normalized tables with joins vs. denormalized self-contained documents |
| Hybrid Schemas | Common fields as columns; variable fields as JSON |
| JSON Normalization | Shredding JSON arrays into relational child tables |
| EAV Pattern | Entity-Attribute-Value rows for sparse, extensible attributes |
| Schema-on-Write/Read | When schema enforcement occurs |
| Data Evolution | Handling schema changes over time |


## 1. Relational versus Document-Style Storage

### Definitions

**Core Definition**
Relational storage normalizes data into multiple tables linked by foreign keys, while document-style storage embeds related data into a single self-contained document.

**Technical Definition**
Relational modeling adheres to normal forms, minimizing redundancy but requiring joins for retrieval. Document modeling follows access patterns, co-locating data that is read together, accepting duplication for single-fetch reads .

**Beginner-Friendly Explanation**
Relational is like a well-organized filing cabinet: customer info in one drawer, orders in another, linked by reference numbers. Document-style is like a folder per customer containing everything about them in one place.

### Purposes (All begin with "To")

- **To** choose the right storage model based on access patterns
- **To** optimize for either flexible querying (relational) or fast aggregate retrieval (document)
- **To** understand the performance implications of joins vs. single-document reads

### Syntax Rules and Structure

**Relational Model**
```sql
CREATE TABLE Customers (CustomerID INT PRIMARY KEY, Name VARCHAR(100));
CREATE TABLE Orders (OrderID INT PRIMARY KEY, CustomerID INT, OrderDate DATE);
-- Retrieval requires JOIN
```

**Document Model**
```json
{
  "customer_id": 1,
  "name": "Alice",
  "orders": [
    {"order_id": 101, "date": "2025-01-15"},
    {"order_id": 102, "date": "2025-02-20"}
  ]
}
```

### Decision Matrix

| Factor | Relational | Document |
|--------|------------|----------|
| Query Flexibility | High (SQL) | Medium (per-path)  |
| Schema Evolution | Migrations required | Flexible  |
| Join Performance | Good (indexed) | Poor (app-side)  |
| Data Duplication | Minimal (normalized) | Possible (denormalized) |
| Best For | Many-to-many, reporting | Aggregate reads, evolving schemas |

### Real-World Cases

- **GitHub**: Uses MySQL as system of record for relational integrity 
- **eBay**: Uses MongoDB for catalog data where listings vary by category 

### References

- Storage Choices: SQL vs NoSQL - https://raw.githubusercontent.com/sujeet-pro/sujeet.pro/refs/heads/main/content/articles/storage-choices-sql-vs-nosql/README.md


## 2. Hybrid Schemas (Polymorphic Data Patterns)

### Definitions

**Core Definition**
A hybrid schema stores predictable, universal attributes in traditional relational columns while placing variable, category-specific attributes in a JSON document column within the same table.

**Technical Definition**
This pattern combines the query performance of typed columns for common fields with the flexibility of semi-structured storage for polymorphic attributes, avoiding sparse columns or complex EAV tables .

**Beginner-Friendly Explanation**
A product catalog has name, price, and category for every product—those go in regular columns. But shirts have size and color, laptops have CPU speed, books have author. Those go in a JSON "extras" column.

### Purposes (All begin with "To")

- **To** avoid creating many sparse columns for rarely-used attributes
- **To** add new product types without altering the table structure
- **To** maintain query performance on common fields while supporting flexibility
- **To** keep related data in a single row for fast retrieval

### Syntax Rules and Structure

**General Syntax (SQL Server 2025+)**
```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name NVARCHAR(200) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    Category NVARCHAR(100) NOT NULL,
    Attributes JSON  -- Variable attributes
);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Fixed columns | Name, Price, Category (universal) |
| JSON column | Attributes (category-specific) |
| Index strategy | B-tree on fixed columns; computed column + index on JSON paths |

### Annotated Code Examples

**Example 1: Hybrid Product Catalog (SQL Server)**

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name NVARCHAR(200) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    Category NVARCHAR(100) NOT NULL,
    Attributes JSON
);

-- Shirt
INSERT INTO Products VALUES (1, 'T-Shirt', 19.99, 'Apparel',
    '{"size": "L", "color": "blue", "fabric": "cotton"}');

-- Laptop
INSERT INTO Products VALUES (2, 'UltraBook', 1299.99, 'Electronics',
    '{"cpu": "i7", "ram_gb": 16, "screen_inches": 14}');

-- Query common fields
SELECT * FROM Products WHERE Category = 'Electronics';

-- Query JSON attribute (requires computed column for index)
SELECT ProductID, Name FROM Products
WHERE JSON_VALUE(Attributes, '$.ram_gb') >= 16;
```

**Expected Output:**
```
ProductID | Name      | Category    | Attributes
1         | T-Shirt   | Apparel     | {"size": "L", ...}
2         | UltraBook | Electronics | {"cpu": "i7", ...}

ProductID | Name
2         | UltraBook
```

**Why This Output Occurs**: Common fields use typed columns for efficient filtering. Category-specific attributes are stored as JSON, allowing both product types to coexist without sparse columns.

### Real-World Cases

- **E-commerce**: Product catalogs with category-specific attributes 
- **User Preferences**: Theme, language, notification settings per user 
- **Audit Logs**: Before/after states that evolve with table schemas 

### References

- Microsoft Learn - Managing JSON columns and indexes - https://learn.microsoft.com/fr-fr/training/modules/design-implement-database-objects/7-design-implement-json
- Microsoft Learn (Japanese) - JSON 列とインデックスを管理する - https://learn.microsoft.com/ja-jp/training/modules/design-implement-database-objects/7-design-implement-json


## 3. JSON Normalization (De-structuring Arrays)

### Definitions

**Core Definition**
JSON normalization (shredding) converts nested JSON arrays into relational child tables, enabling SQL joins, aggregations, and set operations on document data.

**Technical Definition**
Functions like `OPENJSON` (SQL Server) or `jsonb_array_elements` (PostgreSQL) expand JSON arrays into rows, which can then be joined with parent records via `CROSS APPLY` or `LATERAL JOIN` .

**Beginner-Friendly Explanation**
If you have an order with a list of items stored as JSON, shredding turns that list into rows—one row per item—so you can join, sum, and filter like any normal table.

### Purposes (All begin with "To")

- **To** query JSON arrays as if they were relational rows
- **To** join nested data with other tables
- **To** aggregate values stored within JSON arrays
- **To** migrate document data into normalized form

### Syntax Rules and Structure

**SQL Server (OPENJSON)**
```sql
SELECT SalesOrderID, OrderDate, value AS Reason
FROM Sales.SalesOrderHeader
CROSS APPLY OPENJSON(SalesReasons);
```

**PostgreSQL (jsonb_array_elements)**
```sql
SELECT o.id, item->>'product_id' AS product_id
FROM orders o,
LATERAL jsonb_array_elements(o.details->'items') AS item;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `OPENJSON` / `jsonb_array_elements` | Table-valued function expanding arrays |
| `CROSS APPLY` / `LATERAL JOIN` | Correlates parent row with array elements |
| `value` / `->>` | Extracts scalar from element |

### Annotated Code Examples

**Example 1: Shredding JSON Array to Rows (SQL Server)**

```sql
DECLARE @json NVARCHAR(MAX) = '{
  "order_id": 1001,
  "customer": "Alice",
  "items": [
    {"product": "Widget", "qty": 2, "price": 9.99},
    {"product": "Gadget", "qty": 1, "price": 19.99}
  ]
}';

SELECT
    order_id = JSON_VALUE(@json, '$.order_id'),
    customer = JSON_VALUE(@json, '$.customer'),
    product = JSON_VALUE(item.value, '$.product'),
    qty = JSON_VALUE(item.value, '$.qty'),
    price = JSON_VALUE(item.value, '$.price')
FROM OPENJSON(@json, '$.items') AS item;
```

**Expected Output:**
```
order_id | customer | product | qty | price
1001     | Alice    | Widget  | 2   | 9.99
1001     | Alice    | Gadget  | 1   | 19.99
```

**Why This Output Occurs**: `OPENJSON` with path `$.items` expands the array into two rows. `JSON_VALUE` extracts scalars from each element, producing a relational result set .

### Real-World Cases

- **Order Processing**: Shredding line items for reporting 
- **Data Migration**: Converting JSON documents to normalized tables
- **Analytics**: Joining JSON arrays with dimension tables

### References

- Microsoft Learn - Using OPENJSON with the Default Schema - https://learn.microsoft.com/id-id/sql/relational-databases/json/use-openjson-with-the-default-schema-sql-server
- Microsoft Learn (German) - Verwenden von OPENJSON - https://learn.microsoft.com/de-de/sql/relational-databases/json/use-openjson-with-the-default-schema-sql-server


## 4. Flexible Attributes: EAV vs. JSON Column

### Definitions

**Core Definition**
Flexible attributes are data fields that vary per record, requiring either an EAV (Entity-Attribute-Value) table structure or a JSON column to store them without schema changes.

**Technical Definition**
EAV stores each attribute as a separate row in a pivot table (`entity_id`, `attribute_id`, `value`), while JSON stores all variable attributes as a single document per entity. EAV enables attribute-level indexing and constraints; JSON offers simpler queries and atomic updates .

**Beginner-Friendly Explanation**
For custom product properties: EAV is like having a separate spreadsheet row for every property of every product—flexible but slow to reassemble. JSON is like having one cell per product containing all its properties—fast to read but harder to filter across products.

### Purposes (All begin with "To")

- **To** support arbitrary user-defined or category-specific fields
- **To** avoid sparse columns for rarely-populated attributes
- **To** choose between query simplicity (JSON) and constraint flexibility (EAV)

### Performance Comparison

| Aspect | EAV | JSON Column |
|--------|-----|-------------|
| Query Simplicity | Complex (multi-join) | Simple (`->` operators)  |
| Performance | 3-4x slower  | 10-30% slower than columns  |
| Indexing | Per-attribute | Computed columns or GIN  |
| Constraint Enforcement | Per-attribute (foreign key to definitions) | Difficult (application-level)  |
| Storage Efficiency | Good (only stores set attributes) | Good (only stores set keys) |

### Annotated Code Examples

**Example 1: EAV vs. JSON Query Comparison**

```sql
-- EAV: Find products with size L and color red
SELECT p.id, p.name
FROM products p
JOIN product_property pp1 ON p.id = pp1.product_id AND pp1.property_id = 1 AND pp1.value = 'L'
JOIN product_property pp2 ON p.id = pp2.product_id AND pp2.property_id = 2 AND pp2.value = 'red';

-- JSON: Same query
SELECT id, name
FROM products
WHERE attributes->>'size' = 'L'
  AND attributes->>'color' = 'red';
```

**Expected Output:**
```
id | name
1  | T-Shirt
```

**Why This Output Occurs**: EAV requires one join per attribute, making the query complex and slower. JSON uses simple path operators within a single row .

### Real-World Cases

- **Product Properties**: Size, color, fabric that vary by category 
- **User Custom Fields**: Per-tenant settings in SaaS applications 
- **Configuration Management**: Dynamic settings that evolve

### References

- Laravel Daily - Custom Fields: JSON, EAV, or Same Table? - https://laraveldaily.com/post/laravel-custom-fields-json-eav-model-same-table
- CSDN - 弹性属性系统架构设计 - https://guosy.blog.csdn.net/article/details/164055496


## 5. Trade-offs in Schema Flexibility (Schema-on-Write vs. Schema-on-Read)

### Definitions

**Core Definition**
Schema-on-write enforces structure at insertion time (traditional RDBMS); schema-on-read interprets structure at query time (data lakes, document stores).

**Technical Definition**
Schema-on-write provides immediate validation, optimized storage, and fast queries but requires migrations for changes. Schema-on-read enables rapid ingestion of varied data but defers parsing costs and may sacrifice query performance .

**Beginner-Friendly Explanation**
Schema-on-write is like filling out a form with strict fields—everything must fit. Schema-on-read is like throwing papers in a box and sorting them when you need something.

### Purposes (All begin with "To")

- **To** choose the right timing for schema enforcement
- **To** balance ingestion speed against query performance
- **To** handle evolving data sources without ETL bottlenecks

### Comparison

| Aspect | Schema-on-Write | Schema-on-Read |
|--------|-----------------|----------------|
| Enforcement | At ingestion  | At query  |
| Ingestion Speed | Slower (validation) | Faster (raw storage) |
| Query Performance | Fast (indexed) | Slower (parsing)  |
| Schema Changes | Migrations required  | No migration needed |
| Best For | Known, stable schemas | Evolving, unknown schemas |

### Annotated Code Examples

**Example 1: Schema-on-Read Runtime Fields (Elasticsearch)**

```
// Schema-on-read: define runtime field at query time
GET logs/_search
{
  "runtime_mappings": {
    "response_category": {
      "type": "keyword",
      "script": "if (doc['status'].value < 400) emit('success') else emit('error')"
    }
  },
  "aggs": {
    "by_category": { "terms": { "field": "response_category" } }
  }
}
```

**Expected Output:**
```
{
  "aggregations": {
    "by_category": {
      "buckets": [
        {"key": "success", "doc_count": 950},
        {"key": "error", "doc_count": 50}
      ]
    }
  }
}
```

**Why This Output Occurs**: The `response_category` field is computed at query time, not indexed. This avoids reindexing but incurs evaluation cost on each query .

### Real-World Cases

- **Log Analytics**: Ingest raw logs; parse fields on read 
- **Data Lakes**: Store varied JSON; define schema for specific queries 
- **Observability**: Runtime fields for ad-hoc analysis 

### References

- Elastic - Schema on write or schema on read - https://www.elastic.co/jp/blog/observability-logs-parsing-schema-read-write
- Supply Chain Shaman - Schema-on-Write vs Schema-on-Read - https://www.supplychainshaman.com/redefining-your-relationship-with-data/
- Datahike - Schema-on-Read vs Schema-on-Write - https://raw.githubusercontent.com/replikativ/datahike/refs/tags/0.4.1475/doc/schema.md


## 6. Data Evolution and Versioning Strategies

### Definitions

**Core Definition**
Data evolution and versioning strategies manage schema changes over time without breaking existing queries, pipelines, or applications.

**Technical Definition**
Strategies include schema versioning (tracking versions in metadata), evolution operations (add, rename, delete, copy, move, split, merge), and migration approaches (eager, lazy, predictive) that transform data to conform to the latest schema .

**Beginner-Friendly Explanation**
When your data's structure changes, you need a plan: How do you add a new field? What happens to old records? Versioning is like keeping a history of blueprints for your data.

### Purposes (All begin with "To")

- **To** handle new fields without breaking existing queries
- **To** maintain compatibility across schema versions
- **To** migrate data to conform to the latest structure
- **To** audit and roll back schema changes if needed

### Evolution Operations

| Operation | Description | Example |
|-----------|-------------|---------|
| Add Property | New field appears | `promo_code` added  |
| Rename Property | Field name changes | `status` → `order_status` |
| Delete Property | Field removed | `shipping_method` dropped  |
| Type Promotion | Type widens | `int` → `decimal`  |
| Copy/Move | Refactoring | Denormalization/normalization  |

### Annotated Code Examples

**Example 1: AWS Glue Schema Evolution Handling**

```python
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql.functions import col, lit

# Read from Glue Catalog
dyf = glueContext.create_dynamic_frame.from_catalog(
    database="sales_db", table_name="orders")
df = dyf.toDF()

# Handle missing 'promo_code' column
if 'promo_code' not in df.columns:
    df = df.withColumn("promo_code", lit(None).cast("string"))

# Ensure 'discount' is float (type promotion)
df = df.withColumn("discount", col("discount").cast("float"))

# Back to dynamic frame
dyf_transformed = DynamicFrame.fromDF(df, glueContext, "dyf_transformed")
```

**Expected Output:**
```
Schema updated: promo_code added as string, discount promoted to float
```

**Why This Output Occurs**: The ETL job defensively checks for new/missing columns and handles type changes, ensuring the pipeline continues even as the source schema evolves .

### Real-World Cases

- **Data Lakes**: Evolving JSON schemas in S3 
- **Multi-tenant SaaS**: Per-customer custom fields 
- **Analytics Platforms**: Managing schema drift across partitions 

### References

- CloudThat - Managing Schema Evolution in AWS Glue - https://www.cloudthat.com/resources/blog/managing-schema-evolution-in-aws-glue-data-catalog
- SPLC 2025 - Schema Extraction and Evolution for NoSQL - https://2025.splc.net/wp-content/uploads/2025/09/Keynote_SPL_2025_final.pdf
- GitHub Olake - Handling Changing Data Types - https://github.com/datazip-inc/olake-docs/blob/master/blog/2024-10-10-handling-changing-data-type-during-semi-structured-data-ingestion.mdx


## Consolidated References

- Storage Choices: SQL vs NoSQL - https://raw.githubusercontent.com/sujeet-pro/sujeet.pro/refs/heads/main/content/articles/storage-choices-sql-vs-nosql/README.md
- PGDay.ch - Normalize or De-normalize: SQL Columns or JSON Document Attributes - https://www.pgday.ch/common/slides/2025_20250626_pgday.ch_-_Normalize_or_De-normalize_Relational_SQL_Columns_or_JSON_Document_Attributes.pdf
- Microsoft Learn - Using OPENJSON with the Default Schema - https://learn.microsoft.com/id-id/sql/relational-databases/json/use-openjson-with-the-default-schema-sql-server
- Laravel Daily - Custom Fields: JSON, EAV Model, or Same Table? - https://laraveldaily.com/post/laravel-custom-fields-json-eav-model-same-table
- Elastic - Schema on write or schema on read - https://www.elastic.co/jp/blog/observability-logs-parsing-schema-read-write
- GitHub Olake - Handling Changing Data Types - https://github.com/datazip-inc/olake-docs/blob/master/blog/2024-10-10-handling-changing-data-type-during-semi-structured-data-ingestion.mdx
- Microsoft Learn (French) - Gérer les colonnes et index JSON - https://learn.microsoft.com/fr-fr/training/modules/design-implement-database-objects/7-design-implement-json
- GitHub - Choosing a Database - https://raw.githubusercontent.com/nimin1/system-design-vibecoding/refs/heads/main/04-data-layer/04-choosing-database.md
- MariaDB Knowledge Base - RDF (EAV flavor) - https://mariadb.org/wp-content/uploads/2025/03/MariaDBServerKnowledgeBase.pdf
- Microsoft Learn (German) - Verwenden von OPENJSON - https://learn.microsoft.com/de-de/sql/relational-databases/json/use-openjson-with-the-default-schema-sql-server
- CSDN - 弹性属性系统架构设计 - https://guosy.blog.csdn.net/article/details/164055496
- Supply Chain Shaman - Redefining Your Relationship with Data - https://www.supplychainshaman.com/redefining-your-relationship-with-data/
- CloudThat - Managing Schema Evolution in AWS Glue - https://www.cloudthat.com/resources/blog/managing-schema-evolution-in-aws-glue-data-catalog
- Microsoft Learn (Japanese) - JSON 列とインデックスを管理する - https://learn.microsoft.com/ja-jp/training/modules/design-implement-database-objects/7-design-implement-json
- DTU Orbit - Document-oriented data model - https://orbit.dtu.dk/files/270344482/1_s2.0_S0065245821000590_main.pdf
- PostgreSQL Conference EU - Normalize or Denormalize? - https://www.postgresql.eu/events/pgconfeu2025/sessions/session/6868/slides/745/20251022%20-%20Riga%20-%20Normalize%20or%20De-normalize_%20Relational%20SQL%20Columns%20or%20JSON%20Document%20Attributes.pdf
- Microsoft Learn (Turkish) - TransformJsonQueryToTable - https://learn.microsoft.com/tr-tr/dotnet/api/microsoft.entityframeworkcore.query.relationalqueryablemethodtranslatingexpressionvisitor.transformjsonquerytotable
- AGH Journals - JSON Model and EAV Comparison - https://journals.agh.edu.pl/csci/article/download/2505/2005/11746
- Datahike - Schema-on-Read vs Schema-on-Write - https://raw.githubusercontent.com/replikativ/datahike/refs/tags/0.4.1475/doc/schema.md
- SPLC 2025 - Schema Extraction and Evolution for NoSQL - https://2025.splc.net/wp-content/uploads/2025/09/Keynote_SPL_2025_final.pdf