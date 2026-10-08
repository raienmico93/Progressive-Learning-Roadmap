# Jinja Syntax: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Jinja is a modern, designer-friendly templating language for Python that is both web-framework agnostic and language agnostic, used to generate any text-based format (HTML, XML, CSV, LaTeX, etc.) by combining static content with dynamic placeholders.

**Technical Definition:** Jinja2 is a template engine that compiles templates into Python code. A template contains **variables** and/or **expressions**, which get replaced with values when the template is rendered, and **tags**, which control the logic of the template. The syntax is heavily inspired by Django and Python. Jinja uses three primary delimiters: `{% ... %}` for statements, `{{ ... }}` for expressions, and `{# ... #}` for comments. Flask integrates Jinja2 as its default template engine, with autoescaping enabled for HTML templates.

**Beginner-Friendly Explanation:** Jinja is like a fill-in-the-blanks system for HTML. You write a normal HTML page but add special placeholders like `{{ name }}` where dynamic data should go. When Flask renders the template, it replaces those placeholders with actual values. You can also use logic like `{% if user %}` to show or hide parts of the page, and `{% for item in items %}` to repeat content.

### Key Characteristics

- **Three delimiter types:** `{{ }}` for expressions, `{% %}` for statements, `{# #}` for comments.
- **Autoescaping:** HTML templates are autoescaped by default, preventing XSS attacks.
- **Filters:** Variables can be modified using filters with the pipe symbol (`|`).
- **Tests:** Variables can be checked against conditions using the `is` operator.
- **Template inheritance:** Child templates extend base templates for consistent layouts.
- **Whitespace control:** Minus signs (`-`) control whitespace around tags.
- **Raw blocks:** `{% raw %}` prevents Jinja from processing content, useful for frontend frameworks.
- **Line statements:** Optional line-based syntax for cleaner templates.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTML and Python dictionaries.
- Familiarity with Flask routing and `render_template()`.
- Knowledge of Python data types (lists, dictionaries, strings).

### Related Programming Areas

- **Flask Templating:** Jinja is Flask's default template engine.
- **Frontend frameworks:** Vue, Angular, and React use similar `{{ }}` syntax, necessitating raw blocks.
- **Configuration management:** Ansible uses Jinja2 for templating.
- **Email generation:** Jinja is used for personalized email templates.
- **Static site generation:** Tools like Pelican use Jinja for HTML generation.

### Core Concepts / Features

1. Expressions
2. Statements
3. Variables
4. Filters
5. Tests
6. Comments
7. Line Statements (`#`) and Whitespace Control (`{%-` and `-%}`)
8. Literal Escaping (`{% raw %}`)

---

## 1. Expressions

### Definitions

**Core Definition:** Jinja expressions are constructs enclosed in double curly braces (`{{ ... }}`) that are evaluated and their results are printed to the template output.

**Technical Definition:** Expressions in Jinja are everything between `{{` and `}}`. They can be literals (strings, numbers, lists, dictionaries), variables, function calls, and operators. Jinja supports basic Python-like operators including arithmetic (`+`, `-`, `*`, `/`, `//`, `%`, `**`), comparison (`==`, `!=`, `<`, `>`, `<=`, `>=`), logic (`and`, `or`, `not`), and others. The `if` expression is available inline: `<do something> if <condition> else <do something else>`. Expressions are evaluated in the template's context and their string representation is inserted into the output.

**Beginner-Friendly Explanation:** An expression is anything inside `{{ }}`. It could be a simple variable like `{{ name }}`, a calculation like `{{ price * quantity }}`, or a conditional like `{{ "yes" if user else "no" }}`. Jinja calculates the result and puts it in the output.

### Purposes

- To output dynamic values in templates.
- To perform calculations and string operations inline.
- To access attributes and items of objects.
- To use inline conditionals for simple decisions.
- To call functions and filters on values.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{{ expression }}
{{ variable }}
{{ variable.attribute }}
{{ variable['key'] }}
{{ a + b }}
{{ a if condition else b }}
{{ function(arg) }}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{{ ... }}` | Delimits an expression to be output |
| `variable` | A name from the template context |
| `.attribute` | Access an attribute of an object |
| `['key']` | Access an item by subscript |
| `+`, `-`, `*`, etc. | Operators for calculation |
| `if ... else ...` | Inline conditional expression |

**Syntax Rules:**

- Expressions are evaluated and their results are converted to strings.
- Dot notation (`foo.bar`) and subscript notation (`foo['bar']`) are equivalent.
- Inline conditionals use the format `<value> if <condition> else <other_value>`.
- Function calls use parentheses: `{{ url_for('index') }}`.
- Filters are applied with the pipe symbol: `{{ name|upper }}`.

**Constraints and Limitations:**

- Jinja expressions are not Python; some Python constructs (like list comprehensions) are not available.
- Complex logic should be moved to Python code, not templates.
- Undefined variables render as empty strings by default (unless `StrictUndefined` is enabled).

### Annotated Code Examples

**Example 1: Basic Expressions**

```jinja
{# templates/example.html #}
<p>Name: {{ name }}</p>
<p>Age: {{ age }}</p>
<p>Next year: {{ age + 1 }}</p>
<p>Status: {{ "Adult" if age >= 18 else "Minor" }}</p>
<p>Initial: {{ name[0]|upper }}</p>
```

**Expected Output (when `name="Alice"` and `age=30`):**

```html
<p>Name: Alice</p>
<p>Age: 30</p>
<p>Next year: 31</p>
<p>Status: Adult</p>
<p>Initial: A</p>
```

**Why this output:** Each `{{ ... }}` is evaluated. `{{ name }}` outputs the value. `{{ age + 1 }}` performs arithmetic. The inline conditional outputs "Adult" because the condition is true. `{{ name[0]|upper }}` accesses the first character and applies the `upper` filter.

### Real-World Cases

- **User profiles:** `{{ user.name }}` displays the username.
- **E-commerce:** `{{ product.price * quantity }}` calculates line totals.
- **Dashboards:** `{{ "Active" if user.is_active else "Inactive" }}` shows status.
- **Navigation:** `{{ request.path }}` highlights the current page.

### References

- Jinja2 Expressions — https://jinja.palletsprojects.com/en/stable/templates/#expressions
- Jinja2 Variables — https://jinja.palletsprojects.com/en/stable/templates/#variables

---

## 2. Statements

### Definitions

**Core Definition:** Jinja statements are control-flow constructs enclosed in `{% ... %}` that control the logic of the template, such as conditionals, loops, and template inheritance.

**Technical Definition:** Statements in Jinja use the `{% ... %}` delimiter. They include control structures: `{% if %}...{% elif %}...{% else %}...{% endif %}`, `{% for %}...{% endfor %}`, `{% block %}...{% endblock %}`, `{% extends %}`, `{% include %}`, `{% macro %}`, `{% set %}`, and others. Statements do not output text directly; they control how the template is rendered. The default Jinja delimiters are `{% ... %}` for statements, `{{ ... }}` for expressions, and `{# ... #}` for comments.

**Beginner-Friendly Explanation:** Statements are the logic of your template. `{% if user %}` shows content only if a condition is true. `{% for item in items %}` repeats content for each item. `{% extends "base.html" %}` inherits from another template. They use `{% %}` instead of `{{ }}` because they don't output values—they control what gets rendered.

### Purposes

- To implement conditional rendering with `{% if %}`.
- To iterate over data with `{% for %}`.
- To inherit from base templates with `{% extends %}`.
- To define reusable blocks with `{% block %}`.
- To include other templates with `{% include %}`.
- To define reusable macros with `{% macro %}`.
- To assign variables within templates with `{% set %}`.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Conditionals #}
{% if condition %}
    <p>True</p>
{% elif other_condition %}
    <p>Other</p>
{% else %}
    <p>False</p>
{% endif %}

{# Loops #}
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}

{# Template inheritance #}
{% extends "base.html" %}
{% block content %}...{% endblock %}

{# Includes #}
{% include "header.html" %}

{# Macros #}
{% macro input(name, value='') %}
    <input name="{{ name }}" value="{{ value }}">
{% endmacro %}

{# Set #}
{% set x = 10 %}
```

**Component Breakdown:**

| Statement | Purpose |
|-----------|---------|
| `{% if %}` | Conditional rendering |
| `{% for %}` | Iteration |
| `{% extends %}` | Template inheritance |
| `{% block %}` | Named override regions |
| `{% include %}` | Include another template |
| `{% macro %}` | Reusable template functions |
| `{% set %}` | Assign a variable |

**Syntax Rules:**

- Statements must be properly closed with `{% endif %}`, `{% endfor %}`, etc.
- `{% extends %}` must be the first tag in a child template.
- `{% block %}` names must be unique within a template.
- `{% include %}` can include templates with context.

**Constraints and Limitations:**

- Statements cannot output text directly; use expressions for output.
- Jinja does not support arbitrary Python code in statements.
- Deeply nested statements can make templates hard to read.

### Annotated Code Examples

**Example 1: Conditional and Loop Statements**

```jinja
{# templates/users.html #}
{% if users %}
    <ul>
    {% for user in users %}
        <li>{{ user.name }} - {{ "Admin" if user.admin else "User" }}</li>
    {% endfor %}
    </ul>
{% else %}
    <p>No users found.</p>
{% endif %}
```

**Expected Output (with two users, one admin):**

```html
<ul>
    <li>Alice - Admin</li>
    <li>Bob - User</li>
</ul>
```

**Why this output:** The `{% if users %}` checks if the list is non-empty. The `{% for %}` loop iterates over each user. The inline conditional displays the role.

**Example 2: Template Inheritance**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head><title>{% block title %}My Site{% endblock %}</title></head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

```jinja
{# templates/home.html #}
{% extends "base.html" %}
{% block title %}Home{% endblock %}
{% block content %}
    <h1>Welcome!</h1>
{% endblock %}
```

**Expected Output:**

```html
<!DOCTYPE html>
<html>
<head><title>Home</title></head>
<body>
    <h1>Welcome!</h1>
</body>
</html>
```

**Why this output:** `home.html` extends `base.html` and overrides the `title` and `content` blocks.

### Real-World Cases

- **Navigation menus:** Looping over menu items with `{% for %}`.
- **User lists:** Displaying users conditionally based on role.
- **Layouts:** Using `{% extends %}` for consistent site structure.
- **Reusable components:** Defining form inputs with `{% macro %}`.

### References

- Jinja2 Control Structures — https://jinja.palletsprojects.com/en/stable/templates/#list-of-control-structures
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance

---

## 3. Variables

### Definitions

**Core Definition:** Template variables are named values passed from Python code to the template context, accessible within the template using `{{ variable_name }}`.

**Technical Definition:** Template variables are defined by the context dictionary passed to the template. Variables may have attributes or elements accessed via dot notation (`foo.bar`) or subscript notation (`foo['bar']`). Jinja also provides a standard context that includes `config`, `request`, `session`, `g`, `url_for()`, and `get_flashed_messages()` when used with Flask. Undefined variables render as empty strings by default, but this behavior can be changed with `StrictUndefined`.

**Beginner-Friendly Explanation:** Variables are the blanks you fill in. You pass them from your Python view function to the template, and the template displays them using `{{ }}`. For example, if you pass `name="Alice"`, the template can display `{{ name }}` to show "Alice".

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
{{ user['name'] }}
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
- Dot notation and subscript notation are equivalent for attribute access.
- Undefined variables render as empty strings by default.
- Use `{{ variable|default('fallback') }}` for optional variables.

**Constraints and Limitations:**

- Templates cannot modify variables passed from Python.
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
```

```jinja
{# templates/profile.html #}
<h1>{{ user.name }}</h1>
<p>Age: {{ user.age }}</p>
<ul>
{% for hobby in user.hobbies %}
    <li>{{ hobby }}</li>
{% endfor %}
</ul>
```

**Expected Output (for `/profile/alice`):**

```html
<h1>alice</h1>
<p>Age: 30</p>
<ul>
    <li>reading</li>
    <li>coding</li>
</ul>
```

**Why this output:** The `user` dictionary is passed to the template. `{{ user.name }}` accesses the `name` key, and the `{% for %}` loop iterates over the `hobbies` list.

**Example 2: Using Standard Context**

```jinja
<p>Current path: {{ request.path }}</p>
<p>Debug mode: {{ config.DEBUG }}</p>
<p>Logged in: {{ session.get('username', 'Guest') }}</p>
```

**Expected Output:**

```html
<p>Current path: /dashboard</p>
<p>Debug mode: True</p>
<p>Logged in: Guest</p>
```

**Why this output:** Flask automatically injects `request`, `config`, and `session` into the template context.

### Real-World Cases

- **User profiles:** Displaying user names, emails, and preferences.
- **Product pages:** Showing product details, prices, and images.
- **Dashboards:** Rendering metrics and charts with dynamic data.
- **Navigation:** Using `request.path` to highlight the active menu item.

### References

- Jinja2 Variables — https://jinja.palletsprojects.com/en/stable/templates/#variables
- Flask Templating: Standard Context — https://flask.palletsprojects.com/en/stable/templating/#standard-context

---

## 4. Filters

### Definitions

**Core Definition:** Filters are functions that modify variables in Jinja templates, applied using the pipe symbol (`|`), and can be chained to transform data.

**Technical Definition:** Filters are separated from the variable by a pipe symbol (`|`) and may have optional arguments in parentheses. Multiple filters can be chained, with the output of one filter applied to the next. Filters that accept arguments have parentheses around the arguments, like a function call. Jinja includes many built-in filters such as `upper`, `lower`, `title`, `join`, `length`, `default`, `replace`, `round`, and `truncate`.

**Beginner-Friendly Explanation:** Filters are like little machines that transform values. `{{ name|upper }}` makes the name uppercase. `{{ items|join(', ') }}` joins a list with commas. You can chain them: `{{ name|striptags|title }}` removes HTML tags and then title-cases the result.

### Purposes

- To transform text (uppercase, lowercase, title case).
- To format numbers (rounding, absolute value).
- To join lists into strings.
- To provide default values for undefined variables.
- To escape or strip HTML tags.
- To truncate long strings.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{{ variable|filter }}
{{ variable|filter(arg1, arg2) }}
{{ variable|filter1|filter2 }}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `variable` | The value to transform |
| `|` | Pipe symbol, applies the filter |
| `filter` | The filter name |
| `(args)` | Optional filter arguments |

**Common Built-in Filters:**

| Filter | Description | Example |
|--------|-------------|---------|
| `upper` | Uppercase | `{{ name\|upper }}` |
| `lower` | Lowercase | `{{ name\|lower }}` |
| `title` | Title case | `{{ name\|title }}` |
| `join` | Join list | `{{ items\|join(', ') }}` |
| `length` | Length of list/string | `{{ items\|length }}` |
| `default` | Default value | `{{ name\|default('Guest') }}` |
| `replace` | Replace substring | `{{ text\|replace('a', 'b') }}` |
| `round` | Round number | `{{ price\|round(2) }}` |
| `truncate` | Truncate string | `{{ text\|truncate(20) }}` |
| `striptags` | Remove HTML tags | `{{ html\|striptags }}` |

**Syntax Rules:**

- Filters are applied with the pipe symbol (`|`).
- Multiple filters can be chained: `{{ var|filter1|filter2 }}`.
- Filters with arguments use parentheses: `{{ var|filter(arg) }}`.
- The `default` filter handles undefined variables.

**Constraints and Limitations:**

- Filters do not modify the original variable; they return a new value.
- Some filters may raise errors on incompatible types.
- Custom filters can be registered on the Jinja environment.

### Annotated Code Examples

**Example 1: Basic Filter Usage**

```jinja
{# templates/filters.html #}
<p>Name: {{ name|upper }}</p>
<p>Items: {{ items|join(', ') }}</p>
<p>Count: {{ items|length }}</p>
<p>Price: ${{ price|round(2) }}</p>
<p>Default: {{ missing|default('Not provided') }}</p>
```

**Expected Output (when `name="alice"`, `items=['a','b','c']`, `price=19.999`):**

```html
<p>Name: ALICE</p>
<p>Items: a, b, c</p>
<p>Count: 3</p>
<p>Price: $20.0</p>
<p>Default: Not provided</p>
```

**Why this output:** Each filter transforms the variable. `upper` makes the name uppercase. `join` combines the list. `length` returns the count. `round(2)` rounds to two decimal places. `default` provides a fallback for the undefined `missing` variable.

**Example 2: Chaining Filters**

```jinja
{{ "<p>Hello <b>World</b></p>"|striptags|title }}
```

**Expected Output:**

```
Hello World
```

**Why this output:** `striptags` removes the HTML tags, and `title` capitalizes the first letter of each word.

### Real-World Cases

- **User input sanitization:** `{{ user_input|striptags }}` removes HTML.
- **Formatting prices:** `{{ product.price|round(2) }}` rounds to cents.
- **Displaying lists:** `{{ tags|join(', ') }}` joins tags.
- **Handling missing data:** `{{ user.bio|default('No bio yet') }}` provides a fallback.

### References

- Jinja2 Filters — https://jinja.palletsprojects.com/en/stable/templates/#filters
- Jinja2 Built-in Filters — https://jinja.palletsprojects.com/en/stable/templates/#builtin-filters

---

## 5. Tests

### Definitions

**Core Definition:** Tests are functions that evaluate a variable against a condition and return a boolean value, used with the `is` operator in Jinja templates.

**Technical Definition:** Tests can be used to test a variable against a common expression. To test a variable or expression, you add `is` plus the name of the test after the variable. Tests can accept arguments; if the test only takes one argument, you can leave out the parentheses. Built-in tests include `defined`, `undefined`, `none`, `string`, `number`, `iterable`, `mapping`, `sameas`, `even`, `odd`, `divisibleby`, `lower`, and `upper`.

**Beginner-Friendly Explanation:** Tests check if something is true. `{% if user is defined %}` checks if the `user` variable exists. `{% if number is even %}` checks if a number is even. They use the `is` keyword instead of `==` for special checks.

### Purposes

- To check if a variable is defined or undefined.
- To check the type of a variable (string, number, iterable, mapping).
- To check mathematical properties (even, odd, divisible by).
- To compare object identity with `sameas`.
- To check case (lower, upper).

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% if variable is test %}
    ...
{% endif %}

{% if variable is test(arg) %}
    ...
{% endif %}
```

**Common Built-in Tests:**

| Test | Description | Example |
|------|-------------|---------|
| `defined` | Variable is defined | `{% if user is defined %}` |
| `undefined` | Variable is undefined | `{% if user is undefined %}` |
| `none` | Variable is None | `{% if value is none %}` |
| `string` | Variable is a string | `{% if name is string %}` |
| `number` | Variable is a number | `{% if age is number %}` |
| `iterable` | Variable is iterable | `{% if items is iterable %}` |
| `mapping` | Variable is a dictionary | `{% if data is mapping %}` |
| `sameas` | Same object as | `{% if a is sameas(b) %}` |
| `even` | Even number | `{% if n is even %}` |
| `odd` | Odd number | `{% if n is odd %}` |
| `divisibleby` | Divisible by | `{% if n is divisibleby(3) %}` |
| `lower` | All lowercase | `{% if text is lower %}` |
| `upper` | All uppercase | `{% if text is upper %}` |

**Syntax Rules:**

- Tests use the `is` keyword: `{% if variable is test %}`.
- Tests with arguments use parentheses: `{% if n is divisibleby(3) %}`.
- For single-argument tests, parentheses are optional: `{% if n is divisibleby 3 %}`.
- Tests return boolean values used in conditionals.

**Constraints and Limitations:**

- Tests are not filters; they cannot be used with `|`.
- Custom tests can be registered on the Jinja environment.
- Some tests may behave differently depending on the Python version.

### Annotated Code Examples

**Example 1: Checking Variable Definition**

```jinja
{% if user is defined %}
    <p>Welcome, {{ user.name }}!</p>
{% else %}
    <p>Please log in.</p>
{% endif %}
```

**Expected Output (when `user` is defined):**

```html
<p>Welcome, Alice!</p>
```

**Expected Output (when `user` is not defined):**

```html
<p>Please log in.</p>
```

**Why this output:** The `defined` test checks if the `user` variable exists in the context. If it does, the first block is rendered; otherwise, the else block is rendered.

**Example 2: Checking Number Properties**

```jinja
{% for n in range(1, 6) %}
    {% if n is even %}
        <p>{{ n }} is even</p>
    {% else %}
        <p>{{ n }} is odd</p>
    {% endif %}
{% endfor %}
```

**Expected Output:**

```html
<p>1 is odd</p>
<p>2 is even</p>
<p>3 is odd</p>
<p>4 is even</p>
<p>5 is odd</p>
```

**Why this output:** The `even` test checks if each number is even. The loop iterates from 1 to 5, applying the test to each number.

### Real-World Cases

- **Optional data:** `{% if user.bio is defined %}` checks if a bio exists.
- **Type checking:** `{% if value is number %}` formats numbers differently.
- **Permissions:** `{% if user.role is sameas('admin') %}` checks role.
- **Empty checks:** `{% if items is iterable %}` ensures the variable can be looped.

### References

- Jinja2 Tests — https://jinja.palletsprojects.com/en/stable/templates/#tests
- Jinja2 Built-in Tests — https://jinja.palletsprojects.com/en/stable/templates/#builtin-tests

---

## 6. Comments

### Definitions

**Core Definition:** Comments are sections of a template that are ignored by Jinja and not included in the rendered output, used for documentation or debugging.

**Technical Definition:** Comments use the `{# ... #}` syntax. Anything between the opening and closing comment delimiters is ignored by Jinja and not included in the output. Comments cannot be nested. Line-based comments are also available if `line_comment_prefix` is configured on the Jinja environment; everything from the prefix to the end of the line is ignored.

**Beginner-Friendly Explanation:** Comments are notes you write in your template that don't appear in the final page. They're useful for explaining what a section does or temporarily disabling code. Use `{# ... #}` for block comments or `##` for line comments (if enabled).

### Purposes

- To document template logic for other developers.
- To temporarily disable sections of a template during debugging.
- To add notes about why certain decisions were made.
- To prevent Jinja from processing code that should be treated as text.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# This is a comment #}
{# Multi-line
   comment #}

{# Line comment example #}
```

**Line-based comment (if enabled):**

```jinja
## This is a line comment
{% for item in seq %}
    <li>{{ item }}</li>  ## this comment is ignored
{% endfor %}
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `{# ... #}` | Block comment; content is ignored |
| `## ...` | Line comment (requires configuration) |

**Syntax Rules:**

- Comments start with `{#` and end with `#}`.
- Comments cannot be nested.
- Line comments require `line_comment_prefix` to be configured.
- HTML comments (`<!-- -->`) are not removed by Jinja.

**Constraints and Limitations:**

- Comments cannot be nested; attempting to nest them causes errors.
- Line comments are not enabled by default.
- Comments are removed at compile time and do not affect performance.

### Annotated Code Examples

**Example 1: Block Comments**

```jinja
{# This section displays the user's profile #}
<div class="profile">
    <h1>{{ user.name }}</h1>
    {# TODO: Add avatar support #}
    <p>{{ user.bio }}</p>
</div>
```

**Expected Output:**

```html
<div class="profile">
    <h1>Alice</h1>
    <p>Software developer</p>
</div>
```

**Why this output:** The comments are removed from the output. Only the actual content is rendered.

**Example 2: Line Comments (with configuration)**

```python
app.jinja_options['line_comment_prefix'] = '##'
```

```jinja
{% for item in items %}
    <li>{{ item }}</li>  ## this item is rendered
{% endfor %}
## This entire line is ignored
```

**Expected Output:**

```html
<li>Item 1</li>
<li>Item 2</li>
```

**Why this output:** The `##` prefix causes the entire line to be ignored by Jinja. The line comment does not appear in the output.

### Real-World Cases

- **Documentation:** Explaining complex template logic.
- **Debugging:** Commenting out sections during development.
- **Placeholders:** Leaving TODOs for future work.
- **Frontend conflicts:** Using comments to prevent Jinja from processing Vue/Angular syntax.

### References

- Jinja2 Comments — https://jinja.palletsprojects.com/en/stable/templates/#comments
- Jinja2 Line Statements — https://jinja.palletsprojects.com/en/stable/templates/#line-statements

---

## 7. Line Statements (`#`) and Whitespace Control (`{%-` and `-%}`)

### Definitions

**Core Definition:** Line statements allow marking a line as a Jinja statement using a prefix character (default `#`), while whitespace control uses minus signs (`-`) to strip whitespace around tags.

**Technical Definition:** If line statements are enabled by the application (by setting `line_statement_prefix` on the Jinja environment), a line can be marked as a statement using the prefix. For example, if the prefix is `#`, the line `# for item in seq` is equivalent to `{% for item in seq %}`. Line statements can span multiple lines if there are open parentheses, braces, or brackets. Whitespace control uses a minus sign (`-`) added to the start or end of a block, comment, or variable expression to remove whitespace before or after that block. The minus sign must not have whitespace between it and the delimiter.

**Beginner-Friendly Explanation:** Line statements let you write Jinja logic without `{% %}`. Instead of `{% for item in items %}`, you write `# for item in items`. Whitespace control with `-` lets you remove unwanted spaces and newlines around your tags, which is useful for generating clean HTML.

### Purposes

- **Line statements:** To write cleaner templates with less punctuation.
- **Line statements:** To improve readability in templates with many control structures.
- **Whitespace control:** To remove unwanted whitespace and newlines in generated HTML.
- **Whitespace control:** To produce compact output for minified HTML or text formats.
- **Whitespace control:** To control formatting precisely around loops and conditionals.

### Syntax Rules and Structure

**Complete General Syntax:**

**Line Statements (requires configuration):**

```python
app.jinja_options['line_statement_prefix'] = '#'
app.jinja_options['line_comment_prefix'] = '##'
```

```jinja
<ul>
# for item in seq
    <li>{{ item }}</li>
# endfor
</ul>
```

**Whitespace Control:**

```jinja
{%- if foo -%}
    <p>Content</p>
{%- endif -%}
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `# statement` | Line statement (with prefix `#`) |
| `{%-` | Strip whitespace before the tag |
| `-%}` | Strip whitespace after the tag |
| `{{-` | Strip whitespace before expression |
| `-}}` | Strip whitespace after expression |

**Syntax Rules:**

- Line statements require `line_statement_prefix` to be set.
- Line comments require `line_comment_prefix` to be set.
- The minus sign must be directly adjacent to the delimiter (no spaces).
- `{%-` strips whitespace before the tag; `-%}` strips whitespace after.
- Whitespace control can be applied to statements, expressions, and comments.

**Constraints and Limitations:**

- Line statements are not enabled by default.
- Whitespace control can make templates harder to read if overused.
- Incorrect placement of the minus sign causes syntax errors.

### Annotated Code Examples

**Example 1: Line Statements**

```python
from flask import Flask

app = Flask(__name__)
app.jinja_options['line_statement_prefix'] = '#'
app.jinja_options['line_comment_prefix'] = '##'
```

```jinja
<ul>
# for item in items
    <li>{{ item }}</li>
## This is a comment
# endfor
</ul>
```

**Expected Output:**

```html
<ul>
    <li>Item 1</li>
    <li>Item 2</li>
</ul>
```

**Why this output:** The `#` prefix marks each line as a Jinja statement. The `##` line is a comment and is ignored. The output is the same as using `{% for %}` and `{% endfor %}`.

**Example 2: Whitespace Control**

```jinja
{% for item in items -%}
    {{ item }}
{%- endfor %}
```

**Expected Output (with `items = ['a', 'b', 'c']`):**

```
abc
```

**Why this output:** The `-%}` after the `for` tag strips whitespace after it. The `{%-` before `endfor` strips whitespace before it. The result is compact output with no extra whitespace between items.

### Real-World Cases

- **Clean HTML:** Using whitespace control to remove newlines in generated HTML.
- **Minified output:** Producing compact output for performance.
- **Readable templates:** Using line statements to reduce visual clutter.
- **Configuration files:** Generating clean YAML or INI files.

### References

- Jinja2 Line Statements — https://jinja.palletsprojects.com/en/stable/templates/#line-statements
- Jinja2 Whitespace Control — https://jinja.palletsprojects.com/en/stable/templates/#whitespace-control

---

## 8. Literal Escaping (`{% raw %}`)

### Definitions

**Core Definition:** The `{% raw %}` block tells Jinja to treat its contents as plain text, outputting it exactly as written without processing any Jinja syntax.

**Technical Definition:** The `{% raw %}` block prevents Jinja from interpreting `{{ }}`, `{% %}`, or `{# #}` within its contents. Everything between `{% raw %}` and `{% endraw %}` is output literally. This is useful for including Jinja syntax in documentation, embedding content for other template engines (like Vue, Angular, or Go templates), or outputting literal braces.

**Beginner-Friendly Explanation:** Raw blocks tell Jinja "don't touch this part." If you need to show `{{ name }}` on a page without Jinja replacing it, wrap it in `{% raw %}...{% endraw %}`.

### Purposes

- To output literal Jinja syntax in documentation or examples.
- To prevent conflicts with frontend frameworks (Vue, Angular) that use `{{ }}`.
- To embed content for other template engines (Go templates, Prometheus).
- To output literal braces or tags in generated configuration files.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% raw %}
    This {{ will_not_be_interpreted }}
    This {% will_not_be_treated %} as a tag.
    This {# will_not_be_removed #} as a comment.
{% endraw %}
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `{% raw %}` | Start of raw block |
| `{% endraw %}` | End of raw block |
| Content between | Output literally, no Jinja processing |

**Syntax Rules:**

- Everything between `{% raw %}` and `{% endraw %}` is output as-is.
- Raw blocks can contain any Jinja syntax without it being processed.
- Raw blocks cannot be nested.
- A minus sign can be used for whitespace control: `{% raw -%}`.

**Constraints and Limitations:**

- Raw blocks cannot be nested.
- The `{% endraw %}` tag cannot appear inside a raw block.
- Raw blocks do not escape HTML; they only prevent Jinja processing.

### Annotated Code Examples

**Example 1: Basic Raw Block**

```jinja
{% raw %}
<p>Use {{ name }} to display the user's name.</p>
<p>Use {% if user %} to check if a user exists.</p>
{% endraw %}
```

**Expected Output:**

```html
<p>Use {{ name }} to display the user's name.</p>
<p>Use {% if user %} to check if a user exists.</p>
```

**Why this output:** The raw block prevents Jinja from interpreting `{{ name }}` and `{% if user %}`. They are output as literal text.

**Example 2: Vue.js Compatibility**

```jinja
<div id="app">
    {% raw %}
    <p>{{ message }}</p>
    <p>{{ count }}</p>
    {% endraw %}
</div>
```

**Expected Output:**

```html
<div id="app">
    <p>{{ message }}</p>
    <p>{{ count }}</p>
</div>
```

**Why this output:** The raw block allows Vue.js to use its own `{{ }}` syntax without interference from Jinja.

### Real-World Cases

- **Frontend frameworks:** Vue, Angular, and other frameworks use `{{ }}`; raw blocks prevent conflicts.
- **Documentation:** Showing Jinja examples without rendering them.
- **Ansible/Prometheus:** Embedding Go template syntax that also uses `{{ }}`.
- **Configuration files:** Outputting literal braces in YAML or JSON.

### References

- Jinja2 Raw Blocks — https://jinja.palletsprojects.com/en/stable/templates/#escaping
- Jinja2 Escaping — https://jinja.palletsprojects.com/en/stable/templates/#escaping

---

## References

- Jinja2 Template Designer Documentation — https://jinja.palletsprojects.com/en/stable/templates/
- Jinja2 Expressions — https://jinja.palletsprojects.com/en/stable/templates/#expressions
- Jinja2 Statements — https://jinja.palletsprojects.com/en/stable/templates/#list-of-control-structures
- Jinja2 Variables — https://jinja.palletsprojects.com/en/stable/templates/#variables
- Jinja2 Filters — https://jinja.palletsprojects.com/en/stable/templates/#filters
- Jinja2 Built-in Filters — https://jinja.palletsprojects.com/en/stable/templates/#builtin-filters
- Jinja2 Tests — https://jinja.palletsprojects.com/en/stable/templates/#tests
- Jinja2 Built-in Tests — https://jinja.palletsprojects.com/en/stable/templates/#builtin-tests
- Jinja2 Comments — https://jinja.palletsprojects.com/en/stable/templates/#comments
- Jinja2 Whitespace Control — https://jinja.palletsprojects.com/en/stable/templates/#whitespace-control
- Jinja2 Line Statements — https://jinja.palletsprojects.com/en/stable/templates/#line-statements
- Jinja2 Escaping (Raw Blocks) — https://jinja.palletsprojects.com/en/stable/templates/#escaping
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/