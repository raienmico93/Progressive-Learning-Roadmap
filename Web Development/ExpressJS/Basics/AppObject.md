# Express Application Object

The `app` object is the heart of every Express application. It is created by calling the top-level `express()` function, and it provides the methods for routing, configuring middleware, rendering views, and starting the server.

---

## 1. `express()` — The Top-Level Factory Function

### 1.1 What It Is

`express()` is the **top-level factory function** exported by the Express module. Calling it creates and returns a new **application instance** — the `app` object.

```javascript
import express from 'express';

const app = express();
```

> "The `app` object conventionally denotes the Express application. Create it by calling the top-level `express()` function exported by the Express module."

The `express()` function is a **factory function** — it is not a constructor, so `new` is not used. It creates an object when called and returns that object, which is also a function with properties.

### 1.2 The App Object Is a Function

The returned `app` is a **JavaScript function** designed to be passed to Node's HTTP servers as a callback to handle requests.

> "The `app` returned by `express()` is in fact a JavaScript Function, designed to be passed to Node's HTTP servers as a callback to handle requests."

This is why you can write:

```javascript
import http from 'http';
import app from './app.js';

http.createServer(app).listen(3000);
```

The `app` function is called with `(req, res)` for every incoming request.

### 1.3 Sub-Utilities Attached to `express()`

The `express` object itself carries **built-in middleware functions** and utilities:

| Utility | Purpose |
|---|---|
| `express.json()` | Parse JSON request bodies |
| `express.urlencoded()` | Parse URL-encoded form bodies |
| `express.static()` | Serve static files |
| `express.raw()` | Parse bodies as `Buffer` |
| `express.text()` | Parse bodies as strings |
| `express.Router()` | Create a router instance |

**Example — `express.json()`:**

> "This is a built-in middleware function in Express. It parses incoming requests with JSON payloads and is based on `body-parser`. Returns middleware that only parses JSON and only looks at requests where the `Content-Type` header matches the `type` option."

```javascript
app.use(express.json());
```

This middleware populates `req.body` with the parsed JSON object.

### 1.4 Factory Function Mechanics

```javascript
const app1 = express();   // new instance
const app2 = express();   // another new instance

app1 === app2;            // false
```

Each call to `express()` produces an **independent application**. However, when you export a single `app` from a module, all importers share that instance because of Node's module caching.

```javascript
// app.js
const app = express();
export default app;       // single instance, shared by all importers
```

---

## 2. `app.listen()` — Starting the HTTP Server

### 2.1 Signature

```javascript
app.listen([port[, host[, backlog]]][, callback])
app.listen(path[, callback])   // UNIX socket
```

> "Binds and listens for connections on the specified host and port. This method is identical to Node's `http.Server.listen()`."

### 2.2 Creating a Native HTTP Server Behind the Scenes

`app.listen()` is a **convenience wrapper** around `http.createServer()`:

> "The `app.listen()` method returns an `http.Server` object and (for HTTP) is a convenience method for the following: `app.listen = function() { var server = http.createServer(this); return server.listen.apply(server, arguments); }`"

**Equivalent long form:**

```javascript
import http from 'http';
import app from './app.js';

const server = http.createServer(app);
server.listen(3000, () => {
  console.log('Listening on port 3000');
});
```

**Using `app.listen()` directly:**

```javascript
const server = app.listen(3000, () => {
  console.log('Listening on port 3000');
});
```

Both return the same `http.Server` instance.

### 2.3 Passing Configuration Parameters

| Parameter | Description |
|---|---|
| `port` | TCP port (0 = OS assigns a free port) |
| `host` | Interface to bind (`127.0.0.1`, `0.0.0.0`, IP) |
| `backlog` | Max queued connections |
| `callback` | Called once when the server is bound |

```javascript
app.listen(3000, '127.0.0.1', 511, () => {
  console.log('Server ready');
});
```

> "If port is omitted or is 0, the operating system will assign an arbitrary unused port, which is useful for cases like automated tasks (tests, etc.)."

### 2.4 The Returned `http.Server`

```javascript
const server = app.listen(3000);

server.on('error', (err) => console.error(err));
server.on('listening', () => console.log('Ready'));
server.close(() => console.log('Closed'));
```

### 2.5 Separate HTTP and HTTPS

Because `app` is just a callback, the same app can serve both:

```javascript
import http from 'http';
import https from 'https';
import fs from 'fs';
import app from './app.js';

http.createServer(app).listen(80);

https.createServer({
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
}, app).listen(443);
```

> "This makes it easy to provide both HTTP and HTTPS versions of your app with the same code base, as the app does not inherit from these (it is simply a callback)."

---

## 3. `app.use()` — Registering Application-Wide Middleware

### 3.1 Purpose

`app.use()` **mounts middleware** at a path (or globally). It runs for **all HTTP methods** on paths that **begin with** the mount path.

> "Bind application-level middleware to an instance of the `app` object by using the `app.use()` and `app.METHOD()` functions."

### 3.2 Signature

```javascript
app.use([path,] middleware1 [, middleware2, ...])
```

| Argument | Description |
|---|---|
| `path` | Optional mount path (defaults to `/`) |
| `middleware` | One or more `(req, res, next)` functions |

### 3.3 No Mount Path — Runs on Every Request

```javascript
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});
```

> "This example shows a middleware function with no mount path. The function is executed every time the app receives a request."

### 3.4 Mount Path Mechanics

The path in `app.use()` is a **mount path** (or **prefix**), **not** a route path.

> "The `path` is a '*mount*' or '*prefix*' path and limits the middleware to only apply to any paths requested that ***begin*** with it."

```javascript
app.use('/user/:id', (req, res, next) => {
  console.log('Request Type:', req.method);
  next();
});
```

> "The function is executed for any type of HTTP request on the `/user/:id` path."

**Key distinction:**

| Path | Match Behavior |
|---|---|
| `app.use('/api', mw)` | Matches `/api`, `/api/users`, `/api/users/1`, ... |
| `app.get('/api', handler)` | Matches only `GET /api` |

**Mount path `/` matches everything:**

> "By specifying `/` as a '*mount*' path, `app.use()` will respond to any path that starts with `/`, which are all of them and regardless of HTTP verb used."

### 3.5 Sub-Stacks at a Mount Point

Multiple middleware can be mounted together:

```javascript
app.use('/user/:id',
  (req, res, next) => {
    console.log('Request URL:', req.originalUrl);
    next();
  },
  (req, res, next) => {
    console.log('Request Type:', req.method);
    next();
  }
);
```

> "Here is an example of loading a series of middleware functions at a mount point, with a mount path. It illustrates a middleware sub-stack that prints request info for any type of HTTP request to the `/user/:id` path."

### 3.6 Common `app.use()` Patterns

```javascript
// Global body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Static files
app.use(express.static('public'));

// Security headers
app.use(helmet());

// CORS
app.use(cors());

// Logging
app.use(morgan('dev'));

// Request timing
app.use((req, res, next) => {
  req.startTime = Date.now();
  next();
});
```

---

## 4. `app.METHOD()` — Dynamic Route Generation

### 4.1 Purpose

`app.METHOD()` defines **routes** that respond to specific HTTP verbs.

> "Routing refers to how an application's endpoints (URIs) respond to client requests. Define routing using methods of the Express `app` object that correspond to HTTP methods; for example, `app.get()` to handle GET requests and `app.post` to handle POST requests."

### 4.2 Supported Methods

Express supports methods corresponding to **all HTTP request methods**:

| Method | HTTP Verb |
|---|---|
| `app.get()` | GET |
| `app.post()` | POST |
| `app.put()` | PUT |
| `app.patch()` | PATCH |
| `app.delete()` | DELETE |
| `app.head()` | HEAD |
| `app.options()` | OPTIONS |
| `app.all()` | All methods |

> "Express supports methods that correspond to all HTTP request methods: `get`, `post`, and so on."

### 4.3 Signature

```javascript
app.METHOD(path, [callback, ...] callback)
```

> "`METHOD` is an HTTP request method, in lowercase. `PATH` is a path on the server. `HANDLER` is the function executed when the route is matched."

### 4.4 Examples

```javascript
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

app.post('/', (req, res) => {
  res.send('POST request to homepage');
});

app.put('/users/:id', (req, res) => {
  res.send(`PUT for user ${req.params.id}`);
});

app.patch('/users/:id', (req, res) => {
  res.send(`PATCH for user ${req.params.id}`);
});

app.delete('/users/:id', (req, res) => {
  res.send(`DELETE for user ${req.params.id}`);
});
```

### 4.5 Multiple Callbacks

Route methods accept multiple handlers:

```javascript
app.get('/user/:id',
  (req, res, next) => {
    console.log('ID:', req.params.id);
    next();
  },
  (req, res, next) => {
    res.send('User Info');
  }
);
```

> "The example below defines two routes for GET requests to the `/user/:id` path. The second route will not cause any problems, but it will never get called because the first route ends the request-response cycle."

### 4.6 `next('route')`

Skip to the next route (only in `app.METHOD()` or `router.METHOD()` middleware):

```javascript
app.get('/user/:id',
  (req, res, next) => {
    if (req.params.id === '0') return next('route');
    next();
  },
  (req, res, next) => {
    res.send('Regular user');
  }
);

app.get('/user/:id', (req, res) => {
  res.send('Special case: id = 0');
});
```

> "To skip the rest of the middleware functions from a router middleware stack, call `next('route')` to pass control to the next route. Note `next('route')` will work only in middleware functions that were loaded by using the `app.METHOD()` or `router.METHOD()` functions."

### 4.7 `app.all()`

Handles **all** HTTP methods on a path:

```javascript
app.all('/secret', (req, res, next) => {
  console.log('Accessing the secret section...');
  next();
});
```

> "There's a special routing method, `app.all()`, used to load middleware functions at a path for **all** HTTP request methods."

---

## 5. Application Settings

### 5.1 `app.set()` and `app.get()`

Express provides methods for storing and retrieving application settings:

| Method | Purpose |
|---|---|
| `app.set(name, value)` | Assign setting `name` to `value` |
| `app.get(name)` | Retrieve the value of setting `name` |
| `app.enable(name)` | Set a boolean setting to `true` |
| `app.disable(name)` | Set a boolean setting to `false` |
| `app.enabled(name)` | Check if a setting is enabled |
| `app.disabled(name)` | Check if a setting is disabled |

> "Assigns setting `name` to `value`."

```javascript
app.set('title', 'My Site');
app.get('title');  // "My Site"
```

> "`app.set(key, value)` is the same as `app.locals.settings[key] = value`; the former is the preferred way of configuring certain parts of Express (like setting view engine)."

### 5.2 Built-In Application Settings

Express provides several predefined settings:

| Setting | Default | Purpose |
|---|---|---|
| `env` | `process.env.NODE_ENV` or `"development"` | Environment mode |
| `trust proxy` | `false` | Enables reverse proxy support |
| `x-powered-by` | `true` | Send `X-Powered-By: Express` header |
| `etag` | `"weak"` | ETag generation strategy |
| `query parser` | `"extended"` | Query string parser |
| `view engine` | — | Template engine (e.g., `ejs`, `pug`) |
| `views` | `./views` | Directory for templates |
| `jsonp callback name` | `"callback"` | JSONP callback parameter name |
| `json spaces` | 2 in development | JSON response formatting |

> "The following settings are provided to alter how Express will behave."

### 5.3 `trust proxy`

When running behind a reverse proxy (nginx, Heroku, AWS ALB), set `trust proxy`:

```javascript
app.set('trust proxy', true);
```

> "When running an Express app behind a proxy, set (by using `app.set()`) the application variable `trust proxy` to one of the values listed in the following table."

**Values accepted:**

| Value | Meaning |
|---|---|
| `true` | Trust all proxies |
| `false` | Trust none (default) |
| `'loopback'` | Trust loopback addresses |
| `'linklocal'` | Trust link-local addresses |
| `'uniquelocal'` | Trust unique-local addresses |
| IP / subnet | Trust specific addresses |
| Number | Trust `n` hops from the proxy |

**Effects of enabling `trust proxy`:**

- `req.ip` and `req.ips` reflect `X-Forwarded-For`
- `req.protocol` reflects `X-Forwarded-Proto`
- `req.secure` reflects the forwarded protocol
- `req.hostname` reflects `X-Forwarded-Host`

**Security warning:**

> "Never enable trust proxy unless your app is actually behind a proxy you trust — otherwise clients can spoof X-Forwarded-For."

### 5.4 `x-powered-by`

By default, Express sends the header `X-Powered-By: Express`. Disable it to avoid revealing the stack:

```javascript
app.disable('x-powered-by');
```

> "`x-powered-by` | `true` | Send `X-Powered-By: Express` header"

### 5.5 `env` and Environment Mode

```javascript
app.get('env');  // "development" by default
```

Set via `NODE_ENV`:

```bash
NODE_ENV=production node server.js
```

Express uses `env` to adjust behavior (e.g., error verbosity, caching).

### 5.6 Custom Settings

Any key-value pair can be stored:

```javascript
app.set('apiVersion', 'v2');
app.set('maxUploadSize', '10mb');

console.log(app.get('apiVersion'));  // "v2"
```

### 5.7 `app.locals` — Application-Level Template Variables

```javascript
app.locals.siteName = 'My Site';
app.locals.year = new Date().getFullYear();
```

> "The `app.locals` object has properties that are local variables within the application, and will be available in templates rendered with `res.render()`."

**Warning:** App locals persist for the life of the application and **should not contain user-controlled input**.

> "The `locals` object is used by view engines to render a response. The object keys may be particularly sensitive and should not contain user-controlled input, as it may affect the operation of the view engine or provide a path to cross-site scripting."

---

## 6. Application-Level Middleware

### 6.1 Definition

Application-level middleware is bound to the `app` instance via `app.use()` or `app.METHOD()`:

> "Bind application-level middleware to an instance of the `app` object by using the `app.use()` and `app.METHOD()` functions, where `METHOD` is the HTTP method of the request that the middleware function handles (such as GET, PUT, or POST) in lowercase."

### 6.2 Middleware Function Signature

```javascript
function middleware(req, res, next) {
  // 1. Execute any code
  // 2. Make changes to req and res
  // 3. End the request-response cycle
  // 4. Call next() to pass control
}
```

> "Middleware functions have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle. The next middleware function is commonly denoted by a variable named `next`."

### 6.3 The Critical Rule

> "If the current middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function. Otherwise, the request will be left hanging."

### 6.4 Intercepting Every Request

Middleware registered without a mount path runs on **every** incoming request, **before** route matching:

```javascript
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} ${req.method} ${req.url}`);
  next();
});
```

### 6.5 Order Matters

Middleware executes in the **order it is registered**. Global middleware should come before routes:

```javascript
// 1. Global middleware
app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(logger);

// 2. Routes
app.use('/api', apiRoutes);
app.get('/', homeHandler);

// 3. 404 handler
app.use((req, res) => {
  res.status(404).send('Not found');
});

// 4. Error handler (must be last)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke');
});
```

### 6.6 Complete Example — Global Middleware Pipeline

```javascript
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import morgan from 'morgan';

const app = express();

// ─── Global Middleware ────────────────────────────────────────
app.use(helmet());                              // security headers
app.use(cors());                                // CORS
app.use(morgan('dev'));                         // logging
app.use(express.json());                        // JSON body parsing
app.use(express.urlencoded({ extended: true })); // form parsing
app.use(express.static('public'));              // static files

// Request timing middleware
app.use((req, res, next) => {
  req.startTime = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.path} took ${Date.now() - req.startTime}ms`);
  });
  next();
});

// ─── Routes ───────────────────────────────────────────────────
app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.post('/echo', (req, res) => {
  res.json({ received: req.body });
});

// ─── 404 Handler ──────────────────────────────────────────────
app.use((req, res) => {
  res.status(404).json({ error: 'Not found' });
});

// ─── Error Handler ───────────────────────────────────────────
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

export default app;
```

### 6.7 Types of Middleware

| Type | Bound To | Example |
|---|---|---|
| **Application-level** | `app` | `app.use()`, `app.get()` |
| **Router-level** | `express.Router()` | `router.use()` |
| **Error-handling** | `app` or router | `(err, req, res, next)` |
| **Built-in** | Express | `express.json()`, `express.static()` |
| **Third-party** | Installed packages | `helmet()`, `cors()`, `morgan()` |

---

## Summary Table

| Method / Feature | Purpose |
|---|---|
| `express()` | Factory function; creates the app instance |
| `express.json()` | Built-in middleware for JSON bodies |
| `express.static()` | Built-in middleware for static files |
| `app.listen()` | Starts HTTP server; wraps `http.createServer(app)` |
| `app.use()` | Mounts application-level middleware |
| `app.METHOD()` | Defines routes for HTTP verbs |
| `app.all()` | Handles all HTTP methods on a path |
| `app.set()` / `app.get()` | Configure and retrieve settings |
| `app.enable()` / `app.disable()` | Toggle boolean settings |
| `app.locals` | Application-level template variables |
| `app.mountpath` | Mount path(s) of a sub-app |
| `app.router` | Built-in router instance (lazy) |

---

## Key Takeaways

1. **`express()`** is a top-level factory function that returns the `app` object — a function with properties, callable by Node's HTTP server.
2. **Sub-utilities** like `express.json()`, `express.static()`, and `express.Router()` are attached to the `express` object.
3. **`app.listen()`** creates a native `http.Server` behind the scenes and returns it — the app itself is just a callback.
4. **`app.use()`** registers middleware at a **mount path** (prefix), not a route path — it matches all HTTP methods on paths beginning with the mount.
5. **`app.METHOD()`** defines routes for specific HTTP verbs (`get`, `post`, `put`, `patch`, `delete`, `all`).
6. **Application settings** are configured via `app.set()` / `app.get()`; built-in settings include `env`, `trust proxy`, `x-powered-by`, and `view engine`.
7. **`trust proxy`** must be configured when running behind a reverse proxy — but only if the proxy is trusted.
8. **`x-powered-by`** should be disabled in production to avoid revealing the stack.
9. **Application-level middleware** intercepts every request **before** route matching — ideal for parsing, logging, and cross-cutting concerns.
10. **Middleware order matters** — global middleware first, routes next, 404 handler after, error handler last.
11. **`next()` is mandatory** — without it, the request hangs.

---

Would you like me to continue with the next topic — **Express Routing**, **Express Middleware in Depth**, or **Request and Response Objects**? I can format the next section in the same style.