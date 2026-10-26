# API Pagination — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API pagination** is the technique of dividing a large result set into **bounded, sequential slices** (pages) that a client retrieves incrementally. Each page is identified by a **position indicator** — an offset, a page number, or an opaque cursor — and the server returns only a subset of the total collection per request.

**Technical Definition.** Pagination constrains a `GET` collection endpoint to return a **limited number of items** per response, along with **metadata** that allows the client to request the next slice. The three principal strategies are **offset pagination** (skip `offset` rows, return `limit` rows), **cursor pagination** (return an opaque token pointing to the last-seen position), and **keyset pagination** (filter on the last-seen sort key values). The choice of strategy determines **performance at depth** (offset degrades linearly, keyset remains constant), **stability under concurrent writes** (offset drifts, keyset is stable), and **client capabilities** (offset supports random page access, cursor/keyset are forward-only).

**Beginner-Friendly Explanation.** Pagination is **breaking a long list into pages**. Instead of asking for all 10,000 products at once, you ask for the first 20, then the next 20, and so on. The API returns a small chunk each time plus a **bookmark** telling you how to get the next chunk. Without pagination, a single “get all” request could overwhelm your server, your database, and the client. Pagination is **mandatory for any list endpoint** that can return more than a few dozen items.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Bounded responses** | Each page returns a limited number of items (`limit`, `per_page`, or `pageSize`), preventing unbounded payloads. |
| **Position-aware** | The response includes a position indicator — `next` offset, `next_cursor`, or `nextKey` — that the client sends to fetch the following page. |
| **Strategy-dependent performance** | Offset pagination degrades linearly with depth; cursor and keyset pagination maintain constant performance. |
| **Strategy-dependent stability** | Offset pagination drifts when rows are inserted or deleted; cursor and keyset pagination remain stable. |
| **Metadata-rich** | Responses typically include pagination metadata: total count (optional), page size, next/previous links, and `has_more` flags. |
| **Mandatory for collections** | Any endpoint that returns a list should be paginated; unbounded collections are a scalability and availability risk. |

---

### Prerequisites

- **HTTP fundamentals**: query parameters, status codes, and the request/response model.
- **REST fundamentals**: resources, collection endpoints, and representations.
- **SQL fundamentals**: `LIMIT`, `OFFSET`, `ORDER BY`, and `WHERE` clauses (for understanding the database behavior behind each strategy).
- **Basic backend development**: routing and database queries in a language such as Node.js, Python, or Go.
- **A tool to test APIs**: `curl`, Postman, or a browser.

---

### Related Programming Areas

- **API design**: pagination is a mandatory part of the contract for collection endpoints.
- **Database performance**: the choice of pagination strategy directly affects query cost and index usage.
- **Client application design**: infinite scroll, “load more” buttons, and page-number navigation each require different pagination capabilities.
- **API caching**: paginated responses have distinct cache keys per page and must be cached with care.
- **API documentation**: pagination parameters and metadata must be documented for every list endpoint.

---

### Core Concepts / Features

The following five sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Offset Pagination

### Definitions

**Core Definition.** **Offset pagination** (also called **page-based pagination**) returns a slice of a result set by **skipping a specified number of rows** (`offset`) and returning the next `limit` rows. The client specifies either `page` and `pageSize` or `offset` and `limit`.

**Technical Definition.** Offset pagination is implemented in SQL as `SELECT ... ORDER BY ... LIMIT {limit} OFFSET {offset}`. The database **scans and discards** the first `offset` rows before returning the page, which means the cost grows **linearly with offset depth** — `OFFSET 1000000` is pathologically slow even with an index. Because the offset is a **row count**, not a stable position, concurrent inserts or deletes shift the underlying rows and cause **skipped or duplicated items** between pages. Offset pagination is simple, intuitive, and supports **random access** to any page, making it suitable for small, stable datasets and traditional page-number interfaces.

**Beginner-Friendly Explanation.** Offset pagination is **“skip N, take M.”** If you want page 3 with 20 items per page, you skip 40 and take 20. It is the easiest pagination to understand, and it lets users **jump directly to any page number**. The downside is that it gets slower as you go deeper (the database still has to count through all the skipped rows), and if someone adds or deletes a record while you are paging, you might see an item twice or miss one entirely.

### Purposes

- **To provide simple, intuitive pagination** with `page` and `pageSize` parameters that map directly to UI page numbers.
- **To enable random access** — clients can jump to page 47 without fetching pages 1–46.
- **To support total-count displays** (`Page 3 of 12`) by combining offset pagination with a `COUNT(*)` query.
- **To serve small, stable datasets** where deep-offset performance and write-drift are not concerns.

### Syntax Rules and Structure

**General syntax (query parameters).**

```
GET /v1/{resource}?page=2&pageSize=20
GET /v1/{resource}?limit=20&offset=20
```

**Component breakdown.**

- `page` — the **1-based page number** (page 1 is the first page).
- `pageSize` / `limit` — the **number of items per page**.
- `offset` — the **number of rows to skip**; `offset = (page - 1) * pageSize`.

**SQL pattern.**

```sql
SELECT id, name, email, created_at
FROM users
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40;
```

**Offset pagination rules.**

| Rule | Guidance |
|---|---|
| **Always specify a deterministic `ORDER BY`** | Without a stable sort, page boundaries are undefined. |
| **Enforce a maximum `limit`** | Cap `pageSize` at a server-defined maximum (e.g., 100) to prevent abuse. |
| **Use 1-based page numbers** | `page=1` is the first page; document the convention. |
| **Include total count in metadata** | Clients showing “Page X of Y” need `total` and `totalPages`. |

**Constraints and limitations.**

- **Performance degrades linearly with offset depth**; deep pages are slow because the database scans and discards skipped rows.
- **Results drift under concurrent writes**; inserts and deletes shift row positions, causing duplicates or skips.
- **`COUNT(*)` is expensive on large tables**; computing `total` adds a second query to every request.

### Annotated Complete Step by Step Code Examples

#### Example 1: Offset pagination in Express with metadata

```javascript
// app.js
app.get('/api/users', async (req, res) => {
  // Step 1: Parse and clamp pagination parameters.
  const page = Math.max(parseInt(req.query.page, 10) || 1, 1);
  const pageSize = Math.min(
    Math.max(parseInt(req.query.pageSize, 10) || 20, 1),
    100 // maximum page size
  );

  // Step 2: Compute the offset.
  const offset = (page - 1) * pageSize;

  // Step 3: Query the page of data.
  const users = await db.query(
    'SELECT id, name, email FROM users ORDER BY id LIMIT $1 OFFSET $2',
    [pageSize, offset]
  );

  // Step 4: Query the total count for metadata.
  const countResult = await db.query('SELECT COUNT(*) FROM users');
  const totalItems = parseInt(countResult.rows[0].count, 10);
  const totalPages = Math.ceil(totalItems / pageSize);

  // Step 5: Return the data with pagination metadata.
  res.json({
    data: users.rows,
    pagination: {
      page,
      pageSize,
      totalItems,
      totalPages,
      hasNextPage: page < totalPages,
      hasPrevPage: page > 1,
    },
  });
});
```

**Expected Output (Page 2, pageSize 20):**
```json
{
  "data": [
    { "id": 21, "name": "User 21", "email": "user21@example.com" },
    { "id": 22, "name": "User 22", "email": "user22@example.com" }
  ],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": true
  }
}
```

**Why this output occurs.** The handler clamps `page` and `pageSize` to safe bounds, computes `offset = (2 - 1) * 20 = 20`, and queries `LIMIT 20 OFFSET 20`. The metadata tells the client there are 8 total pages and that both next and previous pages exist. The `COUNT(*)` query adds overhead but enables page-number navigation.

---

#### Example 2: Offset pagination with a compound sort

```sql
-- Page 3 (20 per page) ordered by creation date and ID.
SELECT id, title, created_at
FROM articles
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40;
```

**Expected Output (conceptual):**
```
Returns rows 41–60 of the ordered result set.
The compound sort (created_at DESC, id DESC) ensures a deterministic order.
```

**Why this output occurs.** The compound `ORDER BY` provides a stable sort even when multiple articles share the same `created_at` value. Without the `id` tiebreaker, page boundaries could be ambiguous. However, the `OFFSET 40` still causes the database to scan and discard 40 rows, and concurrent inserts would still shift the result set.

---

### Real-World Cases

- **Admin dashboards**: small datasets with page-number navigation and total counts.
- **Search results**: users expect “Page 1 of 12” and the ability to jump to a specific page.
- **Legacy APIs**: offset pagination is the most widely supported and understood pattern.

---

## 2. Cursor Pagination

### Definitions

**Core Definition.** **Cursor pagination** uses an **opaque token** (the cursor) that points to a specific position in the result set. The client sends the cursor from the previous response to request the next page, and the server returns items **after** that position.

**Technical Definition.** The cursor typically encodes the **last-seen sort key value** (e.g., `id` or `created_at`) as a base64url-encoded string. The server decodes the cursor, applies a `WHERE` clause that filters on the sort key, and returns the next `limit` items. Because the query uses an **index seek** on the sort key rather than counting and discarding rows, cursor pagination maintains **constant performance** regardless of page depth. It is **stable under concurrent writes** because the cursor represents a position in the sorted index, not a row count. The trade-off is **forward-only traversal**: clients cannot jump to an arbitrary page.

**Beginner-Friendly Explanation.** Cursor pagination is **“give me the next batch after this bookmark.”** The server does not count rows; it looks at the bookmark and finds the next items in the sorted order. This makes it **fast even for page 10,000** and **stable when data changes** — if someone inserts a new item, it does not shift the items you have already seen. The catch is that you cannot jump to page 47 directly; you can only go forward (and sometimes backward, but not to arbitrary pages).

### Purposes

- **To maintain constant performance** regardless of pagination depth, even for datasets with millions of rows.
- **To provide stable results under concurrent inserts and deletes**, eliminating duplicates and skipped items.
- **To support infinite scroll and “load more”** interfaces where forward-only traversal is sufficient.
- **To avoid the cost of `COUNT(*)`** by not requiring total counts (though counts can be added optionally).

### Syntax Rules and Structure

**General syntax (query parameters).**

```
GET /v1/{resource}?limit=20
GET /v1/{resource}?limit=20&cursor=eyJpZCI6MTIzfQ==
```

**Component breakdown.**

- `limit` — the number of items to return per page.
- `cursor` — an **opaque token** encoding the last-seen position; omitted for the first page.
- The server returns a `next_cursor` (or `nextCursor`) in the response metadata.

**Cursor encoding pattern (Node.js).**

```javascript
function encodeCursor(obj) {
  return Buffer.from(JSON.stringify(obj)).toString('base64url');
}

function decodeCursor(cursor) {
  return JSON.parse(Buffer.from(cursor, 'base64url').toString());
}
```

**Cursor pagination rules.**

| Rule | Guidance |
|---|---|
| **Use an opaque cursor** | Encode the position as base64 JSON; never expose internal IDs or timestamps directly. |
| **Include a unique tiebreaker** | Use a compound sort (e.g., `created_at` + `id`) to handle ties. |
| **Validate the cursor** | Return `400 Bad Request` for malformed or truncated cursors. |
| **Fetch `limit + 1` to detect more pages** | If `limit + 1` items are returned, `has_more` is true; slice off the extra item. |

**Constraints and limitations.**

- **Forward-only traversal**; clients cannot jump to arbitrary pages.
- **No total count by default**; clients cannot display “Page X of Y” without an additional `COUNT(*)` query.
- **Cursor complexity**; encoding, decoding, and validation add implementation overhead.

### Annotated Complete Step by Step Code Examples

#### Example 1: Cursor pagination in Express

```javascript
// app.js
app.get('/api/users', async (req, res) => {
  // Step 1: Parse and clamp the limit.
  const limit = Math.min(
    Math.max(parseInt(req.query.limit, 10) || 20, 1),
    100
  );

  // Step 2: Decode the cursor, if provided.
  let cursor = null;
  if (req.query.cursor) {
    try {
      cursor = JSON.parse(
        Buffer.from(req.query.cursor, 'base64url').toString()
      );
    } catch {
      return res.status(400).json({ error: 'Invalid cursor' });
    }
  }

  // Step 3: Build the query.
  let query, params;
  if (cursor) {
    query = `
      SELECT id, name, email, created_at
      FROM users
      WHERE id > $1
      ORDER BY id
      LIMIT $2
    `;
    params = [cursor.id, limit + 1]; // fetch one extra
  } else {
    query = `
      SELECT id, name, email, created_at
      FROM users
      ORDER BY id
      LIMIT $1
    `;
    params = [limit + 1];
  }

  // Step 4: Execute and check for more results.
  const result = await db.query(query, params);
  const hasMore = result.rows.length > limit;
  const users = hasMore ? result.rows.slice(0, -1) : result.rows;

  // Step 5: Generate the next cursor from the last item.
  const nextCursor =
    hasMore && users.length > 0
      ? Buffer.from(
          JSON.stringify({ id: users[users.length - 1].id })
        ).toString('base64url')
      : null;

  // Step 6: Return the data and cursor metadata.
  res.json({
    data: users,
    pagination: {
      nextCursor,
      hasMore,
    },
  });
});
```

**Expected Output (first page):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", "email": "alice@example.com" },
    { "id": 2, "name": "Bob", "email": "bob@example.com" }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6Mn0=",
    "hasMore": true
  }
}
```

**Expected Output (second page, using `cursor=eyJpZCI6Mn0=`):**
```json
{
  "data": [
    { "id": 3, "name": "Carol", "email": "carol@example.com" },
    { "id": 4, "name": "Dave", "email": "dave@example.com" }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6NH0=",
    "hasMore": true
  }
}
```

**Why this output occurs.** The first page returns items ordered by `id` and encodes the last `id` (`2`) as `eyJpZCI6Mn0=`. The client sends that cursor in the next request, and the server queries `WHERE id > 2`, returning the next batch. Fetching `limit + 1` items allows the server to determine `hasMore` without a separate count query.

---

#### Example 2: Cursor pagination with a compound sort key

```javascript
// Step 1: Compound sort by created_at DESC, id DESC.
const cursor = { created_at: '2026-01-15T10:30:00Z', id: 12345 };

// Step 2: Query with the compound cursor.
const query = `
  SELECT id, title, created_at
  FROM articles
  WHERE (created_at, id) < ($1, $2)
  ORDER BY created_at DESC, id DESC
  LIMIT $3
`;
const params = [cursor.created_at, cursor.id, limit + 1];
```

**Expected Output (conceptual):**
```
Returns articles older than the cursor position, with id as a tiebreaker for identical timestamps.
```

**Why this output occurs.** The compound `WHERE` clause filters on both `created_at` and `id`, handling ties correctly. Without the tiebreaker, multiple articles with the same timestamp could be skipped or duplicated. This is the **keyset** form of cursor pagination, which is more precise than an opaque cursor that encodes only one value.

---

### Real-World Cases

- **Social media feeds**: new posts are inserted at the head; cursor pagination prevents duplicates and skips.
- **Mobile infinite scroll**: forward-only traversal is a natural fit for “load more” interfaces.
- **Log viewers**: cursor pagination handles dynamic data sets without the instability of offset pagination.

---

## 3. Keyset Pagination

### Definitions

**Core Definition.** **Keyset pagination** (also called **seek pagination**) is a form of cursor pagination that uses **actual column values** — not an opaque token — to filter the next page. The query uses a `WHERE` clause on the sort key (e.g., `WHERE id > last_seen_id`) to seek directly to the next position.

**Technical Definition.** Keyset pagination leverages database indexes by rewriting the pagination condition as a **predicate on the sort key**. Instead of `OFFSET N`, the query uses `WHERE (sort_column, tiebreaker) > (last_sort_value, last_tiebreaker)`. Because the database can perform an **index seek** to the position rather than scanning and discarding rows, keyset pagination maintains **O(1) performance** regardless of depth. It requires a **unique, indexed sort key** (or a compound key with a unique tiebreaker) and is **forward-only** (though reverse traversal is possible with `ORDER BY ... DESC` and a reversed predicate). The distinction between keyset and cursor pagination is subtle: **cursor pagination** typically uses an opaque, encoded token; **keyset pagination** uses raw column values. In practice, many systems combine them by encoding the keyset values into a cursor.

**Beginner-Friendly Explanation.** Keyset pagination is **“start after this exact value.”** Instead of skipping 40 rows, you say “give me items with an ID greater than 12345.” Because the database can jump straight to that position using an index, it is **fast no matter how deep you go**. The catch is that you need a **unique, indexed column** to sort by (like an ID or a timestamp with an ID tiebreaker), and you can only move forward through the list.

### Purposes

- **To achieve optimal database performance** by using index seeks instead of offset scans.
- **To maintain constant query cost** regardless of page depth, even on tables with millions of rows.
- **To provide stable pagination** under concurrent writes by anchoring to a value rather than a row count.
- **To support chronological and time-series navigation** where the sort key is naturally ordered (e.g., `created_at`, `id`).

### Syntax Rules and Structure

**General syntax (SQL).**

```sql
-- First page
SELECT id, name, created_at
FROM users
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Next page (keyset: last seen created_at = '2026-01-15 10:30:00', id = 12345)
SELECT id, name, created_at
FROM users
WHERE (created_at, id) < ('2026-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Component breakdown.**

- `(created_at, id) < (...)` — the **keyset predicate**; filters to rows strictly after the last-seen position.
- `ORDER BY created_at DESC, id DESC` — the **sort order**; must match the keyset predicate.
- `LIMIT 20` — the page size.

**Keyset pagination rules.**

| Rule | Guidance |
|---|---|
| **Use an indexed, unique sort key** | The `WHERE` column must be indexed and unique (or have a unique tiebreaker). |
| **Match the predicate to the sort order** | For `DESC` order, use `<`; for `ASC` order, use `>`. |
| **Include a tiebreaker** | Combine the primary sort field with a unique field (e.g., `created_at, id`) to handle ties. |
| **Encode keyset values in the response** | Return `nextKey` or `next_cursor` containing the last row’s sort values. |

**Constraints and limitations.**

- **Requires an indexed, unique sort key**; without one, performance degrades to a full scan.
- **Forward-only traversal**; random page access is not supported.
- **More complex than offset**; the client must carry the keyset values, and the server must validate them.
- **Compound keyset predicates can be complex** to construct correctly for multi-column sorts.

### Annotated Complete Step by Step Code Examples

#### Example 1: Keyset pagination in Node.js with Express

```javascript
// app.js
app.get('/api/users', async (req, res) => {
  // Step 1: Parse the limit.
  const limit = Math.min(
    Math.max(parseInt(req.query.limit, 10) || 20, 1),
    100
  );

  // Step 2: Parse the keyset parameters (last seen values).
  const afterId = req.query.after_id
    ? parseInt(req.query.after_id, 10)
    : null;

  // Step 3: Build the query with the keyset predicate.
  let query, params;
  if (afterId !== null) {
    query = `
      SELECT id, name, email
      FROM users
      WHERE id > $1
      ORDER BY id
      LIMIT $2
    `;
    params = [afterId, limit + 1];
  } else {
    query = `
      SELECT id, name, email
      FROM users
      ORDER BY id
      LIMIT $1
    `;
    params = [limit + 1];
  }

  // Step 4: Execute and determine if more pages exist.
  const result = await db.query(query, params);
  const hasMore = result.rows.length > limit;
  const users = hasMore ? result.rows.slice(0, -1) : result.rows;

  // Step 5: Return the next keyset value.
  const nextKey = hasMore && users.length > 0
    ? { after_id: users[users.length - 1].id }
    : null;

  res.json({
    data: users,
    pagination: {
      nextKey,
      hasMore,
    },
  });
});
```

**Expected Output (first page):**
```json
{
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "pagination": {
    "nextKey": { "after_id": 2 },
    "hasMore": true
  }
}
```

**Expected Output (next page, using `?after_id=2`):**
```json
{
  "data": [
    { "id": 3, "name": "Carol" },
    { "id": 4, "name": "Dave" }
  ],
  "pagination": {
    "nextKey": { "after_id": 4 },
    "hasMore": true
  }
}
```

**Why this output occurs.** The keyset predicate `WHERE id > $1` uses the indexed `id` column to seek directly to the next position. The database does not scan and discard rows; it performs an index seek, maintaining constant performance regardless of how many pages have been fetched.

---

#### Example 2: Keyset pagination with a compound sort key (Python)

```python
# Step 1: Parse keyset parameters.
after_date = request.args.get('after_date')
after_id = request.args.get('after_id')

# Step 2: Build the query with a compound keyset predicate.
query = Article.query.order_by(
    Article.published_at.desc(),
    Article.id.desc()
)

if after_date and after_id:
    try:
        after_date = datetime.fromisoformat(after_date)
        after_id = int(after_id)
    except ValueError:
        return jsonify({'error': 'Invalid keyset parameters'}), 400

    query = query.filter(
        or_(
            Article.published_at < after_date,
            and_(
                Article.published_at == after_date,
                Article.id < after_id
            )
        )
    )

# Step 3: Fetch limit + 1 to check for more.
articles = query.limit(limit + 1).all()
has_more = len(articles) > limit
if has_more:
    articles = articles[:limit]

# Step 4: Build the next keyset from the last article.
result = {
    'data': [a.to_dict() for a in articles],
    'pagination': {'hasMore': has_more}
}
if has_more and articles:
    last = articles[-1]
    result['pagination']['nextKey'] = {
        'after_date': last.published_at.isoformat(),
        'after_id': last.id
    }
return jsonify(result)
```

**Expected Output (conceptual):**
```
Returns articles strictly older than the last-seen (published_at, id) pair.
The compound predicate handles ties on published_at by using id as a tiebreaker.
```

**Why this output occurs.** The `or_` / `and_` structure implements the compound keyset predicate: `(published_at < after_date) OR (published_at = after_date AND id < after_id)`. This is equivalent to the SQL row-value comparison `(published_at, id) < (after_date, after_id)` and handles ties correctly. The query uses the index on `(published_at, id)` for an efficient seek.

---

### Real-World Cases

- **Log viewers and audit trails**: chronological, immutable data where keyset pagination on a timestamp is natural.
- **Time-series data**: sensor readings and metrics ordered by timestamp with an ID tiebreaker.
- **Large exports and backfills**: keyset pagination avoids the deep-offset performance cliff for full-table traversal.

---

## 4. Pagination Metadata

### Definitions

**Core Definition.** **Pagination metadata** is the set of fields in a response that describe **what page was returned**, **how to get the next page**, and **how large the total result set is** (when available). It is typically wrapped in a **pagination envelope** alongside the data array.

**Technical Definition.** Pagination metadata varies by strategy. **Offset pagination** metadata includes `page`, `pageSize` (or `limit`), `totalItems`, `totalPages`, `hasNextPage`, and `hasPrevPage`. **Cursor and keyset pagination** metadata includes `nextCursor` (or `nextKey`), `hasMore`, and optionally `totalCount` if a count query is performed. The response is almost always an **envelope object** — `{ data: [...], pagination: {...} }` — rather than a bare array, so clients can distinguish an empty page from a missing page and can always find the next-page information.

**Beginner-Friendly Explanation.** Pagination metadata is **the instructions on the box**. It tells you how many items you got, how many exist in total, and what to send to get the next batch. Without metadata, you would not know whether there is a next page or how to ask for it. A good response looks like `{ "data": [...], "pagination": { "nextCursor": "...", "hasMore": true } }` — the data is the payload, and the pagination object is the map for navigating the rest.

### Purposes

- **To tell the client how to fetch the next page** via `nextCursor`, `next` offset, or `nextKey`.
- **To indicate whether more results exist** through `hasMore` or `hasNextPage`.
- **To support total-count displays** when the API provides `totalItems` and `totalPages`.
- **To provide a consistent envelope** so clients can parse pagination information uniformly across endpoints.

### Syntax Rules and Structure

**General syntax (offset envelope).**

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": true
  }
}
```

**General syntax (cursor envelope).**

```json
{
  "data": [ ... ],
  "pagination": {
    "nextCursor": "eyJpZCI6MjB9",
    "hasMore": true
  }
}
```

**Component breakdown.**

- `data` — the array of items for the current page.
- `pagination` — the metadata object.
- `nextCursor` / `next` / `nextKey` — the position indicator for the next request.
- `hasMore` / `hasNextPage` — whether another page exists.
- `totalItems` / `totalPages` — optional totals (offset only, or cursor with an explicit count query).

**Metadata rules.**

| Rule | Guidance |
|---|---|
| **Always use an envelope, never a bare array** | A bare array cannot carry pagination instructions. |
| **Include `hasMore` or equivalent** | Clients need a boolean to know when to stop. |
| **Include the next-page indicator** | `nextCursor`, `next` offset, or `nextKey` — even if it is `null` on the last page. |
| **Make totals optional** | `totalItems` requires a `COUNT(*)`; omit it when the count is too expensive. |

**Constraints and limitations.**

- **Cursor pagination cannot provide `totalItems` without a separate count query**; many APIs omit totals for cursor-based lists.
- **Offset pagination totals are expensive on large tables**; `COUNT(*)` adds a second query per request.
- **Inconsistent metadata shapes across endpoints** force clients to write per-endpoint parsers.

### Annotated Complete Step by Step Code Examples

#### Example 1: Offset pagination envelope with full metadata

```javascript
// Step 1: Query the data and the total count.
const users = await db.query(
  'SELECT id, name FROM users ORDER BY id LIMIT $1 OFFSET $2',
  [limit, offset]
);
const countResult = await db.query('SELECT COUNT(*) FROM users');
const total = parseInt(countResult.rows[0].count, 10);

// Step 2: Build the envelope.
res.json({
  data: users.rows,
  pagination: {
    page,
    pageSize: limit,
    totalItems: total,
    totalPages: Math.ceil(total / limit),
    hasNextPage: page < Math.ceil(total / limit),
    hasPrevPage: page > 1,
  },
});
```

**Expected Output:**
```json
{
  "data": [
    { "id": 21, "name": "User 21" },
    { "id": 22, "name": "User 22" }
  ],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": true
  }
}
```

**Why this output occurs.** The envelope separates the payload from the metadata. The client can render “Page 2 of 8” using `page` and `totalPages`, and can enable or disable next/previous buttons using `hasNextPage` and `hasPrevPage`.

---

#### Example 2: Cursor pagination envelope with `nextCursor` and `hasMore`

```javascript
// Step 1: Fetch limit + 1 items to detect more pages.
const result = await db.query(
  `SELECT id, name FROM users
   WHERE id > $1
   ORDER BY id
   LIMIT $2`,
  [cursorId || 0, limit + 1]
);

const hasMore = result.rows.length > limit;
const items = hasMore ? result.rows.slice(0, -1) : result.rows;

const nextCursor = hasMore
  ? Buffer.from(JSON.stringify({ id: items[items.length - 1].id }))
      .toString('base64url')
  : null;

// Step 2: Return the envelope.
res.json({
  data: items,
  pagination: {
    nextCursor,
    hasMore,
  },
});
```

**Expected Output:**
```json
{
  "data": [
    { "id": 3, "name": "Carol" },
    { "id": 4, "name": "Dave" }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6NH0=",
    "hasMore": true
  }
}
```

**Why this output occurs.** The `hasMore` flag tells the client another page exists, and `nextCursor` provides the opaque token to request it. When the last page is reached, `hasMore` is `false` and `nextCursor` is `null`, signaling the client to stop.

---

### Real-World Cases

- **Plane API**: uses a consistent envelope with `data`, `next`, `previous`, `total_count`, and `pagination.style` for offset, and `data`, `next_cursor`, `has_more`, and `pagination.style` for cursor.
- **Stripe API**: returns `data` arrays with `has_more` and `url` for cursor-based pagination.
- **GitHub API**: returns `Link` headers with `rel="next"`, `rel="prev"`, `rel="first"`, and `rel="last"` for pagination navigation.

---

## 5. Pagination Best Practices

### Definitions

**Core Definition.** **Pagination best practices** are empirically established guidelines for designing paginated APIs that are **performant, stable, secure, and client-friendly** across different dataset sizes and access patterns.

**Technical Definition.** Best practices span **strategy selection** (offset for small/stable datasets and random access; cursor/keyset for large/dynamic datasets and sequential traversal), **parameter design** (consistent names, server-side clamping, maximum limits), **cursor security** (opaque encoding, validation, no exposed internal IDs), **metadata shape** (envelope with `hasMore` and next-page indicator), **performance** (indexed sort keys, avoiding `COUNT(*)` on hot paths), and **documentation** (strategy, parameters, limits, and metadata fields).

**Beginner-Friendly Explanation.** Best practices are **the habits that keep pagination from becoming a source of bugs and outages**. Use the right strategy for your data: offset for small, stable lists where users want page numbers; cursor or keyset for large, changing lists where speed and stability matter. Always enforce a maximum page size so one client cannot request a million records. Make cursors opaque so clients cannot guess other users’ data. Always return a next-page indicator. And document exactly how pagination works for every list endpoint.

### Purposes

- **To prevent performance cliffs** by choosing keyset or cursor pagination for large datasets.
- **To eliminate instability** by avoiding offset pagination on frequently updated data.
- **To protect the server** by enforcing maximum page sizes and validating cursors.
- **To provide a consistent client experience** through a uniform envelope and clear metadata.
- **To secure pagination** by using opaque cursors and avoiding the exposure of internal identifiers.

### Syntax Rules and Structure

**Strategy selection decision tree.**

```
Dataset size and update frequency?
  ├── Small, stable (< 10K rows) → Offset pagination (page numbers, random access)
  └── Large or dynamic → Cursor or keyset pagination
        ├── Need random page access? → Offset with caching (if acceptable)
        ├── Need total count? → Offset (or cursor with explicit count)
        └── Sequential traversal / infinite scroll → Cursor or keyset
```

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Strategy** | Offset for small/stable + random access; cursor/keyset for large/dynamic + sequential traversal. |
| **Limits** | Enforce a server-side maximum page size (e.g., 100); clamp values above the max. |
| **Cursor security** | Encode cursors opaquely (base64 JSON); never expose internal IDs or timestamps. |
| **Cursor validation** | Return `400 Bad Request` for malformed or truncated cursors. |
| **Sort key** | Use a unique, indexed sort key with a tiebreaker for cursor/keyset pagination. |
| **Metadata** | Always return `hasMore` (or `has_next`) and `nextCursor` / `next` / `nextKey`. |
| **Totals** | Omit `totalItems` on cursor pagination unless explicitly requested; avoid `COUNT(*)` on hot paths. |
| **Documentation** | Document the strategy, parameters, defaults, maximums, and metadata fields for every list endpoint. |

**Constraints and limitations.**

- **No single strategy is best for all cases**; the choice depends on dataset size, update frequency, and client needs.
- **Offset with caching can mitigate deep-offset performance** but does not fix write-drift instability.
- **Cursor pagination cannot support random access**; if the UI requires “jump to page 47,” offset is necessary.

### Annotated Complete Step by Step Code Examples

#### Example 1: Strategy selection in practice

```javascript
// Step 1: Small, stable dataset → offset pagination.
app.get('/api/categories', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const pageSize = Math.min(parseInt(req.query.pageSize) || 20, 100);
  const offset = (page - 1) * pageSize;

  const [rows, count] = await Promise.all([
    db.query('SELECT * FROM categories ORDER BY name LIMIT $1 OFFSET $2', [pageSize, offset]),
    db.query('SELECT COUNT(*) FROM categories'),
  ]);

  res.json({
    data: rows.rows,
    pagination: {
      page, pageSize,
      totalItems: parseInt(count.rows[0].count, 10),
      totalPages: Math.ceil(parseInt(count.rows[0].count, 10) / pageSize),
    },
  });
});

// Step 2: Large, dynamic dataset → cursor pagination.
app.get('/api/feed', async (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);
  const cursor = req.query.cursor
    ? JSON.parse(Buffer.from(req.query.cursor, 'base64url').toString())
    : null;

  const query = cursor
    ? `SELECT * FROM posts WHERE id < $1 ORDER BY id DESC LIMIT $2`
    : `SELECT * FROM posts ORDER BY id DESC LIMIT $1`;
  const params = cursor ? [cursor.id, limit + 1] : [limit + 1];

  const rows = await db.query(query, params);
  const hasMore = rows.rows.length > limit;
  const items = hasMore ? rows.rows.slice(0, -1) : rows.rows;
  const nextCursor = hasMore
    ? Buffer.from(JSON.stringify({ id: items[items.length - 1].id })).toString('base64url')
    : null;

  res.json({ data: items, pagination: { nextCursor, hasMore } });
});
```

**Expected Output (conceptual):**
```
/api/categories → offset pagination with page numbers and total counts.
/api/feed → cursor pagination with nextCursor and hasMore; no total count.
```

**Why this output occurs.** The categories endpoint uses offset pagination because the dataset is small and stable, and users may want page numbers. The feed endpoint uses cursor pagination because posts are inserted continuously and deep traversal must remain fast and stable.

---

#### Example 2: Enforcing maximum limits and validating cursors

```javascript
// Step 1: Clamp the page size to a server-side maximum.
const MAX_PAGE_SIZE = 100;
const pageSize = Math.min(
  Math.max(parseInt(req.query.pageSize, 10) || 20, 1),
  MAX_PAGE_SIZE
);

// Step 2: Validate the cursor.
if (req.query.cursor) {
  try {
    const decoded = JSON.parse(
      Buffer.from(req.query.cursor, 'base64url').toString()
    );
    if (typeof decoded.id !== 'number') {
      throw new Error('Invalid cursor shape');
    }
  } catch {
    return res.status(400).json({ error: 'Invalid cursor' });
  }
}
```

**Expected Output (conceptual):**
```
pageSize=500 → clamped to 100 (not rejected).
Malformed cursor → 400 Bad Request with an error message.
```

**Why this output occurs.** The server clamps oversized page sizes to prevent abuse and memory exhaustion. It validates the cursor’s structure before using it, returning a clear `400` error instead of crashing or serving incorrect data.

---

### Real-World Cases

- **GitHub API**: uses offset-style `page` and `per_page` parameters with a maximum of 100, and `Link` headers for navigation.
- **Stripe API**: uses cursor-based pagination with `starting_after` and `ending_before`, and a maximum `limit` of 100.
- **Plane API**: defaults to offset pagination with a hard maximum offset of 10,000 and offers opt-in cursor pagination for deep traversal.

---

## References

- How to Implement API Pagination Strategies — OneUptime Blog - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-api-pagination-strategies/README.md
- Pagination Patterns for REST APIs — llm-knowledge-base (GitHub) - https://github.com/glennguilloux/llm-knowledge-base/blob/main/api-design/pagination-patterns.md
- Plane Developer Documentation — Pagination - https://developers.plane.so/api-reference/v2/pagination
- Paginate Results — CockroachDB Docs - https://docs.cockroachlabs.com/docs/v25.4/pagination
- Difference between keyset and cursor paginations? — Stack Overflow - https://stackoverflow.com/questions/66771229/
- API Pagination Guide — DeployStack - https://docs.deploystack.io
- How Should You Design API Pagination for Millions of Records? — Apidog - https://apidog.com
- Why Contentful is moving to cursor-based pagination — Contentful Blog - https://www.contentful.com
- SDK pagination patterns — GitHub - https://github.com
- Pagination — Novu Docs - https://docs.novu.co
- Pagination — Vendasta Developer Portal - https://developers.vendasta.com
- Pagination — NINJIO API Documentation - https://docs.ninjio.com
- ClickHouse Pagination in TypeScript — Offset, Cursor, and Keyset — HypeQuery - https://hypequery.com