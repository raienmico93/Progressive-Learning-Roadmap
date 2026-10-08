# Flask Error Handling: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Error handling in Flask is the mechanism by which an application detects, intercepts, and responds to runtime errors and HTTP error conditions—such as a missing page (404), forbidden access (403), malformed input (400), or an internal crash (500)—returning controlled, user-friendly responses instead of raw tracebacks or default server pages.

**Technical Definition:** Flask's error handling system is built on Werkzeug's exception hierarchy. When an exception is raised during request handling, Flask's dispatch machinery catches it and looks up a registered handler in the application's `error_handler_spec` dictionary. Handlers can be registered for HTTP status codes (e.g., 404, 403), for `HTTPException` subclasses, or for arbitrary Python exception classes. The `@app.errorhandler()` decorator and the `app.register_error_handler()` method are the two registration APIs. If no handler is found, Flask returns a default HTML error page (or propagates the exception in debug mode). The `TRAP_HTTP_EXCEPTIONS` configuration controls whether HTTP exceptions are treated as regular exceptions for handler lookup.

**Beginner-Friendly Explanation:** When something goes wrong in your Flask app—a user visits a page that doesn't exist, or a bug causes a crash—Flask has a system for handling it gracefully. Instead of showing a scary error page, you can tell Flask "when this happens, show this custom page instead." You can handle specific errors like 404 (not found) or 500 (server error), and you can even catch every possible exception to make sure your app never shows a raw crash.

### Key Characteristics

- **Werkzeug exception hierarchy:** Flask's errors are built on `werkzeug.exceptions.HTTPException` and its subclasses (e.g., `NotFound`, `Forbidden`, `BadRequest`, `InternalServerError`).
- **Two registration methods:** `@app.errorhandler()` (decorator) and `app.register_error_handler()` (imperative).
- **Handler lookup by specificity:** Handlers for specific exception classes take precedence over handlers for base classes or status codes.
- **Global catch-all:** `@app.errorhandler(Exception)` catches all unhandled exceptions, preventing crashes.
- **Debug mode behavior:** In debug mode, Flask shows an interactive traceback instead of calling custom 500 handlers.
- **Blueprint support:** Error handlers can be registered on Blueprints using `@bp.errorhandler()`.
- **Third-party integration:** Flask can catch and handle exceptions from libraries like SQLAlchemy, Marshmallow, and external API clients.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Flask routing and view functions.
- Familiarity with Python exception handling (`try`/`except`, `raise`).
- Knowledge of HTTP status codes (400, 403, 404, 500).
- Optional: `pip install flask-wtf` for validation error handling.

### Related Programming Areas

- **HTTP status codes:** Error handlers map to HTTP response codes.
- **REST API design:** Custom error handlers return consistent JSON error payloads.
- **Logging and monitoring:** Error handlers are the central point for logging exceptions.
- **Security:** Error handlers prevent sensitive information leakage in production.
- **Blueprints:** Modular error handling for different application sections.
- **Third-party libraries:** Handling database errors, validation errors, and API client errors.

### Core Concepts / Features

1. 404 (Not Found)
2. 403 (Forbidden)
3. 400 (Bad Request)
4. 500 (Internal Server Error)
5. Custom Error Handlers
6. Error Handler Registration (`@app.errorhandler` vs. `@app.register_error_handler`)
7. Global Unhandled Exception Catching (`@app.errorhandler(Exception)`)
8. HTTP Exceptions from Third-Party Libraries

---

## 1. 404 (Not Found)

### Definitions

**Core Definition:** A 404 Not Found error indicates that the server cannot find the requested resource—typically because the URL does not match any registered route or the requested database record does not exist.

**Technical Definition:** In Flask, a 404 error is raised as a `werkzeug.exceptions.NotFound` exception when no route matches the requested URL, or when `abort(404)` is called explicitly in a view function. Flask's default handler returns an HTML page with the message "The requested URL was not found on the server." Custom handlers can return HTML templates, JSON responses, or redirects.

**Beginner-Friendly Explanation:** A 404 means "I looked everywhere, but I couldn't find what you asked for." It's what you see when you visit a URL that doesn't exist on the site.

### Purposes

- To indicate that a requested resource does not exist.
- To handle missing database records gracefully.
- To provide a user-friendly error page for broken links.
- To avoid revealing whether a resource exists when access is denied.
- To return JSON error responses for API clients.

### Syntax Rules and Structure

```python
from flask import abort, jsonify, render_template

# Raise a 404 explicitly
@app.route('/user/<int:user_id>')
def get_user(user_id):
    user = database.get(user_id)
    if not user:
        abort(404)
    return user

# Custom 404 handler (HTML)
@app.errorhandler(404)
def page_not_found(error):
    return render_template('404.html'), 404

# Custom 404 handler (JSON)
@app.errorhandler(404)
def api_not_found(error):
    return jsonify({'error': 'Not Found', 'message': 'Resource not found'}), 404
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(404)` | Raises `NotFound` exception |
| `@app.errorhandler(404)` | Registers handler for 404 errors |
| `error` | The `NotFound` exception instance |

**Syntax Rules:**

- `abort(404)` can be called anywhere inside a view function.
- The handler must return a response (body, status, headers) or a `Response` object.
- The status code is automatically set to 404 when returning a tuple with `, 404`.
- Handlers for 404 also catch unmatched routes.

**Constraints and Limitations:**

- 404 errors are often cached by browsers; ensure the resource is truly missing.
- Do not use 404 for authorization failures; use 403 or 401.

### Annotated Code Examples

**Example 1: Custom 404 Page**

```python
from flask import Flask, abort, render_template

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

@app.route('/user/<int:user_id>')
def get_user(user_id):
    users = {1: 'Alice', 2: 'Bob'}
    if user_id not in users:
        abort(404)
    return users[user_id]
```

**Expected Output:**
- `GET /user/1` → `"Alice"`
- `GET /user/99` → custom 404 HTML page with status `404`.
- `GET /nonexistent` → same custom 404 page.

**Why this output:** The `@app.errorhandler(404)` decorator registers the `not_found` function as the handler for all 404 errors, including those raised by `abort(404)` and those from unmatched routes.

### Real-World Cases

- **Missing blog post:** `GET /posts/999` returns a custom 404 page.
- **Deleted user profile:** `GET /users/42` returns a 404 with a friendly message.
- **API resource not found:** `GET /api/users/999` returns a JSON 404 error.

### References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Werkzeug `NotFound` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.NotFound

---

## 2. 403 (Forbidden)

### Definitions

**Core Definition:** A 403 Forbidden error indicates that the server understood the request but refuses to authorize it—the client is authenticated but does not have the necessary permissions.

**Technical Definition:** In Flask, a 403 error is raised as a `werkzeug.exceptions.Forbidden` exception, typically via `abort(403)` when a user lacks the required role or permission. Unlike 401 (Unauthorized), which indicates missing or invalid credentials, 403 indicates that the client's identity is known but access is denied.

**Beginner-Friendly Explanation:** A 403 means "I know who you are, but you're not allowed to do this." For example, a regular user trying to access an admin-only page.

### Purposes

- To deny access to resources the authenticated user does not own or have permission to view.
- To enforce role-based access control (RBAC).
- To prevent unauthorized actions by authenticated users.
- To provide a clear distinction from 401 (authentication failure).

### Syntax Rules and Structure

```python
from flask import abort, jsonify

@app.route('/admin')
def admin_panel():
    if not current_user.is_admin:
        abort(403, description="Admin access required")
    return "Admin Panel"

@app.errorhandler(403)
def forbidden(error):
    return jsonify({'error': 'Forbidden', 'message': str(error.description)}), 403
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(403)` | Raises `Forbidden` exception |
| `description` | Optional custom message |
| `@app.errorhandler(403)` | Registers handler for 403 errors |

**Syntax Rules:**

- Use `abort(403)` when the user is authenticated but lacks permission.
- Use `abort(401)` when the user is not authenticated.
- The handler can return HTML, JSON, or any other response format.

**Constraints and Limitations:**

- Do not use 403 for unauthenticated users; use 401.
- Avoid revealing why access is denied in production.

### Annotated Code Examples

**Example 1: Role-Based Access Control**

```python
from flask import Flask, abort, jsonify

app = Flask(__name__)

USERS = {
    'alice': {'role': 'admin'},
    'bob': {'role': 'user'}
}

@app.route('/admin/<username>')
def admin_panel(username):
    user = USERS.get(username)
    if not user:
        abort(404)
    if user['role'] != 'admin':
        abort(403, description='Admin privileges required')
    return jsonify({'message': f'Welcome, admin {username}!'})

@app.errorhandler(403)
def forbidden(error):
    return jsonify({'error': 'Forbidden', 'message': error.description}), 403
```

**Expected Output:**
- `GET /admin/alice` → `{"message": "Welcome, admin alice!"}`
- `GET /admin/bob` → `{"error": "Forbidden", "message": "Admin privileges required"}` with status `403`.

**Why this output:** The view checks the user's role. If not an admin, `abort(403)` is raised, and the custom handler returns a JSON response.

### Real-World Cases

- **Admin panels:** Restricting access to admin-only pages.
- **Resource ownership:** Preventing users from editing others' content.
- **API scopes:** Denying access to endpoints outside the token's scope.

### References

- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort
- Werkzeug `Forbidden` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.Forbidden

---

## 3. 400 (Bad Request)

### Definitions

**Core Definition:** A 400 Bad Request error indicates that the server cannot process the request because of a client-side error, such as malformed syntax, invalid request framing, or missing required parameters.

**Technical Definition:** In Flask, a 400 error is raised as a `werkzeug.exceptions.BadRequest` exception, commonly via `abort(400)` or automatically when `request.get_json()` encounters invalid JSON. The 400 status indicates that the client should not repeat the request without modification.

**Beginner-Friendly Explanation:** A 400 means "I can't understand your request—something is wrong with what you sent." This could be malformed JSON, missing required fields, or bad syntax.

### Purposes

- To signal that the request body is malformed or unparseable.
- To indicate missing required headers or parameters.
- To reject requests with invalid query strings or form data.
- To provide clear feedback to API clients about input errors.

### Syntax Rules and Structure

```python
from flask import abort, request, jsonify

@app.route('/api/data', methods=['POST'])
def receive_data():
    data = request.get_json(silent=True)
    if data is None:
        abort(400, description='Request body must be valid JSON')
    if 'name' not in data:
        abort(400, description='Missing required field: name')
    return jsonify(data), 201

@app.errorhandler(400)
def bad_request(error):
    return jsonify({'error': 'Bad Request', 'message': error.description}), 400
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abort(400)` | Raises `BadRequest` exception |
| `description` | Custom error message |
| `@app.errorhandler(400)` | Registers handler for 400 errors |

**Syntax Rules:**

- Use 400 for syntax errors, not semantic errors (use 422 for those).
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

@app.route('/api/data', methods=['POST'])
def receive_data():
    data = request.get_json(silent=True)
    if data is None:
        abort(400, description='Request body must be valid JSON')
    return jsonify({'received': data})

@app.errorhandler(400)
def bad_request(error):
    return jsonify({'error': str(error.description)}), 400
```

**Expected Output:**
- `POST /api/data` with invalid JSON → `{"error": "Request body must be valid JSON"}` with status `400`.
- `POST /api/data` with valid JSON → `{"received": {...}}`.

**Why this output:** `request.get_json(silent=True)` returns `None` for invalid JSON. The view calls `abort(400)` with a descriptive message, caught by the custom handler.

### Real-World Cases

- **API endpoints:** Rejecting malformed JSON payloads.
- **Form submissions:** Handling missing required fields.
- **Query string validation:** Rejecting invalid query parameters.

### References

- Flask `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort
- Werkzeug `BadRequest` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.BadRequest

---

## 4. 500 (Internal Server Error)

### Definitions

**Core Definition:** A 500 Internal Server Error indicates that the server encountered an unexpected condition that prevented it from fulfilling the request—typically an unhandled Python exception.

**Technical Definition:** In Flask, a 500 error is raised when an unhandled exception occurs in a view function, error handler, or lifecycle hook. In debug mode, Flask shows an interactive traceback; in production, a generic error page is returned unless a custom handler is registered. Since Flask 1.1, handlers for `InternalServerError` or `500` always receive an `InternalServerError` instance, even if the original exception was a different type.

**Beginner-Friendly Explanation:** A 500 means "Something went wrong on the server, and I don't know how to fix it." It's a catch-all for unexpected errors.

### Purposes

- To indicate an unexpected server-side failure.
- To prevent internal error details from being exposed to clients.
- To provide a generic error response when no other code applies.
- To log errors for debugging and monitoring.

### Syntax Rules and Structure

```python
@app.errorhandler(500)
def internal_error(error):
    return jsonify({'error': 'Internal Server Error'}), 500

@app.errorhandler(InternalServerError)
def handle_500(error):
    original = getattr(error, 'original_exception', None)
    if original:
        app.logger.error(f'Original exception: {original}', exc_info=original)
    return render_template('500.html'), 500
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(500)` | Handles 500 errors |
| `@app.errorhandler(InternalServerError)` | Handles `InternalServerError` instances |
| `error.original_exception` | The original exception (if available) |

**Syntax Rules:**

- Register a custom 500 handler to return JSON instead of HTML.
- Log the full traceback server-side; never expose it to clients.
- In debug mode, Flask bypasses 500 handlers and shows the traceback.

**Constraints and Limitations:**

- 500 errors should be rare; investigate and fix the root cause.
- Do not leak internal details in 500 responses.

### Annotated Code Examples

**Example 1: Global 500 Handler**

```python
import logging
from flask import Flask, jsonify
from werkzeug.exceptions import InternalServerError

app = Flask(__name__)
logging.basicConfig(level=logging.ERROR)
logger = logging.getLogger(__name__)

@app.errorhandler(InternalServerError)
def handle_500(error):
    original = getattr(error, 'original_exception', None)
    if original:
        logger.error(f'Unhandled exception: {original}', exc_info=original)
    return jsonify({
        'error': 'Internal Server Error',
        'message': 'An unexpected error occurred'
    }), 500

@app.route('/crash')
def crash():
    raise ValueError('Something went wrong!')
```

**Expected Output:**
- `GET /crash` → `{"error": "Internal Server Error", "message": "An unexpected error occurred"}` with status `500`.
- The server log contains the full traceback.

**Why this output:** The `InternalServerError` handler catches the `ValueError`, logs the original exception, and returns a generic JSON response.

### Real-World Cases

- **Database connection failures:** Unhandled exceptions when the database is down.
- **Third-party API errors:** Unhandled exceptions from external service calls.
- **Bug in view logic:** Any unexpected Python exception.

### References

- Flask Error Handling: Unhandled Exceptions — https://flask.palletsprojects.com/en/stable/errorhandling/#unhandled-exceptions
- Flask 1.1 Release Notes — https://palletsprojects.com/blog/flask-1-1-released/

---

## 5. Custom Error Handlers

### Definitions

**Core Definition:** Custom error handlers are user-defined functions that intercept specific HTTP status codes or exception classes and return a customized response instead of Flask's default error page.

**Technical Definition:** Custom error handlers are registered using `@app.errorhandler(code_or_exception)` or `app.register_error_handler(code_or_exception, f)`. When an error occurs, Flask looks up the most specific handler: first for the exact exception class, then for base classes (e.g., `HTTPException`), then for the status code. Handlers can return strings, tuples, `Response` objects, or rendered templates. Blueprints can register their own error handlers using `@bp.errorhandler()`.

**Beginner-Friendly Explanation:** Custom error handlers let you decide what your users see when something goes wrong. Instead of Flask's default error page, you can show your own branded 404 page, a JSON error for API clients, or a friendly message for server errors.

### Purposes

- To provide consistent, branded error pages across the application.
- To return JSON error responses for API endpoints.
- To log errors with context for debugging.
- To hide sensitive information in production.
- To handle application-specific exceptions with custom logic.

### Syntax Rules and Structure

```python
# Decorator syntax
@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

# Multiple exception types
@app.errorhandler(ValueError)
@app.errorhandler(TypeError)
def handle_value_error(error):
    return jsonify({'error': str(error)}), 400

# Blueprint-level handler
@bp.errorhandler(404)
def bp_not_found(error):
    return render_template('bp_404.html'), 404
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(code)` | Registers handler for status code |
| `@app.errorhandler(ExceptionClass)` | Registers handler for exception class |
| `error` | The exception instance (or status code) |
| Return value | Response body, tuple, or `Response` object |

**Syntax Rules:**

- Handlers can be registered for status codes (e.g., `404`) or exception classes (e.g., `ValueError`).
- Handlers for specific exceptions take precedence over handlers for base classes.
- Multiple decorators can be stacked to handle multiple exception types.
- Handlers must return a valid response.

**Constraints and Limitations:**

- Handlers for `Exception` catch all exceptions, including HTTP exceptions (use with caution).
- In debug mode, Flask may bypass 500 handlers.
- Handlers registered on the app apply globally; Blueprint handlers apply only to that Blueprint.

### Annotated Code Examples

**Example 1: Multiple Error Handlers**

```python
from flask import Flask, jsonify, render_template

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

@app.errorhandler(500)
def server_error(error):
    return render_template('500.html'), 500

@app.errorhandler(ValueError)
def value_error(error):
    return jsonify({'error': 'Invalid value', 'message': str(error)}), 400
```

**Expected Output:**
- `GET /nonexistent` → custom 404 HTML page.
- A view raising `ValueError` → `{"error": "Invalid value", "message": "..."}` with status `400`.

**Why this output:** Each handler is registered for a specific error type. Flask dispatches to the most specific handler.

### Real-World Cases

- **REST APIs:** Returning JSON error responses for all error types.
- **Public websites:** Custom 404 and 500 pages matching the site's design.
- **Validation errors:** Handling `ValueError` and `TypeError` with 400 responses.
- **Authentication:** Custom handlers for 401 and 403 errors.

### References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Flask `errorhandler` API — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.errorhandler

---

## 6. Error Handler Registration (`@app.errorhandler` vs. `@app.register_error_handler`)

### Definitions

**Core Definition:** Flask provides two APIs for registering error handlers: the `@app.errorhandler()` decorator (declarative) and the `app.register_error_handler()` method (imperative).

**Technical Definition:** `@app.errorhandler(code_or_exception)` is a decorator that registers the decorated function as an error handler. `app.register_error_handler(code_or_exception, f)` is the imperative equivalent that registers the function `f` without using a decorator. The decorator internally calls `register_error_handler()`. The imperative form is useful in application factories where handlers are registered programmatically, or when registering a handler for an already-defined function.

**Beginner-Friendly Explanation:** You can register error handlers either by putting a decorator above your function (`@app.errorhandler(404)`) or by calling `app.register_error_handler(404, my_function)` later. The decorator is more common, but the method is useful when you need to register handlers dynamically.

### Purposes

- **Decorator:** To declaratively register error handlers at definition time.
- **Method:** To register error handlers imperatively in application factories or when handlers are defined elsewhere.
- **Method:** To register multiple handlers for different exception types programmatically.
- **Method:** To register handlers from configuration or plugins.

### Syntax Rules and Structure

```python
# Decorator form
@app.errorhandler(404)
def page_not_found(error):
    return 'Not Found', 404

# Imperative form
def page_not_found(error):
    return 'Not Found', 404

app.register_error_handler(404, page_not_found)

# Registering for multiple exceptions
app.register_error_handler(ValueError, handle_error)
app.register_error_handler(TypeError, handle_error)
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `@app.errorhandler(code_or_exception)` | Decorator; registers the function immediately |
| `app.register_error_handler(code_or_exception, f)` | Method; registers `f` for the given code/exception |

**Syntax Rules:**

- Both APIs register the same way internally; the decorator returns the original function unchanged.
- `register_error_handler()` can be called at any time during application setup.
- The decorator is preferred for readability; the method is preferred for factories and dynamic registration.

**Constraints and Limitations:**

- Handlers must be registered before the application starts handling requests.
- If multiple handlers are registered for the same error, the last one wins.
- Handlers for the same exception type are not chained.

### Annotated Code Examples

**Example 1: Decorator vs. Method**

```python
from flask import Flask, jsonify

app = Flask(__name__)

# Decorator form
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not Found'}), 404

# Imperative form (equivalent)
def bad_request(error):
    return jsonify({'error': 'Bad Request'}), 400

app.register_error_handler(400, bad_request)
```

**Expected Output:**
- `GET /nonexistent` → `{"error": "Not Found"}` with status `404`.
- A request that triggers a 400 → `{"error": "Bad Request"}` with status `400`.

**Why this output:** Both APIs register the handler functions for the specified status codes. The behavior is identical.

### Real-World Cases

- **Application factories:** Using `register_error_handler()` to register handlers in `create_app()`.
- **Blueprints:** Using `@bp.errorhandler()` for Blueprint-specific handlers.
- **Plugins:** Extensions registering their own error handlers imperatively.

### References

- Flask `register_error_handler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.register_error_handler
- Flask `errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.errorhandler

---

## 7. Global Unhandled Exception Catching (`@app.errorhandler(Exception)`)

### Definitions

**Core Definition:** Registering `@app.errorhandler(Exception)` catches all unhandled Python exceptions that occur during request handling, preventing the application from crashing and returning a controlled 500 response.

**Technical Definition:** When an exception is raised in a view function and no more specific handler matches, Flask looks for a handler registered for `Exception` (the base class of all Python exceptions). If found, that handler is called with the exception instance. Without this handler, Flask returns its default 500 error page (or propagates the exception in debug mode). The handler should log the exception and return a generic error response.

**Beginner-Friendly Explanation:** A global exception handler is like a safety net. If anything goes wrong that you didn't specifically handle, this catches it, logs it, and shows a friendly error page instead of crashing.

### Purposes

- To prevent unhandled exceptions from crashing the application.
- To log all unexpected errors for debugging and monitoring.
- To return consistent error responses for all unhandled errors.
- To hide sensitive tracebacks from users in production.
- To integrate with error tracking services (Sentry, Rollbar).

### Syntax Rules and Structure

```python
@app.errorhandler(Exception)
def handle_exception(error):
    # Log the error
    app.logger.error(f'Unhandled exception: {error}', exc_info=True)
    
    # Return a generic response
    return jsonify({
        'error': 'Internal Server Error',
        'message': 'An unexpected error occurred'
    }), 500
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(Exception)` | Catches all Python exceptions |
| `error` | The exception instance |
| `exc_info=True` | Includes traceback in the log |

**Syntax Rules:**

- Register `Exception` handler as a last resort after specific handlers.
- Always log the exception with traceback.
- Return a generic response (don't expose internal details).
- In debug mode, this handler is bypassed for HTTP exceptions.

**Constraints and Limitations:**

- `@app.errorhandler(Exception)` does not catch `BaseException` (e.g., `SystemExit`, `KeyboardInterrupt`).
- Handlers for `Exception` also catch HTTP exceptions (e.g., `NotFound`); register specific handlers for those first.
- In debug mode, Flask shows the traceback instead of calling the handler.

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
    logger.error(f'Unhandled exception: {error}', exc_info=True)
    return jsonify({
        'error': 'Internal Server Error',
        'message': 'An unexpected error occurred'
    }), 500

@app.route('/crash')
def crash():
    raise ValueError('Simulated crash')
```

**Expected Output:**
- `GET /crash` → `{"error": "Internal Server Error", "message": "An unexpected error occurred"}` with status `500`.
- The server log contains the full traceback.

**Why this output:** The `Exception` handler catches the `ValueError`, logs it with the traceback, and returns a generic JSON response.

### Real-World Cases

- **Production applications:** Ensuring no unhandled exception crashes the app.
- **API services:** Returning consistent JSON error responses for all errors.
- **Error tracking:** Integrating with Sentry or Rollbar via the global handler.
- **Monitoring:** Logging all exceptions for observability.

### References

- Flask Error Handling: Unhandled Exceptions — https://flask.palletsprojects.com/en/stable/errorhandling/#unhandled-exceptions
- Stack Overflow: Global Exception Handler in Flask — https://stackoverflow.com/questions/78729577

---

## 8. HTTP Exceptions from Third-Party Libraries

### Definitions

**Core Definition:** HTTP exceptions from third-party libraries are errors raised by external packages (e.g., SQLAlchemy, Marshmallow, `requests`) that Flask can intercept and handle using its error handling system.

**Technical Definition:** Third-party libraries may raise exceptions that subclass `werkzeug.exceptions.HTTPException` (e.g., `BadRequest`) or plain `Exception`. Flask can handle both: `HTTPException` subclasses are handled by status code or exception class handlers, while non-HTTP exceptions require handlers registered for their specific class or for `Exception`. Since Flask 1.1, handlers for `InternalServerError` receive an `InternalServerError` instance with the original exception available via `error.original_exception`.

**Beginner-Friendly Explanation:** When a library your app uses throws an error—like a database connection failure or a validation error from Marshmallow—you can catch it in Flask and return a proper response instead of letting it crash the app.

### Purposes

- To handle database errors (e.g., `SQLAlchemyError`, `IntegrityError`) gracefully.
- To handle validation errors from Marshmallow or Pydantic.
- To handle HTTP errors from API clients (e.g., `requests.exceptions.HTTPError`).
- To map library-specific exceptions to appropriate HTTP status codes.
- To provide consistent error responses regardless of the error source.

### Syntax Rules and Structure

```python
from sqlalchemy.exc import IntegrityError
from marshmallow import ValidationError
from werkzeug.exceptions import HTTPException

# Handle SQLAlchemy IntegrityError
@app.errorhandler(IntegrityError)
def handle_integrity_error(error):
    return jsonify({
        'error': 'Conflict',
        'message': 'A database constraint was violated'
    }), 409

# Handle Marshmallow ValidationError
@app.errorhandler(ValidationError)
def handle_validation_error(error):
    return jsonify({
        'error': 'Validation Error',
        'messages': error.messages
    }), 422

# Handle all HTTPExceptions from third-party libraries
@app.errorhandler(HTTPException)
def handle_http_exception(error):
    return jsonify({
        'error': error.name,
        'message': error.description,
        'status': error.code
    }), error.code
```

**Component Breakdown:**

| Exception | Source | Typical Handler |
|-----------|--------|-----------------|
| `IntegrityError` | SQLAlchemy | 409 Conflict |
| `ValidationError` | Marshmallow | 422 Unprocessable |
| `HTTPError` | `requests` | 502 Bad Gateway |
| `HTTPException` | Werkzeug | Map to status code |

**Syntax Rules:**

- Import the exception class from the library.
- Register a handler using `@app.errorhandler(ExceptionClass)`.
- The handler receives the exception instance.
- Return an appropriate status code and response.

**Constraints and Limitations:**

- Some libraries raise exceptions that are not subclasses of `Exception` (e.g., `BaseException`); these require special handling.
- Not all libraries expose structured error information; inspect the exception attributes.
- In debug mode, some exceptions may bypass handlers.

### Annotated Code Examples

**Example 1: Handling SQLAlchemy and Marshmallow Errors**

```python
from flask import Flask, jsonify
from sqlalchemy.exc import IntegrityError
from marshmallow import ValidationError

app = Flask(__name__)

@app.errorhandler(IntegrityError)
def handle_integrity_error(error):
    app.logger.warning(f'Integrity error: {error}')
    return jsonify({
        'error': 'Conflict',
        'message': 'A resource with that identifier already exists'
    }), 409

@app.errorhandler(ValidationError)
def handle_validation_error(error):
    return jsonify({
        'error': 'Validation Error',
        'messages': error.messages
    }), 422

@app.route('/users', methods=['POST'])
def create_user():
    from marshmallow import Schema, fields
    class UserSchema(Schema):
        name = fields.Str(required=True)
        email = fields.Email(required=True)

    data = request.get_json()
    schema = UserSchema()
    validated = schema.load(data)  # Raises ValidationError
    # Save to database (may raise IntegrityError)
    return jsonify(validated), 201
```

**Expected Output:**
- Invalid input → `{"error": "Validation Error", "messages": {...}}` with status `422`.
- Duplicate email → `{"error": "Conflict", "message": "..."}` with status `409`.

**Why this output:** The handlers for `ValidationError` and `IntegrityError` catch exceptions from Marshmallow and SQLAlchemy, returning appropriate JSON error responses.

### Real-World Cases

- **User registration:** Handling duplicate email (IntegrityError → 409) and validation errors (ValidationError → 422).
- **API gateways:** Handling `requests.exceptions.HTTPError` when calling external services.
- **Database operations:** Handling `OperationalError` when the database is unreachable.
- **File uploads:** Handling `OSError` when disk space is exhausted.

### References

- Flask Error Handling: Generic Exception Handlers — https://flask.palletsprojects.com/en/stable/errorhandling/#generic-exception-handlers
- Werkzeug `HTTPException` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.HTTPException
- SQLAlchemy Exceptions — https://docs.sqlalchemy.org/en/20/core/exceptions.html
- Marshmallow ValidationError — https://marshmallow.readthedocs.io/en/stable/marshmallow.exceptions.html

---

## References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Flask API: `errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.errorhandler
- Flask API: `register_error_handler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.register_error_handler
- Flask API: `abort` — https://flask.palletsprojects.com/en/stable/api/#flask.abort
- Flask Error Handling: Unhandled Exceptions — https://flask.palletsprojects.com/en/stable/errorhandling/#unhandled-exceptions
- Flask Error Handling: Generic Exception Handlers — https://flask.palletsprojects.com/en/stable/errorhandling/#generic-exception-handlers
- Werkzeug `HTTPException` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.HTTPException
- Werkzeug `NotFound` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.NotFound
- Werkzeug `Forbidden` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.Forbidden
- Werkzeug `BadRequest` — https://werkzeug.palletsprojects.com/en/stable/exceptions/#werkzeug.exceptions.BadRequest
- Flask 1.1 Release Notes — https://palletsprojects.com/blog/flask-1-1-released/
- Flask-RESTPlus: Error Handling — https://flask-restplus.readthedocs.io/en/0.10.1/errors.html
- Pluralsight: Secure Error Handling for Python — https://www.pluralsight.com/labs/aws/secure-error-handling-for-python
- Stack Overflow: Global Exception Handler in Flask — https://stackoverflow.com/questions/78729577
- GitHub Issue: Catch-all errorhandler does not catch BadRequestKeyError — https://github.com/pallets/flask/issues/2268