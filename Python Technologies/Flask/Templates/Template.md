# Flask Template Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Flask templates are files that contain static HTML mixed with dynamic placeholders, rendered by the Jinja2 template engine to produce the final HTML sent to the browser.

**Technical Definition:** Flask integrates Jinja2 as its default template engine. Jinja2 is a modern, designer-friendly templating engine for Python that compiles templates into Python code. Flask creates a Jinja2 `Environment` configured with a `FileSystemLoader` (or `ChoiceLoader` for blueprints) that searches configured directories for template files. Templates use special delimiters: `{{ ... }}` for expressions (output), `{% ... %}` for statements (control flow), and `{# ... #}` for comments. Autoescaping is enabled by default for templates ending in `.html`, `.htm`, `.xml`, `.xhtml`, and `.svg` when using `render_template()`, and for all strings when using `render_template_string()` .

**Beginner-Friendly Explanation:** A template is an HTML file with blanks you fill in. Instead of writing HTML in your Python code, you write it in a separate file and use special syntax like `{{ username }}` to insert dynamic values. Flask fills in these blanks and sends the complete HTML page to the browser. This keeps your HTML and Python separate and makes your code much cleaner.

### Key Characteristics

- **Jinja2 engine:** Flask uses Jinja2 as its template engine; it is required for Flask to function .
- **Autoescaping:** HTML templates are autoescaped by default, preventing XSS attacks .
- **Template inheritance:** Child templates extend base templates, overriding blocks for consistent layouts .
- **Standard context:** Flask automatically injects `config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()` into template contexts .
- **Multiple template directories:** Flask searches the application's `templates/` folder and any blueprint-specific `template_folder` directories .
- **Configurable environment:** `TEMPLATES_AUTO_RELOAD` controls template reloading during development .

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTML and Python dictionaries.
- Familiarity with Flask routing and view functions.
- Knowledge of Python functions and string formatting.

### Related Programming Areas

- **Web development:** Templates are the presentation layer of web applications.
- **Jinja2:** The underlying template engine with its own syntax and features.
- **Security:** Autoescaping prevents XSS; `Markup` and `|safe` control raw HTML output.
- **Application architecture:** Blueprints organize templates by module.
- **Static assets:** Templates reference CSS, JS, and images via `url_for('static', ...)`.

### Core Concepts / Features

1. Jinja Templates
2. `render_template()` and `render_template_string()`
3. Template Directories (Custom Search Paths and Blueprint Folders)
4. Template Variables
5. Template Inheritance
6. Environment Configuration (Auto-Reload, Cache Settings)

---

## 1. Jinja Templates

### Definitions

**Core Definition:** Jinja templates are text files (usually HTML) that contain static content and dynamic placeholders using Jinja2 syntax, which are rendered into final output by the Jinja2 engine.

**Technical Definition:** Jinja2 uses three delimiter types: `{{ ... }}` for expressions (output), `{% ... %}` for statements (control flow, inheritance), and `{# ... #}` for comments. Expressions are evaluated and their results are converted to strings and inserted into the output. Statements control the flow of rendering (loops, conditionals, blocks). Jinja2 compiles templates to Python bytecode for efficient repeated rendering. Flask configures Jinja2 with autoescaping enabled for HTML templates by default .

**Beginner-Friendly Explanation:** A Jinja template is an HTML file with special placeholders. `{{ name }}` displays the value of a variable. `{% if user %}...{% endif %}` shows content only if a condition is true. `{% for item in items %}...{% endfor %}` repeats content for each item in a list.

### Purposes

- To separate presentation (HTML) from application logic (Python).
- To generate dynamic HTML pages with data from the application.
- To reuse common layout elements through inheritance.
- To automatically escape user input and prevent XSS.
- To support conditional rendering and loops within HTML.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# This is a comment #}

{{ expression }}          {# Output the value of an expression #}

{% statement %}           {# Execute a control-flow statement #}
{% endstatement %}

{% if condition %}
    <p>Shown when true</p>
{% elif other_condition %}
    <p>Shown when other is true</p>
{% else %}
    <p>Shown otherwise</p>
{% endif %}

{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
```

**Component Breakdown:**

| Delimiter | Purpose | Example |
|-----------|---------|---------|
| `{{ ... }}` | Expression output | `{{ user.name }}` |
| `{% ... %}` | Statement/control flow | `{% if user %}` |
| `{# ... #}` | Comment | `{# TODO: fix this #}` |

**Syntax Rules:**

- Expressions can use filters: `{{ name|upper }}`.
- Statements do not output anything; they control flow.
- Comments are removed from the output.
- Whitespace control: `{%-` and `-%}` trim surrounding whitespace.

**Constraints and Limitations:**

- Jinja2 is not Python; some Python constructs (e.g., list comprehensions) are not available.
- Complex logic should be in Python, not templates.
- Autoescaping may need to be disabled for trusted HTML using `|safe` or `Markup`.

### Annotated Code Examples

**Example 1: Basic Template with Variables and Loops**

```html
<!-- templates/users.html -->
<!DOCTYPE html>
<html>
<head><title>Users</title></head>
<body>
    <h1>User List</h1>
    <ul>
    {% for user in users %}
        <li>{{ user.name }} ({{ user.email }})</li>
    {% endfor %}
    </ul>
</body>
</html>
```

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/users")
def users():
    user_list = [
        {"name": "Alice", "email": "alice@example.com"},
        {"name": "Bob", "email": "bob@example.com"}
    ]
    return render_template("users.html", users=user_list)

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /users` → HTML page with `<li>Alice (alice@example.com)</li><li>Bob (bob@example.com)</li>`.

**Why this output:** The `{% for %}` loop iterates over the `users` list. Each `{{ user.name }}` expression outputs the corresponding value from each dictionary.

### Real-World Cases

- **User listings:** Displaying tables of users, products, or orders.
- **Blog posts:** Rendering articles with dynamic titles and content.
- **Dashboards:** Showing metrics and charts with dynamic data.
- **Email templates:** Generating personalized email content.

### References

- Jinja2 Template Documentation — https://jinja.palletsprojects.com/templates/
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/

---

## 2. `render_template()` and `render_template_string()`

### Definitions

**Core Definition:** `render_template()` renders a template file from the templates directory, while `render_template_string()` renders a Jinja2 template from a string source.

**Technical Definition:** `flask.render_template(template_name_or_list, **context)` loads a template from the configured template directories and renders it with the provided context variables. `flask.render_template_string(source, **context)` renders a template directly from a string. Both functions use the application's Jinja2 environment. Autoescaping is enabled for templates ending in `.html`, `.htm`, `.xml`, `.xhtml`, and `.svg` when using `render_template()`, and for all strings when using `render_template_string()` .

**Beginner-Friendly Explanation:** `render_template()` loads an HTML file from your templates folder and fills in the blanks. `render_template_string()` does the same thing but takes the HTML directly as a string instead of from a file. The string version is useful for small templates or dynamic content.

### Purposes

- To render HTML pages from template files (`render_template`).
- To render small templates without creating separate files (`render_template_string`).
- To pass dynamic data to templates via context variables.
- To support both file-based and string-based template rendering.
- To generate email bodies, reports, or other text output.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import render_template, render_template_string

# Render from a file
return render_template("index.html", name="Alice", items=[1, 2, 3])

# Render from a string
return render_template_string("<h1>Hello, {{ name }}!</h1>", name="Alice")
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `render_template(name, **context)` | Renders a template file |
| `render_template_string(source, **context)` | Renders a template string |
| `**context` | Variables passed to the template |

**Syntax Rules:**

- `render_template()` searches configured template directories for the file.
- `render_template_string()` renders the string immediately.
- Both functions return a string (the rendered output).
- Autoescaping is enabled for both, but `render_template_string()` escapes all strings.

**Constraints and Limitations:**

- `render_template()` raises `TemplateNotFound` if the template does not exist.
- `render_template_string()` is vulnerable to SSTI if the template string includes user input .
- Both functions require an active application context.

### Annotated Code Examples

**Example 1: Rendering a Template File**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html", title="Home", user="Alice")

if __name__ == "__main__":
    app.run(debug=True)
```

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head><title>{{ title }}</title></head>
<body>
    <h1>Hello, {{ user }}!</h1>
</body>
</html>
```

**Expected Output:**
- `GET /` → HTML page with `<title>Home</title>` and `<h1>Hello, Alice!</h1>`.

**Why this output:** `render_template()` loads `index.html` from the `templates/` directory and replaces `{{ title }}` and `{{ user }}` with the provided context values.

**Example 2: Rendering a Template String**

```python
from flask import Flask, render_template_string

app = Flask(__name__)

@app.route("/greet/<name>")
def greet(name):
    template = "<h1>Hello, {{ name }}!</h1>"
    return render_template_string(template, name=name)
```

**Expected Output:**
- `GET /greet/Alice` → `<h1>Hello, Alice!</h1>`.

**Why this output:** `render_template_string()` renders the string directly, substituting `{{ name }}` with the URL parameter value.

### Real-World Cases

- **Email templates:** Rendering HTML email bodies from strings.
- **Dynamic widgets:** Rendering small HTML fragments on the fly.
- **Report generation:** Creating HTML or text reports from templates.
- **Testing:** Quickly rendering templates without creating files.

### References

- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template
- Flask `render_template_string` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template_string
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/

---

## 3. Template Directories (Custom Search Paths and Blueprint Folders)

### Definitions

**Core Definition:** Template directories are the filesystem locations where Flask searches for template files. The default is a `templates/` folder in the application root, but custom paths and blueprint-specific folders can be added.

**Technical Definition:** Flask creates a Jinja2 `FileSystemLoader` pointing to the application's `template_folder` (default: `"templates"`). When a blueprint is registered, its `template_folder` is added to a `ChoiceLoader` that searches application templates first, then blueprint templates . Custom search paths can be added by modifying `app.jinja_loader.searchpath` or by configuring the `template_folder` parameter on the `Flask` constructor .

**Beginner-Friendly Explanation:** Flask looks for templates in a `templates/` folder by default. You can tell it to look in other folders too, and blueprints can have their own template folders. This lets you organize templates by module in large applications.

### Purposes

- To organize templates in a logical directory structure.
- To allow blueprints to have their own templates for modularity.
- To add custom template search paths for shared or external templates.
- To override application templates with blueprint-specific versions.
- To support large applications with multiple template sources.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Default template folder
app = Flask(__name__)

# Custom template folder
app = Flask(__name__, template_folder="my_templates")

# Blueprint with template folder
bp = Blueprint("admin", __name__, template_folder="admin_templates")

# Adding a custom search path
import os
app.jinja_loader.searchpath.append(os.path.join(app.root_path, "shared_templates"))
```

**Component Breakdown:**

| Configuration | Description |
|---------------|-------------|
| `template_folder` | Flask constructor parameter for the default folder |
| Blueprint `template_folder` | Blueprint constructor parameter for its templates |
| `app.jinja_loader.searchpath` | List of directories searched for templates |

**Syntax Rules:**

- The default `template_folder` is `"templates"` relative to the application root.
- Blueprint templates are searched after application templates.
- Custom paths added to `searchpath` are searched in order; first match wins.
- Blueprint template folders must be specified when creating the blueprint.

**Constraints and Limitations:**

- Blueprint templates have lower priority than application templates .
- Template names must be unique across all search paths or conflicts occur.
- The search path is fixed at startup; adding paths at runtime requires reinitializing the loader.

### Annotated Code Examples

**Example 1: Blueprint-Specific Templates**

```python
from flask import Flask, Blueprint, render_template

admin = Blueprint("admin", __name__, template_folder="admin_templates")

@admin.route("/")
def admin_index():
    return render_template("admin/index.html")

app = Flask(__name__)
app.register_blueprint(admin)

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /` (admin blueprint) → renders `admin_templates/admin/index.html`.

**Why this output:** The blueprint's `template_folder` is added to the search path. When `render_template("admin/index.html")` is called, Flask searches the application's `templates/` first, then the blueprint's `admin_templates/`.

### Real-World Cases

- **Modular applications:** Each blueprint has its own templates for independent development.
- **Theme support:** Custom template paths allow different themes to be swapped.
- **Extension templates:** Third-party extensions can ship their own templates.

### References

- Flask Blueprints: Templates — https://flask.palletsprojects.com/en/stable/blueprints/#templates
- Flask `Flask.template_folder` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_folder
- Flask `Blueprint.template_folder` — https://flask.palletsprojects.com/en/stable/api/#flask.Blueprint.template_folder

---

## 4. Template Variables

### Definitions

**Core Definition:** Template variables are values passed from Python code to the template context, accessible within the template using `{{ variable_name }}`.

**Technical Definition:** When `render_template()` is called with keyword arguments, those arguments become variables in the Jinja2 context. Jinja2 also provides a standard context that includes `config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()` . Variables can be any Python object: strings, numbers, lists, dictionaries, or custom objects. Jinja2 supports attribute access (`{{ user.name }}`), item access (`{{ data['key'] }}`), and method calls (`{{ user.get_name() }}`).

**Beginner-Friendly Explanation:** Template variables are the blanks you fill in. You pass them from your Python view function to the template, and the template displays them using `{{ }}`. For example, if you pass `name="Alice"`, the template can display `{{ name }}` to show "Alice".

### Purposes

- To display dynamic data in templates.
- To pass complex objects (lists, dictionaries) for iteration and conditional rendering.
- To access Flask's standard context (request, session, config) in templates.
- To use filters and functions on variables.
- To generate personalized content for each user.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Passing variables from Python
return render_template("page.html", name="Alice", age=30, items=[1, 2, 3])
```

```jinja
{# Using variables in templates #}
{{ name }}
{{ age + 1 }}
{{ items[0] }}
{% for item in items %}{{ item }}{% endfor %}
{{ user.name }}
```

**Component Breakdown:**

| Variable Type | Access Pattern |
|---------------|----------------|
| String/Number | `{{ var }}` |
| List | `{{ list[0] }}`, `{% for x in list %}` |
| Dictionary | `{{ dict['key'] }}`, `{{ dict.key }}` |
| Object | `{{ obj.attr }}`, `{{ obj.method() }}` |
| Standard context | `{{ config.DEBUG }}`, `{{ request.path }}` |

**Syntax Rules:**

- Variables are passed as keyword arguments to `render_template()`.
- Undefined variables render as empty strings by default (configurable).
- Use `{{ variable|default('fallback') }}` for optional variables.
- Attribute access uses dots; item access uses brackets.

**Constraints and Limitations:**

- Templates cannot modify variables passed from Python (they are copies in the context).
- Complex logic should be in Python, not templates.
- Undefined variables may cause silent errors; use `default` filter or enable `StrictUndefined`.

### Annotated Code Examples

**Example 1: Displaying Variables**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/profile/<username>")
def profile(username):
    user = {"name": username, "age": 30, "hobbies": ["reading", "coding"]}
    return render_template("profile.html", user=user)

if __name__ == "__main__":
    app.run(debug=True)
```

```html
<!-- templates/profile.html -->
<h1>{{ user.name }}</h1>
<p>Age: {{ user.age }}</p>
<ul>
{% for hobby in user.hobbies %}
    <li>{{ hobby }}</li>
{% endfor %}
</ul>
```

**Expected Output:**
- `GET /profile/alice` → `<h1>alice</h1><p>Age: 30</p><ul><li>reading</li><li>coding</li></ul>`.

**Why this output:** The `user` dictionary is passed to the template. `{{ user.name }}` accesses the `name` key, and the `{% for %}` loop iterates over the `hobbies` list.

**Example 2: Using Standard Context**

```html
<p>Current path: {{ request.path }}</p>
<p>Debug mode: {{ config.DEBUG }}</p>
<p>Logged in: {{ session.get('username', 'Guest') }}</p>
```

**Expected Output:**
- The template displays the request path, debug mode, and logged-in username (or "Guest").

**Why this output:** Flask automatically injects `request`, `config`, and `session` into the template context, making them available without explicit passing.

### Real-World Cases

- **User profiles:** Displaying user names, emails, and preferences.
- **Product pages:** Showing product details, prices, and images.
- **Dashboards:** Rendering metrics and charts with dynamic data.
- **Navigation:** Using `request.path` to highlight the active menu item.

### References

- Flask Templating: Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context
- Jinja2 Variables — https://jinja.palletsprojects.com/templates/#variables

---

## 5. Template Inheritance

### Definitions

**Core Definition:** Template inheritance is a Jinja2 feature that allows child templates to extend a base template and override specific blocks, enabling consistent layouts across multiple pages.

**Technical Definition:** A base template defines `{% block %}` placeholders that child templates can override. A child template uses `{% extends "base.html" %}` as its first tag to inherit from the base template, then defines `{% block name %}...{% endblock %}` to fill in the blocks. The `{{ super() }}` function renders the parent block's content. Template inheritance works similarly to object-oriented inheritance .

**Beginner-Friendly Explanation:** You create a base template with the common parts of your site (header, footer, navigation). Then each page extends the base and fills in only the unique content. This means you don't have to copy the header and footer into every page.

### Purposes

- To eliminate code duplication across templates.
- To maintain consistent layouts across all pages.
- To allow page-specific content within a shared structure.
- To simplify updates (change the base template, all pages update).
- To organize templates hierarchically.

### Syntax Rules and Structure

**Complete General Syntax:**

**Base Template (`base.html`):**

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <header>{% block header %}{% endblock %}</header>
    <main>{% block content %}{% endblock %}</main>
    <footer>{% block footer %}© 2024{% endblock %}</footer>
</body>
</html>
```

**Child Template (`home.html`):**

```html
{% extends "base.html" %}

{% block title %}Home | My Site{% endblock %}

{% block content %}
    <h1>Welcome!</h1>
    <p>This is the home page.</p>
{% endblock %}
```

**Component Breakdown:**

| Tag | Description |
|-----|-------------|
| `{% extends "base.html" %}` | Inherits from the base template |
| `{% block name %}...{% endblock %}` | Defines a named block |
| `{{ super() }}` | Renders the parent block's content |

**Syntax Rules:**

- `{% extends %}` must be the first tag in the child template.
- Blocks with the same name override the parent's blocks.
- The base template's default block content is used if not overridden.
- `{{ super() }}` includes the parent's block content.

**Constraints and Limitations:**

- Child templates cannot define content outside of blocks.
- Changes to the base template affect all child templates.
- Deep inheritance chains can be hard to debug.

### Annotated Code Examples

**Example 1: Base and Child Templates**

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('about') }}">About</a>
    </nav>
    {% block content %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/home.html -->
{% extends "base.html" %}

{% block title %}Home | My Site{% endblock %}

{% block content %}
    <h1>Welcome to the Home Page</h1>
{% endblock %}
```

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("home.html")

@app.route("/about")
def about():
    return render_template("about.html")

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /` → full HTML page with navigation, title "Home | My Site", and welcome heading.
- `GET /about` → full HTML page with navigation and about content.

**Why this output:** `home.html` extends `base.html`, inheriting the navigation and structure. Only the `title` and `content` blocks are overridden.

### Real-World Cases

- **Multi-page websites:** Consistent header and footer across all pages.
- **Admin panels:** Shared layout with page-specific content.
- **Documentation sites:** Consistent sidebar and navigation.
- **E-commerce:** Shared product page layout with variable content.

### References

- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/templates/#template-inheritance

---

## 6. Environment Configuration (Auto-Reload, Cache Settings)

### Definitions

**Core Definition:** Jinja2 environment configuration controls how templates are loaded, cached, and reloaded, with different settings optimal for development versus production.

**Technical Definition:** Flask exposes Jinja2 environment settings via configuration variables: `TEMPLATES_AUTO_RELOAD` (whether to check for template changes on each request), `EXPLAIN_TEMPLATE_LOADING` (debug logging for template resolution), and `SEND_FILE_MAX_AGE_DEFAULT` (static file caching). The Jinja2 `Environment` also has `cache_size` (default 400) and `auto_reload` (default True) parameters . In development, `TEMPLATES_AUTO_RELOAD` is automatically enabled when `DEBUG=True`; in production, it should be disabled for performance .

**Beginner-Friendly Explanation:** During development, you want Flask to notice when you change a template and reload it automatically. In production, you want templates cached for speed. Flask lets you control this with configuration settings.

### Purposes

- To enable automatic template reloading during development.
- To disable reloading in production for performance.
- To control template caching to balance memory and speed.
- To debug template resolution issues with `EXPLAIN_TEMPLATE_LOADING`.
- To optimize static file caching for production.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
app = Flask(__name__)

# Development: auto-reload templates
app.config["TEMPLATES_AUTO_RELOAD"] = True

# Production: disable auto-reload (default outside debug mode)
app.config["TEMPLATES_AUTO_RELOAD"] = False

# Disable static file caching in development
app.config["SEND_FILE_MAX_AGE_DEFAULT"] = 0

# Debug template loading
app.config["EXPLAIN_TEMPLATE_LOADING"] = True
```

**Component Breakdown:**

| Configuration | Default | Description |
|---------------|---------|-------------|
| `TEMPLATES_AUTO_RELOAD` | `None` (follows `DEBUG`) | Reload templates on change |
| `SEND_FILE_MAX_AGE_DEFAULT` | `None` (12 hours) | Static file cache age |
| `EXPLAIN_TEMPLATE_LOADING` | `False` | Log template resolution |

**Syntax Rules:**

- `TEMPLATES_AUTO_RELOAD` defaults to the value of `DEBUG` if not set.
- In production, set `TEMPLATES_AUTO_RELOAD = False` for performance.
- `SEND_FILE_MAX_AGE_DEFAULT = 0` disables browser caching of static files.
- `EXPLAIN_TEMPLATE_LOADING` logs which loader resolved each template.

**Constraints and Limitations:**

- Auto-reloading adds overhead; disable in production.
- Template caching uses memory; large numbers of templates increase memory usage.
- `EXPLAIN_TEMPLATE_LOADING` should be disabled in production (performance and security).

### Annotated Code Examples

**Example 1: Development Configuration**

```python
from flask import Flask

app = Flask(__name__)
app.config["TEMPLATES_AUTO_RELOAD"] = True
app.config["SEND_FILE_MAX_AGE_DEFAULT"] = 0
app.config["EXPLAIN_TEMPLATE_LOADING"] = True

@app.route("/")
def index():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- Templates reload automatically when changed.
- Static files are not cached by the browser.
- Template loading is logged to the console.

**Why this output:** These settings are optimized for development: auto-reload for immediate feedback, no static caching for fresh assets, and logging for debugging.

**Example 2: Production Configuration**

```python
app = Flask(__name__)
app.config["TEMPLATES_AUTO_RELOAD"] = False
app.config["SEND_FILE_MAX_AGE_DEFAULT"] = 31536000  # 1 year
```

**Expected Output:**
- Templates are cached and not reloaded on change.
- Static files are cached by the browser for one year.

**Why this output:** These settings are optimized for production: no reloading overhead, and long-lived static file caching for performance.

### Real-World Cases

- **Development:** Auto-reload templates for rapid iteration.
- **Production:** Cache templates and static files for performance.
- **Debugging:** Use `EXPLAIN_TEMPLATE_LOADING` to diagnose template resolution issues.

### References

- Flask Configuration: `TEMPLATES_AUTO_RELOAD` — https://flask.palletsprojects.com/en/stable/config/#TEMPLATES_AUTO_RELOAD
- Flask Configuration: `SEND_FILE_MAX_AGE_DEFAULT` — https://flask.palletsprojects.com/en/stable/config/#SEND_FILE_MAX_AGE_DEFAULT
- Flask Configuration: `EXPLAIN_TEMPLATE_LOADING` — https://flask.palletsprojects.com/en/stable/config/#EXPLAIN_TEMPLATE_LOADING
- Jinja2 Environment Documentation — https://jinja.palletsprojects.com/api/#jinja2.Environment

---

## References

- Jinja2 Template Documentation — https://jinja.palletsprojects.com/templates/
- Jinja2 Environment Documentation — https://jinja.palletsprojects.com/api/#jinja2.Environment
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/
- Flask `render_template` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template
- Flask `render_template_string` — https://flask.palletsprojects.com/en/stable/api/#flask.render_template_string
- Flask Blueprints — https://flask.palletsprojects.com/en/stable/blueprints/
- Flask Configuration — https://flask.palletsprojects.com/en/stable/config/
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/templates/#template-inheritance