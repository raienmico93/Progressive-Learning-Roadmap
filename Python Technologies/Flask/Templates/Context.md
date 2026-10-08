# Flask Template Context: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Template context in Flask refers to the collection of variables, functions, and objects that are made available to a Jinja2 template when it is rendered, providing the dynamic data that the template uses to generate output.

**Technical Definition:** The template context is a dictionary-like namespace that Flask constructs before rendering a template. It includes variables passed explicitly via `render_template()`, variables injected by context processors, and a set of built-in global variables and functions provided by Flask (`config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()`). The `Flask.update_template_context()` method orchestrates the merging of these sources, ensuring that explicitly passed values take precedence over context processor values, which in turn take precedence over defaults. Context processors are functions registered with the application or a blueprint that return dictionaries; their key-value pairs are merged into the context for all templates (app-level) or templates rendered from a specific blueprint (blueprint-level).

**Beginner-Friendly Explanation:** When Flask renders a template, it needs to give the template all the data it needs to build the page. This data is called the "template context." Some data you pass explicitly when you call `render_template()`, like `render_template("index.html", name="Alice")`. Other data is automatically provided by Flask, like the current request object, the session, and the `url_for` function. Context processors are a way to automatically add your own custom data to every template, like the current year or a user object.

### Key Characteristics

- **Explicit passing:** Variables are passed to templates as keyword arguments to `render_template()`.
- **Context processors:** Functions that inject variables into all templates (app-level) or blueprint-specific templates.
- **Built-in globals:** Flask automatically provides `config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()` to all templates.
- **Precedence rules:** Explicitly passed variables override context processor values, which override defaults.
- **Request-aware:** Some globals (`request`, `session`, `g`) are only available when a request context is active.
- **Blueprint support:** Context processors can be registered on blueprints to apply only to that blueprint's templates.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Jinja2 syntax and template inheritance.
- Familiarity with Flask routing, view functions, and `render_template()`.
- Knowledge of Python dictionaries and functions.

### Related Programming Areas

- **Flask Templating:** Jinja2 integration and template rendering.
- **Context Processors:** Automatic variable injection for templates.
- **Request Lifecycle:** The request context and its relationship to template rendering.
- **Blueprints:** Modular applications with blueprint-specific context.
- **Session Management:** Accessing session data in templates.

### Core Concepts / Features

1. Passing Variables
2. Context Processors
3. Global Template Variables
4. Request-Aware Rendering
5. Built-in Globals (`request`, `session`, `g`, `url_for`, `get_flashed_messages`)

---

## 1. Passing Variables

### Definitions

**Core Definition:** Passing variables is the act of providing data from a Flask view function to a template by passing keyword arguments to `render_template()`.

**Technical Definition:** When a view function calls `render_template(template_name, **context)`, the keyword arguments become variables in the template's context dictionary. These variables can be accessed in the template using Jinja2 expressions (`{{ variable_name }}`). The context dictionary is merged with Flask's default context and any context processor outputs, with the explicitly passed variables taking highest precedence. Variables can be of any Python type: strings, numbers, lists, dictionaries, objects, or functions.

**Beginner-Friendly Explanation:** When you want to show data in a template, you pass it as a keyword argument to `render_template()`. For example, `render_template("hello.html", name="Alice")` makes `{{ name }}` output "Alice" in the template.

### Purposes

- To provide dynamic data from the application to the template.
- To pass user-specific information (names, preferences, permissions).
- To pass collections of data (lists, dictionaries) for iteration.
- To pass objects with attributes for access in the template.
- To provide functions or helpers specific to a page.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import render_template

@app.route("/profile/<username>")
def profile(username):
    user = {"name": username, "email": f"{username}@example.com"}
    return render_template("profile.html", user=user, page_title="Profile")
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `template_name` | Path to the template file (relative to `templates/`) |
| `**context` | Keyword arguments that become template variables |
| Variable types | Strings, numbers, lists, dicts, objects, functions |

**Syntax Rules:**

- Variables are passed as keyword arguments to `render_template()`.
- The variable name in the template matches the keyword argument name.
- Multiple variables can be passed in a single call.
- A dictionary can be unpacked with `**` to pass multiple variables.
- Explicitly passed variables override context processor values.

**Constraints and Limitations:**

- Variables passed to templates are read-only; templates cannot modify them.
- Complex logic should be handled in Python, not templates.
- Undefined variables render as empty strings by default unless `StrictUndefined` is configured.

### Annotated Code Examples

**Example 1: Passing Multiple Variables**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/user/<username>")
def user_profile(username):
    user_data = {
        "name": username,
        "email": f"{username}@example.com",
        "hobbies": ["reading", "coding"]
    }
    return render_template(
        "profile.html",
        user=user_data,
        page_title=f"Profile of {username}",
        is_admin=(username == "admin")
    )

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/profile.html #}
<h1>{{ page_title }}</h1>
<p>Name: {{ user.name }}</p>
<p>Email: {{ user.email }}</p>
<ul>
{% for hobby in user.hobbies %}
    <li>{{ hobby }}</li>
{% endfor %}
</ul>
{% if is_admin %}
    <p>Admin privileges enabled.</p>
{% endif %}
```

**Expected Output (for `/user/alice`):**

```html
<h1>Profile of alice</h1>
<p>Name: alice</p>
<p>Email: alice@example.com</p>
<ul>
    <li>reading</li>
    <li>coding</li>
</ul>
```

**Why this output:** The view function passes three variables (`user`, `page_title`, `is_admin`) to the template. The template accesses `user.name` and `user.email` via dot notation, iterates over `user.hobbies`, and uses `is_admin` in a conditional.

**Example 2: Unpacking a Dictionary**

```python
@app.route("/dashboard")
def dashboard():
    context = {
        "username": "Alice",
        "stats": {"visits": 150, "posts": 42},
        "notifications": ["New comment", "New follower"]
    }
    return render_template("dashboard.html", **context)
```

**Expected Output:**
- The template receives `username`, `stats`, and `notifications` as separate variables.

**Why this output:** The `**` operator unpacks the dictionary, passing each key-value pair as a keyword argument. This is equivalent to passing them individually.

### Real-World Cases

- **User profiles:** Passing user data (name, email, avatar URL).
- **Product pages:** Passing product details (name, price, description).
- **Dashboards:** Passing statistics and chart data.
- **Blog posts:** Passing post content, author, and comments.

### References

- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/

---

## 2. Context Processors

### Definitions

**Core Definition:** A context processor is a function that returns a dictionary of variables, which Flask automatically merges into the template context for every rendered template (app-level) or templates rendered from a specific blueprint (blueprint-level).

**Technical Definition:** Context processors are registered using the `@app.context_processor` decorator (or `@blueprint.context_processor` for blueprint-level). When a template is rendered, Flask calls all registered context processors and merges their returned dictionaries into the template context. Context processors run before the template is rendered and can inject new values or functions. The `update_template_context()` method handles this merging, ensuring that explicitly passed variables take precedence over context processor outputs. Blueprint context processors apply only to templates rendered from that blueprint's views; to affect all templates from a blueprint, use `@blueprint.app_context_processor`.

**Beginner-Friendly Explanation:** A context processor is like a helper that automatically adds data to every template. For example, you can write a context processor that adds the current year to every template, so you don't have to pass it manually in every view function.

### Purposes

- To inject variables that are needed in every template (e.g., current year, site name).
- To provide utility functions available to all templates.
- To add user objects or navigation data to every template.
- To avoid passing the same variables repeatedly in every view function.
- To provide blueprint-specific context for modular applications.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# App-level context processor
@app.context_processor
def inject_globals():
    return {
        "current_year": 2024,
        "site_name": "My Application"
    }

# Blueprint-level context processor
@admin_blueprint.context_processor
def inject_admin_context():
    return {"admin_section": True}

# Context processor with functions
@app.context_processor
def utility_processor():
    def format_price(amount, currency="€"):
        return f"{amount:.2f}{currency}"
    return {"format_price": format_price}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.context_processor` | Decorator to register an app-level processor |
| `@blueprint.context_processor` | Decorator to register a blueprint-level processor |
| Return value | A dictionary of variables to inject |
| `@blueprint.app_context_processor` | Blueprint processor that applies to all templates |

**Syntax Rules:**

- Context processors are functions that return a dictionary.
- The keys of the returned dictionary become template variables.
- App-level processors run for every template render.
- Blueprint-level processors run only for templates rendered from that blueprint's views.
- Explicitly passed variables override context processor values.
- Multiple context processors can be registered; their outputs are merged.

**Constraints and Limitations:**

- Context processors run on every request, so they should be efficient.
- Values returned by context processors cannot be overridden by later context processors (first wins).
- Context processors cannot access the template name being rendered.
- Blueprint context processors do not apply to templates rendered outside the blueprint.

### Annotated Code Examples

**Example 1: App-Level Context Processor**

```python
from flask import Flask, render_template
from datetime import datetime

app = Flask(__name__)

@app.context_processor
def inject_globals():
    return {
        "current_year": datetime.now().year,
        "site_name": "My Flask App"
    }

@app.route("/")
def index():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/index.html #}
<footer>
    &copy; {{ current_year }} {{ site_name }}
</footer>
```

**Expected Output:**

```html
<footer>
    &copy; 2024 My Flask App
</footer>
```

**Why this output:** The context processor returns `current_year` and `site_name`, which are automatically injected into every template. The footer uses these variables without the view function passing them explicitly.

**Example 2: Context Processor with Utility Function**

```python
@app.context_processor
def utility_processor():
    def format_price(amount, currency="€"):
        return f"{amount:.2f}{currency}"
    return dict(format_price=format_price)
```

```jinja
{# templates/product.html #}
<p>Price: {{ format_price(product.price) }}</p>
```

**Expected Output:**

```html
<p>Price: €19.99</p>
```

**Why this output:** The context processor injects the `format_price` function into every template. The template calls it like any other function.

**Example 3: Blueprint-Level Context Processor**

```python
from flask import Blueprint

admin = Blueprint("admin", __name__)

@admin.context_processor
def inject_admin_context():
    return {"admin_section": True}

@admin.route("/dashboard")
def dashboard():
    return render_template("admin/dashboard.html")
```

```jinja
{# templates/admin/dashboard.html #}
{% if admin_section %}
    <p>Admin panel active.</p>
{% endif %}
```

**Expected Output:**

```html
<p>Admin panel active.</p>
```

**Why this output:** The blueprint context processor injects `admin_section` only into templates rendered from the `admin` blueprint's views. Other blueprints do not see this variable.

### Real-World Cases

- **Current year:** Injecting the current year into footers across all templates.
- **Site name:** Providing the site name to headers and footers.
- **User object:** Injecting `current_user` from Flask-Login into all templates.
- **Utility functions:** Providing `format_price`, `format_date`, or `url_for` wrappers.
- **Navigation menus:** Injecting navigation data for consistent menus.

### References

- Flask Context Processors — https://flask.palletsprojects.com/en/stable/templating/#context-processors
- Flask `context_processor` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.context_processor
- Flask Blueprint `context_processor` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.context_processor

---

## 3. Global Template Variables

### Definitions

**Core Definition:** Global template variables are variables that are available in all templates without being explicitly passed by the view function, provided by Flask's default context or by context processors.

**Technical Definition:** Flask automatically injects a set of standard variables into every template context: `config` (the application configuration object), `request` (the current request object), `session` (the current session object), `g` (the request-bound global object), `url_for()` (the URL building function), and `get_flashed_messages()` (the flash message retrieval function). These variables are not true globals but are added to the context of each template. They are available in imported templates as of Flask 0.10, but not in macros imported without `with context`. Context processors can also add custom global variables.

**Beginner-Friendly Explanation:** Global template variables are things Flask gives you for free in every template. You don't have to pass them in `render_template()`. For example, `url_for` is always available to generate links, and `request` gives you access to the current request. Context processors let you add your own globals, like the site name or current year.

### Purposes

- To provide access to application configuration in templates.
- To enable URL generation with `url_for()`.
- To display flash messages with `get_flashed_messages()`.
- To access request and session data for conditional rendering.
- To inject custom variables that are needed everywhere.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Built-in globals are automatically available
# No explicit passing needed in view functions
```

```jinja
{# Using built-in globals in templates #}
<a href="{{ url_for('index') }}">Home</a>

{% with messages = get_flashed_messages() %}
    {% if messages %}
        <ul>
        {% for message in messages %}
            <li>{{ message }}</li>
        {% endfor %}
        </ul>
    {% endif %}
{% endwith %}

{% if config.DEBUG %}
    <p>Debug mode is on.</p>
{% endif %}
```

**Component Breakdown:**

| Variable | Description |
|----------|-------------|
| `config` | Application configuration object |
| `request` | Current request object |
| `session` | Current session object |
| `g` | Request-bound global object |
| `url_for()` | URL building function |
| `get_flashed_messages()` | Flash message retrieval function |

**Syntax Rules:**

- These variables are automatically available in all templates rendered within an application context.
- `request`, `session`, and `g` are only available when a request context is active.
- `url_for()` and `get_flashed_messages()` are available as functions.
- `config` is always available.
- Imported templates have access to these globals as of Flask 0.10.

**Constraints and Limitations:**

- `request`, `session`, and `g` are unavailable when rendering templates without an active request context (e.g., in background tasks).
- These variables are not truly global; they are added to each template's context.
- Macros imported without `with context` do not have access to these variables.

### Annotated Code Examples

**Example 1: Using `url_for()` and `get_flashed_messages()`**

```python
from flask import Flask, render_template, flash, redirect, url_for

app = Flask(__name__)
app.secret_key = "secret"

@app.route("/")
def index():
    flash("Welcome to the site!")
    return render_template("index.html")

@app.route("/about")
def about():
    return render_template("about.html")

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/index.html #}
<nav>
    <a href="{{ url_for('index') }}">Home</a>
    <a href="{{ url_for('about') }}">About</a>
</nav>

{% with messages = get_flashed_messages() %}
    {% if messages %}
        <ul class="flashes">
        {% for message in messages %}
            <li>{{ message }}</li>
        {% endfor %}
        </ul>
    {% endif %}
{% endwith %}
```

**Expected Output:**

```html
<nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
</nav>
<ul class="flashes">
    <li>Welcome to the site!</li>
</ul>
```

**Why this output:** `url_for()` generates the correct URLs for the routes. `get_flashed_messages()` retrieves the flashed message, which is displayed in a list. Both are available without being passed by the view function.

**Example 2: Using `config` and `request`**

```jinja
{# templates/debug.html #}
{% if config.DEBUG %}
    <p>Debug mode is enabled.</p>
{% endif %}

<p>Current path: {{ request.path }}</p>
<p>Request method: {{ request.method }}</p>
```

**Expected Output:**

```html
<p>Debug mode is enabled.</p>
<p>Current path: /debug</p>
<p>Request method: GET</p>
```

**Why this output:** `config` provides access to the application configuration, and `request` provides access to the current request object. Both are automatically available in the template context.

### Real-World Cases

- **Navigation:** Using `url_for()` to generate links in navigation menus.
- **Flash messages:** Displaying success or error messages after redirects.
- **Debug indicators:** Showing debug mode status during development.
- **Request info:** Displaying the current path or method in templates.

### References

- Flask Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context
- Flask `url_for` — https://flask.palletsprojects.com/en/stable/api/#flask.url_for
- Flask `get_flashed_messages` — https://flask.palletsprojects.com/en/stable/api/#flask.get_flashed_messages

---

## 4. Request-Aware Rendering

### Definitions

**Core Definition:** Request-aware rendering refers to the behavior of Flask's template context when a request context is active, making `request`, `session`, and `g` available in templates, and affecting how context processors and URL generation behave.

**Technical Definition:** When a template is rendered during a request (i.e., within a view function), Flask's request context is active. This activates the `request` and `session` proxies and the `g` object, making them available in the template context. `url_for()` uses the request context to generate relative URLs, and `get_flashed_messages()` retrieves messages from the session. When a template is rendered outside a request context (e.g., in a background task or CLI command), these variables are unavailable, and `url_for()` requires `SERVER_NAME` to generate external URLs. The `update_template_context()` method conditionally includes `request`, `session`, and `g` based on whether a request context is active.

**Beginner-Friendly Explanation:** When you render a template from a view function, Flask knows about the current request. This means the template can access `request` (the current request), `session` (the user's session), and `g` (a place to store data during the request). If you render a template outside a request (like in a script), these variables are not available.

### Purposes

- To access request data (headers, path, method) in templates.
- To access session data (logged-in user, preferences) in templates.
- To use `g` for request-scoped data in templates.
- To generate URLs that respect the current request context.
- To conditionally render content based on request state.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Within a view (request context active)
@app.route("/dashboard")
def dashboard():
    g.user = get_current_user()
    return render_template("dashboard.html")
```

```jinja
{# Template has access to request, session, g #}
{% if session.get('user_id') %}
    <p>Welcome back!</p>
{% endif %}
<p>Method: {{ request.method }}</p>
{% if g.user %}
    <p>User: {{ g.user.name }}</p>
{% endif %}
```

**Component Breakdown:**

| Variable | Availability | Description |
|----------|--------------|-------------|
| `request` | Request context active | Current request object |
| `session` | Request context active | Current session object |
| `g` | Request context active | Request-bound global object |
| `url_for()` | Always (external needs SERVER_NAME) | URL building function |
| `get_flashed_messages()` | Request context active | Flash message retrieval |

**Syntax Rules:**

- `request`, `session`, and `g` are available only within an active request context.
- When rendering outside a request context, these variables are `None` or unavailable.
- `url_for()` works in both contexts but requires `SERVER_NAME` for external URLs outside a request.
- `g` is useful for storing data that multiple functions need during a single request.

**Constraints and Limitations:**

- Rendering templates without a request context (e.g., in background tasks) means `request`, `session`, and `g` are unavailable.
- `g` is cleared after each request; do not store data that needs to persist.
- Using `request` in templates couples them to HTTP; consider passing data explicitly for better testability.

### Annotated Code Examples

**Example 1: Accessing Request and Session in Templates**

```python
from flask import Flask, render_template, session, g

app = Flask(__name__)
app.secret_key = "secret"

@app.route("/dashboard")
def dashboard():
    session["user_id"] = 42
    g.user_name = "Alice"
    return render_template("dashboard.html")

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/dashboard.html #}
{% if session.get('user_id') %}
    <p>User ID: {{ session['user_id'] }}</p>
{% endif %}
<p>Request path: {{ request.path }}</p>
{% if g.user_name %}
    <p>User name: {{ g.user_name }}</p>
{% endif %}
```

**Expected Output:**

```html
<p>User ID: 42</p>
<p>Request path: /dashboard</p>
<p>User name: Alice</p>
```

**Why this output:** The view function sets `session["user_id"]` and `g.user_name` before rendering. The template accesses these values directly because the request context is active.

**Example 2: Rendering Outside a Request Context**

```python
from flask import Flask, render_template

app = Flask(__name__)

# Render without a request context
with app.app_context():
    html = render_template("static_page.html")
    # request, session, and g are unavailable
```

```jinja
{# templates/static_page.html #}
{% if request %}
    <p>Request is available</p>
{% else %}
    <p>Request is NOT available</p>
{% endif %}
```

**Expected Output:**

```html
<p>Request is NOT available</p>
```

**Why this output:** Outside a request context, the `request` variable is undefined or `None`. The template's conditional handles this gracefully.

### Real-World Cases

- **User dashboards:** Accessing `session` to display user-specific data.
- **Request logging:** Using `request` to log the current path or method.
- **Multi-step forms:** Using `g` to store intermediate data across requests.
- **Background rendering:** Rendering templates in Celery tasks without a request context.

### References

- Flask Request Context — https://flask.palletsprojects.com/en/stable/reqcontext/
- Flask Application Context — https://flask.palletsprojects.com/en/stable/appcontext/
- Flask Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context

---

## 5. Built-in Globals (`request`, `session`, `g`, `url_for`, `get_flashed_messages`)

### Definitions

**Core Definition:** Flask provides six built-in globals to every template context: `config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()`, which provide access to application configuration, request data, session data, request-scoped storage, URL generation, and flash messages respectively.

**Technical Definition:** These globals are injected into the Jinja2 context by Flask's `update_template_context()` method. `config` is the application's configuration object (a `Config` instance). `request` and `session` are `LocalProxy` objects that point to the current request and session. `g` is a `LocalProxy` to a namespace object for storing request-scoped data. `url_for()` is Flask's URL building function. `get_flashed_messages()` retrieves messages from the session's flash storage. As of Flask 0.10, these globals are available even in imported templates, but macros imported without `with context` do not have access to them.

**Beginner-Friendly Explanation:** These are the freebies Flask gives you in every template. `url_for` helps you build links, `get_flashed_messages` shows notifications, `config` tells you about app settings, `request` tells you about the current request, `session` remembers things about the user, and `g` is a scratchpad for the current request.

### Purposes

- **`request`:** To access request data (headers, path, method) in templates.
- **`session`:** To access session data (user ID, preferences) in templates.
- **`g`:** To share data between functions during a single request.
- **`url_for()`:** To generate URLs for routes and static files.
- **`get_flashed_messages()`:** To display flash messages to the user.
- **`config`:** To access application configuration values.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# request #}
<p>Path: {{ request.path }}</p>
<p>Method: {{ request.method }}</p>
<p>User-Agent: {{ request.headers.get('User-Agent') }}</p>

{# session #}
{% if session.get('user_id') %}
    <p>Logged in as user {{ session['user_id'] }}</p>
{% endif %}

{# g #}
<p>{{ g.user_name }}</p>

{# url_for #}
<a href="{{ url_for('index') }}">Home</a>
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">

{# get_flashed_messages #}
{% with messages = get_flashed_messages() %}
    {% if messages %}
        <ul>
        {% for message in messages %}
            <li>{{ message }}</li>
        {% endfor %}
        </ul>
    {% endif %}
{% endwith %}

{# config #}
{% if config.DEBUG %}
    <p>Debug mode</p>
{% endif %}
```

**Component Breakdown:**

| Global | Type | Availability |
|--------|------|--------------|
| `request` | Request object | Request context active |
| `session` | Session object | Request context active |
| `g` | Namespace object | Request context active |
| `url_for()` | Function | Always (external needs SERVER_NAME) |
| `get_flashed_messages()` | Function | Request context active |
| `config` | Config object | Always |

**Syntax Rules:**

- `request`, `session`, and `g` require an active request context.
- `url_for()` can be used with or without a request context.
- `get_flashed_messages()` requires a request context and a `SECRET_KEY`.
- `config` is always available.
- These globals are injected automatically; no explicit passing is needed.

**Constraints and Limitations:**

- `request`, `session`, and `g` are unavailable outside a request context.
- `get_flashed_messages()` requires `SECRET_KEY` to be set.
- Macros imported without `with context` cannot access these globals.
- `g` is cleared after each request; do not store data that needs to persist.

### Annotated Code Examples

**Example 1: Using All Built-in Globals**

```python
from flask import Flask, render_template, session, g, flash

app = Flask(__name__)
app.secret_key = "secret"
app.config["SITE_NAME"] = "My App"

@app.route("/")
def index():
    session["user_id"] = 42
    g.user_name = "Alice"
    flash("Welcome back!")
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/index.html #}
<h1>{{ config.SITE_NAME }}</h1>

{% if session.get('user_id') %}
    <p>User ID: {{ session['user_id'] }}</p>
{% endif %}

{% if g.user_name %}
    <p>Hello, {{ g.user_name }}!</p>
{% endif %}

<nav>
    <a href="{{ url_for('index') }}">Home</a>
</nav>

{% with messages = get_flashed_messages() %}
    {% if messages %}
        <ul class="flashes">
        {% for message in messages %}
            <li>{{ message }}</li>
        {% endfor %}
        </ul>
    {% endif %}
{% endwith %}

<p>Request path: {{ request.path }}</p>
```

**Expected Output:**

```html
<h1>My App</h1>
<p>User ID: 42</p>
<p>Hello, Alice!</p>
<nav>
    <a href="/">Home</a>
</nav>
<ul class="flashes">
    <li>Welcome back!</li>
</ul>
<p>Request path: /</p>
```

**Why this output:** Each built-in global provides its specific functionality: `config` gives the site name, `session` provides the user ID, `g` provides the user name, `url_for` generates the home link, `get_flashed_messages` displays the flash message, and `request` provides the current path.

### Real-World Cases

- **Authentication UI:** Using `session` to show/hide login/logout links.
- **Navigation:** Using `url_for` to generate consistent links.
- **Notifications:** Using `get_flashed_messages` to display success/error messages.
- **Request info:** Using `request` to display the current page in breadcrumbs.
- **Configuration:** Using `config` to show/hide debug features.

### References

- Flask Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context
- Flask `request` — https://flask.palletsprojects.com/en/stable/api/#flask.request
- Flask `session` — https://flask.palletsprojects.com/en/stable/api/#flask.session
- Flask `g` — https://flask.palletsprojects.com/en/stable/api/#flask.g
- Flask `url_for` — https://flask.palletsprojects.com/en/stable/api/#flask.url_for
- Flask `get_flashed_messages` — https://flask.palletsprojects.com/en/stable/api/#flask.get_flashed_messages
- Flask `config` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.config

---

## References

- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/
- Flask Context Processors — https://flask.palletsprojects.com/en/stable/templating/#context-processors
- Flask Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context
- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template
- Flask `context_processor` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.context_processor
- Flask Blueprint `context_processor` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.context_processor
- Flask Request Context — https://flask.palletsprojects.com/en/stable/reqcontext/
- Flask Application Context — https://flask.palletsprojects.com/en/stable/appcontext/
- Flask `url_for` — https://flask.palletsprojects.com/en/stable/api/#flask.url_for
- Flask `get_flashed_messages` — https://flask.palletsprojects.com/en/stable/api/#flask.get_flashed_messages
- Flask `session` — https://flask.palletsprojects.com/en/stable/api/#flask.session
- Flask `g` — https://flask.palletsprojects.com/en/stable/api/#flask.g
- Flask `config` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.config
- Flask Message Flashing — https://flask.palletsprojects.com/en/stable/patterns/flashing/