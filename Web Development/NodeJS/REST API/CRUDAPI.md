# Data Querying & Partial Updates (CRUD) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Data querying and partial updates in REST APIs encompass the patterns and conventions for creating, reading, updating, and deleting resources, with particular emphasis on the distinction between full resource replacement (PUT) and partial modification (PATCH), efficient data selection, and strategies for handling large datasets.

**Technical Definition:** The CRUD cycle maps HTTP methods to resource operations: POST (Create), GET (Read), PUT/PATCH (Update), and DELETE (Delete). PUT replaces the entire resource representation and is idempotent; PATCH applies partial modifications and is not guaranteed to be idempotent. Standardized PATCH formats include JSON Merge Patch (RFC 7396), which sends a sparse document resembling the target, and JSON Patch (RFC 6902), which expresses a sequence of operations to apply. Large dataset handling employs pagination (offset-based vs. cursor-based/keyset), sparse fieldsets for field filtering, and query string syntaxes for filtering, sorting, and full-text search.

**Beginner-Friendly Explanation:** CRUD is the four basic things you do with data: Create (add new), Read (look up), Update (change), and Delete (remove). Think of a contact list on your phone. Adding a contact is Create. Viewing it is Read. Editing the phone number is Update — and there are two ways to edit: PUT means "replace the entire contact card with this new one," while PATCH means "just change the phone number, leave everything else alone." When you have thousands of contacts, you need pagination (showing 20 at a time), filtering (show only contacts starting with "A"), and sorting (alphabetical order).

### Key Characteristics

- **Method semantics:** PUT replaces the entire resource; PATCH modifies specific fields; POST creates; DELETE removes.
- **Idempotency:** PUT and DELETE are idempotent; POST and PATCH are not guaranteed to be.
- **Standardized PATCH formats:** JSON Merge Patch (RFC 7396) is simple and readable; JSON Patch (RFC 6902) is operation-based and supports array manipulation.
- **Sparse fieldsets:** Clients can request only the fields they need, reducing payload size.
- **Pagination strategies:** Offset-based (simple, page numbers) vs. cursor-based/keyset (stable, performant at depth).
- **Query string filtering:** Operators like `eq`, `gte`, `lte`, `contains` enable precise data selection.
- **Full-text search:** Advanced search capabilities using `tsvector`/`tsquery` (PostgreSQL) or dedicated search engines.

### Prerequisites

- **HTTP fundamentals:** Request methods, status codes, headers, and bodies.
- **REST architectural constraints:** Resource-oriented design, statelessness, uniform interface.
- **JSON fundamentals:** Object structure, nesting, and serialization.
- **Basic understanding of databases:** SQL or NoSQL query concepts.
- **Familiarity with a server framework:** Express, Fastify, or NestJS.

### Related Programming Areas

- **Advanced API Design & Response Formatting:** Status codes, error handling, and RFC 7807.
- **Database Querying:** SQL LIMIT/OFFSET, WHERE clauses, and indexes.
- **Search Engines:** Elasticsearch, Typesense, and PostgreSQL full-text search.
- **API Documentation:** OpenAPI Specification for describing query parameters and request bodies.

### Core Concepts

1. **The Core CRUD Cycle** — creating, reading, destroying; PUT vs. PATCH; JSON Merge Patch vs. JSON Patch.
2. **Data Selection & Optimization** — sparse fieldsets.
3. **Large Dataset Handling** — pagination, filtering, sorting, and full-text search.

---

## Core Concept 1: The Core CRUD Cycle

### Sub-Feature 1.1: Creating, Reading, and Destroying Resources

#### Definitions

**Core Definition:** The CRUD cycle consists of four fundamental operations: Create (POST), Read (GET), Update (PUT/PATCH), and Delete (DELETE), which map directly to HTTP methods for resource manipulation.

**Technical Definition:** In REST, resources are created via POST to a collection URI (e.g., `POST /users`), read via GET to a collection or item URI (e.g., `GET /users` or `GET /users/42`), updated via PUT or PATCH to an item URI (e.g., `PUT /users/42`), and deleted via DELETE to an item URI (e.g., `DELETE /users/42`). PUT is applied to individual resources, not collections, and is idempotent — sending the same request multiple times always results in the same resource state. POST and PATCH are not guaranteed to be idempotent.

**Beginner-Friendly Explanation:** Create is like adding a new file to a filing cabinet (POST). Read is like looking up a file (GET). Update is like editing a file (PUT replaces the whole file; PATCH changes just a few lines). Delete is like removing the file from the cabinet (DELETE). The key rule: PUT says "here's the complete new version," while PATCH says "here are just the changes."

#### Purposes

- To provide a standardised, predictable interface for resource manipulation.
- To leverage HTTP method semantics for caching, retry logic, and idempotency.
- To enable consistent client-side handling of resource lifecycles.

#### Syntax Rules and Structure

| Operation | HTTP Method | URI Pattern | Status Code | Idempotent |
|-----------|-------------|-------------|-------------|------------|
| Create | POST | `/collection` | 201 Created | No |
| Read (collection) | GET | `/collection` | 200 OK | Yes |
| Read (item) | GET | `/collection/{id}` | 200 OK | Yes |
| Full update | PUT | `/collection/{id}` | 200/204 | Yes |
| Partial update | PATCH | `/collection/{id}` | 200/204 | No |
| Delete | DELETE | `/collection/{id}` | 204 No Content | Yes |

**Constraints and Limitations:**
- PUT requests should be idempotent; sending the same request multiple times should not change the resource state beyond the first application.
- POST and PATCH are not guaranteed to be idempotent.
- DELETE may return 204 (success, no body) or 200 (success with body).

#### Annotated Code Example

```javascript
// Express: Complete CRUD cycle
const express = require('express');
const app = express();
app.use(express.json());

let users = [{ id: 1, name: 'Alice', email: 'alice@example.com' }];
let nextId = 2;

// CREATE — POST /users
app.post('/v1/users', (req, res) => {
  const user = { id: nextId++, name: req.body.name, email: req.body.email };
  users.push(user);
  res.status(201).location(`/v1/users/${user.id}`).json(user);
});

// READ (collection) — GET /users
app.get('/v1/users', (req, res) => {
  res.json({ data: users, meta: { total: users.length } });
});

// READ (item) — GET /users/:id
app.get('/v1/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json({ data: user });
});

// DELETE — DELETE /users/:id
app.delete('/v1/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ error: 'User not found' });
  users.splice(index, 1);
  res.status(204).end();
});

app.listen(3000);
```

**Expected Output (for `POST /v1/users` with `{"name":"Bob","email":"bob@example.com"}`):**
```http
HTTP/1.1 201 Created
Location: /v1/users/2

{"id":2,"name":"Bob","email":"bob@example.com"}
```

**Expected Output (for `DELETE /v1/users/1`):**
```http
HTTP/1.1 204 No Content
```

**Why this output:** POST creates a new user and returns 201 with a `Location` header. DELETE removes the user and returns 204 with no body. The server assigns the ID, which is standard practice for POST creation — if the client can assign the URI, PUT may be used for creation instead.

#### Real-World Cases

- **User management APIs:** POST to create, GET to list/retrieve, DELETE to remove.
- **E-commerce:** POST `/orders` to place an order, GET `/orders` to view history, DELETE `/orders/42` to cancel.
- **Content management:** POST `/posts` to publish, GET `/posts` to read, DELETE `/posts/42` to remove.

---

### Sub-Feature 1.2: Complete vs. Partial Updates — The Strict Difference Between PUT and PATCH

#### Definitions

**Core Definition:** PUT replaces the entire resource representation with the request payload, requiring the client to send every field; PATCH applies a partial modification, allowing the client to send only the fields that need to change.

**Technical Definition:** A PUT request replaces the entire resource representation; the client must send every field, even if only one is changing. A PATCH request performs a partial update for an existing resource; the client specifies a set of changes to apply. This method can be more efficient than using PUT because the client sends only the changes, not the whole resource representation. PUT must be idempotent — re-sending the same request always results in the same resource state. PATCH is not guaranteed to be idempotent.

**Beginner-Friendly Explanation:** PUT is like replacing a whole document with a new version — you must provide the complete document. PATCH is like using a sticky note to say "change the phone number on page 3 to 555-1234" — you only specify what changes. If you PUT a document that's missing some fields, they get erased. If you PATCH, only the specified fields change.

#### Purposes

- To allow efficient updates when only a few fields change (PATCH).
- To ensure predictable, idempotent full replacements (PUT).
- To reduce bandwidth and payload size for partial modifications.
- To avoid accidental data loss from incomplete PUT requests.

#### Syntax Rules and Structure

| Aspect | PUT | PATCH |
|--------|-----|-------|
| Scope | Full replacement | Partial modification |
| Body | Complete representation | Only changed fields |
| Idempotent | Yes | No (not guaranteed) |
| Missing fields | Set to null/removed | Left unchanged |
| Status codes | 200, 201, 204 | 200, 204 |
| Media type | `application/json` | `application/merge-patch+json` or `application/json-patch+json` |

**Constraints and Limitations:**
- PUT requests should be idempotent; PATCH requests are not.
- PUT with missing required fields should fail with 400 or 409.
- PATCH requires a patch document format; the server must know how to apply it.
- RFC 5789 defines PATCH but does not mandate a specific patch format.

#### Annotated Code Example

```javascript
// Express: PUT vs. PATCH
const express = require('express');
const app = express();
app.use(express.json());

let user = { id: 42, name: 'Alice', email: 'alice@example.com', age: 30 };

// PUT — Full replacement (client must send all fields)
app.put('/v1/users/42', (req, res) => {
  // Validate required fields
  if (!req.body.name || !req.body.email) {
    return res.status(400).json({ error: 'name and email are required' });
  }
  // Replace entire resource
  user = { id: 42, ...req.body };
  res.status(200).json(user);
});

// PATCH — Partial update (client sends only changed fields)
app.patch('/v1/users/42', (req, res) => {
  // Apply only the provided fields
  Object.assign(user, req.body);
  res.status(200).json(user);
});

app.listen(3000);
```

**Expected Output (for `PUT /v1/users/42` with `{"name":"Alice Smith","email":"alice.smith@example.com"}`):**
```json
{"id":42,"name":"Alice Smith","email":"alice.smith@example.com"}
```
Note: `age` is removed because PUT replaces the entire resource.

**Expected Output (for `PATCH /v1/users/42` with `{"email":"new@example.com"}`):**
```json
{"id":42,"name":"Alice Smith","email":"new@example.com","age":30}
```
Note: Only `email` changes; `name` and `age` are preserved.

**Why this output:** PUT replaces the entire resource, so fields not included in the request body are removed. PATCH merges the provided fields into the existing resource, preserving unchanged fields. This demonstrates the fundamental semantic difference between the two methods.

#### Real-World Cases

- **User profiles:** PATCH to update a single field (e.g., email); PUT to replace the entire profile.
- **Configuration management:** PUT to replace a complete config; PATCH to toggle one setting.
- **Document editing:** PATCH to apply a diff; PUT to upload a new version.

---

### Sub-Feature 1.3: Implementing Standardized PATCH Formats — JSON Merge Patch vs. JSON Patch (RFC 6902)

#### Definitions

**Core Definition:** JSON Merge Patch (RFC 7396) sends a sparse document that looks like the target resource and is merged into it; JSON Patch (RFC 6902) sends an array of operations (add, remove, replace, move, copy, test) to apply to the target document.

**Technical Definition:** JSON Merge Patch is a standardized way to send the part of a JSON document that changed instead of sending the whole document. A patch looks like the document it changes: object fields recurse, ordinary values replace, and `null` deletes a field. JSON Patch (RFC 6902) defines a JSON document structure for expressing a sequence of operations to apply to a JSON document; it is suitable for use with the HTTP PATCH method. The patch document is a JSON array where each item is an object describing a modification. JSON Patch supports array manipulation (add, remove, move, copy) and test operations, while JSON Merge Patch does not.

**Beginner-Friendly Explanation:** JSON Merge Patch is like telling someone "change the colour to red and delete the size field" — you describe the result you want. JSON Patch is like giving step-by-step instructions: "first, replace the name; second, add a new phone number to the list; third, remove the old email." JSON Merge Patch is simpler; JSON Patch is more powerful, especially for arrays.

#### Purposes

- To standardise partial update formats across APIs.
- To enable array manipulation (JSON Patch) or simple field-level changes (JSON Merge Patch).
- To support optimistic concurrency with test operations (JSON Patch).
- To reduce payload size for partial updates.

#### Syntax Rules and Structure

**JSON Merge Patch media type:** `application/merge-patch+json`

**JSON Patch media type:** `application/json-patch+json`

**JSON Merge Patch rules:**
| Rule | Behaviour |
|------|-----------|
| Object field | Recursively merged |
| Scalar value | Replaced |
| `null` value | Field deleted |
| Array | Replaced wholesale |

**JSON Patch operations:**
| Operation | Description | Example |
|-----------|-------------|---------|
| `add` | Add a value at a path | `{"op":"add","path":"/tags/-","value":"new"}` |
| `remove` | Remove a value | `{"op":"remove","path":"/age"}` |
| `replace` | Replace a value | `{"op":"replace","path":"/name","value":"Bob"}` |
| `move` | Move a value | `{"op":"move","from":"/a","path":"/b"}` |
| `copy` | Copy a value | `{"op":"copy","from":"/a","path":"/b"}` |
| `test` | Test a value | `{"op":"test","path":"/name","value":"Alice"}` |

**Constraints and Limitations:**
- JSON Merge Patch cannot manipulate individual array elements; arrays are replaced wholesale.
- JSON Patch requires the server to support the specific operation set.
- JSON Merge Patch with `null` deletes fields; you cannot set a field to `null` using merge patch.
- JSON Patch is more verbose but more precise.

#### Annotated Code Example

```javascript
// Express: JSON Merge Patch implementation
const express = require('express');
const app = express();
app.use(express.json({ type: 'application/merge-patch+json' }));

let user = {
  id: 42,
  name: 'Alice',
  email: 'alice@example.com',
  tags: ['admin', 'user'],
  address: { city: 'London', country: 'UK' },
};

function applyMergePatch(target, patch) {
  for (const [key, value] of Object.entries(patch)) {
    if (value === null) {
      delete target[key];
    } else if (typeof value === 'object' && !Array.isArray(value) && typeof target[key] === 'object') {
      applyMergePatch(target[key], value);
    } else {
      target[key] = value;
    }
  }
  return target;
}

app.patch('/v1/users/42', (req, res) => {
  user = applyMergePatch(user, req.body);
  res.json(user);
});

app.listen(3000);
```

**Expected Output (for PATCH with `{"email":"new@example.com","address":{"city":"Paris"},"tags":["user"]}`):**
```json
{
  "id": 42,
  "name": "Alice",
  "email": "new@example.com",
  "address": { "city": "Paris", "country": "UK" },
  "tags": ["user"]
}
```

**Why this output:** The email is replaced. The address object is recursively merged — `city` changes to "Paris" but `country` remains "UK". The `tags` array is replaced wholesale. This demonstrates JSON Merge Patch's recursive object merge and wholesale array replacement.

```javascript
// Express: JSON Patch implementation (simplified)
app.patch('/v1/users/42', (req, res) => {
  const patch = req.body; // JSON Patch array

  for (const op of patch) {
    const path = op.path.split('/').filter(Boolean);

    if (op.op === 'replace') {
      let target = user;
      for (let i = 0; i < path.length - 1; i++) target = target[path[i]];
      target[path[path.length - 1]] = op.value;
    } else if (op.op === 'add') {
      // Handle array append (path ends with '-')
      if (path[path.length - 1] === '-') {
        const arr = path.slice(0, -1).reduce((o, k) => o[k], user);
        arr.push(op.value);
      }
    } else if (op.op === 'remove') {
      let target = user;
      for (let i = 0; i < path.length - 1; i++) target = target[path[i]];
      delete target[path[path.length - 1]];
    }
  }
  res.json(user);
});
```

**Expected Output (for JSON Patch with `[{"op":"replace","path":"/name","value":"Bob"},{"op":"add","path":"/tags/-","value":"editor"}]`):**
```json
{
  "id": 42,
  "name": "Bob",
  "email": "new@example.com",
  "tags": ["user", "editor"],
  "address": { "city": "Paris", "country": "UK" }
}
```

**Why this output:** JSON Patch applies each operation in sequence. `replace` changes the name. `add` with path `/tags/-` appends "editor" to the tags array. Unlike JSON Merge Patch, JSON Patch can manipulate individual array elements.

#### Real-World Cases

- **Configuration APIs:** JSON Merge Patch for updating nested config objects.
- **Collaborative editing:** JSON Patch for applying diffs between document versions.
- **Feature flag APIs:** JSON Merge Patch for toggling individual flags.
- **Array-based data:** JSON Patch for adding/removing items in lists (tags, permissions, etc.).

---

## Core Concept 2: Data Selection & Optimization

### Sub-Feature 2.1: Fields Filtering / Sparse Fieldsets

#### Definitions

**Core Definition:** Sparse fieldsets allow clients to request only a subset of fields from a resource, reducing response payload size and improving performance.

**Technical Definition:** Sparse fieldsets are specified using the `fields[TYPE]` query parameter, where `TYPE` is the resource type. The value is a comma-separated list of field names to be returned. This is a JSON:API feature but is widely adopted by other APIs. The `fields` parameter must be URI-encoded because field names can contain unsafe URL characters such as square brackets. Support for fieldsets is endpoint-specific.

**Beginner-Friendly Explanation:** Imagine a user profile with 50 fields, but you only need the name and email. Instead of receiving all 50 fields (wasting bandwidth and processing time), you tell the server "just give me name and email." That's a sparse fieldset. It's like ordering a burger with only the toppings you want, instead of getting everything on the menu.

#### Purposes

- To reduce response payload size and network bandwidth.
- To improve client-side processing performance.
- To enable mobile or low-bandwidth clients to fetch only essential data.
- To support different client needs (e.g., list view vs. detail view).

#### Syntax Rules and Structure

```
GET /v1/invoices?fields[invoices]=total,subtotal,tax,createdAt,updatedAt
```

| Component | Description |
|-----------|-------------|
| `fields` | Query parameter name. |
| `[invoices]` | The resource type in brackets. |
| `total,subtotal,...` | Comma-separated field names. |

**Constraints and Limitations:**
- The `fields` parameter must be URI-encoded (e.g., `%5B` for `[`, `%5D` for `]`).
- An empty value (`fields[invoices]=`) indicates no fields should be returned.
- Support is endpoint-specific; not all fields may be available for selection.
- Required fields (e.g., `id`) may be automatically included.

#### Annotated Code Example

```javascript
// Express: Sparse fieldsets implementation
const express = require('express');
const app = express();

const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com', age: 30, role: 'admin', createdAt: '2026-01-01' },
  { id: 2, name: 'Bob', email: 'bob@example.com', age: 25, role: 'user', createdAt: '2026-01-02' },
];

app.get('/v1/users', (req, res) => {
  // Parse fields[users]=id,name,email
  const fieldsParam = req.query.fields;
  const selectedFields = fieldsParam?.['users']?.split(',').filter(Boolean);

  const data = users.map(user => {
    if (!selectedFields) return user; // No sparse fieldset — return all fields
    const filtered = {};
    for (const field of selectedFields) {
      if (field in user) filtered[field] = user[field];
    }
    return filtered;
  });

  res.json({ data });
});

app.listen(3000);
```

**Expected Output (for `GET /v1/users?fields[users]=id,name`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ]
}
```

**Expected Output (for `GET /v1/users`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", "email": "alice@example.com", "age": 30, "role": "admin", "createdAt": "2026-01-01" },
    { "id": 2, "name": "Bob", "email": "bob@example.com", "age": 25, "role": "user", "createdAt": "2026-01-02" }
  ]
}
```

**Why this output:** When `fields[users]=id,name` is provided, only the `id` and `name` fields are returned for each user. Without the parameter, all fields are returned. This reduces the payload size from ~150 bytes per user to ~30 bytes per user — an 80% reduction.

#### Real-World Cases

- **Mobile apps:** Fetching only `id` and `name` for list views; full details on tap.
- **Dashboards:** Requesting only aggregate fields (e.g., `total`, `count`).
- **API performance:** Reducing bandwidth costs for high-volume list endpoints.

---

## Core Concept 3: Large Dataset Handling

### Sub-Feature 3.1: Pagination Strategies — Offset-based vs. Cursor-based (Keyset) Pagination

#### Definitions

**Core Definition:** Offset-based pagination uses `limit` and `offset` (or `page` and `per_page`) to skip a fixed number of records; cursor-based (keyset) pagination uses a cursor identifying the last seen record to fetch the next batch without counting.

**Technical Definition:** Offset pagination maps directly to SQL `LIMIT` and `OFFSET`. The client sends a page number and a page size; the server translates them into `LIMIT 25 OFFSET 50`. Cursor-based pagination — also called keyset pagination — drops the row counter. Instead of "skip 50 rows," the client says "give me rows after this specific record." The cursor identifies the last row the client saw, so the server can seek directly to the next batch. Offset pagination suffers from page drift (inserts/deletes shift pages) and deep-offset scans (OFFSET 500000 scans all skipped rows). Cursor pagination provides stable results and consistent latency at any depth.

**Beginner-Friendly Explanation:** Offset pagination is like reading a book by saying "skip to page 5." If someone inserts pages earlier in the book while you're reading, you might miss or repeat content. Cursor pagination is like putting a bookmark on the last page you read — "continue from here." No matter how many pages are inserted before your bookmark, you always continue from exactly where you left off.

#### Purposes

- To split large result sets into manageable chunks.
- To reduce server load and response times for list endpoints.
- To provide stable, consistent pagination for real-time data.
- To enable infinite scroll and "load more" patterns in UIs.

#### Syntax Rules and Structure

**Offset-based:**
```
GET /v1/orders?page=3&per_page=25
```
```sql
SELECT * FROM orders ORDER BY created_at DESC LIMIT 25 OFFSET 50;
```

**Cursor-based:**
```
GET /v1/orders?after=ord_8821&limit=25
```
```sql
SELECT * FROM orders WHERE id > 'ord_8821' ORDER BY id LIMIT 25;
```

| Aspect | Offset-based | Cursor-based |
|--------|-------------|--------------|
| Params | `page`, `per_page` (or `limit`, `offset`) | `after`, `before`, `limit` |
| Page jumping | Yes | No |
| Total count | Available | Not available |
| Page drift | Yes | No |
| Deep performance | Degrades (O(n)) | Constant (O(1)) |
| Best for | Small datasets, admin UIs | Large datasets, real-time feeds |

**Constraints and Limitations:**
- Offset pagination is only acceptable for small datasets or admin UIs where page jumping matters.
- Cursor pagination does not support jumping to a specific page.
- Cursor-based pagination requires a stable, unique ordering column (e.g., `id`, `created_at`).
- Cursors should be opaque (base64-encoded) to prevent clients from constructing them.

#### Annotated Code Example

```javascript
// Express: Offset-based pagination
app.get('/v1/orders', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const perPage = Math.min(parseInt(req.query.per_page) || 25, 100);
  const offset = (page - 1) * perPage;

  // Simulated database query
  const orders = db.query(
    'SELECT * FROM orders ORDER BY created_at DESC LIMIT $1 OFFSET $2',
    [perPage, offset]
  );

  res.json({
    data: orders,
    page,
    per_page: perPage,
    total: db.query('SELECT COUNT(*) FROM orders'),
  });
});
```

```javascript
// Express: Cursor-based pagination
app.get('/v1/orders', (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 25, 100);
  const after = req.query.after;

  let query = 'SELECT * FROM orders';
  const params = [limit + 1]; // Fetch one extra to detect hasMore

  if (after) {
    const decodedCursor = Buffer.from(after, 'base64').toString();
    query += ' WHERE created_at < $2';
    params.push(decodedCursor);
  }
  query += ' ORDER BY created_at DESC LIMIT $1';

  const rows = db.query(query, params);
  const hasMore = rows.length > limit;
  const data = hasMore ? rows.slice(0, limit) : rows;

  const nextCursor = hasMore
    ? Buffer.from(data[data.length - 1].created_at).toString('base64')
    : null;

  res.json({
    data,
    pagination: {
      has_more: hasMore,
      next_cursor: nextCursor,
    },
  });
});
```

**Expected Output (offset-based, `GET /v1/orders?page=2&per_page=25`):**
```json
{
  "data": [ /* 25 orders */ ],
  "page": 2,
  "per_page": 25,
  "total": 1848203
}
```

**Expected Output (cursor-based, `GET /v1/orders?limit=25`):**
```json
{
  "data": [ /* 25 orders */ ],
  "pagination": {
    "has_more": true,
    "next_cursor": "MjAyNi0wOC0zMFQxNDoyMjowN1o="
  }
}
```

**Why this output:** Offset pagination returns the page number, page size, and total count. Cursor pagination returns a `next_cursor` that the client sends in the next request. The cursor is base64-encoded to prevent clients from constructing it. The `has_more` flag indicates whether more data exists.

#### Real-World Cases

- **Stripe:** Cursor-based pagination for all list endpoints (`starting_after`, `ending_before`).
- **Slack:** Cursor-based pagination for message history.
- **GitHub:** Offset-based pagination for most list endpoints (with `page` and `per_page`).

---

### Sub-Feature 3.2: Filtering & Sorting — Designing Query String Syntaxes for Complex Filters

#### Definitions

**Core Definition:** Filtering allows clients to narrow the result set by applying conditions; sorting allows clients to order the results by one or more fields in ascending or descending order.

**Technical Definition:** Filtering in REST APIs can use either a query parameter approach (`?age[gte]=18`) or an OData-inspired `$filter` syntax (`?$filter=age ge 18`). Common comparison operators include `eq` (equals), `ne` (not equals), `gt` (greater than), `gte` (greater than or equal), `lt` (less than), `lte` (less than or equal), `co` (contains), `sw` (starts with), and `ew` (ends with). Sorting uses an `$orderby` parameter with comma-separated field names and optional `asc`/`desc` suffixes: `?$orderby=name asc,created_at desc`.

**Beginner-Friendly Explanation:** Filtering is like using a search form: "show me users aged 18 or older who live in London." Sorting is like choosing how to arrange the results: "sort by name A to Z." A well-designed query string syntax makes both operations intuitive and composable.

#### Purposes

- To enable clients to retrieve only relevant data.
- To support complex search and filtering requirements.
- To allow users to sort results by different criteria.
- To reduce the amount of data transferred and processed client-side.

#### Syntax Rules and Structure

| Operator | Meaning | Example |
|----------|---------|---------|
| `eq` | Equal | `?status[eq]=active` |
| `ne` | Not equal | `?status[ne]=deleted` |
| `gt` | Greater than | `?age[gt]=18` |
| `gte` | Greater than or equal | `?age[gte]=18` |
| `lt` | Less than | `?price[lt]=100` |
| `lte` | Less than or equal | `?price[lte]=100` |
| `co` | Contains | `?name[co]=ali` |
| `sw` | Starts with | `?name[sw]=Ali` |
| `ew` | Ends with | `?email[ew]=@example.com` |

**Sorting:**
```
GET /v1/users?sort=name,-createdAt
```
- Prefix `-` indicates descending order.
- Without prefix, ascending order.

**Constraints and Limitations:**
- Filter operators and syntax vary by API; there is no universal standard.
- URL encoding is required for operators and values containing special characters.
- Complex filters may impact database performance; indexes are essential.
- Nested filters (e.g., `?address[city][eq]=London`) require careful parsing.

#### Annotated Code Example

```javascript
// Express: Filtering and sorting
app.get('/v1/users', (req, res) => {
  let query = 'SELECT * FROM users WHERE 1=1';
  const params = [];
  let paramIndex = 1;

  // Parse filters: age[gte]=18, status[eq]=active
  for (const [key, value] of Object.entries(req.query)) {
    const match = key.match(/^(\w+)\[(\w+)\]$/);
    if (!match) continue;

    const [, field, operator] = match;
    const allowedFields = ['age', 'status', 'name', 'created_at'];
    const allowedOps = ['eq', 'ne', 'gt', 'gte', 'lt', 'lte', 'co', 'sw', 'ew'];
    if (!allowedFields.includes(field) || !allowedOps.includes(operator)) continue;

    const opMap = { eq: '=', ne: '!=', gt: '>', gte: '>=', lt: '<', lte: '<=' };
    if (operator === 'co') {
      query += ` AND ${field} LIKE $${paramIndex}`;
      params.push(`%${value}%`);
    } else if (operator === 'sw') {
      query += ` AND ${field} LIKE $${paramIndex}`;
      params.push(`${value}%`);
    } else if (operator === 'ew') {
      query += ` AND ${field} LIKE $${paramIndex}`;
      params.push(`%${value}`);
    } else {
      query += ` AND ${field} ${opMap[operator]} $${paramIndex}`;
      params.push(value);
    }
    paramIndex++;
  }

  // Parse sorting: sort=name,-created_at
  if (req.query.sort) {
    const allowedSortFields = ['name', 'created_at', 'age'];
    const sortFields = req.query.sort.split(',').map(f => {
      const desc = f.startsWith('-');
      const field = desc ? f.slice(1) : f;
      return allowedSortFields.includes(field)
        ? `${field} ${desc ? 'DESC' : 'ASC'}`
        : null;
    }).filter(Boolean);

    if (sortFields.length > 0) {
      query += ` ORDER BY ${sortFields.join(', ')}`;
    }
  }

  query += ' LIMIT 50';
  const users = db.query(query, params);
  res.json({ data: users });
});
```

**Expected Output (for `GET /v1/users?age[gte]=18&status[eq]=active&sort=-created_at`):**
```sql
SELECT * FROM users WHERE 1=1 AND age >= 18 AND status = 'active' ORDER BY created_at DESC LIMIT 50
```

**Response:**
```json
{
  "data": [
    { "id": 3, "name": "Charlie", "age": 25, "status": "active", "created_at": "2026-01-03" },
    { "id": 1, "name": "Alice", "age": 30, "status": "active", "created_at": "2026-01-01" }
  ]
}
```

**Why this output:** The filter `age[gte]=18` translates to `age >= 18`. The filter `status[eq]=active` translates to `status = 'active'`. The sort `-created_at` translates to `ORDER BY created_at DESC`. The query returns only users matching the filters, sorted by creation date descending.

#### Real-World Cases

- **E-commerce:** `?price[gte]=50&price[lte]=200&category[eq]=electronics&sort=-rating`.
- **Admin panels:** `?status[eq]=pending&sort=created_at`.
- **Analytics:** `?created_at[gte]=2026-01-01&created_at[lte]=2026-12-31`.

---

### Sub-Feature 3.3: Full-Text Searching and Basic Pattern Matching

#### Definitions

**Core Definition:** Full-text search (FTS) allows natural language queries against text fields using indexed search engines; pattern matching uses `LIKE`/`ILIKE` operators for simple wildcard-based string matching.

**Technical Definition:** PostgreSQL full-text search uses `tsvector` (document representation) and `tsquery` (query representation) for efficient searching. PostgREST supports four FTS operators: `fts`, `plfts`, `phfts`, and `wfts`. Pattern matching uses the `LIKE` operator (case-sensitive) and `ILIKE` (case-insensitive) with `%` as a wildcard for multiple characters and `_` for a single character. To avoid URL encoding, `*` can be used as an alias for `%` in PostgREST.

**Beginner-Friendly Explanation:** Pattern matching is like using a wildcard in a search: "find all names starting with 'Al'" (`LIKE 'Al%'`). Full-text search is more advanced — it understands language: "find documents about database performance" returns documents containing those words, ranked by relevance, even if the exact phrase doesn't appear.

#### Purposes

- To enable users to search across text content.
- To provide relevant, ranked search results (FTS).
- To support autocomplete and prefix matching (pattern matching).
- To search structured and unstructured data.

#### Syntax Rules and Structure

**Pattern matching (LIKE/ILIKE):**
```
GET /v1/users?name=like.*Ali*
```
| Pattern | Meaning |
|---------|---------|
| `%` or `*` | Any sequence of characters. |
| `_` | Any single character. |
| `LIKE` | Case-sensitive. |
| `ILIKE` | Case-insensitive. |

**Full-text search (PostgreSQL):**
```sql
SELECT * FROM posts WHERE to_tsvector('english', content) @@ to_tsquery('english', 'database & performance');
```

**PostgREST FTS operators:**
| Operator | Description |
|----------|-------------|
| `fts` | Full-text search (AND by default). |
| `plfts` | Plain full-text search (natural language). |
| `phfts` | Phrase full-text search. |
| `wfts` | Web search (Google-like syntax). |

**Constraints and Limitations:**
- `LIKE` with leading wildcards (`%term`) cannot use standard indexes.
- Full-text search requires a `tsvector` column and a GIN/GiST index.
- PostgreSQL FTS is language-specific; you must specify the language configuration.
- Dedicated search engines (Elasticsearch, Typesense) offer more features (fuzzy matching, synonyms, ranking).

#### Annotated Code Example

```javascript
// Express: Full-text search with PostgreSQL
const { Pool } = require('pg');
const pool = new Pool();

app.get('/v1/posts/search', async (req, res) => {
  const { q, limit = 20 } = req.query;

  if (!q) {
    return res.status(400).json({ error: 'Query parameter q is required' });
  }

  // PostgreSQL full-text search with ranking
  const result = await pool.query(
    `SELECT id, title, content,
            ts_rank(to_tsvector('english', title || ' ' || content),
                    plainto_tsquery('english', $1)) AS rank
     FROM posts
     WHERE to_tsvector('english', title || ' ' || content) @@ plainto_tsquery('english', $1)
     ORDER BY rank DESC
     LIMIT $2`,
    [q, limit]
  );

  res.json({
    data: result.rows,
    meta: { query: q, total: result.rowCount },
  });
});

// Pattern matching with LIKE
app.get('/v1/users/search', async (req, res) => {
  const { name } = req.query;

  const result = await pool.query(
    `SELECT id, name, email FROM users WHERE name ILIKE $1 LIMIT 20`,
    [`%${name}%`] // Wildcard for contains
  );

  res.json({ data: result.rows });
});
```

**Expected Output (for `GET /v1/posts/search?q=database performance`):**
```json
{
  "data": [
    {
      "id": 42,
      "title": "Optimizing Database Performance",
      "content": "...",
      "rank": 0.075990885
    }
  ],
  "meta": { "query": "database performance", "total": 1 }
}
```

**Expected Output (for `GET /v1/users/search?name=Ali`):**
```json
{
  "data": [
    { "id": 1, "name": "Alice", "email": "alice@example.com" },
    { "id": 5, "name": "Alison", "email": "alison@example.com" }
  ]
}
```

**Why this output:** The full-text search uses `plainto_tsquery` to convert the query string into a `tsquery`, matches it against the `tsvector` of each post, and ranks the results by relevance (`ts_rank`). The pattern matching uses `ILIKE '%Ali%'` to find all names containing "Ali" (case-insensitive). Both approaches return matching records, but FTS provides relevance ranking while pattern matching provides simple substring matching.

#### Real-World Cases

- **Blog platforms:** Full-text search across articles with relevance ranking.
- **E-commerce:** Pattern matching for product name autocomplete.
- **Documentation sites:** Full-text search across documentation pages.
- **Log analysis:** Pattern matching for error codes in log messages.

---

## References

- RFC 7396 — JSON Merge Patch — https://datatracker.ietf.org/doc/html/rfc7396
- RFC 6902 — JavaScript Object Notation (JSON) Patch — https://www.rfc-editor.org/rfc/rfc6902
- RFC 5789 — PATCH Method for HTTP — https://www.rfc-editor.org/rfc/rfc5789
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- Microsoft Azure Architecture Center — Web API Design Best Practices — https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- JSON:API Specification — Sparse Fieldsets — https://jsonapi.org/format/#fetching-sparse-fieldsets
- Mews POS API — Sparse Fieldsets — https://docs.mews.com/pos-api/guidelines/sparse-fieldsets
- Apidog — Cursor-Based Pagination vs Offset Pagination — https://apidog.com/blog/cursor-vs-offset-pagination/
- Apidog — How Should You Design API Pagination for Millions of Records? — https://apidog.com/blog/api-pagination-guide/
- Bump.sh — Pagination — https://docs.bump.sh/openapi/v3.2/advanced/pagination/
- DreamFactory — How to Filter Events in REST APIs — https://blog.dreamfactory.com/how-to-filter-events-in-rest-apis
- PostgREST — Full-Text Search — https://postgrest.org/en/stable/references/api/tables_views.html#full-text-search
- PostgREST — Pattern Matching — https://postgrest.org/en/stable/references/api/tables_views.html#pattern-matching
- Typesense — Building a Search API with Node.js, Express, Drizzle ORM, and Typesense — https://typesense.org/docs/guide/
- ETSI — Filter Operators Table — https://www.etsi.org/
- Oracle — Query Parameters — https://docs.oracle.com/en-us/iaas/Content/Identity/api-getstarted/OCISQueryParameters.htm
- MDN Web Docs — HTTP Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods