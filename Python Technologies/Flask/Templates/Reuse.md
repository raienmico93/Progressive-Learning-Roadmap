# Flask Template Reuse: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Template reuse in Flask refers to the Jinja2 features that allow template code to be shared across multiple templates, eliminating duplication through includes, macros, custom filters, tests, and global functions.

**Technical Definition:** Jinja2 provides several mechanisms for template reuse: `{% include %}` inserts a template's rendered output into the current template; `{% import %}` and `{% from ... import ... %}` load macros from another template as a module; `{% macro %}` defines reusable parameterized template fragments; custom filters transform values in expressions; custom tests evaluate conditions; and custom globals expose functions to all templates. The `with context` and `without context` modifiers control whether the current template's context variables are passed to included or imported templates, with significant implications for caching and performance.

**Beginner-Friendly Explanation:** Template reuse means you write a piece of template code once and use it in many places. An include is like copy-pasting a fragment into your page. A macro is like a function you can call with different arguments. Custom filters let you transform values in new ways. Custom tests let you check conditions. And globals are functions available everywhere. These tools keep your templates DRY (Don't Repeat Yourself).

### Key Characteristics

- **Include vs. Import:** `{% include %}` renders a template inline; `{% import %}` loads macros as a module and is cached.
- **Context behavior:** Includes pass the current context by default; imports do not, for caching efficiency.
- **Macros as functions:** Macros accept arguments, have default values, and can access context when imported `with context`.
- **Custom filters:** Registered via `@app.template_filter()` or `app.jinja_env.filters`.
- **Custom tests:** Registered via `@app.template_test()` or `app.jinja_env.tests`.
- **Custom globals:** Registered via `@app.template_global()` or `app.jinja_env.globals`.
- **Caching and performance:** `without context` enables caching of includes; imports are cached by default.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Solid understanding of Jinja2 syntax (expressions, statements, filters, tests).
- Familiarity with Flask's `render_template()` and template context.
- Knowledge of Python functions, decorators, and modules.

### Related Programming Areas

- **Jinja2:** The underlying template engine providing include, import, macro, filter, test, and global features.
- **Flask Templating:** Flask integrates Jinja2 and provides decorators for registering custom filters, tests, and globals.
- **Template Inheritance:** Reuse complements inheritance; includes and macros work inside blocks.
- **Blueprints:** Blueprints can register their own filters, tests, and globals.
- **Performance optimization:** Context modifiers and caching affect rendering speed.

### Core Concepts / Features

1. Includes
2. Macros
3. Reusable Components
4. Custom Filters
5. Custom Tests and Global Functions
6. The `with context` and `without context` Modifiers

---

## 1. Includes

### Definitions

**Core Definition:** The `{% include %}` tag renders another template file inline within the current template, inserting its output at that location.

**Technical Definition:** `{% include 'template.html' %}` loads the specified template, renders it with the current context (by default), and inserts the rendered output into the parent template. The include tag supports `ignore missing` to suppress errors when the template does not exist, and `with context` / `without context` modifiers to control context propagation. By default, included templates are passed the current context, and the include is not cached. When `without context` is used, the included template does not receive the parent's context, and Jinja optimizes by caching the rendered output.

**Beginner-Friendly Explanation:** An include is like copy-pasting the content of another file into your template. It's useful for shared fragments like headers, footers, or sidebars that are identical across pages. The included file gets access to all the variables from the parent page by default.

### Purposes

- To insert reusable fragments (headers, footers, sidebars) into multiple templates.
- To break large templates into smaller, manageable pieces.
- To share common UI elements across pages without using inheritance.
- To conditionally include templates based on variables.
- To include templates with different contexts for caching optimization.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Basic include (passes current context) #}
{% include 'header.html' %}

{# Include with ignore missing #}
{% include 'sidebar.html' ignore missing %}

{# Include without context (enables caching) #}
{% include 'header.html' without context %}

{# Include with context (explicit) #}
{% include 'header.html' with context %}

{# Conditional include #}
{% include 'admin_panel.html' if user.is_admin else 'user_panel.html' %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% include 'path' %}` | Renders the specified template inline |
| `ignore missing` | Suppresses `TemplateNotFound` errors |
| `with context` | Passes current context (default) |
| `without context` | Does not pass context; enables caching |

**Syntax Rules:**

- The template path is relative to the templates directory.
- `ignore missing` must be placed before `with context` or `without context`.
- By default, the included template receives the current context.
- When `without context` is used, the included template can only access global variables (e.g., `config`, `request`).
- Includes cannot override blocks from the parent template.

**Constraints and Limitations:**

- Includes are rendered with the current context by default; this can cause unexpected behavior if variable names collide.
- `without context` disables caching benefits if the included template depends on context variables.
- Includes are not cached by default when context is passed.

### Annotated Code Examples

**Example 1: Basic Include**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<body>
    {% include 'partials/header.html' %}
    <main>{% block content %}{% endblock %}</main>
    {% include 'partials/footer.html' %}
</body>
</html>
```

```jinja
{# templates/partials/header.html #}
<header>
    <h1>{{ site_name }}</h1>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
    </nav>
</header>
```

**Expected Output (when `site_name = "My Site"`):**

```html
<!DOCTYPE html>
<html>
<body>
    <header>
        <h1>My Site</h1>
        <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
        </nav>
    </header>
    <main>...</main>
    <footer>...</footer>
</body>
</html>
```

**Why this output:** The `{% include 'partials/header.html' %}` tag renders the header template inline. The `site_name` variable from the parent context is available in the included template because includes pass context by default.

**Example 2: Include with `without context` for Caching**

```jinja
{# templates/page.html #}
{% include 'partials/static_banner.html' without context %}
```

```jinja
{# templates/partials/static_banner.html #}
<div class="banner">
    <p>Welcome to our website!</p>
</div>
```

**Expected Output:**

```html
<div class="banner">
    <p>Welcome to our website!</p>
</div>
```

**Why this output:** `without context` means the included template does not receive the parent's variables. Since the banner is static, Jinja can cache its rendered output, improving performance on subsequent renders.

### Real-World Cases

- **Headers and footers:** Identical across all pages.
- **Sidebars:** Shared navigation or advertisement panels.
- **Modal dialogs:** Reusable modal templates included on demand.
- **Analytics snippets:** Tracking code included in the base template.

### References

- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include
- Jinja2 Import Context Behavior — https://jinja.palletsprojects.com/en/stable/templates/#import-context-behavior

---

## 2. Macros

### Definitions

**Core Definition:** A macro is a reusable template fragment with parameters, defined using `{% macro name(args) %}...{% endmacro %}`, that can be called like a function within templates.

**Technical Definition:** Macros are comparable with functions in regular programming languages. They are useful to put often-used HTML idioms into reusable elements to avoid repetition. Macros can be defined in any template and must be imported before use (unless defined in the same template). Macros support arguments with default values, variable arguments (`varargs`), keyword arguments (`kwargs`), and access to the caller's context via the `caller()` function. Macros can be imported `with context` to access the importing template's context.

**Beginner-Friendly Explanation:** A macro is like a function for your templates. You define it once with parameters, then call it with different arguments wherever you need it. For example, a form input macro can generate the correct HTML for any field name and type.

### Purposes

- To create reusable parameterized template fragments.
- To generate consistent HTML structures (forms, cards, buttons).
- To reduce repetition of complex markup.
- To encapsulate rendering logic for repeated patterns.
- To build component libraries within templates.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Defining a macro #}
{% macro input(name, value='', type='text', size=20) %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}" size="{{ size }}">
{% endmacro %}

{# Calling a macro (defined in same template) #}
{{ input('username') }}
{{ input('password', type='password') }}

{# Importing a macro from another template #}
{% import 'forms.html' as forms %}
{{ forms.input('username') }}

{# Importing specific macros #}
{% from 'forms.html' import input, textarea %}
{{ input('username') }}

{# Import with context (to access current context) #}
{% from 'forms.html' import input with context %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% macro name(args) %}` | Defines a macro with parameters |
| `{% endmacro %}` | Closes the macro definition |
| `{{ macro_name(args) }}` | Calls the macro |
| `{% import 'file' as name %}` | Imports all macros from a file |
| `{% from 'file' import macro %}` | Imports specific macros |
| `with context` | Makes the importing template's context available |

**Syntax Rules:**

- Macro arguments can have default values.
- Macros and variables starting with underscores are private and cannot be imported.
- Macros imported without context do not have access to the importing template's context.
- The `caller()` function allows macros to use blocks passed to them.
- Macros can call other macros.

**Constraints and Limitations:**

- Macros cannot access the template context unless imported `with context`.
- Macros are not cached when imported `with context`.
- Macros cannot modify variables outside their scope.
- Macros cannot be used as decorators or have side effects.

### Annotated Code Examples

**Example 1: Form Input Macro**

```jinja
{# templates/macros/forms.html #}
{% macro input(name, value='', type='text', label='') %}
    <div class="form-group">
        {% if label %}<label for="{{ name }}">{{ label }}</label>{% endif %}
        <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}" id="{{ name }}">
    </div>
{% endmacro %}

{% macro textarea(name, value='', rows=5, cols=40) %}
    <textarea name="{{ name }}" rows="{{ rows }}" cols="{{ cols }}">{{ value|e }}</textarea>
{% endmacro %}
```

```jinja
{# templates/register.html #}
{% from 'macros/forms.html' import input, textarea %}

<form method="POST">
    {{ input('username', label='Username') }}
    {{ input('password', type='password', label='Password') }}
    {{ input('email', type='email', label='Email') }}
    {{ textarea('bio') }}
    <button type="submit">Register</button>
</form>
```

**Expected Output:**

```html
<form method="POST">
    <div class="form-group">
        <label for="username">Username</label>
        <input type="text" name="username" value="" id="username">
    </div>
    <div class="form-group">
        <label for="password">Password</label>
        <input type="password" name="password" value="" id="password">
    </div>
    <div class="form-group">
        <label for="email">Email</label>
        <input type="email" name="email" value="" id="email">
    </div>
    <textarea name="bio" rows="5" cols="40"></textarea>
    <button type="submit">Register</button>
</form>
```

**Why this output:** The `input` and `textarea` macros generate consistent form HTML. Each call provides different arguments to customize the field. The macros are imported from `macros/forms.html` and called like functions.

**Example 2: Macro with Caller**

```jinja
{# templates/macros/ui.html #}
{% macro card(title) %}
    <div class="card">
        <h3>{{ title }}</h3>
        <div class="card-body">{{ caller() }}</div>
    </div>
{% endmacro %}
```

```jinja
{# templates/page.html #}
{% from 'macros/ui.html' import card %}

{% call card('User Information') %}
    <p>Name: {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
{% endcall %}
```

**Expected Output:**

```html
<div class="card">
    <h3>User Information</h3>
    <div class="card-body">
        <p>Name: Alice</p>
        <p>Email: alice@example.com</p>
    </div>
</div>
```

**Why this output:** The `caller()` function inside the macro renders the content passed via the `{% call %}` block. This allows macros to wrap caller-provided content in a consistent structure.

### Real-World Cases

- **Form fields:** Generating consistent input, textarea, and select elements.
- **UI components:** Cards, buttons, alerts, and modals.
- **Table rows:** Generating rows with consistent formatting.
- **Navigation items:** Creating list items with active states.

### References

- Jinja2 Macros — https://jinja.palletsprojects.com/en/stable/templates/#macros
- Jinja2 Import — https://jinja.palletsprojects.com/en/stable/templates/#import
- Jinja2 Call — https://jinja.palletsprojects.com/en/stable/templates/#call

---

## 3. Reusable Components

### Definitions

**Core Definition:** Reusable components are template fragments (includes, macros, or combinations) designed to be used across multiple templates, providing consistent UI elements throughout an application.

**Technical Definition:** Reusable components in Jinja2 are implemented through includes and macros. Includes are suited for static or context-dependent fragments; macros are suited for parameterized fragments. Components can be organized into dedicated template files (e.g., `components/`, `partials/`, `macros/`) for discoverability. Blueprint templates can also provide components that extend application-level components. The `{% include %}` tag and macro imports work together to build component libraries.

**Beginner-Friendly Explanation:** Reusable components are the building blocks of your UI. A button, a form field, a card, or a navigation bar—each is a component you define once and use everywhere. Includes are for components that don't need parameters (or use context), and macros are for components that take parameters.

### Purposes

- To create a consistent UI across all pages.
- To organize reusable fragments into a component library.
- To reduce duplication of complex markup.
- To enable rapid development by composing components.
- To maintain consistency when the design changes.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# components/button.html — static include #}
<button class="btn {{ class|default('btn-primary') }}">{{ label }}</button>

{# Usage: #}
{% include 'components/button.html' %}

{# components/alert.html — macro #}
{% macro alert(message, type='info') %}
    <div class="alert alert-{{ type }}">{{ message }}</div>
{% endmacro %}

{# Usage: #}
{% from 'components/alert.html' import alert %}
{{ alert('Operation successful', 'success') }}
```

**Component Breakdown:**

| Component Type | Approach | Use Case |
|----------------|----------|----------|
| Static component | `{% include %}` | No parameters; context-based |
| Parameterized component | `{% macro %}` | Takes arguments |
| Context-aware component | `{% include %}` with context | Uses parent context |

**Syntax Rules:**

- Components should be organized in dedicated directories.
- Macro components should be imported at the top of the template.
- Include components should be placed where the output is needed.
- Components can be nested (a macro can include another component).
- Blueprint components can override application components via template search order.

**Constraints and Limitations:**

- Includes cannot accept parameters directly; use macros for parameterized components.
- Component names must be unique or conflicts occur in the template search path.
- Deeply nested components can impact rendering performance.

### Annotated Code Examples

**Example 1: Reusable Card Component**

```jinja
{# components/card.html #}
{% macro card(title, class='') %}
    <div class="card {{ class }}">
        <div class="card-header">{{ title }}</div>
        <div class="card-body">{{ caller() }}</div>
    </div>
{% endmacro %}
```

```jinja
{# templates/dashboard.html #}
{% from 'components/card.html' import card %}

<div class="dashboard">
    {% call card('Statistics') %}
        <p>Users: {{ stats.users }}</p>
        <p>Posts: {{ stats.posts }}</p>
    {% endcall %}
    
    {% call card('Recent Activity', class='card-highlight') %}
        <ul>
        {% for activity in recent_activities %}
            <li>{{ activity }}</li>
        {% endfor %}
        </ul>
    {% endcall %}
</div>
```

**Expected Output:**

```html
<div class="dashboard">
    <div class="card ">
        <div class="card-header">Statistics</div>
        <div class="card-body">
            <p>Users: 150</p>
            <p>Posts: 42</p>
        </div>
    </div>
    <div class="card card-highlight">
        <div class="card-header">Recent Activity</div>
        <div class="card-body">
            <ul>
                <li>User registered</li>
                <li>Post published</li>
            </ul>
        </div>
    </div>
</div>
```

**Why this output:** The `card` macro creates a consistent card structure with a title and body. The `caller()` function renders the content passed via `{% call %}`. The `class` parameter allows customization.

### Real-World Cases

- **Design systems:** A library of buttons, cards, alerts, and modals.
- **Dashboards:** Reusable widget components.
- **E-commerce:** Product cards, price displays, and rating components.
- **Admin panels:** Data tables, filter bars, and action buttons.

### References

- Jinja2 Macros — https://jinja.palletsprojects.com/en/stable/templates/#macros
- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include
- Flask Blueprint Templates — https://flask.palletsprojects.com/en/stable/blueprints/#templates

---

## 4. Custom Filters

### Definitions

**Core Definition:** Custom filters are Python functions registered with Jinja2 that transform values in template expressions using the pipe (`|`) syntax.

**Technical Definition:** Custom filters are registered using the `@app.template_filter()` decorator or by adding functions to `app.jinja_env.filters`. A filter function accepts the value to transform as its first argument and optionally additional arguments. The filter name defaults to the function name but can be overridden. Filters are looked up in a separate namespace from tests and can contain dots for grouping. Flask also provides `add_template_filter()` for imperative registration.

**Beginner-Friendly Explanation:** A custom filter is a function you write that transforms a value in a template. If you need to format dates in a specific way, or abbreviate long text, or convert markdown to HTML, you write a filter and use it like `{{ value|my_filter }}`.

### Purposes

- To add domain-specific transformations to template expressions.
- To format dates, numbers, and strings consistently.
- To convert data formats (markdown, JSON, CSV).
- To apply business logic to displayed values.
- To extend Jinja's built-in filter set.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using decorator
@app.template_filter('name')
def my_filter(value, arg1, arg2):
    # Transform value
    return transformed

# Without name (uses function name)
@app.template_filter()
def my_filter(value):
    return transformed

# Imperative registration
def my_filter(value):
    return transformed
app.add_template_filter(my_filter, 'name')
```

```jinja
{# Usage in template #}
{{ value|my_filter }}
{{ value|my_filter(arg1, arg2) }}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@app.template_filter('name')` | Registers a filter with the given name |
| `value` | The value to transform (first argument) |
| Additional args | Optional arguments passed from the template |
| Return value | The transformed value |

**Syntax Rules:**

- The filter function's first argument is the value to transform.
- Additional arguments are passed from the template.
- The filter name defaults to the function name if not specified.
- Filters can be chained: `{{ value|filter1|filter2 }}`.
- Filters can contain dots for grouping (e.g., `myapp.format`).

**Constraints and Limitations:**

- Filters cannot access the template context unless registered with `@contextfilter`.
- Filter names must be unique or they override existing filters.
- Filters should be pure functions; avoid side effects.

### Annotated Code Examples

**Example 1: Date Formatting Filter**

```python
from flask import Flask, render_template
from datetime import datetime

app = Flask(__name__)

@app.template_filter('format_date')
def format_date(value, format='%Y-%m-%d'):
    """Format a datetime object."""
    if isinstance(value, datetime):
        return value.strftime(format)
    return value

@app.route("/")
def index():
    return render_template("index.html", now=datetime(2024, 1, 15, 10, 30))

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/index.html #}
<p>Date: {{ now|format_date }}</p>
<p>Custom: {{ now|format_date('%B %d, %Y') }}</p>
```

**Expected Output:**

```html
<p>Date: 2024-01-15</p>
<p>Custom: January 15, 2024</p>
```

**Why this output:** The `format_date` filter takes the `now` datetime object and formats it. The default format is `%Y-%m-%d`, but the template can override it with an argument.

**Example 2: Markdown to HTML Filter**

```python
import markdown

@app.template_filter('markdown')
def markdown_filter(value):
    """Convert markdown text to HTML."""
    return markdown.markdown(value)

@app.route("/post")
def post():
    content = "# Hello\n\nThis is **bold**."
    return render_template("post.html", content=content)
```

```jinja
{# templates/post.html #}
<div class="content">{{ content|markdown }}</div>
```

**Expected Output:**

```html
<div class="content"><h1>Hello</h1>
<p>This is <strong>bold</strong>.</p>
</div>
```

**Why this output:** The `markdown` filter converts the markdown string to HTML. The resulting HTML is marked safe by the filter, so it is rendered as HTML rather than escaped.

### Real-World Cases

- **Date formatting:** Displaying dates in user-friendly formats.
- **Text truncation:** Shortening long descriptions with ellipsis.
- **Markdown rendering:** Converting markdown content to HTML.
- **Number formatting:** Adding thousands separators or currency symbols.
- **JSON formatting:** Pretty-printing JSON for display.

### References

- Flask `template_filter` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_filter
- Jinja2 Custom Filters — https://jinja.palletsprojects.com/en/stable/api/#custom-filters
- Jinja2 Filters — https://jinja.palletsprojects.com/en/stable/templates/#filters

---

## 5. Custom Tests and Global Functions

### Definitions

**Core Definition:** Custom tests are Python functions registered with Jinja2 that evaluate a condition and return a boolean, used with the `is` operator. Custom globals are Python functions or values registered with Jinja2 that are available in all templates without explicit passing.

**Technical Definition:** Custom tests are registered using the `@app.template_test()` decorator or by adding functions to `app.jinja_env.tests`. A test function accepts the value to test as its first argument and returns a boolean. Custom globals are registered using the `@app.template_global()` decorator or by adding to `app.jinja_env.globals`. Globals are available in all templates and can be called like functions. Both tests and globals are looked up in separate namespaces and can contain dots for grouping.

**Beginner-Friendly Explanation:** A custom test is a condition you define, like `{% if value is my_test %}`. A custom global is a function you can use anywhere in any template, like `{{ my_function() }}`. They extend what Jinja can do without passing extra data to every template.

### Purposes

- **Tests:** To add domain-specific conditions (e.g., `is valid_email`, `is prime`).
- **Tests:** To check application-specific types or states.
- **Globals:** To provide utility functions available everywhere.
- **Globals:** To expose constants, helpers, or formatting functions.
- **Globals:** To reduce the need for context processors.

### Syntax Rules and Structure

**Complete General Syntax:**

**Custom Tests:**

```python
@app.template_test('name')
def my_test(value):
    return True or False

# Imperative
app.jinja_env.tests['name'] = my_test
```

```jinja
{% if value is my_test %}...{% endif %}
{% if value is my_test(arg) %}...{% endif %}
```

**Custom Globals:**

```python
@app.template_global('name')
def my_function(arg):
    return result

# Imperative
app.jinja_env.globals['name'] = my_function
```

```jinja
{{ my_function('arg') }}
{% set result = my_function('arg') %}
```

**Component Breakdown:**

| Feature | Decorator | Registration |
|---------|-----------|--------------|
| Test | `@app.template_test()` | `app.jinja_env.tests['name']` |
| Global | `@app.template_global()` | `app.jinja_env.globals['name']` |

**Syntax Rules:**

- Test functions return `True` or `False`.
- Globals can return any value or callable.
- Both can contain dots in their names for grouping.
- Globals are available in all templates automatically.
- Tests are used with the `is` operator.

**Constraints and Limitations:**

- Tests cannot access the template context.
- Globals should be pure functions or constants.
- Overriding existing tests or globals may break templates.

### Annotated Code Examples

**Example 1: Custom Test**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.template_test('prime')
def is_prime(n):
    """Check if a number is prime."""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

@app.route("/")
def index():
    return render_template("index.html", numbers=[2, 3, 4, 5, 6, 7])

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/index.html #}
<ul>
{% for n in numbers %}
    <li>{{ n }} {% if n is prime %}(prime){% else %}(not prime){% endif %}</li>
{% endfor %}
</ul>
```

**Expected Output:**

```html
<ul>
    <li>2 (prime)</li>
    <li>3 (prime)</li>
    <li>4 (not prime)</li>
    <li>5 (prime)</li>
    <li>6 (not prime)</li>
    <li>7 (prime)</li>
</ul>
```

**Why this output:** The `is prime` test checks each number using the `is_prime` function. Numbers that are prime display "(prime)"; others display "(not prime)".

**Example 2: Custom Global Function**

```python
@app.template_global('format_currency')
def format_currency(amount, currency='USD'):
    """Format a number as currency."""
    symbols = {'USD': '$', 'EUR': '€', 'GBP': '£'}
    symbol = symbols.get(currency, currency)
    return f"{symbol}{amount:,.2f}"

@app.route("/product")
def product():
    return render_template("product.html", price=1234.5)

if __name__ == "__main__":
    app.run(debug=True)
```

```jinja
{# templates/product.html #}
<p>Price: {{ format_currency(price) }}</p>
<p>EUR Price: {{ format_currency(price, 'EUR') }}</p>
```

**Expected Output:**

```html
<p>Price: $1,234.50</p>
<p>EUR Price: €1,234.50</p>
```

**Why this output:** The `format_currency` global is available in all templates without explicit passing. It formats the price with the appropriate currency symbol and thousands separators.

### Real-World Cases

- **Tests:** `is valid_email`, `is admin`, `is recent` (date check).
- **Globals:** `format_currency`, `current_year`, `site_name`, `url_for` (built-in).
- **Tests:** `is even`, `is divisibleby` (built-in, can be extended).
- **Globals:** `get_flashed_messages` (built-in), `config` (built-in).

### References

- Flask `template_test` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_test
- Flask `template_global` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_global
- Jinja2 Custom Tests — https://jinja.palletsprojects.com/en/stable/api/#custom-tests
- Jinja2 The Global Namespace — https://jinja.palletsprojects.com/en/stable/api/#the-global-namespace

---

## 6. The `with context` and `without context` Modifiers

### Definitions

**Core Definition:** The `with context` and `without context` modifiers control whether the current template's context variables are passed to included or imported templates.

**Technical Definition:** By default, included templates are passed the current context and imported templates are not. The reason for this is that imports, unlike includes, are cached; as imports are often used just as a module that holds macros, caching them improves performance. This behavior can be changed explicitly by adding `with context` or `without context` to the import/include directive. When `with context` is used, the current context is passed to the template, and caching is disabled automatically. When `without context` is used, the current context is not passed, and Jinja can optimize by caching the rendered output.

**Beginner-Friendly Explanation:** By default, includes get all the variables from the parent template, but imports don't. If you want an import to see the parent's variables, add `with context`. If you want an include to be faster by not sharing variables, add `without context`.

### Purposes

- To control variable visibility in included and imported templates.
- To optimize performance by enabling caching with `without context`.
- To give macros access to the current template's context with `with context`.
- To prevent variable name collisions in includes.
- To balance convenience (context access) against performance (caching).

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Include with default behavior (passes context) #}
{% include 'header.html' %}

{# Include without context (enables caching) #}
{% include 'header.html' without context %}

{# Include with context (explicit) #}
{% include 'header.html' with context %}

{# Import without context (default, cached) #}
{% from 'macros.html' import input %}

{# Import with context (accesses current context, not cached) #}
{% from 'macros.html' import input with context %}

{# Import whole template with context #}
{% import 'macros.html' as forms with context %}
```

**Component Breakdown:**

| Modifier | Include Behavior | Import Behavior |
|----------|------------------|-----------------|
| (default) | Passes context | Does not pass context |
| `with context` | Passes context (explicit) | Passes context; disables caching |
| `without context` | Does not pass context; enables caching | Does not pass context; enables caching |

**Syntax Rules:**

- The modifier must be placed after the template path and before any other options (e.g., `ignore missing`).
- `with context` disables caching automatically.
- `without context` enables caching of the included/imported template.
- Default behavior differs between include (context passed) and import (context not passed).

**Constraints and Limitations:**

- Using `with context` on an import disables caching, which can impact performance.
- `without context` means the template cannot access variables from the parent context.
- Global variables (e.g., `config`, `request`) are always available regardless of context modifiers.

### Annotated Code Examples

**Example 1: Include with and without Context**

```jinja
{# templates/page.html #}
{% set username = 'Alice' %}

{# Default: context passed #}
{% include 'partials/greeting.html' %}
{# Output: Hello, Alice! #}

{# Without context: context not passed #}
{% include 'partials/greeting.html' without context %}
{# Output: Hello, Guest! #}
```

```jinja
{# templates/partials/greeting.html #}
<p>Hello, {{ username|default('Guest') }}!</p>
```

**Expected Output:**

```html
<p>Hello, Alice!</p>
<p>Hello, Guest!</p>
```

**Why this output:** The first include passes the `username` variable from the parent context. The second include (`without context`) does not receive `username`, so the `default` filter outputs "Guest".

**Example 2: Import with Context for Macro Access**

```jinja
{# templates/macros/user.html #}
{% macro user_badge() %}
    <span class="badge">{{ current_user.name }}</span>
{% endmacro %}
```

```jinja
{# templates/page.html #}
{% from 'macros/user.html' import user_badge with context %}

<div class="header">
    {{ user_badge() }}
</div>
```

**Expected Output:**

```html
<div class="header">
    <span class="badge">Alice</span>
</div>
```

**Why this output:** Without `with context`, the macro would not have access to `current_user`. With `with context`, the macro can access the importing template's context, including `current_user`.

### Real-World Cases

- **Static includes:** Headers and footers that don't depend on page-specific variables → `without context` for caching.
- **Dynamic includes:** Sidebars that show user-specific content → default (context passed).
- **Macro libraries:** Macros that need `current_user`, `config`, or `request` → `with context`.
- **Performance-critical pages:** Using `without context` for static fragments to enable caching.

### References

- Jinja2 Import Context Behavior — https://jinja.palletsprojects.com/en/stable/templates/#import-context-behavior
- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include
- Jinja2 Import — https://jinja.palletsprojects.com/en/stable/templates/#import

---

## References

- Jinja2 Template Designer Documentation — https://jinja.palletsprojects.com/en/stable/templates/
- Jinja2 Import Context Behavior — https://jinja.palletsprojects.com/en/stable/templates/#import-context-behavior
- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include
- Jinja2 Import — https://jinja.palletsprojects.com/en/stable/templates/#import
- Jinja2 Macros — https://jinja.palletsprojects.com/en/stable/templates/#macros
- Jinja2 Call — https://jinja.palletsprojects.com/en/stable/templates/#call
- Jinja2 Custom Filters — https://jinja.palletsprojects.com/en/stable/api/#custom-filters
- Jinja2 Custom Tests — https://jinja.palletsprojects.com/en/stable/api/#custom-tests
- Jinja2 The Global Namespace — https://jinja.palletsprojects.com/en/stable/api/#the-global-namespace
- Jinja2 Filters — https://jinja.palletsprojects.com/en/stable/templates/#filters
- Flask `template_filter` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_filter
- Flask `template_test` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_test
- Flask `template_global` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.template_global
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/
- Flask Blueprint Templates — https://flask.palletsprojects.com/en/stable/blueprints/#templates