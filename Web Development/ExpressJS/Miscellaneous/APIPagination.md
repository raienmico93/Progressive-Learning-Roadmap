# API Pagination — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API pagination is the technique of dividing a large dataset into smaller, manageable chunks (called "pages") that clients can retrieve incrementally, rather than receiving the entire dataset in a single response.

**Technical Definition:** Pagination is a technique used to take a REST API endpoint's response and segment it into smaller, more manageable units, often called "pages." Instead of delivering a potentially massive dataset in one go, the API returns a small, predictable chunk of the data along with metadata that allows a client to incrementally fetch subsequent chunks. The two primary strategies are **offset-based pagination** (using `LIMIT` and `OFFSET` in SQL) and **cursor-based pagination** (also called keyset or seek pagination, using a pointer to the last-seen record).

**Beginner-Friendly Explanation:** Imagine a library with a million books. Instead of dumping all one million books on your desk at once, the librarian gives you a cart that holds 20 books at a time. When you finish with those 20, you come back and get the next 20. Pagination is the system that tells you which 20 books to get next, and how to know when you've seen them all.

### Key Characteristics

- **Performance:** Reduces database load, memory usage, network transfer, and client-side rendering time.
- **Scalability:** Enables APIs to handle millions of records without timing out.
- **Consistency:** Well-designed pagination prevents duplicate or missing records during traversal.
- **Two main strategies:** Offset-based (simple, supports random access) and cursor-based (scalable, sequential access only).
- **Metadata-driven:** Responses include pagination metadata (next links, total counts, cursors) so clients know how to navigate.
- **Sort-dependent:** Pagination must be combined with a stable sort order to function correctly.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A database** (PostgreSQL, MySQL, MongoDB, etc.) with indexed sort columns.
- **Basic JavaScript knowledge:** Async/await, objects, and arrays.
- **Understanding of SQL:** `LIMIT`, `OFFSET`, `WHERE`, and `ORDER BY` clauses.

### Related Programming Areas

- **Database indexing:** Proper indexes on sort columns are essential for pagination performance.
- **API design:** Pagination is a core component of RESTful API design.
- **Sorting and filtering:** Pagination interacts with sort order and filter conditions.
- **Caching:** Paginated responses can be cached individually.
- **Rate limiting:** Pagination reduces the need for large payloads, easing rate-limit pressure.
- **Infinite scroll:** Cursor pagination is the foundation of infinite-scroll UIs.

### Core Concepts

1. **Offset Pagination** — implementation, syntax, and performance degradation on large datasets.
2. **Cursor Pagination (Keyset)** — implementation using unique sequential pointers and why it scales efficiently.
3. **Pagination Metadata** — returning data control links via headers (Link) or JSON envelope.
4. **Sorting & Filtering Integration** — how pagination interacts with sorted indexes to prevent duplicates or missing records.

---

## Core Concept 1: Offset Pagination

### Definitions

**Core Definition:** Offset pagination divides a dataset by skipping a specified number of records (`OFFSET`) and returning a limited number (`LIMIT`), typically driven by `page` and `limit` query parameters.

**Technical Definition:** Offset pagination maps directly to SQL. The client sends a page number and a page size; the server translates them into `LIMIT` and `OFFSET`. For example, `?page=3&per_page=25` becomes `LIMIT 25 OFFSET 50`. The database must scan all rows up to the offset, even though it discards them, making performance degrade linearly with page depth.

**Beginner-Friendly Explanation:** Offset pagination is like counting rows in a spreadsheet. To get to page 5 (rows 101–125), the database counts from row 1, discards rows 1–100, and returns rows 101–125. The deeper you go, the more rows the database has to count and discard.

### Purposes

- To provide simple, intuitive pagination that maps directly to SQL `LIMIT`/`OFFSET`.
- To enable clients to jump to any page directly (`?page=47`).
- To support traditional page-number interfaces in admin panels and tables.
- To return total counts and total pages for UI pagination controls.

### Syntax Rules and Structure

#### Query Parameters

```
GET /api/users?page=3&limit=20
```

| Parameter | Description |
|-----------|-------------|
| `page` | Page number (1-indexed). |
| `limit` | Items per page. |

#### SQL Translation

```sql
SELECT id, name, email FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;
```

| Component | Breakdown |
|-----------|-----------|
| `LIMIT` | Number of rows to return. |
| `OFFSET` | Number of rows to skip (`(page - 1) * limit`). |
| `ORDER BY` | **Critical** — pagination is meaningless without a deterministic sort. |

#### Syntax Rules

- `offset` is calculated as `(page - 1) * limit`.
- Always enforce a maximum `limit` (e.g., 100) to prevent abuse.
- Provide sensible defaults (`page = 1`, `limit = 20`).
- Always include an `ORDER BY` clause; without it, the database may return rows in arbitrary order.

#### Constraints and Limitations

- **Deep-offset performance degradation:** `OFFSET 1000000` forces the database to scan and discard 1,000,000 rows. Performance is O(n), degrading linearly with page depth.
- **Page drift:** When rows are inserted or deleted between requests, page boundaries shift, causing duplicate or missing records.
- **COUNT(*) overhead:** Calculating `total` for pagination metadata requires a full count, which is expensive on large tables.
- **DoS vulnerability:** Deep pagination queries can be used for denial-of-service attacks.

### Annotated Code Example

```js
// offset-pagination.js
const express = require('express');
const app = express();

// Simulated database (1000 records)
const users = Array.from({ length: 1000 }, (_, i) => ({
  id: i + 1,
  name: `User ${i + 1}`,
  createdAt: new Date(Date.now() - i * 86400000).toISOString()
}));

app.get('/api/users', (req, res) => {
  // Parse and validate pagination parameters
  const page = Math.max(parseInt(req.query.page, 10) || 1, 1);
  const limit = Math.min(Math.max(parseInt(req.query.limit, 10) || 20, 1), 100);
  const offset = (page - 1) * limit;

  // Simulate DB query: SELECT ... ORDER BY createdAt DESC LIMIT ? OFFSET ?
  const sorted = [...users].sort(
    (a, b) => new Date(b.createdAt) - new Date(a.createdAt)
  );
  const paginated = sorted.slice(offset, offset + limit);
  const total = users.length;
  const totalPages = Math.ceil(total / limit);

  res.json({
    data: paginated,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users?page=2&limit=3`):**
```json
{
  "data": [
    { "id": 4, "name": "User 4", "createdAt": "..." },
    { "id": 5, "name": "User 5", "createdAt": "..." },
    { "id": 6, "name": "User 6", "createdAt": "..." }
  ],
  "pagination": {
    "page": 2,
    "limit": 3,
    "total": 1000,
    "totalPages": 334,
    "hasNext": true,
    "hasPrev": true
  }
}
```

**Why this output:** The handler calculates `offset = (2 - 1) * 3 = 3`, slices the sorted array from index 3 to 6, and returns the three records. The pagination metadata tells the client there are 334 total pages and that both next and previous pages exist.

### Real-World Cases

- **Admin dashboards:** Page-number tables with "jump to page" functionality.
- **Small datasets (< 10,000 records):** Where offset performance is acceptable.
- **Internal APIs:** Controlled usage where deep pagination is unlikely.
- **Legacy systems:** Where the database does not support efficient keyset queries.

---

## Core Concept 2: Cursor Pagination (Keyset Pagination)

### Definitions

**Core Definition:** Cursor pagination (also called keyset or seek pagination) uses an opaque pointer — the cursor — that encodes the position of the last-seen record, allowing the server to fetch the next set of records without scanning skipped rows.

**Technical Definition:** Cursor pagination uses an opaque token that points to a specific position in the result set. Instead of "give me page 3," clients say "give me items after this cursor." The cursor typically encodes the last seen value of the sort field. The database query uses a `WHERE` clause comparing the sort column to the cursor value, which an index on the sort column can answer by seeking directly to the position. This is O(1) per page regardless of depth.

**Beginner-Friendly Explanation:** Cursor pagination is like using a bookmark. Instead of counting from the beginning every time ("skip 1,000 rows"), you simply say "start after this bookmark." The database jumps straight to the bookmark using an index, so the 1,000th page is just as fast as the first page.

### Purposes

- To provide stable, efficient pagination for large, high-frequency datasets.
- To eliminate page drift (duplicate or missing records) during traversal.
- To enable infinite-scroll feeds and real-time data streams.
- To maintain consistent latency regardless of page depth.

### Syntax Rules and Structure

#### Query Parameters

```
GET /api/users?limit=20&cursor=eyJpZCI6MTAwfQ==
```

| Parameter | Description |
|-----------|-------------|
| `limit` | Items per page. |
| `cursor` | Opaque token encoding the last-seen sort value. |

#### SQL Translation (Keyset)

```sql
-- Page 1
SELECT id, name, email FROM users
ORDER BY id ASC
LIMIT 20;

-- Page 2 (using cursor = last seen id = 20)
SELECT id, name, email FROM users
WHERE id > 20
ORDER BY id ASC
LIMIT 20;
```

| Component | Breakdown |
|-----------|-----------|
| `WHERE id > 20` | Seek condition — starts after the last-seen record. |
| `ORDER BY id ASC` | Must match the cursor's sort column. |
| `LIMIT 20` | Items per page. |

#### Syntax Rules

- The cursor must be **opaque** — clients should not parse or construct it.
- The cursor should encode the sort key(s) and direction.
- A `null` or absent `nextCursor` indicates the last page.
- The `ORDER BY` column(s) must be **unique** or composed with a tiebreaker (e.g., `created_at DESC, id DESC`) to prevent duplicates.
- The cursor must be **stable** — changes to the sort criteria can invalidate it.

#### Constraints and Limitations

- **No random access:** Clients cannot jump to page 47; only sequential traversal is supported.
- **Requires a composite index:** The sort column(s) and tiebreaker must be indexed for optimal performance.
- **Opaque cursors can become invalid:** If the underlying dataset changes, cursor tokens may become invalid.
- **Tiebreaker required:** If the sort column has duplicates, a second stable column (e.g., `id`) must be included to avoid duplicates or gaps.

### Annotated Code Example

```js
// cursor-pagination.js
const express = require('express');
const app = express();

// Simulated database
const users = Array.from({ length: 1000 }, (_, i) => ({
  id: i + 1,
  name: `User ${i + 1}`
}));

// Encode cursor (opaque base64)
function encodeCursor(id) {
  return Buffer.from(JSON.stringify({ id })).toString('base64');
}

// Decode cursor
function decodeCursor(cursor) {
  try {
    return JSON.parse(Buffer.from(cursor, 'base64').toString());
  } catch {
    return null;
  }
}

app.get('/api/users', (req, res) => {
  const limit = Math.min(parseInt(req.query.limit, 10) || 20, 100);
  const cursor = req.query.cursor ? decodeCursor(req.query.cursor) : null;
  const lastId = cursor ? cursor.id : 0;

  // Simulate DB query: SELECT ... WHERE id > lastId ORDER BY id ASC LIMIT ?
  const filtered = users.filter(u => u.id > lastId);
  const paginated = filtered.slice(0, limit);

  // Determine if there is a next page
  const hasNext = filtered.length > limit;
  const nextCursor = hasNext
    ? encodeCursor(paginated[paginated.length - 1].id)
    : null;

  res.json({
    data: paginated,
    pagination: {
      nextCursor,
      hasNext,
      limit
    }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users?limit=3`):**
```json
{
  "data": [
    { "id": 1, "name": "User 1" },
    { "id": 2, "name": "User 2" },
    { "id": 3, "name": "User 3" }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6M30=",
    "hasNext": true,
    "limit": 3
  }
}
```

**Expected Output (for `GET /api/users?limit=3&cursor=eyJpZCI6M30=`):**
```json
{
  "data": [
    { "id": 4, "name": "User 4" },
    { "id": 5, "name": "User 5" },
    { "id": 6, "name": "User 6" }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6Nn0=",
    "hasNext": true,
    "limit": 3
  }
}
```

**Why this output:** The cursor `eyJpZCI6M30=` decodes to `{ id: 3 }`. The server filters users with `id > 3`, returns the next three, and encodes the new cursor as `{ id: 6 }` in base64. The client passes the cursor back to get the next page.

### Real-World Cases

- **Social media feeds:** Infinite scroll on Twitter/X, Instagram, Facebook.
- **Real-time data streams:** Logs, events, and messages that change frequently.
- **Stripe and Slack APIs:** Both use cursor-based pagination for list endpoints.
- **Data synchronisation:** ETL pipelines that need stable traversal of large tables.

---

## Core Concept 3: Pagination Metadata

### Definitions

**Core Definition:** Pagination metadata is the set of fields included in a paginated response that tells the client how to navigate to other pages — including next/previous links, total counts, page size, and cursor tokens.

**Technical Definition:** Pagination metadata can be returned in two ways: inside the JSON response body (as a `meta` or `pagination` object) or via HTTP `Link` headers per RFC 8288. The RFC 8288 Web Linking specification defines how to serialise pagination links in HTTP headers using `rel="next"`, `rel="prev"`, `rel="first"`, and `rel="last"`. Links included in the response payload SHOULD also be included as Link headers in the HTTP response according to RFC 8288.

**Beginner-Friendly Explanation:** Pagination metadata is like the "Page 3 of 20" indicator at the bottom of a book page, plus a "next page" arrow. It tells you where you are, how many pages there are, and how to get to the next one — either inside the response body (JSON) or in the HTTP headers (Link header).

### Purposes

- To inform clients about the total size of a collection and their current position.
- To provide navigation links (next, previous, first, last) for discoverability.
- To enable clients to render pagination UI controls.
- To standardise pagination across all endpoints of an API.
- To support both offset and cursor pagination with a consistent metadata structure.

### Sub-Feature 3.1: JSON Envelope Metadata

#### Syntax Rules and Structure

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 145,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true,
    "next": "/api/users?page=3&limit=20",
    "prev": "/api/users?page=1&limit=20"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `page` | Integer | Current page (offset pagination). |
| `limit` | Integer | Items per page. |
| `total` | Integer | Total matching records. |
| `totalPages` | Integer | Total pages. |
| `hasNext` | Boolean | Whether a next page exists. |
| `hasPrev` | Boolean | Whether a previous page exists. |
| `next` | String | URL for the next page. |
| `prev` | String | URL for the previous page. |

#### Rules

- Include pagination metadata in **every** paginated response.
- Use a consistent envelope structure across all endpoints for predictable client deserialisation.
- For cursor pagination, replace `page`/`total` with `nextCursor` and `hasNext`.
- A well-designed pagination response should be an object that "wraps" the data and includes metadata — not a bare array.

---

### Sub-Feature 3.2: Link Header (RFC 8288)

#### Syntax Rules and Structure

```
Link: <https://api.example.com/users?page=3>; rel="next",
      <https://api.example.com/users?page=1>; rel="prev",
      <https://api.example.com/users?page=1>; rel="first",
      <https://api.example.com/users?page=8>; rel="last"
```

| Component | Breakdown |
|-----------|-----------|
| `<URL>` | The target URL for the link. |
| `rel="next"` | Relationship type. |
| `rel="prev"` | Previous page. |
| `rel="first"` | First page. |
| `rel="last"` | Last page. |

#### Rules

- Links included in the response payload SHOULD also be included as Link headers per RFC 8288.
- The `Link` header keeps the response body clean — the body stays as `{ items }`.
- Clients should not assume that paging is safe against changes to the dataset while iterating through next links.
- Pagination links should include `first`, `next`, and `previous` URLs.

### Annotated Code Example

```js
// pagination-metadata.js
const express = require('express');
const app = express();

const items = Array.from({ length: 100 }, (_, i) => ({ id: i + 1 }));

app.get('/api/items', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = Math.min(parseInt(req.query.limit) || 10, 100);
  const offset = (page - 1) * limit;
  const total = items.length;
  const totalPages = Math.ceil(total / limit);
  const paginated = items.slice(offset, offset + limit);

  // Build pagination links
  const baseUrl = `${req.protocol}://${req.get('host')}${req.path}`;
  const links = [];
  if (page < totalPages) {
    links.push(`<${baseUrl}?page=${page + 1}&limit=${limit}>; rel="next"`);
  }
  if (page > 1) {
    links.push(`<${baseUrl}?page=${page - 1}&limit=${limit}>; rel="prev"`);
  }
  links.push(`<${baseUrl}?page=1&limit=${limit}>; rel="first"`);
  links.push(`<${baseUrl}?page=${totalPages}&limit=${limit}>; rel="last"`);

  res.set('Link', links.join(', '));
  res.json({
    data: paginated,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/items?page=2&limit=10`):**
```
HTTP/1.1 200 OK
Link: <http://localhost:3000/api/items?page=3&limit=10>; rel="next",
      <http://localhost:3000/api/items?page=1&limit=10>; rel="prev",
      <http://localhost:3000/api/items?page=1&limit=10>; rel="first",
      <http://localhost:3000/api/items?page=10&limit=10>; rel="last"
Content-Type: application/json

{
  "data": [...],
  "pagination": { "page": 2, "limit": 10, "total": 100, "totalPages": 10, ... }
}
```

**Why this output:** The handler builds the `Link` header with URLs for next, previous, first, and last pages. The JSON body also includes the pagination metadata. Clients can use either the Link header or the JSON metadata to navigate.

### Real-World Cases

- **GitHub API:** Uses `Link` headers with `rel="next"`, `rel="last"` for paginated resources.
- **Shopify API:** Uses RFC 8288 pagination format with `page_info` cursors.
- **Stripe API:** Returns `has_more` and `starting_after` in the JSON body.
- **Slack API:** Uses `response_metadata.next_cursor` in the JSON body.

---

## Core Concept 4: Sorting & Filtering Integration

### Definitions

**Core Definition:** Pagination must be combined with a deterministic sort order and filter conditions to ensure that records are neither duplicated nor skipped across pages.

**Technical Definition:** When paginating, the database query must include an `ORDER BY` clause with a **unique** or **composite unique** sort key. Without a stable sort order, the database may return rows in arbitrary order, causing duplicates or missing records. Keyset pagination requires the sort criteria to uniquely identify each entity; if the sort column has duplicates, a second stable column (e.g., `id`) must be included as a tiebreaker. Filtering conditions (`WHERE` clauses) must be applied consistently across all page requests; changing filters mid-traversal invalidates the cursor.

**Beginner-Friendly Explanation:** Imagine sorting a deck of cards by suit, but many cards have the same suit. If you don't also sort by rank as a tiebreaker, the order of cards within the same suit is arbitrary — you might see the same card twice or miss one entirely. Pagination works the same way: you need a unique sort key (or a combination of keys) to guarantee stable, consistent results.

### Purposes

- To prevent duplicate records from appearing on consecutive pages.
- To prevent records from being skipped entirely.
- To ensure that pagination is deterministic and reproducible.
- To enable efficient index usage for both sorting and filtering.

### Sub-Feature 4.1: Page Drift and Data Consistency

#### Problem: Page Drift with Offset Pagination

When rows are inserted or deleted between requests, the offset shifts. If 3 new orders arrive between page 1 and page 2 requests, rows 23–25 from page 1 are pushed down into positions 26–28, and the user sees them again on page 2. Deletion flips it: removing 3 rows from page 1 causes `OFFSET 25` to skip 3 rows the user never saw — silent data loss.

#### Solution: Keyset Pagination

Keyset pagination avoids this because it queries relative to a last-known position, not a numerical offset. The shift backward of numerical position of existing entities will not cause missed entities because keyset pagination does not query based on a numerical position. However, if you modify entity properties which are used as the sort criteria, keyset pagination cannot prevent the same entity from appearing again or never appearing due to the altered values.

### Sub-Feature 4.2: Composite Sort Keys

#### Syntax Rules and Structure

```sql
-- ❌ WRONG: Non-unique sort key (duplicates possible)
SELECT * FROM users ORDER BY created_at DESC LIMIT 20;

-- ✅ CORRECT: Composite sort key with tiebreaker
SELECT * FROM users ORDER BY created_at DESC, id DESC LIMIT 20;
```

| Component | Breakdown |
|-----------|-----------|
| `created_at DESC` | Primary sort column. |
| `id DESC` | Tiebreaker column — ensures uniqueness. |

#### Rules

- The sort criteria for keyset pagination **must uniquely identify each entity**.
- If the sort column has duplicates, include a second stable field (e.g., `id`, `uuid`) in the sort and cursor strategy to avoid duplicates or gaps.
- The index must match the composite sort key for optimal performance.
- When filtering, the filter conditions must be applied consistently across all page requests.

### Annotated Code Example

```js
// sorting-pagination.js
const express = require('express');
const app = express();

// Simulated data with duplicate sort values
const posts = [
  { id: 1, title: 'Post A', createdAt: '2026-01-15' },
  { id: 2, title: 'Post B', createdAt: '2026-01-15' },  // Same date
  { id: 3, title: 'Post C', createdAt: '2026-01-14' },
  { id: 4, title: 'Post D', createdAt: '2026-01-14' },  // Same date
  { id: 5, title: 'Post E', createdAt: '2026-01-13' }
];

app.get('/api/posts', (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 3, 10);
  const cursor = req.query.cursor
    ? JSON.parse(Buffer.from(req.query.cursor, 'base64').toString())
    : null;

  // Sort by createdAt DESC, then id DESC (composite key)
  let sorted = [...posts].sort((a, b) => {
    if (a.createdAt !== b.createdAt) {
      return a.createdAt > b.createdAt ? -1 : 1;
    }
    return b.id - a.id; // Tiebreaker: id DESC
  });

  // Apply keyset filter: start after the cursor position
  if (cursor) {
    sorted = sorted.filter(p => {
      if (p.createdAt < cursor.createdAt) return true;
      if (p.createdAt === cursor.createdAt && p.id < cursor.id) return true;
      return false;
    });
  }

  const paginated = sorted.slice(0, limit);
  const hasNext = sorted.length > limit;
  const last = paginated[paginated.length - 1];
  const nextCursor = hasNext
    ? Buffer.from(JSON.stringify({ createdAt: last.createdAt, id: last.id })).toString('base64')
    : null;

  res.json({
    data: paginated,
    pagination: { nextCursor, hasNext, limit }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/posts?limit=3`):**
```json
{
  "data": [
    { "id": 2, "title": "Post B", "createdAt": "2026-01-15" },
    { "id": 1, "title": "Post A", "createdAt": "2026-01-15" },
    { "id": 4, "title": "Post D", "createdAt": "2026-01-14" }
  ],
  "pagination": {
    "nextCursor": "eyJjcmVhdGVkQXQiOiIyMDI2LTAxLTE0IiwiaWQiOjR9",
    "hasNext": true,
    "limit": 3
  }
}
```

**Expected Output (for the next page with the cursor):**
```json
{
  "data": [
    { "id": 3, "title": "Post C", "createdAt": "2026-01-14" },
    { "id": 5, "title": "Post E", "createdAt": "2026-01-13" }
  ],
  "pagination": {
    "nextCursor": null,
    "hasNext": false,
    "limit": 3
  }
}
```

**Why this output:** The sort uses `createdAt DESC, id DESC` as a composite key. For posts with the same `createdAt`, the `id` tiebreaker determines order. The cursor encodes both `createdAt` and `id`, so the next page correctly starts after `{ createdAt: '2026-01-14', id: 4 }`. Without the tiebreaker, Post C and Post D (both dated 2026-01-14) could appear in either order, causing duplicates or missing records.

### Real-World Cases

- **Time-series data:** Sensor readings, logs, or events sorted by timestamp with a tiebreaker ID.
- **E-commerce orders:** Orders sorted by creation date, with order ID as tiebreaker.
- **Social media feeds:** Posts sorted by engagement score, with post ID as tiebreaker.
- **Data exports:** Full-table extracts using keyset pagination to avoid missed rows.

---

## References

- Apidog — Cursor-Based Pagination vs Offset Pagination — https://apidog.com/blog/cursor-vs-offset-pagination/
- Apidog — How Should You Design API Pagination for Millions of Records? — https://apidog.com/blog/how-to-design-api-pagination-millions-of-records/
- Apidog — REST API Pagination: An In-Depth Guide — https://apidog.com/blog/rest-api-pagination/
- OneUptime — How to Implement API Pagination Strategies — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-api-pagination-strategies/README.md
- StackSync — Postgres Pagination: Keyset vs Offset vs Cursor — https://www.stacksync.com/blog/keyset-cursors-postgres-pagination-fast-accurate-scalable
- Cybrosys — Why Large OFFSET Queries Are Slow in PostgreSQL — https://www.cybrosys.com/research-and-development/postgres/why-large-offset-queries-are-slow-in-postgresql-and-what-to-use-instead
- CoreUI — How to implement pagination in Node.js — https://coreui.io/answers/how-to-implement-pagination-in-nodejs/
- tsecurity — Understanding Offset and Cursor-Based Pagination in Node.js — https://tsecurity.de/de/2155189/IT+Programmierung/Understanding+Offset+and+Cursor-Based+Pagination+in+Node.js/
- Bump.sh — Pagination — https://docs.bump.sh/openapi/v3.2/advanced/pagination/
- RFC 8288 — Web Linking — https://datatracker.ietf.org/doc/html/rfc8288
- Supabase — Cursor Pagination Best Practices — https://github.com/aiskillstore/marketplace/blob/f93e9bb0daca99badb6a7e574b97737155d57cb3/skills/supabase/supabase-postgres-best-practices/rules/data-pagination.md
- GitHub — OFFSET pagination degrades at high page numbers — https://github.com/pymc-labs/decision-hub/issues/56
- Tinybird — Paginate Endpoint results with a cursor — https://tinybird-docs.vercel.app
- Corva — Filtering, Sorting, and Pagination — https://dc-docs.corva.ai
- SAP — Query rerun for each page — https://help.sap.com