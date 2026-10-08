# Flask Custom Error Pages: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Custom error pages in Flask are user-defined responses that replace the framework's default error messages, allowing developers to return branded HTML pages for browser users, structured JSON payloads for API clients, or dynamically negotiated responses based on the request's `Accept` header.

**Technical Definition:** Flask's error handling system registers handlers via `@app.errorhandler(code_or_exception)` or `app.register_error_handler()`. When an exception is raised, Flask's dispatch machinery looks up the most specific handler in `error_handler_spec`, checking blueprint-specific handlers first (by code, then by exception class), followed by application-level handlers. Handlers receive the exception instance and must return a valid response (string, dict, tuple, or `Response` object). For API consistency, handlers return `jsonify()` responses with structured error payloads. For browser users, handlers return `render_template()` with branded error pages. Dynamic content negotiation uses `request.accept_mimetypes.best_match()` to return HTML or JSON depending on the client's `Accept` header.

**Beginner-Friendly Explanation:** When something goes wrong in your Flask app—a page isn't found, or the server crashes—Flask normally shows a plain error page. With custom error pages, you can show your own branded 404 page to browser users, return structured JSON errors to API clients, and even decide which format to use automatically based on what the client says it wants.

### Key Characteristics

- **Two response formats:** HTML for browser users and JSON for API clients.
- **Two registration APIs:** `@app.errorhandler()` (decorator) and `app.register_error_handler()` (imperative).
- **Blueprint scoping:** `@blueprint.errorhandler()` applies only to requests handled by that blueprint; `@blueprint.app_errorhandler()` applies globally.
- **Handler specificity:** Handlers for specific exception classes take precedence over handlers for base classes or status codes.
- **Content negotiation:** `request.accept_mimetypes.best_match()` selects the best response format based on the client's `Accept` header.
- **Structured error payloads:** JSON error responses include application-specific error keys, status codes, and messages.
- **Debug mode behavior:** In debug mode, Flask shows an interactive traceback instead of calling custom 500 handlers.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Flask routing, view functions, and Blueprints.
- Familiarity with Jinja2 templates and `jsonify()`.
- Knowledge of HTTP status codes (400, 403, 404, 500).
- Understanding of HTTP `Accept` headers (helpful).

### Related Programming Areas

- **HTTP status codes:** Error handlers map to status codes (404, 403, 400, 500).
- **REST API design:** JSON error responses with consistent schemas.
- **Blueprints:** Modular error handling for different application sections.
- **Content negotiation:** Returning different formats based on client preferences.
- **Logging and monitoring:** Error handlers are the central point for logging exceptions.
- **Security:** Preventing sensitive information leakage in production.

### Core Concepts / Features

1. HTML Error Pages
2. JSON Error Responses (Consistent API Error Payloads)
3. Blueprint-Specific Error Handling (`@blueprint.errorhandler` vs. `@blueprint.app_errorhandler`)
4. Dynamic Content Negotiation for Error Responses

---

## 1. HTML Error Pages

### Definitions

**Core Definition:** An HTML error page is a Jinja2 template rendered when an error occurs, providing a branded, user-friendly interface that matches the application's design instead of Flask's default plain error page.

**Technical Definition:** Custom HTML error pages are registered using `@app.errorhandler(status_code)` or `@app.errorhandler(ExceptionClass)`. The handler function returns `render_template('error.html', error=error)` along with the appropriate status code. Templates can access the exception's `code`, `name`, and `description` attributes. For maximum reuse, a single template can be used for multiple error codes, with the template rendering different messages based on the error code.

**Beginner-Friendly Explanation:** When a user visits a page that doesn't exist, instead of showing Flask's default "Not Found" page, you can show your own 404 page with your site's logo, navigation, and a friendly message. This makes your site look professional and helps users find their way back.

### Purposes

- To provide a branded, consistent user experience when errors occur.
- To help users recover from errors by including navigation links.
- To hide technical error details from end users.
- To maintain a consistent design across all pages, including error states.
- To improve SEO by returning proper HTTP status codes with custom content.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return render_template('errors/404.html', error=error), 404

@app.errorhandler(500)
def server_error(error):
    return render_template('errors/500.html', error=error), 500

@app.errorhandler(403)
def forbidden(error):
    return render_template('errors/403.html', error=error), 403

@app.errorhandler(400)
def bad_request(error):
    return render_template('errors/400.html', error=error), 400
```

**Template Structure (`templates/errors/404.html`):**

```html
{% extends "base.html" %}

{% block title %}Page Not Found{% endblock %}

{% block content %}
<div class="error-page">
    <h1>404 - Page Not Found</h1>
    <p>The page you're looking for doesn't exist.</p>
    <a href="{{ url_for('index') }}">Go back to the homepage</a>
</div>
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.errorhandler(404)` | Registers handler for 404 status code |
| `render_template('errors/404.html')` | Renders the custom error template |
| `error` | The exception instance (access `error.code`, `error.name`, `error.description`) |
| `, 404` | Sets the HTTP status code |

**Syntax Rules:**

- Handlers must return a response (template, string, or `Response` object).
- The status code is automatically set to the error code when using `abort()`; explicit `, 404` ensures it for direct returns.
- Templates can extend a base template for consistent layout.
- Use `error.description` to display the default message, or provide a custom message.

**Constraints and Limitations:**

- In debug mode, Flask shows the traceback instead of the custom 500 page.
- Templates must exist in the templates directory or a registered template folder.
- Do not expose internal error details in production templates.

### Annotated Code Examples

**Example 1: Single Template for Multiple Error Codes**

```python
from flask import Flask, render_template, abort

app = Flask(__name__)

@app.errorhandler(400)
@app.errorhandler(403)
@app.errorhandler(404)
@app.errorhandler(500)
def handle_error(error):
    code = getattr(error, 'code', 500)
    return render_template('error.html', code=code, error=error), code

@app.route('/admin')
def admin():
    abort(403, description='Admin access required')

@app.route('/user/<int:user_id>')
def user(user_id):
    if user_id != 1:
        abort(404)
    return 'User 1'
```

```html
<!-- templates/error.html -->
<!DOCTYPE html>
<html>
<head><title>Error {{ code }}</title></head>
<body>
    <h1>Error {{ code }}</h1>
    <p>{{ error.description }}</p>
    <a href="/">Return Home</a>
</body>
</html>
```

**Expected Output:**
- `GET /admin` → HTML page with "Error 403" and "Admin access required".
- `GET /user/99` → HTML page with "Error 404" and "The requested URL was not found on the server."
- `GET /user/1` → `"User 1"`.

**Why this output:** The single `handle_error` function is registered for multiple error codes. It uses `getattr(error, 'code', 500)` to determine the status code and renders the shared template with the appropriate message.

### Real-World Cases

- **E-commerce sites:** A branded 404 page suggesting products or search.
- **Documentation sites:** A 404 page with a search bar and navigation.
- **SaaS applications:** A 500 page with a "try again" button and support link.
- **Blogs:** A 404 page with recent posts to keep users engaged.

### References

- Flask Error Handling: Error Handlers — https://flask.palletsprojects.com/en/stable/errorhandling/#error-handlers
- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template

---

## 2. JSON Error Responses (Consistent API Error Payloads)

### Definitions

**Core Definition:** A JSON error response is a structured JSON payload returned when an error occurs, providing API clients with machine-readable error information including an error code, message, status, and optional details.

**Technical Definition:** JSON error responses are created using `jsonify()` and returned from error handlers with the appropriate HTTP status code. A consistent error schema typically includes fields such as `error` (application-specific error key), `message` (human-readable description), `status` (HTTP status code), and optionally `details` (field-level validation errors). For API-only Blueprints, error handlers return JSON for all errors; for mixed HTML/JSON applications, content negotiation determines the format.

**Beginner-Friendly Explanation:** When an API client sends a bad request, it expects a JSON response it can parse—not an HTML page. A JSON error response says something like `{"error": "VALIDATION_ERROR", "message": "Email is required", "status": 422}`, which the client can handle programmatically.

### Purposes

- To provide API clients with machine-readable error information.
- To maintain a consistent error schema across all endpoints.
- To include application-specific error codes for programmatic handling.
- To provide field-level validation errors for form submissions.
- To distinguish between client errors (4xx) and server errors (5xx).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(400)
def bad_request(error):
    return jsonify({
        'error': 'BAD_REQUEST',
        'message': error.description,
        'status': 400
    }), 400

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        'error': 'NOT_FOUND',
        'message': 'The requested resource does not exist',
        'status': 404
    }), 404

@app.errorhandler(422)
def unprocessable(error):
    return jsonify({
        'error': 'VALIDATION_ERROR',
        'message': 'Validation failed',
        'status': 422,
        'details': error.description  # Could be a list of field errors
    }), 422
```

**Component Breakdown:**

| Field | Description |
|-------|-------------|
| `error` | Application-specific error key (e.g., `VALIDATION_ERROR`) |
| `message` | Human-readable error description |
| `status` | HTTP status code |
| `details` | Optional field-level error information |

**Syntax Rules:**

- Use `jsonify()` to create the response.
- Return a tuple with `, status_code` to set the HTTP status.
- Use consistent field names across all error handlers.
- The `error` key should be a stable, machine-readable string.

**Constraints and Limitations:**

- JSON error responses are not suitable for browser users; use content negotiation for mixed applications.
- Avoid exposing internal exception messages in production.
- Not all clients send `Accept: application/json`; content negotiation handles this.

### Annotated Code Examples

**Example 1: API Error Handlers with Consistent Schema**

```python
from flask import Flask, jsonify, abort, request

app = Flask(__name__)

def error_response(error, message, status, details=None):
    payload = {
        'error': error,
        'message': message,
        'status': status
    }
    if details:
        payload['details'] = details
    return jsonify(payload), status

@app.errorhandler(400)
def bad_request(error):
    return error_response('BAD_REQUEST', str(error.description), 400)

@app.errorhandler(404)
def not_found(error):
    return error_response('NOT_FOUND', 'Resource not found', 404)

@app.errorhandler(422)
def validation_error(error):
    return error_response('VALIDATION_ERROR', 'Validation failed', 422,
                          details=getattr(error, 'description', None))

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json(silent=True)
    if not data:
        abort(400, description='Request body must be valid JSON')
    if 'email' not in data:
        abort(422, description=[{'field': 'email', 'message': 'Email is required'}])
    return jsonify({'id': 1, 'email': data['email']}), 201
```

**Expected Output:**
- `POST /api/users` with invalid JSON → `{"error": "BAD_REQUEST", "message": "Request body must be valid JSON", "status": 400}` with status `400`.
- `POST /api/users` with `{}` → `{"error": "VALIDATION_ERROR", "message": "Validation failed", "status": 422, "details": [{"field": "email", "message": "Email is required"}]}` with status `422`.
- `GET /nonexistent` → `{"error": "NOT_FOUND", "message": "Resource not found", "status": 404}` with status `404`.

**Why this output:** All error handlers use the `error_response` helper function to produce a consistent JSON schema. The status code is set via the tuple return value.

### Real-World Cases

- **REST APIs:** Returning JSON errors for all endpoints.
- **Mobile app backends:** Providing structured errors for mobile clients.
- **Microservices:** Consistent error schemas across services.
- **Validation feedback:** Returning field-level errors for form submissions.

### References

- Flask `jsonify` — https://flask.palletsprojects.com/en/stable/api/#flask.json.jsonify
- Flask Error Handling: Generic Exception Handlers — https://flask.palletsprojects.com/en/stable/errorhandling/#generic-exception-handlers

---

## 3. Blueprint-Specific Error Handling (`@blueprint.errorhandler` vs. `@blueprint.app_errorhandler`)

### Definitions

**Core Definition:** Blueprint-specific error handling allows different parts of an application (Blueprints) to define their own error handlers. `@blueprint.errorhandler()` applies only to requests handled by that Blueprint, while `@blueprint.app_errorhandler()` applies globally to the entire application.

**Technical Definition:** `Blueprint.errorhandler(code_or_exception)` registers an error handler scoped to the Blueprint. When an error occurs during a request handled by the Blueprint, Flask checks for a Blueprint-specific handler first, then falls back to application-level handlers. `Blueprint.app_errorhandler(code_or_exception)` registers a handler that is added to the application's error handler registry, applying to all requests regardless of which Blueprint (or no Blueprint) handled the request.

**Beginner-Friendly Explanation:** If you have an API section and a website section in your app, you might want different 404 pages for each. `@blueprint.errorhandler(404)` gives the API Blueprint its own JSON 404 handler, while the main website keeps its HTML 404 page. `@blueprint.app_errorhandler(404)` would override the 404 handler for the entire app.

### Purposes

- **`errorhandler`:** To provide Blueprint-specific error responses (e.g., JSON for an API Blueprint, HTML for a web Blueprint).
- **`app_errorhandler`:** To register a global error handler from within a Blueprint (e.g., a dedicated error-handling Blueprint).
- **`errorhandler`:** To isolate error handling logic between different application modules.
- **`app_errorhandler`:** To avoid circular imports when error handlers need Blueprint context.

### Syntax Rules and Structure

```python
from flask import Blueprint, jsonify, render_template

# API Blueprint with JSON error handlers
api = Blueprint('api', __name__, url_prefix='/api')

@api.errorhandler(404)
def api_not_found(error):
    return jsonify({'error': 'NOT_FOUND', 'message': 'Resource not found'}), 404

@api.errorhandler(400)
def api_bad_request(error):
    return jsonify({'error': 'BAD_REQUEST', 'message': str(error.description)}), 400

# Web Blueprint with HTML error handlers
web = Blueprint('web', __name__)

@web.errorhandler(404)
def web_not_found(error):
    return render_template('errors/404.html', error=error), 404

# Error-handling Blueprint with app-wide handlers
errors = Blueprint('errors', __name__)

@errors.app_errorhandler(500)
def global_server_error(error):
    return render_template('errors/500.html'), 500

@errors.app_errorhandler(403)
def global_forbidden(error):
    return render_template('errors/403.html'), 403
```

**Component Breakdown:**

| Decorator | Scope | Behavior |
|-----------|-------|----------|
| `@bp.errorhandler(404)` | Blueprint only | Handles 404 errors for requests handled by `bp` |
| `@bp.app_errorhandler(500)` | Application-wide | Handles 500 errors for all requests |

**Syntax Rules:**

- `errorhandler` is inherited from Flask's `Scaffold` class; `app_errorhandler` is Blueprint-specific.
- Blueprint error handlers are checked before application-level handlers.
- `app_errorhandler` registers the handler in the application's error handler registry.
- Blueprints must be registered on the application for their handlers to take effect.

**Constraints and Limitations:**

- If a Blueprint raises an error and re-raises it, Flask may wrap it as `InternalServerError`, triggering the 500 handler instead of the original error handler.
- `app_errorhandler` should be used in a dedicated error-handling Blueprint to avoid circular imports.
- Blueprint error handlers for `HTTPException` subclasses work differently than for custom exceptions.

### Annotated Code Examples

**Example 1: API vs. Web Blueprint Error Handling**

```python
from flask import Flask, Blueprint, jsonify, render_template, abort

app = Flask(__name__)

# API Blueprint
api = Blueprint('api', __name__, url_prefix='/api')

@api.errorhandler(404)
def api_not_found(error):
    return jsonify({'error': 'NOT_FOUND', 'message': 'API resource not found'}), 404

@api.route('/users/<int:user_id>')
def get_user(user_id):
    if user_id != 1:
        abort(404)
    return jsonify({'id': 1, 'name': 'Alice'})

# Web Blueprint
web = Blueprint('web', __name__)

@web.errorhandler(404)
def web_not_found(error):
    return render_template('errors/404.html'), 404

@web.route('/page/<name>')
def page(name):
    if name != 'home':
        abort(404)
    return 'Home Page'

app.register_blueprint(api)
app.register_blueprint(web)

if __name__ == '__main__':
    app.run(debug=True)
```

**Expected Output:**
- `GET /api/users/99` → `{"error": "NOT_FOUND", "message": "API resource not found"}` with status `404`.
- `GET /page/unknown` → custom HTML 404 page with status `404`.
- `GET /nonexistent` → application-level 404 (default or custom).

**Why this output:** The `api` Blueprint has its own 404 handler that returns JSON. The `web` Blueprint has a separate 404 handler that returns HTML. Each handler applies only to requests handled by its respective Blueprint.

**Example 2: Global Error Handler from a Blueprint**

```python
from flask import Flask, Blueprint, jsonify

app = Flask(__name__)

errors = Blueprint('errors', __name__)

@errors.app_errorhandler(500)
def handle_500(error):
    return jsonify({'error': 'INTERNAL_ERROR', 'message': 'Something went wrong'}), 500

@errors.app_errorhandler(Exception)
def handle_exception(error):
    return jsonify({'error': 'UNEXPECTED_ERROR', 'message': str(error)}), 500

app.register_blueprint(errors)

@app.route('/crash')
def crash():
    raise ValueError('Simulated crash')
```

**Expected Output:**
- `GET /crash` → `{"error": "UNEXPECTED_ERROR", "message": "Simulated crash"}` with status `500`.

**Why this output:** The `errors` Blueprint uses `app_errorhandler` to register global handlers. The `handle_exception` function catches all unhandled exceptions, including the `ValueError` raised in the `/crash` route.

### Real-World Cases

- **API + Web applications:** Different error formats for API and web Blueprints.
- **Dedicated error Blueprint:** Centralizing error handling in one Blueprint.
- **Modular applications:** Each Blueprint manages its own error responses.
- **Multi-tenant SaaS:** Tenant-specific error pages via Blueprint handlers.

### References

- Flask Blueprint Error Handlers — https://flask.palletsprojects.com/en/stable/blueprints/#error-handlers
- Flask `Blueprint.errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.errorhandler
- Flask `Blueprint.app_errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.app_errorhandler
- Stack Overflow: Blueprint error handler vs app_errorhandler — https://stackoverflow.com/questions/12655102/flask-error-handler-for-blueprints

---

## 4. Dynamic Content Negotiation for Error Responses

### Definitions

**Core Definition:** Dynamic content negotiation for error responses is the practice of returning HTML error pages for browser requests and JSON error responses for API requests from the same error handler, based on the client's `Accept` header.

**Technical Definition:** Flask's `request.accept_mimetypes` is a `MIMEAccept` object parsed from the `Accept` header. The `best_match(matches, default=None)` method returns the best match from a list of supported mimetypes based on quality factors and specificity. Error handlers check the result of `best_match(['text/html', 'application/json'])` to determine whether to return an HTML template or a JSON response. For API-only Blueprints, JSON is returned by default; for mixed applications, the `Accept` header determines the format.

**Beginner-Friendly Explanation:** When an error occurs, you want to show an HTML page to browser users but return JSON to API clients. Instead of creating separate error handlers, you can use one handler that checks whether the client prefers HTML or JSON and returns the appropriate format.

### Purposes

- To serve HTML error pages to browser users and JSON errors to API clients from the same handler.
- To avoid duplicating error handling logic for different client types.
- To respect the client's `Accept` header and return the preferred format.
- To support mixed applications with both web and API endpoints.
- To maintain a consistent error experience regardless of client type.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import Flask, request, jsonify, render_template

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    best = request.accept_mimetypes.best_match(
        ['text/html', 'application/json'],
        default='text/html'
    )
    if best == 'application/json':
        return jsonify({
            'error': 'NOT_FOUND',
            'message': 'Resource not found',
            'status': 404
        }), 404
    return render_template('errors/404.html', error=error), 404

@app.errorhandler(500)
def server_error(error):
    best = request.accept_mimetypes.best_match(
        ['text/html', 'application/json'],
        default='text/html'
    )
    if best == 'application/json':
        return jsonify({
            'error': 'INTERNAL_ERROR',
            'message': 'An unexpected error occurred',
            'status': 500
        }), 500
    return render_template('errors/500.html', error=error), 500
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `request.accept_mimetypes.best_match()` | Returns the best matching mimetype |
| `['text/html', 'application/json']` | List of supported formats |
| `default='text/html'` | Fallback if no match |
| `jsonify()` | Returns JSON response |
| `render_template()` | Returns HTML response |

**Syntax Rules:**

- Call `best_match()` inside the error handler (requires request context).
- Provide a `default` value for clients without an `Accept` header.
- The order of mimetypes in the list can influence the result; use `default` for explicit fallback.
- For API-only Blueprints, prefer returning JSON directly without negotiation.

**Constraints and Limitations:**

- `best_match()` may return a format the server cannot actually produce if the list is not curated.
- Clients that send `Accept: */*` will match the first item in the list (or the default).
- Content negotiation adds a small overhead to every error response.
- For API-only applications, explicit JSON handlers are simpler and more predictable.

### Annotated Code Examples

**Example 1: Content-Negotiated 404 Handler**

```python
from flask import Flask, request, jsonify, render_template

app = Flask(__name__)

def wants_json():
    """Check if the client prefers JSON over HTML."""
    best = request.accept_mimetypes.best_match(
        ['text/html', 'application/json'],
        default='text/html'
    )
    return best == 'application/json'

@app.errorhandler(404)
def not_found(error):
    if wants_json():
        return jsonify({
            'error': 'NOT_FOUND',
            'message': 'The requested resource does not exist',
            'status': 404
        }), 404
    return render_template('errors/404.html', error=error), 404

@app.route('/api/data')
def api_data():
    return jsonify({'data': 'value'})

@app.route('/page')
def page():
    return render_template('page.html')
```

**Expected Output:**
- `GET /nonexistent` with `Accept: text/html` → HTML 404 page.
- `GET /nonexistent` with `Accept: application/json` → `{"error": "NOT_FOUND", "message": "The requested resource does not exist", "status": 404}` with status `404`.
- `GET /nonexistent` with no `Accept` header → HTML 404 page (default).

**Why this output:** The `wants_json()` helper checks the client's `Accept` header. If JSON is preferred, the handler returns a JSON response; otherwise, it returns the HTML template. The `default='text/html'` ensures browser-like clients receive HTML.

**Example 2: Content Negotiation with API and Web Blueprints**

```python
from flask import Flask, Blueprint, request, jsonify, render_template

app = Flask(__name__)

api = Blueprint('api', __name__, url_prefix='/api')
web = Blueprint('web', __name__)

@api.errorhandler(404)
def api_not_found(error):
    return jsonify({'error': 'NOT_FOUND', 'message': 'API resource not found'}), 404

@web.errorhandler(404)
def web_not_found(error):
    best = request.accept_mimetypes.best_match(
        ['text/html', 'application/json'],
        default='text/html'
    )
    if best == 'application/json':
        return jsonify({'error': 'NOT_FOUND', 'message': 'Resource not found'}), 404
    return render_template('errors/404.html'), 404

@web.route('/page')
def page():
    return render_template('page.html')

app.register_blueprint(api)
app.register_blueprint(web)
```

**Expected Output:**
- `GET /api/nonexistent` → JSON 404 (API Blueprint handler).
- `GET /page/unknown` with `Accept: text/html` → HTML 404 page.
- `GET /page/unknown` with `Accept: application/json` → JSON 404 response.

**Why this output:** The API Blueprint always returns JSON, while the web Blueprint negotiates between HTML and JSON based on the `Accept` header.

### Real-World Cases

- **Mixed web/API applications:** Serving different error formats to different clients.
- **Public APIs:** Returning JSON errors to API consumers while showing HTML to browser users.
- **Progressive web apps:** Negotiating between HTML and JSON based on client capabilities.
- **Content negotiation standards:** Following HTTP specifications for `Accept` header handling.

### References

- Flask `request.accept_mimetypes` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.accept_mimetypes
- Werkzeug `MIMEAccept` — https://werkzeug.palletsprojects.com/en/stable/datastructures/#werkzeug.datastructures.MIMEAccept
- Flask Content Negotiation (Stack Overflow) — https://stackoverflow.com/questions/38677826/flask-change-response-based-on-content-type
- Flask-Negotiate — https://pythonhosted.org/Flask-Negotiate/

---

## References

- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/
- Flask Error Handlers — https://flask.palletsprojects.com/en/stable/errorhandling/#error-handlers
- Flask `errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.errorhandler
- Flask `register_error_handler` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.register_error_handler
- Flask `jsonify` — https://flask.palletsprojects.com/en/stable/api/#flask.json.jsonify
- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template
- Flask Blueprint Error Handlers — https://flask.palletsprojects.com/en/stable/blueprints/#error-handlers
- Flask `Blueprint.errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.errorhandler
- Flask `Blueprint.app_errorhandler` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.app_errorhandler
- Flask `request.accept_mimetypes` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.accept_mimetypes
- Werkzeug `MIMEAccept` — https://werkzeug.palletsprojects.com/en/stable/datastructures/#werkzeug.datastructures.MIMEAccept
- Stack Overflow: Blueprint error handler vs app_errorhandler — https://stackoverflow.com/questions/12655102/flask-error-handler-for-blueprints
- Stack Overflow: Content-type based error handlers — https://stackoverflow.com/questions/38677826/flask-change-response-based-on-content-type
- Flask-Negotiate — https://pythonhosted.org/Flask-Negotiate/
- Flask Discussion: Propagating Blueprint Errors — https://github.com/pallets/flask/discussions/5699