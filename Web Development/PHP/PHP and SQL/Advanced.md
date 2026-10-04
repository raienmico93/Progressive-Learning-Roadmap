# Advanced Query Patterns & Performance Optimization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Advanced query patterns and performance optimization encompass the techniques, architectural decisions, and tooling used to maintain database-driven PHP applications that remain fast, memory-efficient, and scalable as data volume and request concurrency grow.

**Technical Definition**

Advanced query patterns include pagination strategies (offset-based vs. cursor-based), relationship loading strategies (lazy vs. eager), and programmatic data generation (seeders and factories). Performance optimization in this context involves index-aware query design, eliminating redundant database round-trips (the N+1 problem), and controlling memory consumption during large result set traversal.

**Beginner-Friendly Explanation**

Imagine you manage a warehouse with millions of items. Basic query patterns work fine when you have 100 items on 10 shelves. But when you have 10 million items across 100,000 shelves, you need smarter strategies: how to walk through the shelves without visiting every one (pagination), how to pick all the items you need in one trip instead of 10,000 trips (N+1 elimination), and how to fill empty shelves with realistic test inventory before opening day (seeding).

### Key Characteristics

- **Scalability-Aware Design:** Techniques must maintain acceptable performance as data volume grows by orders of magnitude.
- **Query Efficiency:** Minimizing database round-trips, scanned rows, and memory consumption.
- **Consistency Under Concurrency:** Pagination and relationship loading must produce stable, correct results even when the underlying data changes between requests.
- **Testability:** Seeders and factories enable reproducible test environments without production data.
- **Index Dependence:** Most optimization techniques (cursor pagination, eager loading) rely on proper database indexing to achieve their performance benefits.

### Prerequisites

- **Solid understanding of SQL** — joins, indexes, `ORDER BY`, `LIMIT`, and query execution plans (`EXPLAIN`).
- **Working knowledge of PDO** — prepared statements, parameter binding, and fetch modes.
- **Database server access** for creating indexes and running `EXPLAIN` queries.
- **Composer** for installing libraries like `fakerphp/faker`.
- **A non-production database** for seeding and testing operations.

### Related Programming Areas

- **Database Indexing and Query Planning:** Understanding B-tree indexes, covering indexes, and how the query optimizer chooses execution plans.
- **API Design:** Cursor-based pagination is the standard for modern REST and GraphQL APIs.
- **ORM and Data Mapper Patterns:** The N+1 problem is most prevalent in ORM-based applications (Doctrine, Eloquent, Propel).
- **Automated Testing and CI/CD:** Seeders and factories are foundational to integration tests and continuous delivery pipelines.

### Core Concepts / Features

1. **Data Pagination** — Efficient `LIMIT`/`OFFSET` queries, deep-page bottlenecks, and cursor-based pagination.
2. **The N+1 Query Problem** — Identifying loop-driven query explosion and solving it via eager loading.
3. **Database Seeding & Factories** — Programmatic generation of realistic mock data using Faker.

---

## Core Concept 1: Data Pagination

### Definitions

**Core Definition**

Data pagination is the technique of dividing a large result set into discrete, manageable subsets (pages) that can be retrieved independently.

**Technical Definition**

Pagination in SQL is typically implemented with `LIMIT` (the maximum number of rows to return) and `OFFSET` (the number of rows to skip). **Offset-based pagination** uses `LIMIT n OFFSET (page - 1) * n`. **Cursor-based pagination** (also called keyset pagination or the seek method) replaces `OFFSET` with a `WHERE` condition that seeks past the last row of the previous page using an indexed sort key.

**Beginner-Friendly Explanation**

Imagine reading a very long book. Offset-based pagination is like counting pages from the beginning every time you want to jump to a chapter — the further into the book you go, the longer it takes to find your place. Cursor-based pagination is like using a bookmark: you remember exactly where you left off and start from there directly.

### Purposes

- To limit the amount of data transferred from the database to PHP in a single request.
- To provide a predictable, navigable interface for end-users to browse large collections.
- To avoid PHP memory exhaustion when rendering lists of records.
- To maintain consistent performance as the total data volume grows.
- To produce stable results under concurrent inserts and deletes.

### Sub-Feature 1.1: Offset-Based Pagination (`LIMIT` / `OFFSET`)

#### Definitions

**Core Definition**

Offset-based pagination retrieves a page of results by specifying a limit (page size) and an offset (number of rows to skip).

**Technical Definition**

The SQL syntax is `SELECT ... FROM table ORDER BY column LIMIT :limit OFFSET :offset`, where `:offset = (page - 1) * :limit`. The database must generate the full result set, then discard the first `:offset` rows before returning the `:limit` rows.

**Beginner-Friendly Explanation**

This is like counting "skip 40 people, then take the next 20" in a line. It works fine when the line is short, but if there are 200,000 people, counting past the first 199,980 is wasteful.

#### Purposes

- To provide simple, page-number-based navigation (e.g., "Page 3 of 50").
- To support random access to any page without sequential traversal.
- To enable "jump to page N" functionality in user interfaces.
- To allow the total number of pages to be calculated when a count query is run.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$page = max(1, (int) ($_GET['page'] ?? 1));
$perPage = 20;
$offset = ($page - 1) * $perPage;

$stmt = $pdo->prepare(
    'SELECT id, title, created_at FROM articles ORDER BY created_at DESC, id DESC LIMIT :limit OFFSET :offset'
);
$stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$stmt->execute();
$articles = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

**Component Breakdown:**

- `$page` — The current page number (1-indexed), validated to be at least 1.
- `$perPage` — The number of rows per page. Should be bounded (e.g., maximum 100) to prevent abuse.
- `$offset` — The number of rows to skip. Calculated as `($page - 1) * $perPage`.
- `LIMIT :limit` — The maximum number of rows to return.
- `OFFSET :offset` — The number of rows to skip before returning results.

**Syntax Rules:**

- `LIMIT` and `OFFSET` parameters must be bound as `PDO::PARAM_INT` when using native prepared statements. Some drivers do not accept string values for these parameters.
- The `ORDER BY` clause must be deterministic. If the sort column is not unique, ties will cause rows to be skipped or duplicated across pages.
- A count query (`SELECT COUNT(*)`) is typically required to calculate total pages.

**Constraints and Limitations:**

- **Deep-page performance degradation:** `OFFSET N` forces the database to generate and discard N rows. On a table with millions of rows, `OFFSET 1000000` reads a million rows only to discard them. The cost grows linearly with page depth.
- **Inconsistent results under concurrent writes:** If a row is inserted or deleted between page requests, subsequent pages may show duplicate or skipped rows.
- **Count query cost:** `SELECT COUNT(*)` on a large table can be expensive, especially without a suitable index.

#### Multiple Annotated Step-by-Step Code Examples

**Example 1: Basic Offset Pagination with Validation**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=blog', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Step 1: Validate and sanitise pagination parameters
$page = max(1, (int) ($_GET['page'] ?? 1));
$perPage = min(50, max(1, (int) ($_GET['per_page'] ?? 20))); // clamp between 1 and 50
$offset = ($page - 1) * $perPage;

// Step 2: Get total count for pagination controls
$totalStmt = $pdo->query('SELECT COUNT(*) FROM articles');
$totalArticles = (int) $totalStmt->fetchColumn();
$totalPages = (int) ceil($totalArticles / $perPage);

// Step 3: Fetch the current page
$stmt = $pdo->prepare(
    'SELECT id, title, created_at FROM articles ORDER BY created_at DESC, id DESC LIMIT :limit OFFSET :offset'
);
$stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$stmt->execute();
$articles = $stmt->fetchAll();

// Step 4: Output
echo "Page $page of $totalPages (Total: $totalArticles articles)\n";
foreach ($articles as $article) {
    echo "- {$article['title']} ({$article['created_at']})\n";
}
```

**Expected Output (page 1 of a blog with 100 articles):**

```
Page 1 of 5 (Total: 100 articles)
- Article 100 (2025-01-15 10:00:00)
- Article 99 (2025-01-14 09:30:00)
- Article 98 (2025-01-13 14:20:00)
... (17 more)
```

**Why:** The script clamps `$perPage` between 1 and 50 to prevent a client from requesting 10,000 rows in one page. It uses `COUNT(*)` to determine the total number of pages for navigation controls. The `LIMIT` and `OFFSET` parameters are bound as integers, which is required for native prepared statements on MySQL.

**Example 2: Demonstrating the Deep-Page Performance Problem**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=blog', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Simulate deep pagination — page 10,000
$perPage = 20;
$page = 10000;
$offset = ($page - 1) * $perPage; // 199,980

$start = microtime(true);

$stmt = $pdo->prepare(
    'SELECT id, title FROM articles ORDER BY created_at DESC, id DESC LIMIT :limit OFFSET :offset'
);
$stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$stmt->execute();
$articles = $stmt->fetchAll();

$elapsed = microtime(true) - $start;

echo "Page $page (OFFSET $offset): " . count($articles) . " rows returned.\n";
echo "Query time: " . round($elapsed * 1000, 2) . " ms\n";
```

**Expected Output (illustrative, on a table with 1 million rows):**

```
Page 10000 (OFFSET 199980): 20 rows returned.
Query time: 342.15 ms
```

**Why:** The query must scan and discard 199,980 rows before returning the 20 rows for page 10,000. The execution time grows roughly linearly with the offset. On page 1, the same query would take less than 1 ms.

#### Real-World Cases

**Case 1: Admin Panel with Small Data Sets**

An admin panel managing 500–2,000 records uses offset-based pagination because deep pages are rarely accessed, and the overhead of `OFFSET` is negligible at this scale. The page-number navigation is user-friendly for administrators who need to jump to specific pages.

**Case 2: Search Results with Count Display**

A search engine displays "Showing results 1–20 of 12,450" and provides numbered page links. Offset-based pagination is appropriate here because the count is already required for the result summary, and users rarely navigate beyond the first few pages.

---

### Sub-Feature 1.2: Cursor-Based Pagination (Keyset Pagination)

#### Definitions

**Core Definition**

Cursor-based pagination retrieves pages by seeking past the last row of the previous page using an indexed sort key, rather than counting and skipping rows.

**Technical Definition**

Keyset pagination uses a `WHERE` clause that compares the sort key values against those of the last row seen. For a single-column sort: `WHERE id > :last_id ORDER BY id LIMIT :limit`. For multi-column sorts: `WHERE (created_at, id) < (:last_created, :last_id) ORDER BY created_at DESC, id DESC LIMIT :limit`. The row-value comparison ensures a total, unambiguous ordering when the sort column contains duplicates.

**Beginner-Friendly Explanation**

Instead of counting from the start of the list, cursor pagination remembers the last item you saw and asks the database: "Give me the next 20 items that come after this one." Because the database can jump directly to that position using an index, the speed is the same whether you are on page 1 or page 10,000.

#### Purposes

- To provide constant-time pagination performance regardless of page depth.
- To eliminate the risk of duplicate or skipped rows caused by concurrent inserts and deletes.
- To scale efficiently on tables with millions or billions of rows.
- To provide a stable "next page" link in API responses.

#### Syntax Rules and Structure

**Complete General Syntax: Single-Column Cursor**

```php
// First page (no cursor)
$sql = 'SELECT id, title, created_at FROM articles ORDER BY id ASC LIMIT :limit';

// Subsequent pages (with cursor)
$sql = 'SELECT id, title, created_at FROM articles WHERE id > :cursor ORDER BY id ASC LIMIT :limit';
```

**Complete General Syntax: Multi-Column Cursor (Row-Value Comparison)**

```php
$sql = 'SELECT id, title, created_at FROM articles
        WHERE (created_at, id) < (:last_created, :last_id)
        ORDER BY created_at DESC, id DESC
        LIMIT :limit';
```

**Component Breakdown:**

- `:cursor` / `:last_id` — The unique identifier of the last row from the previous page.
- `:last_created` — The `created_at` value of the last row from the previous page.
- `(created_at, id) < (:last_created, :last_id)` — A row-value (tuple) comparison. It returns rows that come strictly after the cursor in the sort order.
- `ORDER BY ... LIMIT :limit` — The same sort order and limit as the first page.

**Syntax Rules:**

- The `WHERE`, `ORDER BY`, and the database index must all share the same column order and direction, or the database cannot use the index for the seek.
- The cursor must be unique. If the sort column is not unique, append a unique tiebreaker (typically the primary key) to the sort and the cursor.
- The cursor values must be bound as prepared statement parameters — never concatenated into SQL.

**Constraints and Limitations:**

- **No random access:** Users cannot jump to "page 500" directly; they must follow the cursor chain sequentially.
- **Index dependency:** Without a matching index on the sort columns, cursor pagination is no faster than `OFFSET`.
- **Complex filtering:** Cursor pagination interacts poorly with dynamic, multi-column filters because the cursor must encode all sort and filter state.
- **Forward-only (by default):** Bidirectional cursor pagination requires additional logic (e.g., a `direction` parameter and inverted sort order).

#### Multiple Annotated Step-by-Step Code Examples

**Example 1: Forward-Only Cursor Pagination (Single Column)**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=blog', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

$perPage = 20;
$cursor = isset($_GET['cursor']) ? (int) $_GET['cursor'] : null;

if ($cursor === null) {
    // First page: no cursor
    $stmt = $pdo->prepare(
        'SELECT id, title, created_at FROM articles ORDER BY id ASC LIMIT :limit'
    );
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
} else {
    // Subsequent pages: seek past the cursor
    $stmt = $pdo->prepare(
        'SELECT id, title, created_at FROM articles WHERE id > :cursor ORDER BY id ASC LIMIT :limit'
    );
    $stmt->bindValue(':cursor', $cursor, PDO::PARAM_INT);
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
}

$stmt->execute();
$articles = $stmt->fetchAll();

// Determine next cursor
$lastArticle = end($articles);
$nextCursor = $lastArticle ? $lastArticle['id'] : null;

echo "Fetched " . count($articles) . " articles.\n";
foreach ($articles as $article) {
    echo "- [{$article['id']}] {$article['title']}\n";
}
echo "Next cursor: " . ($nextCursor ?? 'none') . "\n";
```

**Expected Output (first page):**

```
Fetched 20 articles.
- [1] First Article
- [2] Second Article
... (18 more)
Next cursor: 20
```

**Expected Output (second page, `?cursor=20`):**

```
Fetched 20 articles.
- [21] Twenty-First Article
... (19 more)
Next cursor: 40
```

**Why:** The first request has no cursor and returns the first 20 rows by `id ASC`. The last row has `id = 20`, which becomes the cursor for the next request. The second request uses `WHERE id > 20` to seek directly to the next slice using the primary key index. The query time is the same for both pages.

**Example 2: Multi-Column Cursor Pagination (Row-Value Comparison)**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=blog', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

$perPage = 20;
$lastCreated = $_GET['last_created'] ?? null;
$lastId = isset($_GET['last_id']) ? (int) $_GET['last_id'] : null;

$sql = 'SELECT id, title, created_at FROM articles';
$params = [':limit' => $perPage];

if ($lastCreated !== null && $lastId !== null) {
    // Row-value comparison for stable multi-column cursor
    $sql .= ' WHERE (created_at, id) < (:last_created, :last_id)';
    $params[':last_created'] = $lastCreated;
    $params[':last_id'] = $lastId;
}

$sql .= ' ORDER BY created_at DESC, id DESC LIMIT :limit';

$stmt = $pdo->prepare($sql);
foreach ($params as $key => $value) {
    $type = ($key === ':limit' || $key === ':last_id') ? PDO::PARAM_INT : PDO::PARAM_STR;
    $stmt->bindValue($key, $value, $type);
}
$stmt->execute();
$articles = $stmt->fetchAll();

foreach ($articles as $article) {
    echo "- [{$article['id']}] {$article['title']} ({$article['created_at']})\n";
}

// Build next cursor from the last row
$last = end($articles);
if ($last) {
    $nextCursor = http_build_query([
        'last_created' => $last['created_at'],
        'last_id' => $last['id'],
    ]);
    echo "Next page: ?$nextCursor\n";
}
```

**Expected Output (first page):**

```
- [150] Newest Article (2025-06-15 14:30:00)
- [149] Another Article (2025-06-15 14:30:00)
- [148] Earlier Article (2025-06-14 09:15:00)
... (17 more)
Next page: ?last_created=2025-06-13+10%3A00%3A00&last_id=131
```

**Why:** The `created_at` column is not unique — two articles can share the same timestamp. The row-value comparison `(created_at, id) < (:last_created, :last_id)` ensures a total ordering: when two rows share the same `created_at`, the `id` tiebreaker determines their relative position. Without this, rows with identical timestamps would be skipped or repeated at page boundaries.

**Example 3: Bidirectional Cursor Pagination (Previous/Next)**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=blog', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

$perPage = 20;
$direction = $_GET['direction'] ?? 'forward'; // 'forward' or 'backward'
$cursor = isset($_GET['cursor']) ? (int) $_GET['cursor'] : null;

if ($direction === 'backward' && $cursor !== null) {
    // Fetch previous page: reverse sort order, then reverse the results
    $stmt = $pdo->prepare(
        'SELECT id, title FROM articles WHERE id < :cursor ORDER BY id DESC LIMIT :limit'
    );
    $stmt->bindValue(':cursor', $cursor, PDO::PARAM_INT);
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
    $stmt->execute();
    $articles = array_reverse($stmt->fetchAll());
} else {
    // Fetch next page (or first page)
    if ($cursor !== null) {
        $stmt = $pdo->prepare(
            'SELECT id, title FROM articles WHERE id > :cursor ORDER BY id ASC LIMIT :limit'
        );
        $stmt->bindValue(':cursor', $cursor, PDO::PARAM_INT);
    } else {
        $stmt = $pdo->prepare('SELECT id, title FROM articles ORDER BY id ASC LIMIT :limit');
    }
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
    $stmt->execute();
    $articles = $stmt->fetchAll();
}

// Determine cursors for navigation links
$first = reset($articles);
$last = end($articles);

echo "Direction: $direction\n";
foreach ($articles as $article) {
    echo "- [{$article['id']}] {$article['title']}\n";
}

if ($first) {
    echo "Previous cursor: {$first['id']}\n";
}
if ($last) {
    echo "Next cursor: {$last['id']}\n";
}
```

**Expected Output (forward, cursor=20):**

```
Direction: forward
- [21] Article 21
... (19 more)
Previous cursor: 21
Next cursor: 40
```

**Why:** Backward pagination inverts the sort order (`DESC` instead of `ASC`) and the comparison (`<` instead of `>`), then reverses the result array to present rows in the original order. This allows users to navigate both forward and backward while maintaining constant-time performance in both directions.

#### Real-World Cases

**Case 1: Social Media Feed (Infinite Scroll)**

A social media application displays an infinite-scrolling feed of posts sorted by `created_at DESC, id DESC`. Each API response includes a `next_cursor` field containing the `created_at` and `id` of the last post. The client sends this cursor in the next request to fetch the next batch. Because the sort columns are indexed, the query time remains constant regardless of how deep the user scrolls.

**Case 2: E-Commerce Product Listing (Modern API)**

A modern e-commerce API uses cursor-based pagination for product listings, following the UN/CEFACT recommendation that keyset-based pagination "SHALL be used" for API pagination. The API provides `links.next` and `links.prev` in the response body, and the cursor is an opaque string encoding the sort values of the last row.

**Case 3: Log Viewer with Time-Based Cursor**

A log analysis tool displays millions of log entries sorted by timestamp. Cursor pagination on the `(logged_at, id)` composite key allows operators to browse through months of logs without the performance cliff that would occur with `OFFSET` on page 50,000.

---

## Core Concept 2: The N+1 Query Problem

### Definitions

**Core Definition**

The N+1 query problem is a performance anti-pattern in which an application executes one query to retrieve a list of N parent records, then executes N additional queries — one for each parent — to retrieve related child records.

**Technical Definition**

The N+1 problem occurs when relationship loading is performed lazily inside a loop. For example, fetching 100 orders and then fetching the customer for each order results in 1 + 100 = 101 queries. The total query count scales linearly with the number of parent records, causing excessive database round-trips, increased latency, and higher server load. The problem is most prevalent in ORM-based applications where lazy loading is the default behaviour.

**Beginner-Friendly Explanation**

Imagine you are a teacher with 30 students. You ask the office for the list of students (1 request). Then, for each student, you call the office again to ask for their address (30 more requests). That is 31 phone calls when you could have asked for all addresses in one go. The N+1 problem is the database version of this inefficiency.

### Purposes

- To recognise the pattern of queries executed inside loops that depend on the results of a prior query.
- To understand the performance cost of lazy loading in ORM and PDO-based applications.
- To eliminate redundant database round-trips through eager loading or batched `IN` queries.
- To reduce application response time and database server load in list views.
- To adopt a query-shape mindset where the number of queries is independent of the number of rows.

### Syntax Rules and Structure

**The Problem (Lazy Loading Inside a Loop)**

```php
// Step 1: Fetch all orders (1 query)
$orders = $pdo->query('SELECT id, customer_id, total FROM orders')->fetchAll();

// Step 2: Fetch the customer for each order (N queries)
foreach ($orders as $order) {
    $stmt = $pdo->prepare('SELECT name, email FROM customers WHERE id = :id');
    $stmt->execute([':id' => $order['customer_id']]);
    $customer = $stmt->fetch();
    echo "{$customer['name']} ordered \${$order['total']}\n";
}
// Total queries: 1 + N (where N = number of orders)
```

**The Solution (Eager Loading with a JOIN)**

```php
// Single query with JOIN
$stmt = $pdo->query(
    'SELECT o.id, o.total, c.name AS customer_name, c.email AS customer_email
     FROM orders o
     INNER JOIN customers c ON c.id = o.customer_id'
);
$orders = $stmt->fetchAll();

foreach ($orders as $order) {
    echo "{$order['customer_name']} ordered \${$order['total']}\n";
}
// Total queries: 1
```

**The Solution (Batched IN Query)**

```php
// Step 1: Fetch all orders (1 query)
$orders = $pdo->query('SELECT id, customer_id, total FROM orders')->fetchAll();

// Step 2: Collect unique customer IDs
$customerIds = array_unique(array_column($orders, 'customer_id'));

// Step 3: Fetch all customers in one query (1 query)
$placeholders = implode(',', array_fill(0, count($customerIds), '?'));
$stmt = $pdo->prepare("SELECT id, name, email FROM customers WHERE id IN ($placeholders)");
$stmt->execute(array_values($customerIds));
$customers = [];
foreach ($stmt->fetchAll() as $customer) {
    $customers[$customer['id']] = $customer;
}

// Step 4: Map customers to orders in PHP (0 additional queries)
foreach ($orders as $order) {
    $customer = $customers[$order['customer_id']];
    echo "{$customer['name']} ordered \${$order['total']}\n";
}
// Total queries: 2 (constant, regardless of N)
```

**Component Breakdown:**

- **JOIN approach:** Combines parent and child data in a single query. Best when the relationship is one-to-one or many-to-one, or when the child result set per parent is small.
- **Batched IN approach:** Fetches all parents, collects their foreign keys, fetches all children in one `WHERE IN (...)` query, then maps them in PHP. Best when the relationship is one-to-many and a JOIN would cause row explosion.
- `array_column($orders, 'customer_id')` — Extracts an array of customer IDs from the orders array.
- `array_unique(...)` — Removes duplicate customer IDs so each customer is fetched only once.
- `array_fill(0, count($customerIds), '?')` — Creates an array of `?` placeholders matching the number of unique IDs.
- `implode(',', ...)` — Joins the placeholders into a comma-separated string for the `IN` clause.

**Syntax Rules:**

- The `IN` clause requires one placeholder per value. You cannot bind an array to a single placeholder.
- If the number of unique IDs is very large (e.g., tens of thousands), the `IN` query may hit query size limits. In that case, chunk the IDs into batches of a few thousand.
- The `JOIN` approach can cause row explosion when a parent has many children. For example, 100 orders each with 5 line items produces 500 rows in the result set, repeating the order data 5 times.
- Both approaches eliminate the N+1 problem. The choice depends on the relationship type and the size of the data.

**Constraints and Limitations:**

- **Doctrine's documented N+1 limitation:** Even with `fetch="EAGER"`, Doctrine ORM's OneToMany and ManyToMany relations generate separate eager-loading queries rather than a single JOIN, resulting in 2–4 queries instead of 1. Solving this requires JSON aggregation or DBAL-level queries.
- **JOIN with pagination:** A JOIN that multiplies rows makes `LIMIT`/`OFFSET` pagination incorrect because the limit applies to the multiplied rows, not the parent entities. Solutions like `DISTINCT` or subqueries are needed.
- **Memory usage:** The batched `IN` approach loads all children into PHP memory at once. For very large result sets, this may be impractical.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Demonstrating the N+1 Problem**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Step 1: Fetch all orders (1 query)
$orders = $pdo->query('SELECT id, customer_id, total FROM orders')->fetchAll();
echo "Fetched " . count($orders) . " orders.\n";

// Step 2: Fetch customer for each order (N queries)
$queryCount = 1; // Start with the orders query
foreach ($orders as $order) {
    $stmt = $pdo->prepare('SELECT name FROM customers WHERE id = :id');
    $stmt->execute([':id' => $order['customer_id']]);
    $customer = $stmt->fetch();
    $queryCount++;
    echo "Order #{$order['id']}: {$customer['name']} paid \${$order['total']}\n";
}

echo "\nTotal queries executed: $queryCount\n";
```

**Expected Output:**

```
Fetched 100 orders.
Order #1: Alice paid $99.99
Order #2: Bob paid $49.50
... (98 more)
Total queries executed: 101
```

**Why:** The first query fetches 100 orders. The loop then executes one query per order to fetch the customer name. With 100 orders, the total is 1 + 100 = 101 queries. If the orders list had 10,000 rows, the application would execute 10,001 queries, overwhelming the database and causing severe latency.

**Example 2: Solving N+1 with a Batched `IN` Query**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Step 1: Fetch all orders (1 query)
$orders = $pdo->query('SELECT id, customer_id, total FROM orders')->fetchAll();

// Step 2: Collect unique customer IDs
$customerIds = array_values(array_unique(array_column($orders, 'customer_id')));

// Step 3: Fetch all customers in one query (1 query)
$placeholders = implode(',', array_fill(0, count($customerIds), '?'));
$stmt = $pdo->prepare("SELECT id, name FROM customers WHERE id IN ($placeholders)");
$stmt->execute($customerIds);
$customers = [];
foreach ($stmt->fetchAll() as $customer) {
    $customers[$customer['id']] = $customer;
}

// Step 4: Map customers to orders in PHP
foreach ($orders as $order) {
    $customer = $customers[$order['customer_id']];
    echo "Order #{$order['id']}: {$customer['name']} paid \${$order['total']}\n";
}

echo "\nTotal queries executed: 2\n";
```

**Expected Output:**

```
Order #1: Alice paid $99.99
Order #2: Bob paid $49.50
... (98 more)
Total queries executed: 2
```

**Why:** The first query fetches all orders. The second query fetches all customers whose IDs appear in the orders, using a single `WHERE IN (...)` clause. The PHP loop then maps customers to orders using an in-memory associative array. The total query count is 2, regardless of whether there are 100 orders or 100,000 orders.

**Example 3: Solving N+1 with a JOIN**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Single query with JOIN
$stmt = $pdo->query(
    'SELECT o.id, o.total, c.name AS customer_name
     FROM orders o
     INNER JOIN customers c ON c.id = o.customer_id
     ORDER BY o.id'
);
$rows = $stmt->fetchAll();

foreach ($rows as $row) {
    echo "Order #{$row['id']}: {$row['customer_name']} paid \${$row['total']}\n";
}

echo "\nTotal queries executed: 1\n";
```

**Expected Output:**

```
Order #1: Alice paid $99.99
Order #2: Bob paid $49.50
... (98 more)
Total queries executed: 1
```

**Why:** The `INNER JOIN` combines the orders and customers tables in a single query. The database retrieves all the data in one round-trip. This is the most efficient approach when the relationship is many-to-one (many orders to one customer) and each order has exactly one customer. If the relationship were one-to-many (one order with many line items), the JOIN would produce duplicate order rows, and the batched `IN` approach would be preferable.

### Real-World Cases

**Case 1: Laravel Eloquent with `with()`**

A Laravel application displays a list of 50 posts, each with its author and category. Without eager loading, Eloquent executes 1 query for posts, 50 for authors, and 50 for categories — 101 queries. Adding `Post::with(['author', 'category'])->get()` reduces this to 3 queries: one for posts, one for all authors (`WHERE id IN (...)`), and one for all categories.

**Case 2: Doctrine with JSON Aggregation**

A Symfony application using Doctrine ORM has a page that loads 100 partners with their profiles, countries, and promocodes. Standard Doctrine eager loading produces 4 separate queries. A library like `rgalstyan/symfony-aggregated-queries` replaces these with a single SQL statement using `JSON_OBJECT` and `JSON_ARRAYAGG`, reducing database round-trips by 75% and memory usage by over 90%.

**Case 3: GraphQL Resolver with Batched Loading**

A GraphQL API backed by Eloquent uses the `rebing/graphql-laravel-select-fields` package, which analyses the GraphQL query's field selection and eager-loads only the requested relations. This prevents N+1 queries and avoids over-fetching columns that the client did not request.

---

## Core Concept 3: Database Seeding & Factories

### Definitions

**Core Definition**

Database seeding is the process of programmatically populating a database with realistic test or development data using generators and factory blueprints.

**Technical Definition**

A **factory** defines a blueprint for creating records of a specific type (e.g., a `UserFactory` that defines default attributes for a user row). A **seeder** is a script that invokes factories — or direct inserts — to populate one or more tables. The Faker library (`fakerphp/faker`) provides locale-aware generators for names, addresses, emails, dates, text, and other realistic data types.

**Beginner-Friendly Explanation**

Think of a factory as a cookie cutter that shapes dough into a specific form. The seeder is the baker who uses the cookie cutter to produce dozens of cookies and places them on the tray (the database). Faker is the flavour supplier that makes each cookie taste slightly different — one vanilla, one chocolate, one strawberry — so the data looks realistic rather than identical.

### Purposes

- To populate development and testing databases with realistic data without using production records.
- To create reproducible test fixtures for automated integration and end-to-end tests.
- To enable developers to work with a database that mirrors real-world data shapes.
- To generate large volumes of data for performance testing and stress testing.
- To provide locale-specific data for testing internationalisation features.

### Syntax Rules and Structure

**Complete General Syntax: Factory Definition (Framework-Agnostic)**

```php
class UserFactory
{
    public static function definition(\Faker\Generator $faker): array
    {
        return [
            'name'       => $faker->name(),
            'email'      => $faker->unique()->safeEmail(),
            'created_at' => $faker->dateTimeBetween('-1 year', 'now')->format('Y-m-d H:i:s'),
        ];
    }
}
```

**Complete General Syntax: Seeder Execution with PDO**

```php
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$faker = \Faker\Factory::create('en_US');

$stmt = $pdo->prepare(
    'INSERT INTO users (name, email, created_at) VALUES (:name, :email, :created_at)'
);

for ($i = 0; $i < 50; $i++) {
    $data = UserFactory::definition($faker);
    $stmt->execute($data);
}

echo "Seeded 50 users.\n";
```

**Component Breakdown:**

- `Faker\Factory::create('en_US')` — Creates a Faker generator configured for the `en_US` locale. Other locales (e.g., `fr_FR`, `de_DE`) produce locale-appropriate names, addresses, and phone numbers.
- `$faker->name()` — Generates a random full name.
- `$faker->unique()->safeEmail()` — Generates a unique, safe email address. The `unique()` modifier ensures no duplicate values are produced within the same Faker instance.
- `$faker->dateTimeBetween('-1 year', 'now')` — Generates a random `DateTime` object within the specified range.
- `$stmt->execute($data)` — Binds the associative array to the named placeholders and executes the insert.

**Syntax Rules:**

- Faker generators must be called with `()` (e.g., `$faker->name()`), not passed as bare properties. In Laravel's factory system, bare properties (e.g., `$this->faker->name`) work because Laravel wraps them, but in standalone Faker, parentheses are required.
- The `unique()` modifier must be called before the generator method: `$faker->unique()->email()`.
- When seeding related tables, insert parent records first, then capture their IDs (`$pdo->lastInsertId()`) and use them as foreign keys in child records.
- Use prepared statements for all inserts — Faker generates values that may contain special characters (apostrophes, non-ASCII text) that would break concatenated SQL.

**Constraints and Limitations:**

- **Seeding is destructive if misused:** Tools like `tebazil/db-seeder` truncate tables before refilling them. Never run seeding operations on production databases.
- **Faker uniqueness scope:** The `unique()` modifier only guarantees uniqueness within a single Faker instance. If you create multiple Faker instances or restart the script, duplicate values may be produced.
- **Performance:** Inserting hundreds of thousands of rows one at a time is slow. Use batch inserts or wrap the operation in a transaction for better performance.
- **Foreign key integrity:** When seeding related tables, insert order matters. Parent tables must be populated before child tables that reference them.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Standalone PDO Seeder with Faker**

```php
<?php
require 'vendor/autoload.php';

// Step 1: Connect to the database
$pdo = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8mb4', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 2: Create the table if it does not exist
$pdo->exec('
    CREATE TABLE IF NOT EXISTS users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        email VARCHAR(150) NOT NULL UNIQUE,
        bio TEXT,
        created_at DATETIME NOT NULL
    )
');

// Step 3: Initialise Faker
$faker = Faker\Factory::create('en_US');

// Step 4: Prepare the INSERT statement
$stmt = $pdo->prepare(
    'INSERT INTO users (name, email, bio, created_at) VALUES (:name, :email, :bio, :created_at)'
);

// Step 5: Generate and insert 25 users
for ($i = 0; $i < 25; $i++) {
    $stmt->execute([
        ':name'       => $faker->name(),
        ':email'      => $faker->unique()->safeEmail(),
        ':bio'        => $faker->paragraph(3),
        ':created_at' => $faker->dateTimeBetween('-6 months', 'now')->format('Y-m-d H:i:s'),
    ]);
}

echo "Seeded 25 users successfully.\n";

// Step 6: Verify
$count = $pdo->query('SELECT COUNT(*) FROM users')->fetchColumn();
echo "Total users in database: $count\n";
```

**Expected Output:**

```
Seeded 25 users successfully.
Total users in database: 25
```

**Why:** The script uses PDO prepared statements to safely insert Faker-generated data. Each iteration generates a unique email address, a random name, a three-sentence bio, and a random creation date within the last six months. Prepared statements prevent SQL injection even if Faker generates values containing special characters.

**Example 2: Seeding Related Tables with Foreign Keys**

```php
<?php
require 'vendor/autoload.php';

$pdo = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8mb4', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Create tables
$pdo->exec('
    CREATE TABLE IF NOT EXISTS categories (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL
    )
');
$pdo->exec('
    CREATE TABLE IF NOT EXISTS products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        category_id INT NOT NULL,
        name VARCHAR(150) NOT NULL,
        price DECIMAL(10,2) NOT NULL,
        FOREIGN KEY (category_id) REFERENCES categories(id)
    )
');

$faker = Faker\Factory::create();

// Step 2: Seed categories (parents first)
$categoryStmt = $pdo->prepare('INSERT INTO categories (name) VALUES (:name)');
$categoryIds = [];
for ($i = 0; $i < 5; $i++) {
    $categoryStmt->execute([':name' => $faker->word()]);
    $categoryIds[] = (int) $pdo->lastInsertId();
}

echo "Seeded 5 categories with IDs: " . implode(', ', $categoryIds) . "\n";

// Step 3: Seed products (children, referencing parent IDs)
$productStmt = $pdo->prepare(
    'INSERT INTO products (category_id, name, price) VALUES (:category_id, :name, :price)'
);
for ($i = 0; $i < 30; $i++) {
    $productStmt->execute([
        ':category_id' => $faker->randomElement($categoryIds),
        ':name'        => $faker->words(3, true),
        ':price'       => $faker->randomFloat(2, 1, 500),
    ]);
}

echo "Seeded 30 products referencing random categories.\n";

// Step 4: Verify with a JOIN
$stmt = $pdo->query(
    'SELECT p.name AS product, p.price, c.name AS category
     FROM products p
     INNER JOIN categories c ON c.id = p.category_id
     ORDER BY p.id
     LIMIT 5'
);
foreach ($stmt->fetchAll(PDO::FETCH_ASSOC) as $row) {
    echo "- {$row['product']} (\${$row['price']}) in category: {$row['category']}\n";
}
```

**Expected Output:**

```
Seeded 5 categories with IDs: 1, 2, 3, 4, 5
Seeded 30 products referencing random categories.
- "qui et quia" ($234.56) in category: aut
- "ut velit quisquam" ($89.12) in category: voluptas
... (3 more)
```

**Why:** Categories are seeded first because products reference them via a foreign key. After each category insert, `$pdo->lastInsertId()` captures the auto-incremented ID. The product seeder then uses `$faker->randomElement($categoryIds)` to assign a random existing category to each product. The final JOIN verifies that the foreign key relationships are correct.

**Example 3: Framework-Agnostic Factory Pattern with Dolly**

```php
<?php
require 'vendor/autoload.php';

use Dolly\Factory;
use Dolly\Storage;

// Step 1: Implement the Storage interface with PDO
class PdoStorage implements Storage
{
    private PDO $pdo;

    public function __construct()
    {
        $this->pdo = new PDO('mysql:host=localhost;dbname=testdb', 'user', 'pass');
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    }

    public function query($query) { $this->pdo->query($query); return true; }
    public function quote($value) { return $this->pdo->quote($value); }
    public function getLastInsertId() { return $this->pdo->lastInsertId(); }
}

// Step 2: Set up Dolly with the storage
$storage = new PdoStorage();
Factory::setup(['storage' => $storage]);

// Step 3: Define a factory blueprint
Factory::define('user', [
    'username' => 'TestUser',
    'email'    => 'test@example.com',
    'password' => '123456',
]);

// Step 4: Create a row using the factory
$user = Factory::create('user');
echo "Created user: {$user->username} ({$user->email})\n";

// Step 5: Override defaults
$user2 = Factory::create('user', [
    'username' => 'ModifiedUser',
    'email'    => 'modified@example.com',
]);
echo "Created user: {$user2->username} ({$user2->email})\n";
```

**Expected Output:**

```
Created user: TestUser (test@example.com)
Created user: ModifiedUser (modified@example.com)
```

**Why:** Dolly is a lightweight, framework-agnostic factory library inspired by Ruby's factory_bot. It allows factories to be defined as blueprints with default values, which can be overridden per-creation. The `Storage` interface abstracts the database connection, making the factory system portable across different PDO-backed projects.

### Real-World Cases

**Case 1: Laravel Development Database**

A Laravel project uses `php artisan migrate:fresh --seed` to drop all tables, recreate the schema, and populate it with test data. Seeders use `User::factory(50)->create()` and `Post::factory(200)->create()` to generate a realistic development database with users and their posts. This command is destructive and is only used in development and testing environments.

**Case 2: Integration Test Fixtures**

A PHPUnit test suite uses Dolly factories to create just enough data for each test. For example, a test for an order checkout might create one user, one product, and one cart, then execute the checkout logic and assert the expected database state. Factories ensure each test starts with a clean, known state.

**Case 3: Performance Testing with Large Data Sets**

A team preparing for a database migration uses `tebazil/db-seeder` to populate a staging database with 500,000 articles and 2 million comments. The seeder uses Faker to generate realistic text and relationships, allowing the team to benchmark query performance and index effectiveness at production scale before the real migration.

---

## References

- Armour Infosec: Keyset Pagination — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Using-PHP-to-Access-MySQL/Keyset-Pagination.md
- Supabase: Cursor-Based Pagination Best Practices — https://github.com/archerverified/alphawolfedecals-app/blob/f70ef35a35566e8051e398d36f8b78e1f65d5463/.agents/skills/supabase-postgres-best-practices/rules/data-pagination.md
- UN/CEFACT: API Technical Specification — Keyset Pagination — https://uncefact.unece.org/download/attachments/83591898/API-TECH-SPEC_OpenAPI_NDR_20220607.docx
- Tencent Cloud: 深度分页问题 (Deep Pagination Problem) — https://cloud.tencent.cn/developer/article/2504126
- Symfony Aggregated Queries (N+1 Solution) — https://packagist.org/packages/rgalstyan/symfony-aggregated-queries
- Doctrine ORM Issue #4762: N+1 with EAGER — https://github.com/doctrine/orm/issues/4762
- Rebing GraphQL Laravel Select Fields (N+1 Prevention) — https://packagist.org/packages/rebing/graphql-laravel-select-fields
- Dolly: Lightweight PHP Fixture Library — https://packagist.org/packages/dolly/dolly
- Tebazil DB Seeder — https://packagist.org/packages/tebazil/db-seeder
- FakerPHP Faker — https://packagist.org/packages/fakerphp/faker
- Laravel: Eloquent Factories — https://laravel.com/docs/eloquent-factories
- Laravel: Database Seeders — https://mintlify.wiki/OlallaSanchez17/Laravel_MP0613_RA7_RA8_Olalla/database/seeders
- Laravel Model Factories and Faker — https://m.yisu.com/zixun/927715.html
- Stack Overflow: PDO Second Query in While Loop (N+1) — https://stackoverflow.com/questions/13169949/pdo-second-query-in-while-loop
- PHP Delusions: PDO — https://phpdelusions.net/pdo