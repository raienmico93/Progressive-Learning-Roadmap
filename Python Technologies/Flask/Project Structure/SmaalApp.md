# Flask Small Application Structure: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Small application structure in Flask refers to the organizational patterns used for small-to-medium Flask projects, ranging from a single Python file to a minimal package with separate modules for templates, static files, and application logic.

**Technical Definition:** Flask is a microframework that imposes no required project structure. The smallest valid Flask application is a single Python file containing the `Flask` instance and route definitions. As complexity grows, the recommended pattern is to transition to a **package structure** (a directory with an `__init__.py`) using the **application factory pattern** (`create_app()`), with separate directories for `templates/`, `static/`, and optionally Blueprints. Flask auto-discovers `templates/` and `static/` relative to the application's root path (the location of the module or package).

**Beginner-Friendly Explanation:** When you first start with Flask, you can put everything in one Python file. That works great for small projects. But as your app grows—more routes, more templates, more static files—it gets messy. At that point, you reorganize into a folder structure with separate files for different concerns. This cheat sheet shows you when and how to make that transition.

### Key Characteristics

- **No imposed structure:** Flask does not require any particular project layout; you choose based on project size.
- **Single-file simplicity:** The smallest Flask app is one `.py` file with `app = Flask(__name__)` and route definitions.
- **Package transition:** When a single file becomes unwieldy, transition to a package (`myapp/__init__.py`) with the application factory pattern.
- **Automatic discovery:** Flask automatically looks for `templates/` and `static/` directories relative to the application root.
- **Blueprint support:** For larger small apps, Blueprints modularize routes without the full weight of a large application structure.
- **Instance folder:** The `instance/` folder provides a deployment-specific location for configuration and data files.
- **Environment files:** `.flaskenv` and `.env` files live in the project root for Flask CLI and environment variables.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Python modules and packages.
- Familiarity with Flask routing and view functions.
- Optional: `python-dotenv` for `.flaskenv` and `.env` support.

### Related Programming Areas

- **Application factory pattern:** The `create_app()` function for instantiating apps with different configurations.
- **Blueprints:** Modular route organization for medium-sized applications.
- **Package management:** `__init__.py`, `setup.py`, and `pyproject.toml` for distributable packages.
- **Deployment:** Containerization (Docker) and WSGI servers (Gunicorn) with the project structure.
- **Testing:** `tests/` directory with pytest fixtures for the application factory.

### Core Concepts / Features

1. Single-File Applications
2. Basic Package Structure (Application Factory)
3. Templates (Jinja2 Template Organization)
4. Static Files (CSS, JavaScript, Images)
5. Flat vs. Nested Module Layouts

---

## 1. Single-File Applications

### Definitions

**Core Definition:** A single-file Flask application is a project where the entire application—routes, configuration, and helpers—lives in one Python file, typically `app.py` or `main.py`.

**Technical Definition:** The minimal Flask application is created by instantiating `Flask(__name__)` and using `@app.route()` decorators to define view functions. All configuration is set via `app.config`, and helpers are defined as module-level functions. Templates and static files are stored in `templates/` and `static/` directories relative to the file. This structure is sufficient for prototypes, small APIs, and simple web applications.

**Beginner-Friendly Explanation:** A single-file app is exactly what it sounds like—everything in one file. You import Flask, create the app, define your routes, and run it. This is perfect for small projects, learning, and quick prototypes.

### Purposes

- To prototype ideas quickly without project overhead.
- To learn Flask concepts without distraction.
- To build small APIs or single-page apps.
- To create minimal reproducible examples for debugging or sharing.
- To serve as the starting point for larger applications.

### Syntax Rules and Structure

**Complete General Syntax (`app.py`):**

```python
from flask import Flask, render_template, jsonify, request

app = Flask(__name__)
app.config['SECRET_KEY'] = 'dev-key'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/data')
def api_data():
    return jsonify({'data': 'value'})

@app.route('/submit', methods=['POST'])
def submit():
    data = request.get_json()
    return jsonify({'received': data}), 201

if __name__ == '__main__':
    app.run(debug=True)
```

**Directory Structure:**

```
myproject/
├── app.py              # The entire application
├── templates/
│   └── index.html
├── static/
│   └── style.css
└── .flaskenv           # Optional: FLASK_APP=app.py
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `app.py` | The single application file |
| `app = Flask(__name__)` | The Flask application instance |
| `@app.route()` | Route decorators |
| `templates/` | Jinja2 templates directory |
| `static/` | Static assets directory |
| `if __name__ == '__main__':` | Development server entry point |

**Syntax Rules:**

- `Flask(__name__)` uses the current module's name to locate templates and static files.
- Templates are in a `templates/` folder next to `app.py`.
- Static files are in a `static/` folder next to `app.py`.
- The `if __name__ == '__main__':` block runs the development server.
- Use `flask --app app run` or `python app.py` to start the server.

**Constraints and Limitations:**

- All routes, config, and helpers in one file become unwieldy as the app grows.
- No separation of concerns (models, views, config).
- Difficult to test individual components.
- Circular imports can occur if the file grows too large.

### Annotated Code Examples

**Example 1: Minimal Single-File App**

```python
from flask import Flask, render_template, jsonify, request

app = Flask(__name__)
app.config['SITE_NAME'] = 'My Small App'

@app.route('/')
def index():
    return render_template('index.html', site_name=app.config['SITE_NAME'])

@app.route('/api/greet/<name>')
def greet(name):
    return jsonify({'message': f'Hello, {name}!'})

@app.route('/api/echo', methods=['POST'])
def echo():
    data = request.get_json(silent=True)
    if data is None:
        return jsonify({'error': 'Invalid JSON'}), 400
    return jsonify({'echo': data}), 201

@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

**Expected Output:**
- `GET /` → renders `index.html` with `site_name="My Small App"`.
- `GET /api/greet/Alice` → `{"message": "Hello, Alice!"}`.
- `POST /api/echo` with `{"key": "value"}` → `{"echo": {"key": "value"}}` with status `201`.
- `GET /nonexistent` → `{"error": "Not found"}` with status `404`.

**Why this output:** All routes and error handlers are defined in one file. Flask's `__name__` points to the current module, so `templates/` and `static/` are resolved relative to `app.py`.

### Real-World Cases

- **Prototypes:** Testing an idea in minutes.
- **Small APIs:** A REST API with a handful of endpoints.
- **Microservices:** A single-purpose service.
- **Learning:** Teaching Flask fundamentals.
- **Reproducible examples:** Sharing minimal code for bug reports.

### References

- Flask Quickstart — https://flask.palletsprojects.com/en/stable/quickstart/
- Flask Minimal Application — https://flask.palletsprojects.com/en/stable/quickstart/#a-minimal-application

---

## 2. Basic Package Structure (Application Factory)

### Definitions

**Core Definition:** A basic package structure organizes a Flask application as a Python package—a directory with an `__init__.py` file—using the application factory pattern to create the app instance and separate concerns into modules.

**Technical Definition:** The package structure uses `create_app()` (the application factory) inside `myapp/__init__.py` to instantiate and configure the Flask application. Routes are organized into Blueprints (e.g., `myapp/auth.py`, `myapp/blog.py`), configuration is in `myapp/config.py` or loaded from environment variables, and extensions are initialized in `myapp/extensions.py` and bound to the app via `init_app()`. Templates and static files live in `myapp/templates/` and `myapp/static/`, or at the project root for larger projects.

**Beginner-Friendly Explanation:** A package structure breaks your app into multiple files inside a folder. Instead of one giant `app.py`, you have a folder called `myapp/` with an `__init__.py` that creates the app, separate files for different features (like `auth.py` and `blog.py`), and a `config.py` for settings. This keeps your code organized and easier to maintain.

### Purposes

- To separate concerns (routes, config, extensions, models).
- To support multiple configurations (development, testing, production).
- To enable testing with different app instances.
- To avoid circular imports as the app grows.
- To support Blueprints for modular features.
- To prepare for larger application structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```
myproject/
├── myapp/
│   ├── __init__.py          # Application factory
│   ├── config.py            # Configuration classes
│   ├── extensions.py        # Extension instances
│   ├── auth.py              # Auth blueprint
│   ├── blog.py              # Blog blueprint
│   ├── templates/
│   │   ├── base.html
│   │   ├── auth/
│   │   │   └── login.html
│   │   └── blog/
│   │       └── index.html
│   └── static/
│       ├── css/
│       │   └── style.css
│       └── js/
│           └── main.js
├── tests/
│   ├── conftest.py
│   └── test_auth.py
├── instance/
│   └── config.py            # Deployment-specific config (gitignored)
├── .flaskenv                # FLASK_APP=myapp
├── .env                     # Secrets (gitignored)
├── requirements.txt
└── run.py                   # Optional: entry point
```

**`myapp/__init__.py` (Application Factory):**

```python
from flask import Flask
from .config import DevelopmentConfig
from .extensions import db, login_manager

def create_app(config_object=DevelopmentConfig):
    app = Flask(__name__, instance_relative_config=True)
    app.config.from_object(config_object)
    
    # Load instance config if it exists
    app.config.from_pyfile('config.py', silent=True)
    
    # Initialize extensions
    db.init_app(app)
    login_manager.init_app(app)
    
    # Register blueprints
    from .auth import auth_bp
    from .blog import blog_bp
    app.register_blueprint(auth_bp)
    app.register_blueprint(blog_bp)
    
    # Register error handlers
    from .errors import register_error_handlers
    register_error_handlers(app)
    
    return app
```

**`myapp/config.py`:**

```python
import os

class BaseConfig:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-key')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(BaseConfig):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'

class ProductionConfig(BaseConfig):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
    SESSION_COOKIE_SECURE = True

class TestingConfig(BaseConfig):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False
```

**`myapp/extensions.py`:**

```python
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

db = SQLAlchemy()
login_manager = LoginManager()
```

**`myapp/auth.py`:**

```python
from flask import Blueprint, render_template, request, redirect, url_for
from .extensions import db

auth_bp = Blueprint('auth', __name__, url_prefix='/auth')

@auth_bp.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Handle login
        pass
    return render_template('auth/login.html')
```

**`run.py`:**

```python
from myapp import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `myapp/__init__.py` | Application factory (`create_app`) |
| `myapp/config.py` | Configuration classes |
| `myapp/extensions.py` | Extension instances |
| `myapp/auth.py` | Auth blueprint |
| `myapp/templates/` | Jinja2 templates |
| `myapp/static/` | Static assets |
| `instance/` | Deployment-specific config (gitignored) |
| `tests/` | Test suite |
| `.flaskenv` | Flask CLI environment variables |

**Syntax Rules:**

- `create_app()` is the recommended application factory.
- Extensions are instantiated in `extensions.py` and bound via `init_app()`.
- Blueprints are registered inside `create_app()`.
- The `instance/` folder is resolved relative to the package when `instance_relative_config=True`.
- `.flaskenv` sets `FLASK_APP=myapp` so the CLI finds the factory.

**Constraints and Limitations:**

- More files and structure than a single-file app.
- Requires understanding of Python packages and circular imports.
- The application factory pattern requires tests to create app instances explicitly.

### Annotated Code Examples

**Example 1: Complete Minimal Package**

```
myproject/
├── myapp/
│   ├── __init__.py
│   ├── config.py
│   ├── routes.py
│   ├── templates/
│   │   └── index.html
│   └── static/
│       └── style.css
├── .flaskenv
└── run.py
```

```python
# myapp/__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)
    app.config.from_pyfile('config.py', silent=True)
    
    from .routes import main
    app.register_blueprint(main)
    
    return app
```

```python
# myapp/routes.py
from flask import Blueprint, render_template

main = Blueprint('main', __name__)

@main.route('/')
def index():
    return render_template('index.html')
```

```python
# myapp/config.py
SECRET_KEY = 'dev-key'
DEBUG = True
```

```ini
# .flaskenv
FLASK_APP=myapp
FLASK_DEBUG=1
```

```python
# run.py
from myapp import create_app

app = create_app()

if __name__ == '__main__':
    app.run()
```

**Expected Output:**
- `flask run` → starts the development server.
- `GET /` → renders `myapp/templates/index.html`.

**Why this output:** The application factory creates the app, loads configuration, and registers the blueprint. Flask CLI finds the factory via `FLASK_APP=myapp`.

### Real-World Cases

- **Small-to-medium web apps:** Blogs, dashboards, and simple CRMs.
- **APIs with multiple resources:** Separating routes by resource.
- **Multi-environment deployments:** Using different config classes.
- **Testable applications:** Creating app instances in tests.

### References

- Flask Application Factories — https://flask.palletsprojects.com/en/stable/patterns/appfactories/
- Flask Tutorial: Application Setup — https://flask.palletsprojects.com/en/stable/tutorial/factory/
- Flask Blueprints — https://flask.palletsprojects.com/en/stable/blueprints/

---

## 3. Templates (Jinja2 Template Organization)

### Definitions

**Core Definition:** Templates in Flask are Jinja2 files stored in a `templates/` directory, rendered by `render_template()`, and organized by feature, blueprint, or shared layout.

**Technical Definition:** Flask automatically registers a Jinja2 `FileSystemLoader` pointing to the `templates/` directory relative to the application's root path. For Blueprints, a separate `template_folder` can be specified, and its templates are added to a `ChoiceLoader`. Templates use `{% extends %}`, `{% block %}`, and `{% include %}` for inheritance and reuse. Static assets are referenced via `url_for('static', filename='...')`.

**Beginner-Friendly Explanation:** Templates are your HTML files with placeholders for dynamic content. You keep them in a `templates/` folder. As your app grows, you can organize them into subfolders (like `auth/` and `blog/`) to keep things tidy.

### Purposes

- To separate presentation (HTML) from logic (Python).
- To reuse common layout elements through inheritance.
- To organize templates by feature or blueprint.
- To enable Blueprint-specific templates.
- To support internationalization and theming.

### Syntax Rules and Structure

**Small App (flat templates):**

```
templates/
├── base.html
├── index.html
├── about.html
└── contact.html
```

**Medium App (nested by feature):**

```
templates/
├── base.html
├── errors/
│   ├── 404.html
│   └── 500.html
├── auth/
│   ├── login.html
│   └── register.html
├── blog/
│   ├── index.html
│   ├── post.html
│   └── edit.html
└── partials/
    ├── header.html
    ├── footer.html
    └── nav.html
```

**Blueprint-Specific Templates:**

```
myapp/
├── templates/          # Application-level templates
│   └── base.html
├── auth/
│   └── templates/
│       └── auth/
│           └── login.html
└── blog/
    └── templates/
        └── blog/
            └── index.html
```

**Component Breakdown:**

| Directory | Purpose |
|-----------|---------|
| `templates/` | Application-level templates |
| `templates/errors/` | Error page templates |
| `templates/auth/` | Authentication templates |
| `templates/partials/` | Reusable fragments (header, footer) |
| `blueprint/templates/` | Blueprint-specific templates |

**Syntax Rules:**

- Templates are resolved relative to the `templates/` folder.
- Blueprint templates are searched after application templates.
- Use `{% extends "base.html" %}` for inheritance.
- Use `{% include "partials/header.html" %}` for reusable fragments.
- Reference static files with `url_for('static', filename='...')`.

**Constraints and Limitations:**

- Template names must be unique across search paths.
- Deeply nested templates can make paths long.
- Blueprint templates have lower priority than application templates.

### Annotated Code Examples

**Example 1: Organized Templates with Inheritance**

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    {% include 'partials/nav.html' %}
    <main>{% block content %}{% endblock %}</main>
    {% include 'partials/footer.html' %}
</body>
</html>
```

```html
<!-- templates/blog/index.html -->
{% extends "base.html" %}
{% block title %}Blog | My Site{% endblock %}
{% block content %}
    <h1>Blog Posts</h1>
    {% for post in posts %}
        <article>
            <h2>{{ post.title }}</h2>
            <p>{{ post.excerpt }}</p>
        </article>
    {% endfor %}
{% endblock %}
```

**Expected Output:**
- `GET /blog/` → HTML page with the base layout, navigation, blog posts, and footer.

**Why this output:** The `blog/index.html` template extends `base.html`, inheriting the layout. The `partials/nav.html` and `partials/footer.html` are included for reuse.

### Real-World Cases

- **Multi-page websites:** Shared layout via base template.
- **Blogs and CMS:** Templates organized by content type.
- **Admin panels:** Separate template hierarchy for admin pages.
- **Error pages:** Custom 404 and 500 templates.

### References

- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance

---

## 4. Static Files (CSS, JavaScript, Images)

### Definitions

**Core Definition:** Static files in Flask are assets such as CSS, JavaScript, images, and fonts that are served directly from the `static/` directory without server-side processing.

**Technical Definition:** Flask automatically registers a `static` endpoint that serves files from the `static/` directory (default) at the `/static/<path:filename>` URL. In templates, `url_for('static', filename='...')` generates the correct URL. For Blueprints, a separate `static_folder` and `static_url_path` can be specified. In production, static files are typically served by a web server (Nginx) or CDN rather than Flask.

**Beginner-Friendly Explanation:** Static files are your CSS, JavaScript, images, and fonts—the files that don't change based on user input. You put them in a `static/` folder, and Flask serves them to the browser. You reference them in templates with `url_for('static', filename='...')`.

### Purposes

- To serve CSS for styling.
- To serve JavaScript for interactivity.
- To serve images, icons, and fonts.
- To organize assets for maintainability.
- To enable caching and CDN integration.

### Syntax Rules and Structure

**Small App (flat static):**

```
static/
├── style.css
├── main.js
└── logo.png
```

**Medium App (organized static):**

```
static/
├── css/
│   ├── style.css
│   ├── forms.css
│   └── vendor/
│       └── bootstrap.min.css
├── js/
│   ├── main.js
│   ├── forms.js
│   └── vendor/
│       └── jquery.min.js
├── images/
│   ├── logo.png
│   └── icons/
│       └── favicon.ico
└── fonts/
    ├── roboto.woff2
    └── roboto.woff
```

**Referencing Static Files:**

```html
<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
<script src="{{ url_for('static', filename='js/main.js') }}"></script>
<img src="{{ url_for('static', filename='images/logo.png') }}" alt="Logo">
```

**Component Breakdown:**

| Directory | Purpose |
|-----------|---------|
| `static/css/` | Stylesheets |
| `static/js/` | JavaScript files |
| `static/images/` | Images |
| `static/fonts/` | Web fonts |
| `static/vendor/` | Third-party libraries |

**Syntax Rules:**

- The `static` endpoint is automatically registered.
- `filename` is relative to the `static/` folder.
- Use forward slashes in paths, even on Windows.
- Blueprint static files are accessed via `url_for('blueprint.static', ...)`.

**Constraints and Limitations:**

- Flask's built-in static serving is not recommended for production.
- The default cache timeout is 12 hours (`SEND_FILE_MAX_AGE_DEFAULT`).
- Static files are not processed by Jinja2.

### Annotated Code Examples

**Example 1: Organized Static Files**

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/vendor/bootstrap.min.css') }}">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    {% block content %}{% endblock %}
    <script src="{{ url_for('static', filename='js/vendor/jquery.min.js') }}"></script>
    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
</body>
</html>
```

**Expected Output:**
- The HTML page links to `/static/css/vendor/bootstrap.min.css`, `/static/css/style.css`, `/static/js/vendor/jquery.min.js`, and `/static/js/main.js`.

**Why this output:** `url_for('static', filename='...')` generates the correct URLs for each static asset, and the browser loads them.

### Real-World Cases

- **Any web application:** CSS, JavaScript, and images.
- **Multi-theme apps:** Theme-specific static folders.
- **CDN integration:** Serving static files from a CDN.

### References

- Flask Static Files — https://flask.palletsprojects.com/en/stable/quickstart/#static-files
- Flask Blueprint Static Files — https://flask.palletsprojects.com/en/stable/blueprints/#static-files

---

## 5. Flat vs. Nested Module Layouts

### Definitions

**Core Definition:** Flat vs. nested module layouts describe two approaches to organizing a Flask project: a flat layout places all application files at the top level, while a nested layout organizes them into a package directory with submodules.

**Technical Definition:** The **flat layout** places `app.py`, `models.py`, `views.py`, and other modules directly in the project root. The **nested layout** creates a package directory (e.g., `myapp/`) containing `__init__.py`, `models.py`, `views.py`, and subpackages. The nested layout is recommended for larger applications because it prevents name collisions, supports the application factory pattern, and enables proper Python package distribution. Flask's own documentation and tutorials use the nested layout for the tutorial application.

**Beginner-Friendly Explanation:** A flat layout means all your Python files are side by side in one folder. It's simple but gets messy. A nested layout puts your app inside its own folder (a "package"), which keeps things organized and lets you use imports like `from myapp.models import User`. When your app grows, you should switch from flat to nested.

### Purposes

- **Flat:** To keep small projects simple and easy to navigate.
- **Nested:** To organize larger projects with clear boundaries.
- **Nested:** To enable proper package distribution and testing.
- **Nested:** To avoid circular imports and name collisions.
- **Nested:** To support the application factory pattern.

### Syntax Rules and Structure

**Flat Layout:**

```
myproject/
├── app.py
├── models.py
├── views.py
├── forms.py
├── config.py
├── templates/
├── static/
└── requirements.txt
```

**Nested Layout:**

```
myproject/
├── myapp/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   ├── config.py
│   ├── templates/
│   └── static/
├── tests/
├── instance/
├── requirements.txt
└── .flaskenv
```

**Component Breakdown:**

| Aspect | Flat Layout | Nested Layout |
|--------|-------------|---------------|
| Simplicity | High | Medium |
| Scalability | Low | High |
| Package distribution | Difficult | Easy |
| Import paths | `from models import User` | `from myapp.models import User` |
| Testing | Harder to isolate | Easier with app factory |
| Circular imports | Common as app grows | Avoided with blueprints |

**Syntax Rules:**

- Start with a flat layout for prototypes and single-file apps.
- Transition to a nested layout when:
  - The single file exceeds ~200–300 lines.
  - You need multiple Blueprints.
  - You need environment-specific configuration.
  - You want to write tests with different app instances.
  - Multiple developers are working on the project.
- Use the application factory pattern in the nested layout.

**Constraints and Limitations:**

- The flat layout becomes unwieldy quickly.
- The nested layout requires understanding of Python packages.
- Refactoring from flat to nested requires updating import statements.

### Annotated Code Examples

**Example 1: Transitioning from Flat to Nested**

**Flat (`app.py`):**

```python
from flask import Flask, render_template
from models import User
from forms import LoginForm

app = Flask(__name__)

@app.route('/login')
def login():
    form = LoginForm()
    return render_template('login.html', form=form)

if __name__ == '__main__':
    app.run(debug=True)
```

**Nested (`myapp/__init__.py`):**

```python
from flask import Flask

def create_app():
    app = Flask(__name__)
    app.config.from_pyfile('config.py')
    
    from .auth import auth_bp
    app.register_blueprint(auth_bp)
    
    return app
```

**Nested (`myapp/auth.py`):**

```python
from flask import Blueprint, render_template
from .forms import LoginForm

auth_bp = Blueprint('auth', __name__, url_prefix='/auth')

@auth_bp.route('/login')
def login():
    form = LoginForm()
    return render_template('auth/login.html', form=form)
```

**Expected Output:**
- `GET /auth/login` → renders the login page.
- The nested structure keeps models, forms, and views in separate modules.

**Why this output:** The nested layout organizes the code into a package with Blueprints, making it easier to maintain as the app grows.

### Real-World Cases

- **Prototypes:** Start flat, then refactor.
- **Production applications:** Use nested layout with Blueprints.
- **Open-source packages:** Use nested layout for distribution.
- **Team projects:** Use nested layout for clear module boundaries.

### References

- Flask Tutorial: Application Setup — https://flask.palletsprojects.com/en/stable/tutorial/factory/
- Flask Application Factories — https://flask.palletsprojects.com/en/stable/patterns/appfactories/
- Python Packages — https://docs.python.org/3/tutorial/modules.html#packages
- Choosing a Project Layout (Real Python) — https://realpython.com/python-application-layouts/

---

## References

- Flask Quickstart — https://flask.palletsprojects.com/en/stable/quickstart/
- Flask Minimal Application — https://flask.palletsprojects.com/en/stable/quickstart/#a-minimal-application
- Flask Application Factories — https://flask.palletsprojects.com/en/stable/patterns/appfactories/
- Flask Tutorial: Application Setup — https://flask.palletsprojects.com/en/stable/tutorial/factory/
- Flask Blueprints — https://flask.palletsprojects.com/en/stable/blueprints/
- Flask Blueprint Static Files — https://flask.palletsprojects.com/en/stable/blueprints/#static-files
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/
- Flask Static Files — https://flask.palletsprojects.com/en/stable/quickstart/#static-files
- Flask Instance Folders — https://flask.palletsprojects.com/en/stable/config/#instance-folders
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Python Packages — https://docs.python.org/3/tutorial/modules.html#packages
- Choosing a Project Layout (Real Python) — https://realpython.com/python-application-layouts/
- Flask-Layouts (Nicolas Perriault) — https://github.com/nperriault/flask-layouts
- Flask Project Structure (GitHub) — https://github.com/JoMingyu/Flask-Project-Structure