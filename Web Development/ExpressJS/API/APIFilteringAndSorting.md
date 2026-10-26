# API Filtering and Sorting — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API filtering and sorting** are the mechanisms by which a client narrows and orders a collection of resources returned by an API. **Filtering** applies predicates to resource attributes to exclude non-matching items; **sorting** establishes a deterministic order over the matching items. Together with **field selection** (sparse fieldsets) and **embedding** (compound documents), they form the **collection query interface** — the set of controls that transform a raw collection endpoint into a precise, efficient, and usable data-retrieval surface.

**Technical Definition.** In a REST API, filtering and sorting are expressed through **query parameters** on `GET` collection endpoints. Filtering predicates may be **equality-based** (`?status=active`) or **operator-based** (`?price[gte]=100`, using bracket notation such as Stripe’s `field[operator]=value`). Sorting is expressed as a **single `sort` parameter** with comma-separated field names and a **`-` prefix for descending order** (JSON:API convention), or as a vendor-specific `$orderby` (OData/DAB). **Field selection** uses a `fields` parameter (optionally keyed by resource type: `fields[articles]=title,body`) to return only requested attributes. **Embedding** uses an `include` parameter (`?include=author,comments`) to return related resources alongside the primary resource in a **compound document**, eliminating the need for follow-up requests. All four mechanisms must be **whitelisted, type-validated, and indexed** to be both safe and performant.

**Beginner-Friendly Explanation.** Filtering and sorting are **how you ask for exactly the data you want, in the order you want it**. Filtering is like telling a librarian “only show me books published after 2020.” Sorting is “arrange them by title, A to Z.” Field selection is “just show me the title and author, not the whole record.” Embedding is “and while you’re at it, pull the author’s biography too, so I don’t have to ask for it separately.” Without these controls, every client gets the entire collection in an arbitrary order — a recipe for slow responses, bloated payloads, and frustrated developers.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Query-parameter driven** | Filtering, sorting, field selection, and embedding are expressed as query parameters, keeping the resource path stable. |
| **Whitelist-based** | Filterable, sortable, and selectable fields must be explicitly allowed; unknown fields are rejected to prevent injection and information leakage. |
| **Type-validated** | Operators are validated against field types (e.g., `contains` is invalid on an integer field). |
| **Index-dependent** | Filtering and sorting rely on database indexes; unindexed fields cause full scans and degrade performance. |
| **Cache-aware** | Query parameters are part of the cache key; sorting and filtering reduce cacheability and must be documented accordingly. |
| **Composable** | Filtering, sorting, field selection, and embedding can be combined in a single request to produce a precisely tailored response. |

---

### Prerequisites

- **HTTP fundamentals**: query parameters, status codes, and the request/response model.
- **REST fundamentals**: collection endpoints and representations.
- **SQL fundamentals**: `WHERE`, `ORDER BY`, `LIMIT`, and indexed columns (for understanding the database behavior).
- **JSON:API specification basics** (optional but recommended for field selection and embedding).
- **A tool to test APIs**: `curl`, Postman, or a browser.

---

### Related Programming Areas

- **API pagination**: filtering and sorting determine **which** items are returned and **in what order**; pagination determines **how many** and **which slice**.
- **API design**: filtering and sorting are part of the collection-query contract.
- **Database performance**: whitelisting and indexing are essential for safe and fast filtering.
- **Security**: unfiltered query parameters are a vector for SQL injection and data exfiltration.
- **API documentation**: filterable fields, sortable fields, allowed operators, and default sort order must be documented.

---

### Core Concepts / Features

The following five sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Filtering

### Definitions

**Core Definition.** **Filtering** is the mechanism by which a client supplies predicates that **narrow a collection** to only the resources matching those predicates. The server evaluates the predicates against resource attributes and returns the subset that satisfies them.

**Technical Definition.** Filtering is implemented through **query parameters** on a `GET` collection endpoint. The simplest form uses **direct field-value pairs** (`?status=active&category=books`), which are combined with **AND semantics** by default. Advanced filtering uses **operator-based bracket notation** (`?price[gte]=100`, `?name[contains]=widget`), where the operator is validated against the field’s type. The server must **whitelist** filterable fields (rejecting unknown fields with `400 Bad Request`), **validate operator–field compatibility**, and ensure filterable columns are **indexed**. The de facto standard for operator syntax is `field[operator]=value`, used by Stripe, Shopify, and many other APIs.

**Beginner-Friendly Explanation.** Filtering is **narrowing a list by conditions**. You say “only show me orders with status `shipped` and a cost of at least 100.” The API translates those conditions into a database `WHERE` clause and returns only the matching rows. The safest and most common syntax is `field=value` for equality and `field[operator]=value` for comparisons. The API must decide **which fields you are allowed to filter on** — you should not be able to filter on a password hash, for example — and must reject filters it does not recognize.

### Purposes

- **To reduce the result set** to only records relevant to the client’s use case.
- **To enable user-driven narrowing** in search, dashboards, and list views.
- **To prevent data over-fetching** and reduce network transfer.
- **To support complex queries** (ranges, membership, substring matches) through typed operators.
- **To enforce access boundaries** by limiting which fields can be used as filters.

### Syntax Rules and Structure

**General syntax (equality filtering).**

```
GET /v1/{resource}?{field}={value}&{field2}={value2}
```

**General syntax (operator-based filtering).**

```
GET /v1/{resource}?{field}[{operator}]={value}
```

**Component breakdown.**

```
GET /v1/orders?status=shipped&minCost=100
GET /v1/products?price[gte]=100&name[contains]=widget
```

- `status=shipped` — **equality** filter on the `status` field.
- `minCost=100` — a **named convenience filter** for a minimum threshold.
- `price[gte]=100` — **operator-based** filter using **bracket notation** for “greater than or equal”.
- `name[contains]=widget` — **substring** filter; valid only because `name` is a string field.

**Common filter operators.**

| Operator | Meaning | Valid Field Types |
|---|---|---|
| `eq` / `=` | Equal | All |
| `ne` / `!=` | Not equal | All |
| `gt` / `>` | Greater than | Numbers, dates |
| `gte` / `>=` | Greater than or equal | Numbers, dates |
| `lt` / `<` | Less than | Numbers, dates |
| `lte` / `<=` | Less than or equal | Numbers, dates |
| `contains` | Substring match | Strings |
| `in` | Membership in a list | Strings, numbers |

**Filtering rules.**

| Rule | Guidance |
|---|---|
| **Whitelist filterable fields** | Reject unknown filters with `400 Bad Request`. |
| **Validate operator–type compatibility** | Reject `contains` on numeric fields at parse time. |
| **Use parameterized queries** | Never concatenate filter values into SQL strings. |
| **Index filterable columns** | Ensure common filters use indexes to avoid full scans. |
| **Document filter semantics** | Clarify whether `name=John` is exact or substring matching. |

**Constraints and limitations.**

- **Unindexed filters cause full table scans** and degrade performance.
- **Complex filter combinations can exceed query-parameter limits**; consider a `POST /search` endpoint with a request body for very large filters.
- **Filter parameter names are not standardized** across APIs; choose a convention and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: Equality filtering in Express

```javascript
// app.js
app.get('/v1/orders', async (req, res) => {
  // Step 1: Build a whitelist of filterable fields.
  const FILTERABLE = ['status', 'minCost', 'customerId'];

  // Step 2: Extract and validate filters.
  const filters = {};
  for (const [key, value] of Object.entries(req.query)) {
    if (FILTERABLE.includes(key)) {
      filters[key] = value;
    } else {
      return res.status(400).json({ error: `Unknown filter: ${key}` });
    }
  }

  // Step 3: Translate filters into a parameterized query.
  const conditions = [];
  const params = [];
  let idx = 1;

  if (filters.status) {
    conditions.push(`status = $${idx++}`);
    params.push(filters.status);
  }
  if (filters.minCost) {
    conditions.push(`cost >= $${idx++}`);
    params.push(Number(filters.minCost));
  }
  if (filters.customerId) {
    conditions.push(`customer_id = $${idx++}`);
    params.push(filters.customerId);
  }

  const where = conditions.length ? `WHERE ${conditions.join(' AND ')}` : '';
  const result = await db.query(
    `SELECT id, status, cost, customer_id FROM orders ${where} ORDER BY id`,
    params
  );

  res.json({ data: result.rows });
});
```

**Expected Output (conceptual):**
```
GET /v1/orders?status=shipped&minCost=100
→ Returns only orders with status "shipped" and cost >= 100.
GET /v1/orders?password=... → 400 Bad Request (unknown filter).
```

**Why this output occurs.** The handler whitelists filterable fields, rejects unknown ones, and builds a parameterized `WHERE` clause from the validated filters. This prevents SQL injection and ensures only approved fields can be used for filtering.

---

#### Example 2: Operator-based filtering with type validation

```javascript
// Step 1: Define allowed operators per field type.
const OPERATORS = {
  string: ['eq', 'ne', 'contains'],
  number: ['eq', 'ne', 'gt', 'gte', 'lt', 'lte'],
  date:   ['eq', 'gt', 'gte', 'lt', 'lte'],
};

// Step 2: Parse a filter like "price[gte]=100".
function parseFilter(param, schema) {
  const match = param.match(/^(\w+)\[(\w+)\]$/);
  if (!match) {
    // Simple equality filter.
    const [field, value] = param.split('=');
    return { field, operator: 'eq', value };
  }
  const [, field, operator] = match;
  const fieldType = schema[field];
  if (!fieldType || !OPERATORS[fieldType].includes(operator)) {
    throw new Error(`Unsupported operator "${operator}" for field "${field}"`);
  }
  return { field, operator, value: /* extracted value */ };
}

// Step 3: Build the SQL predicate.
function toSQL({ field, operator, value }) {
  const sqlOps = { eq: '=', ne: '!=', gt: '>', gte: '>=', lt: '<', lte: '<=', contains: 'ILIKE' };
  if (operator === 'contains') return { sql: `${field} ILIKE $1`, value: `%${value}%` };
  return { sql: `${field} ${sqlOps[operator]} $1`, value };
}
```

**Expected Output (conceptual):**
```
price[gte]=100 → valid (number supports gte)
price[contains]=100 → 400 Bad Request (contains is invalid on a number)
```

**Why this output occurs.** The parser checks the operator against the field’s type. `contains` is only valid for strings, so applying it to a numeric field is rejected at parse time rather than causing a database error or unexpected results.

---

### Real-World Cases

- **Azure API Design**: `/orders?minCost=100&status=shipped` — equality-based filtering on collection endpoints.
- **Stripe API**: uses bracket notation (`?created[gte]=...`, `?status=active`) for advanced filtering.
- **OData/DAB**: supports `$filter` with operators such as `eq`, `ne`, `gt`, `ge`, `lt`, `le`, `and`, `or`, and `not`.

---

## 2. Sorting

### Definitions

**Core Definition.** **Sorting** is the mechanism by which a client specifies the **order** in which a collection is returned. The server applies one or more sort keys, each with a direction (ascending or descending), to produce a deterministic sequence.

**Technical Definition.** Sorting is expressed through a **`sort` query parameter** whose value is a **comma-separated list of field names**, with a leading **`-`** indicating **descending** order (the JSON:API convention). For example, `sort=-created_at,priority` sorts first by `created_at` descending, then by `priority` ascending. Vendors may use alternative syntax: OData/DAB uses `$orderby=price desc,name asc`; some APIs split direction into separate parameters (`sortBy` + `sortOrder`), but this cannot express multi-column sorts and forces clients to learn custom parameter names. Sortable fields must be **whitelisted**, and sort columns should be **indexed**. Sorting affects **cacheability** because query parameters are part of the cache key.

**Beginner-Friendly Explanation.** Sorting is **choosing the order of a list**. You say “sort by price, highest first” or “sort by last name, then first name.” The API arranges the data accordingly and returns it. The cleanest syntax is a **single `sort` parameter** with comma-separated fields and a minus sign for descending: `sort=-price` means “most expensive first.” The server must decide **which fields are sortable** (sorting by a password hash would be a security risk) and should reject fields it does not recognize.

### Purposes

- **To present data in a meaningful order** (newest first, highest rated, alphabetical).
- **To support user-controlled ordering** through query parameters.
- **To enable stable pagination** when combined with cursor/keyset pagination on the sort key.
- **To provide deterministic results** across requests, preventing items from appearing or disappearing between pages.
- **To ensure consistent multi-field ordering** when a single field has ties.

### Syntax Rules and Structure

**General syntax (JSON:API convention).**

```
GET /v1/{resource}?sort=-{field},{field2}
```

**General syntax (OData/DAB).**

```
GET /v1/{resource}?$orderby={field} desc,{field2} asc
```

**Component breakdown.**

```
GET /v1/products?sort=-price,name
```

- `sort` — the sorting parameter.
- `-price` — **descending** order on `price` (the `-` prefix means descending).
- `name` — **ascending** order on `name` (no prefix means ascending).
- Comma-separated fields are applied **in order** (multi-field sorting).

**Sorting rules.**

| Rule | Guidance |
|---|---|
| **Use a single `sort` parameter** | Comma-separated fields with `-` for descending; avoids custom per-API parameter names. |
| **Whitelist sortable fields** | Reject unknown sort fields with `400 Bad Request`. |
| **Index sortable columns** | Ensure common sort fields use indexes for performance. |
| **Document the default sort** | Specify the order when no `sort` is provided (often primary key ascending). |
| **Always include a tiebreaker** | Append a unique field (e.g., `id`) to ensure deterministic ordering. |

**Constraints and limitations.**

- **Sorting on unindexed fields** causes full scans and degraded performance.
- **Multi-field sorting** requires compound indexes for optimal performance.
- **Query parameters are part of the cache key**; sorting reduces cacheability compared with unsorted responses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Sorting with a single `sort` parameter

```javascript
// app.js
app.get('/v1/products', async (req, res) => {
  // Step 1: Whitelist sortable fields.
  const SORTABLE = ['price', 'name', 'created_at'];

  // Step 2: Parse the sort parameter.
  const sortParam = req.query.sort || '-created_at'; // default: newest first
  const sortFields = sortParam.split(',').map((s) => {
    const desc = s.startsWith('-');
    const field = desc ? s.slice(1) : s;
    if (!SORTABLE.includes(field)) {
      throw new Error(`Unknown sort field: ${field}`);
    }
    return `${field} ${desc ? 'DESC' : 'ASC'}`;
  });

  // Step 3: Always append a unique tiebreaker.
  sortFields.push('id ASC');

  // Step 4: Query with the ORDER BY clause.
  const result = await db.query(
    `SELECT id, name, price, created_at
     FROM products
     ORDER BY ${sortFields.join(', ')}
     LIMIT 20`
  );

  res.json({ data: result.rows });
});
```

**Expected Output (conceptual):**
```
GET /v1/products?sort=-price,name
→ Sorts by price DESC, then name ASC, then id ASC as a tiebreaker.
GET /v1/products?sort=password → 400 Bad Request (unknown sort field).
```

**Why this output occurs.** The handler parses the comma-separated `sort` parameter, validates each field against the whitelist, converts the `-` prefix into `DESC`, and appends `id ASC` as a tiebreaker. The resulting `ORDER BY` clause produces a deterministic multi-field sort.

---

#### Example 2: Default sort and stable pagination

```javascript
// Step 1: Apply a default sort when none is provided.
const sort = req.query.sort || '-created_at,id';

// Step 2: Use the same sort key for cursor pagination.
// The cursor encodes the last-seen (created_at, id) pair.
const cursor = req.query.cursor
  ? decodeCursor(req.query.cursor)
  : null;

const query = cursor
  ? `SELECT * FROM posts
     WHERE (created_at, id) < ($1, $2)
     ORDER BY created_at DESC, id DESC
     LIMIT $3`
  : `SELECT * FROM posts
     ORDER BY created_at DESC, id DESC
     LIMIT $1`;

const params = cursor
  ? [cursor.created_at, cursor.id, limit + 1]
  : [limit + 1];
```

**Expected Output (conceptual):**
```
Without a cursor → first page, sorted by created_at DESC, id DESC.
With a cursor → next page, continuing from the last-seen position.
```

**Why this output occurs.** The sort key (`created_at DESC, id DESC`) is used both for ordering and for the cursor predicate. The `id` tiebreaker ensures that posts with identical `created_at` values are ordered deterministically, preventing duplicates or skips between pages.

---

### Real-World Cases

- **JSON:API convention**: `sort=-created_at,title` — comma-separated fields with `-` for descending.
- **OData/DAB**: `$orderby=price desc,name asc` — explicit direction tokens.
- **Azure API Design**: `sort=price` — a simple single-field sort parameter.

---

## 3. Field Selection (Sparse Fieldsets)

### Definitions

**Core Definition.** **Field selection** (also called **sparse fieldsets**) is the mechanism by which a client specifies **which attributes** of a resource should be included in the response, allowing the server to omit unrequested fields and return a **partial representation**.

**Technical Definition.** Field selection is expressed through a **`fields` query parameter** whose value is a **comma-separated list of attribute names**. In JSON:API, the parameter is keyed by resource type: `?fields[articles]=title,body&fields[people]=name`, allowing different field sets for the primary resource and each included resource type. The server must **whitelist** selectable fields, **always include mandatory fields** (`id` and `type` in JSON:API) regardless of the parameter, and **narrow the database `SELECT`** to avoid loading unneeded columns. Field selection is most valuable on **collection endpoints** with many items; single-resource `GET` endpoints should default to the full representation.

**Beginner-Friendly Explanation.** Field selection is **asking for only the parts of a record you need**. If you are showing a list of orders in a mobile app, you might only need the `id`, `status`, and `total` — not the full 30-field record with nested addresses and line items. With `?fields=id,status,total`, the API returns just those fields. This makes responses **smaller and faster**, especially on mobile networks. The API must decide **which fields are selectable** and must always include critical fields like `id`.

### Purposes

- **To reduce payload size** by returning only the attributes the client needs.
- **To improve performance** on mobile and low-bandwidth clients.
- **To narrow database queries** so unneeded columns are not loaded.
- **To support different client needs** from the same endpoint without separate resource representations.
- **To preserve hypermedia navigability** by always including mandatory identifiers and links.

### Syntax Rules and Structure

**General syntax (flat field selection).**

```
GET /v1/{resource}?fields={field1},{field2},{field3}
```

**General syntax (JSON:API typed field selection).**

```
GET /v1/articles?include=author&fields[articles]=title,body&fields[people]=name
```

**Component breakdown.**

```
GET /v1/orders?fields=id,status,total
```

- `fields` — the field-selection parameter.
- `id,status,total` — a **comma-separated** list of attributes to return.
- In JSON:API, `fields[articles]=title,body` scopes the selection to the `articles` resource type.

**Field selection rules.**

| Rule | Guidance |
|---|---|
| **Whitelist selectable fields** | Reject unknown fields; never expose fields the client is not authorized to see. |
| **Always include mandatory fields** | `id` (and `type` in JSON:API) must be included regardless of the parameter. |
| **Narrow the database `SELECT`** | Apply the field list to the SQL query, not just the serializer. |
| **Default to full representation** | Single-resource `GET` endpoints should return all fields unless `fields` is specified. |
| **Advertise support in links** | Include a templated link for field selection in hypermedia responses. |

**Constraints and limitations.**

- **Empty `fields` value** may mean “no fields” or “all fields” depending on the API; document the behavior.
- **Field selection on single-resource endpoints** provides less benefit and may surprise clients; default to full representation.
- **Nested field selection** (selecting fields within embedded resources) requires per-type `fields[TYPE]` parameters in JSON:API.

### Annotated Complete Step by Step Code Examples

#### Example 1: Field selection on a collection endpoint

```javascript
// app.js
app.get('/v1/orders', async (req, res) => {
  // Step 1: Whitelist selectable fields.
  const ALLOWED = ['id', 'status', 'total', 'currency', 'notes', 'shipping_address', 'billing_address'];

  // Step 2: Parse the fields parameter.
  const requested = req.query.fields
    ? req.query.fields.split(',').map((f) => f.trim())
    : ALLOWED;

  // Step 3: Validate and narrow to allowed fields.
  const selected = requested.filter((f) => ALLOWED.includes(f));
  if (selected.length !== requested.length) {
    return res.status(400).json({ error: 'Unknown field in fields parameter' });
  }

  // Step 4: Ensure mandatory fields are present.
  if (!selected.includes('id')) selected.unshift('id');

  // Step 5: Narrow the SQL SELECT and the serializer output.
  const result = await db.query(
    `SELECT ${selected.join(', ')} FROM orders WHERE customer_id = $1`,
    [req.user.id]
  );

  res.json({ data: result.rows });
});
```

**Expected Output (conceptual):**
```
GET /v1/orders?fields=id,status,total
→ Returns only id, status, and total for each order.
GET /v1/orders?fields=password → 400 Bad Request (unknown field).
```

**Why this output occurs.** The handler parses the comma-separated `fields` parameter, validates each field against the whitelist, ensures `id` is present, and narrows both the SQL `SELECT` and the response serialization. This reduces payload size and avoids loading unneeded columns.

---

#### Example 2: JSON:API typed field selection with includes

```http
GET /v1/articles?include=author&fields[articles]=title,body&fields[people]=name HTTP/1.1
```

**Expected Response (conceptual):**
```json
{
  "data": [
    {
      "type": "articles",
      "id": "1",
      "attributes": { "title": "Hello", "body": "World" },
      "relationships": { "author": { "data": { "type": "people", "id": "9" } } }
    }
  ],
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": { "name": "Alice" }
    }
  ]
}
```

**Why this output occurs.** The `fields[articles]` parameter limits article attributes to `title` and `body`, while `fields[people]` limits included author attributes to `name`. The `id` and `type` members are always included regardless of the field selection. This produces a minimal compound document containing only the requested data.

---

### Real-World Cases

- **Mews POS API**: uses `?fields[invoices]=total,subtotal,tax,createdAt,updatedAt` to return only requested invoice attributes.
- **Commerce Layer**: uses a comma-separated `fields` parameter to reduce payload size on collection endpoints.
- **Azure API Design**: recommends `fields=ProductID,Quantity` to let clients project only needed fields.

---

## 4. Embedding (Compound Documents)

### Definitions

**Core Definition.** **Embedding** (also called **inclusion** or **compound documents**) is the mechanism by which a client requests that **related resources** be returned **alongside the primary resource** in a single response, eliminating the need for follow-up requests.

**Technical Definition.** In JSON:API, embedding is expressed through an **`include` query parameter** whose value is a **comma-separated list of relationship paths** (e.g., `?include=author,comments`). The server returns a **compound document** containing the primary resource(s) in `data` and the related resources in a top-level `included` array, linked by `type` and `id` references. Relationship paths may be **nested** (`?include=author.comments`) to traverse multiple levels. The server may support **sparse fieldsets for included resources** via `fields[TYPE]` to limit which attributes of the included resources are returned. Embedding reduces the number of HTTP requests at the cost of larger responses and more complex queries.

**Beginner-Friendly Explanation.** Embedding is **asking for related data in the same response**. Instead of fetching an article, then making a second request for its author, then a third for the comments, you say `?include=author,comments` and get everything in one response. The primary data is in `data`, and the related records are in `included`. This is great for reducing round trips, especially on mobile networks. The trade-off is that the response is larger and the server has to join more tables — so you should include only what you actually need.

### Purposes

- **To reduce the number of HTTP requests** by returning related resources alongside the primary resource.
- **To improve performance** on high-latency networks by avoiding multiple round trips.
- **To support compound documents** that represent a resource and its relationships in one payload.
- **To enable nested inclusion** of deeply related resources through relationship paths.
- **To combine with field selection** to control the size of included resources.

### Syntax Rules and Structure

**General syntax (JSON:API include).**

```
GET /v1/{resource}?include={relationship},{relationship}
```

**General syntax (nested inclusion).**

```
GET /v1/articles?include=author.comments
```

**Component breakdown.**

```
GET /v1/articles?include=author,comments
```

- `include` — the embedding parameter.
- `author,comments` — a **comma-separated list of relationship paths**.
- The response contains a top-level `included` array with the related resources.
- Nested paths (`author.comments`) traverse relationships to include deeper resources.

**Embedding rules.**

| Rule | Guidance |
|---|---|
| **Whitelist includable relationships** | Reject unknown relationship paths with `400 Bad Request`. |
| **Return a compound document** | Place related resources in the top-level `included` array, linked by `type` and `id`. |
| **Support sparse fieldsets for included types** | Allow `fields[TYPE]` to limit attributes of included resources. |
| **Avoid unbounded nesting** | Cap relationship-path depth to prevent runaway queries. |
| **Document available includes** | List the relationship paths each endpoint supports. |

**Constraints and limitations.**

- **Larger responses**; embedding increases payload size and may reduce the benefit on fast networks.
- **More complex queries**; the server must join related tables, which can be expensive without proper indexes.
- **Over-fetching risk**; including too many relationships can duplicate data and bloat the response.
- **Not all relationships are includable**; some may be restricted for security or performance reasons.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic inclusion with `include`

```http
GET /v1/articles?include=author HTTP/1.1
```

**Expected Response (compound document):**
```json
{
  "data": [
    {
      "type": "articles",
      "id": "1",
      "attributes": { "title": "Hello", "body": "World" },
      "relationships": {
        "author": { "data": { "type": "people", "id": "9" } }
      }
    }
  ],
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": { "name": "Alice" }
    }
  ]
}
```

**Why this output occurs.** The `include=author` parameter instructs the server to return the related `author` resource in the top-level `included` array. The primary article is in `data`, and its `relationships.author.data` references the included person by `type` and `id`. The client can resolve the relationship without a second request.

---

#### Example 2: Nested inclusion with field selection

```http
GET /v1/articles?include=author.comments&fields[people]=name&fields[comments]=body HTTP/1.1
```

**Expected Response (conceptual):**
```json
{
  "data": [
    {
      "type": "articles",
      "id": "1",
      "relationships": {
        "author": { "data": { "type": "people", "id": "9" } }
      }
    }
  ],
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": { "name": "Alice" },
      "relationships": {
        "comments": {
          "data": [ { "type": "comments", "id": "100" } ]
        }
      }
    },
    {
      "type": "comments",
      "id": "100",
      "attributes": { "body": "Great article!" }
    }
  ]
}
```

**Why this output occurs.** The nested `include=author.comments` path traverses from article to author to comments. The `fields[people]` and `fields[comments]` parameters limit the attributes of the included resources to `name` and `body`, respectively. The response is a compound document that contains the full relationship graph while keeping each included resource minimal.

---

### Real-World Cases

- **JSON:API specification**: `?include=author,comments` returns a compound document with primary and related resources.
- **Drupal JSON:API**: supports compound documents via the `include` parameter for entity relationships.
- **Mews POS API**: combines `include` with sparse fieldsets to fetch related resources with only the needed attributes.

---

## 5. Filtering and Sorting Best Practices

### Definitions

**Core Definition.** **Filtering and sorting best practices** are empirically established guidelines for designing, securing, and operating collection-query mechanisms so that they are **safe, performant, consistent, and well-documented**.

**Technical Definition.** Best practices span **whitelisting** (explicitly allowing filterable, sortable, and selectable fields), **operator validation** (matching operators to field types), **parameterized queries** (preventing SQL injection), **indexing** (ensuring filter and sort columns are indexed), **deterministic ordering** (always including a unique tiebreaker in sorts), **consistent parameter naming** (single `sort` parameter with comma-separated fields and `-` prefix), **field selection defaults** (full representation on single-resource endpoints, sparse fieldsets on collections), **inclusion limits** (capping relationship-path depth), and **documentation** (listing allowed fields, operators, defaults, and constraints).

**Beginner-Friendly Explanation.** Best practices are **the rules that keep filtering and sorting from becoming dangerous or slow**. **Whitelist** the fields clients can filter, sort, and select — never let them touch fields you have not approved. **Validate** that operators make sense for the field type (no “contains” on numbers). **Parameterize** your queries so user input cannot become SQL. **Index** the columns you filter and sort on. **Always add a tiebreaker** to sorts so results are deterministic. **Document** everything: which fields are allowed, which operators work, what the default sort is, and what the maximum limits are. These are the habits that separate a safe, fast API from one that is vulnerable and slow.

### Purposes

- **To prevent SQL injection** by never concatenating user input into queries.
- **To prevent data leakage** by restricting filtering and selection to authorized fields.
- **To maintain performance** by ensuring filter and sort columns are indexed.
- **To provide deterministic results** by always including a unique tiebreaker in sort orders.
- **To give clients a predictable, consistent interface** through uniform parameter naming and documented defaults.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Whitelisting** | Explicitly allow filterable, sortable, and selectable fields; reject unknown ones with `400`. |
| **Operator validation** | Validate operator–field-type compatibility at parse time. |
| **Parameterization** | Use parameterized queries; never concatenate filter values into SQL. |
| **Indexing** | Index filterable and sortable columns; use compound indexes for multi-field sorts. |
| **Sorting syntax** | Use a single `sort` parameter with comma-separated fields and `-` for descending. |
| **Tiebreaker** | Always append a unique field (e.g., `id`) to sort orders for deterministic pagination. |
| **Field selection** | Default to full representation on single-resource `GET`; support `fields` on collections. |
| **Inclusion depth** | Cap nested `include` paths to prevent runaway queries. |
| **Documentation** | Document allowed fields, operators, defaults, maximums, and supported includes. |

**Constraints and limitations.**

- **Whitelisting requires ongoing maintenance**; new fields must be explicitly added to the allowlist.
- **Indexing adds write overhead**; balance query performance against write throughput.
- **No single parameter convention is universal**; choose one (JSON:API `sort` + `fields` + `include` is a strong default) and apply it consistently.

### Annotated Complete Step by Step Code Examples

#### Example 1: A secure, validated filter-sort pipeline

```javascript
// Step 1: Define allowlists for each mechanism.
const FILTERABLE = { status: 'string', minCost: 'number', created_at: 'date' };
const SORTABLE = ['price', 'name', 'created_at'];
const SELECTABLE = ['id', 'status', 'total', 'currency', 'created_at'];

// Step 2: Validate and parameterize filters.
function buildFilterQuery(query) {
  const conditions = [];
  const params = [];
  let idx = 1;

  for (const [key, value] of Object.entries(query)) {
    if (!FILTERABLE[key]) throw new Error(`Unknown filter: ${key}`);
    const type = FILTERABLE[key];
    if (type === 'number') {
      conditions.push(`${key} >= $${idx++}`);
      params.push(Number(value));
    } else if (type === 'date') {
      conditions.push(`${key} >= $${idx++}`);
      params.push(value);
    } else {
      conditions.push(`${key} = $${idx++}`);
      params.push(value);
    }
  }

  return { where: conditions.length ? `WHERE ${conditions.join(' AND ')}` : '', params };
}

// Step 3: Validate and build the sort clause.
function buildSortClause(sortParam) {
  const fields = (sortParam || '-created_at,id').split(',');
  const clauses = fields.map((s) => {
    const desc = s.startsWith('-');
    const field = desc ? s.slice(1) : s;
    if (!SORTABLE.includes(field) && field !== 'id') {
      throw new Error(`Unknown sort field: ${field}`);
    }
    return `${field} ${desc ? 'DESC' : 'ASC'}`;
  });
  return `ORDER BY ${clauses.join(', ')}`;
}

// Step 4: Validate and narrow field selection.
function buildSelectClause(fieldsParam) {
  const requested = fieldsParam ? fieldsParam.split(',') : SELECTABLE;
  const selected = requested.filter((f) => SELECTABLE.includes(f));
  if (!selected.includes('id')) selected.unshift('id');
  return selected.join(', ');
}
```

**Expected Output (conceptual):**
```
Valid filters, sorts, and field selections → parameterized query with whitelisted columns.
Invalid filter or sort field → 400 Bad Request with a clear error.
```

**Why this output occurs.** The pipeline validates every query parameter against an allowlist, parameterizes filter values, builds a deterministic sort clause with an `id` tiebreaker, and narrows the `SELECT` to approved fields. This prevents SQL injection, data leakage, and unexpected database errors.

---

#### Example 2: Combining filtering, sorting, and field selection

```http
GET /v1/orders?status=shipped&minCost=100&sort=-created_at&fields=id,status,total HTTP/1.1
```

**Expected Output (conceptual):**
```json
{
  "data": [
    { "id": 1001, "status": "shipped", "total": 150 },
    { "id": 1002, "status": "shipped", "total": 200 }
  ]
}
```

**Why this output occurs.** The filter narrows the result set to shipped orders with a cost of at least 100, the sort orders them by `created_at` descending (with an implicit `id` tiebreaker), and the field selection returns only `id`, `status`, and `total`. The three mechanisms compose to produce a minimal, precisely tailored response.

---

### Real-World Cases

- **Stripe API**: whitelists filterable fields, uses bracket-notation operators, and documents every parameter.
- **JSON:API implementations**: enforce sparse fieldsets and compound documents with per-type `fields[TYPE]` and `include` parameters.
- **Azure API Design**: recommends limiting returned items, validating requested fields, and documenting filtering and sorting behavior.

---

## References

- Azure Architecture Center — Web API Design Best Practices (Filtering, Sorting, Field Selection) - https://learn.microsoft.com/pl-pl/azure/architecture/best-practices/api-design
- RestLib ADR-013 — Filter Operators (Bracket Syntax, Per-Property Allow-Lists, Type Safety) - https://github.com/Adrian01987/RestLib/blob/main/docs/adr/013-filter-operators.md
- dot-skills — Collection Field Selection (Sparse Fieldsets via `fields` Parameter) - https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/restful-hateoas/references/coll-field-selection.md
- dot-skills — Collection Sorting Convention (Single `sort` Parameter, Comma-Separated, `-` Prefix) - https://github.com/pproenca/dot-skills/blob/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.curated/restful-hateoas/references/coll-sorting-convention.md
- JSON:API Specification v1.1 — Sparse Fieldsets - https://jsonapi.org/format/#fetching-sparse-fieldsets
- JSON:API Specification v1.1 — Compound Documents - https://jsonapi.org/format/#document-compound-documents
- Mews POS API — Sparse Fieldsets - https://docs.mews.com/pos-api/guidelines/sparse-fieldsets
- Data API Builder (Microsoft Learn) — `$filter` and `$orderby` - https://learn.microsoft.com/en-us/azure/data-api-builder/concept/rest/filter
- Data API Builder (Microsoft Learn) — `$orderby` (REST) - https://learn.microsoft.com/en-us/azure/data-api-builder/concept/rest/orderby
- Alibaba Cloud Developer — API Interface SQL Injection Protection (Whitelisting, Parameterization) - https://developer.aliyun.com/article/1578822
- BloomViewSetConfig — Filtering and Ordering Allowlists - https://pub.dev/documentation/bloom_server/latest/bloom_server/BloomViewSetConfig-class.html
- Commerce Layer Docs — Sparse Fieldsets - https://docs.commercelayer.io/developers/sparse-fieldsets