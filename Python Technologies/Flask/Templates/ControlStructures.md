# Jinja Template Control Structures: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Control structures in Jinja are special keywords enclosed in `{% ... %}` blocks that control the flow of template rendering, including conditionals, loops, and variable assignments.

**Technical Definition:** A control structure (or statement) is a special keyword that can be used through blocks to achieve conditional logic in a template. Jinja's control structures include `if`/`elif`/`else` for conditional rendering, `for` for iteration, `set` for variable assignment, `with` for scoped blocks, and `block`/`extends`/`include` for template composition . Control structures appear inside `{% ... %}` blocks in the default syntax . Unlike Python, Jinja does not support `break` or `continue` in loops; instead, sequence filtering is used to skip items . Variable scoping in Jinja differs from Python: variables set inside blocks (including loops) are not visible outside of them, with `if` statements being the only exception .

**Beginner-Friendly Explanation:** Control structures are the logic of your template. `{% if user %}` shows content only if a user exists. `{% for item in items %}` repeats content for each item. `{% set x = 10 %}` creates a variable. They use `{% %}` because they control what gets rendered, not what value gets output. Think of them as the "if" and "for" statements of your HTML pages.

### Key Characteristics

- **Pythonic syntax:** Jinja's control structures are deliberately similar to Python's, making them intuitive for Python developers .
- **Block-based:** All control structures use `{% ... %}` delimiters and require explicit closing tags (`{% endif %}`, `{% endfor %}`, `{% endset %}`) .
- **Loop metadata:** The special `loop` variable provides iteration metadata (index, first, last, length, etc.) .
- **Loop filtering:** Sequences can be filtered during iteration using `{% for x in seq if condition %}`, replacing Python's `continue` .
- **Recursive loops:** The `recursive` modifier enables rendering of nested hierarchical data like comment trees and sitemaps .
- **Scoping rules:** Variables set inside blocks/loops are scoped to that block; namespace objects (Jinja 2.10+) propagate changes across scopes .

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTML and Python control flow (if/for).
- Familiarity with Flask's `render_template()` and template context.
- Knowledge of Python dictionaries, lists, and loops.

### Related Programming Areas

- **Flask Templating:** Jinja is Flask's default template engine.
- **Template Inheritance:** Control structures work with `{% extends %}` and `{% block %}`.
- **Frontend Frameworks:** Vue/Angular use similar syntax; raw blocks prevent conflicts.
- **Data Rendering:** Loops and conditionals are essential for rendering dynamic data.
- **Scoping and Namespaces:** Understanding variable scope prevents common template bugs.

### Core Concepts / Features

1. `if`, `elif`, `else` (Conditional Rendering)
2. `for` (Iteration)
3. Loop Metadata (The `loop` Variable Properties)
4. Conditional Rendering (Inline and Block Forms)
5. Loop Filtering and the `recursive` Modifier
6. Variable Scoping Within Blocks and Loops (`set` Behavior)

---

## 1. `if`, `elif`, `else` (Conditional Rendering)

### Definitions

**Core Definition:** The `if` statement in Jinja conditionally renders a block of template content based on whether an expression evaluates to true, with `elif` and `else` providing additional branches.

**Technical Definition:** The `if` statement in Jinja is comparable with Python's `if` statement. In the simplest form, it tests if a variable is defined, not empty, or not false. For multiple branches, `elif` and `else` can be used like in Python, and complex expressions are supported . The `if` statement does not introduce a new scope, meaning variables set inside an `if` block are visible outside of it . The condition evaluates to false for: `False`, `0`, `0.0`, empty strings (`""`), the string `"0"`, and empty arrays .

**Beginner-Friendly Explanation:** `{% if user %}` shows content only when `user` is truthy. `{% elif user.admin %}` checks a second condition. `{% else %}` is the fallback. It works exactly like Python's `if`/`elif`/`else`, but you must close it with `{% endif %}`.

### Purposes

- To conditionally display content based on variable values.
- To handle empty lists or undefined variables gracefully.
- To implement multi-branch logic (e.g., different messages for different user roles).
- To control the rendering of UI elements based on application state.
- To provide fallback content when conditions are not met.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% if condition %}
    Content when true
{% elif other_condition %}
    Content when other condition is true
{% else %}
    Fallback content
{% endif %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% if condition %}` | Starts a conditional block; condition is a Jinja expression |
| `{% elif condition %}` | Optional; additional condition to check |
| `{% else %}` | Optional; renders when no conditions are true |
| `{% endif %}` | Required; closes the conditional block |

**Syntax Rules:**

- The `{% if %}` block must be closed with `{% endif %}` .
- `elif` and `else` are optional; multiple `elif` branches are allowed.
- Conditions can use comparison operators (`==`, `!=`, `<`, `>`), logical operators (`and`, `or`, `not`), and tests (`is defined`, `is none`).
- The `if` statement does not introduce a new scope .

**Constraints and Limitations:**

- Jinja does not support the ternary operator in the same form as Python; use inline `if` expressions instead.
- Complex conditions should be simplified or moved to Python code.
- Undefined variables in conditions may cause silent failures; use `is defined` tests.

### Annotated Code Examples

**Example 1: Basic Conditional Rendering**

```jinja
{# templates/conditional.html #}
{% if user %}
    <h1>Welcome, {{ user.name }}!</h1>
{% elif guest_name %}
    <h1>Welcome, {{ guest_name }}!</h1>
{% else %}
    <h1>Welcome, Guest!</h1>
{% endif %}
```

**Expected Output (when `user = {"name": "Alice"}`):**

```html
<h1>Welcome, Alice!</h1>
```

**Expected Output (when `guest_name = "Bob"`):**

```html
<h1>Welcome, Bob!</h1>
```

**Expected Output (when neither is defined):**

```html
<h1>Welcome, Guest!</h1>
```

**Why this output:** The `{% if user %}` checks if `user` is truthy. If `user` is a non-empty dictionary, it renders the first block. If not, `{% elif guest_name %}` checks the second condition. If neither is true, the `{% else %}` block renders.

**Example 2: Conditional with Loop**

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

**Expected Output (with empty list):**

```html
<p>No users found.</p>
```

**Why this output:** The `{% if users %}` checks if the list is non-empty. If so, the loop iterates over each user. The inline `if` expression (`"Admin" if user.admin else "User"`) conditionally displays the role. If the list is empty, the `{% else %}` block renders.

### Real-World Cases

- **User authentication:** Showing login/logout links based on session state.
- **Role-based UI:** Displaying admin controls only for administrators.
- **Empty states:** Showing "No items found" when a list is empty.
- **Error handling:** Displaying error messages when validation fails.

### References

- Jinja2 `if` Statement — https://jinja.palletsprojects.com/en/stable/templates/#if
- Jinja2 Control Structures — https://jinja.palletsprojects.com/en/stable/templates/#list-of-control-structures

---

## 2. `for` (Iteration)

### Definitions

**Core Definition:** The `for` statement in Jinja iterates over each item in a sequence (list, dictionary, tuple, etc.) and renders the block content for each item.

**Technical Definition:** The `for` control structure loops over each item in a sequence. The syntax is very similar to Python's `for` loop: `{% for item in sequence %}...{% endfor %}` . Jinja's `for` loop can iterate over lists, dictionaries (returning key-value pairs), and other iterables. Unlike Python, Jinja does not support `break` or `continue`; instead, use `{% for item in seq if condition %}` to filter items . If the sequence is empty, the optional `{% else %}` block renders a replacement .

**Beginner-Friendly Explanation:** `{% for item in items %}` repeats the content between the tags for each item in the list. It's exactly like Python's for loop but written with `{% %}`. You must close it with `{% endfor %}`.

### Purposes

- To render lists of data (users, products, comments).
- To iterate over dictionary keys and values.
- To generate repetitive HTML structures (tables, lists, cards).
- To process nested data structures with recursive loops.
- To provide fallback content when a sequence is empty.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% for item in sequence %}
    Content for each item
{% endfor %}

{# With else fallback #}
{% for item in sequence %}
    {{ item }}
{% else %}
    <p>No items found</p>
{% endfor %}

{# With filtering #}
{% for item in sequence if condition %}
    {{ item }}
{% endfor %}

{# Iterating over dictionary #}
{% for key, value in my_dict.items() %}
    {{ key }}: {{ value }}
{% endfor %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% for item in sequence %}` | Starts the loop; `item` is the loop variable |
| `{% endfor %}` | Closes the loop (required) |
| `{% else %}` | Optional; renders when the sequence is empty |
| `if condition` | Optional; filters items during iteration |

**Syntax Rules:**

- The loop must be closed with `{% endfor %}` .
- The `{% else %}` block renders when the sequence is empty or all items were filtered out .
- Dictionaries can be iterated with `{% for key, value in my_dict.items() %}` .
- Loop filtering uses `if condition` at the end of the `for` tag .
- The `recursive` modifier enables recursive loops for nested data .

**Constraints and Limitations:**

- `break` and `continue` are not supported; use filtering instead .
- Dictionary iteration order is not guaranteed; use the `dictsort` filter for sorted output .
- Modifying the sequence during iteration is not supported.

### Annotated Code Examples

**Example 1: Basic Loop Over a List**

```jinja
{# templates/users.html #}
<ul>
{% for user in users %}
    <li>{{ user.name }} ({{ user.email }})</li>
{% else %}
    <li>No users found</li>
{% endfor %}
</ul>
```

**Expected Output (with two users):**

```html
<ul>
    <li>Alice (alice@example.com)</li>
    <li>Bob (bob@example.com)</li>
</ul>
```

**Expected Output (with empty list):**

```html
<ul>
    <li>No users found</li>
</ul>
```

**Why this output:** The loop iterates over each user in the `users` list. If the list is empty, the `{% else %}` block renders the fallback message.

**Example 2: Iterating Over a Dictionary**

```jinja
{# templates/settings.html #}
<dl>
{% for key, value in settings.items() %}
    <dt>{{ key }}</dt>
    <dd>{{ value }}</dd>
{% endfor %}
</dl>
```

**Expected Output (with `settings = {"theme": "dark", "lang": "en"}`):**

```html
<dl>
    <dt>theme</dt>
    <dd>dark</dd>
    <dt>lang</dt>
    <dd>en</dd>
</dl>
```

**Why this output:** The `for key, value in settings.items()` syntax unpacks the dictionary into key-value pairs, rendering each as a definition term and description.

### Real-World Cases

- **User lists:** Rendering tables of users, products, or orders.
- **Navigation menus:** Looping over menu items to generate links.
- **Comment sections:** Iterating over comments with nested replies.
- **Dashboard widgets:** Rendering metrics cards from a list of data.

### References

- Jinja2 `for` Statement — https://jinja.palletsprojects.com/en/stable/templates/#for
- Jinja2 Loop Filtering — https://jinja.palletsprojects.com/en/stable/templates/#loop-filtering

---

## 3. Loop Metadata (The `loop` Variable Properties)

### Definitions

**Core Definition:** Inside a `for` loop, Jinja provides a special `loop` variable that exposes metadata about the current iteration, such as the index, whether it's the first or last item, and the total number of items.

**Technical Definition:** Inside of a for-loop block, special variables are available through the `loop` object. These include `loop.index` (1-indexed current iteration), `loop.index0` (0-indexed), `loop.revindex` (1-indexed from the end), `loop.revindex0` (0-indexed from the end), `loop.first` (True if first iteration), `loop.last` (True if last iteration), `loop.length` (number of items in the sequence), `loop.cycle` (helper to cycle between values), `loop.depth` (recursion depth, 1-indexed), and `loop.depth0` (recursion depth, 0-indexed) .

**Beginner-Friendly Explanation:** The `loop` variable gives you information about where you are in the loop. `loop.index` tells you the current iteration number (starting at 1). `loop.first` is true on the first iteration. `loop.last` is true on the last. This is useful for adding commas between items or styling the first/last item differently.

### Purposes

- To add separators (commas, pipes) between loop items without trailing characters.
- To style the first or last item differently.
- To display the current iteration number.
- To cycle CSS classes for alternating row colors.
- To track recursion depth in recursive loops.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% for item in items %}
    {{ loop.index }}: {{ item }}
    {% if loop.first %}(first){% endif %}
    {% if loop.last %}(last){% endif %}
{% endfor %}
```

**Component Breakdown:**

| Variable | Description |
|----------|-------------|
| `loop.index` | Current iteration (1-indexed) |
| `loop.index0` | Current iteration (0-indexed) |
| `loop.revindex` | Iterations from the end (1-indexed) |
| `loop.revindex0` | Iterations from the end (0-indexed) |
| `loop.first` | True if first iteration |
| `loop.last` | True if last iteration |
| `loop.length` | Total number of items in the sequence |
| `loop.cycle(...)` | Cycles between values on each iteration |
| `loop.depth` | Recursion depth (1-indexed) |
| `loop.depth0` | Recursion depth (0-indexed) |

**Syntax Rules:**

- The `loop` variable is only available inside a `for` loop.
- `loop.index` starts at 1; `loop.index0` starts at 0.
- `loop.first` and `loop.last` are booleans.
- `loop.cycle('odd', 'even')` alternates between the provided values.
- `loop.length` is only available if the sequence has a known length.

**Constraints and Limitations:**

- `loop.length` may not be available for generators or iterators without a predefined length.
- The `loop` variable is overwritten in nested loops; use `loop.depth` to track nesting.
- Custom loop variables cannot be added.

### Annotated Code Examples

**Example 1: Adding Separators Between Items**

```jinja
{# templates/tags.html #}
<p>Tags:
{% for tag in tags %}
    {{ tag }}{% if not loop.last %}, {% endif %}
{% endfor %}
</p>
```

**Expected Output (with `tags = ['python', 'flask', 'jinja']`):**

```html
<p>Tags:
    python, flask, jinja
</p>
```

**Why this output:** The `{% if not loop.last %}` condition adds a comma after each tag except the last one. This avoids a trailing comma.

**Example 2: Alternating Row Colors with `loop.cycle`**

```jinja
{# templates/table.html #}
<table>
{% for row in rows %}
    <tr class="{{ loop.cycle('odd', 'even') }}">
        <td>{{ row.name }}</td>
        <td>{{ row.value }}</td>
    </tr>
{% endfor %}
</table>
```

**Expected Output:**

```html
<table>
    <tr class="odd"><td>...</td><td>...</td></tr>
    <tr class="even"><td>...</td><td>...</td></tr>
    <tr class="odd"><td>...</td><td>...</td></tr>
</table>
```

**Why this output:** `loop.cycle('odd', 'even')` alternates between "odd" and "even" on each iteration, producing striped table rows.

**Example 3: Displaying Iteration Numbers**

```jinja
{# templates/steps.html #}
<ol>
{% for step in steps %}
    <li>Step {{ loop.index }} of {{ loop.length }}: {{ step }}</li>
{% endfor %}
</ol>
```

**Expected Output (with `steps = ['First', 'Second', 'Third']`):**

```html
<ol>
    <li>Step 1 of 3: First</li>
    <li>Step 2 of 3: Second</li>
    <li>Step 3 of 3: Third</li>
</ol>
```

**Why this output:** `loop.index` gives the current iteration number (1-indexed), and `loop.length` gives the total number of items.

### Real-World Cases

- **E-commerce:** Alternating row colors in product tables.
- **Navigation:** Adding separators between breadcrumb links.
- **Documentation:** Numbering steps in a tutorial.
- **Dashboards:** Displaying progress indicators ("Step 2 of 5").

### References

- Jinja2 Loop Variables — https://jinja.palletsprojects.com/en/stable/templates/#for
- Jinja2 `loop.cycle` — https://jinja.palletsprojects.com/en/stable/templates/#for

---

## 4. Conditional Rendering (Inline and Block Forms)

### Definitions

**Core Definition:** Conditional rendering in Jinja allows content to be displayed or hidden based on conditions, using either block form (`{% if %}`) or inline form (`<value> if <condition> else <other_value>`).

**Technical Definition:** Jinja supports conditional rendering in two forms. The block form uses `{% if %}...{% elif %}...{% else %}...{% endif %}` for multi-branch logic . The inline form uses the expression `<value> if <condition> else <other_value>`, which can be used directly within `{{ }}` expressions . Inline conditionals are useful for simple decisions within output expressions, while block conditionals are used for larger sections of template content.

**Beginner-Friendly Explanation:** You can use `{% if %}` for big blocks of content that should only appear under certain conditions. For quick decisions inside a single expression, use inline: `{{ "Yes" if user.active else "No" }}`. Both work the same way logically.

### Purposes

- To display different content based on user state or data values.
- To conditionally apply CSS classes or HTML attributes.
- To show/hide UI elements based on application logic.
- To provide fallback values when data is missing.
- To implement role-based or permission-based rendering.

### Syntax Rules and Structure

**Complete General Syntax:**

**Block Form:**

```jinja
{% if user.is_admin %}
    <a href="/admin">Admin Panel</a>
{% endif %}
```

**Inline Form:**

```jinja
{{ "Active" if user.is_active else "Inactive" }}
```

**Component Breakdown:**

| Form | Syntax | Use Case |
|------|--------|----------|
| Block | `{% if %}` | Large sections, multi-branch logic |
| Inline | `{{ x if cond else y }}` | Simple value selection |

**Syntax Rules:**

- Block form requires `{% endif %}`.
- Inline form is an expression, used within `{{ }}`.
- Inline conditionals can be nested but become hard to read.
- Both forms can use the same types of conditions.

**Constraints and Limitations:**

- Inline conditionals cannot contain statements (loops, includes).
- Block conditionals cannot be used inside expressions.
- Complex inline conditionals should be replaced with block form or Python logic.

### Annotated Code Examples

**Example 1: Block Conditional for Role-Based UI**

```jinja
{# templates/nav.html #}
<nav>
    <a href="/">Home</a>
    {% if user.is_authenticated %}
        <a href="/profile">Profile</a>
        {% if user.is_admin %}
            <a href="/admin">Admin</a>
        {% endif %}
        <a href="/logout">Logout</a>
    {% else %}
        <a href="/login">Login</a>
    {% endif %}
</nav>
```

**Expected Output (for admin user):**

```html
<nav>
    <a href="/">Home</a>
    <a href="/profile">Profile</a>
    <a href="/admin">Admin</a>
    <a href="/logout">Logout</a>
</nav>
```

**Why this output:** The outer `{% if user.is_authenticated %}` shows profile and logout links for logged-in users. The nested `{% if user.is_admin %}` shows the admin link only for administrators.

**Example 2: Inline Conditional for Status Badge**

```jinja
{# templates/status.html #}
<span class="badge {{ 'badge-success' if user.active else 'badge-danger' }}">
    {{ "Active" if user.active else "Inactive" }}
</span>
```

**Expected Output (for active user):**

```html
<span class="badge badge-success">
    Active
</span>
```

**Why this output:** Both the CSS class and the displayed text use inline conditionals. The `badge-success` class is applied for active users, and "Active" is displayed.

### Real-World Cases

- **Navigation menus:** Showing different links based on authentication state.
- **Status indicators:** Displaying "Active" or "Inactive" with appropriate styling.
- **Permission-based UI:** Showing admin controls only for administrators.
- **Data display:** Showing "N/A" when a field is missing.

### References

- Jinja2 `if` Statement — https://jinja.palletsprojects.com/en/stable/templates/#if
- Jinja2 Inline Expressions — https://jinja.palletsprojects.com/en/stable/templates/#if-expression

---

## 5. Loop Filtering and the `recursive` Modifier

### Definitions

**Core Definition:** Loop filtering allows skipping items during iteration using an `if` condition in the `for` tag, while the `recursive` modifier enables a loop to call itself for nested hierarchical data.

**Technical Definition:** Unlike Python, Jinja does not support `break` or `continue` in loops. Instead, the sequence can be filtered during iteration using `{% for item in sequence if condition %}`, which skips items that do not match the condition . The special `loop` variable counts correctly, not counting filtered items . The `recursive` modifier enables recursive loops for nested data such as sitemaps or comment trees. To use recursive loops, add the `recursive` modifier to the loop definition and call the `loop` variable with the new iterable where recursion should occur .

**Beginner-Friendly Explanation:** Filtering lets you loop over only the items that meet a condition, like `{% for user in users if user.active %}` to show only active users. Recursive loops let you render trees of data—like a comment thread where each comment can have replies, which can have their own replies. The loop calls itself for each level of the tree.

### Purposes

- **Filtering:** To skip items during iteration without using `continue`.
- **Filtering:** To loop over only active, visible, or relevant items.
- **Recursive loops:** To render hierarchical data (comment trees, file systems, sitemaps).
- **Recursive loops:** To handle data with unknown nesting depth.
- **Recursive loops:** To generate nested HTML lists or menus.

### Syntax Rules and Structure

**Complete General Syntax:**

**Filtering:**

```jinja
{% for user in users if user.active %}
    <li>{{ user.name }}</li>
{% endfor %}
```

**Recursive Loops:**

```jinja
{% for item in items recursive %}
    <li>{{ item.title }}
    {% if item.children %}
        <ul>{{ loop(item.children) }}</ul>
    {% endif %}
    </li>
{% endfor %}
```

**Component Breakdown:**

| Feature | Syntax | Description |
|---------|--------|-------------|
| Filtering | `for x in seq if cond` | Skips items not matching `cond` |
| Recursive | `for x in seq recursive` | Enables recursive calls |
| Recursive call | `loop(new_iterable)` | Renders the loop body with new data |

**Syntax Rules:**

- Filtering uses `if condition` at the end of the `for` tag .
- Filtered items are not counted in `loop.index` .
- The `recursive` modifier must be added to the `for` tag .
- The loop body is called recursively with `loop(new_iterable)` .
- The `loop.depth` and `loop.depth0` variables track recursion depth .

**Constraints and Limitations:**

- Filtering cannot be used with `else` to detect if all items were filtered; the `else` block only runs if the sequence was empty .
- Recursive loops can cause infinite recursion if data has cycles.
- Deep recursion may hit Python's recursion limit.

### Annotated Code Examples

**Example 1: Loop Filtering**

```jinja
{# templates/active_users.html #}
<ul>
{% for user in users if user.active %}
    <li>{{ user.name }} ({{ loop.index }})</li>
{% else %}
    <li>No active users</li>
{% endfor %}
</ul>
```

**Expected Output (with 3 users, 2 active):**

```html
<ul>
    <li>Alice (1)</li>
    <li>Charlie (2)</li>
</ul>
```

**Why this output:** The `if user.active` filter skips inactive users. The `loop.index` counts only the active users (1 and 2), not the skipped ones. If no users were active, the `{% else %}` block would render.

**Example 2: Recursive Loop for Comment Tree**

```jinja
{# templates/comments.html #}
<ul class="comments">
{% for comment in comments recursive %}
    <li>
        <p>{{ comment.author }}: {{ comment.text }}</p>
        {% if comment.replies %}
            <ul class="replies">{{ loop(comment.replies) }}</ul>
        {% endif %}
    </li>
{% endfor %}
</ul>
```

**Expected Output (with nested replies):**

```html
<ul class="comments">
    <li>
        <p>Alice: Great post!</p>
        <ul class="replies">
            <li>
                <p>Bob: Thanks!</p>
                <ul class="replies">
                    <li><p>Alice: You're welcome!</p></li>
                </ul>
            </li>
        </ul>
    </li>
</ul>
```

**Why this output:** The `recursive` modifier allows the loop to call itself with `loop(comment.replies)` for each level of replies. Each nested level renders its own `<ul>` with the appropriate indentation.

### Real-World Cases

- **Comment threads:** Rendering nested replies with recursive loops.
- **File browsers:** Displaying directory trees with recursive loops.
- **Sitemaps:** Generating nested navigation with recursive loops.
- **Active item filtering:** Showing only enabled menu items with loop filtering.

### References

- Jinja2 Loop Filtering — https://jinja.palletsprojects.com/en/stable/templates/#loop-filtering
- Jinja2 Recursive Loops — https://jinja.palletsprojects.com/en/stable/templates/#for

---

## 6. Variable Scoping Within Blocks and Loops (`set` Behavior)

### Definitions

**Core Definition:** Variable scoping in Jinja determines where variables set with `{% set %}` are visible. Variables set inside blocks or loops are scoped to that block and are not visible outside of it.

**Technical Definition:** It is not possible to set variables inside a block and have them show up outside of it. This also applies to loops. The only exception to this rule are `if` statements, which do not introduce a scope . As of Jinja 2.10, more complex use cases can be handled using namespace objects which allow propagating of changes across scopes . The `obj.attr` notation in the `set` tag is only allowed for namespace objects; attempting to assign an attribute on any other object raises an exception .

**Beginner-Friendly Explanation:** If you set a variable inside a `{% for %}` loop, it won't be available after the loop ends. This is different from Python. To keep a variable across scopes, use a namespace object. The `if` statement is the exception—variables set inside `if` blocks are visible outside.

### Purposes

- To understand why variables set in loops don't persist (a common source of bugs).
- To use namespace objects for accumulating values across iterations.
- To use the `loop` variable's `else` block instead of a flag variable.
- To write correct templates that don't rely on cross-scope variable persistence.

### Syntax Rules and Structure

**Complete General Syntax:**

**Scoping Behavior (Bug Example):**

```jinja
{% set iterated = false %}
{% for item in seq %}
    {{ item }}
    {% set iterated = true %}
{% endfor %}
{% if not iterated %}
    did not iterate
{% endif %}
```

This does not work as expected because `iterated` is scoped to the loop body .

**Correct Alternatives:**

```jinja
{# Use loop else block #}
{% for item in seq %}
    {{ item }}
{% else %}
    did not iterate
{% endfor %}
```

```jinja
{# Use namespace object (Jinja 2.10+) #}
{% set ns = namespace(found=false) %}
{% for item in items %}
    {% if item.check_something() %}
        {% set ns.found = true %}
    {% endif %}
{% endfor %}
Found: {{ ns.found }}
```

**Component Breakdown:**

| Approach | Description |
|----------|-------------|
| `{% set %}` in loop | Scoped to loop body; not visible outside |
| `{% set %}` in `if` | Visible outside (no new scope) |
| `namespace()` | Object that allows cross-scope mutation |
| `loop else` | Renders when sequence is empty |

**Syntax Rules:**

- Variables set inside `for` loops are not visible outside the loop .
- Variables set inside `if` blocks are visible outside (no new scope) .
- Namespace objects allow attribute assignment with `{% set ns.attr = value %}` .
- The `obj.attr` notation in `set` only works for namespace objects .

**Constraints and Limitations:**

- Namespace objects require Jinja 2.10 or later.
- Without namespaces, accumulating values across loop iterations is not possible with `set` alone.
- The `loop` variable is the only variable that persists across iterations.

### Annotated Code Examples

**Example 1: Bug — Variable Set in Loop Not Persisting**

```jinja
{# This does NOT work as expected #}
{% set found = false %}
{% for item in items %}
    {% if item == "target" %}
        {% set found = true %}
    {% endif %}
{% endfor %}
{% if found %}
    Found!
{% else %}
    Not found.
{% endif %}
```

**Expected Output:**

```
Not found.
```

**Why this output:** The `{% set found = true %}` inside the loop is scoped to the loop body. The outer `found` remains `false`. This is a common scoping bug.

**Example 2: Fix with Namespace Object**

```jinja
{% set ns = namespace(found=false) %}
{% for item in items %}
    {% if item == "target" %}
        {% set ns.found = true %}
    {% endif %}
{% endfor %}
{% if ns.found %}
    Found!
{% else %}
    Not found.
{% endif %}
```

**Expected Output (when "target" is in items):**

```
Found!
```

**Why this output:** The namespace object `ns` allows the `found` attribute to be modified across scopes. The change persists after the loop ends.

**Example 3: Fix with Loop Else**

```jinja
{% for item in items %}
    {% if item == "target" %}
        Found!
    {% endif %}
{% else %}
    Not found.
{% endfor %}
```

**Expected Output (when "target" is in items):**

```
Found!
```

**Why this output:** The `{% else %}` block renders only when the sequence is empty or all items were filtered out. This is a cleaner alternative when the loop's sole purpose is to check for existence.

### Real-World Cases

- **Finding items:** Checking if a value exists in a list (use `loop else` or namespaces).
- **Accumulating values:** Summing numbers or collecting items across iterations (use namespaces).
- **Flags:** Setting a flag when a condition is met (use namespaces or `loop else`).
- **Counters:** Counting items that match a condition (use namespaces).

### References

- Jinja2 Scoping Behavior — https://jinja.palletsprojects.com/en/stable/templates/#assignments
- Jinja2 Namespace Objects — https://jinja.palletsprojects.com/en/stable/templates/#assignments
- Jinja2 Block Scoping — https://jinja.palletsprojects.com/en/stable/templates/#block-scoping

---

## References

- Jinja2 Template Designer Documentation — https://jinja.palletsprojects.com/en/stable/templates/
- Jinja2 Control Structures — https://jinja.palletsprojects.com/en/stable/templates/#list-of-control-structures
- Jinja2 `if` Statement — https://jinja.palletsprojects.com/en/stable/templates/#if
- Jinja2 `for` Statement — https://jinja.palletsprojects.com/en/stable/templates/#for
- Jinja2 Loop Filtering — https://jinja.palletsprojects.com/en/stable/templates/#loop-filtering
- Jinja2 Scoping Behavior — https://jinja.palletsprojects.com/en/stable/templates/#assignments
- Jinja2 Namespace Objects — https://jinja.palletsprojects.com/en/stable/templates/#assignments
- Jinja2 Block Scoping — https://jinja.palletsprojects.com/en/stable/templates/#block-scoping
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Flask Templating — https://flask.palletsprojects.com/en/stable/templating/