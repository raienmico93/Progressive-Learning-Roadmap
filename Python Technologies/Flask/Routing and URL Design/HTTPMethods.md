# HTTP Methods in Flask: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP methods are verbs defined by the HTTP protocol that indicate the desired action to be performed on a resource. In Flask, routes specify which methods they accept via the `methods` parameter, and view functions can branch on `request.method` to handle different behaviors.

**Technical Definition:** HTTP/1.1 methods (RFC 9110) are tokens sent in the request line that convey the semantics of a request. Flask routes are registered with Werkzeug `Rule` objects that carry a `methods` set. By default, a rule listens for `GET` and implicitly `HEAD`; since Flask 0.6, `OPTIONS` is also added automatically and handled by standard request dispatching. The `methods` parameter in `@app.route()` or `app.add_url_rule()` limits the rule to the specified methods; requests using other methods receive `405 Method Not Allowed`.

**Beginner-Friendly Explanation:** When your browser or an API client talks to your Flask server, it says what it wants to do using an HTTP method—like "give me this page" (GET), "here is some new data" (POST), "replace this" (PUT), "change just this part" (PATCH), or "remove this" (DELETE). In Flask, you tell each route which methods it should respond to, and inside your function you can check `request.method` to do different things depending on what the client asked for.

### Key Characteristics

- **Default method is GET:** A route defined without a `methods` parameter only answers `GET` requests (plus implicit `HEAD` and `OPTIONS`).
- **Explicit method restriction:** The `methods` parameter accepts a list of strings (e.g., `["GET", "POST"]`); the route then rejects all other methods with `405`.
- **Conditional logic:** A single route can accept multiple methods and branch on `request.method` to handle each differently.
- **Automatic HEAD and OPTIONS:** If `GET` is present, `HEAD` is added implicitly; `OPTIONS` is added automatically unless explicitly included or disabled via `provide_automatic_options`.
- **Method overriding:** Because some proxies and clients cannot send `PUT`, `PATCH`, or `DELETE`, Flask documents a middleware pattern using `X-HTTP-Method-Override` to tunnel those methods through `POST`.
- **CORS preflight:** Cross-origin requests with non-simple methods or content types trigger an automatic `OPTIONS` preflight; Flask-CORS can override Flask's default `OPTIONS` handling to return the required CORS headers.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTTP request/response cycle.
- Familiarity with Flask routing (see the Flask Routes cheat sheet).
- Optional: `flask-cors` for cross-origin scenarios (`pip install flask-cors`).

### Related Programming Areas

- **REST API design:** HTTP methods map directly to CRUD operations.
- **Web security:** Safe and idempotent method semantics affect caching and retry behavior.
- **Reverse proxies and load balancers:** Some proxies restrict or rewrite HTTP methods.
- **Browser fetch/XHR:** CORS preflight and method availability affect front-end integration.
- **API clients:** Tools like `curl`, `requests`, and `httpx` send methods as part of the request.

### Core Concepts / Features

1. GET (Safe and Idempotent Retrieval)
2. POST (Resource Creation and Payload Parsing)
3. PUT (Idempotent Replacement)
4. PATCH (Partial Resource Modification)
5. DELETE (Resource Removal)
6. Multiple Methods on One Route (Conditional Logic)
7. Method-Specific Behavior (HEAD, OPTIONS, and CORS Preflight)
8. HTTP Method Overriding (Headers and Form Fields)

---

## 1. GET (Safe and Idempotent Retrieval)

### Definitions

**Core Definition:** The `GET` method requests a representation of a specified resource. It is safe (read-only) and idempotent (repeating it has no additional side effects).

**Technical Definition:** Per RFC 9110, `GET` is safe and idempotent, meaning it must not change server state and identical requests produce the same effect as a single request. In Flask, `GET` is the default method for any route; data is passed via query string parameters and accessed through `request.args`.

**Beginner-Friendly Explanation:** GET is what happens when you type a URL into your browser. It says “please show me this page” and must not change anything on the server. You can safely refresh a GET request.

### Purposes

- To retrieve a resource without modifying server state.
- To provide a safe, cacheable, and bookmarkable URL for a resource.
- To pass simple parameters via the query string.
- To serve as the default method for browser navigation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/resource", methods=["GET"])
def get_resource():
    param = request.args.get("key", default="")
    return f"Retrieved: {param}"

# GET is default — this is equivalent
@app.route("/default")
def default_get():
    return "Default GET route"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=["GET"]` | Explicitly restricts to GET (optional; GET is default) |
| `request.args` | ImmutableMultiDict of query string parameters |
| `request.args.get("key")` | Safely retrieve a parameter with optional default |

**Syntax Rules:**

- If `methods` is omitted, the route accepts `GET` (and implicitly `HEAD` and `OPTIONS`).
- Query parameters are always strings; convert types manually if needed.
- `GET` requests should never have a request body with meaningful content; browsers and servers may ignore it.

**Constraints and Limitations:**

- `GET` requests are limited in length by browsers and servers (typically ~2000 characters for the URL).
- Sensitive data should not be sent via `GET` because URLs are logged, cached, and stored in browser history.
- Caching proxies may serve stale responses for `GET` unless cache headers are set.

### Annotated Code Examples

**Example 1: Basic GET Route with Query Parameters**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/search", methods=["GET"])
def search():
    # Retrieve query parameters with defaults
    query = request.args.get("q", "")
    limit = request.args.get("limit", 10, type=int)
    
    # Simulate search results
    results = [f"Result {i} for '{query}'" for i in range(limit)]
    return jsonify({"query": query, "limit": limit, "results": results})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /search?q=flask&limit=3` → `{"query": "flask", "limit": 3, "results": ["Result 0 for 'flask'", "Result 1 for 'flask'", "Result 2 for 'flask'"]}`
- `GET /search` → `{"query": "", "limit": 10, "results": ["Result 0 for ''", ..., "Result 9 for ''"]}`

**Why this output:** `request.args` holds the query string. `.get("q", "")` returns the empty string if `q` is absent. `.get("limit", 10, type=int)` converts the value to an integer; if conversion fails, Flask returns `None`, but with a default provided, it falls back to `10`.

**Example 2: Conditional GET with `request.method`**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/items", methods=["GET", "POST"])
def items():
    if request.method == "GET":
        return "Listing items"
    # POST logic here (not shown)
    return "Creating item"

# A route that only ever handles GET
@app.route("/about")
def about():
    return "About page"
```

**Expected Output:**
- `GET /items` → `"Listing items"`
- `POST /items` → `"Creating item"`
- `GET /about` → `"About page"`

**Why this output:** The `/items` route accepts both `GET` and `POST`. Inside the view, `request.method` determines the branch. The `/about` route has no `methods` parameter, so it defaults to `GET` only; a `POST` request would return `405`.

### Real-World Cases

- **Search pages:** `/search?q=term` returns results without changing server state.
- **REST API reads:** `GET /api/users/42` retrieves a user profile.
- **Static content:** Serving HTML, CSS, or documentation pages.

### References

- RFC 9110: HTTP Semantics (Section 9.3.1) — https://www.rfc-editor.org/rfc/rfc9110
- Flask Quickstart: Routing — https://flask.palletsprojects.com/en/stable/quickstart/#routing
- Flask API: `request.args` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.args

---

## 2. POST (Resource Creation and Payload Parsing)

### Definitions

**Core Definition:** The `POST` method submits data to be processed by the server, typically resulting in the creation of a new resource or the triggering of a side effect. It is neither safe nor idempotent.

**Technical Definition:** Per RFC 9110, `POST` is not safe and not idempotent; repeating a `POST` may create duplicate resources. In Flask, `POST` data is parsed from the request body using `request.form` (for `application/x-www-form-urlencoded` and `multipart/form-data`) or `request.get_json()` (for `application/json`).

**Beginner-Friendly Explanation:** POST is what happens when you submit a form or send data to an API. It can change things on the server—like creating a new account or posting a comment. Repeating the same POST might create duplicate entries, so it is not idempotent.

### Purposes

- To create a new resource on the server.
- To submit form data or JSON payloads for processing.
- To trigger side effects that are not safely repeatable.
- To upload files and complex data structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/resource", methods=["POST"])
def create_resource():
    # Form data
    form_value = request.form.get("field")
    
    # JSON data
    json_data = request.get_json(silent=True)
    
    return jsonify({"received": True}), 201
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=["POST"]` | Restricts route to POST only |
| `request.form` | Parsed form data (URL-encoded or multipart) |
| `request.get_json()` | Parsed JSON body; returns `None` if invalid or wrong content type |
| `silent=True` | Prevents raising an error on invalid JSON; returns `None` |
| `201` | HTTP status “Created” — appropriate for successful POST creation |

**Syntax Rules:**

- `request.form` and `request.get_json()` are mutually exclusive based on the `Content-Type` header.
- `request.get_json()` requires `Content-Type: application/json`; otherwise it raises `415 Unsupported Media Type` (unless `force=True`).
- POST data is not cached by default; responses to POST are not cached by browsers.
- A `POST` request can have a body; `CONTENT_LENGTH` and `CONTENT_TYPE` headers describe it.

**Constraints and Limitations:**

- POST is not idempotent; retrying a failed POST may create duplicates unless the server implements idempotency keys.
- CSRF protection is essential for browser-based POST forms (Flask-WTF provides this).
- Large file uploads require streaming handling; `request.files` provides `FileStorage` objects.

### Annotated Code Examples

**Example 1: Handling JSON POST Data**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/api/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if not data:
        return jsonify({"error": "No JSON body provided"}), 400
    
    name = data.get("name")
    email = data.get("email")
    if not name or not email:
        return jsonify({"error": "Missing name or email"}), 422
    
    # In a real app, save to database here
    return jsonify({"id": 1, "name": name, "email": email}), 201
```

**Expected Output:**
- `POST /api/users` with `{"name": "Alice", "email": "alice@example.com"}` → `{"id": 1, "name": "Alice", "email": "alice@example.com"}` with status `201`
- `POST /api/users` with empty body → `{"error": "No JSON body provided"}` with status `400`
- `POST /api/users` with `{"name": "Alice"}` → `{"error": "Missing name or email"}` with status `422`

**Why this output:** `request.get_json()` parses the JSON body into a Python dict. Validation checks ensure required fields are present. Status codes reflect the outcome: `201` for created, `400` for bad request, `422` for unprocessable entity.

**Example 2: Handling Form POST Data**

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

FORM = """
<form method="POST" action="/submit">
    <input name="username" placeholder="Username">
    <input name="password" type="password" placeholder="Password">
    <button type="submit">Log in</button>
</form>
"""

@app.route("/")
def index():
    return render_template_string(FORM)

@app.route("/submit", methods=["POST"])
def submit():
    username = request.form.get("username")
    password = request.form.get("password")
    if not username or not password:
        return "Missing credentials", 400
    return f"Welcome, {username}!"
```

**Expected Output:**
- Submitting the form with `username=alice` and `password=secret` → `"Welcome, alice!"`
- Submitting with empty fields → `"Missing credentials"` with status `400`

**Why this output:** `request.form` parses `application/x-www-form-urlencoded` data from the HTML form. The `.get()` method safely retrieves fields, returning `None` if absent.

### Real-World Cases

- **User registration:** `POST /register` creates a new account.
- **Comment posting:** `POST /posts/42/comments` adds a comment.
- **File upload:** `POST /upload` with `multipart/form-data` sends a file.
- **Webhook receivers:** Third-party services `POST` events to your endpoint.

### References

- RFC 9110: HTTP Semantics (Section 9.3.3) — https://www.rfc-editor.org/rfc/rfc9110
- Flask Quickstart: HTTP Methods — https://flask.palletsprojects.com/en/stable/quickstart/#http-methods
- Flask API: `request.get_json` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.get_json

---

## 3. PUT (Idempotent Replacement)

### Definitions

**Core Definition:** The `PUT` method replaces the entire representation of a target resource with the request payload. It is idempotent but not safe.

**Technical Definition:** Per RFC 9110, `PUT` is not safe but is idempotent; repeating the same `PUT` request produces the same server state as a single request. In Flask, `PUT` is handled like `POST` for payload parsing (`request.get_json()` or `request.form`), but the URL typically includes the resource identifier (e.g., `/users/42`).

**Beginner-Friendly Explanation:** PUT means “replace the whole thing with this new version.” If you PUT the same data twice, the result is the same as doing it once. It is used when you want to overwrite an existing resource completely.

### Purposes

- To replace an existing resource with a new representation.
- To create a resource at a known URL if it does not exist (idempotent upsert).
- To ensure that retrying a request does not change the outcome beyond the first successful attempt.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/resource/<int:resource_id>", methods=["PUT"])
def replace_resource(resource_id):
    data = request.get_json()
    # Replace resource with resource_id using data
    return jsonify({"id": resource_id, "replaced": True})
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=["PUT"]` | Restricts route to PUT only |
| `<int:resource_id>` | URL variable identifying the resource to replace |
| `request.get_json()` | Parses the new representation from the body |
| `200` or `204` | Typical success status codes (204 if no body returned) |

**Syntax Rules:**

- The request body of a `PUT` should contain the complete new representation of the resource.
- If the resource does not exist, `PUT` may create it (upsert) or return `404`, depending on application semantics.
- `PUT` requests with bodies should include `Content-Type` headers.
- Some proxies and clients do not support `PUT`; method overriding may be required.

**Constraints and Limitations:**

- `PUT` is not safe; it modifies server state.
- The client must send the full representation; partial updates require `PATCH`.
- Idempotency depends on correct server implementation; a buggy handler could create duplicates.

### Annotated Code Examples

**Example 1: Replacing a User Resource**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# In-memory store for demonstration
users = {
    1: {"name": "Alice", "email": "alice@example.com"},
    2: {"name": "Bob", "email": "bob@example.com"}
}

@app.route("/api/users/<int:user_id>", methods=["PUT"])
def replace_user(user_id):
    data = request.get_json()
    if not data:
        return jsonify({"error": "No JSON body"}), 400
    
    # Complete replacement — all fields must be provided
    users[user_id] = {
        "name": data.get("name"),
        "email": data.get("email")
    }
    return jsonify({"id": user_id, "user": users[user_id]}), 200
```

**Expected Output:**
- `PUT /api/users/1` with `{"name": "Alice Smith", "email": "alice.smith@example.com"}` → `{"id": 1, "user": {"name": "Alice Smith", "email": "alice.smith@example.com"}}`
- Repeating the same `PUT` produces the identical result (idempotent).

**Why this output:** The handler replaces the entire user record with the provided data. Because the operation is idempotent, repeating the request does not change the outcome beyond the first application.

**Example 2: PUT with 204 No Content**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

items = {1: "item one"}

@app.route("/api/items/<int:item_id>", methods=["PUT"])
def replace_item(item_id):
    data = request.get_json()
    items[item_id] = data.get("value", "")
    return "", 204  # No content returned
```

**Expected Output:**
- `PUT /api/items/1` with `{"value": "updated"}` → status `204 No Content` with empty body.

**Why this output:** `204 No Content` is appropriate when the server has fulfilled the request and does not need to return a body. The client knows the replacement succeeded from the status code alone.

### Real-World Cases

- **Configuration management:** `PUT /config` replaces the entire configuration.
- **File uploads (overwrite):** `PUT /files/report.pdf` replaces the file at that path.
- **REST API updates:** `PUT /api/products/42` replaces a product record.

### References

- RFC 9110: HTTP Semantics (Section 9.3.4) — https://www.rfc-editor.org/rfc/rfc9110
- Flask API: `request.get_json` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.get_json

---

## 4. PATCH (Partial Resource Modification)

### Definitions

**Core Definition:** The `PATCH` method applies partial modifications to an existing resource, sending only the changes rather than the complete representation.

**Technical Definition:** Defined in RFC 5789, `PATCH` is neither safe nor idempotent, though it can be made idempotent depending on the patch document format. In Flask, `PATCH` is handled identically to `POST` and `PUT` for payload parsing; the method must be explicitly listed in the route's `methods` parameter.

**Beginner-Friendly Explanation:** PATCH is like PUT, but instead of replacing the whole resource, you only send the parts you want to change. For example, if you only want to update a user's email, you PATCH just the email field.

### Purposes

- To update only specific fields of a resource without resending the entire representation.
- To reduce payload size for large resources.
- To avoid overwriting fields that the client did not intend to change.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/resource/<int:resource_id>", methods=["PATCH"])
def patch_resource(resource_id):
    data = request.get_json()
    # Apply partial update from data
    return jsonify({"id": resource_id, "patched": True})
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=["PATCH"]` | Restricts route to PATCH only |
| `request.get_json()` | Parses the partial update payload |
| `200` or `204` | Typical success status codes |

**Syntax Rules:**

- `PATCH` must be explicitly listed in `methods`; it is not a default method.
- The request body should contain only the fields to be modified.
- RFC 5789 does not mandate a specific patch document format; JSON Merge Patch (RFC 7386) and JSON Patch (RFC 6902) are common choices.
- `PATCH` is not idempotent in general, though specific patch formats can be designed to be.

**Constraints and Limitations:**

- Some HTTP proxies and clients do not support `PATCH`; method overriding may be required.
- Without a standard patch format, clients and servers must agree on semantics.
- `PATCH` responses are not cacheable.

### Annotated Code Examples

**Example 1: Partial Update of a User**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

users = {
    1: {"name": "Alice", "email": "alice@example.com", "age": 30}
}

@app.route("/api/users/<int:user_id>", methods=["PATCH"])
def patch_user(user_id):
    if user_id not in users:
        return jsonify({"error": "User not found"}), 404
    
    data = request.get_json()
    if not data:
        return jsonify({"error": "No JSON body"}), 400
    
    # Apply only provided fields
    user = users[user_id]
    for key in data:
        if key in user:
            user[key] = data[key]
    
    return jsonify({"id": user_id, "user": user}), 200
```

**Expected Output:**
- `PATCH /api/users/1` with `{"email": "new@example.com"}` → `{"id": 1, "user": {"name": "Alice", "email": "new@example.com", "age": 30}}`
- `PATCH /api/users/1` with `{"age": 31, "name": "Alice Smith"}` → updates both fields.

**Why this output:** The handler iterates over the provided keys and updates only those fields. Fields not present in the request body remain unchanged, demonstrating the partial-update semantics of PATCH.

**Example 2: PATCH vs PUT Comparison**

```python
@app.route("/api/users/<int:user_id>", methods=["PUT"])
def replace_user(user_id):
    data = request.get_json()
    users[user_id] = data  # Complete replacement
    return jsonify(users[user_id])

@app.route("/api/users/<int:user_id>", methods=["PATCH"])
def patch_user(user_id):
    data = request.get_json()
    users[user_id].update(data)  # Partial update
    return jsonify(users[user_id])
```

**Expected Output:**
- `PUT` with `{"name": "Alice"}` → user becomes `{"name": "Alice"}` (email and age lost).
- `PATCH` with `{"name": "Alice"}` → user becomes `{"name": "Alice", "email": "...", "age": 30}` (other fields preserved).

**Why this output:** `PUT` replaces the entire resource, while `PATCH` merges the provided fields into the existing resource. This is the core semantic difference between the two methods.

### Real-World Cases

- **Profile updates:** A user changes only their phone number via `PATCH /users/me`.
- **Inventory adjustments:** `PATCH /products/42` with `{"stock": 15}` updates only the stock field.
- **Settings toggles:** `PATCH /settings` with `{"notifications": false}` changes one preference.

### References

- RFC 5789: PATCH Method for HTTP — https://datatracker.ietf.org/doc/rfc5789/
- RFC 7386: JSON Merge Patch — https://www.rfc-editor.org/rfc/rfc7386
- RFC 6902: JSON Patch — https://www.rfc-editor.org/rfc/rfc6902

---

## 5. DELETE (Resource Removal)

### Definitions

**Core Definition:** The `DELETE` method requests that the server remove the resource identified by the request URI. It is idempotent but not safe.

**Technical Definition:** Per RFC 9110, `DELETE` is not safe but is idempotent; repeating the same `DELETE` request has the same effect as a single request (the resource is removed or already absent). In Flask, `DELETE` is handled with `methods=["DELETE"]` and typically does not have a request body.

**Beginner-Friendly Explanation:** DELETE removes a resource from the server. If you delete something twice, the second attempt simply finds nothing to delete, which is fine—the result is the same.

### Purposes

- To remove a resource from the server permanently.
- To provide idempotent removal semantics (repeated calls do not cause errors).
- To clean up resources in CRUD APIs.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/resource/<int:resource_id>", methods=["DELETE"])
def delete_resource(resource_id):
    # Remove resource with resource_id
    return "", 204
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=["DELETE"]` | Restricts route to DELETE only |
| `<int:resource_id>` | URL variable identifying the resource to remove |
| `204 No Content` | Typical success status (resource removed, no body) |
| `404 Not Found` | Returned if the resource did not exist |

**Syntax Rules:**

- `DELETE` requests typically have no body; the resource is identified by the URL.
- A successful `DELETE` returns `200` (with a body) or `204` (no body); a missing resource should return `404`.
- Because `DELETE` is idempotent, deleting an already-deleted resource may return `404` or `204` depending on API design; both are acceptable.

**Constraints and Limitations:**

- `DELETE` is not safe; it modifies server state permanently.
- Some clients and proxies do not support `DELETE`; method overriding may be required.
- Soft deletes (marking as deleted) vs. hard deletes (physical removal) must be chosen based on business requirements.

### Annotated Code Examples

**Example 1: Deleting a Resource**

```python
from flask import Flask, jsonify

app = Flask(__name__)

users = {
    1: {"name": "Alice"},
    2: {"name": "Bob"}
}

@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    if user_id not in users:
        return jsonify({"error": "User not found"}), 404
    
    del users[user_id]
    return "", 204
```

**Expected Output:**
- `DELETE /api/users/1` → status `204 No Content`; user 1 removed from `users`.
- `DELETE /api/users/1` again → status `404 Not Found`.

**Why this output:** The first `DELETE` removes the user and returns `204`. The second `DELETE` finds no user and returns `404`. The operation is idempotent in effect—the resource is gone either way—but the status code reflects whether the resource existed at the time of the request.

**Example 2: Soft Delete Pattern**

```python
@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def soft_delete_user(user_id):
    user = users.get(user_id)
    if not user:
        return jsonify({"error": "Not found"}), 404
    user["deleted"] = True
    return jsonify({"message": "User deactivated"}), 200
```

**Expected Output:**
- `DELETE /api/users/1` → `{"message": "User deactivated"}` with status `200`.
- The user remains in the data store but is marked as deleted.

**Why this output:** Soft deletion marks the resource as inactive rather than physically removing it. This is common in applications that need audit trails or the ability to restore deleted resources.

### Real-World Cases

- **REST API deletes:** `DELETE /api/posts/42` removes a blog post.
- **Account deletion:** `DELETE /api/users/me` deactivates or removes the current user's account.
- **Cache invalidation:** `DELETE /cache/key` removes a cached entry.

### References

- RFC 9110: HTTP Semantics (Section 9.3.5) — https://www.rfc-editor.org/rfc/rfc9110
- Flask API: `request.method` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.method

---

## 6. Multiple Methods on One Route (Conditional Logic)

### Definitions

**Core Definition:** A single route can accept multiple HTTP methods by listing them in the `methods` parameter, with the view function using `request.method` to branch on the incoming method.

**Technical Definition:** The `methods` parameter accepts a list of method strings. The resulting Werkzeug `Rule` matches any of those methods. Inside the view, `request.method` returns the method string (e.g., `"GET"`, `"POST"`), allowing conditional execution.

**Beginner-Friendly Explanation:** Instead of creating separate routes for GET and POST, you can use one route and check `request.method` inside the function to decide what to do. This is common for form pages that display a form on GET and process it on POST.

### Purposes

- To handle multiple HTTP methods with shared URL logic (e.g., form display and submission).
- To reduce the number of route definitions for related operations.
- To implement RESTful endpoints where the same URL supports different actions based on method.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/endpoint", methods=["GET", "POST", "PUT", "DELETE"])
def multi_method():
    if request.method == "GET":
        return "Retrieving"
    elif request.method == "POST":
        return "Creating"
    elif request.method == "PUT":
        return "Replacing"
    elif request.method == "DELETE":
        return "Deleting"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods=[...]` | List of accepted methods |
| `request.method` | String indicating the current request's method |
| `if/elif` chain | Branches execution based on method |

**Syntax Rules:**

- The `methods` list must contain valid HTTP method strings (uppercase).
- `request.method` is always uppercase (e.g., `"GET"`, `"POST"`).
- If the incoming method is not in the `methods` list, Flask returns `405 Method Not Allowed`.
- The order of methods in the list does not affect routing behavior.

**Constraints and Limitations:**

- Long `if/elif` chains can become unwieldy; class-based views (`MethodView`) provide a cleaner alternative for complex cases.
- All methods share the same URL and endpoint; `url_for()` does not distinguish between methods.
- Method-specific error handling must be implemented manually within the view.

### Annotated Code Examples

**Example 1: Form Display and Submission**

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

FORM_HTML = """
<form method="POST">
    <input name="name" placeholder="Your name">
    <button type="submit">Submit</button>
</form>
"""

@app.route("/form", methods=["GET", "POST"])
def form():
    if request.method == "POST":
        name = request.form.get("name", "Anonymous")
        return f"Hello, {name}!"
    return render_template_string(FORM_HTML)
```

**Expected Output:**
- `GET /form` → the HTML form.
- `POST /form` with `name=Alice` → `"Hello, Alice!"`.

**Why this output:** The route accepts both GET and POST. On GET, the form is rendered. On POST, the submitted data is processed and a personalized greeting is returned.

**Example 2: REST-Style Resource Handler**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

resources = {1: "Resource one"}

@app.route("/api/resources/<int:rid>", methods=["GET", "PUT", "DELETE"])
def resource_handler(rid):
    if request.method == "GET":
        if rid not in resources:
            return jsonify({"error": "Not found"}), 404
        return jsonify({"id": rid, "value": resources[rid]})
    
    elif request.method == "PUT":
        data = request.get_json()
        resources[rid] = data.get("value", "")
        return jsonify({"id": rid, "value": resources[rid]}), 200
    
    elif request.method == "DELETE":
        resources.pop(rid, None)
        return "", 204
```

**Expected Output:**
- `GET /api/resources/1` → `{"id": 1, "value": "Resource one"}`
- `PUT /api/resources/1` with `{"value": "Updated"}` → `{"id": 1, "value": "Updated"}`
- `DELETE /api/resources/1` → `204 No Content`

**Why this output:** The single route handles three methods on the same URL. The `request.method` check directs execution to the appropriate branch. This is a compact way to implement a RESTful resource endpoint.

### Real-World Cases

- **Form pages:** GET displays the form; POST processes it.
- **REST resources:** GET retrieves, PUT replaces, DELETE removes the same resource.
- **Webhook endpoints:** POST receives events; GET provides a health check.

### References

- Flask Quickstart: HTTP Methods — https://flask.palletsprojects.com/en/stable/quickstart/#http-methods
- Flask API: `request.method` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.method

---

## 7. Method-Specific Behavior (HEAD, OPTIONS, and CORS Preflight)

### Definitions

**Core Definition:** Flask automatically handles `HEAD` and `OPTIONS` requests for routes that accept `GET`. `HEAD` returns headers without a body; `OPTIONS` returns the allowed methods. CORS preflight requests use `OPTIONS` and require specific response headers.

**Technical Definition:** If `GET` is present in a route's methods, Flask adds `HEAD` implicitly. Since Flask 0.6, `OPTIONS` is added automatically and handled by standard request dispatching; the response includes an `Allow` header listing valid methods. The `provide_automatic_options` attribute on a view function can disable or force-enable this behavior. Flask-CORS can override Flask's default `OPTIONS` handling to return CORS headers for preflight requests.

**Beginner-Friendly Explanation:** When your browser sends a HEAD request, it wants the same information as GET but without the body (useful for checking if a page exists). When it sends OPTIONS, it is asking “what methods does this URL support?” Flask answers these automatically. For cross-origin requests, the browser sends an OPTIONS preflight first, and Flask-CORS makes sure the right headers are returned.

### Purposes

- To automatically respond to `HEAD` requests with the same headers as `GET` but no body.
- To automatically respond to `OPTIONS` requests with an `Allow` header listing supported methods.
- To handle CORS preflight `OPTIONS` requests by returning `Access-Control-Allow-*` headers.
- To allow developers to disable or customize automatic `OPTIONS` handling per route.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, current_app

app = Flask(__name__)

@app.route("/resource")
def resource():
    return "Content"

# Disable automatic OPTIONS for a specific view
@app.route("/custom-options", provide_automatic_options=False, methods=["GET", "OPTIONS"])
def custom_options():
    if request.method == "OPTIONS":
        return "", 204, {"Allow": "GET, OPTIONS", "X-Custom": "value"}
    return "Custom"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `provide_automatic_options=False` | Disables Flask's automatic OPTIONS response |
| `methods=["GET", "OPTIONS"]` | Explicitly includes OPTIONS |
| `request.method == "OPTIONS"` | Branch for manual OPTIONS handling |
| `Allow` header | Lists the methods supported by the resource |

**Syntax Rules:**

- `HEAD` is automatically added when `GET` is in the methods list; do not add it manually.
- `OPTIONS` is automatically added unless explicitly listed in `methods` or disabled via `provide_automatic_options=False`.
- The automatic `OPTIONS` response has a `200` status and an `Allow` header.
- Flask-CORS's `automatic_options=True` (default) overrides Flask's default OPTIONS handling to include CORS headers.

**Constraints and Limitations:**

- Automatic `OPTIONS` handling does not include CORS headers; Flask-CORS or manual handling is required for cross-origin preflight.
- Disabling automatic `OPTIONS` means you must implement the response yourself, including the `Allow` header.
- `HEAD` requests must not return a body; Flask handles this automatically for routes that serve `GET`.

### Annotated Code Examples

**Example 1: Automatic HEAD and OPTIONS**

```python
from flask import Flask

app = Flask(__name__)

@app.route("/page")
def page():
    return "Page content"
```

**Expected Output:**
- `GET /page` → `"Page content"`
- `HEAD /page` → empty body with same headers as GET (e.g., `Content-Length` set)
- `OPTIONS /page` → `200 OK` with `Allow: GET, HEAD, OPTIONS`

**Why this output:** Flask adds `HEAD` because `GET` is present. `OPTIONS` is added automatically. The `OPTIONS` response includes the `Allow` header listing all supported methods.

**Example 2: Manual OPTIONS Handling with CORS Headers**

```python
from flask import Flask, request, make_response

app = Flask(__name__)

@app.route("/api/data", methods=["GET", "POST", "OPTIONS"])
def api_data():
    if request.method == "OPTIONS":
        response = make_response("", 204)
        response.headers["Allow"] = "GET, POST, OPTIONS"
        response.headers["Access-Control-Allow-Origin"] = "*"
        response.headers["Access-Control-Allow-Methods"] = "GET, POST, OPTIONS"
        response.headers["Access-Control-Allow-Headers"] = "Content-Type"
        return response
    
    if request.method == "POST":
        return "Data created"
    return "Data retrieved"
```

**Expected Output:**
- `OPTIONS /api/data` → `204 No Content` with `Allow`, `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers` headers.
- `GET /api/data` → `"Data retrieved"`
- `POST /api/data` → `"Data created"`

**Why this output:** The view explicitly handles `OPTIONS` and returns the CORS headers required for a preflight request. This is necessary when not using Flask-CORS.

**Example 3: Disabling Automatic OPTIONS**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/no-options", provide_automatic_options=False, methods=["GET"])
def no_options():
    return "No automatic OPTIONS"
```

**Expected Output:**
- `GET /no-options` → `"No automatic OPTIONS"`
- `OPTIONS /no-options` → `405 Method Not Allowed` (because OPTIONS is not in the methods list and automatic handling is disabled).

**Why this output:** With `provide_automatic_options=False`, Flask does not add `OPTIONS` to the rule. Since the route only accepts `GET`, an `OPTIONS` request returns `405`.

### Real-World Cases

- **CORS APIs:** Single-page applications hosted on a different origin require preflight `OPTIONS` handling with CORS headers.
- **API discovery:** Clients send `OPTIONS` to discover supported methods before sending a request.
- **Health checks:** Load balancers send `HEAD` requests to check if the server is alive without transferring the full response body.
- **Flask-CORS:** Automatically handles preflight requests when `CORS(app)` is configured.

### References

- Flask API: `provide_automatic_options` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.add_url_rule
- Flask-CORS Configuration — https://flask-cors.readthedocs.io/en/3.0.10/configuration.html
- MDN: CORS Preflight — https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests

---

## 8. HTTP Method Overriding (Headers and Form Fields)

### Definitions

**Core Definition:** HTTP method overriding allows clients that cannot send certain HTTP methods (e.g., `PUT`, `PATCH`, `DELETE`) to tunnel those methods through `POST` using a header or form field.

**Technical Definition:** Some HTTP proxies and clients do not support arbitrary or newer HTTP methods. Flask documents a middleware pattern where the client sends an HTTP `POST` with the `X-HTTP-Method-Override` header set to the desired method; middleware replaces `REQUEST_METHOD` in the WSGI environ before Flask processes the request. A form-field approach using `_method` is also common but requires middleware to read the form body and rewrite the method.

**Beginner-Friendly Explanation:** If your browser or a proxy won't let you send a DELETE or PUT request, you can send a POST instead and include a special header or hidden form field saying “actually, treat this as DELETE.” Flask middleware then rewrites the request before your view sees it.

### Purposes

- To work around proxies, firewalls, or clients that block non-GET/POST methods.
- To enable `PUT`, `PATCH`, and `DELETE` in HTML forms, which only support GET and POST natively.
- To maintain RESTful API semantics while accommodating limited client environments.
- To provide a fallback mechanism for older browsers.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request
from werkzeug.wrappers import Request

class HTTPMethodOverrideMiddleware:
    allowed_methods = frozenset([
        "GET", "HEAD", "POST", "DELETE", "PUT", "PATCH", "OPTIONS"
    ])
    bodyless_methods = frozenset(["GET", "HEAD", "OPTIONS", "DELETE"])
    
    def __init__(self, app):
        self.app = app
    
    def __call__(self, environ, start_response):
        method = environ.get("HTTP_X_HTTP_METHOD_OVERRIDE", "").upper()
        if method in self.allowed_methods:
            environ["REQUEST_METHOD"] = method
            if method in self.bodyless_methods:
                environ["CONTENT_LENGTH"] = "0"
        return self.app(environ, start_response)

app = Flask(__name__)
app.wsgi_app = HTTPMethodOverrideMiddleware(app.wsgi_app)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `X-HTTP-Method-Override` | Header carrying the desired method |
| `HTTP_X_HTTP_METHOD_OVERRIDE` | WSGI environ key for the header |
| `allowed_methods` | Whitelist of methods that can be overridden |
| `bodyless_methods` | Methods that should not carry a body |
| `CONTENT_LENGTH = "0"` | Clears body length for bodyless methods |
| `app.wsgi_app` | Wrapping the WSGI application with middleware |

**Syntax Rules:**

- The middleware reads the header before Flask's routing logic.
- Only methods in `allowed_methods` are accepted; others are ignored.
- For bodyless methods (GET, HEAD, OPTIONS, DELETE), `CONTENT_LENGTH` is set to `0` to prevent body-parsing errors.
- The form-field `_method` approach requires reading `request.form` in middleware, which can conflict with later form access.

**Constraints and Limitations:**

- Method overriding is a protocol violation; it should only be used when necessary.
- Flask does not support method overriding natively; middleware must be added manually.
- Security risk: Allowing arbitrary method overrides can expose endpoints to unexpected methods. The `allowed_methods` whitelist mitigates this.
- The form-field `_method` approach can cause request hanging if the view later accesses `request.form`.

### Annotated Code Examples

**Example 1: Header-Based Method Override**

```python
from flask import Flask, request

class HTTPMethodOverrideMiddleware:
    allowed_methods = frozenset([
        "GET", "HEAD", "POST", "DELETE", "PUT", "PATCH", "OPTIONS"
    ])
    bodyless_methods = frozenset(["GET", "HEAD", "OPTIONS", "DELETE"])
    
    def __init__(self, app):
        self.app = app
    
    def __call__(self, environ, start_response):
        method = environ.get("HTTP_X_HTTP_METHOD_OVERRIDE", "").upper()
        if method in self.allowed_methods:
            environ["REQUEST_METHOD"] = method
            if method in self.bodyless_methods:
                environ["CONTENT_LENGTH"] = "0"
        return self.app(environ, start_response)

app = Flask(__name__)
app.wsgi_app = HTTPMethodOverrideMiddleware(app.wsgi_app)

@app.route("/api/resource/1", methods=["PUT"])
def update_resource():
    return f"Method was: {request.method}"

# Client sends:
# POST /api/resource/1
# X-HTTP-Method-Override: PUT
```

**Expected Output:**
- `POST /api/resource/1` with header `X-HTTP-Method-Override: PUT` → `"Method was: PUT"`.
- `POST /api/resource/1` without the header → `405 Method Not Allowed` (because the route only accepts PUT).

**Why this output:** The middleware reads the override header and sets `REQUEST_METHOD` to `PUT`. Flask's routing then matches the PUT route. Without the header, the request remains POST and no matching route exists.

**Example 2: Form-Field `_method` Override**

```python
from flask import Flask, request
from werkzeug.wrappers import Request

class FormMethodOverrideMiddleware:
    def __init__(self, app):
        self.app = app
    
    def __call__(self, environ, start_response):
        request = Request(environ)
        if request.method == "POST" and "_method" in request.form:
            method = request.form["_method"].upper()
            if method in ("PUT", "PATCH", "DELETE"):
                environ["REQUEST_METHOD"] = method
        return self.app(environ, start_response)

app = Flask(__name__)
app.wsgi_app = FormMethodOverrideMiddleware(app.wsgi_app)

@app.route("/form-resource", methods=["DELETE"])
def delete_via_form():
    return "Deleted via form override!"
```

**Expected Output:**
- An HTML form with `<input type="hidden" name="_method" value="DELETE">` submitted via POST → triggers the DELETE route.
- The view returns `"Deleted via form override!"`.

**Why this output:** The middleware reads the form field `_method` from the POST body and rewrites `REQUEST_METHOD` before Flask routes the request. This allows HTML forms to trigger DELETE (or PUT/PATCH) routes.

### Real-World Cases

- **HTML form deletions:** A table row with a “Delete” button sends a POST with `_method=DELETE` because HTML forms cannot send DELETE directly.
- **Corporate proxies:** Some corporate networks block PUT and DELETE; method override via POST header keeps APIs working.
- **Older browsers:** Internet Explorer and some mobile browsers do not support `fetch` with arbitrary methods; override provides a fallback.

### References

- Flask Documentation: Adding HTTP Method Overrides — https://flask.palletsprojects.com/en/stable/patterns/methodoverrides/
- WSGI environ specification — https://peps.python.org/pep-3333/
- Stack Overflow: Method override with `_method` — https://stackoverflow.com/questions/17750075/

---

## References

- RFC 9110: HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 5789: PATCH Method for HTTP — https://datatracker.ietf.org/doc/rfc5789/
- RFC 7386: JSON Merge Patch — https://www.rfc-editor.org/rfc/rfc7386
- RFC 6902: JSON Patch — https://www.rfc-editor.org/rfc/rfc6902
- Flask Quickstart: HTTP Methods — https://flask.palletsprojects.com/en/stable/quickstart/#http-methods
- Flask API: `Flask.add_url_rule` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.add_url_rule
- Flask API: `request.method` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.method
- Flask API: `request.get_json` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.get_json
- Flask Documentation: Method Overrides — https://flask.palletsprojects.com/en/stable/patterns/methodoverrides/
- Flask-CORS Configuration — https://flask-cors.readthedocs.io/en/3.0.10/configuration.html
- MDN: HTTP Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN: CORS Preflight — https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests