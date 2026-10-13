# Fetch API — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Fetch API is a modern, Promise-based interface for making HTTP requests from JavaScript. It provides a more powerful and flexible alternative to the older `XMLHttpRequest` (XHR) object.

**Technical Definition:** Per MDN, the Fetch API provides an interface for fetching resources (including across the network). It will seem familiar to anyone who has used `XMLHttpRequest`, but the new API provides a more powerful and flexible feature set. The Fetch API is defined in the WHATWG Fetch Living Standard. It is built around three core interfaces: `Request` (represents a resource request), `Response` (represents the response to a request), and `Headers` (represents HTTP headers). The `fetch()` method is a global function available in both `Window` and `WorkerGlobalScope` contexts. The API uses Promises, making it composable with `async/await`. The Fetch API has become the standard for making HTTP requests in modern JavaScript, though it has notable differences from `XMLHttpRequest` — particularly in error handling (fetch only rejects on network failures, not HTTP errors) and CORS behaviour.

**Beginner-Friendly Explanation:** The Fetch API is how modern JavaScript talks to servers. Instead of the old, clunky `XMLHttpRequest`, you write `fetch(url)` and get back a Promise. It's clean, readable, and works beautifully with `async/await`. You can fetch data from APIs, send form data, upload files, and even stream responses. The only tricky part is that `fetch()` doesn't throw an error for HTTP error statuses like 404 or 500 — you have to check `response.ok` yourself.

### Key Characteristics

- **Promise-based:** `fetch()` returns a Promise that resolves to a `Response` object.
- **Global function:** `fetch()` is available globally in browsers and workers.
- **Streaming:** Response bodies are streams, allowing incremental processing.
- **Composable:** Works seamlessly with `async/await` and Promise combinators.
- **No HTTP error rejection:** `fetch()` only rejects on network failures (e.g., DNS errors, offline). HTTP errors (404, 500) resolve normally with `response.ok === false`.
- **CORS-aware:** Cross-origin requests require the server to include CORS headers.
- **Credentials control:** Cookies and authentication headers can be controlled via the `credentials` option.
- **Cancellable:** Requests can be aborted using `AbortController` and `AbortSignal`.
- **Not a drop-in XHR replacement:** Some XHR features (progress events, synchronous mode) are not available or differ in fetch.

### Prerequisites

- Understanding of JavaScript Promises and `async/await`.
- Familiarity with HTTP basics (methods, headers, status codes).
- Awareness of the same-origin policy and CORS.
- Basic knowledge of JSON and FormData.

### Related Programming Areas

- **REST APIs:** Fetching and sending data to RESTful services.
- **GraphQL:** Sending GraphQL queries via POST requests.
- **File uploads:** Sending files with `FormData`.
- **Streaming:** Processing large responses incrementally.
- **Service Workers:** Fetching resources in offline-first applications.
- **Progressive Web Apps:** Caching and offline support.

---

## `fetch()`

### Definitions

**Core Definition:** `fetch()` is a global function that starts the process of fetching a resource from the network, returning a Promise that resolves to a `Response` object.

**Technical Definition:** Per MDN, the global `fetch()` method starts the process of fetching a resource from the network, returning a promise that is fulfilled once the response is available. The promise resolves to the `Response` object representing the response to your request. A `fetch()` promise only rejects when the request fails, for example, because of a badly-formed request URL or a network error. A `fetch()` promise does not reject on HTTP errors (`404`, `500`, etc.) — instead, a `then()` handler must check the `Response.ok` and/or `Response.status` properties.

### Purposes

- To retrieve resources (data, files, APIs) from a URL.
- To send data to a server (POST, PUT, DELETE).
- To upload files and form data.
- To implement AJAX-driven web applications.
- To stream large responses incrementally.

### Syntax Rules and Structure

```js
fetch(resource)
fetch(resource, options)
```

**Component Breakdown:**
- `resource`: The URL or a `Request` object.
- `options` (optional): An object containing custom settings for the request.

### Basic Usage

```js
// Simple GET request:
fetch("https://api.example.com/users")
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error("Error:", error));

// With async/await:
async function getUsers() {
  try {
    const response = await fetch("https://api.example.com/users");
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error:", error);
  }
}
```

### Annotated Code Examples

#### Example 1: Basic GET Request

```js
// Step 1: Fetch data from an API.
async function fetchUser(id) {
  try {
    const response = await fetch(`https://api.example.com/users/${id}`);

    // Step 2: Check if the response is OK (status 200-299).
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    // Step 3: Parse the JSON body.
    const user = await response.json();
    return user;
  } catch (error) {
    console.error("Failed to fetch user:", error.message);
  }
}

fetchUser(1).then((user) => console.log(user));
```

**Expected Output:**
```
{ id: 1, name: 'Alice', email: 'alice@example.com' }
```

**Why This Output:** The fetch request is made, the response is checked for HTTP errors, and the JSON body is parsed.

#### Example 2: POST Request with JSON

```js
// Step 1: Define the data to send.
const newUser = { name: "Alice", email: "alice@example.com" };

// Step 2: Send a POST request.
async function createUser(user) {
  const response = await fetch("https://api.example.com/users", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(user),
  });

  if (!response.ok) {
    throw new Error(`HTTP error! Status: ${response.status}`);
  }

  return response.json();
}

createUser(newUser).then((created) => console.log(created));
```

**Expected Output:**
```
{ id: 2, name: 'Alice', email: 'alice@example.com' }
```

**Why This Output:** The `method: "POST"` option specifies the HTTP method. The `headers` option sets the content type. The `body` option contains the serialised JSON data.

### Real-World Cases

- **Data fetching:** Loading user data, posts, comments from REST APIs.
- **Authentication:** Sending login credentials and receiving tokens.
- **Search:** Sending search queries and receiving results.
- **File uploads:** Uploading images, documents, and other files.

---

## Request Object

### Definitions

**Core Definition:** The `Request` object represents a resource request, encapsulating the URL, method, headers, body, and other options.

**Technical Definition:** Per MDN, the `Request` interface of the Fetch API represents a resource request. You can create a new `Request` object using the `Request()` constructor, but you are more likely to encounter a `Request` object being returned as the result of another API operation, such as a service worker `FetchEvent.request`.

### Syntax Rules and Structure

```js
const request = new Request(url, options);
```

**Component Breakdown:**
- `url`: The URL of the resource.
- `options`: An object with properties like `method`, `headers`, `body`, `mode`, `credentials`, `cache`, `redirect`, `referrer`, `signal`, etc.

### Key Properties

| Property | Description |
|----------|-------------|
| `method` | The HTTP method (GET, POST, etc.) |
| `url` | The full URL |
| `headers` | The `Headers` object |
| `body` | The body (ReadableStream) |
| `bodyUsed` | Whether the body has been read |
| `credentials` | The credentials mode |
| `cache` | The cache mode |
| `mode` | The request mode (cors, no-cors, same-origin) |
| `signal` | The AbortSignal |

### Annotated Code Example

```js
// Step 1: Create a Request object.
const request = new Request("https://api.example.com/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice" }),
  credentials: "include",
  cache: "no-cache",
});

// Step 2: Use the Request object with fetch.
fetch(request)
  .then((response) => response.json())
  .then((data) => console.log(data));
```

**Expected Output:** The request is sent with the specified options, and the response is logged.

**Why This Output:** The `Request` object encapsulates all the request options, which can be reused or modified.

### Real-World Cases

- **Reusable requests:** Creating a Request object once and using it multiple times.
- **Service Workers:** Intercepting and modifying requests.
- **Request cloning:** Cloning requests for retry logic.

---

## Response Object

### Definitions

**Core Definition:** The `Response` object represents the response to a request, containing the status, headers, and body.

**Technical Definition:** Per MDN, the `Response` interface of the Fetch API represents the response to a request. You can create a new `Response` object using the `Response()` constructor, but you are more likely to encounter a `Response` object being returned as the result of another API operation, such as a service worker `FetchEvent.respondWith`.

### Key Properties

| Property | Description |
|----------|-------------|
| `ok` | `true` if status is 200–299 |
| `status` | The HTTP status code (e.g., 200, 404, 500) |
| `statusText` | The status message (e.g., "OK", "Not Found") |
| `headers` | The `Headers` object |
| `body` | The body (ReadableStream) |
| `bodyUsed` | Whether the body has been read |
| `redirected` | Whether the response was redirected |
| `type` | The response type (basic, cors, error, opaque) |
| `url` | The URL of the response |

### Key Methods

| Method | Description |
|--------|-------------|
| `json()` | Parses the body as JSON |
| `text()` | Parses the body as text |
| `blob()` | Parses the body as a Blob |
| `arrayBuffer()` | Parses the body as an ArrayBuffer |
| `formData()` | Parses the body as FormData |
| `clone()` | Clones the response (body can only be read once) |

### Annotated Code Example

```js
const response = await fetch("https://api.example.com/users");

// Step 1: Check status.
console.log(response.ok);         // true
console.log(response.status);     // 200
console.log(response.statusText); // "OK"

// Step 2: Read headers.
console.log(response.headers.get("Content-Type")); // "application/json"

// Step 3: Parse the body.
const data = await response.json();
console.log(data);
```

**Expected Output:**
```
true
200
OK
application/json
[{ id: 1, name: 'Alice' }]
```

**Why This Output:** The `Response` object provides access to status, headers, and body. The body can only be read once, so `clone()` is needed if multiple reads are required.

### Real-World Cases

- **Status checking:** Using `response.ok` and `response.status` to handle errors.
- **Header inspection:** Reading `Content-Type`, `Cache-Control`, etc.
- **Body parsing:** Using `json()`, `text()`, or `blob()` based on content type.

---

## Headers

### Definitions

**Core Definition:** The `Headers` object represents HTTP request and response headers, providing methods to read, set, and delete headers.

**Technical Definition:** Per MDN, the `Headers` interface of the Fetch API allows you to perform various actions on HTTP request and response headers. These actions include retrieving, setting, adding to, and removing headers from the list of the request's headers.

### Syntax Rules and Structure

```js
const headers = new Headers();
headers.append("Content-Type", "application/json");
headers.set("Authorization", "Bearer token");
headers.get("Content-Type");
headers.has("Content-Type");
headers.delete("Authorization");
```

### Key Methods

| Method | Description |
|--------|-------------|
| `append(name, value)` | Adds a header (allows duplicates) |
| `set(name, value)` | Sets a header (replaces existing) |
| `get(name)` | Gets a header value |
| `getAll(name)` | Gets all values for a header |
| `has(name)` | Checks if a header exists |
| `delete(name)` | Removes a header |
| `forEach()` | Iterates over headers |
| `entries()`, `keys()`, `values()` | Iterators |

### Annotated Code Example

```js
// Step 1: Create headers.
const headers = new Headers();
headers.append("Content-Type", "application/json");
headers.append("Authorization", "Bearer abc123");

// Step 2: Use with fetch.
fetch("https://api.example.com/data", {
  method: "POST",
  headers: headers,
  body: JSON.stringify({ key: "value" }),
});

// Step 3: Read response headers.
const response = await fetch("https://api.example.com/data");
console.log(response.headers.get("Content-Type"));
```

**Expected Output:** The headers are sent with the request and read from the response.

### Real-World Cases

- **Authentication:** Sending `Authorization` headers.
- **Content negotiation:** Setting `Accept` and `Content-Type`.
- **Caching:** Reading `Cache-Control` headers.
- **CORS:** Sending `Origin` and reading `Access-Control-Allow-Origin`.

---

## Body

### Definitions

**Core Definition:** The body of a request or response contains the data being sent or received.

**Technical Definition:** Per MDN, the `body` property of the `Request` interface contains the body of the request, while the `body` property of the `Response` interface contains the body of the response. Both are `ReadableStream` objects. The body can be consumed using methods like `json()`, `text()`, `blob()`, `arrayBuffer()`, and `formData()`.

### Body Types

| Type | Description |
|------|-------------|
| `json()` | Parses as JSON |
| `text()` | Parses as plain text |
| `blob()` | Parses as a Blob (binary data) |
| `arrayBuffer()` | Parses as an ArrayBuffer (binary data) |
| `formData()` | Parses as FormData |

### Important: Body Can Only Be Read Once

```js
const response = await fetch(url);
const data1 = await response.json();
// const data2 = await response.json(); // Error: body already read

// Use clone() if you need to read multiple times:
const response2 = response.clone();
const data3 = await response2.json();
```

### Annotated Code Example

```js
const response = await fetch("https://api.example.com/users");

// Step 1: Parse as JSON.
const users = await response.json();
console.log(users);

// Step 2: Parse as text (different response).
const textResponse = await fetch("https://example.com");
const text = await textResponse.text();
console.log(text);

// Step 3: Parse as Blob (image).
const imageResponse = await fetch("https://example.com/image.png");
const blob = await imageResponse.blob();
const imageUrl = URL.createObjectURL(blob);
```

**Expected Output:** The response bodies are parsed according to their content types.

### Real-World Cases

- **JSON APIs:** Using `json()` for REST API responses.
- **HTML pages:** Using `text()` for HTML content.
- **File downloads:** Using `blob()` for images, PDFs, etc.
- **Binary data:** Using `arrayBuffer()` for binary processing.

---

## HTTP Methods

### Definitions

**Core Definition:** HTTP methods (also called verbs) indicate the desired action to be performed on a resource.

**Technical Definition:** Per MDN, HTTP defines a set of request methods to indicate the purpose of the request and what is expected if the request is successful. The most common methods are GET, POST, PUT, PATCH, and DELETE.

### Common Methods

| Method | Description | Idempotent | Safe |
|--------|-------------|------------|------|
| `GET` | Retrieves a resource | Yes | Yes |
| `POST` | Creates a resource | No | No |
| `PUT` | Replaces a resource | Yes | No |
| `PATCH` | Partially updates a resource | No | No |
| `DELETE` | Deletes a resource | Yes | No |
| `HEAD` | Retrieves headers only | Yes | Yes |
| `OPTIONS` | Describes communication options | Yes | Yes |

### Annotated Code Example

```js
// GET:
await fetch("/api/users");

// POST:
await fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice" }),
});

// PUT:
await fetch("/api/users/1", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice Updated" }),
});

// PATCH:
await fetch("/api/users/1", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice Patched" }),
});

// DELETE:
await fetch("/api/users/1", { method: "DELETE" });
```

**Expected Output:** Each request performs its respective action on the server.

### Real-World Cases

- **REST APIs:** CRUD operations using GET, POST, PUT, PATCH, DELETE.
- **Authentication:** POST for login, DELETE for logout.
- **Search:** GET with query parameters.

---

## Request Options

### Definitions

**Core Definition:** Request options are the configuration settings passed to `fetch()` that control how the request is made.

### Key Options

| Option | Description | Default |
|--------|-------------|---------|
| `method` | HTTP method | `"GET"` |
| `headers` | Request headers | `{}` |
| `body` | Request body | `null` |
| `mode` | Request mode (`cors`, `no-cors`, `same-origin`) | `"cors"` |
| `credentials` | Credentials mode (`omit`, `same-origin`, `include`) | `"same-origin"` |
| `cache` | Cache mode (`default`, `no-store`, `reload`, `no-cache`, `force-cache`, `only-if-cached`) | `"default"` |
| `redirect` | Redirect mode (`follow`, `error`, `manual`) | `"follow"` |
| `referrer` | Referrer URL | `"about:client"` |
| `referrerPolicy` | Referrer policy | `""` |
| `integrity` | Subresource integrity hash | `""` |
| `signal` | AbortSignal for cancellation | `null` |
| `keepalive` | Whether to allow the request to outlive the page | `false` |

### Annotated Code Example

```js
const response = await fetch("/api/data", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token",
  },
  body: JSON.stringify({ key: "value" }),
  credentials: "include",
  cache: "no-cache",
  mode: "cors",
  redirect: "follow",
  signal: abortController.signal,
});
```

**Expected Output:** The request is made with all the specified options.

### Real-World Cases

- **Authenticated requests:** Using `credentials: "include"` for cookies.
- **Cache control:** Using `cache: "no-cache"` for fresh data.
- **Cancellation:** Using `signal` for abortable requests.

---

## Response Handling

### Definitions

**Core Definition:** Response handling is the process of checking the response status, reading headers, and parsing the body.

### Handling Patterns

```js
// Pattern 1: Check response.ok
const response = await fetch(url);
if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}
const data = await response.json();

// Pattern 2: Handle specific status codes
switch (response.status) {
  case 200:
    return response.json();
  case 404:
    throw new Error("Not found");
  case 500:
    throw new Error("Server error");
  default:
    throw new Error(`Unexpected status: ${response.status}`);
}
```

### Annotated Code Example

```js
async function fetchWithErrorHandling(url) {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      // Handle HTTP errors
      if (response.status === 404) {
        throw new Error("Resource not found");
      }
      if (response.status === 401) {
        throw new Error("Unauthorized");
      }
      throw new Error(`HTTP error: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    // Handle network errors and thrown errors
    console.error("Fetch failed:", error.message);
    throw error;
  }
}
```

**Expected Output:** The function returns the parsed JSON or throws a descriptive error.

### Real-World Cases

- **API error handling:** Distinguishing between network errors and HTTP errors.
- **Retry logic:** Retrying on 5xx errors.
- **User feedback:** Showing appropriate messages for different error types.

---

## JSON Parsing

### Definitions

**Core Definition:** JSON parsing is the process of converting a JSON string from a response body into a JavaScript object using `response.json()`.

**Technical Definition:** The `json()` method of the `Response` interface takes a `Response` stream and reads it to completion. It returns a promise which resolves with the result of parsing the body text as JSON.

### Annotated Code Example

```js
const response = await fetch("https://api.example.com/users");
const users = await response.json();
console.log(users); // [{ id: 1, name: 'Alice' }]
```

### Real-World Cases

- **REST APIs:** Almost all modern APIs return JSON.
- **Configuration:** Loading JSON configuration files.
- **Data exchange:** Sending and receiving structured data.

---

## Text Parsing

### Definitions

**Core Definition:** Text parsing reads the response body as a plain text string using `response.text()`.

### Annotated Code Example

```js
const response = await fetch("https://example.com/page.html");
const html = await response.text();
console.log(html); // The HTML source
```

### Real-World Cases

- **HTML fetching:** Loading HTML content.
- **CSV parsing:** Reading CSV files.
- **Plain text:** Reading text files.

---

## Blob Handling

### Definitions

**Core Definition:** Blob handling reads the response body as a `Blob` object, representing binary data such as images, audio, or PDFs.

### Annotated Code Example

```js
const response = await fetch("https://example.com/image.png");
const blob = await response.blob();
const imageUrl = URL.createObjectURL(blob);

const img = document.createElement("img");
img.src = imageUrl;
document.body.appendChild(img);
```

### Real-World Cases

- **Image downloads:** Fetching and displaying images.
- **File downloads:** Downloading PDFs, ZIPs, etc.
- **Media processing:** Fetching audio or video files.

---

## FormData

### Definitions

**Core Definition:** `FormData` is an API for constructing key/value pairs representing form fields and their values, easily sent with `fetch()`.

### Annotated Code Example

```js
// Step 1: Create FormData from a form.
const form = document.getElementById("upload");
const data = new FormData(form);

// Step 2: Add extra data.
data.append("timestamp", Date.now());

// Step 3: Send via fetch.
const response = await fetch("/api/upload", {
  method: "POST",
  body: data, // No Content-Type header — browser sets it automatically
});
```

### Real-World Cases

- **File uploads:** Sending files with form data.
- **Multipart forms:** Submitting complex forms via AJAX.

---

## AbortController

### Definitions

**Core Definition:** `AbortController` is an API for aborting one or more fetch requests using an `AbortSignal`.

### Annotated Code Example

```js
// Step 1: Create a controller.
const controller = new AbortController();

// Step 2: Start a fetch with the signal.
const promise = fetch("/api/data", { signal: controller.signal });

// Step 3: Abort after 2 seconds.
setTimeout(() => controller.abort(), 2000);

try {
  const response = await promise;
  const data = await response.json();
  console.log(data);
} catch (error) {
  if (error.name === "AbortError") {
    console.log("Fetch was aborted");
  }
}
```

**Expected Output:** The fetch is cancelled after 2 seconds, and an `AbortError` is caught.

### Real-World Cases

- **Search autocomplete:** Cancelling previous requests when a new one starts.
- **Navigation:** Cancelling requests when the user leaves the page.
- **Timeouts:** Aborting requests that take too long.

---

## Timeouts

### Definitions

**Core Definition:** A timeout aborts a fetch request if it does not complete within a specified time.

### Annotated Code Example

```js
// Method 1: AbortSignal.timeout()
const response = await fetch("/api/data", {
  signal: AbortSignal.timeout(5000), // 5 seconds
});

// Method 2: Manual timeout with AbortController
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch("/api/data", { signal: controller.signal });
  clearTimeout(timeoutId);
  return response;
} catch (error) {
  if (error.name === "AbortError") {
    throw new Error("Request timed out");
  }
  throw error;
}
```

**Expected Output:** The request is aborted if it exceeds the timeout.

### Real-World Cases

- **Slow networks:** Preventing indefinite waits.
- **User experience:** Showing a timeout message after a reasonable duration.
- **Resource management:** Freeing up connections.

---

## Error Handling

### Definitions

**Core Definition:** Error handling in fetch involves distinguishing between network errors (which reject the Promise) and HTTP errors (which resolve with `response.ok === false`).

### Types of Errors

| Error Type | Cause | How to Handle |
|------------|-------|---------------|
| Network error | DNS failure, offline, CORS | `try...catch` or `.catch()` |
| HTTP error | 4xx, 5xx status codes | Check `response.ok` |
| Abort error | `AbortController.abort()` | Check `error.name === "AbortError"` |
| Parse error | Invalid JSON | `try...catch` around `response.json()` |

### Annotated Code Example

```js
async function fetchData(url) {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    if (error.name === "AbortError") {
      console.log("Request was aborted");
    } else if (error instanceof TypeError) {
      console.log("Network error:", error.message);
    } else {
      console.log("Error:", error.message);
    }
    throw error;
  }
}
```

**Expected Output:** Different error types are handled appropriately.

### Real-World Cases

- **Offline detection:** Distinguishing network errors from HTTP errors.
- **User feedback:** Showing different messages for different error types.
- **Retry logic:** Retrying only on network errors or 5xx responses.

---

## CORS

### Definitions

**Core Definition:** Cross-Origin Resource Sharing (CORS) is a security mechanism that allows a server to indicate which origins are permitted to access its resources.

**Technical Definition:** Per MDN, Cross-Origin Resource Sharing (CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, to check that the server will permit the actual request.

### How CORS Works

1. **Simple request:** The browser sends the request directly with an `Origin` header.
2. **Preflight request:** For non-simple requests, the browser sends an `OPTIONS` request first.
3. **Server response:** The server includes `Access-Control-Allow-Origin` and other CORS headers.
4. **Browser enforcement:** If the headers don't match, the browser blocks the response.

### Common CORS Headers

| Header | Description |
|--------|-------------|
| `Access-Control-Allow-Origin` | Which origins are allowed |
| `Access-Control-Allow-Methods` | Which methods are allowed |
| `Access-Control-Allow-Headers` | Which headers are allowed |
| `Access-Control-Allow-Credentials` | Whether credentials are allowed |
| `Access-Control-Max-Age` | How long preflight results are cached |

### Annotated Code Example

```js
// Client request:
fetch("https://api.other-origin.com/data", {
  method: "GET",
  mode: "cors", // Default for cross-origin
  credentials: "include", // Send cookies
});

// Server response must include:
// Access-Control-Allow-Origin: https://my-origin.com
// Access-Control-Allow-Credentials: true
```

**Expected Output:** The request succeeds if the server includes the correct CORS headers; otherwise, the browser blocks it.

### Real-World Cases

- **Third-party APIs:** Accessing APIs on different domains.
- **CDNs:** Loading fonts, scripts, and images from CDNs.
- **Microservices:** Frontend accessing multiple backend services.

---

## Credentials

### Definitions

**Core Definition:** The `credentials` option controls whether cookies, HTTP authentication, and TLS client certificates are sent with cross-origin requests.

### Options

| Value | Description |
|-------|-------------|
| `"omit"` | Never send credentials |
| `"same-origin"` | Send credentials only for same-origin requests (default) |
| `"include"` | Always send credentials, even for cross-origin requests |

### Annotated Code Example

```js
// Sending cookies with a cross-origin request:
fetch("https://api.example.com/data", {
  credentials: "include",
});

// Server must respond with:
// Access-Control-Allow-Origin: https://my-origin.com
// Access-Control-Allow-Credentials: true
// Note: Access-Control-Allow-Origin cannot be "*" when credentials are included.
```

### Real-World Cases

- **Authenticated APIs:** Sending session cookies.
- **Single sign-on:** Cross-origin authentication.
- **Third-party widgets:** Embedded content with authentication.

---

## Caching

### Definitions

**Core Definition:** The `cache` option controls how the browser's HTTP cache is used for the request.

### Options

| Value | Description |
|-------|-------------|
| `"default"` | Use the browser's default caching behaviour |
| `"no-store"` | Never cache the response |
| `"reload"` | Bypass the cache, but update it |
| `"no-cache"` | Validate with the server before using the cache |
| `"force-cache"` | Use the cache even if stale |
| `"only-if-cached"` | Return from cache; fail if not cached |

### Annotated Code Example

```js
// Force fresh data:
const response = await fetch("/api/data", { cache: "no-store" });

// Use cache if available:
const cachedResponse = await fetch("/api/data", { cache: "force-cache" });
```

### Real-World Cases

- **Fresh data:** Using `no-store` for sensitive or real-time data.
- **Performance:** Using `force-cache` for static assets.
- **Offline:** Using `only-if-cached` for offline-first behaviour.

---

## Streaming Responses

### Definitions

**Core Definition:** Streaming responses allow processing the response body incrementally as chunks arrive, rather than waiting for the entire body.

**Technical Definition:** Per MDN, the `Response.body` property is a `ReadableStream` of the body contents. You can use a `ReadableStream` reader to process chunks as they arrive.

### Annotated Code Example

```js
const response = await fetch("/api/large-data");
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value, { stream: true });
  console.log("Chunk:", chunk);
}
```

**Expected Output:** Chunks of the response are logged as they arrive.

### Real-World Cases

- **Large files:** Downloading large files incrementally.
- **Server-Sent Events (SSE):** Processing event streams.
- **AI streaming:** Receiving AI-generated text incrementally.
- **Progress indicators:** Showing download progress.

---

## Fetch vs XMLHttpRequest

### Definitions

**Core Definition:** `fetch()` and `XMLHttpRequest` (XHR) are two APIs for making HTTP requests. Fetch is the modern replacement, but there are differences.

### Comparison

| Feature | Fetch | XMLHttpRequest |
|---------|-------|----------------|
| **Promise-based** | Yes | No (callback/event-based) |
| **Streaming** | Yes | Limited |
| **Progress events** | No (needs ReadableStream) | Yes (`onprogress`) |
| **Request cancellation** | AbortController | `xhr.abort()` |
| **HTTP error rejection** | No | Yes (`onerror`) |
| **Cookies** | `credentials` option | `withCredentials` |
| **Synchronous mode** | No | Yes (deprecated) |
| **Service Worker support** | Yes | No |
| **CORS** | Yes | Yes |
| **Timeout** | AbortSignal | `xhr.timeout` |

### Annotated Code Example: XHR vs Fetch

```js
// XMLHttpRequest:
const xhr = new XMLHttpRequest();
xhr.open("GET", "/api/data");
xhr.onload = () => {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.onerror = () => console.error("Network error");
xhr.send();

// Fetch:
fetch("/api/data")
  .then((response) => {
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((error) => console.error("Error:", error));
```

### Real-World Cases

- **New projects:** Always prefer `fetch()`.
- **Legacy code:** XHR may still be present.
- **Progress tracking:** XHR has built-in progress events; fetch requires manual stream reading.

---

## References

- MDN Web Docs – Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN Web Docs – Using the Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- MDN Web Docs – Window: fetch() — https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch
- MDN Web Docs – Request — https://developer.mozilla.org/en-US/docs/Web/API/Request
- MDN Web Docs – Response — https://developer.mozilla.org/en-US/docs/Web/API/Response
- MDN Web Docs – Headers — https://developer.mozilla.org/en-US/docs/Web/API/Headers
- MDN Web Docs – AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – AbortSignal — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- MDN Web Docs – FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs – Cross-Origin Resource Sharing (CORS) — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN Web Docs – HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN Web Docs – Cache — https://developer.mozilla.org/en-US/docs/Web/API/Cache
- WHATWG – Fetch Living Standard — https://fetch.spec.whatwg.org/
- WHATWG – XMLHttpRequest Living Standard — https://xhr.spec.whatwg.org/
- Chrome for Developers – Fetch API — https://developer.chrome.com/docs/
- web.dev – Fetch API — https://web.dev/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf