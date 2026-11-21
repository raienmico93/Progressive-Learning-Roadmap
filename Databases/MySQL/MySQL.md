# MySQL Comprehensive, Structured, and Progressive Learning Roadmap

## From Foundational Concepts to Advanced Database Engineering and Production Data Systems (MySQL Edition)

This roadmap merges foundational database theory, SQL syntax, querying, modeling, performance, security, and production engineering into a single progressive MySQL curriculum.

Progression: **Relational Theory → MySQL Syntax → CRUD → Querying → Joins → Aggregation → Subqueries → CTEs → Window Functions → Database Design → Normalization → Views → Advanced SQL → Transactions → Programmability → Indexing → Query Optimization → Security → Recovery → Replication → Partitioning → Data Warehousing → Distributed Architecture → Production MySQL Engineering.**

---

# I. MySQL Foundations and Database Fundamentals

- **1. What a Database Is**
  - Databases
  - Database management systems
  - Relational databases
  - Non-relational databases
  - Tables
  - Rows and records
  - Columns and attributes
  - Schemas (databases in MySQL terminology)
  - Catalogs
  - Instances
  - Clusters
  - Relationships
  - Constraints
  - Indexes
  - Views
  - Stored programs
  - Data, information, and metadata

- **2. Relational Model**
  - Relations
  - Tuples
  - Attributes
  - Domains
  - Keys
    - Primary keys
    - Candidate keys
    - Alternate keys
    - Composite keys
    - Surrogate keys
    - Natural keys
    - Foreign keys
  - Referential integrity
  - Entity integrity
  - Relational algebra
  - Relational calculus
  - Codd's rules

- **3. MySQL Language Overview**
  - What SQL is
  - Structured Query Language
  - Declarative programming paradigm
  - Standard language for relational database interaction
  - SQL standards
    - SQL-86
    - SQL-89
    - SQL-92
    - SQL:1999
    - SQL:2003
    - SQL:2008
    - SQL:2011
    - SQL:2016
    - SQL:2023
  - MySQL versions and feature evolution
    - MySQL 5.7
    - MySQL 8.0
    - MySQL 8.4 LTS
    - MySQL 9.x Innovation releases
  - MySQL vs MariaDB
  - MySQL vs PostgreSQL
  - MySQL dialects and compatibility modes
  - SQL sublanguages
    - DDL
    - DML
    - DQL
    - DCL
    - TCL
  - Declarative vs imperative
  - Set-based thinking

- **4. MySQL Environment and Tooling**
  - MySQL command-line client (`mysql`)
  - MySQL Shell (`mysqlsh`)
  - MySQL Workbench
  - phpMyAdmin
  - Adminer
  - DBeaver
  - DataGrip
  - TablePlus
  - HeidiSQL
  - Query editors
  - Database explorers
  - Connection management
  - MySQL connection parameters
    - Host
    - Port (default 3306)
    - Database name
    - Username
    - Authentication (`caching_sha2_password`, `mysql_native_password`)
    - Socket file
    - SSL/TLS parameters
  - Executing SQL scripts
  - Importing and exporting SQL files
  - `SOURCE` command
  - `mysqldump` and `mysql` client restore

- **5. MySQL Syntax Fundamentals**
  - SQL statement structure
  - Keywords
  - Identifiers
  - Backtick quoting for identifiers
  - Literals
  - Operators
  - Expressions
  - Clauses
  - Statements
  - Comments
    - Single-line comments (`-- ` and `#`)
    - Multi-line comments (`/* */`)
    - Executable comments (`/*! ... */`)
  - Statement terminators (`;`, `\G`, `\g`)
  - Identifier naming conventions
  - Case sensitivity considerations
    - `lower_case_table_names` setting
    - Column name case insensitivity

- **6. NULL Semantics**
  - What NULL means
  - NULL vs empty string
  - NULL vs zero
  - Three-valued logic
  - `IS NULL`
  - `IS NOT NULL`
  - `<=>` NULL-safe equality operator
  - `COALESCE`
  - `IFNULL`
  - `NULLIF`
  - NULL propagation
  - NULL in comparisons
  - NULL in aggregates
  - NULL in joins
  - NULL in indexes
  - NULL in constraints

---

# II. MySQL Data Types

- **7. Data Type Fundamentals**
  - Data types
  - Type categories
  - Type selection
  - Data type best practices
  - Precision and scale
  - Storage implications
  - Strict SQL mode vs non-strict mode

- **8. Numeric Types**
  - `TINYINT`
  - `SMALLINT`
  - `MEDIUMINT`
  - `INT` / `INTEGER`
  - `BIGINT`
  - `DECIMAL`
  - `NUMERIC`
  - `FLOAT`
  - `DOUBLE`
  - `BIT`
  - `UNSIGNED` and `ZEROFILL` attributes
  - `AUTO_INCREMENT`
  - Numeric type best practices

- **9. Character Types**
  - `CHAR`
  - `VARCHAR`
  - `TINYTEXT`
  - `TEXT`
  - `MEDIUMTEXT`
  - `LONGTEXT`
  - Character sets (`utf8mb4`, `utf8`, `latin1`)
  - Collations (`utf8mb4_0900_ai_ci`, `utf8mb4_general_ci`, `utf8mb4_unicode_ci`)
  - Binary collations
  - Character type best practices

- **10. Date and Time Types**
  - `DATE`
  - `TIME`
  - `DATETIME`
  - `TIMESTAMP`
  - `YEAR`
  - `DATETIME` vs `TIMESTAMP`
  - Time zone handling (`time_zone` session variable)
  - `INTERVAL` expressions
  - Date and time best practices

- **11. Boolean Type**
  - `BOOLEAN` as `TINYINT(1)` alias
  - `TRUE`
  - `FALSE`
  - `NULL`
  - Boolean best practices

- **12. Binary Types**
  - `BINARY`
  - `VARBINARY`
  - `TINYBLOB`
  - `BLOB`
  - `MEDIUMBLOB`
  - `LONGBLOB`
  - Binary operations
  - Binary best practices

- **13. UUID Type**
  - `UUID` stored as `CHAR(36)` or `BINARY(16)`
  - `UUID()` function
  - `UUID_TO_BIN()` and `BIN_TO_UUID()`
  - `UUID_SHORT()`
  - UUID best practices

- **14. JSON Types**
  - `JSON` data type (MySQL 5.7+)
  - JSON vs JSONB (MySQL has only `JSON`)
  - JSON document structure
  - JSON operations
  - JSON functions
    - `JSON_OBJECT()`
    - `JSON_ARRAY()`
    - `JSON_AGG()` (MySQL 8.0.14+)
    - `JSON_ARRAYAGG()`
    - `JSON_OBJECTAGG()`
    - `JSON_EXTRACT()`
    - `JSON_SET()`
    - `JSON_INSERT()`
    - `JSON_REPLACE()`
    - `JSON_REMOVE()`
    - `JSON_PRETTY()`
    - `JSON_VALID()`
    - `JSON_TYPE()`
    - `JSON_LENGTH()`
    - `JSON_KEYS()`
    - `JSON_CONTAINS()`
    - `JSON_CONTAINS_PATH()`
    - `JSON_SEARCH()`
    - `JSON_MERGE_PATCH()`
    - `JSON_MERGE_PRESERVE()`
    - `JSON_TABLE()`
    - `JSON_PATH` expressions (`$`, `$.key`, `$[0]`, `$[*]`)
  - JSON best practices
  - JSON indexing (generated columns + indexes)

- **15. Array Types**
  - MySQL does not have native array types
  - Emulating arrays with JSON
  - Emulating arrays with child tables
  - `FIND_IN_SET()` for comma-separated values
  - `GROUP_CONCAT()` for aggregation
  - Array best practices in MySQL

- **16. Range Types**
  - MySQL does not have native range types
  - Emulating ranges with two columns
  - Emulating ranges with `GEOMETRY` types
  - Range best practices in MySQL

- **17. Network Address Types**
  - `INET6_ATON()` and `INET6_NTOA()`
  - `INET_ATON()` and `INET_NTOA()`
  - Storing IP addresses as `VARCHAR` or `VARBINARY`
  - Network address best practices

- **18. Bit String Types**
  - `BIT`
  - Bitwise operators
  - Bit string best practices

- **19. Text Search Types**
  - MySQL `FULLTEXT` indexes
  - `MATCH ... AGAINST` syntax
  - Natural language mode
  - Boolean mode
  - Query expansion mode
  - Text search best practices

- **20. Geometric Types**
  - `GEOMETRY`
  - `POINT`
  - `LINESTRING`
  - `POLYGON`
  - `MULTIPOINT`
  - `MULTILINESTRING`
  - `MULTIPOLYGON`
  - `GEOMETRYCOLLECTION`
  - Spatial reference systems (SRID)
  - Geometric operations
  - Geometric best practices

- **21. Enumerated Types**
  - `ENUM`
  - Enum creation
  - Enum operations
  - Enum best practices
  - Enum limitations

- **22. Composite Types**
  - MySQL does not have native composite types
  - Emulating with multiple columns
  - Emulating with JSON
  - Composite best practices in MySQL

- **23. Domain Types**
  - MySQL does not have native domain types
  - Emulating with `CHECK` constraints (MySQL 8.0.16+)
  - Domain best practices in MySQL

- **24. Custom Types**
  - MySQL does not support `CREATE TYPE`
  - Using `ENUM` and `SET` as alternatives
  - Custom type best practices in MySQL

- **25. XML and Specialized Types**
  - `XML` stored as `TEXT`
  - `ExtractValue()` and `UpdateXML()` functions
  - Large object types
  - Specialized types
  - Choosing appropriate data types

---

# III. Database and Schema Management

- **26. Database Management**
  - Creating databases (`CREATE DATABASE`)
  - Selecting databases (`USE`)
  - Listing databases (`SHOW DATABASES`)
  - Renaming databases (not directly supported; use `RENAME TABLE` or dump/restore)
  - Dropping databases (`DROP DATABASE`)
  - Database ownership
  - Database configuration
  - Database-level permissions
  - `information_schema.SCHEMATA`

- **27. Schema Management**
  - Schema vs database in MySQL (synonymous)
  - Creating schemas (`CREATE SCHEMA`)
  - Altering schemas (`ALTER SCHEMA`)
  - Dropping schemas (`DROP SCHEMA`)
  - Schema ownership
  - Schema namespaces
  - Organizing objects by schema
  - Multi-schema application design
  - `SCHEMA()` function
  - Schema permissions
  - Schema versioning

- **28. Table Fundamentals**
  - Creating tables (`CREATE TABLE`)
  - Viewing table definitions (`DESCRIBE`, `SHOW CREATE TABLE`)
  - Listing tables (`SHOW TABLES`)
  - Renaming tables (`RENAME TABLE`)
  - Truncating tables (`TRUNCATE TABLE`)
  - Dropping tables (`DROP TABLE`)
  - Temporary tables
  - In-memory tables (`ENGINE=MEMORY`)
  - Partitioned tables
  - Table cloning (`CREATE TABLE ... LIKE`, `CREATE TABLE ... AS SELECT`)
  - Derived tables
  - Table metadata (`information_schema.TABLES`)
  - Storage engines (`InnoDB`, `MyISAM`, `MEMORY`, `CSV`, `ARCHIVE`, `BLACKHOLE`)

---

# IV. Data Definition Language (DDL)

- **29. Creating Database Objects**
  - Tables
  - Schemas
  - Views
  - Indexes
  - Sequences (emulated with `AUTO_INCREMENT` or tables)
  - Constraints
  - Triggers
  - Stored procedures
  - Functions
  - Events

- **30. Creating Tables**
  - `CREATE TABLE`
  - Column definitions
  - Data types
  - Column constraints
  - Table constraints
  - Default values
  - Generated columns
    - `GENERATED ALWAYS AS (...) VIRTUAL`
    - `GENERATED ALWAYS AS (...) STORED`
  - Computed columns
  - Identity columns (`AUTO_INCREMENT`)
  - Sequences (emulated)
  - Temporary tables
  - `ENGINE` specification
  - `CHARSET` and `COLLATE`
  - Partitioned tables
  - `CREATE TABLE ... LIKE`
  - `CREATE TABLE ... AS SELECT`

- **31. Altering Tables**
  - `ALTER TABLE`
  - Adding columns
  - Dropping columns
  - Renaming columns (`RENAME COLUMN`, MySQL 8.0+)
  - Changing data types (`MODIFY COLUMN`)
  - Changing column names and types (`CHANGE COLUMN`)
  - Adding constraints
  - Dropping constraints
  - Renaming tables
  - Moving tables between databases (`RENAME TABLE db1.t TO db2.t`)
  - Adding defaults
  - Removing defaults
  - Alter performance
  - Online DDL (`ALGORITHM=INPLACE`, `ALGORITHM=INSTANT`, `ALGORITHM=COPY`)
  - Locking considerations (`LOCK=NONE`, `LOCK=SHARED`, `LOCK=EXCLUSIVE`)
  - Schema modifications in production
  - `pt-online-schema-change` and `gh-ost`

- **32. Dropping Tables**
  - `DROP TABLE`
  - `IF EXISTS`
  - `CASCADE` (not supported in MySQL; foreign keys must be dropped first)
  - `RESTRICT` (default behavior)
  - Dependency handling
  - Data recovery

- **33. Table Constraints**
  - `NOT NULL`
  - `UNIQUE`
  - `PRIMARY KEY`
  - `FOREIGN KEY`
  - `CHECK` (MySQL 8.0.16+)
  - `DEFAULT`
  - `EXCLUDE` (not supported in MySQL)
  - Composite constraints
  - Constraint naming
  - Constraint enforcement
  - Constraint dependency
  - Deferrable constraints (not supported in MySQL; use `SET FOREIGN_KEY_CHECKS=0`)
  - Constraint timing
  - Constraint validation
  - Constraint performance

- **34. Referential Integrity**
  - Parent and child tables
  - Foreign key relationships
  - Referential actions
    - `CASCADE`
    - `SET NULL`
    - `SET DEFAULT` (not supported by InnoDB)
    - `RESTRICT`
    - `NO ACTION`
  - Orphan records
  - Referential integrity validation
  - `FOREIGN_KEY_CHECKS` session variable

- **35. Indexes**
  - What indexes are
  - B-tree indexes (default in InnoDB)
  - Hash indexes (MEMORY engine, InnoDB adaptive hash index)
  - Full-text indexes
  - Spatial indexes (R-tree)
  - `USING BTREE` and `USING HASH`
  - Partial indexes (not directly supported; emulate with generated columns)
  - Expression indexes (via generated columns)
  - Covering indexes
  - Composite indexes
  - Index column order
  - Index cardinality
  - Index selectivity
  - Index maintenance
  - Index bloat
  - Index rebuild
  - Index creation performance
  - Invisible indexes (MySQL 8.0+)
  - Descending indexes (MySQL 8.0+)
  - Functional indexes (MySQL 8.0.13+)

- **36. Views**
  - `CREATE VIEW`
  - Simple views
  - Complex views
  - Updatable views
  - `WITH CHECK OPTION`
  - `ALGORITHM=MERGE` and `ALGORITHM=TEMPTABLE`
  - Materialized views (not natively supported; emulate with tables + triggers/events)
  - View performance
  - View security (`SQL SECURITY DEFINER` / `INVOKER`)
  - View dependencies
  - `information_schema.VIEWS`

- **37. Sequences**
  - MySQL does not have native sequences
  - `AUTO_INCREMENT` as sequence alternative
  - Emulating sequences with tables
  - `LAST_INSERT_ID()`
  - Sequence gaps
  - Identity vs sequence
  - `AUTO_INCREMENT` locking modes (`innodb_autoinc_lock_mode`)

- **38. Synonyms and Aliases**
  - MySQL does not support synonyms
  - Table aliases
  - Column aliases
  - Cross-schema references (`db.table`)
  - Cross-database references
  - Federated tables (`FEDERATED` engine)

---

# V. SQL Data Manipulation Language (DML)

- **39. INSERT Operations**
  - `INSERT INTO`
  - Single-row insert
  - Multi-row insert
  - Explicit column lists
  - `INSERT ... VALUES`
  - `INSERT ... SELECT`
  - `INSERT ... SET`
  - Inserting default values
  - Handling generated identifiers
  - Handling constraint violations
  - `DEFAULT` values
  - `RETURNING` (not supported in MySQL; use `LAST_INSERT_ID()`)
  - `ON DUPLICATE KEY UPDATE`
  - `REPLACE INTO`
  - `INSERT IGNORE`
  - Upsert patterns
  - Bulk inserts
  - Insert performance
  - `LOAD DATA INFILE`

- **40. UPDATE Operations**
  - `UPDATE`
  - Single-row update
  - Multi-row update
  - Single-column update
  - Multi-column update
  - Conditional updates
  - Updating using expressions
  - Update with `JOIN`
  - Update with subqueries
  - Update with CTEs (MySQL 8.0+)
  - Updating from related data
  - `RETURNING` (not supported in MySQL)
  - Update performance
  - Locking considerations
  - Safe update practices (`--safe-updates`, `sql_safe_updates`)
  - `ORDER BY` and `LIMIT` in UPDATE

- **41. DELETE Operations**
  - `DELETE`
  - `DELETE` with `WHERE`
  - `DELETE` with `JOIN`
  - `DELETE` with subqueries
  - `DELETE` with CTEs (MySQL 8.0+)
  - `RETURNING` (not supported in MySQL)
  - Deleting all rows
  - `TRUNCATE`
  - `TRUNCATE` vs `DELETE`
  - `DROP` vs `TRUNCATE` vs `DELETE`
  - Cascading deletes
  - Delete performance
  - Safe deletion strategies
  - `ORDER BY` and `LIMIT` in DELETE
  - Multi-table delete

- **42. MERGE Operations**
  - MySQL does not support `MERGE` statement
  - `INSERT ... ON DUPLICATE KEY UPDATE` as upsert
  - `REPLACE INTO`
  - Synchronization patterns
  - Performance
  - Concurrency

- **43. Bulk Operations**
  - Bulk insert
  - Bulk update
  - Bulk delete
  - `LOAD DATA INFILE`
  - `LOAD DATA LOCAL INFILE`
  - `LOAD XML`
  - `mysqlimport`
  - Staging tables
  - Batch processing
  - Performance optimization
  - `max_allowed_packet` considerations

---

# VI. Querying Fundamentals

- **44. Basic Query Structure**
  - `SELECT`
  - `FROM`
  - `WHERE`
  - `GROUP BY`
  - `HAVING`
  - `ORDER BY`
  - `LIMIT`
  - `OFFSET`
  - `DISTINCT`
  - `DISTINCT ON` (not supported in MySQL)
  - Logical order of execution
  - Physical order of execution

- **45. Selecting Data**
  - Selecting columns
  - Selecting all columns
  - Column aliases
  - Table aliases
  - Expressions in SELECT
  - Literals
  - Calculated columns
  - Computed columns
  - Concatenation (`CONCAT`, `CONCAT_WS`, `||` with `PIPES_AS_CONCAT`)
  - Arithmetic operations
  - String functions
  - Numeric functions
  - Date functions
  - Type casting (`CAST`, `CONVERT`)
  - Query readability and formatting

- **46. Filtering Data**
  - Core Filtering
    - WHERE clause
    - Boolean expressions
  - Comparison Operators
    - `=`
    - `<>`
    - `!=`
    - `<`
    - `>`
    - `<=`
    - `>=`
    - `<=>` (NULL-safe equal)
  - Logical Operators & Precedence
    - AND
    - OR
    - NOT
    - XOR
    - Operator precedence
    - Parentheses
  - Range (BETWEEN)
    - `BETWEEN ... AND ...`
    - `NOT BETWEEN ... AND ...`
  - Set Membership
    - `IN`
    - `NOT IN`
  - Pattern Matching
    - `LIKE`
    - `NOT LIKE`
    - `RLIKE` / `REGEXP`
    - Regular expressions
    - Escaping special characters
  - Null & Distinct Checks
    - `IS NULL`
    - `IS NOT NULL`
    - `IS DISTINCT FROM` (emulated with `<=>`)
  - Subquery Operators
    - `EXISTS`
    - `ANY`
    - `ALL`
  - `FIND_IN_SET()`
  - `STRCMP()`

- **47. Sorting Data**
  - `ORDER BY`
  - Ascending order (`ASC`)
  - Descending order (`DESC`)
  - Multiple sort keys
  - Sort priority
  - Sorting by expressions
  - Sorting by aliases
  - Sorting NULLs (NULLs sort first by default)
  - `NULLS FIRST` / `NULLS LAST` (emulated with `ISNULL()` or `CASE`)
  - Collations
  - Case-sensitive sorting (`BINARY` cast)
  - Locale-aware sorting
  - `ORDER BY ... USING` (MySQL 8.0+)

- **48. Limiting Results**
  - `LIMIT`
  - `OFFSET`
  - `LIMIT offset, count`
  - `FETCH FIRST` (not supported in MySQL)
  - `TOP` (not supported in MySQL)
  - Vendor differences
  - Pagination patterns
  - Offset pagination
  - Keyset pagination
  - Cursor pagination
  - Pagination performance

- **49. Removing Duplicates**
  - `DISTINCT`
  - `DISTINCTROW`
  - Multiple columns
  - Performance considerations
  - Alternatives to DISTINCT
  - `GROUP BY` as alternative

---

# VII. SQL Functions and Expressions

- **50. Conditional Expressions**
  - `CASE`
  - Simple `CASE`
  - Searched `CASE`
  - Nested `CASE`
  - `CASE` in SELECT
  - `CASE` in WHERE
  - `CASE` in ORDER BY
  - `CASE` in GROUP BY
  - `CASE` in aggregates
  - `IF()`
  - `IFNULL()`
  - `NULLIF()`
  - `COALESCE()`
  - `GREATEST()`
  - `LEAST()`
  - Conditional calculations
  - Conditional categorization

- **51. String Functions**
  - Length & Measurement
    - `LENGTH()` (bytes)
    - `CHAR_LENGTH()` / `CHARACTER_LENGTH()` (characters)
    - `BIT_LENGTH()`
  - Case Conversion
    - `UPPER()` / `UCASE()`
    - `LOWER()` / `LCASE()`
  - Trimming & Padding
    - `TRIM()`
    - `LTRIM()`
    - `RTRIM()`
    - `LPAD()`
    - `RPAD()`
  - Extraction & Substrings
    - `SUBSTRING()` / `SUBSTR()`
    - `LEFT()`
    - `RIGHT()`
    - `SUBSTRING_INDEX()`
    - `MID()`
  - Searching & Position
    - `LOCATE()`
    - `INSTR()`
    - `POSITION()`
    - `FIND_IN_SET()`
  - Replacement & Translation
    - `REPLACE()`
    - `INSERT()`
    - `TRANSLATE()` (not native)
  - Concatenation & Aggregation
    - `CONCAT()`
    - `CONCAT_WS()`
    - `GROUP_CONCAT()`
  - Regular Expressions
    - `REGEXP_REPLACE()` (MySQL 8.0+)
    - `REGEXP_SUBSTR()` (MySQL 8.0+)
    - `REGEXP_INSTR()` (MySQL 8.0+)
    - `REGEXP_LIKE()` (MySQL 8.0+)
    - `RLIKE`
  - Comparison & Standards
    - `STRCMP()`
    - `SOUNDEX()`
    - String normalization
    - String comparison
    - Collations

- **52. Numeric Functions**
  - Basic Arithmetic & Sign
    - `ABS()`
    - `SIGN()`
    - `MOD()`
  - Rounding & Truncation
    - `CEIL()` / `CEILING()`
    - `FLOOR()`
    - `ROUND()`
    - `TRUNCATE()`
  - Exponents, Roots & Logarithms
    - `POW()` / `POWER()`
    - `SQRT()`
    - `EXP()`
    - `LN()`
    - `LOG()`
    - `LOG2()`
    - `LOG10()`
  - Constants & Random Generation
    - `PI()`
    - `RAND()`
  - Numeric Behavior & Limits
    - Numeric precision
    - Numeric rounding
    - Numeric overflow
    - `DIV` integer division
    - `%` modulo operator

- **53. Date and Time Functions**
  - Current Date & Time
    - `CURDATE()` / `CURRENT_DATE()`
    - `CURTIME()` / `CURRENT_TIME()`
    - `NOW()` / `CURRENT_TIMESTAMP()`
    - `SYSDATE()`
    - `UTC_DATE()`
    - `UTC_TIME()`
    - `UTC_TIMESTAMP()`
  - Extraction & Truncation
    - `EXTRACT()`
    - `DATE()`
    - `TIME()`
    - `YEAR()`
    - `MONTH()`
    - `DAY()` / `DAYOFMONTH()`
    - `HOUR()`
    - `MINUTE()`
    - `SECOND()`
    - `DAYOFWEEK()`
    - `DAYOFYEAR()`
    - `WEEK()`
    - `WEEKDAY()`
    - `QUARTER()`
    - `LAST_DAY()`
  - Intervals & Arithmetic
    - `DATE_ADD()` / `ADDDATE()`
    - `DATE_SUB()` / `SUBDATE()`
    - `DATEDIFF()`
    - `TIMEDIFF()`
    - `TIMESTAMPDIFF()`
    - `TIMESTAMPADD()`
    - `INTERVAL` expressions
    - Date arithmetic
    - Date differences
    - Date ranges
  - Time Zones
    - `CONVERT_TZ()`
    - `time_zone` session variable
    - Time zone handling
  - Formatting & Parsing
    - `DATE_FORMAT()`
    - `TIME_FORMAT()`
    - `STR_TO_DATE()`
    - `GET_FORMAT()`
  - Calendar & Business Logic
    - Weekday calculations
    - Business day calculations
    - Fiscal calendars

- **54. Casting and Conversion**
  - `CAST()`
  - `CONVERT()`
  - `BINARY` operator
  - Type conversion
  - Implicit conversion
  - Explicit conversion
  - Conversion pitfalls
  - `CAST(... AS CHAR)`, `CAST(... AS SIGNED)`, `CAST(... AS UNSIGNED)`, `CAST(... AS DECIMAL)`, `CAST(... AS DATE)`, `CAST(... AS DATETIME)`, `CAST(... AS TIME)`, `CAST(... AS JSON)`

---

# VIII. Aggregation and Grouping

- **55. Aggregate Functions**
  - Basic Aggregations
    - `COUNT()`
    - `SUM()`
    - `AVG()`
    - `MIN()`
    - `MAX()`
  - Collection & String Aggregations
    - `GROUP_CONCAT()`
    - `JSON_ARRAYAGG()` (MySQL 8.0.14+)
    - `JSON_OBJECTAGG()` (MySQL 8.0.14+)
  - Boolean Aggregations
    - `BIT_AND()`
    - `BIT_OR()`
    - `BIT_XOR()`
  - Statistical & Distribution Functions
    - `STD()` / `STDDEV()` / `STDDEV_POP()`
    - `STDDEV_SAMP()`
    - `VARIANCE()` / `VAR_POP()`
    - `VAR_SAMP()`
  - Aggregate Behavior & Optimization
    - Aggregate NULL handling
    - Aggregate performance
    - `WITH ROLLUP`

- **56. Grouping Data**
  - `GROUP BY`
  - Grouping by columns
  - Grouping by expressions
  - Grouping by multiple columns
  - Grouping sets (emulated with `UNION ALL`)
  - `ROLLUP` (via `WITH ROLLUP`)
  - `CUBE` (not supported; emulate with `UNION ALL`)
  - `GROUPING SETS` (not supported; emulate with `UNION ALL`)
  - `GROUPING()` function (not native; use `GROUPING_ID` emulation)
  - Grouping NULLs
  - Grouping performance
  - `ONLY_FULL_GROUP_BY` SQL mode

- **57. Filtering Groups**
  - `HAVING`
  - `HAVING` vs `WHERE`
  - Aggregate conditions
  - Multiple conditions
  - Multi-condition group filters
  - Performance considerations

- **58. Advanced Aggregation**
  - Conditional aggregation
  - Multiple aggregates
  - Distinct aggregation (`COUNT(DISTINCT ...)`)
  - Ordered-set aggregates (not supported; emulate with window functions)
  - Hypothetical-set aggregates (not supported)
  - User-defined aggregates (not supported in MySQL)
  - Aggregate FILTER clause (not supported; use `CASE` inside aggregate)
  - Window aggregates (MySQL 8.0+)
  - Aggregate over partitions
  - Advanced reporting queries

---

# IX. Relational Joins

- **59. Join Fundamentals**
  - What joins do
  - Purpose of joins
  - Join conditions
  - Join keys
  - Join cardinality
  - Join order
  - Primary-key/foreign-key relationships
  - Join algorithms
    - Nested loop join
    - Block nested loop join
    - Hash join (MySQL 8.0.18+)
    - Batched key access (BKA)
  - Join performance
  - Inner versus outer joins
  - Syntax and operators

- **60. Inner Joins**
  - `INNER JOIN`
  - `JOIN`
  - `STRAIGHT_JOIN`
  - Join conditions
  - Multiple join conditions
  - Joining multiple tables
  - Self joins
  - Equi-joins
  - Non-equi joins
  - `USING` vs `ON`
  - Avoiding unintended Cartesian products
  - Join hints (`STRAIGHT_JOIN`, optimizer hints `/*+ ... */`)

- **61. Outer Joins**
  - `LEFT JOIN`
  - `LEFT OUTER JOIN`
  - `RIGHT JOIN`
  - `RIGHT OUTER JOIN`
  - `FULL JOIN` (not supported; emulate with `LEFT JOIN ... UNION ... RIGHT JOIN`)
  - `FULL OUTER JOIN` (not supported; emulate)
  - NULL padding
  - Join direction
  - Anti-joins
  - Semi-joins
  - Identifying unmatched records
  - Preserving rows from one side

- **62. Cross Joins**
  - `CROSS JOIN`
  - Comma-separated joins
  - Cartesian product
  - Generating combinations
  - Use cases
  - Appropriate versus inappropriate use
  - Performance concerns

- **63. Natural Joins**
  - `NATURAL JOIN`
  - `NATURAL LEFT JOIN`
  - `NATURAL RIGHT JOIN`
  - Implicit join conditions
  - Pitfalls
  - When to avoid

- **64. Self Joins**
  - Self joins
  - Joining a table to itself
  - Hierarchical data
  - Employee-manager relationships
  - Parent-child relationships
  - Graph-like data
  - Recursive relationships

- **65. Advanced Join Patterns**
  - Joining subqueries
  - Joining CTEs (MySQL 8.0+)
  - Joining derived tables
  - Joining lateral subqueries (`LATERAL` not supported; use correlated subqueries)
  - `APPLY` (not supported)
  - Conditional joins
  - Non-equijoin conditions
  - Range joins
  - Join elimination
  - Join reordering
  - Join hints
  - Join filtering strategies
  - `STRAIGHT_JOIN` for join order control
  - Optimizer hints (`/*+ JOIN_ORDER(...) */`, `/*+ JOIN_PREFIX(...) */`, MySQL 8.0+)

---

# X. Subqueries and Nested Queries

- **66. Subquery Fundamentals**
  - What subqueries are
  - Subquery types
    - Scalar subqueries
    - Row subqueries
    - Table subqueries
    - Correlated subqueries
    - Non-correlated subqueries
    - Single-row subqueries
    - Multi-row subqueries
  - Subquery placement
    - SELECT clause
    - FROM clause
    - WHERE clause
    - HAVING clause
    - JOIN clause

- **67. Scalar Subqueries**
  - Single-value subqueries
  - Single-row subqueries
  - Single-column subqueries
  - Multiple-row subqueries
  - Multiple-column subqueries
  - Error handling
  - Performance

- **68. Subqueries with Operators**
  - `IN`
  - `EXISTS`
  - `ANY` / `SOME`
  - `ALL`
  - Comparison operators
  - `NOT IN` and NULL pitfalls

- **69. Correlated Subqueries**
  - Correlated subqueries
  - Outer-query references
  - Outer references
  - Row-by-row logical evaluation
  - Execution model
  - Performance implications
  - Rewriting as joins
  - Appropriate use cases
  - `EXISTS`
  - `NOT EXISTS`
  - `IN`
  - `NOT IN`
  - `ANY`
  - `ALL`

- **70. Derived Tables**
  - Inline views
  - Derived tables
  - Subqueries in FROM
  - Named derived tables
  - Aliasing (required in MySQL)
  - Nesting
  - Multi-stage query construction
  - Performance
  - Materialization
  - Derived table merging (MySQL 5.7+)

- **71. Lateral Subqueries**
  - `LATERAL` (not supported in MySQL)
  - `CROSS JOIN LATERAL` (not supported)
  - `LEFT JOIN LATERAL` (not supported)
  - `APPLY` (not supported)
  - Emulating lateral joins with correlated subqueries
  - Use cases
  - Performance

---

# XI. Common Table Expressions (CTEs)

- **72. CTE Fundamentals**
  - `WITH` clause (MySQL 8.0+)
  - CTE syntax
  - CTE naming
  - CTE scope
  - CTE vs subqueries
  - CTE vs derived tables
  - CTE readability

- **73. Multiple CTEs**
  - Multiple CTEs in one query
  - CTE referencing CTEs
  - CTE dependencies
  - CTE ordering
  - CTE organization
  - CTE chaining

- **74. Recursive CTEs**
  - `WITH RECURSIVE`
  - Recursive CTE structure
  - Anchor member
  - Recursive member
  - Termination conditions
  - Cycle detection
  - Hierarchical queries
  - Tree traversal
  - Graph traversal
  - Bill of materials
  - Organizational charts
  - Path enumeration
  - Recursive CTE performance
  - `cte_max_recursion_depth` system variable

- **75. CTE Materialization**
  - Materialized CTEs
  - Inline CTEs
  - `MATERIALIZED` hint (not supported in MySQL)
  - `NOT MATERIALIZED` hint (not supported in MySQL)
  - Performance implications
  - Optimization fence

- **76. Data-Modifying CTEs**
  - `INSERT` in CTE (not supported in MySQL)
  - `UPDATE` in CTE (not supported in MySQL)
  - `DELETE` in CTE (not supported in MySQL)
  - `RETURNING` clause (not supported in MySQL)
  - Chained data modifications
  - Use cases
  - Pitfalls

---

# XII. Window Functions

- **77. Window Function Fundamentals**
  - What window functions are (MySQL 8.0+)
  - Window vs aggregate functions
  - `OVER` clause
  - Window partitioning
  - Window ordering
  - Window frames
  - Window function execution
  - Window function performance

- **78. Window Partitioning**
  - `PARTITION BY`
  - Multiple partition columns
  - Partition vs GROUP BY
  - Partition ordering
  - Partition scope

- **79. Window Ordering**
  - `ORDER BY` in window
  - Ascending and descending
  - Multiple sort keys
  - NULL ordering
  - Ordering vs partitioning

- **80. Window Frames**
  - `ROWS`
  - `RANGE`
  - `GROUPS` (not supported in MySQL)
  - `BETWEEN`
  - `UNBOUNDED PRECEDING`
  - `UNBOUNDED FOLLOWING`
  - `CURRENT ROW`
  - `n PRECEDING`
  - `n FOLLOWING`
  - Default frames
  - Frame exclusion (not supported in MySQL)

- **81. Ranking Functions**
  - `ROW_NUMBER()`
  - `RANK()`
  - `DENSE_RANK()`
  - `PERCENT_RANK()`
  - `CUME_DIST()`
  - `NTILE()`
  - Ranking use cases
  - Top-N per group
  - Deduplication

- **82. Offset Functions**
  - `LAG()`
  - `LEAD()`
  - `FIRST_VALUE()`
  - `LAST_VALUE()`
  - `NTH_VALUE()`
  - Default values
  - Use cases
  - Time-series analysis

- **83. Aggregate Window Functions**
  - `SUM() OVER`
  - `AVG() OVER`
  - `COUNT() OVER`
  - `MIN() OVER`
  - `MAX() OVER`
  - `GROUP_CONCAT() OVER` (not supported as window function)
  - Running totals
  - Moving averages
  - Cumulative sums
  - Sliding windows

- **84. Advanced Window Patterns**
  - Named windows (`WINDOW w AS (...)`)
  - Window inheritance
  - Window function chaining
  - Gaps and islands
  - Sessionization
  - Cohort analysis
  - Funnel analysis
  - Retention analysis
  - Period-over-period comparisons
  - Sequential event analysis

---

# XIII. Set Operations

- **85. Combining Result Sets**
  - `UNION`
  - `UNION ALL`
  - `UNION DISTINCT`
  - `INTERSECT` (MySQL 8.0.31+)
  - `INTERSECT ALL` (MySQL 8.0.31+)
  - `EXCEPT` (MySQL 8.0.31+)
  - `EXCEPT ALL` (MySQL 8.0.31+)
  - `MINUS` (not supported in MySQL)
  - Set operation rules
  - Column compatibility
  - Data-type compatibility
  - Ordering
  - Performance
  - Duplicate handling

- **86. Set-Based Thinking**
  - Relational operations
  - Difference between row-wise and set-wise logic
  - Replacing procedural loops with set operations
  - Query composition

---

# XIV. Database Design and Data Modeling

- **87. Conceptual Data Modeling**
  - Business requirements
  - Entities
  - Attributes
  - Relationships
  - Cardinality
  - Optionality
  - Business rules

- **88. Entity-Relationship Modeling**
  - ER diagrams
  - Entity identification
  - Relationship modeling
  - One-to-one relationships
  - One-to-many relationships
  - Many-to-many relationships
  - Associative entities
  - Chen notation
  - Crow's foot notation
  - UML notation
  - MySQL Workbench EER diagrams

- **89. Logical Database Design**
  - Relational schema design
  - Attribute selection
  - Key selection
  - Relationship implementation
  - Constraint design

- **90. Physical Database Design**
  - Storage considerations
  - Index strategy
  - Partitioning considerations
  - Tablespaces
  - Physical organization
  - Workload-oriented design
  - InnoDB row format (`DYNAMIC`, `COMPRESSED`)
  - `innodb_file_per_table`

- **91. Keys and Relationships**
  - Primary keys
  - Foreign keys
  - Composite keys
  - Surrogate keys
  - Natural keys
  - Alternate keys
  - Candidate keys
  - One-to-one relationships
  - One-to-many relationships
  - Many-to-many relationships
  - Self-referential relationships
  - Polymorphic relationships

- **92. Table Design Patterns**
  - Lookup tables
  - Junction tables
  - Association tables
  - History tables
  - Audit tables
  - Staging tables
  - Summary tables
  - Aggregate tables
  - Partitioned tables
  - Temporal tables
  - Ledger tables
  - Event tables

- **93. Advanced Data Modeling**
  - Star schema
  - Snowflake schema
  - Galaxy schema
  - Data vault
  - Anchor modeling
  - Entity-attribute-value
  - Document modeling
  - Graph modeling
  - Time-series modeling
  - Geospatial modeling

- **94. Schema Evolution**
  - Additive changes
  - Breaking changes
  - Backward compatibility
  - Forward compatibility
  - Migrations
  - Migration tools
    - Flyway
    - Liquibase
    - Alembic
    - Prisma Migrate
    - Sequelize migrations
    - TypeORM migrations
    - MySQL Shell utilities (`util.dumpSchemas`, `util.loadDump`)
  - Versioned schemas
  - Expand-contract pattern
  - Blue-green deployments
  - Zero-downtime migrations

---

# XV. Database Normalization and Data Integrity

- **95. Functional Dependencies**
  - Functional dependency
  - Full dependency
  - Partial dependency
  - Transitive dependency

- **96. Normal Forms**
  - First Normal Form
    - Atomic attributes
    - Elimination of repeating groups
  - Second Normal Form
    - Removal of partial dependencies
  - Third Normal Form
    - Removal of transitive dependencies
  - Boyce-Codd Normal Form
  - Fourth Normal Form
    - Multivalued dependencies
  - Fifth Normal Form
    - Join dependencies
  - Domain-key normal form

- **97. Denormalization**
  - Purpose of denormalization
  - Performance-oriented denormalization
  - Redundant data
  - Materialized summaries
  - Trade-offs between normalization and performance

- **98. Data Integrity**
  - Entity integrity
  - Referential integrity
  - Domain integrity
  - User-defined integrity
  - Constraint enforcement
  - Validation strategies
  - `CHECK` constraints (MySQL 8.0.16+)
  - Triggers for validation

---

# XVI. Views and Abstraction

- **99. Views**
  - Creating views (`CREATE VIEW`)
  - Querying views
  - Updating views
  - Dropping views
  - Renaming views (`RENAME TABLE` for views, or `ALTER VIEW`)
  - View dependencies

- **100. Advanced Views**
  - Updatable views
  - Read-only views
  - Security views
  - Complex views
  - Nested views
  - `WITH CHECK OPTION`
  - `ALGORITHM` clause
  - Materialized views (not natively supported)

- **101. Materialized Views**
  - Physical result storage (emulated with tables)
  - Refresh strategies
  - Manual refresh
  - Scheduled refresh (via `EVENT` scheduler)
  - Incremental refresh where supported
  - Reporting workloads
  - Emulation with summary tables and triggers

---

# XVII. Indexing and Query Performance

- **102. Index Fundamentals**
  - Purpose of indexes
  - Index lookup
  - Index scan
  - Table scan
  - Index selectivity
  - Index maintenance cost
  - InnoDB clustered index
  - Secondary indexes
  - `EXPLAIN` output

- **103. Common Index Structures**
  - B-tree indexes (default)
  - Hash indexes (MEMORY engine, adaptive hash index)
  - Full-text indexes
  - Spatial indexes (R-tree)
  - Specialized indexes
    - Full-text
    - Spatial
    - JSON (via generated columns)
    - Expression-based indexes (functional indexes, MySQL 8.0.13+)
  - Invisible indexes (MySQL 8.0+)
  - Descending indexes (MySQL 8.0+)

- **104. Index Design**
  - Single-column indexes
  - Composite indexes
  - Column order
  - Covering indexes
  - Unique indexes
  - Partial or filtered indexes (emulated with generated columns)
  - Functional indexes
  - Prefix indexes
  - Index cardinality and selectivity
  - `USE INDEX`, `FORCE INDEX`, `IGNORE INDEX`

- **105. Query Execution**
  - Query parsing
  - Query planning
  - Query optimization
  - Execution plan
  - Cost estimation
  - Cardinality estimation
  - Join algorithms
    - Nested loop join
    - Block nested loop join
    - Hash join (MySQL 8.0.18+)
    - Batched key access (BKA)
  - MySQL optimizer
  - Optimizer trace (`optimizer_trace`)
  - Optimizer hints (`/*+ ... */`, MySQL 8.0+)

- **106. Execution Plan Analysis**
  - `EXPLAIN`
  - `EXPLAIN ANALYZE` (MySQL 8.0.18+)
  - `EXPLAIN FORMAT=JSON`
  - `EXPLAIN FORMAT=TREE` (MySQL 8.0.16+)
  - Query-plan operators
  - Estimated versus actual rows
  - Sequential scans (full table scans)
  - Index scans
  - Sort operations
  - Aggregation operations
  - Join strategies
  - `Handler_read_*` status variables
  - `performance_schema` instrumentation

- **107. Statistics**
  - Table statistics
  - Column statistics
  - Histograms (MySQL 8.0+)
    - `ANALYZE TABLE ... UPDATE HISTOGRAM`
    - `information_schema.COLUMN_STATISTICS`
  - Most common values
  - Distinct values
  - Correlation
  - Statistics targets
  - `ANALYZE TABLE`
  - `OPTIMIZE TABLE`
  - Statistics staleness
  - Statistics impact on plans
  - `innodb_stats_persistent`, `innodb_stats_auto_recalc`

- **108. Query Rewriting**
  - Rewriting subqueries
  - Rewriting joins
  - Rewriting aggregates
  - Removing redundant operations
  - Predicate pushdown
  - Join elimination
  - Constant folding
  - Common subexpression elimination
  - Query simplification
  - Semi-join and anti-join transformations

- **109. Join Optimization**
  - Join order
  - Join algorithms
  - Join hints (`STRAIGHT_JOIN`, optimizer hints)
  - Join elimination
  - Join reordering
  - Hash join vs merge join
  - Nested loop performance
  - Join memory (`join_buffer_size`)
  - Join spill

- **110. SQL Performance Optimization**
  - Avoiding unnecessary columns
  - Reducing unnecessary rows
  - Predicate optimization
  - SARGable predicates
  - Join optimization
  - Index optimization
  - Pagination optimization
  - Query rewriting
  - Avoiding unnecessary subqueries
  - Statistics management

- **111. Query Performance Tuning**
  - Identifying slow queries
  - Slow query log (`slow_query_log`, `long_query_time`)
  - `performance_schema` queries
  - Query profiling (`SHOW PROFILE`, `performance_schema`)
  - Query timing
  - Resource consumption
  - I/O statistics
  - CPU statistics
  - Memory statistics
  - Lock waits (`sys.innodb_lock_waits`)
  - Blocking queries
  - Query cancellation (`KILL QUERY`)
  - Query timeouts (`max_execution_time`)
  - `EXPLAIN ANALYZE`

- **112. Query Anti-Patterns**
  - `SELECT *`
  - N+1 queries
  - Functions on indexed columns
  - Implicit type conversions
  - Non-sargable predicates
  - Overuse of `DISTINCT`
  - Overuse of `OR`
  - Overuse of subqueries
  - Overuse of `UNION`
  - Unnecessary sorting
  - Unnecessary aggregation
  - Large `IN` lists
  - Correlated subqueries in SELECT
  - Wildcard prefixes in `LIKE`
  - OR conditions on different columns

---

# XVIII. Transactions and Concurrency Control

- **113. Transaction Fundamentals**
  - What transactions are
  - Atomic unit of work
  - ACID properties
    - Atomicity
    - Consistency
    - Isolation
    - Durability
  - Transaction boundaries
  - `START TRANSACTION` / `BEGIN`
  - `COMMIT`
  - `ROLLBACK`
  - `SAVEPOINT`
  - `RELEASE SAVEPOINT`
  - `ROLLBACK TO SAVEPOINT`
  - Implicit transactions
  - Autocommit (`autocommit` system variable)
  - Nested transactions (via savepoints)
  - `SET TRANSACTION`
  - `XA` transactions (distributed transactions)

- **114. Transaction Isolation**
  - Read uncommitted
  - Read committed
  - Repeatable read (MySQL InnoDB default)
  - Serializable
  - Snapshot isolation (InnoDB MVCC)
  - Isolation anomalies
    - Dirty reads
    - Non-repeatable reads
    - Phantom reads
    - Lost updates
    - Write skew
  - Isolation level selection (`SET TRANSACTION ISOLATION LEVEL`)
  - Performance implications
  - `transaction_isolation` system variable
  - Gap locks and next-key locks (InnoDB)

- **115. Locking**
  - Lock types
    - Shared locks (`LOCK IN SHARE MODE`, `FOR SHARE`)
    - Exclusive locks (`FOR UPDATE`)
    - Intent locks
  - Lock granularity
    - Row locks
    - Page locks
    - Table locks (`LOCK TABLES`)
    - Database locks
    - Metadata locks (MDL)
  - Lock modes
    - `FOR UPDATE`
    - `FOR SHARE` (MySQL 8.0+)
    - `LOCK IN SHARE MODE`
  - Lock escalation
  - Lock timeouts (`innodb_lock_wait_timeout`)
  - Deadlocks
  - Deadlock detection (`innodb_deadlock_detect`)
  - Deadlock prevention
  - Lock contention
  - Lock monitoring (`information_schema.INNODB_TRX`, `performance_schema.data_locks`, `performance_schema.data_lock_waits`)
  - `SHOW ENGINE INNODB STATUS`

- **116. MVCC**
  - Multi-version concurrency control
  - Snapshots
  - Visibility rules
  - Undo logs
  - Purge
  - History list length
  - Bloat
  - Transaction IDs
  - Long-running transactions
  - Snapshot isolation
  - `innodb_undo_tablespaces`
  - `information_schema.INNODB_TRX`

- **117. Concurrency Patterns**
  - Optimistic concurrency
  - Pessimistic concurrency
  - Version columns
  - Timestamp columns
  - Compare-and-swap
  - Advisory locks (`GET_LOCK()`, `RELEASE_LOCK()`)
  - Named locks
  - Distributed locks
  - Idempotency

- **118. Transaction Design**
  - Short transactions
  - Atomic business operations
  - Error handling
  - Retry strategies
  - Idempotent operations
  - Transaction boundaries

- **119. Advanced Transaction Topics**
  - Two-phase commit
  - Distributed transactions (`XA`)
  - Savepoints
  - Autonomous transactions (not supported in MySQL)
  - Transaction log (InnoDB redo log)
  - Write-ahead log
  - Durability guarantees (`innodb_flush_log_at_trx_commit`)
  - Crash recovery
  - Point-in-time recovery (binary logs)

---

# XIX. Stored Procedures, Functions, and Programmability

- **120. Functions**
  - `CREATE FUNCTION`
  - Function parameters
  - Function return types
  - SQL functions
  - Deterministic functions (`DETERMINISTIC`)
  - Non-deterministic functions
  - `READS SQL DATA`
  - `MODIFIES SQL DATA`
  - `NO SQL`
  - Scalar functions
  - Table-valued functions (via stored procedures)
  - Function overloading (not supported)
  - Function security (`SQL SECURITY DEFINER` / `INVOKER`)
  - Function limitations
  - Function performance
  - `DROP FUNCTION`

- **121. Stored Procedures**
  - `CREATE PROCEDURE`
  - Procedure parameters
  - `IN` parameters
  - `OUT` parameters
  - `INOUT` parameters
  - `CALL`
  - Procedure vs function
  - Transaction control
  - Error handling
  - Procedure security (`SQL SECURITY DEFINER` / `INVOKER`)
  - `DROP PROCEDURE`
  - `SHOW CREATE PROCEDURE`
  - `information_schema.ROUTINES`

- **122. MySQL Stored Programs (SQL/PSM)**
  - Block structure (`BEGIN ... END`)
  - Declarations (`DECLARE`)
  - Variables
  - Constants
  - Control structures
    - `IF ... THEN ... ELSEIF ... ELSE ... END IF`
    - `CASE ... WHEN ... END CASE`
    - `LOOP ... END LOOP`
    - `WHILE ... DO ... END WHILE`
    - `REPEAT ... UNTIL ... END REPEAT`
    - `FOR` (not native; use `WHILE` or cursors)
    - `LEAVE` (equivalent to `EXIT`)
    - `ITERATE` (equivalent to `CONTINUE`)
  - Cursors (`DECLARE ... CURSOR`, `OPEN`, `FETCH`, `CLOSE`)
  - Handlers (`DECLARE ... HANDLER FOR`)
  - Exceptions (`SIGNAL SQLSTATE`, `RESIGNAL`)
  - `GET DIAGNOSTICS`
  - Dynamic SQL (`PREPARE`, `EXECUTE`, `DEALLOCATE PREPARE`)
  - Triggers
  - Debugging
  - `SHOW WARNINGS`, `SHOW ERRORS`

- **123. SQL Procedural Extensions**
  - Variables
  - Conditional statements
  - Loops
  - Cursors
  - Exception handling
  - Dynamic SQL
  - `SIGNAL` and `RESIGNAL`

- **124. Dynamic SQL**
  - Dynamic statement construction
  - `PREPARE`
  - `EXECUTE`
  - `DEALLOCATE PREPARE`
  - Parameterized dynamic SQL (`USING`)
  - Safe dynamic SQL
  - SQL injection risks
  - Metadata-driven queries

- **125. User-Defined Types**
  - MySQL does not support user-defined types
  - `ENUM` and `SET` as alternatives
  - Composite types (emulated with multiple columns or JSON)
  - Domain types (emulated with `CHECK` constraints)
  - Type casts

- **126. User-Defined Aggregates**
  - MySQL does not support user-defined aggregates
  - Emulating with stored functions and `GROUP_CONCAT`
  - Aggregate performance

---

# XX. Triggers and Event-Driven Database Logic

- **127. Trigger Fundamentals**
  - Trigger definition
  - `CREATE TRIGGER`
  - `BEFORE` triggers
  - `AFTER` triggers
  - `INSTEAD OF` triggers (not supported in MySQL)
  - Row-level triggers (`FOR EACH ROW`)
  - Statement-level triggers (not supported in MySQL)
  - Trigger functions
  - `NEW`
  - `OLD`
  - Trigger ordering (`FOLLOWS`, `PRECEDES`, MySQL 5.7+)
  - Trigger recursion (`max_sp_recursion_depth`)
  - Trigger performance
  - `DROP TRIGGER`
  - `information_schema.TRIGGERS`

- **128. Trigger Applications**
  - Auditing
  - Automatic timestamps
  - Derived data maintenance
  - Business-rule enforcement
  - Change tracking
  - Validation triggers

- **129. Trigger Risks**
  - Hidden side effects
  - Recursive triggers
  - Performance overhead
  - Debugging complexity
  - Transaction interactions
  - Maintainability concerns
  - Trigger limitations in MySQL (no statement-level triggers, no `INSTEAD OF`)

---

# XXI. Recursive and Hierarchical SQL

- **130. Hierarchical Data**
  - Parent-child relationships
  - Organizational structures
  - Category trees
  - Folder structures
  - Bill-of-materials structures

- **131. Recursive CTEs**
  - `WITH RECURSIVE` (MySQL 8.0+)
  - Anchor member
  - Recursive member
  - Termination conditions
  - Depth tracking
  - Path construction
  - `cte_max_recursion_depth`

- **132. Hierarchical Query Techniques**
  - Traversing downward
  - Traversing upward
  - Tree depth
  - Ancestor identification
  - Descendant identification
  - Cycle detection
  - Nested set model
  - Path enumeration
  - Adjacency list model
  - Closure table model

---

# XXII. JSON, XML, and Semi-Structured Data

- **133. JSON in SQL**
  - `JSON` data type (MySQL 5.7+)
  - JSON document structure
  - JSON extraction
    - `JSON_EXTRACT()`
    - `->` operator
    - `->>` operator
  - JSON construction
    - `JSON_OBJECT()`
    - `JSON_ARRAY()`
  - JSON modification
    - `JSON_SET()`
    - `JSON_INSERT()`
    - `JSON_REPLACE()`
    - `JSON_REMOVE()`
  - JSON aggregation
    - `JSON_ARRAYAGG()`
    - `JSON_OBJECTAGG()`
  - JSON indexing
    - Generated columns
    - Functional indexes
  - JSON path expressions (`$`, `$.key`, `$[0]`, `$[*]`)
  - JSON operators
    - `->`
    - `->>`
  - JSON functions
    - `JSON_EXTRACT`
    - `JSON_UNQUOTE`
    - `JSON_CONTAINS`
    - `JSON_CONTAINS_PATH`
    - `JSON_KEYS`
    - `JSON_LENGTH`
    - `JSON_TYPE`
    - `JSON_VALID`
    - `JSON_SEARCH`
    - `JSON_MERGE_PATCH`
    - `JSON_MERGE_PRESERVE`
    - `JSON_PRETTY`
    - `JSON_TABLE()` (MySQL 8.0+)
  - JSON querying
  - JSON updates
  - JSON vs relational modeling

- **134. XML in SQL**
  - XML storage as `TEXT`
  - `ExtractValue()`
  - `UpdateXML()`
  - XML querying
  - XML extraction
  - XML transformation
  - XML indexing (via generated columns)

- **135. Semi-Structured Data Modeling**
  - Relational versus document-style storage
  - Hybrid schemas
  - JSON normalization
  - Flexible attributes
  - Trade-offs in schema flexibility

- **136. Enterprise JSON/XML Storage and Query Performance**
  - Storage overhead of semi-structured formats
  - Textual parsing overhead vs binary formats
  - Partial updates vs full document rewrites
  - Inline vs out-of-line storage for large documents
  - Query optimization: Pushdown predicates into JSON documents
  - `JSON_TABLE()` for relational views of JSON

- **137. Modern Big Data Semi-Structured Formats**
  - Binary JSON variants
  - Row-based vs columnar semi-structured data
  - External tables and Object Storage querying
  - MySQL HeatWave and analytical processing

- **138. Array Support**
  - MySQL does not have native array types
  - Emulating arrays with JSON
  - `JSON_ARRAY()`, `JSON_ARRAYAGG()`
  - `JSON_LENGTH()`, `JSON_EXTRACT()`
  - `FIND_IN_SET()` for comma-separated lists
  - `GROUP_CONCAT()` for aggregation
  - Array performance in MySQL

- **139. Full-Text Search**
  - Full-text search concepts
  - `FULLTEXT` indexes
  - Documents
  - Tokens
  - Stop words (`innodb_ft_stopword_table`)
  - Stemming
  - `MATCH ... AGAINST`
  - Natural language mode (`IN NATURAL LANGUAGE MODE`)
  - Boolean mode (`IN BOOLEAN MODE`)
  - Query expansion mode (`WITH QUERY EXPANSION`)
  - Boolean operators (`+`, `-`, `*`, `"`, `()`, `~`, `<`, `>`)
  - Ranking
  - `MATCH()` score
  - `innodb_ft_min_token_size`, `innodb_ft_max_token_size`
  - Full-text search performance
  - Full-text indexes on InnoDB and MyISAM

- **140. Regular Expressions**
  - POSIX regular expressions
  - `REGEXP`
  - `RLIKE`
  - `NOT REGEXP`
  - `REGEXP_LIKE()` (MySQL 8.0+)
  - `REGEXP_REPLACE()` (MySQL 8.0+)
  - `REGEXP_SUBSTR()` (MySQL 8.0+)
  - `REGEXP_INSTR()` (MySQL 8.0+)
  - Capture groups
  - Backreferences
  - Lookahead
  - Lookbehind
  - Regular expression performance
  - `regexp_stack_limit`, `regexp_time_limit`

- **141. Geospatial Data**
  - Geospatial data types
  - `POINT`
  - `LINESTRING`
  - `POLYGON`
  - `MULTIPOINT`
  - `MULTILINESTRING`
  - `MULTIPOLYGON`
  - `GEOMETRYCOLLECTION`
  - `GEOMETRY`
  - SRID (Spatial Reference System Identifier)
  - Spatial indexes (R-tree)
  - Spatial operations
    - `ST_Distance()`
    - `ST_Contains()`
    - `ST_Intersects()`
    - `ST_Within()`
    - `ST_Area()`
    - `ST_Length()`
    - `ST_Buffer()`
    - `ST_Union()`
    - `ST_Intersection()`
  - Distance calculations
  - Containment
  - Intersection
  - Nearest neighbor
  - Geospatial queries
  - `SPATIAL` indexes (InnoDB, MyISAM)

- **142. Range Types**
  - MySQL does not have native range types
  - Emulating ranges with two columns
  - Range operators
  - Range indexes
  - Exclusion constraints (not supported)
  - Temporal data
  - Scheduling applications

---

# XXIII. Data Import, Export, and Transformation

- **143. Data Loading**
  - Bulk inserts
  - Batch inserts
  - CSV import
  - `LOAD DATA INFILE`
  - `LOAD DATA LOCAL INFILE`
  - `LOAD XML`
  - `mysqlimport`
  - External data loading
  - Bulk-loading utilities
  - `max_allowed_packet`
  - `secure_file_priv`

- **144. Data Export**
  - Query result export
  - CSV
  - JSON
  - SQL dump (`mysqldump`)
  - `SELECT ... INTO OUTFILE`
  - `SELECT ... INTO DUMPFILE`
  - Database backup export
  - `mysqlpump` (deprecated)
  - MySQL Shell `util.dumpSchemas()`, `util.dumpInstance()`

- **145. Data Transformation**
  - Type conversion
  - String transformation
  - Data cleansing
  - Deduplication
  - Standardization
  - Data validation

- **146. ETL-Oriented SQL**
  - Extract
  - Transform
  - Load
  - Staging tables
  - Transformation pipelines
  - Incremental loading
  - Change detection
  - Change Data Capture (CDC)
  - Binary log-based replication

---

# XXIV. SQL for Data Analytics

- **147. Analytical Query Design**
  - Exploratory queries
  - Descriptive statistics
  - Aggregation
  - Segmentation
  - Trend analysis

- **148. Business Metrics**
  - Revenue
  - Profit
  - Average order value
  - Customer counts
  - Conversion rates
  - Retention
  - Churn
  - Growth rates

- **149. Time-Series Analysis**
  - Daily aggregation
  - Weekly aggregation
  - Monthly aggregation
  - Year-over-year comparison
  - Month-over-month comparison
  - Rolling averages
  - Cumulative totals

- **150. Advanced Analytics**
  - Cohort analysis
  - Funnel analysis
  - Customer segmentation
  - Ranking analysis
  - Statistical aggregation
  - Percentiles (emulated with window functions)
  - Median calculations
  - Distribution analysis
  - RFM analysis
  - Market basket analysis
  - Anomaly detection
  - Forecasting

---

# XXV. MySQL Security

- **151. Database Security Fundamentals**
  - Authentication
  - Authorization
  - Least privilege
  - Security boundaries
  - Database users
  - Roles (MySQL 8.0+)
  - Groups
  - Role membership
  - Role inheritance
  - Security best practices

- **152. Authentication**
  - Database authentication
  - Password authentication
  - `caching_sha2_password` (default in MySQL 8.0+)
  - `mysql_native_password` (deprecated)
  - `sha256_password`
  - `auth_socket` plugin
  - `mysql_no_login` plugin
  - LDAP authentication (Enterprise)
  - PAM authentication (Enterprise)
  - Certificate authentication
  - Authentication configuration
  - `CREATE USER`, `ALTER USER`
  - Password policies (`validate_password` component)
  - `mysql_secure_installation`

- **153. Privileges**
  - `GRANT`
  - `REVOKE`
  - Object privileges
  - Schema privileges
  - Database privileges
  - Global privileges
  - Role-based access control (MySQL 8.0+)
  - Privileges
    - `SELECT`
    - `INSERT`
    - `UPDATE`
    - `DELETE`
    - `TRUNCATE` (not a separate privilege; covered by `DROP`)
    - `REFERENCES`
    - `TRIGGER`
    - `CREATE`
    - `CONNECT` (not a MySQL privilege)
    - `TEMPORARY`
    - `EXECUTE`
    - `USAGE`
    - `ALL PRIVILEGES`
  - Default privileges
  - Object ownership
  - Privilege inheritance
  - Column-level privileges
  - Row-level security (not natively supported; emulate with views)
  - `SHOW GRANTS`
  - `information_schema.USER_PRIVILEGES`, `SCHEMA_PRIVILEGES`, `TABLE_PRIVILEGES`, `COLUMN_PRIVILEGES`

- **154. Row-Level Security**
  - MySQL does not have native row-level security
  - Emulating with views
  - Emulating with application logic
  - User-specific visibility
  - Tenant isolation
  - Policy-based filtering
  - Multi-tenant access control
  - Performance implications

- **155. SQL Injection Prevention**
  - What SQL injection is
  - Injection fundamentals
  - Injection vectors
  - Unsafe string concatenation
  - Dynamic SQL
  - Parameterized queries
  - Prepared statements
  - Stored procedures
  - Input validation
  - Escaping (`mysql_real_escape_string`)
  - ORM protection
  - Least privilege
  - Detection
  - Prevention

- **156. Data Protection and Encryption**
  - Sensitive-data handling
  - Encryption at rest (InnoDB tablespace encryption)
  - Encryption in transit (TLS/SSL)
  - Column-level encryption
  - `AES_ENCRYPT()`, `AES_DECRYPT()`
  - `SHA2()`, `SHA1()`, `MD5()`
  - Hashing
  - Salting
  - Key management
  - Key rotation
  - Transparent data encryption (Enterprise)
  - Disk encryption
  - Data masking
  - Data anonymization

- **157. Auditing**
  - Audit logging
  - MySQL Enterprise Audit
  - Audit tables
  - Triggers for auditing
  - General query log (`general_log`)
  - Slow query log
  - Error log
  - Binary log
  - Logging configuration
  - Log retention
  - Log analysis
  - Compliance requirements
  - PII protection
  - Access logging

- **158. Security Best Practices**
  - Least privilege
  - Separation of duties
  - Strong authentication
  - Password policies
  - Network isolation
  - Firewall rules
  - SSL/TLS
  - Connection limits (`max_connections`, `max_user_connections`)
  - Session timeouts (`wait_timeout`, `interactive_timeout`)
  - Secure configuration
  - Patch management
  - Vulnerability scanning
  - Penetration testing

---

# XXVI. Backup, Recovery, and High Availability

- **159. Backup Fundamentals**
  - Full backups
  - Incremental backups
  - Differential backups
  - Logical backups
  - Physical backups
  - `mysqldump`
  - `mysqlpump` (deprecated)
  - `mysqlbackup` (Enterprise)
  - `Percona XtraBackup`
  - MySQL Shell `util.dumpInstance()`, `util.dumpSchemas()`
  - Binary log backups
  - Backup types and strategies
  - `SELECT ... INTO OUTFILE`

- **160. Recovery**
  - Restore operations
  - Point-in-time recovery (binary logs)
  - Recovery objectives
    - RPO
    - RTO
  - Recovery targets
  - Recovery testing
  - Disaster recovery planning
  - `mysqlbinlog` for point-in-time recovery

- **161. High Availability**
  - High availability concepts
  - Replication
  - Primary-replica architectures
  - Failover
  - Automatic failover
  - Load balancing
  - Read replicas
  - Write splitting
  - Health checks
  - Split-brain prevention
  - Quorum
  - Consensus
  - HA tools
    - MySQL Group Replication
    - MySQL InnoDB Cluster
    - MySQL Router
    - MySQL Shell AdminAPI
    - Orchestrator
    - ProxySQL
    - MHA (Master High Availability)
    - Galera Cluster (Percona XtraDB Cluster, MariaDB Galera)
    - Keepalived
    - HAProxy

- **162. Database Maintenance**
  - Statistics updates (`ANALYZE TABLE`)
  - Index maintenance
  - Table maintenance (`OPTIMIZE TABLE`)
  - Storage monitoring
  - Log management
  - Bloat management
  - Binary log archiving
  - Backup retention
  - Backup encryption
  - Backup monitoring
  - Disaster recovery
  - RPO and RTO
  - `innodb_purge_threads`, undo log management

---

# XXVII. Replication, Partitioning, and Scalability

- **163. Database Replication**
  - Why replication matters
  - Replication types
    - Asynchronous replication (default)
    - Semi-synchronous replication
    - Group Replication (MySQL 5.7+)
    - Statement-based replication (SBR)
    - Row-based replication (RBR)
    - Mixed-based replication (MBR)
    - GTID-based replication
    - Multi-source replication
    - Cascading replication
    - Delayed replication
  - Primary-replica topology
  - Multi-primary topology (Group Replication)
  - Replication filters
  - Replication lag
  - Failover strategies
  - Promotion
  - Replication monitoring (`SHOW REPLICA STATUS`, `performance_schema.replication_*`)
  - Replication conflicts
  - `CHANGE REPLICATION SOURCE TO` (MySQL 8.0.23+)
  - `START REPLICA`, `STOP REPLICA`
  - Binary log format (`binlog_format`)

- **164. Table Partitioning**
  - Why partitioning matters
  - Partitioning strategies
    - Range partitioning (`PARTITION BY RANGE`)
    - List partitioning (`PARTITION BY LIST`)
    - Hash partitioning (`PARTITION BY HASH`)
    - Key partitioning (`PARTITION BY KEY`)
    - Composite partitioning (`PARTITION BY RANGE ... SUBPARTITION BY HASH`)
  - Partition pruning
  - Partition-wise joins
  - Partition-wise aggregates
  - Partition maintenance
  - Partition creation (`ALTER TABLE ... ADD PARTITION`)
  - Partition detachment (`ALTER TABLE ... DROP PARTITION`, `TRUNCATE PARTITION`)
  - Partition indexes
  - Partition constraints
  - Partition performance
  - Declarative partitioning
  - Partition limitations in MySQL (no global indexes, foreign keys not supported on partitioned tables until MySQL 8.0)
  - `information_schema.PARTITIONS`

- **165. Connection Pooling**
  - Why connection pooling matters
  - Connection overhead
  - Pool sizing
  - Pool modes
    - Session pooling
    - Transaction pooling
    - Statement pooling
  - ProxySQL
  - MySQL Router
  - Application-level pooling (HikariCP, c3p0, etc.)
  - Connection limits (`max_connections`)
  - Connection leaks
  - Pool monitoring

- **166. Horizontal and Vertical Scaling**
  - Vertical scaling
  - Horizontal scaling
  - Read scaling
  - Write scaling
  - Sharding
    - Range sharding
    - Hash sharding
    - Directory sharding
    - Geographic sharding
  - Shard keys
  - Shard rebalancing
  - Cross-shard queries
  - Distributed transactions
  - Sharding middleware
    - ProxySQL
    - Vitess
    - ShardingSphere
    - MySQL Router (limited)
    - Citus (PostgreSQL)
  - Application-level sharding
  - Sharding trade-offs
  - Distributed SQL considerations
  - MySQL NDB Cluster

- **167. Scalability Engineering**
  - Workload analysis
  - Connection management
  - Query concurrency
  - Caching strategies (query cache removed in MySQL 8.0; use application-level caching)
  - Read/write separation
  - Capacity planning

---

# XXVIII. Data Warehousing and Analytics

- **168. Data Warehousing Fundamentals**
  - OLTP vs OLAP
  - Data warehouse
  - Data mart
  - Data lake
  - Data lakehouse
  - ETL
  - ELT
  - Data pipelines
  - Batch processing
  - Stream processing
  - MySQL as a data warehouse (limitations)
  - MySQL HeatWave

- **169. Dimensional Modeling**
  - Facts
  - Dimensions
  - Star schema
  - Snowflake schema
  - Galaxy schema
  - Slowly changing dimensions
    - Type 0
    - Type 1
    - Type 2
    - Type 3
    - Type 4
    - Type 6
  - Conformed dimensions
  - Junk dimensions
  - Degenerate dimensions
  - Bridge tables
  - Factless fact tables
  - Grain
  - Measures
  - Additive measures
  - Semi-additive measures
  - Non-additive measures

- **170. Analytical SQL**
  - Grouping sets (emulated)
  - Rollup (`WITH ROLLUP`)
  - Cube (emulated)
  - Window functions (MySQL 8.0+)
  - Ranking
  - Running totals
  - Moving averages
  - Percentiles (emulated)
  - Median
  - Mode
  - Cohort analysis
  - Funnel analysis
  - Retention analysis
  - Sessionization
  - Gaps and islands
  - Time-series analysis
  - Pivot and unpivot (emulated with `CASE` and `UNION`)

- **171. Columnar Databases**
  - Row-oriented vs column-oriented
  - Columnar storage
  - Compression
  - Vectorized execution
  - Columnar databases
    - MySQL HeatWave
    - Redshift
    - BigQuery
    - Snowflake
    - ClickHouse
    - DuckDB
    - Vertica
  - Columnar performance
  - Columnar use cases

- **172. Analytical Patterns**
  - Time-series analysis
  - Cohort analysis
  - Funnel analysis
  - Attribution modeling
  - Customer segmentation
  - RFM analysis
  - Market basket analysis
  - Anomaly detection
  - Forecasting
  - Materialized views (emulated)
  - Summary tables
  - Pre-aggregation

---

# XXIX. Distributed SQL and NewSQL

- **173. Distributed SQL Fundamentals**
  - Distributed databases
  - CAP theorem
  - Consistency models
  - Availability
  - Partition tolerance
  - ACID in distributed systems
  - BASE
  - Eventual consistency
  - Strong consistency
  - Linearizability
  - Serializability

- **174. Distributed SQL Databases**
  - MySQL NDB Cluster
  - MySQL Group Replication (multi-primary)
  - MySQL InnoDB Cluster
  - Vitess (MySQL sharding)
  - ProxySQL (query routing)
  - TiDB (MySQL-compatible)
  - CockroachDB
  - YugabyteDB
  - PlanetScale (Vitess-based)
  - Distributed SQL characteristics
  - Distributed transactions
  - Distributed query planning
  - Distributed joins
  - Distributed aggregates

- **175. NewSQL**
  - NewSQL characteristics
  - NewSQL vs traditional SQL
  - NewSQL vs NoSQL
  - Scalability
  - Consistency
  - Performance
  - Use cases

- **176. Geo-Distribution**
  - Geo-distributed databases
  - Replication across regions
  - Latency considerations
  - Data residency
  - Compliance
  - Conflict resolution
  - Multi-region writes
  - Read locality

---

# XXX. SQL in Application Development

- **177. SQL in Backend Applications**
  - Database connectivity
  - Connection strings
  - Connection management
  - Connection pools
  - Query execution
  - Result mapping
  - Error handling
  - Transaction management
  - Transactions from applications
  - Retry logic
  - Timeouts
  - Prepared statements
  - Parameterized queries

- **178. Database APIs**
  - JDBC
  - ODBC
  - ADO.NET
  - Python database drivers (`mysqlclient`, `PyMySQL`, `mysql-connector-python`)
  - Node.js drivers (`mysql2`, `mysql`)
  - PHP drivers (`mysqli`, `PDO_MYSQL`)
  - Go drivers (`go-sql-driver/mysql`)
  - ORM database integrations

- **179. ORMs and Query Builders**
  - What ORMs are
  - ORM benefits
  - ORM drawbacks
  - ORM performance
  - Entity mapping
  - Relationship mapping
  - Relationships
  - Lazy loading
  - Eager loading
  - N+1 problem
  - Migrations
  - Query builders
  - Raw SQL in ORMs
  - ORM-generated SQL
  - ORM tools
    - Prisma
    - Sequelize
    - TypeORM
    - Drizzle
    - Hibernate
    - SQLAlchemy
    - ActiveRecord
    - Entity Framework
    - Eloquent (Laravel)
    - GORM (Go)

- **180. Application-Level Data Access**
  - Repository patterns
  - Data-access layers
  - Service layers
  - Transaction boundaries
  - Connection lifecycle
  - Error handling

- **181. Database Migrations**
  - Migration fundamentals
  - Versioned migrations
  - Migration tools
  - Forward migrations
  - Rollback migrations
  - Data migrations
  - Schema migrations
  - Zero-downtime migrations
  - Expand-contract pattern
  - Migration testing
  - Migration deployment
  - MySQL Shell upgrade utilities

- **182. Testing Database Code**
  - Unit testing SQL
  - Integration testing SQL
  - Test databases
  - Test data
  - Fixtures
  - Factories
  - Test isolation
  - Transaction rollback
  - Testcontainers
  - Database seeding
  - Snapshot testing

- **183. Database DevOps**
  - Database version control
  - Database CI/CD
  - Database deployments
  - Database rollbacks
  - Database monitoring
  - Database automation
  - Infrastructure as code
  - Database as code

---

# XXXI. SQL Debugging, Troubleshooting, and Testing

- **184. Syntax Debugging**
  - Missing keywords
  - Invalid identifiers
  - Incorrect commas
  - Parentheses mismatches
  - Incorrect statement structure

- **185. Logical Debugging**
  - Incorrect joins
  - Duplicate rows
  - Incorrect filters
  - NULL-related errors
  - Incorrect aggregation
  - Wrong grouping
  - Incorrect subquery logic

- **186. Performance Debugging**
  - Slow queries
  - Full table scans
  - Missing indexes
  - Poor cardinality estimates
  - Expensive joins
  - Excessive sorting
  - Inefficient pagination

- **187. Transaction Debugging**
  - Lock contention
  - Deadlocks
  - Long-running transactions
  - Isolation-related anomalies
  - Uncommitted transactions

- **188. Data Debugging**
  - Duplicate data
  - Orphaned records
  - Invalid foreign keys
  - Inconsistent values
  - Unexpected NULLs
  - Incorrect derived data

- **189. Monitoring and Diagnostics**
  - Identifying blockers
  - Analyzing block chains
  - Deadlock logging
  - Under-the-hood sizing
  - `performance_schema`
  - `sys` schema
  - `SHOW ENGINE INNODB STATUS`

- **190. Query Testing**
  - Positive test cases
  - Negative test cases
  - Boundary testing
  - NULL testing
  - Empty-result testing
  - Duplicate-data testing

- **191. Database Testing**
  - Constraint testing
  - Referential-integrity testing
  - Trigger testing
  - Stored-procedure testing
  - Function testing
  - Transaction testing

- **192. Performance Testing**
  - Benchmarking
  - Load testing
  - Stress testing
  - Concurrency testing
  - Execution-plan comparison
  - `sysbench`

- **193. Data Quality Testing**
  - Completeness
  - Accuracy
  - Consistency
  - Uniqueness
  - Validity
  - Referential integrity

---

# XXXII. SQL Development Practices

- **194. SQL Code Quality**
  - Consistent formatting
  - Descriptive aliases
  - Meaningful naming
  - Explicit column lists
  - Avoidance of unnecessary complexity
  - Maintainable query structure

- **195. SQL Documentation**
  - Schema documentation
  - Data dictionaries
  - Relationship documentation
  - Query documentation
  - Business-rule documentation
  - `SHOW CREATE TABLE`
  - `information_schema`

- **196. Database Change Management**
  - Schema migrations
  - Version control
  - Migration scripts
  - Rollback scripts
  - Deployment sequencing

- **197. SQL Collaboration**
  - Code review
  - SQL style standards
  - Branching strategies
  - Migration review
  - Change auditing

---

# XXXIII. Advanced Database Architecture

- **198. OLTP Systems**
  - Transaction-oriented workloads
  - Normalized schemas
  - High concurrency
  - Low-latency operations
  - InnoDB tuning for OLTP

- **199. OLAP Systems**
  - Analytical workloads
  - Large scans
  - Aggregations
  - Dimensional modeling
  - MySQL HeatWave for OLAP

- **200. Data Warehousing**
  - Fact tables
  - Dimension tables
  - Star schema
  - Snowflake schema
  - Slowly changing dimensions
  - Surrogate keys

- **201. Modern Data Architectures**
  - Data lakes
  - Lakehouses
  - Data warehouses
  - Hybrid architectures
  - Analytical databases
  - MySQL HeatWave Lakehouse

---

# XXXIV. Advanced SQL Engineering

- **202. Complex Query Optimization**
  - Query decomposition
  - Predicate pushdown
  - Join-order optimization
  - Subquery rewriting
  - Aggregation optimization
  - Materialization strategies
  - Optimizer hints (MySQL 8.0+)

- **203. Advanced Index Engineering**
  - Composite index design
  - Covering indexes
  - Partial indexes (emulated)
  - Expression indexes (functional indexes)
  - Index-only scans
  - Index maintenance trade-offs
  - Invisible indexes
  - Descending indexes

- **204. Advanced Transaction Engineering**
  - Isolation tuning
  - Lock management
  - Deadlock analysis
  - Retry mechanisms
  - Distributed transaction considerations (`XA`)

- **205. Advanced Distributed Database Concepts**
  - Replication consistency
  - Partitioning
  - Sharding
  - Consensus concepts
  - Distributed transactions
  - CAP-related trade-offs
  - MySQL Group Replication consistency modes

---

# XXXV. Performance and Scalability Operations

- **206. Performance Fundamentals**
  - Latency
  - Throughput
  - Concurrency
  - Resource utilization
  - Bottleneck identification
  - Capacity planning
  - SLOs and SLAs

- **207. Database Configuration**
  - Memory configuration
    - `innodb_buffer_pool_size`
    - `innodb_log_buffer_size`
    - `key_buffer_size`
    - `sort_buffer_size`
    - `join_buffer_size`
    - `read_buffer_size`
    - `read_rnd_buffer_size`
    - `tmp_table_size`
    - `max_heap_table_size`
  - CPU configuration
  - I/O configuration
    - `innodb_io_capacity`
    - `innodb_io_capacity_max`
    - `innodb_flush_method`
    - `innodb_flush_log_at_trx_commit`
  - Connection configuration
    - `max_connections`
    - `max_user_connections`
    - Connection pooling
    - `thread_cache_size`
  - Binary log configuration
    - `log_bin`
    - `binlog_format`
    - `sync_binlog`
  - InnoDB redo log configuration
    - `innodb_log_file_size`
    - `innodb_log_files_in_group`
  - Checkpoint configuration
    - `innodb_max_dirty_pages_pct`
  - InnoDB purge configuration
    - `innodb_purge_threads`
  - Query planner configuration
    - `optimizer_switch`
    - `eq_range_index_dive_limit`
  - Logging configuration
  - `performance_schema` configuration

- **208. Monitoring and Observability**
  - Database metrics
    - Connections
    - Transactions
    - Queries
    - Locks
    - Waits
    - I/O
    - CPU
    - Memory
    - Disk
  - System catalogs
    - `information_schema`
    - `performance_schema`
    - `sys` schema
    - `SHOW STATUS`
    - `SHOW ENGINE INNODB STATUS`
    - `SHOW PROCESSLIST`
    - `performance_schema.events_statements_summary_by_digest`
    - `sys.statement_analysis`
    - `sys.innodb_lock_waits`
    - `sys.schema_table_statistics`
    - `sys.schema_index_statistics`
    - `performance_schema.replication_*`
  - Logging
    - Slow query log
    - Error log
    - General query log
    - Binary log
    - Audit log
    - Log analysis
  - Monitoring tools
    - MySQL Workbench
    - MySQL Enterprise Monitor
    - Percona Monitoring and Management (PMM)
    - Prometheus + `mysqld_exporter`
    - Grafana
    - Datadog
    - New Relic
    - SolarWinds
    - Nagios
    - Zabbix
  - Alerting
  - Dashboards
  - Capacity trending

---

# XXXVI. SQL Tools and Ecosystem

- **209. SQL Clients**
  - MySQL Workbench
  - MySQL Shell
  - phpMyAdmin
  - Adminer
  - DBeaver
  - DataGrip
  - TablePlus
  - Beekeeper Studio
  - HeidiSQL
  - Navicat
  - Sequel Pro / Sequel Ace
  - Azure Data Studio
  - MySQL for Visual Studio

- **210. Command-Line Tools**
  - `mysql`
  - `mysqlsh`
  - `mysqldump`
  - `mysqlimport`
  - `mysqladmin`
  - `mysqlbinlog`
  - `mysqlcheck`
  - `mysqlpump` (deprecated)
  - `mysqld_safe`
  - `mysqld`
  - `LOAD DATA INFILE`
  - `SELECT ... INTO OUTFILE`

- **211. Migration Tools**
  - Flyway
  - Liquibase
  - Alembic
  - Prisma Migrate
  - Sequelize migrations
  - TypeORM migrations
  - Drizzle migrations
  - Rails migrations
  - Django migrations
  - Entity Framework migrations
  - MySQL Shell upgrade utilities
  - `pt-online-schema-change`
  - `gh-ost`

- **212. Monitoring Tools**
  - MySQL Workbench
  - MySQL Enterprise Monitor
  - Percona Monitoring and Management (PMM)
  - `performance_schema`
  - `sys` schema
  - Prometheus + `mysqld_exporter`
  - Grafana
  - Datadog
  - New Relic
  - SolarWinds
  - `pt-query-digest`
  - `pt-mysql-summary`
  - `pt-variable-advisor`

- **213. Backup Tools**
  - `mysqldump`
  - `mysqlpump` (deprecated)
  - MySQL Shell `util.dumpInstance()`, `util.dumpSchemas()`
  - `mysqlbackup` (Enterprise)
  - Percona XtraBackup
  - `mariabackup`
  - Binary log backups
  - Filesystem snapshots (LVM, ZFS)

- **214. Replication Tools**
  - Native asynchronous replication
  - Semi-synchronous replication
  - Group Replication
  - MySQL InnoDB Cluster
  - MySQL Router
  - MySQL Shell AdminAPI
  - Orchestrator
  - ProxySQL
  - MHA (Master High Availability)
  - Galera Cluster (Percona XtraDB Cluster, MariaDB Galera)
  - HAProxy
  - Keepalived

- **215. Performance Tools**
  - `EXPLAIN`
  - `EXPLAIN ANALYZE`
  - `EXPLAIN FORMAT=JSON`
  - `EXPLAIN FORMAT=TREE`
  - `performance_schema`
  - `sys` schema
  - `pt-query-digest`
  - `pt-index-usage`
  - `pt-duplicate-key-checker`
  - `pt-archiver`
  - `pt-online-schema-change`
  - `gh-ost`
  - `sysbench`
  - MySQL Enterprise Monitor

---

# XXXVII. Production MySQL Mastery

- **216. Production Database Operations**
  - Monitoring
  - Alerting
  - Capacity planning
  - Query monitoring
  - Error analysis
  - Resource utilization

- **217. Production Performance Engineering**
  - Slow-query analysis
  - Query regression detection
  - Index regression
  - Workload profiling
  - Performance baselines

- **218. Production Reliability**
  - Backup verification
  - Disaster recovery testing
  - Failover testing
  - Recovery procedures
  - Incident response

- **219. Production Security**
  - Privilege audits
  - Credential management
  - Access reviews
  - Audit trails
  - Security monitoring
  - Regulatory considerations

---

# XXXVIII. SQL Projects by Difficulty

## Beginner Projects

- **1. Student Records System**
  - Students
  - Courses
  - Enrollments
  - Grades
  - Basic queries
  - CRUD operations

- **2. Library Management Database**
  - Books
  - Authors
  - Members
  - Loans
  - Borrowing records
  - Basic joins

- **3. Personal Finance Tracker**
  - Accounts
  - Transactions
  - Categories
  - Budgets
  - Reports

- **4. Inventory Database**
  - Products
  - Categories
  - Suppliers
  - Stock
  - Basic CRUD

---

## Intermediate Projects

- **5. E-Commerce Database**
  - Customers
  - Products
  - Categories
  - Orders
  - Order items
  - Payments
  - Inventory
  - Reporting queries

- **6. Blog Database**
  - Users
  - Posts
  - Comments
  - Tags
  - Categories
  - Full-text search (`FULLTEXT` indexes)
  - Pagination

- **7. HR Management System**
  - Employees
  - Departments
  - Positions
  - Salaries
  - Attendance
  - Managers
  - Hierarchical queries (recursive CTEs)
  - Reporting

- **8. Hospital Management Database**
  - Patients
  - Physicians
  - Appointments
  - Treatments

- **9. Analytics Dashboard Database**
  - Events
  - Users
  - Sessions
  - Aggregations
  - Window functions
  - Time-series analysis
  - Summary tables

---

## Advanced Projects

- **10. Financial Transaction System**
  - ACID transactions
  - Concurrency
  - Audit trails
  - Security

- **11. Business Analytics Platform**
  - Complex joins
  - CTEs
  - Window functions
  - KPI reporting

- **12. Data Warehouse**
  - Star schema
  - Snowflake schema
  - Slowly changing dimensions
  - ETL pipelines
  - Aggregations
  - Summary tables
  - Analytical queries

- **13. Multi-Tenant SaaS Database**
  - Tenant isolation
  - Row-level security (emulated with views)
  - Tenant schemas
  - Shared tables
  - Auditing
  - Partitioning
  - Performance tuning

- **14. High-Volume Transaction System**
  - Partitioning
  - Indexing strategies
  - Connection pooling (ProxySQL)
  - Replication (Group Replication)
  - Failover
  - Monitoring
  - Performance tuning

- **15. Real-Time Analytics Platform**
  - Event ingestion
  - Time-series data
  - Aggregations
  - Window functions
  - Summary tables
  - Continuous aggregates (emulated with events)
  - Retention policies

---

## Expert Projects

- **16. Distributed SQL Platform**
  - Sharding (Vitess, ProxySQL)
  - Replication
  - Distributed transactions (`XA`)
  - Distributed query planning
  - Failover
  - Consistency
  - Geo-distribution

- **17. Database-as-a-Service**
  - Multi-tenancy
  - Provisioning
  - Backups
  - Monitoring
  - Scaling
  - Security
  - Billing

- **18. OLAP Data Warehouse**
  - Columnar storage (MySQL HeatWave)
  - Partitioning
  - Compression
  - Vectorized execution
  - Query optimization
  - Summary tables
  - Pre-aggregation
  - BI integration

- **19. Enterprise Analytics Platform**
  - Data warehouse architecture
  - ETL pipelines
  - Dimensional modeling
  - Advanced analytical SQL

---

# XXXIX. Progressive MySQL Learning Sequence

## Level 1 — Foundation

- Master:
  - Database concepts
  - Relational model
  - MySQL syntax
  - Data types
  - Tables
  - Basic constraints
  - `CREATE TABLE`
  - `INSERT`
  - `SELECT`
  - `UPDATE`
  - `DELETE`
  - `WHERE`
  - `ORDER BY`

## Level 2 — Core Querying

- Master:
  - Filtering
  - NULL handling
  - Functions
  - Aggregation
  - Grouping
  - Joins
  - `GROUP BY`
  - `HAVING`
  - `INNER JOIN`
  - `LEFT JOIN`
  - `CASE`
  - Aggregate functions

## Level 3 — Intermediate SQL

- Master:
  - Subqueries
  - CTEs (MySQL 8.0+)
  - Set operations
  - Views
  - Data modeling
  - Normalization
  - Correlated subqueries
  - Recursive CTE concepts
  - `UNION`
  - `INTERSECT` (MySQL 8.0.31+)
  - `EXCEPT` (MySQL 8.0.31+)

## Level 4 — Advanced SQL

- Master:
  - Window functions (MySQL 8.0+)
  - Advanced aggregation
  - Recursive queries
  - JSON
  - Stored procedures
  - Functions
  - Triggers
  - Ranking
  - Running totals
  - Period comparisons
  - Hierarchical queries
  - Complex transformations

## Level 5 — Database Engineering

- Master:
  - Indexing
  - Query optimization
  - Execution plans (`EXPLAIN`, `EXPLAIN ANALYZE`)
  - Transactions
  - Concurrency
  - Locking
  - `EXPLAIN`
  - Query-plan interpretation
  - Index selection
  - Isolation levels
  - Deadlock analysis

## Level 6 — Production Engineering

- Master:
  - Security
  - Backup and recovery
  - Replication
  - Partitioning
  - Monitoring
  - High availability
  - Secure database architecture
  - Recovery strategies
  - Scalability patterns
  - Production troubleshooting

## Level 7 — Expert / Enterprise Mastery

- Master:
  - Distributed databases
  - Data warehousing
  - Dimensional modeling
  - Advanced analytics
  - Large-scale optimization
  - Database architecture
  - Designing production-grade schemas
  - Engineering high-performance queries
  - Building resilient database architectures
  - Optimizing complex workloads
  - Making architecture-level database decisions

---

# XL. Recommended Mastery Progression

- **Stage 1 — Understand**
  - Learn terminology
  - Understand relational concepts
  - Study MySQL syntax

- **Stage 2 — Reproduce**
  - Write basic SQL statements
  - Recreate simple schemas
  - Execute CRUD operations

- **Stage 3 — Apply**
  - Solve realistic query problems
  - Combine multiple SQL features
  - Design relational schemas

- **Stage 4 — Analyze**
  - Read execution plans
  - Diagnose incorrect results
  - Identify performance bottlenecks

- **Stage 5 — Optimize**
  - Improve queries
  - Design effective indexes
  - Tune transactions
  - Reduce resource consumption

- **Stage 6 — Engineer**
  - Build secure database systems
  - Design scalable schemas
  - Implement recovery and availability strategies

- **Stage 7 — Architect**
  - Design enterprise database architectures
  - Select appropriate storage and scaling strategies
  - Balance consistency, performance, reliability, and maintainability

---

# XLI. Final MySQL Competency Map

- **Foundational SQL**
  - Syntax
  - Data types
  - CRUD
  - Filtering
  - Sorting

- **Relational SQL**
  - Joins
  - Aggregation
  - Subqueries
  - Set operations
  - Constraints

- **Database Design**
  - ER modeling
  - Keys
  - Normalization
  - Integrity
  - Schema architecture

- **Advanced Querying**
  - CTEs
  - Window functions
  - Recursive queries
  - Advanced aggregation
  - Analytical SQL

- **Database Programming**
  - Procedures
  - Functions
  - Triggers
  - Dynamic SQL
  - Events

- **Performance Engineering**
  - Indexes
  - Execution plans
  - Query optimization
  - Statistics
  - Workload analysis

- **Database Reliability**
  - Transactions
  - ACID
  - Isolation
  - Locking
  - Recovery

- **Database Security**
  - Authentication
  - Authorization
  - Roles
  - Encryption
  - SQL injection prevention

- **Data Engineering**
  - ETL
  - Data transformation
  - Warehousing
  - Dimensional modeling
  - Analytical workloads

- **Enterprise Mastery**
  - Replication
  - Partitioning
  - High availability
  - Distributed databases
  - Scalability
  - Production operations

---

## Recommended Overall Progression

**MySQL Fundamentals → MySQL Syntax → CRUD → Querying → Joins → Aggregation → Subqueries → CTEs → Database Design → Normalization → Views → Advanced SQL → Window Functions → Transactions → Programmability → Indexing → Query Optimization → Security → Recovery → Replication → Partitioning → Data Warehousing → Distributed Architecture → Production MySQL Engineering.**