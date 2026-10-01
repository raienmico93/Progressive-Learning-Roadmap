# Express.js Route Parameters — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Route parameters are named URL segments defined in a route path with a colon prefix (`:paramName`), whose values are captured from the incoming request URL and made available as properties on the `req.params` object.

**Technical Definition:** Express uses the `path-to-regexp` library to compile route path strings containing colon-prefixed placeholders into regular expressions. When an incoming request URL matches the compiled pattern, the captured values for each named placeholder are stored in the `req.params` object, keyed by the parameter name. For unnamed wildcard matches or regular expression routes, captured groups are stored by numeric index (`req.params[0]`, `req.params[1]`, etc.). Express automatically decodes the values in `req.params` using `decodeURIComponent`.

**Beginner-Friendly Explanation:** Route parameters are like fill-in-the-blank templates for URLs. When you define a route as `/users/:userId`, the `:userId` part is a blank that Express will fill in with whatever value appears in that position when a request arrives. If someone visits `/users/42`, Express tells your code: "The userId is 42." You access it as `req.params.userId`.

### Key Characteristics

- **Named segments:** Parameters are defined with a colon prefix (`:name`) and mapped to keys in `req.params`.
- **Always strings:** Captured values are always strings; type conversion is the developer's responsibility.
- **Auto-decoding:** Express automatically decodes URI-encoded values via `decodeURIComponent`.
- **Positional capture:** Parameters capture URL segments based on their position in the route path.
- **Regex constraints:** Parameters can be constrained with inline regular expressions or validated via `app.param()` middleware.
- **Optional support:** Parameters can be made optional using syntax that differs between Express 4 and Express 5.
- **Wildcard parameters:** Unnamed wildcards (`*`) and named splats (`/*splat`) capture multiple path segments.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, objects, and asynchronous code.
- **Understanding of HTTP routing:** Methods, paths, and handlers.
- **Familiarity with regular expressions** (for validation sections).

### Related Programming Areas

- **RESTful API design:** Route parameters identify specific resources (`/users/42`, `/posts/7`).
- **Middleware:** `app.param()` is a specialised middleware for parameter lifecycle hooks.
- **Validation libraries:** `express-validator`, `joi`, or `zod` for complex validation.
- **Database queries:** Parameters are typically used to look up records by ID.
- **Router modularity:** Parameters work identically in `express.Router()` instances.

### Core Concepts

1. **`req.params`** — the dictionary object auto-populated by Express.
2. **Dynamic Route Segments** — the colon syntax for defining structural variables.
3. **Multiple Parameters** — managing multi-tier dynamic relationships.
4. **Parameter Validation** — regex constraints and `app.param()` middleware.
5. **Optional Parameters** — Express 4 vs. Express 5 syntax variations.

---

## Core Concept 1: `req.params`

### Definitions

**Core Definition:** `req.params` is an object containing properties mapped to the named route parameters defined in the route path.

**Technical Definition:** `req.params` is a plain object whose keys correspond to the parameter names declared in the route path. For a route `/user/:name`, the property `req.params.name` contains the captured value. The object defaults to `{}`. When a regular expression is used for the route definition, capture groups are provided in the array using `req.params[n]`, where `n` is the nth capture group. This rule also applies to unnamed wildcard matches with string routes such as `/file/*`.

**Beginner-Friendly Explanation:** `req.params` is like a collection of labelled boxes. When a request comes in that matches a route with parameters, Express fills each box with the value from the corresponding URL segment. Your handler can then open any box by its label (`req.params.name`) to see what's inside.

### Purposes

- To capture URL segment state and make it available to the route handler.
- To identify specific resources in RESTful URLs without using query strings.
- To decouple route definitions from specific resource identifiers.
- To provide a consistent interface for accessing dynamic URL values.

### Syntax Rules and Structure

```js
// Route definition
app.get('/user/:name', (req, res) => {
  const name = req.params.name;   // Access the captured value
  res.send(`User: ${name}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `:name` | Parameter name in the route path. |
| `req.params` | Object containing all captured parameters. |
| `req.params.name` | The value captured for the `:name` parameter (always a string). |

**Rules:**
- The key in `req.params` matches the parameter name exactly (without the colon).
- Values are **always strings**, even if they look like numbers.
- Express automatically decodes URI-encoded values using `decodeURIComponent`.
- If you need to change a key in `req.params`, use the `app.param()` handler. Changes made directly to `req.params` in a middleware or route handler are reset.

**Constraints and Limitations:**
- Parameter names must be valid JavaScript identifiers (letters, digits, underscores; cannot start with a digit).
- The order of properties in `req.params` matches the order of parameters in the route path.
- Unmatched optional parameters are omitted from `req.params` entirely in Express 5 (in Express 4, they were set to `undefined`).

### Annotated Code Examples

#### Example 1: Single Parameter Access

```js
// req-params-single.js
const express = require('express');
const app = express();

app.get('/user/:name', (req, res) => {
  // req.params.name contains the captured URL segment
  console.log('Full params object:', req.params);  // { name: 'tj' }
  console.log('Name property:', req.params.name);   // "tj"
  res.send(`User: ${req.params.name}`);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /user/tj`):**
```
Full params object: { name: 'tj' }
Name property: tj
User: tj
```

**Why this output:** The route path `/user/:name` defines a parameter `name`. When the request URL `/user/tj` arrives, Express captures the segment `tj` and stores it in `req.params.name`. The handler reads this value and sends it in the response. Note that `req.params` is a plain object with exactly one key.

#### Example 2: Regular Expression Capture Groups

```js
// req-params-regex.js
const express = require('express');
const app = express();

app.get(/^\/file\/(.*)$/, (req, res) => {
  // Unnamed capture groups are stored by numeric index
  console.log('Params array-like:', req.params);    // { '0': 'path/to/file.txt' }
  console.log('First capture:', req.params[0]);      // "path/to/file.txt"
  res.send(`File path: ${req.params[0]}`);
});

app.listen(3000, () => console.log('Regex params on 3000'));
```

**Expected Output (for `GET /file/path/to/file.txt`):**
```
Params array-like: { '0': 'path/to/file.txt' }
First capture: path/to/file.txt
File path: path/to/file.txt
```

**Why this output:** When a regular expression is used as the route path, Express stores captured groups by numeric index (`req.params[0]`, `req.params[1]`, etc.) rather than by name. The first (and only) capture group contains the full path after `/file/`.

#### Example 3: Decoding and String Nature

```js
// req-params-decode.js
const express = require('express');
const app = express();

app.get('/search/:query', (req, res) => {
  console.log('Raw:', req.params.query);           // Decoded value
  console.log('Type:', typeof req.params.query);   // "string"
  res.json({ query: req.params.query });
});

app.listen(3000, () => console.log('Decode server on 3000'));
```

**Expected Output (for `GET /search/hello%20world`):**
```
Raw: hello world
Type: string
{"query":"hello world"}
```

**Why this output:** Express automatically decodes URI-encoded values. The `%20` in the URL is decoded to a space in `req.params.query`. The value is always a string, never a number or other type, even if it looks numeric.

### Real-World Cases

- **User profiles:** `/users/:username` to display a user's public profile.
- **Blog posts:** `/posts/:slug` to retrieve a post by its URL-friendly identifier.
- **File downloads:** `/download/:filename` to serve a specific file.
- **API resources:** `/api/products/:productId` to fetch product details.

---

## Core Concept 2: Dynamic Route Segments

### Definitions

**Core Definition:** A dynamic route segment is a placeholder in a route path defined with a colon (`:`) prefix that matches any value in that URL position.

**Technical Definition:** In the route path string, a colon followed by a valid JavaScript identifier creates a named parameter. Express's `path-to-regexp` compiler converts this into a regular expression that captures the corresponding URL segment. The captured value becomes a property of `req.params`.

**Beginner-Friendly Explanation:** A dynamic route segment is like a blank in a form. Instead of hardcoding `/users/42` to handle only user 42, you write `/users/:id` to handle any user ID. The `:id` is the blank, and Express fills it with whatever value appears in the URL.

### Purposes

- To define structural variables in route paths that match multiple URLs.
- To create reusable route definitions that work for any resource identifier.
- To enable RESTful URL design where resources are identified by their position in the hierarchy.
- To avoid duplicating route definitions for each possible resource ID.

### Syntax Rules and Structure

```js
app.get('/users/:userId', handler);
app.get('/posts/:postId/comments/:commentId', handler);
```

| Component | Breakdown |
|-----------|-----------|
| `:` | Prefix indicating a named parameter. |
| `userId` | Parameter name (valid JavaScript identifier). |
| `/users/` | Static prefix (must match exactly). |

**Rules:**
- The colon must immediately precede the parameter name with no space.
- Parameter names must be valid JavaScript identifiers: letters, digits, underscores; cannot start with a digit.
- A parameter captures exactly **one** URL segment (i.e., it stops at the next `/`).
- To capture multiple segments, use a wildcard (`*`) or a regular expression.
- Parameters can be placed anywhere in the path: beginning, middle, or end.

**Constraints and Limitations:**
- Parameter values are always strings; no automatic type conversion occurs.
- A route parameter cannot contain a `/` by default; use a regex pattern to allow slashes.
- Route paths are matched in definition order; the first matching route wins.

### Annotated Code Example

```js
// dynamic-segments.js
const express = require('express');
const app = express();

// Parameter at the end of the path
app.get('/users/:userId', (req, res) => {
  res.send(`User ID: ${req.params.userId}`);
});

// Parameter in the middle of the path
app.get('/posts/:postId/comments', (req, res) => {
  res.send(`Comments for post: ${req.params.postId}`);
});

// Multiple segments with different names
app.get('/api/:version/:resource', (req, res) => {
  res.json({
    version: req.params.version,
    resource: req.params.resource
  });
});

app.listen(3000, () => console.log('Dynamic segments on 3000'));
```

**Expected Output (for `GET /users/42`):**
```
User ID: 42
```

**Expected Output (for `GET /posts/7/comments`):**
```
Comments for post: 7
```

**Expected Output (for `GET /api/v2/users`):**
```
{"version":"v2","resource":"users"}
```

**Why this output:** Each colon-prefixed segment captures the corresponding URL segment. In `/api/:version/:resource`, the first parameter captures `v2` and the second captures `users`. The handler maps them to JSON properties. Express stops each parameter at the next `/`, so `:version` captures only `v2` and not `v2/users`.

### Real-World Cases

- **E-commerce categories:** `/products/:category/:productId` to identify a product within a category.
- **Social media:** `/users/:username/posts/:postId` to identify a specific post by a user.
- **API versioning:** `/api/:version/users` to support multiple API versions.
- **File browsers:** `/files/:folder/:filename` to navigate a virtual file system.

---

## Core Concept 3: Multiple Parameters

### Definitions

**Core Definition:** Multiple route parameters allow a single route to capture several dynamic URL segments, each stored under its own key in `req.params`.

**Technical Definition:** A route path can contain any number of colon-prefixed parameters, each capturing one URL segment. The parameters are mapped to `req.params` keys in the order they appear in the path. This enables modelling hierarchical relationships such as `/users/:userId/books/:bookId`.

**Beginner-Friendly Explanation:** Multiple parameters are like a form with several blanks. A URL like `/users/42/books/7` fills in two blanks: one for the user ID (42) and one for the book ID (7). Your handler can read both values from `req.params`.

### Purposes

- To manage multi-tier dynamic relationships in a single route.
- To model hierarchical resource paths (user → book, category → product, etc.).
- To avoid nested route definitions for related resources.
- To capture composite identifiers from a single URL.

### Syntax Rules and Structure

```js
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  res.json({ userId, bookId });
});
```

| Component | Breakdown |
|-----------|-----------|
| `:userId` | First parameter (captures `42`). |
| `:bookId` | Second parameter (captures `7`). |
| `req.params` | `{ userId: '42', bookId: '7' }` |

**Rules:**
- Parameters are matched in the order they appear in the path.
- Static segments between parameters must match exactly.
- Each parameter captures exactly one URL segment.
- Parameter names must be unique within a single route path.

**Constraints and Limitations:**
- Ambiguous routes can cause unexpected matches (e.g., `/users/:id` vs. `/users/:name`).
- Route paths are matched in definition order; place more specific routes first.
- Values are always strings; convert types explicitly.

### Annotated Code Example

```js
// multiple-params.js
const express = require('express');
const app = express();

// Two parameters in a hierarchical relationship
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  console.log('Full params:', req.params);
  res.json({
    userId,
    bookId,
    message: `Book ${bookId} belongs to user ${userId}`
  });
});

// Three parameters for deeper hierarchy
app.get('/api/:version/users/:userId/posts/:postId', (req, res) => {
  const { version, userId, postId } = req.params;
  res.json({ version, userId, postId });
});

// Numeric conversion example
app.get('/orders/:orderId/items/:itemId', (req, res) => {
  const orderId = parseInt(req.params.orderId, 10);
  const itemId = parseInt(req.params.itemId, 10);
  res.json({ orderId, itemId, sum: orderId + itemId });
});

app.listen(3000, () => console.log('Multiple params on 3000'));
```

**Expected Output (for `GET /users/42/books/7`):**
```
Full params: { userId: '42', bookId: '7' }
{"userId":"42","bookId":"7","message":"Book 7 belongs to user 42"}
```

**Expected Output (for `GET /orders/100/items/5`):**
```
{"orderId":100,"itemId":5,"sum":105}
```

**Why this output:** In the first example, both parameters are captured as strings. In the second example, `parseInt` converts them to numbers, and the sum is calculated. The `req.params` object contains exactly the keys defined in the route path, in the order they appear.

### Real-World Cases

- **Library management:** `/libraries/:libraryId/books/:bookId` to identify a book within a library.
- **Project management:** `/projects/:projectId/tasks/:taskId` to identify a task within a project.
- **E-commerce orders:** `/orders/:orderId/items/:itemId` to identify an item within an order.
- **Versioned APIs:** `/api/:version/users/:userId` to identify a user within a specific API version.

---

## Core Concept 4: Parameter Validation

### Definitions

**Core Definition:** Parameter validation ensures that captured URL parameters conform to expected types or formats before the route handler processes them.

**Technical Definition:** Express supports two primary validation mechanisms: inline regular expression constraints within the parameter syntax (e.g., `:id(\\d+)`) and `app.param()` middleware, which registers a callback that runs before any route handler using that parameter name. The callback receives the parameter value and can validate, transform, or reject it.

**Beginner-Friendly Explanation:** Parameter validation is like a bouncer at a club door. Before the request reaches the handler, the bouncer checks the ID (the parameter value) to make sure it's the right type. If the ID is invalid (e.g., "abc" instead of a number), the request is turned away.

### Purposes

- To validate parameter types (e.g., forcing `:id` to be a UUID or an integer).
- To trigger lifecycle validation on specific parameter names using `app.param()`.
- To reject invalid requests before they reach route handlers, reducing error-handling code.
- To transform parameter values (e.g., string to integer) automatically.

### Sub-Feature 4.1: Inline Regular Expression Constraints

#### Syntax Rules and Structure

```js
app.get('/items/:id(\\d+)', handler);
app.get('/users/:userId([a-f0-9]{24})', handler);
app.get('/products/:code([A-Z]{3}-\\d{4})', handler);
```

| Component | Breakdown |
|-----------|-----------|
| `:id(\\d+)` | Parameter `id` constrained to one or more digits. |
| `:userId([a-f0-9]{24})` | Parameter constrained to a 24-character hex string. |
| `:code([A-Z]{3}-\\d{4})` | Parameter constrained to a product code format. |

**Rules:**
- The regex pattern is placed in parentheses immediately after the parameter name.
- The pattern must be escaped in JavaScript strings (`\\d` for `\d`).
- If the URL segment does not match the pattern, the route does not match.
- In Express 5, inline regex in string paths is supported but `path-to-regexp@8` requires careful escaping.

#### Annotated Code Example

```js
// regex-validation.js
const express = require('express');
const app = express();

// Only match numeric IDs
app.get('/items/:id(\\d+)', (req, res) => {
  res.send(`Item ID (number): ${req.params.id}`);
});

// Only match UUID format
app.get('/users/:userId([a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12})', (req, res) => {
  res.send(`User UUID: ${req.params.userId}`);
});

// Fallback for non-numeric IDs
app.get('/items/:id', (req, res) => {
  res.status(400).send('Invalid item ID: must be numeric');
});

app.listen(3000, () => console.log('Regex validation on 3000'));
```

**Expected Output (for `GET /items/42`):**
```
Item ID (number): 42
```

**Expected Output (for `GET /items/abc`):**
```
Invalid item ID: must be numeric
```

**Why this output:** The route `/items/:id(\\d+)` only matches URLs where the `id` segment consists of digits. `abc` fails the regex test, so Express falls through to the next matching route (`/items/:id`), which returns a 400 error. This demonstrates how inline regex constraints can route invalid requests to dedicated error handlers.

#### Constraints and Limitations

- Inline regex patterns are compiled by `path-to-regexp`; not all regex features are supported.
- Complex regex patterns can become difficult to read and maintain.
- In Express 5, unnamed wildcards must be replaced with named splats or `(.*)`.

---

### Sub-Feature 4.2: `app.param()` Middleware

#### Definitions

**Core Definition:** `app.param()` registers a callback function that is triggered whenever a route parameter with the specified name is present in a matched route.

**Technical Definition:** `app.param(name, callback)` maps the given parameter name(s) to the given callback(s). The callback uses the same signature as middleware: `(req, res, next, value)`, where `value` is the captured parameter value. Once `next()` is invoked, execution continues to the route handler or subsequent parameter functions. This is useful for providing pre-conditions to routes that use normalised placeholders, such as automatically loading a user's information from a database.

**Beginner-Friendly Explanation:** `app.param()` is like a pre-flight checklist for routes. When a route uses a parameter named `:userId`, Express automatically runs your `app.param('userId', ...)` function before the route handler. You can use this to load data, validate the value, or convert types — all in one place, without repeating code in every route.

#### Syntax Rules and Structure

```js
app.param('userId', (req, res, next, value) => {
  // Validate or transform the parameter value
  if (!/^\d+$/.test(value)) {
    return res.status(400).send('Invalid user ID');
  }
  req.userId = parseInt(value, 10);
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `'userId'` | The parameter name to attach the callback to. |
| `req` | The request object. |
| `res` | The response object. |
| `next` | Passes control to the next parameter callback or route handler. |
| `value` | The captured parameter value (always a string). |

**Rules:**
- The callback runs **before** any route handler that uses the named parameter.
- Call `next()` to proceed; call `next('route')` to skip the current route.
- Multiple `app.param()` callbacks for the same name execute in registration order.
- In Express 5, the leading colon in the parameter name is silently ignored (`app.param(':user', ...)` is equivalent to `app.param('user', ...)`). The `app.param(fn)` signature is no longer supported.

**Constraints and Limitations:**
- `app.param()` callbacks are only triggered for routes that define the parameter.
- The callback is not triggered for parameters that are not present in the matched route.
- Changes made to `req.params` inside the callback are reset after the callback runs.

#### Annotated Code Example

```js
// app-param-middleware.js
const express = require('express');
const app = express();

// Simulated user database
const users = {
  1: { id: 1, name: 'Alice' },
  2: { id: 2, name: 'Bob' }
};

// app.param() middleware for :userId
app.param('userId', (req, res, next, value) => {
  const id = parseInt(value, 10);

  // Validate: must be a positive integer
  if (!Number.isInteger(id) || id <= 0) {
    return res.status(400).json({ error: 'User ID must be a positive integer' });
  }

  // Load user from database
  const user = users[id];
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  // Attach to request object for use in route handlers
  req.user = user;
  next();
});

// Routes using :userId automatically trigger the middleware
app.get('/users/:userId', (req, res) => {
  res.json(req.user);  // req.user is pre-loaded
});

app.get('/users/:userId/posts', (req, res) => {
  res.json({ user: req.user.name, posts: [] });
});

// Routes without :userId are unaffected
app.get('/users', (req, res) => {
  res.json(Object.values(users));
});

app.listen(3000, () => console.log('app.param() on 3000'));
```

**Expected Output (for `GET /users/1`):**
```
{"id":1,"name":"Alice"}
```

**Expected Output (for `GET /users/99`):**
```
{"error":"User not found"}
```

**Expected Output (for `GET /users/abc`):**
```
{"error":"User ID must be a positive integer"}
```

**Why this output:** The `app.param('userId', ...)` callback runs before every route that includes `:userId`. It validates the value, loads the user from the simulated database, and attaches it to `req.user`. If validation fails, it sends an error response without calling `next()`. The route handler simply returns `req.user`, which is already populated. The `/users` route (without `:userId`) is not affected by the middleware.

#### Real-World Cases

- **Database pre-loading:** Loading a user, post, or product by ID before the route handler runs.
- **Type coercion:** Converting parameter strings to integers, UUIDs, or other types.
- **Access control:** Checking if the requesting user has permission to access the resource identified by the parameter.
- **Slug resolution:** Resolving a URL-friendly slug to a database ID.

---

## Core Concept 5: Optional Parameters

### Definitions

**Core Definition:** Optional parameters allow a URL segment to be omitted from the request without causing the route to fail to match.

**Technical Definition:** Express supports optional parameters through different syntaxes depending on the major version. Express 4 uses the `?` suffix after the parameter name (e.g., `:id?`). Express 5 uses curly braces to denote optional path segments (e.g., `{/:id}`), as the `?` suffix is no longer supported by `path-to-regexp@8`.

**Beginner-Friendly Explanation:** An optional parameter is like a bonus question on a form. The route `/users/:id?` matches both `/users` (no ID) and `/users/42` (with ID). The handler must check whether the parameter was provided.

### Purposes

- To define routes that work with or without a specific URL segment.
- To support both list and detail views with a single route definition.
- To maintain backward compatibility with URLs that may or may not include an optional identifier.
- To simplify route definitions for resources with optional filters.

### Sub-Feature 5.1: Express 4 Syntax — `:param?`

#### Syntax Rules and Structure

```js
app.get('/user/:id?', (req, res) => {
  const id = req.params.id || 'No ID';
  res.send(`User: ${id}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `:id?` | Optional parameter; matches with or without the segment. |
| `req.params.id` | `undefined` if the segment is omitted. |

**Rules:**
- The `?` suffix makes the preceding parameter optional.
- Multiple optional parameters can be chained: `/users/:id?/:action?`.
- If omitted, `req.params.id` is `undefined`.

**Constraints:**
- **Deprecated in Express 5:** The `?` suffix throws a `PathError` with `path-to-regexp@8`.
- Optional parameters at the beginning of a path may not work as expected.

---

### Sub-Feature 5.2: Express 5 Syntax — `{/:param}`

#### Syntax Rules and Structure

```js
app.get('/user{/:id}', (req, res) => {
  const id = req.params.id || 'No ID';
  res.send(`User: ${id}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `{/:id}` | Optional path segment; the slash is inside the braces. |
| `req.params.id` | **Omitted** entirely if the segment is absent. |

**Rules:**
- Braces `{ }` define optional parts of the path.
- The slash must be **inside** the braces (`{/:id}`, not `/:id`).
- Multiple optional segments: `/articles{/:year}{/:month}{/:day}`.
- If omitted, the parameter key is absent from `req.params` (not set to `undefined`).

**Constraints:**
- The optional parameter must be placed **after** a static segment; placing it at the beginning of the path may fail to match.
- Express 5 requires Node.js 18 or higher.

### Annotated Code Examples

#### Example 1: Express 4 — Optional Parameter

```js
// optional-express4.js
const express = require('express');
const app = express();

app.get('/user/:id?', (req, res) => {
  const id = req.params.id;
  console.log('Params:', req.params);        // { id: undefined } or { id: '42' }
  res.send(id ? `User ID: ${id}` : 'All users');
});

app.listen(3000, () => console.log('Express 4 optional on 3000'));
```

**Expected Output (for `GET /user`):**
```
Params: { id: undefined }
All users
```

**Expected Output (for `GET /user/42`):**
```
Params: { id: '42' }
User ID: 42
```

**Why this output:** In Express 4, the optional parameter `:id?` matches both paths. When omitted, `req.params.id` is `undefined`. The handler uses the `||` operator to provide a fallback message.

---

#### Example 2: Express 5 — Optional Parameter

```js
// optional-express5.js
const express = require('express');
const app = express();

app.get('/user{/:id}', (req, res) => {
  console.log('Params:', req.params);        // {} or { id: '42' }
  const id = req.params.id;
  res.send(id ? `User ID: ${id}` : 'All users');
});

// Multiple optional segments
app.get('/articles{/:year}{/:month}{/:day}', (req, res) => {
  const { year, month, day } = req.params;
  res.json({
    year: year || 'all',
    month: month || 'all',
    day: day || 'all'
  });
});

app.listen(3000, () => console.log('Express 5 optional on 3000'));
```

**Expected Output (for `GET /user`):**
```
Params: {}
All users
```

**Expected Output (for `GET /user/42`):**
```
Params: { id: '42' }
User ID: 42
```

**Expected Output (for `GET /articles/2026/01`):**
```
{"year":"2026","month":"01","day":"all"}
```

**Why this output:** In Express 5, the optional segment `{/:id}` is entirely omitted from the matched path when absent, so `req.params` is an empty object (`{}`) rather than `{ id: undefined }`. For multiple optional segments, each `{/:param}` independently matches or is omitted, allowing flexible URL structures.

### Real-World Cases

- **Pagination:** `/users{/:page}` to support both `/users` and `/users/2`.
- **Date archives:** `/posts{/:year}{/:month}` to browse posts by year and month.
- **API compatibility:** Supporting both `/api/users` and `/api/users/42` with one route.
- **Localisation:** `/products{/:locale}` to optionally specify a locale.

---

## References

- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Express.js 5.x API — `req.params` — https://expressjs.com/en/5x/api.html#req.params
- Express.js 5.x API — `app.param()` — https://expressjs.com/en/5x/api.html#app.param
- Express.js Migrating to 5 — https://expressjs.com/en/guide/migrating-5.html
- Express.js 4.x API — `app.param()` — https://expressjs.com/en/4x/api.html#app.param
- path-to-regexp Documentation — https://github.com/pillarjs/path-to-regexp
- path-to-regexp — Optional Parameters — https://www.npmjs.com/package/path-to-regexp#optional
- Express 5 Release Blog — https://expressjs.com/en/blog/2024-10-15-v5-release
- MDN — Regular Expressions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions
- MDN — Named Capturing Groups — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Named_capturing_group