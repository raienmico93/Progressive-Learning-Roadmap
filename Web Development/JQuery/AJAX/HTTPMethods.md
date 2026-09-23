# HTTP Methods in AJAX: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
HTTP methods in AJAX are the verbs used in asynchronous requests to specify the desired action on a server resource — retrieving, creating, updating, or deleting data.

**Technical Definition**
The HTTP protocol defines request methods that indicate the semantics of a request. In jQuery's `$.ajax()`, the `type` (or `method` in jQuery 1.9+) setting specifies which HTTP method to use. GET requests data without side effects and may be cached; POST submits data and typically creates resources; PUT replaces an entire resource idempotently; PATCH applies partial modifications; DELETE removes a resource .

**Beginner-Friendly Explanation**
HTTP methods are like different verbs you use when talking to a server. GET means "give me data," POST means "create something new," PUT means "replace this entirely," PATCH means "change just this part," and DELETE means "remove this."

### Key Characteristics

- **Semantic Meaning**: Each method has defined semantics regarding safety and idempotency.
- **Browser Support**: GET and POST are universally supported; PUT, PATCH, and DELETE require modern browsers .
- **Caching Behaviour**: GET requests may be cached; jQuery provides a `cache` option to control this .
- **Idempotency**: GET, PUT, and DELETE are idempotent (repeated calls have the same effect); POST and PATCH are not .

### Prerequisites

- Basic understanding of HTTP and REST concepts.
- jQuery library included via CDN or local file.
- Familiarity with `$.ajax()` configuration.

### Related Programming Areas

- **REST APIs**: Representational State Transfer architecture uses HTTP methods semantically.
- **CORS**: Cross-origin requests may trigger preflight OPTIONS requests for non-simple methods.
- **Caching**: Browser and server caching behaviour differs by method.

### Core Concepts / Features

1. GET (Retrieving Data and Caching Behaviours)
2. POST (Submitting Data)
3. PUT and PATCH (Updating Entire vs. Partial Resources)
4. DELETE (Removing Resources)

---

## Core Concept 1: GET (Retrieving Data and Caching Behaviours)

### Definitions

**Core Definition**
GET is the HTTP method used to retrieve data from a server without causing side effects, with data typically sent as query parameters.

**Technical Definition**
The GET method requests a representation of the specified resource. In AJAX, GET requests append data to the URL as a query string. GET requests are considered "safe" (no side effects) and "idempotent" (repeated calls return the same result). Browsers may cache GET responses, and jQuery's `cache` setting controls whether jQuery appends a cache-busting timestamp parameter .

**Beginner-Friendly Explanation**
GET is for reading data. You ask the server "give me this information," and the server sends it back. GET requests can be cached, which means the browser might remember the response and not ask the server again — this is usually good for performance but can cause stale data issues.

### Purposes

- To retrieve data from a server without modifying it.
- To search or filter data using query parameters.
- To load HTML fragments, JSON, or XML content.
- To implement idempotent read operations in REST APIs.
- To leverage browser caching for improved performance.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/resource",
  type: "GET",           // Default method
  data: { key: "value" }, // Appended as query string
  cache: true,           // Default: true (browser caching allowed)
  dataType: "json"
});
```

**Component Breakdown**

- `type: "GET"` : Default HTTP method .
- `data` : Object or string serialized and appended to URL as query string.
- `cache` : Boolean. `true` (default) allows browser caching; `false` appends `_={timestamp}` to force fresh requests .

**Syntax Rules**

1. GET is the default method for `$.ajax()` .
2. Data is appended to the URL as a query string (`?key=value&key2=value2`).
3. `cache: false` appends a timestamp parameter to bypass browser cache .
4. GET requests should be used only for non-destructive operations .
5. URL length limits apply; use POST for large data.

**Constraints and Limitations**

- **URL Length**: Limited by browser and server URL length constraints.
- **Security**: Data is visible in URL, browser history, and server logs.
- **Caching**: Unpredictable caching behaviour can lead to stale data .
- **Binary Data**: Not suitable for binary data; use POST with `FormData`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic GET Request**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GET — Basic Retrieval</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="loadBtn">Load Data</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#loadBtn").on("click", function () {
        // Step 1: Send GET request with query parameters
        $.ajax({
          url: "/api/users",
          type: "GET",  // Default, but explicit
          data: { role: "admin", limit: 10 },
          dataType: "json"
        }).done(function (users) {
          // Step 2: Handle response
          $("#output").text("Loaded " + users.length + " admin users.");
        }).fail(function (xhr, status, error) {
          $("#output").text("Error: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Load Data" sends `GET /api/users?role=admin&limit=10`.
- Displays the count of loaded users.

**Why This Output Occurs**
The `data` object is serialized into a query string and appended to the URL for GET requests .

---

**Example 2: Controlling Cache with `cache: false`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GET — Cache Control</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="freshBtn">Load Fresh Data</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#freshBtn").on("click", function () {
        // Step 1: Force fresh request by disabling cache
        $.ajax({
          url: "/api/current-time",
          type: "GET",
          cache: false,  // Appends _={timestamp} to URL
          dataType: "text"
        }).done(function (time) {
          $("#output").text("Server time: " + time);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Each click fetches the current server time (no cached response).
- URL includes `?_=1234567890` timestamp parameter.

**Why This Output Occurs**
`cache: false` appends a timestamp parameter to the URL, making each request unique and bypassing browser cache .

### Real-World Cases

- **Search Queries**: Sending search terms as GET parameters.
- **Autocomplete**: Fetching suggestions as the user types.
- **Content Loading**: Loading HTML fragments or JSON data.
- **Polling**: Periodically checking for updates (use `cache: false`).

### References

- Learn jQuery — Key Concepts (GET vs POST) – https://learn.jquery.com/ajax/key-concepts/
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- MDN Web Docs — GET – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET

---

## Core Concept 2: POST (Submitting Data)

### Definitions

**Core Definition**
POST is the HTTP method used to submit data to a server, typically resulting in the creation of a new resource or a change in server state.

**Technical Definition**
The POST method sends data in the request body, with the format indicated by the `Content-Type` header. Unlike PUT, POST is not idempotent — repeated identical requests may have additional side effects (e.g., creating multiple orders). POST is the standard method for HTML form submission and is suitable for creating resources, appending data, or submitting data blocks for processing .

**Beginner-Friendly Explanation**
POST is for sending data to the server, usually to create something new. You put the data in the request body (not the URL), and the server processes it. If you send the same POST request twice, you might create two records.

### Purposes

- To create new resources on the server.
- To submit form data that changes server state.
- To send large amounts of data (no URL length limit).
- To upload binary data (with `multipart/form-data`).
- To append data to an existing resource.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/resource",
  type: "POST",
  data: { name: "New Item", price: 19.99 },
  contentType: "application/x-www-form-urlencoded; charset=UTF-8",
  dataType: "json"
});
```

**Component Breakdown**

- `type: "POST"` : Explicitly sets POST method.
- `data` : Serialized into request body.
- `contentType` : Format of the data being sent (default: `application/x-www-form-urlencoded`) .

**Common Content Types**

| Content Type | Use Case |
|---|---|
| `application/x-www-form-urlencoded` | Standard form data |
| `multipart/form-data` | File uploads |
| `application/json` | JSON API payloads |
| `text/plain` | Plain text |

**Syntax Rules**

1. POST sends data in the request body, not the URL.
2. Default `contentType` is `application/x-www-form-urlencoded; charset=UTF-8` .
3. For JSON APIs, set `contentType: "application/json"` and stringify data.
4. POST is not idempotent — repeated requests may create duplicates .
5. Success responses typically have status `201 Created`.

**Constraints and Limitations**

- **Non-Idempotent**: Repeated calls may create multiple resources.
- **CSRF**: POST requests are vulnerable to Cross-Site Request Forgery; use CSRF tokens.
- **Caching**: POST responses are not cached by default.

### Multiple Annotated Complete Code Examples

**Example 1: POST with Form-Encoded Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>POST — Form Data</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="createBtn">Create User</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#createBtn").on("click", function () {
        // Step 1: Send POST with form-encoded data
        $.ajax({
          url: "/api/users",
          type: "POST",
          data: {
            name: "Alice",
            email: "alice@example.com",
            role: "admin"
          },
          dataType: "json"
        }).done(function (response) {
          // Step 2: Handle created resource
          $("#output").text("Created user ID: " + response.id);
        }).fail(function (xhr, status, error) {
          $("#output").text("Creation failed: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Create User" sends `POST /api/users` with body `name=Alice&email=alice%40example.com&role=admin`.
- Server returns the new user's ID.

**Why This Output Occurs**
The `data` object is serialized into the request body in `application/x-www-form-urlencoded` format .

---

**Example 2: POST with JSON Data**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>POST — JSON Payload</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="jsonBtn">Create JSON</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#jsonBtn").on("click", function () {
        // Step 1: Send POST with JSON body
        $.ajax({
          url: "/api/products",
          type: "POST",
          contentType: "application/json; charset=utf-8",
          data: JSON.stringify({
            name: "Laptop",
            price: 999.99,
            tags: ["electronics", "computers"]
          }),
          dataType: "json"
        }).done(function (response) {
          $("#output").text("Created: " + response.name);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Request body is JSON: `{"name":"Laptop","price":999.99,"tags":["electronics","computers"]}`.
- Server returns the created product.

**Why This Output Occurs**
`contentType: "application/json"` tells the server the body is JSON. `JSON.stringify()` converts the object to a JSON string .

### Real-World Cases

- **User Registration**: Creating new accounts with POST.
- **Order Placement**: Submitting orders with line items.
- **File Uploads**: Sending files with `multipart/form-data`.
- **Comment Systems**: Posting new comments to a thread.

### References

- MDN Web Docs — POST – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST
- MDN Web Docs — POST (Chinese) – https://mdn.org.cn/en-US/docs/Web/HTTP/Methods/POST
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/

---

## Core Concept 3: PUT and PATCH (Updating Entire vs. Partial Resources)

### Definitions

**Core Definition**
PUT replaces an entire resource with the provided data, while PATCH applies partial modifications to a resource.

**Technical Definition**
Both PUT and PATCH send data in the request body to update an existing resource. PUT is idempotent — sending the same PUT request multiple times produces the same result. PATCH is not necessarily idempotent — it applies a set of changes that may not be safe to repeat. PUT is used when you want to replace the entire resource; PATCH is used for partial updates .

**Beginner-Friendly Explanation**
PUT means "here is the complete new version of this resource — replace what you have." PATCH means "here are the specific changes to apply — update just these fields." PUT requires you to send the entire resource; PATCH lets you send only what changed.

### Purposes

- To replace an entire resource with a new representation (PUT).
- To update specific fields without sending the whole resource (PATCH).
- To implement idempotent update operations (PUT).
- To reduce payload size for partial updates (PATCH).

### Syntax Rules and Structure

**Complete General Syntaxes**

**PUT (Full Replacement):**
```javascript
$.ajax({
  url: "/api/users/123",
  type: "PUT",
  contentType: "application/json",
  data: JSON.stringify({
    name: "Alice Smith",
    email: "alice.smith@example.com",
    role: "admin"
  })
});
```

**PATCH (Partial Update):**
```javascript
$.ajax({
  url: "/api/users/123",
  type: "PATCH",
  contentType: "application/json",
  data: JSON.stringify({
    email: "newemail@example.com"
  })
});
```

**Component Breakdown**

- `type: "PUT"` : Full resource replacement.
- `type: "PATCH"` : Partial modification.
- `url` : Includes the resource identifier (e.g., `/users/123`).

**Syntax Rules**

1. PUT is idempotent; PATCH is not necessarily idempotent .
2. PUT typically requires the full resource representation.
3. PATCH sends only the changed fields.
4. PATCH has limited browser support compared to PUT/DELETE .
5. CORS preflight (OPTIONS) is required for cross-origin PUT/PATCH.

**Constraints and Limitations**

- **Browser Support**: PUT and DELETE work in all modern browsers; PATCH has weaker support .
- **CORS**: Cross-origin requests trigger preflight OPTIONS .
- **Method Spoofing**: Some servers/frameworks don't support PUT/PATCH natively; method override may be needed.

### Multiple Annotated Complete Code Examples

**Example 1: PUT for Full Replacement**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>PUT — Full Update</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="putBtn">Replace User</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#putBtn").on("click", function () {
        // Step 1: Send PUT with full resource data
        $.ajax({
          url: "/api/users/123",
          type: "PUT",
          contentType: "application/json",
          data: JSON.stringify({
            name: "Alice Smith",
            email: "alice.smith@example.com",
            role: "superadmin",
            active: true
          }),
          dataType: "json"
        }).done(function (response) {
          $("#output").text("Replaced: " + response.name);
        }).fail(function (xhr, status, error) {
          $("#output").text("Error: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Sends `PUT /api/users/123` with the complete new representation.
- Server replaces the entire resource.

**Why This Output Occurs**
PUT replaces the entire resource with the provided representation .

---

**Example 2: PATCH for Partial Update**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>PATCH — Partial Update</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="patchBtn">Update Email Only</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#patchBtn").on("click", function () {
        // Step 1: Send PATCH with only changed field
        $.ajax({
          url: "/api/users/123",
          type: "PATCH",
          contentType: "application/json",
          data: JSON.stringify({
            email: "newemail@example.com"
          }),
          dataType: "json"
        }).done(function (response) {
          $("#output").text("Updated email for: " + response.name);
        }).fail(function (xhr, status, error) {
          $("#output").text("PATCH failed: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Sends `PATCH /api/users/123` with only `{"email":"newemail@example.com"}`.
- Server updates only the email field.

**Why This Output Occurs**
PATCH applies partial modifications, sending only the changed fields .

### Real-World Cases

- **Profile Updates**: PUT for full profile replacement, PATCH for changing just an email.
- **Inventory Management**: PUT to replace product details, PATCH for stock adjustments.
- **Configuration**: PUT for complete config replacement, PATCH for toggling a setting.

### References

- MDN Web Docs — PUT – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT
- MDN Web Docs — PATCH – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH
- Stack Overflow — PUT/DELETE browser support – https://stackoverflow.com/questions/5894400/

---

## Core Concept 4: DELETE (Removing Resources)

### Definitions

**Core Definition**
DELETE is the HTTP method used to remove a specified resource from the server.

**Technical Definition**
The DELETE method requests that the server delete the resource identified by the request URL. DELETE is idempotent — calling it multiple times has the same effect (the resource is deleted, and subsequent calls return 404 or success). DELETE requests typically have no request body. jQuery supports `type: "DELETE"` in `$.ajax()`, though cross-origin requests may trigger CORS preflight .

**Beginner-Friendly Explanation**
DELETE is for removing things. You tell the server "delete this resource," and it's gone. If you try to delete it again, the server will say it's already gone.

### Purposes

- To remove resources from the server.
- To implement RESTful deletion endpoints.
- To allow users to delete their own content.
- To clean up temporary or expired resources.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$.ajax({
  url: "/api/resource/123",
  type: "DELETE",
  dataType: "json"
});
```

**Component Breakdown**

- `type: "DELETE"` : Specifies the DELETE method.
- `url` : Includes the resource identifier to delete.

**Syntax Rules**

1. DELETE is idempotent — repeated calls have the same effect .
2. DELETE requests typically have no body.
3. Success responses may be `200 OK` or `204 No Content`.
4. CORS preflight is triggered for cross-origin DELETE requests .
5. Some older browsers had issues with DELETE; modern browsers support it .

**Constraints and Limitations**

- **CORS**: Cross-origin DELETE requires server CORS configuration .
- **Method Spoofing**: Some servers require `X-HTTP-Method-Override` for DELETE.
- **No Undo**: Deletion is permanent unless soft-delete is implemented server-side.

### Multiple Annotated Complete Code Examples

**Example 1: Basic DELETE Request**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DELETE — Remove Resource</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="deleteBtn" data-id="123">Delete Item</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#deleteBtn").on("click", function () {
        var itemId = $(this).data("id");

        // Step 1: Send DELETE request
        $.ajax({
          url: "/api/items/" + itemId,
          type: "DELETE",
          dataType: "json"
        }).done(function (response) {
          // Step 2: Handle successful deletion
          $("#output").text("Item deleted successfully.");
        }).fail(function (xhr, status, error) {
          $("#output").text("Delete failed: " + error);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Delete Item" sends `DELETE /api/items/123`.
- Server removes the item and returns success.

**Why This Output Occurs**
`type: "DELETE"` sends the DELETE HTTP method to the specified URL .

---

**Example 2: DELETE with Confirmation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DELETE — With Confirmation</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="confirmDelete" data-id="456">Delete with Confirm</button>
  <div id="output"></div>

  <script>
    $(function () {
      $("#confirmDelete").on("click", function () {
        var $btn = $(this);
        var itemId = $btn.data("id");

        // Step 1: Confirm before deleting
        if (!confirm("Are you sure you want to delete item " + itemId + "?")) {
          return;
        }

        // Step 2: Disable button during request
        $btn.prop("disabled", true);

        // Step 3: Send DELETE
        $.ajax({
          url: "/api/items/" + itemId,
          type: "DELETE"
        }).always(function () {
          $btn.prop("disabled", false);
        }).done(function () {
          $("#output").text("Deleted.");
        }).fail(function () {
          $("#output").text("Delete failed.");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking shows a confirmation dialog.
- If confirmed, sends DELETE request and disables the button during the request.

**Why This Output Occurs**
The confirmation prevents accidental deletions. The button is disabled to prevent duplicate requests .

### Real-World Cases

- **User Content**: Deleting posts, comments, or uploaded files.
- **Shopping Carts**: Removing items from a cart.
- **Admin Panels**: Deleting user accounts or records.
- **REST APIs**: Standard DELETE endpoints for resource removal.

### References

- Stack Overflow — DELETE with jQuery AJAX – https://stackoverflow.com/questions/29123885/
- Stack Overflow — DELETE verb in AJAX – https://stackoverflow.com/questions/10659293/
- Aliyun — AJAX PUT/DELETE Implementation – https://developer.aliyun.com/ask/71912

---

## Summary Comparison Table

| Method | Purpose | Idempotent | Safe | Cacheable | Browser Support |
|---|---|---|---|---|---|
| **GET** | Retrieve data | Yes | Yes | Yes | Universal |
| **POST** | Create/submit | No | No | No | Universal |
| **PUT** | Replace resource | Yes | No | No | Modern browsers |
| **PATCH** | Partial update | No | No | No | Limited |
| **DELETE** | Remove resource | Yes | No | No | Modern browsers |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **Default Method**: `$.ajax()` defaults to GET if `type` is not specified .
2. **Browser Support**: PUT and DELETE work in modern browsers; PATCH has weaker support .
3. **CORS**: Cross-origin PUT/DELETE/PATCH requests trigger preflight OPTIONS .
4. **Cache Option**: `cache: false` only applies to GET and HEAD requests .
5. **jQuery 1.9+**: The `method` alias was added for `type`.
6. **jQuery 4.1**: `responseURL` property added for accessing final URL after redirects .

---

## References

- Learn jQuery — Key Concepts – https://learn.jquery.com/ajax/key-concepts/
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- MDN Web Docs — HTTP Methods – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN Web Docs — POST – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST
- MDN Web Docs — PUT – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT
- MDN Web Docs — PATCH – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH
- MDN Web Docs — DELETE – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE
- Stack Overflow — jQuery AJAX Caching – https://stackoverflow.com/questions/74004889/
- Stack Overflow — PUT/DELETE Browser Support – https://stackoverflow.com/questions/5894400/
- Stack Overflow — DELETE with jQuery AJAX – https://stackoverflow.com/questions/29123885/
- Aliyun — AJAX PUT/DELETE – https://developer.aliyun.com/ask/71912