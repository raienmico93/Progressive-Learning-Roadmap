# Flask Status Codes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP status codes are three-digit integers returned by a server in an HTTP response to indicate the outcome of a client's request. In Flask, status codes are set on `Response` objects and can be returned from view functions via tuples, `make_response()`, or `abort()`.

**Technical Definition:** HTTP status codes are defined by RFC 9110 (HTTP Semantics) and maintained by the IANA HTTP Status Code Registry. They are grouped into five classes: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error). In Flask, the default status code for a response is `200 OK`. Developers can override this by returning a tuple `(body, status)`, using `make_response()`, setting `response.status_code`, or calling `abort(status_code)`. Flask's `abort()` function raises a `werkzeug.exceptions.HTTPException` subclass corresponding to the status code, which is then handled by the registered error handler or the default Werkzeug error page.

**Beginner-Friendly Explanation:** Every time a browser or API client talks to your Flask app, the server replies with a status code—a number that says whether everything went fine (200), something new was created (201), the client made a mistake (400), or the server had a problem (500). Flask lets you set these codes easily so clients know exactly what happened.

### Key Characteristics

- **Five classes:** 1xx, 2xx, 3xx, 4xx, 5xx, each indicating a different outcome category.
- **Default is 200:** Any view that returns a value without an explicit status code produces `200 OK`.
- **Multiple ways to set:** Tuples `(body, status)`, `make_response()`, `response.status_code`, or `abort()`.
- **Error handling integration:** `abort()` raises exceptions that can be caught by `@app.errorhandler()`.
- **Standardized payloads:** Custom error handlers can standardize JSON error responses across all codes.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTTP request/response cycle.
- Familiarity with Flask view functions, `Response` objects, and error handling.
- Knowledge of Python tuples and exception handling.

### Related Programming Areas

- **REST API design:** Status codes are the primary mechanism for communicating outcomes in APIs.
- **Error handling:** `abort()` and `@app.errorhandler()` are the core tools for error responses.
- **Redirects:** 301/302 codes drive browser navigation.
- **Validation:** 400 and 422 codes signal client input problems.
- **Security:** 401 and 403 codes enforce authentication and authorization.

### Core Concepts / Features

1. 200 OK (Successful Retrieval or Modification)
2. 201 Created (Resource Generation with Location Header)
3. 204 No Content (Successful Actions Without a Body)
4. 301/302 Redirects (Permanent vs. Temporary Routing)
5. 400 Bad Request (Client-Side Syntax Errors)
6. 401 Unauthorized (Missing or Invalid Credentials)
7. 403 Forbidden (Authenticated but Lacking Permissions)
8. 404 Not Found (Target Resource Missing)
9. 409 Conflict (State Violations)
10. 422 Unprocessable Content (Semantically Invalid Data)
11. 500 Internal Server Error (Unhandled Server Exceptions)
12. Global Error Payload Standardization

---

## 1. 200 OK (Successful Retrieval or Modification)

### Definitions

**Core Definition:** `200 OK` indicates that the request succeeded and the response body contains the requested representation or the result of the action.

**Technical Definition:** `200 OK` is the default status code for successful HTTP requests. In Flask, any view function that returns a value without an explicit status code produces `200 OK`. The response body may contain HTML, JSON, plain text, or binary data. For `GET` requests, the body contains the resource; for `PUT` and `PATCH`, it may contain the updated representation or a status message.

**Beginner-Friendly Explanation:** 200 means everything worked. The server is giving you what you asked for.

### Purposes

- To indicate a successful resource retrieval.
- To confirm a successful modification (PUT, PATCH).
- To return data to the client after a successful request.
- To serve as the default status code when no other code is specified.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Implicit 200 (default)
@app.route("/")
def index():
    return "Hello"

# Explicit 200 via tuple
@app.route("/explicit")
def explicit():
    return "Hello", 200

# Explicit 200 via make_response
from flask import make_response

@app.route("/make")
def make():
    resp = make_response("Hello")
    resp.status_code = 200
    return resp
```

**Component Breakdown:**

| Approach | Syntax |
|----------|--------|
| Implicit | `return "body"` |
| Tuple | `return "body", 200` |
| `make_response` | `resp = make_response("body"); resp.status_code = 200` |

**Syntax Rules:**

- The default status code for any Flask view is `200 OK`.
- `200` can be used explicitly with tuples or `make_response()`.
- The response body can be a string, dict, list, or `Response` object.

**Constraints and Limitations:**

- For successful creation, prefer `201 Created` over `200 OK`.
- For successful actions with no body, prefer `204 No Content` over `200 OK`.

### Annotated Code Examples

**Example 1: Implicit 200 Response**

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Welcome!"

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /` → `"Welcome!"` with status `200 OK`.

**Why this output:** No status code is specified, so Flask uses the default `200 OK`. The string body is encoded and sent to the client.

**Example 2: Explicit 200 with `make_response`**

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route("/data")
def data():
    resp = make_response({"key": "value"})
    resp.status_code = 200
    return resp
```

**Expected Output:**
- `GET /data` → `{"key": "value"}` with status `200 OK`.

**Why this output:** `make_response()` converts the dictionary to a JSON response. The status code is explicitly set to `200`.

### Real-World Cases

- **API resource retrieval:** `GET /api/users/42` returns the user with `200 OK`.
- **Successful updates:** `PUT /api/users/42` returns the updated user with `200 OK`.
- **Health checks:** `GET /health` returns `"OK"` with `200 OK`.

### References

- RFC 9110: 200 OK — https://www.rfc-editor.org/rfc/rfc9110#section-15.3.1
- Flask `make_response` — https://flask.palletsprojects.com/en/stable/api/#flask.make_response

---

## 2. 201 Created (Resource Generation with Location Header)

### Definitions

**Core Definition:** `201 Created` indicates that the request succeeded and a new resource was created as a result. The `Location` header in the response identifies the URL of the newly created resource.

**Technical Definition:** `201 Created` is used with `POST` (or `PUT`) requests that create a resource. The `Location` header must contain an absolute URI identifying the new resource, per RFC 9110. Flask/Werkzeug automatically converts relative `Location` headers to absolute URLs. The `autocorrect_location_header` attribute on the `Response` object can disable this behavior if needed. The response body may contain a representation of the created resource.

**Beginner-Friendly Explanation:** 201 means "I created something new, and here's where you can find it." The `Location` header tells the client the URL of the new resource.

### Purposes

- To confirm successful creation of a new resource.
- To provide the client with the URL of the newly created resource.
- To return a representation of the created resource for immediate use.
- To follow RESTful conventions for `POST` requests.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import jsonify, url_for

@app.route("/users", methods=["POST"])
def create_user():
    # Create the resource
    user = {"id": 1, "name": "Alice"}
    response = jsonify(user)
    response.status_code = 201
    response.headers["Location"] = url_for("get_user", user_id=1)
    return response
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `response.status_code = 201` | Sets the status code |
| `response.headers["Location"]` | Sets the Location header |
| `url_for("get_user", user_id=1)` | Generates the URL of the new resource |

**Syntax Rules:**

- Use `201` for successful `POST` requests that create a resource.
- Include a `Location` header with the absolute URL of the new resource.
- The response body may contain the created resource's representation.
- Werkzeug automatically converts relative `Location` headers to absolute URLs.

**Constraints and Limitations:**

- If no `Location` header is provided, the client may not know where the resource was created.
- Some APIs return `200 OK` for creation; `201` is more semantically correct.

### Annotated Code Examples

**Example 1: Creating a User with 201 and Location**

```python
from flask import Flask, jsonify, url_for

app = Flask(__name__)

@app.route("/users", methods=["POST"])
def create_user():
    # Simulate creating a user
    user = {"id": 1, "name": "Alice"}
    response = jsonify(user)
    response.status_code = 201
    response.headers["Location"] = url_for("get_user", user_id=1)
    return response

@app.route("/users/<int:user_id>")
def get_user(user_id):
    return jsonify({"id": user_id, "name": "Alice"})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /users` → `{"id": 1, "name": "Alice"}` with status `201 Created` and header `Location: http://localhost/users/1`.

**Why this output:** The view creates a user, sets the status code to `201`, and sets the `Location` header to the URL of the new resource. The `url_for()` function generates the correct URL.

### Real-World Cases

- **User registration:** `POST /users` creates a new account and returns `201` with the profile URL.
- **Order creation:** `POST /orders` creates an order and returns `201` with the order URL.
- **Resource upload:** `POST /files` creates a file and returns `201` with the file URL.

### References

- RFC 9110: 201 Created — https://www.rfc-editor.org/rfc/rfc9110#section-15.3.2
- Flask `url_for` — https://flask.palletsprojects.com/en/stable/api/#flask.url_for

---

## 3. 204 No Content (Successful Actions Without a Body)

### Definitions

**Core Definition:** `204 No Content` indicates that the request succeeded but the server does not need to return a response body. This is commonly used for `DELETE` requests.

**Technical Definition:** `204 No Content` is a 2xx status code that signals successful processing without returning content. The response must not include a message body. In Flask, returning an empty string with status `204` or using `make_response("", 204)` produces the correct response. Flask/Werkzeug automatically omits the body for `204` responses.

**Beginner-Friendly Explanation:** 204 means "I did what you asked, but there's nothing to show you." It's commonly used after deleting something.

### Purposes

- To confirm successful deletion without returning the deleted resource.
- To acknowledge successful `PUT` or `PATCH` requests where no body is needed.
- To reduce bandwidth by omitting unnecessary response bodies.
- To follow RESTful conventions for `DELETE` operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
@app.route("/resource/<int:id>", methods=["DELETE"])
def delete_resource(id):
    # Delete the resource
    return "", 204
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `return "", 204` | Empty body with 204 status |
| `make_response("", 204)` | Alternative using make_response |

**Syntax Rules:**

- Use `204` when the action succeeded and no body is needed.
- The response body must be empty.
- `204` can be returned via tuple or `make_response()`.

**Constraints and Limitations:**

- Do not include a body with `204`; some clients may discard it.
- For deletions where you want to return a confirmation message, use `200 OK` instead.

### Annotated Code Examples

**Example 1: Deleting a Resource**

```python
from flask import Flask, jsonify

app = Flask(__name__)
resources = {1: "Item 1", 2: "Item 2"}

@app.route("/resources/<int:resource_id>", methods=["DELETE"])
def delete_resource(resource_id):
    if resource_id not in resources:
        return jsonify({"error": "Not found"}), 404
    del resources[resource_id]
    return "", 204

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `DELETE /resources/1` → empty body with status `204 No Content`.
- `DELETE /resources/99` → `{"error": "Not found"}` with status `404`.

**Why this output:** The resource is deleted, and the server returns `204` to indicate success without a body. If the resource does not exist, a `404` is returned.

### Real-World Cases

- **User deletion:** `DELETE /users/42` returns `204 No Content`.
- **Cart item removal:** `DELETE /cart/items/7` returns `204 No Content`.
- **Unsubscribe:** `DELETE /subscriptions/123` returns `204 No Content`.

### References

- RFC 9110: 204 No Content — https://www.rfc-editor.org/rfc/rfc9110#section-15.3.5
- Flask Quickstart: About Responses — https://flask.palletsprojects.com/en/stable/quickstart/#about-responses

---

## 4. 301/302 Redirects (Permanent vs. Temporary Routing)

### Definitions

**Core Definition:** 301 and 302 are 3xx status codes that instruct the client's browser to navigate to a different URL. `301 Moved Permanently` indicates a permanent change, while `302 Found` indicates a temporary change.

**Technical Definition:** `301 Moved Permanently` tells the client that the resource has permanently moved to the URL in the `Location` header; browsers and search engines should update their records. `302 Found` tells the client that the resource is temporarily available at a different URL; the original URL should continue to be used. In Flask, both are produced using `redirect(location, code=301)` or `redirect(location, code=302)` (302 is the default).

**Beginner-Friendly Explanation:** A redirect tells the browser "go to this other URL instead." If it's permanent (301), the browser remembers the new URL. If it's temporary (302), the browser keeps using the old URL next time.

### Purposes

- To redirect users after login or logout (temporary).
- To redirect old URLs to new ones after a site restructure (permanent).
- To enforce HTTPS by redirecting HTTP requests (permanent or temporary).
- To redirect after form submission to prevent duplicate submissions (temporary).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import redirect, url_for

# Temporary redirect (302, default)
@app.route("/old")
def old():
    return redirect(url_for("new"))

# Permanent redirect (301)
@app.route("/legacy")
def legacy():
    return redirect(url_for("new"), code=301)
```

**Component Breakdown:**

| Code | Meaning | Use Case |
|------|---------|----------|
| 301 | Permanent | Site restructure, HTTPS enforcement |
| 302 | Temporary | Login redirect, form submission |

**Syntax Rules:**

- `redirect(location, code=302)` is the default.
- Use `code=301` for permanent redirects.
- The `Location` header must be an absolute or relative URL.
- Flask automatically converts relative URLs to absolute for redirects.

**Constraints and Limitations:**

- 301 redirects are cached by browsers; use with caution if the destination may change.
- 302 redirects are not cached, making them safer for temporary changes.

### Annotated Code Examples

**Example 1: Temporary Redirect After Login**

```python
from flask import Flask, redirect, url_for

app = Flask(__name__)

@app.route("/login")
def login():
    # After successful login, redirect to dashboard
    return redirect(url_for("dashboard"))

@app.route("/dashboard")
def dashboard():
    return "Dashboard"

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /login` → `302 Found` redirect to `/dashboard`.

**Why this output:** `redirect()` with the default code (`302`) tells the browser to navigate to the dashboard URL temporarily.

**Example 2: Permanent Redirect for Legacy URL**

```python
@app.route("/old-page")
def old_page():
    return redirect(url_for("new_page"), code=301)

@app.route("/new-page")
def new_page():
    return "New Page"
```

**Expected Output:**
- `GET /old-page` → `301 Moved Permanently` redirect to `/new-page`.

**Why this output:** The `code=301` parameter makes the redirect permanent. Browsers and search engines will update their records to point to `/new-page`.

### Real-World Cases

- **Login/logout redirects:** Temporarily redirect users after authentication.
- **Site migration:** Permanently redirect old URLs to new ones.
- **HTTPS enforcement:** Redirect HTTP requests to HTTPS.
- **Trailing slash canonicalization:** Redirect `/about` to `/about/` or vice versa.

### References

- Flask `redirect` — https://flask.palletsprojects.com/en/stable/api/#flask.redirect
- RFC 9110: 301 Moved Permanently — https://www.rfc-editor.org/rfc/rfc9110#section-15.4.2
- RFC 9110: 302 Found — https://www.rfc-editor.org/rfc/rfc9110#section-15.4.3

---

## 5. 400 Bad Request (Client-Side Syntax Errors)

### Definitions

**Core Definition:** `400 Bad Request` indicates that the server cannot process the request because of a client-side error, such as malformed syntax, invalid request framing, or deceptive request routing.

**Technical Definition:** `400 Bad Request` is a 4xx status code indicating that the client's request is malformed or cannot be understood by the server. In Flask, `400` is raised automatically by `request.get_json()` when the body is invalid JSON, or can be raised explicitly with `abort(400)`. The response body may contain details about the error, but should not expose internal implementation details.

**Beginner-Friendly Explanation:** 400 means "I can't understand your request—something is wrong with what you sent." This could be malformed JSON, missing required headers, or bad syntax.

### Purposes

- To signal that the request body is malformed or unparseable.
- To indicate missing required headers or parameters.
- To reject requests with invalid query strings or form data.
- To provide clear feedback to API clients about input errors.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort, jsonify

# Using abort
@app.route("/validate", methods=["POST"])
def validate():
    data = request.get_json(silent=True)
    if data is None:
        abort(400, description="Invalid JSON body")
    return jsonify(data)

# Returning a tuple
@app.route("/check")
def check():
    if not request.args.get("id"):
        return jsonify({"error": "Missing id parameter"}), 400
    return "OK"
```

**Component Breakdown:**

| Approach | Syntax |
|----------|--------|
| `abort(400)` | Raises HTTPException; handled by error handler |
| `return body, 400` | Returns response with 400 status |
| `return jsonify(...), 400` | JSON error response |

**Syntax Rules:**

- Use `400` for syntax errors, not semantic errors (use `422` for those).
- `abort(400)` can include a description.
- Custom error handlers can format the 400 response.

**Constraints and Limitations:**

- Do not expose internal error details in 400 responses.
- Distinguish between 400 (syntax) and 422 (semantic).

### Annotated Code Examples

**Example 1: Handling Malformed JSON**

```python
from flask import Flask, request, jsonify, abort

app = Flask(__name__)

@app.route("/api/data", methods=["POST"])
def receive_data():
    data = request.get_json(silent=True)
    if data is None:
        abort(400, description="Request body must be valid JSON")
    return jsonify({"received": data})

@app.errorhandler(400)
def bad_request(error):
    return jsonify({"error": str(error.description)}), 400

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /api/data` with invalid JSON → `{"error": "Request body must be valid JSON"}` with status `400`.
- `POST /api/data` with valid JSON → `{"received": {...}}`.

**Why this output:** `request.get_json(silent=True)` returns `None` for invalid JSON. The view calls `abort(400)` with a descriptive message, which is caught by the custom error handler and returned as JSON.

### Real-World Cases

- **API endpoints:** Rejecting malformed JSON payloads.
- **Form submissions:** Handling missing required fields.
- **Query string validation:** Rejecting invalid query parameters.

### References

- RFC 9110: 400 Bad Request — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.1
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/

---

## 6. 401 Unauthorized (Missing or Invalid Credentials)

### Definitions

**Core Definition:** `401 Unauthorized` indicates that the request requires authentication and the client has not provided valid credentials. Despite the name, it is about authentication, not authorization.

**Technical Definition:** `401 Unauthorized` is a 4xx status code that indicates the client must authenticate itself to get the requested response. The response must include a `WWW-Authenticate` header listing the authentication schemes the server supports. In Flask, `401` is typically raised with `abort(401)` when authentication is missing or invalid.

**Beginner-Friendly Explanation:** 401 means "You need to log in first" or "Your username/password is wrong." It's about proving who you are.

### Purposes

- To signal that authentication is required.
- To indicate that provided credentials are invalid or expired.
- To challenge the client to provide credentials via `WWW-Authenticate`.
- To protect endpoints that require login.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort, jsonify, request

@app.route("/protected")
def protected():
    auth = request.headers.get("Authorization")
    if not auth:
        abort(401, description="Authentication required")
    # Validate credentials
    return "Access granted"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(401)` | Raises 401 error |
| `WWW-Authenticate` | Response header indicating auth scheme |
| `Authorization` | Request header with credentials |

**Syntax Rules:**

- Use `401` when credentials are missing or invalid.
- Include a `WWW-Authenticate` header in the response.
- Use `403` when the client is authenticated but lacks permission.

**Constraints and Limitations:**

- Do not use `401` for authorization failures; use `403`.
- `WWW-Authenticate` is required by RFC 9110 for 401 responses.

### Annotated Code Examples

**Example 1: Basic Authentication Check**

```python
from flask import Flask, request, jsonify, abort

app = Flask(__name__)
VALID_TOKENS = {"secret-token": "alice"}

@app.route("/protected")
def protected():
    auth = request.headers.get("Authorization", "")
    if not auth.startswith("Bearer "):
        return jsonify({"error": "Missing Bearer token"}), 401, {"WWW-Authenticate": "Bearer"}
    token = auth[7:]
    if token not in VALID_TOKENS:
        return jsonify({"error": "Invalid token"}), 401, {"WWW-Authenticate": "Bearer"}
    return jsonify({"message": f"Hello, {VALID_TOKENS[token]}!"})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /protected` without `Authorization` → `{"error": "Missing Bearer token"}` with status `401` and `WWW-Authenticate: Bearer`.
- `GET /protected` with `Authorization: Bearer secret-token` → `{"message": "Hello, alice!"}`.

**Why this output:** The view checks for the `Authorization` header and validates the token. Missing or invalid tokens return `401` with the `WWW-Authenticate` header.

### Real-World Cases

- **API authentication:** Requiring Bearer tokens for protected endpoints.
- **Session expiry:** Returning 401 when a session token has expired.
- **Login required:** Redirecting to login page with 401.

### References

- RFC 9110: 401 Unauthorized — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.2
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort

---

## 7. 403 Forbidden (Authenticated but Lacking Permissions)

### Definitions

**Core Definition:** `403 Forbidden` indicates that the server understands the request but refuses to authorize it. Unlike 401, the client is authenticated but does not have the necessary permissions.

**Technical Definition:** `403 Forbidden` is a 4xx status code that indicates the server understood the request but will not fulfill it. The client's identity is known, but the client lacks the required permissions or roles. In Flask, `403` is raised with `abort(403)` or returned as a tuple.

**Beginner-Friendly Explanation:** 403 means "I know who you are, but you're not allowed to do this." For example, a regular user trying to access an admin page.

### Purposes

- To deny access to resources the authenticated user does not own or have permission to view.
- To enforce role-based access control (RBAC).
- To prevent unauthorized actions by authenticated users.
- To provide a clear distinction from 401 (authentication failure).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort

@app.route("/admin")
def admin():
    if not current_user.is_admin:
        abort(403, description="Admin access required")
    return "Admin Panel"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(403)` | Raises 403 error |
| `current_user` | Flask-Login proxy for the authenticated user |

**Syntax Rules:**

- Use `403` when the user is authenticated but lacks permission.
- Use `401` when the user is not authenticated.
- The response may include a description, but should not reveal why access is denied.

**Constraints and Limitations:**

- Do not use `403` for unauthenticated users; use `401`.
- Avoid revealing whether a resource exists if the user lacks permission.

### Annotated Code Examples

**Example 1: Role-Based Access Control**

```python
from flask import Flask, jsonify, abort

app = Flask(__name__)

USERS = {
    "alice": {"role": "admin"},
    "bob": {"role": "user"}
}

@app.route("/admin/<username>")
def admin_panel(username):
    user = USERS.get(username)
    if not user:
        abort(404, description="User not found")
    if user["role"] != "admin":
        abort(403, description="Admin privileges required")
    return jsonify({"message": f"Welcome, admin {username}!"})

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /admin/alice` → `{"message": "Welcome, admin alice!"}`.
- `GET /admin/bob` → `{"error": "Admin privileges required"}` with status `403`.

**Why this output:** The view checks the user's role. If the user is not an admin, `abort(403)` is raised. The custom error handler formats the response.

### Real-World Cases

- **Admin panels:** Restricting access to admin-only pages.
- **Resource ownership:** Preventing users from editing others' content.
- **API scopes:** Denying access to endpoints outside the token's scope.

### References

- RFC 9110: 403 Forbidden — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.4
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort

---

## 8. 404 Not Found (Target Resource Missing)

### Definitions

**Core Definition:** `404 Not Found` indicates that the server cannot find the requested resource. This is the most common error response.

**Technical Definition:** `404 Not Found` is a 4xx status code indicating that the origin server did not find a current representation for the target resource or is not willing to disclose that one exists. In Flask, `404` is returned automatically when no route matches the requested URL, or can be raised with `abort(404)` for missing resources in a view.

**Beginner-Friendly Explanation:** 404 means "I looked everywhere, but I can't find what you're asking for." It's what you see when you visit a URL that doesn't exist.

### Purposes

- To indicate that a requested resource does not exist.
- To handle missing database records gracefully.
- To provide a user-friendly error page for broken links.
- To avoid revealing whether a resource exists when access is denied (use 404 instead of 403 in some cases).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort

@app.route("/user/<int:user_id>")
def get_user(user_id):
    user = database.get(user_id)
    if not user:
        abort(404, description="User not found")
    return user
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(404)` | Raises 404 error |
| `@app.errorhandler(404)` | Custom handler for 404 errors |

**Syntax Rules:**

- Use `404` when a resource is not found.
- Customize the 404 page with `@app.errorhandler(404)`.
- Avoid exposing internal details in 404 messages.

**Constraints and Limitations:**

- Do not use `404` for authorization failures; use `403` or `401`.
- 404 errors are often cached by browsers; ensure the resource is truly missing.

### Annotated Code Examples

**Example 1: Custom 404 Error Page**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        "error": "Not Found",
        "message": "The requested resource does not exist"
    }), 404

@app.route("/user/<int:user_id>")
def get_user(user_id):
    users = {1: "Alice", 2: "Bob"}
    if user_id not in users:
        abort(404)
    return users[user_id]
```

**Expected Output:**
- `GET /user/1` → `"Alice"`.
- `GET /user/99` → `{"error": "Not Found", "message": "The requested resource does not exist"}` with status `404`.
- `GET /nonexistent` → same JSON 404 response.

**Why this output:** The `@app.errorhandler(404)` decorator registers a custom handler that returns a JSON response for all 404 errors, including those raised by `abort(404)` and those from unmatched routes.

### Real-World Cases

- **Missing blog post:** `GET /posts/999` returns 404.
- **Deleted user profile:** `GET /users/42` returns 404.
- **Broken links:** Any unmatched URL returns 404.

### References

- RFC 9110: 404 Not Found — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.5
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/

---

## 9. 409 Conflict (State Violations)

### Definitions

**Core Definition:** `409 Conflict` indicates that the request could not be completed due to a conflict with the current state of the target resource, such as a duplicate unique key or a concurrent edit collision.

**Technical Definition:** `409 Conflict` is a 4xx status code used when the request conflicts with the current state of the server. Common causes include attempting to create a resource with a duplicate unique identifier, or updating a resource that has been modified by another client (optimistic locking). In Flask, `409` is raised with `abort(409)` or returned as a tuple.

**Beginner-Friendly Explanation:** 409 means "There's a conflict—something you're trying to do clashes with the current state." For example, trying to register a username that's already taken.

### Purposes

- To signal duplicate unique keys (e.g., email, username).
- To handle concurrent edit collisions (optimistic locking).
- To prevent conflicting state changes.
- To provide a clear error when the client's request cannot be reconciled with the current state.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort

@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if User.query.filter_by(email=data["email"]).first():
        abort(409, description="Email already registered")
    # Create user
    return user, 201
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(409)` | Raises 409 error |
| Duplicate check | Query database for existing resource |

**Syntax Rules:**

- Use `409` for state conflicts, not syntax errors (400) or semantic errors (422).
- Include a descriptive message explaining the conflict.
- Return the conflicting resource or a reference if helpful.

**Constraints and Limitations:**

- 409 is often confused with 422; use 409 for state conflicts and 422 for semantic validation failures.
- Concurrent edit collisions may require retry logic on the client side.

### Annotated Code Examples

**Example 1: Duplicate Email Conflict**

```python
from flask import Flask, request, jsonify, abort

app = Flask(__name__)
USERS = [{"email": "alice@example.com", "name": "Alice"}]

@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if not data or "email" not in data:
        abort(400, description="Email is required")
    if any(u["email"] == data["email"] for u in USERS):
        abort(409, description="Email already registered")
    USERS.append(data)
    return jsonify({"message": "User created", "user": data}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /users` with `{"email": "bob@example.com", "name": "Bob"}` → `{"message": "User created", ...}` with status `201`.
- `POST /users` with `{"email": "alice@example.com"}` → `{"error": "Email already registered"}` with status `409`.

**Why this output:** The view checks if a user with the given email already exists. If so, `abort(409)` is raised. The custom error handler formats the response.

### Real-World Cases

- **User registration:** Duplicate email or username.
- **Inventory management:** Trying to purchase more items than available.
- **Concurrent editing:** Two users editing the same document simultaneously.

### References

- RFC 9110: 409 Conflict — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.10
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort

---

## 10. 422 Unprocessable Content (Semantically Invalid Data)

### Definitions

**Core Definition:** `422 Unprocessable Content` (formerly "Unprocessable Entity") indicates that the request is syntactically correct but contains semantically invalid data, such as a negative price or an invalid date range.

**Technical Definition:** `422 Unprocessable Content` is a 4xx status code defined in RFC 9110 (formerly WebDAV) that indicates the server understands the content type and the syntax is correct, but it cannot process the contained instructions. In Flask, `422` is commonly used with validation libraries like Marshmallow, Pydantic, or webargs when data passes syntax checks but fails semantic validation.

**Beginner-Friendly Explanation:** 422 means "I understood your request, but the data doesn't make sense." For example, a birthdate in the future, or a price that's negative.

### Purposes

- To signal semantic validation failures (e.g., value out of range).
- To distinguish from 400 (syntax errors) and 409 (state conflicts).
- To provide detailed field-level error information.
- To integrate with validation libraries that return 422 automatically.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import abort, jsonify

@app.route("/products", methods=["POST"])
def create_product():
    data = request.get_json()
    errors = []
    if data.get("price", 0) <= 0:
        errors.append("price must be positive")
    if data.get("quantity", 0) < 0:
        errors.append("quantity cannot be negative")
    if errors:
        return jsonify({"errors": errors}), 422
    return jsonify(data), 201
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `return body, 422` | Returns 422 with validation errors |
| `errors` list | Collection of field-level error messages |

**Syntax Rules:**

- Use `422` for semantic errors, not syntax errors (400).
- Include a list of field-level errors in the response.
- Validation libraries like Marshmallow return 422 automatically.

**Constraints and Limitations:**

- 422 is not universally understood by all clients; some use 400 for all client errors.
- RFC 9110 renamed "Unprocessable Entity" to "Unprocessable Content" in recent versions.

### Annotated Code Examples

**Example 1: Validating Product Data**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/products", methods=["POST"])
def create_product():
    data = request.get_json()
    if not data:
        return jsonify({"error": "Request body required"}), 400
    
    errors = []
    if not data.get("name"):
        errors.append({"field": "name", "message": "Name is required"})
    if data.get("price", 0) <= 0:
        errors.append({"field": "price", "message": "Price must be positive"})
    if data.get("quantity", 0) < 0:
        errors.append({"field": "quantity", "message": "Quantity cannot be negative"})
    
    if errors:
        return jsonify({"errors": errors}), 422
    
    return jsonify({"message": "Product created", "product": data}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `POST /products` with `{"name": "Widget", "price": 10, "quantity": 5}` → `{"message": "Product created", ...}` with status `201`.
- `POST /products` with `{"name": "", "price": -5, "quantity": -1}` → `{"errors": [{"field": "name", ...}, {"field": "price", ...}, {"field": "quantity", ...}]}` with status `422`.

**Why this output:** The view performs semantic validation on the parsed JSON. All validation errors are collected and returned with status `422`. The response includes field-level details.

### Real-World Cases

- **E-commerce:** Validating product prices, quantities, and dates.
- **User registration:** Validating password strength, age, and email format.
- **Booking systems:** Validating date ranges and availability.

### References

- RFC 9110: 422 Unprocessable Content — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.21
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort

---

## 11. 500 Internal Server Error (Unhandled Server Exceptions)

### Definitions

**Core Definition:** `500 Internal Server Error` indicates that the server encountered an unexpected condition that prevented it from fulfilling the request.

**Technical Definition:** `500 Internal Server Error` is a 5xx status code that indicates the server encountered an unexpected condition. In Flask, a 500 error is returned when an unhandled exception occurs in a view function, error handler, or `before_request`/`after_request` hook. In debug mode, Flask shows a traceback; in production, a generic error page is shown unless a custom error handler is registered.

**Beginner-Friendly Explanation:** 500 means "Something went wrong on the server, and I don't know how to fix it." It's a catch-all for unexpected errors.

### Purposes

- To indicate an unexpected server-side failure.
- To prevent internal error details from being exposed to clients.
- To provide a generic error response when no other code applies.
- To log errors for debugging and monitoring.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
@app.errorhandler(500)
def internal_error(error):
    return jsonify({"error": "Internal server error"}), 500

@app.errorhandler(Exception)
def handle_exception(error):
    # Log the error
    app.logger.error(f"Unhandled exception: {error}")
    return jsonify({"error": "Internal server error"}), 500
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(500)` | Handles 500 errors |
| `@app.errorhandler(Exception)` | Handles all unhandled exceptions |
| `app.logger` | Flask's logger for error logging |

**Syntax Rules:**

- Register a custom 500 handler to return JSON instead of HTML.
- Use `@app.errorhandler(Exception)` to catch all unhandled exceptions.
- Log the full traceback server-side; never expose it to clients.
- In debug mode, Flask bypasses 500 handlers and shows the traceback.

**Constraints and Limitations:**

- 500 errors should be rare; investigate and fix the root cause.
- Do not leak internal details in 500 responses.
- Some exceptions (e.g., `SystemExit`) are not caught by `errorhandler(Exception)`.

### Annotated Code Examples

**Example 1: Global Exception Handler**

```python
import logging
from flask import Flask, jsonify

app = Flask(__name__)
logging.basicConfig(level=logging.ERROR)
logger = logging.getLogger(__name__)

@app.errorhandler(Exception)
def handle_exception(error):
    logger.error(f"Unhandled exception: {error}", exc_info=True)
    return jsonify({
        "error": "Internal Server Error",
        "message": "An unexpected error occurred"
    }), 500

@app.route("/crash")
def crash():
    raise ValueError("Something went wrong!")

if __name__ == "__main__":
    app.run(debug=False)
```

**Expected Output:**
- `GET /crash` → `{"error": "Internal Server Error", "message": "An unexpected error occurred"}` with status `500`.
- The server log contains the full traceback.

**Why this output:** The `@app.errorhandler(Exception)` decorator catches the `ValueError` raised in the view. The handler logs the error and returns a generic JSON response with status `500`, without exposing internal details.

### Real-World Cases

- **Database connection failures:** Unhandled exceptions when the database is down.
- **Third-party API errors:** Unhandled exceptions from external service calls.
- **Bug in view logic:** Any unexpected Python exception.

### References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- RFC 9110: 500 Internal Server Error — https://www.rfc-editor.org/rfc/rfc9110#section-15.6.1

---

## 12. Global Error Payload Standardization

### Definitions

**Core Definition:** Global error payload standardization is the practice of formatting all error responses (4xx and 5xx) with a consistent JSON structure, making it easier for API clients to handle errors uniformly.

**Technical Definition:** Flask allows registering error handlers for specific status codes or exception classes. By registering handlers for `HTTPException` (the base class for all HTTP errors) and `Exception` (for unhandled errors), developers can return a standardized JSON payload for all error responses. A common structure includes `error.code`, `error.message`, `error.status`, and `error.request_id` fields.

**Beginner-Friendly Explanation:** Instead of having some errors return HTML pages and others return JSON, you make all errors return the same JSON format. This makes it much easier for front-end applications to display error messages consistently.

### Purposes

- To provide a consistent error format across all endpoints.
- To simplify client-side error handling.
- To include helpful metadata like request IDs for debugging.
- To prevent HTML error pages from being returned to API clients.
- To centralize error logging and monitoring.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, jsonify, request, g
from werkzeug.exceptions import HTTPException
import uuid

app = Flask(__name__)

@app.before_request
def add_request_id():
    g.request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))

@app.errorhandler(HTTPException)
def handle_http_exception(error):
    response = {
        "error": {
            "code": error.name.upper().replace(" ", "_"),
            "message": error.description,
            "status": error.code,
            "request_id": getattr(g, "request_id", None)
        }
    }
    return jsonify(response), error.code

@app.errorhandler(Exception)
def handle_generic_error(error):
    return jsonify({
        "error": {
            "code": "INTERNAL_ERROR",
            "message": "An unexpected error occurred",
            "status": 500,
            "request_id": getattr(g, "request_id", None)
        }
    }), 500
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(HTTPException)` | Catches all HTTP errors (4xx, 5xx) |
| `@app.errorhandler(Exception)` | Catches unhandled exceptions |
| `request_id` | Correlation ID for tracing |

**Syntax Rules:**

- Register `HTTPException` handler to catch all `abort()` calls and routing errors.
- Register `Exception` handler to catch unhandled exceptions.
- Include a `request_id` for distributed tracing.
- Never expose internal error details in the `message` field.

**Constraints and Limitations:**

- Some extensions may raise custom exceptions not caught by these handlers.
- In debug mode, Flask bypasses `Exception` handlers.
- Standardizing error payloads requires discipline across the codebase.

### Annotated Code Examples

**Example 1: Complete Error Standardization**

```python
import uuid
import logging
from flask import Flask, jsonify, request, g
from werkzeug.exceptions import HTTPException

app = Flask(__name__)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.before_request
def add_request_id():
    g.request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))

@app.errorhandler(HTTPException)
def handle_http_exception(error):
    logger.warning(f"[{g.request_id}] {error.code} {error.name}: {error.description}")
    return jsonify({
        "error": {
            "code": error.name.upper().replace(" ", "_"),
            "message": error.description,
            "status": error.code,
            "request_id": g.request_id
        }
    }), error.code

@app.errorhandler(Exception)
def handle_generic_error(error):
    logger.error(f"[{g.request_id}] Unhandled: {error}", exc_info=True)
    return jsonify({
        "error": {
            "code": "INTERNAL_ERROR",
            "message": "An unexpected error occurred",
            "status": 500,
            "request_id": g.request_id
        }
    }), 500

@app.route("/ok")
def ok():
    return jsonify({"message": "Success"})

@app.route("/not-found")
def not_found():
    abort(404)

@app.route("/crash")
def crash():
    raise ValueError("Simulated crash")

if __name__ == "__main__":
    app.run(debug=False)
```

**Expected Output:**
- `GET /ok` → `{"message": "Success"}` with status `200`.
- `GET /not-found` → `{"error": {"code": "NOT_FOUND", "message": "...", "status": 404, "request_id": "..."}}` with status `404`.
- `GET /crash` → `{"error": {"code": "INTERNAL_ERROR", "message": "An unexpected error occurred", "status": 500, "request_id": "..."}}` with status `500`.

**Why this output:** The `HTTPException` handler catches all HTTP errors (including `abort(404)` and routing 404s). The `Exception` handler catches unhandled exceptions. Both return the same JSON structure with a request ID.

### Real-World Cases

- **Public APIs:** Providing consistent error responses to API consumers.
- **Microservices:** Standardizing error formats across services.
- **Frontend integration:** Allowing front-end code to display errors uniformly.
- **Observability:** Including request IDs for distributed tracing.

### References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Werkzeug `HTTPException` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.HTTPException
- Flask `before_request` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.before_request

---

## References

- RFC 9110: HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort
- Flask `make_response` — https://flask.palletsprojects.com/en/stable/api/#flask.make_response
- Flask `redirect` — https://flask.palletsprojects.com/en/stable/api/#flask.redirect
- Flask `url_for` — https://flask.palletsprojects.com/en/stable/api/#flask.url_for
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Flask Quickstart: About Responses — https://flask.palletsprojects.com/en/stable/quickstart/#about-responses
- Werkzeug `HTTPException` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.HTTPException
- Flask `before_request` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.before_request
- Flask Status Codes Reference — https://python-flask.readthedocs.io/en/latest/gen/flask_status_codes.html
- Flask API Status Codes — https://flask-api.github.io/flask-api/api-guide/status-codes/