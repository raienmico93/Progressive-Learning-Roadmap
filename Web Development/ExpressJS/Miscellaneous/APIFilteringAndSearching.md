# API Filtering, Searching, and Advanced Querying — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API filtering, searching, and advanced querying are techniques that allow clients to retrieve specific subsets of data from an API by passing criteria through URL query parameters, rather than fetching entire datasets and filtering them client-side.

**Technical Definition:** Filtering and sorting allow clients to retrieve specific subsets of data efficiently. Well-designed query parameters make APIs flexible and powerful. These techniques are implemented by parsing `req.query` in Express, mapping query parameters to database query conditions (SQL `WHERE`, `ORDER BY`, `LIMIT` clauses or ORM filter objects), and returning only matching records. The core challenge is safely constructing dynamic database queries from arbitrary user-supplied parameters while preventing injection vulnerabilities.

**Beginner-Friendly Explanation:** Imagine a library with millions of books. Without filtering, you'd have to receive every book and search through them yourself. With filtering, you tell the librarian "show me only science fiction books under $20, sorted by newest first" — and the librarian (your API) retrieves exactly what you need. This cheat sheet covers the different ways to express those criteria (exact matches, ranges, text searches), how to combine them, how to sort results, and how to build the database queries safely.

### Key Characteristics

- **Query-parameter driven:** All filtering, searching, and sorting is expressed through URL query parameters (`req.query`).
- **Database-agnostic patterns:** The same query parameter conventions work across SQL (PostgreSQL, MySQL) and NoSQL (MongoDB) databases.
- **Injection-sensitive:** Dynamic query construction from user input is a primary SQL/NoSQL injection vector; parameterized queries are mandatory.
- **Composable:** Filters, searches, and sorts can be combined in a single request.
- **Whitelist-dependent:** Only pre-approved fields, operators, and sort columns should be accepted from clients.
- **Performance-critical:** Database indexes on filtered and sorted columns are essential for scalability.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A database** (PostgreSQL, MySQL, MongoDB, etc.) with indexed filter/sort columns.
- **Basic JavaScript knowledge:** Async/await, objects, arrays, and array methods.
- **Understanding of SQL:** `WHERE`, `ORDER BY`, `LIKE`, and parameterized query syntax.

### Related Programming Areas

- **Pagination:** Filtering and sorting are typically combined with pagination (`limit`, `offset`, or cursor).
- **Database indexing:** GIN indexes for full-text search, B-tree indexes for range filters.
- **API security:** Injection prevention, input validation, and whitelisting.
- **ORM/query builders:** Sequelize, Prisma, Knex, and Mongoose provide programmatic query construction.
- **Full-text search engines:** PostgreSQL `tsvector`, Elasticsearch, and Algolia for advanced text search.
- **Caching:** Filtered results can be cached by query signature.

### Core Concepts

1. **Exact Filters** — mapping exact fields directly to database queries.
2. **Range Filters** — handling numeric or chronological windows using field suffixes or logical brackets.
3. **Text Searches** — partial match filtering (LIKE), case-insensitive searches, and full-text search integration.
4. **Multiple Filters** — combining filter options using logical AND vs. OR rules.
5. **Sorting** — specifying sort order and multiple fields dynamically.
6. **Dynamic Query Construction** — safely building SQL or NoSQL queries from arbitrary URL parameters.

---

## Core Concept 1: Exact Filters

### Definitions

**Core Definition:** Exact filters map a query parameter directly to an equality condition on a database field, returning only records whose value matches exactly.

**Technical Definition:** An exact filter uses a query parameter key that corresponds to a database column name, with the value serving as the equality condition. For example, `?status=active` translates to `WHERE status = 'active'`. In Express, the parameter is read from `req.query.status` and bound to the database query using a parameterized placeholder (e.g., `$1` in PostgreSQL or `?` in MySQL). The field name must be validated against an allow-list of filterable columns to prevent injection.

**Beginner-Friendly Explanation:** Exact filtering is like asking a librarian for books by a specific author. You say "show me books by Stephen King" and you get exactly those books — not similar authors, not partial matches, just the ones that match exactly.

### Purposes

- To filter records by exact field values (e.g., `?status=active`, `?role=admin`).
- To provide the simplest and most intuitive filtering mechanism for clients.
- To map directly to indexed columns for fast query performance.
- To serve as the foundation for more complex filter combinations.

### Syntax Rules and Structure

#### Query Parameter Syntax

```
GET /api/users?status=active
GET /api/products?category=electronics
```

#### SQL Translation

```sql
SELECT * FROM users WHERE status = $1;
-- Parameters: ['active']
```

| Component | Breakdown |
|-----------|-----------|
| `status` | The database column name (must be in the allow-list). |
| `active` | The exact value to match. |
| `$1` | Parameterized placeholder (PostgreSQL). |
| `?` | Parameterized placeholder (MySQL). |

#### Syntax Rules

- The query parameter key **must match** the database column name (or be mapped to it).
- The parameter value is bound using a **placeholder** — never concatenated into the SQL string.
- The field name **must be validated** against an allow-list of filterable columns.
- Values are always strings from `req.query`; cast to the correct database type when necessary.

#### Constraints and Limitations

- Only equality matching is supported; range queries require additional operators.
- Filtering on non-indexed columns can cause full-table scans on large datasets.
- Values must be sanitized and validated before use (e.g., `status` must be one of `active`, `inactive`, `pending`).

### Annotated Code Example

```js
// exact-filters.js
const express = require('express');
const { Pool } = require('pg');
const app = express();
const pool = new Pool();

// Allow-list of filterable columns
const ALLOWED_FILTERS = ['status', 'role', 'category'];

app.get('/api/users', async (req, res) => {
  const conditions = [];
  const values = [];
  let paramIndex = 1;

  // Build WHERE clause dynamically from exact filters
  for (const [key, value] of Object.entries(req.query)) {
    if (!ALLOWED_FILTERS.includes(key)) continue; // Security: whitelist
    conditions.push(`${key} = $${paramIndex}`);
    values.push(value);
    paramIndex++;
  }

  const whereClause = conditions.length > 0
    ? `WHERE ${conditions.join(' AND ')}`
    : '';

  const query = `SELECT id, name, email, status, role FROM users ${whereClause}`;

  try {
    const result = await pool.query(query, values);
    res.json({ data: result.rows, count: result.rowCount });
  } catch (err) {
    res.status(500).json({ error: 'Query failed' });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users?status=active&role=admin`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", "email": "alice@example.com", "status": "active", "role": "admin" }
  ],
  "count": 1
}
```

**Why this output:** The handler iterates over `req.query`, skips parameters not in the `ALLOWED_FILTERS` allow-list, and builds a parameterized `WHERE` clause. The `conditions` array contains `status = $1` and `role = $2`, with `values` containing `['active', 'admin']`. The database executes the parameterized query, returning only users matching both conditions.

### Real-World Cases

- **User management:** `?status=active` to list only active users.
- **E-commerce:** `?category=electronics` to browse products in a specific category.
- **Order processing:** `?orderStatus=pending` to list orders awaiting fulfilment.
- **CRM:** `?leadSource=website` to filter leads by acquisition channel.

---

## Core Concept 2: Range Filters

### Definitions

**Core Definition:** Range filters retrieve records whose field values fall within a specified numeric, chronological, or alphabetical window — for example, products priced between $10 and $50, or orders placed in the last 30 days.

**Technical Definition:** Range filters use comparison operators (`gt`, `gte`, `lt`, `lte`, `ne`, `in`, `between`) expressed in the query string, either as bracketed objects (`?price[gte]=10&price[lte]=50`) or as field suffixes (`?price_gte=10&price_lte=50`). In Express 4, the default `qs` parser converts bracket notation into nested objects (`req.query.price = { gte: '10', lte: '50' }`); in Express 5, the default "simple" parser does not support nested objects, so field suffix notation (`price_gte`) is recommended. These operators are mapped to SQL comparison operators or ORM operators.

**Beginner-Friendly Explanation:** Range filtering is like asking a librarian for "books published between 2020 and 2024" or "books costing less than $20." You're specifying a window rather than an exact value. The API translates this into a database query that finds everything within that range.

### Purposes

- To handle numeric or chronological windows (price, age, date ranges).
- To enable "greater than", "less than", "between", and "not equal" comparisons.
- To support date-based filtering for reports, analytics, and historical queries.
- To provide flexible filtering beyond exact equality.

### Syntax Rules and Structure

#### Bracket Notation (Express 4)

```
GET /api/products?price[gte]=10&price[lte]=50
GET /api/orders?createdAt[gte]=2026-01-01&createdAt[lte]=2026-01-31
```

#### Field Suffix Notation (Express 5)

```
GET /api/products?price_gte=10&price_lte=50
GET /api/orders?createdAt_gte=2026-01-01&createdAt_lte=2026-01-31
```

#### SQL Translation

```sql
SELECT * FROM products WHERE price >= $1 AND price <= $2;
-- Parameters: [10, 50]
```

| Operator | SQL Equivalent | Meaning |
|----------|---------------|---------|
| `gt` | `>` | Greater than |
| `gte` | `>=` | Greater than or equal |
| `lt` | `<` | Less than |
| `lte` | `<=` | Less than or equal |
| `ne` | `!=` | Not equal |
| `in` | `IN (...)` | In a list of values |
| `between` | `BETWEEN ... AND ...` | Inclusive range |

#### Syntax Rules

- **Express 4:** Bracket notation works with the default `qs` (extended) parser.
- **Express 5:** The default parser changed from "extended" to "simple". Use field suffix notation (`price_gte`) or configure the extended parser with `app.set('query parser', 'extended')`.
- Operators must be **whitelisted** — only the operators your API supports should be accepted.
- Values must be **cast** to the correct type (numeric, date) before binding to the query.
- Date values should use ISO 8601 format (`2026-01-15T00:00:00Z`).

#### Constraints and Limitations

- Bracket notation requires the extended query parser, which has prototype pollution considerations in some versions.
- Field suffix notation is more parser-independent but requires string manipulation to extract the operator.
- Range filters on non-indexed columns are slow on large datasets.
- The `in` operator with many values can degrade performance; consider using `= ANY($1)` with an array parameter in PostgreSQL.

### Annotated Code Example

```js
// range-filters.js
const express = require('express');
const { Pool } = require('pg');
const app = express();
const pool = new Pool();

// Use extended parser for bracket notation (Express 4 style)
// In Express 5, use field suffix notation instead
app.set('query parser', 'extended');

const ALLOWED_RANGE_FIELDS = ['price', 'createdAt', 'age'];
const RANGE_OPERATORS = {
  gte: '>=',
  lte: '<=',
  gt: '>',
  lt: '<',
  ne: '!='
};

app.get('/api/products', async (req, res) => {
  const conditions = [];
  const values = [];
  let paramIndex = 1;

  for (const [field, operators] of Object.entries(req.query)) {
    if (!ALLOWED_RANGE_FIELDS.includes(field)) continue;

    // Handle both bracket notation (object) and plain values
    if (typeof operators === 'object') {
      for (const [op, value] of Object.entries(operators)) {
        const sqlOp = RANGE_OPERATORS[op];
        if (!sqlOp) continue; // Security: whitelist operators

        conditions.push(`${field} ${sqlOp} $${paramIndex}`);
        values.push(value);
        paramIndex++;
      }
    }
  }

  const whereClause = conditions.length > 0
    ? `WHERE ${conditions.join(' AND ')}`
    : '';

  const query = `SELECT id, name, price, category FROM products ${whereClause}`;

  try {
    const result = await pool.query(query, values);
    res.json({ data: result.rows, count: result.rowCount });
  } catch (err) {
    res.status(500).json({ error: 'Query failed' });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/products?price[gte]=10&price[lte]=50`):**
```json
{
  "data": [
    { "id": 3, "name": "Keyboard", "price": 45, "category": "accessories" },
    { "id": 5, "name": "Mouse", "price": 25, "category": "accessories" }
  ],
  "count": 2
}
```

**Why this output:** The extended parser converts `price[gte]=10&price[lte]=50` into `req.query.price = { gte: '10', lte: '50' }`. The handler iterates over the `price` object, maps `gte` to `>=` and `lte` to `<=`, and builds the parameterized condition `price >= $1 AND price <= $2` with values `[10, 50]`. Only products within that price range are returned.

### Real-World Cases

- **E-commerce:** `?price[gte]=100&price[lte]=500` for price-range browsing.
- **Analytics:** `?createdAt[gte]=2026-01-01&createdAt[lte]=2026-01-31` for monthly reports.
- **HR systems:** `?age_gte=18&age_lte=65` for workforce filtering.
- **Real estate:** `?bedrooms_gte=2&price_lte=500000` for property searches.

---

## Core Concept 3: Text Searches

### Definitions

**Core Definition:** Text search filters return records containing a specified substring or matching a natural-language query, using partial matching (`LIKE`), case-insensitive comparison, or dedicated full-text search engines.

**Technical Definition:** Text search spans a spectrum from simple `LIKE '%term%'` patterns to full-text search using PostgreSQL's `tsvector`/`tsquery` types and GIN indexes, or external engines like Elasticsearch. PostgreSQL's `tsvector` is a sorted list of normalized lexemes with positional information; `tsquery` represents a search query with boolean operators. The `@@` operator performs the match. For production use, always store `tsvector` as a column and index it with GIN to avoid computing `to_tsvector` on every row.

**Beginner-Friendly Explanation:** Text search is like asking a librarian to find books that mention "space exploration" somewhere in their content. There are different levels: a simple search that looks for the exact phrase, a smarter search that understands word variations (searching for "run" also finds "running"), and an advanced search engine that ranks results by relevance.

### Purposes

- To implement partial match filtering (`LIKE`) for quick text searches.
- To perform case-insensitive searches across text columns.
- To integrate full-text search engines for natural-language queries, ranking, and highlighting.
- To support search-as-you-type and autocomplete features.

### Sub-Feature 3.1: Partial Match with LIKE

#### Syntax Rules and Structure

```sql
SELECT * FROM articles WHERE title ILIKE $1;
-- Parameters: ['%node%']  -- Case-insensitive
```

| Component | Breakdown |
|-----------|-----------|
| `LIKE` | Case-sensitive pattern matching. |
| `ILIKE` | Case-insensitive pattern matching (PostgreSQL). |
| `%` | Wildcard matching zero or more characters. |
| `_` | Wildcard matching exactly one character. |

#### Constraints

- `LIKE '%term%'` cannot use standard B-tree indexes; performance degrades on large tables.
- For case-insensitive search in MySQL, use `LOWER(column) LIKE LOWER(?)`.
- Always parameterize the pattern (`%${value}%`) — never concatenate into SQL.

---

### Sub-Feature 3.2: PostgreSQL Full-Text Search

#### Syntax Rules and Structure

```sql
-- Add a tsvector column and GIN index
ALTER TABLE articles ADD COLUMN search_vector tsvector;
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);

-- Query using websearch_to_tsquery
SELECT id, title, ts_rank(search_vector, query) AS rank
FROM articles, websearch_to_tsquery('english', $1) query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

| Component | Breakdown |
|-----------|-----------|
| `tsvector` | Pre-computed sorted list of lexemes. |
| `tsquery` | Search query with boolean operators. |
| `@@` | Match operator. |
| `websearch_to_tsquery` | Converts natural-language input to `tsquery`. |
| `ts_rank` | Ranks results by relevance. |
| GIN index | Generalized Inverted Index for fast lookups. |

#### Rules

- Store `tsvector` as a **column** and index it with GIN — computing `to_tsvector` on every row during `SELECT` is expensive and prevents index usage.
- Use `websearch_to_tsquery` for user-facing search input; it supports natural syntax like `"react server" OR components`.
- Use `ts_rank` to order results by relevance.
- For fuzzy matching, combine with `pg_trgm` and trigram indexes.

### Annotated Code Example

```js
// text-search.js
const express = require('express');
const { Pool } = require('pg');
const app = express();
const pool = new Pool();

// PostgreSQL full-text search endpoint
app.get('/api/articles/search', async (req, res) => {
  const q = req.query.q;

  if (!q || q.trim().length === 0) {
    return res.status(400).json({ error: 'Query parameter "q" is required' });
  }

  if (q.length > 200) {
    return res.status(400).json({ error: 'Search term too long' });
  }

  try {
    // Use websearch_to_tsquery for natural language input
    const result = await pool.query(
      `SELECT id, title, ts_rank(search_vector, query) AS rank
       FROM articles,
            websearch_to_tsquery('english', $1) query
       WHERE search_vector @@ query
       ORDER BY rank DESC
       LIMIT 20`,
      [q]
    );

    res.json({
      query: q,
      count: result.rowCount,
      results: result.rows
    });
  } catch (err) {
    res.status(500).json({ error: 'Search failed' });
  }
});

// Simple LIKE search for partial matches
app.get('/api/articles/like', async (req, res) => {
  const q = req.query.q;
  if (!q) return res.status(400).json({ error: 'q is required' });

  const result = await pool.query(
    `SELECT id, title FROM articles
     WHERE title ILIKE $1
     LIMIT 20`,
    [`%${q}%`]
  );

  res.json({ query: q, results: result.rows });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/articles/search?q=express routing`):**
```json
{
  "query": "express routing",
  "count": 2,
  "results": [
    { "id": 1, "title": "Express Routing Guide", "rank": 0.6079 },
    { "id": 5, "title": "Advanced Express Patterns", "rank": 0.3041 }
  ]
}
```

**Expected Output (for `GET /api/articles/like?q=routing`):**
```json
{
  "query": "routing",
  "results": [
    { "id": 1, "title": "Express Routing Guide" }
  ]
}
```

**Why this output:** The `/search` endpoint uses `websearch_to_tsquery` to convert "express routing" into a `tsquery`, matches it against the pre-computed `search_vector` column using `@@`, and ranks results by `ts_rank`. The `/like` endpoint uses parameterized `ILIKE` with wildcards for simple partial matching.

### Real-World Cases

- **Documentation search:** Full-text search across help articles and API docs.
- **E-commerce product search:** PostgreSQL FTS for product titles and descriptions.
- **Content platforms:** Elasticsearch for blog post search with highlighting and facets.
- **Log analysis:** Full-text search across log messages for incident investigation.

---

## Core Concept 4: Multiple Filters (AND vs. OR)

### Definitions

**Core Definition:** Multiple filters allow clients to combine several filter conditions in a single request, using either logical AND (all conditions must match) or logical OR (at least one condition must match) semantics.

**Technical Definition:** AND filtering combines conditions in the SQL `WHERE` clause with `AND` operators; OR filtering uses `OR` operators or `IN` clauses. In Express, multiple query parameters naturally form an AND relationship (e.g., `?status=active&role=admin` means both conditions must be true). OR semantics require explicit syntax, such as the `in` operator (`?status_in=active,pending`) or a Sequelize `Op.or` array.

**Beginner-Friendly Explanation:** AND filtering is like saying "show me red cars that are also convertibles" — both conditions must be true. OR filtering is like saying "show me red cars or blue cars" — either condition can be true.

### Purposes

- To narrow results by combining multiple independent filter conditions (AND).
- To broaden results by accepting any one of several matching conditions (OR).
- To enable complex search forms with multiple optional fields.
- To support faceted search where users select multiple values for the same attribute (e.g., multiple categories).

### Syntax Rules and Structure

#### AND Filtering

```
GET /api/products?category=electronics&inStock=true&brand=Apple
```

#### OR Filtering (In List)

```
GET /api/products?status_in=active,pending
```

#### SQL Translation

```sql
-- AND
SELECT * FROM products
WHERE category = $1 AND in_stock = $2 AND brand = $3;

-- OR (IN)
SELECT * FROM products
WHERE status IN ($1, $2);
```

#### Sequelize OR Example

```js
const where = {
  [Op.or]: [
    { name: { [Op.like]: `%${search}%` } },
    { email: { [Op.like]: `%${search}%` } }
  ]
};
```

**Rules:**
- Multiple query parameters naturally form an **AND** relationship.
- OR semantics require the `in` operator or an explicit `Op.or` array.
- All field names and operator names must be **whitelisted**.
- The search parameter in Sequelize uses `Op.like` to perform partial matching across multiple fields.

### Annotated Code Example

```js
// multiple-filters.js
const express = require('express');
const app = express();

// Simulated dataset
const products = [
  { id: 1, name: 'Laptop', category: 'electronics', brand: 'Apple', price: 1200, status: 'active' },
  { id: 2, name: 'Phone', category: 'electronics', brand: 'Samsung', price: 800, status: 'active' },
  { id: 3, name: 'Tablet', category: 'electronics', brand: 'Apple', price: 600, status: 'pending' },
  { id: 4, name: 'Desk', category: 'furniture', brand: 'IKEA', price: 350, status: 'active' }
];

app.get('/api/products', (req, res) => {
  const { category, brand, status_in } = req.query;
  let result = products;

  // AND filtering: each condition narrows the result
  if (category) {
    result = result.filter(p => p.category === category);
  }
  if (brand) {
    result = result.filter(p => p.brand === brand);
  }

  // OR filtering: status_in accepts multiple values
  if (status_in) {
    const statuses = status_in.split(',').map(s => s.trim());
    result = result.filter(p => statuses.includes(p.status));
  }

  res.json({ data: result, count: result.length });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/products?category=electronics&brand=Apple`):**
```json
{
  "data": [
    { "id": 1, "name": "Laptop", "category": "electronics", "brand": "Apple", "price": 1200, "status": "active" },
    { "id": 3, "name": "Tablet", "category": "electronics", "brand": "Apple", "price": 600, "status": "pending" }
  ],
  "count": 2
}
```

**Expected Output (for `GET /api/products?category=electronics&status_in=active,pending`):**
```json
{
  "data": [
    { "id": 1, "name": "Laptop", "category": "electronics", "brand": "Apple", "price": 1200, "status": "active" },
    { "id": 2, "name": "Phone", "category": "electronics", "brand": "Samsung", "price": 800, "status": "active" },
    { "id": 3, "name": "Tablet", "category": "electronics", "brand": "Apple", "price": 600, "status": "pending" }
  ],
  "count": 3
}
```

**Why this output:** The `category` and `brand` filters are applied as AND conditions (each narrows the result). The `status_in` filter is applied as an OR condition (any of `active` or `pending` matches). The combined query returns electronics products from Apple that are either active or pending.

### Real-World Cases

- **E-commerce:** `?category=electronics&brand=Apple&price_lte=1000` for precise product filtering.
- **User management:** `?status_in=active,pending&role=admin` for admin dashboards.
- **CRM:** `?source_in=website,referral&createdAt_gte=2026-01-01` for lead analysis.
- **Content platforms:** `?author=alice&tag_in=nodejs,express` for filtering posts by author and tags.

---

## Core Concept 5: Sorting

### Definitions

**Core Definition:** Sorting allows clients to specify the order in which results are returned, using a `sort` query parameter that accepts field names with optional direction modifiers (e.g., `?sort=-createdAt,title` where `-` denotes descending order).

**Technical Definition:** The `sort` parameter is parsed by splitting on commas and interpreting a leading `-` as descending order and an optional `+` or no prefix as ascending. Each field is mapped to an `ORDER BY` clause in SQL or an equivalent sort object in the ORM. The field names must be whitelisted — only pre-approved sortable columns should be accepted. Multiple sort fields are applied in order, creating a composite sort key.

**Beginner-Friendly Explanation:** Sorting is like telling a librarian "arrange the books by newest first, and for books published on the same date, arrange them alphabetically by title." The `-` prefix is a shorthand for "descending" (newest first, highest first).

### Purposes

- To specify sort order and multiple fields dynamically.
- To support user-controlled result ordering (newest first, price low-to-high, etc.).
- To enable deterministic pagination by ensuring a stable sort order.
- To combine with pagination for consistent page-by-page traversal.

### Syntax Rules and Structure

#### Query Parameter Syntax

```
GET /api/products?sort=-createdAt
GET /api/products?sort=-price,name
GET /api/users?sort=lastName,-createdAt
```

#### SQL Translation

```sql
SELECT * FROM products ORDER BY price DESC, name ASC;
```

| Syntax | Meaning |
|--------|---------|
| `sort=name` | Sort by `name` ascending. |
| `sort=-name` | Sort by `name` descending. |
| `sort=-price,name` | Sort by `price` descending, then `name` ascending. |

#### Parsing Pattern

```js
const sort = req.query.sort || '-createdAt';
const order = sort.split(',').map(field => {
  const direction = field.startsWith('-') ? 'DESC' : 'ASC';
  const column = field.replace(/^[+-]/, '');
  return [column, direction];
});
```

#### Rules

- The `-` prefix indicates descending order; no prefix or `+` indicates ascending order.
- Multiple sort fields are comma-separated.
- All sort field names must be validated against an allow-list.
- A stable sort key (e.g., `id`) should be included as a tiebreaker to prevent duplicate records across pages.
- The `sort` parameter name should be consistent across all endpoints.

#### Constraints and Limitations

- Sorting on non-indexed columns is slow on large datasets.
- Multi-column sorts require a composite index matching the sort fields.
- The `sort` parameter must not be confused with pagination parameters.

### Annotated Code Example

```js
// sorting.js
const express = require('express');
const app = express();

const products = [
  { id: 1, name: 'Laptop', price: 1200, createdAt: '2026-01-10' },
  { id: 2, name: 'Phone', price: 800, createdAt: '2026-01-15' },
  { id: 3, name: 'Tablet', price: 600, createdAt: '2026-01-05' },
  { id: 4, name: 'Monitor', price: 800, createdAt: '2026-01-12' }
];

const ALLOWED_SORT_FIELDS = ['name', 'price', 'createdAt'];

app.get('/api/products', (req, res) => {
  const sortParam = req.query.sort || '-createdAt';

  // Parse sort parameter: "field1,-field2" → [['field1', 'ASC'], ['field2', 'DESC']]
  const sortFields = sortParam.split(',').map(field => {
    const trimmed = field.trim();
    const direction = trimmed.startsWith('-') ? 'DESC' : 'ASC';
    const column = trimmed.replace(/^[+-]/, '');
    return { column, direction };
  });

  // Validate all sort fields against whitelist
  const invalid = sortFields.find(f => !ALLOWED_SORT_FIELDS.includes(f.column));
  if (invalid) {
    return res.status(400).json({
      error: `Invalid sort field: ${invalid.column}`,
      allowed: ALLOWED_SORT_FIELDS
    });
  }

  // Apply sorting
  const sorted = [...products].sort((a, b) => {
    for (const { column, direction } of sortFields) {
      const aVal = a[column];
      const bVal = b[column];
      if (aVal === bVal) continue;
      const cmp = aVal > bVal ? 1 : -1;
      return direction === 'ASC' ? cmp : -cmp;
    }
    return a.id - b.id; // Tiebreaker
  });

  res.json({ data: sorted, count: sorted.length });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/products?sort=-price,name`):**
```json
{
  "data": [
    { "id": 1, "name": "Laptop", "price": 1200, "createdAt": "2026-01-10" },
    { "id": 4, "name": "Monitor", "price": 800, "createdAt": "2026-01-12" },
    { "id": 2, "name": "Phone", "price": 800, "createdAt": "2026-01-15" },
    { "id": 3, "name": "Tablet", "price": 600, "createdAt": "2026-01-05" }
  ],
  "count": 4
}
```

**Why this output:** The sort parameter `-price,name` is parsed into `[{ column: 'price', direction: 'DESC' }, { column: 'name', direction: 'ASC' }]`. Products are sorted by price descending (Laptop $1200 first), and for the two products with the same price ($800), they are sorted by name ascending (Monitor before Phone).

### Real-World Cases

- **E-commerce:** `?sort=price` (cheapest first) or `?sort=-price` (most expensive first).
- **Blogs:** `?sort=-publishedAt` for newest posts first.
- **User directories:** `?sort=lastName,firstName` for alphabetical ordering.
- **Analytics:** `?sort=-revenue` for top-performing products.

---

## Core Concept 6: Dynamic Query Construction

### Definitions

**Core Definition:** Dynamic query construction is the process of building database queries programmatically from arbitrary request parameters, while strictly preventing injection vulnerabilities through parameterized queries, field whitelisting, and operator whitelisting.

**Technical Definition:** SQL injection happens when attacker-controlled input is concatenated directly into a SQL query string. The single most effective prevention is parameterized queries (also called prepared statements). With parameterized queries, the SQL structure is fixed at prepare time and user input is sent as separate, typed values that the database cannot interpret as SQL. However, parameterized queries only protect **values** — they cannot parameterize **identifiers** (column names, table names, operators). Therefore, dynamic query construction requires three layers of defence: (1) parameterized value binding, (2) field name whitelisting, and (3) operator whitelisting.

**Beginner-Friendly Explanation:** Dynamic query construction is like building a custom order at a restaurant. You tell the chef what you want (filter by price, sort by name), and the chef follows a standard recipe but adjusts the ingredients. The chef never lets you write on the recipe card yourself — you just say "more salt" and the chef adds salt. Parameterized queries are the chef's rule: "I'll read your order, but I'll write the recipe myself."

### Purposes

- To safely build SQL or NoSQL database queries dynamically from arbitrary request URL parameters.
- To strictly prevent injection vulnerabilities (SQL injection, NoSQL operator injection).
- To enable flexible filtering and sorting without hardcoding every possible query combination.
- To maintain a clean separation between query structure (trusted) and query values (untrusted).

### Syntax Rules and Structure

#### Three-Layer Defence

```js
// Layer 1: Parameterized value binding
pool.query('SELECT * FROM users WHERE status = $1', [status]);

// Layer 2: Field name whitelisting
if (!ALLOWED_FIELDS.includes(field)) return;

// Layer 3: Operator whitelisting
const ALLOWED_OPS = { eq: '=', gte: '>=', lte: '<=' };
if (!ALLOWED_OPS[op]) return;
```

| Layer | Protects Against | Mechanism |
|-------|-----------------|-----------|
| Parameterized values | SQL injection via values | Placeholders (`$1`, `?`). |
| Field whitelisting | SQL injection via column names | Allow-list of filterable columns. |
| Operator whitelisting | SQL injection via operators | Allow-list of comparison operators. |

#### Dynamic WHERE Clause Builder

```js
function buildWhereClause(filters, allowedFields, allowedOps) {
  const conditions = [];
  const values = [];
  let idx = 1;

  for (const [field, value] of Object.entries(filters)) {
    if (!allowedFields.includes(field)) continue;
    conditions.push(`${field} = $${idx++}`);
    values.push(value);
  }

  return {
    clause: conditions.length ? `WHERE ${conditions.join(' AND ')}` : '',
    values
  };
}
```

#### Syntax Rules

- **Never** concatenate user input into SQL strings. The `query` string should contain only placeholders and whitelisted identifiers.
- **Always** use parameterized queries for values — every mainstream Node.js driver supports them.
- **Always** whitelist field names (column names) and operator names.
- Use `express-validator` or `zod` to validate and sanitize query parameters before building the query.
- For MongoDB, install `express-mongo-sanitize` to strip `$` and `.` characters from user input.

#### Constraints and Limitations

- Parameterized queries protect values but **cannot** parameterize column names, table names, or SQL keywords. Whitelisting is mandatory for these.
- ORMs like Sequelize and Prisma provide built-in parameterization but have raw-query escape hatches that can reintroduce injection if misused.
- NoSQL databases (MongoDB) are vulnerable to operator injection (`$gt`, `$ne`) — sanitize query objects.

### Annotated Code Example

```js
// dynamic-query-builder.js
const express = require('express');
const { Pool } = require('pg');
const app = express();
const pool = new Pool();

// Security configuration
const ALLOWED_FIELDS = ['status', 'role', 'category', 'price', 'createdAt'];
const ALLOWED_OPERATORS = {
  eq: '=',
  ne: '!=',
  gt: '>',
  gte: '>=',
  lt: '<',
  lte: '<='
};

/**
 * Safely builds a parameterized SQL query from request query parameters.
 * @param {Object} query - Express req.query object
 * @returns {{ text: string, values: Array }} Parameterized query object
 */
function buildQuery(query) {
  const conditions = [];
  const values = [];
  let paramIndex = 1;

  // Pagination and sorting are handled separately
  const EXCLUDED = ['page', 'limit', 'sort', 'fields'];

  for (const [key, value] of Object.entries(query)) {
    if (EXCLUDED.includes(key)) continue;

    // Parse field and operator (e.g., "price_gte" → field="price", op="gte")
    const operators = Object.keys(ALLOWED_OPERATORS);
    const op = operators.find(o => key.endsWith(`_${o}`)) || 'eq';
    const field = op === 'eq' ? key : key.slice(0, -(op.length + 1));

    // Security: whitelist field names
    if (!ALLOWED_FIELDS.includes(field)) continue;

    // Security: whitelist operators
    const sqlOp = ALLOWED_OPERATORS[op];
    if (!sqlOp) continue;

    // Add parameterized condition
    conditions.push(`${field} ${sqlOp} $${paramIndex}`);
    values.push(value);
    paramIndex++;
  }

  const where = conditions.length > 0
    ? `WHERE ${conditions.join(' AND ')}`
    : '';

  return {
    text: `SELECT id, name, email, status, role, category, price FROM users ${where}`,
    values
  };
}

app.get('/api/users', async (req, res) => {
  const { text, values } = buildQuery(req.query);

  try {
    const result = await pool.query(text, values);
    res.json({ data: result.rows, count: result.rowCount });
  } catch (err) {
    console.error('Query error:', err.message);
    res.status(500).json({ error: 'Query failed' });
  }
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users?status=active&price_gte=100`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", "email": "alice@example.com", "status": "active", "role": "admin", "category": "electronics", "price": 1200 }
  ],
  "count": 1
}
```

**Expected Output (for `GET /api/users?unknownField=value&price_gte=100`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", ... "price": 1200 }
  ],
  "count": 1
}
```

**Why this output:** The `buildQuery` function iterates over `req.query`, skips excluded keys (pagination/sorting), parses field-operator pairs, and checks each field against `ALLOWED_FIELDS` and each operator against `ALLOWED_OPERATORS`. Only whitelisted fields and operators are included in the SQL string. All values are bound as parameterized placeholders (`$1`, `$2`). The `unknownField` parameter is silently ignored because it is not in the allow-list, preventing any injection attempt from affecting the query structure.

### Real-World Cases

- **Public APIs:** Safely handling arbitrary filter parameters from untrusted clients.
- **Multi-tenant SaaS:** Building dynamic queries for tenant-specific data views.
- **Admin dashboards:** Allowing administrators to filter and sort by any whitelisted column.
- **Data export tools:** Dynamically constructing queries for CSV/JSON exports with user-selected filters.
- **GraphQL resolvers:** Mapping GraphQL arguments to parameterized database queries.

---

## References

- OneUptime — How to Implement Filtering and Sorting in REST APIs — https://oneuptime.com/blog/post/2026-01-26-rest-api-filtering-sorting/view
- Shattered.io — SQL Injection Prevention in Node.js: 12 Steps [2026] — https://shattered.io/sql-injection-prevention-nodejs/
- CoreUI — How to handle filtering in Node.js APIs — https://coreui.io/answers/how-to-handle-filtering-in-nodejs-apis/
- Grizzly Peak Software — PostgreSQL Full-Text Search: Implementation Guide — https://www.grizzlypeaksoftware.com/library/postgresql-full-text-search-implementation-guide-3ojmi4ub
- REST API Query Parameters: Filtering, Pagination, Sorting, and Best Practices — https://www.cleverence.com
- Safeguard.sh — SQL Injection Prevention in Node.js and JavaScript — https://safeguard.sh
- DevBytes — Dynamic APIs with Query Parameters — https://devbytes.co.in
- PostgreSQL Full-Text Search: tsvector, tsquery, GIN indexes — https://docs.postgresql.org
- express-query-parser2 — npm — https://www.npmjs.com/package/express-query-parser2
- express-zod-api — npm — https://www.npmjs.com/package/express-zod-api
- express-mongo-sanitize — npm — https://www.npmjs.com/package/express-mongo-sanitize
- @exortek/express-mongo-sanitize — npm — https://www.npmjs.com/package/@exortek/express-mongo-sanitize
- hppx — npm — https://www.npmjs.com/package/hppx
- @rxstack/query-filter — npm — https://www.npmjs.com/package/@rxstack/query-filter
- prisma-smart-query — npm — https://www.npmjs.com/package/prisma-smart-query
- MongoDB — Query Documents — https://docs.mongodb.com/manual/tutorial/query-documents/
- PostgreSQL — Full Text Search — https://www.postgresql.org/docs/current/textsearch.html
- Elasticsearch — Full-Text Search with Express — https://medium.com
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP — Top Ten 2021: A03 Injection — https://owasp.org/Top10/A03_2021-Injection/
- RFC 3339 — Date and Time on the Internet: Timestamps — https://datatracker.ietf.org/doc/html/rfc3339