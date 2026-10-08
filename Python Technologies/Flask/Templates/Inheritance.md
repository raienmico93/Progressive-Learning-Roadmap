# Flask Template Inheritance: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Template inheritance is a Jinja2 feature that allows child templates to extend a base "skeleton" template, overriding specific sections (called blocks) while inheriting the rest of the structure.

**Technical Definition:** Template inheritance is implemented through the `{% extends %}` tag, which must be the first tag in a child template. When Jinja renders a child template, it first locates the parent template, then evaluates the child's blocks, overriding the corresponding blocks in the parent. The `{{ super() }}` function renders the parent block's content, allowing incremental extension rather than complete replacement. Jinja supports dynamic inheritance, where the `{% extends %}` tag can be placed inside an `{% if %}` block to conditionally inherit from different parent templates. Jinja does not support multiple inheritance; only one `extends` tag may be executed per rendering.

**Beginner-Friendly Explanation:** Template inheritance lets you create a master layout with common elements (header, footer, navigation) and then build individual pages that fill in only the unique content. Instead of copying the header and footer into every page, you define them once in a base template and let each page "extend" it. This is like a fill-in-the-blanks system where the base template provides the structure and the child templates provide the content.

### Key Characteristics

- **Single inheritance:** Jinja does not support multiple inheritance; only one `{% extends %}` tag may be executed per rendering.
- **Block overrides:** Child templates override named blocks defined in the parent template.
- **`super()` support:** The parent block's content can be rendered using `{{ super() }}`.
- **Dynamic inheritance:** The `{% extends %}` tag can be conditional, enabling multi-theme applications.
- **Block nesting:** Blocks can be nested for complex layouts.
- **Scope awareness:** Variables set inside blocks are not visible outside; use the `scoped` modifier for loop variables.
- **Blueprint integration:** Blueprint templates can extend application templates, with application templates taking priority.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTML and Jinja2 syntax.
- Familiarity with Flask's `render_template()` and template directories.
- Knowledge of Jinja2 control structures (`{% if %}`, `{% for %}`).

### Related Programming Areas

- **Flask Templating:** Jinja2 is Flask's default template engine.
- **Jinja2:** The underlying template engine with inheritance as its most powerful feature.
- **Blueprints:** Blueprint-specific template folders integrate with template inheritance.
- **Multi-theme applications:** Dynamic inheritance enables theme switching.
- **Web application architecture:** Inheritance promotes DRY principles and maintainability.

### Core Concepts / Features

1. Base Templates
2. Blocks
3. Child Templates
4. Reusable Layouts
5. Nested Inheritance
6. `super()` Block Execution
7. Dynamic Inheritance (Conditional `extends`)

---

## 1. Base Templates

### Definitions

**Core Definition:** A base template is a "skeleton" template that defines the common structure and elements of a website, with named blocks that child templates can override.

**Technical Definition:** A base template is a Jinja2 template file that contains the HTML skeleton of a page, including `{% block %}` tags that define override points. It is typically named `base.html`, `layout.html`, or `skeleton.html`. The base template is never rendered directly; it is always extended by child templates. The `{% block %}` tags define four blocks that child templates can fill in: `head`, `title`, `content`, and `footer`.

**Beginner-Friendly Explanation:** A base template is like a picture frame. It provides the structure (the frame) with empty spaces (blocks) where each page puts its own content. You write the header, navigation, and footer once in the base template, and then each page just fills in its unique content.

### Purposes

- To define the common HTML structure (doctype, head, body, footer) in one place.
- To provide named blocks for child templates to override.
- To eliminate duplication of headers, footers, and navigation across pages.
- To ensure consistent layout and styling across the entire application.
- To simplify maintenance by centralizing structural changes.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head>
    {% block head %}
        <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
        <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
</head>
<body>
    <div id="content">{% block content %}{% endblock %}</div>
    <div id="footer">
        {% block footer %}
            © Copyright 2010 by <a href="http://domain.invalid/">you</a>.
        {% endblock %}
    </div>
</body>
</html>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% block head %}` | Block for head content (styles, meta tags) |
| `{% block title %}` | Block for the page title |
| `{% block content %}` | Block for the main page content |
| `{% block footer %}` | Block for the footer content |

**Syntax Rules:**

- The base template is a regular Jinja2 template with `{% block %}` tags.
- Blocks are named placeholders that child templates can override.
- The base template can have default content inside blocks.
- The base template is never rendered directly; it is always extended.

**Constraints and Limitations:**

- The base template cannot know which child template is extending it.
- Blocks with the same name must be unique within a template.
- The base template's default content is used if a child does not override the block.

### Annotated Code Examples

**Example 1: Basic Base Template**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head>
    {% block head %}
        <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
        <title>{% block title %}{% endblock %} - My Site</title>
    {% endblock %}
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('about') }}">About</a>
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        {% block footer %}
            <p>&copy; 2024 My Site</p>
        {% endblock %}
    </footer>
</body>
</html>
```

**Expected Output (when extended by a child template):**
- The base template provides the doctype, head, navigation, and footer.
- Child templates fill in the `title` and `content` blocks.

**Why this output:** The base template defines the overall structure with blocks for dynamic content. The `{% block head %}`, `{% block title %}`, `{% block content %}`, and `{% block footer %}` tags mark the areas where child templates can inject their content.

### Real-World Cases

- **Multi-page websites:** Consistent header, footer, and navigation across all pages.
- **Admin panels:** Shared layout with page-specific content areas.
- **Documentation sites:** Consistent sidebar and navigation structure.
- **E-commerce:** Shared product page layout with variable content.

### References

- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/

---

## 2. Blocks

### Definitions

**Core Definition:** Blocks are named placeholders within a template that child templates can override, defined using the `{% block name %}...{% endblock %}` syntax.

**Technical Definition:** The `{% block %}` tag defines a section that child templates can override. All the `block` tag does is tell the template engine that a child template may override those portions of the template. Blocks can be nested for more complex layouts. Starting with Jinja 2.2, blocks can be marked as `scoped` to make loop variables available inside the block. When a child template overrides a block, the parent's block content is replaced unless `{{ super() }}` is called.

**Beginner-Friendly Explanation:** Blocks are the empty spaces in your base template. You give each block a name (like "content" or "title"), and then each page fills in those blocks with its own content. If a page doesn't fill in a block, the base template's default content is used.

### Purposes

- To define override points in base templates.
- To allow child templates to customize specific sections.
- To provide default content that child templates can optionally replace.
- To support nested layouts with multiple levels of blocks.
- To enable `super()` calls for incremental extension.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Defining a block in a base template #}
{% block block_name %}
    Default content
{% endblock %}

{# Overriding a block in a child template #}
{% block block_name %}
    New content
{% endblock %}

{# Using super() to include parent content #}
{% block block_name %}
    {{ super() }}
    Additional content
{% endblock %}

{# Scoped block (makes loop variables available) #}
{% block loop_item scoped %}
    {{ item }}
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% block name %}` | Defines a named block |
| `{% endblock %}` | Closes the block (name optional) |
| `{{ super() }}` | Renders the parent block's content |
| `scoped` | Modifier to make loop variables available |

**Syntax Rules:**

- Block names must be unique within a template.
- Blocks can be nested inside other blocks.
- The `{% endblock %}` tag can optionally include the block name for clarity.
- The `scoped` modifier is required to access loop variables inside a block.
- When overriding a block, the `scoped` modifier does not have to be provided.

**Constraints and Limitations:**

- Blocks cannot be defined outside of a template's root level.
- Block names cannot be dynamically generated.
- Variables set inside a block are scoped to that block; they do not leak outside.

### Annotated Code Examples

**Example 1: Defining and Overriding Blocks**

```jinja
{# templates/base.html #}
<html>
<head>
    <title>{% block title %}Default Title{% endblock %}</title>
</head>
<body>
    {% block content %}
        <p>Default content</p>
    {% endblock %}
</body>
</html>
```

```jinja
{# templates/home.html #}
{% extends "base.html" %}
{% block title %}Home Page{% endblock %}
{% block content %}
    <h1>Welcome!</h1>
    <p>This is the home page.</p>
{% endblock %}
```

**Expected Output:**

```html
<html>
<head>
    <title>Home Page</title>
</head>
<body>
    <h1>Welcome!</h1>
    <p>This is the home page.</p>
</body>
</html>
```

**Why this output:** The `home.html` template extends `base.html` and overrides the `title` and `content` blocks. The base template's default content for those blocks is replaced.

**Example 2: Scoped Blocks**

```jinja
{# templates/base.html #}
<ul>
{% for item in items %}
    <li>{% block loop_item scoped %}{{ item }}{% endblock %}</li>
{% endfor %}
</ul>
```

**Expected Output (with `items = ['a', 'b', 'c']`):**

```html
<ul>
    <li>a</li>
    <li>b</li>
    <li>c</li>
</ul>
```

**Why this output:** The `scoped` modifier makes the loop variable `item` available inside the block. Without `scoped`, the block would output empty `<li>` items because `item` would be unavailable.

### Real-World Cases

- **Navigation menus:** Overriding the navigation block for different sections.
- **Sidebars:** Page-specific sidebar content.
- **Footer links:** Customizing footer content per page.
- **Meta tags:** Page-specific SEO meta tags in the head block.

### References

- Jinja2 Blocks — https://jinja.palletsprojects.com/en/stable/templates/#blocks
- Jinja2 Scoped Blocks — https://jinja.palletsprojects.com/en/stable/templates/#block-scoping

---

## 3. Child Templates

### Definitions

**Core Definition:** A child template is a template that extends a base template using the `{% extends %}` tag and overrides specific blocks to provide page-specific content.

**Technical Definition:** A child template begins with `{% extends "base.html" %}` as its first tag, which tells the template engine that this template extends another template. The child template then defines `{% block %}` tags that override the corresponding blocks in the parent. The `extends` tag must be the first tag in the template. If the child template does not override a block, the parent's default content is used.

**Beginner-Friendly Explanation:** A child template is a specific page that inherits from a base template. It starts with `{% extends "base.html" %}` and then fills in the blocks with its own content. The child template only needs to define what's different from the base template.

### Purposes

- To create specific pages that inherit the common layout.
- To override only the blocks that need customization.
- To reuse the base template's structure across many pages.
- To keep page-specific content in separate files.
- To support template organization by feature or module.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% extends "base.html" %}

{% block title %}Page Title{% endblock %}

{% block content %}
    Page-specific content
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% extends "base.html" %}` | Must be the first tag |
| `{% block name %}...{% endblock %}` | Overrides parent blocks |
| Unoverridden blocks | Use parent's default content |

**Syntax Rules:**

- `{% extends %}` must be the first tag in the child template.
- The child template can override any number of blocks.
- Blocks not overridden use the parent's default content.
- The child template cannot define content outside of blocks.
- The `{% extends %}` tag can be conditional (dynamic inheritance).

**Constraints and Limitations:**

- Jinja does not support multiple inheritance; only one `extends` tag may be executed per rendering.
- The child template cannot access the parent's variables directly except through `super()`.
- Blocks must have matching names to override the parent's blocks.

### Annotated Code Examples

**Example 1: Basic Child Template**

```jinja
{# templates/about.html #}
{% extends "base.html" %}

{% block title %}About Us{% endblock %}

{% block content %}
    <h1>About Us</h1>
    <p>We are a company that builds great things.</p>
{% endblock %}
```

**Expected Output:**
- The page inherits the base template's structure (head, nav, footer).
- The `title` block is replaced with "About Us".
- The `content` block is replaced with the about page content.

**Why this output:** The `{% extends "base.html" %}` tag tells Jinja to use `base.html` as the skeleton. The `{% block %}` tags replace the corresponding blocks in the base template.

**Example 2: Child Template with `super()`**

```jinja
{# templates/products.html #}
{% extends "base.html" %}

{% block head %}
    {{ super() }}
    <link rel="stylesheet" href="{{ url_for('static', filename='products.css') }}">
{% endblock %}

{% block content %}
    <h1>Products</h1>
    {% for product in products %}
        <div class="product">{{ product.name }}</div>
    {% endfor %}
{% endblock %}
```

**Expected Output:**
- The head block includes both the parent's default head content and the products.css stylesheet.
- The content block displays the products.

**Why this output:** `{{ super() }}` renders the parent block's content, then the child adds its own content. This allows incremental extension rather than complete replacement.

### Real-World Cases

- **Home page:** Overriding the content block with a welcome message.
- **About page:** Overriding the title and content blocks.
- **Product listing:** Adding a page-specific stylesheet via `super()`.
- **Contact form:** Overriding the content block with a form.

### References

- Jinja2 Child Templates — https://jinja.palletsprojects.com/en/stable/templates/#child-template
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/

---

## 4. Reusable Layouts

### Definitions

**Core Definition:** Reusable layouts are base templates designed to be extended by multiple child templates, providing a consistent structure that can be reused across an application.

**Technical Definition:** Reusable layouts are base templates that define common HTML structure and blocks for multiple pages. They are typically stored in a `templates/` directory and can be extended by any number of child templates. Blueprint templates can extend application-level layouts, with application templates taking priority in the search path. The `{% include %}` tag can be used alongside `{% extends %}` to include reusable components (like navigation bars) within blocks.

**Beginner-Friendly Explanation:** A reusable layout is a base template that many pages use. For example, you might have a `base.html` for the public site and an `admin_base.html` for the admin panel. Each layout provides a different structure, and pages extend the appropriate one.

### Purposes

- To create different layouts for different sections of an application.
- To reuse common structural elements across multiple pages.
- To support multiple blueprints with shared layouts.
- To combine inheritance with includes for maximum reuse.
- To maintain consistency while allowing section-specific customization.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Application-level layout #}
{# templates/base.html #}
<html>
<body>
    {% include "partials/nav.html" %}
    {% block content %}{% endblock %}
    {% include "partials/footer.html" %}
</body>
</html>
```

```jinja
{# Blueprint-level layout #}
{# blueprints/admin/templates/admin_base.html #}
{% extends "base.html" %}
{% block content %}
    <div class="admin-layout">
        {% block admin_content %}{% endblock %}
    </div>
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% include %}` | Includes another template in place |
| `{% extends %}` | Inherits from a parent template |
| Layout hierarchy | Application base → Blueprint base → Page |

**Syntax Rules:**

- Layouts can be organized hierarchically (application → blueprint → page).
- `{% include %}` is for reusable components; `{% extends %}` is for inheritance.
- Included templates are passed the current context.
- Blueprint templates have lower priority than application templates.

**Constraints and Limitations:**

- `{% include %}` renders the included template independently; it cannot override blocks.
- Included templates that contain `{% extends %}` can affect the current template.
- Template search order matters for blueprint templates.

### Annotated Code Examples

**Example 1: Application Layout with Includes**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head><title>{% block title %}My Site{% endblock %}</title></head>
<body>
    {% include "partials/header.html" %}
    <main>{% block content %}{% endblock %}</main>
    {% include "partials/footer.html" %}
</body>
</html>
```

```jinja
{# templates/partials/header.html #}
<header>
    <nav>
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('about') }}">About</a>
    </nav>
</header>
```

**Expected Output:**
- Every page includes the header and footer partials.
- The `content` block varies per page.

**Why this output:** The `{% include %}` tag inserts the header and footer partials into the base template. This is useful for components that are identical across all pages.

**Example 2: Blueprint Layout Extending Application Layout**

```jinja
{# app/templates/base.html #}
<html>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

```jinja
{# app/admin/templates/admin_base.html #}
{% extends "base.html" %}
{% block content %}
    <div class="admin-container">
        <h1>Admin Panel</h1>
        {% block admin_content %}{% endblock %}
    </div>
{% endblock %}
```

```jinja
{# app/admin/templates/dashboard.html #}
{% extends "admin_base.html" %}
{% block admin_content %}
    <p>Dashboard content</p>
{% endblock %}
```

**Expected Output:**
- The dashboard page inherits from `admin_base.html`, which inherits from `base.html`.
- The final page has the admin container and the dashboard content.

**Why this output:** Blueprint templates can extend application templates. The `admin_base.html` extends `base.html` and adds an admin-specific layout with its own `admin_content` block.

### Real-World Cases

- **Public site vs. admin panel:** Different layouts for different user types.
- **Multi-tenant SaaS:** Tenant-specific layouts extending a common base.
- **Documentation vs. blog:** Different sections with different layouts.
- **Email templates:** Reusable layout for transactional emails.

### References

- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Flask Blueprint Templates — https://flask.palletsprojects.com/en/stable/blueprints/#templates
- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include

---

## 5. Nested Inheritance

### Definitions

**Core Definition:** Nested inheritance is a multi-level template hierarchy where a child template extends a base template that itself extends another base template, creating a chain of inheritance.

**Technical Definition:** Jinja2 supports multiple levels of template inheritance. A template can extend another template, which in turn extends a third. The inheritance chain is resolved at render time, with each level overriding or extending blocks from the level above. Blocks can be nested for more complex layouts. The `super()` function can be called at multiple levels to render content from the parent block.

**Beginner-Friendly Explanation:** Nested inheritance means you can have a "grandparent" template, a "parent" template that extends it, and a "child" template that extends the parent. For example, a base layout for the whole site, an admin layout that extends it, and a specific admin page that extends the admin layout.

### Purposes

- To create increasingly specialized layouts for different sections.
- To share common structure across multiple levels of an application.
- To allow section-specific layouts to build on a common base.
- To organize large applications with multiple template hierarchies.
- To support complex applications with multiple layout tiers.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{# Level 1: Application base #}
{# templates/base.html #}
<html>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

```jinja
{# Level 2: Section base #}
{# templates/admin_base.html #}
{% extends "base.html" %}
{% block content %}
    <div class="admin">
        {% block admin_content %}{% endblock %}
    </div>
{% endblock %}
```

```jinja
{# Level 3: Specific page #}
{# templates/admin_dashboard.html #}
{% extends "admin_base.html" %}
{% block admin_content %}
    <h1>Dashboard</h1>
{% endblock %}
```

**Component Breakdown:**

| Level | Template | Extends | Overrides |
|-------|----------|---------|-----------|
| 1 | `base.html` | — | Defines `content` block |
| 2 | `admin_base.html` | `base.html` | Overrides `content`, defines `admin_content` |
| 3 | `admin_dashboard.html` | `admin_base.html` | Overrides `admin_content` |

**Syntax Rules:**

- Each level can override any block from the level above.
- New blocks can be defined at any level.
- `super()` can be called at any level to render the parent's block content.
- Only one `{% extends %}` tag may be executed per rendering.

**Constraints and Limitations:**

- Deep inheritance chains can be hard to debug.
- Each level must be a valid template file.
- Block names must be consistent across levels for overrides to work.

### Annotated Code Examples

**Example 1: Three-Level Inheritance**

```jinja
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head><title>{% block title %}Site{% endblock %}</title></head>
<body>
    <header>{% block header %}Default Header{% endblock %}</header>
    <main>{% block content %}{% endblock %}</main>
    <footer>{% block footer %}© 2024{% endblock %}</footer>
</body>
</html>
```

```jinja
{# templates/shop_base.html #}
{% extends "base.html" %}
{% block header %}
    {{ super() }}
    <nav>Shop Navigation</nav>
{% endblock %}
{% block content %}
    <div class="shop">{% block shop_content %}{% endblock %}</div>
{% endblock %}
```

```jinja
{# templates/product.html #}
{% extends "shop_base.html" %}
{% block title %}Product Page{% endblock %}
{% block shop_content %}
    <h1>{{ product.name }}</h1>
    <p>{{ product.price }}</p>
{% endblock %}
```

**Expected Output:**

```html
<!DOCTYPE html>
<html>
<head><title>Product Page</title></head>
<body>
    <header>Default Header<nav>Shop Navigation</nav></header>
    <main><div class="shop"><h1>Widget</h1><p>$9.99</p></div></main>
    <footer>© 2024</footer>
</body>
</html>
```

**Why this output:** The `product.html` extends `shop_base.html`, which extends `base.html`. The `header` block in `shop_base.html` calls `{{ super() }}` to include the parent's default header, then adds the shop navigation. The `shop_content` block is overridden by the product page.

### Real-World Cases

- **E-commerce:** Base layout → Shop layout → Product page.
- **Admin panels:** Base layout → Admin layout → User management page.
- **Documentation:** Base layout → API docs layout → Endpoint page.
- **Multi-theme apps:** Base layout → Theme layout → Page.

### References

- Jinja2 Nested Blocks — https://jinja.palletsprojects.com/en/stable/templates/#nested-blocks
- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance

---

## 6. `super()` Block Execution

### Definitions

**Core Definition:** `super()` is a Jinja2 function that, when called inside a block in a child template, renders the contents of the parent template's block, allowing incremental extension rather than complete replacement.

**Technical Definition:** `{{ super() }}` renders the parent block's content. It is useful when a child template wants to add to the parent's content rather than completely replace it. The `super()` function can be called at any level of an inheritance chain. In Jinja2, `super()` returns a `Markup` object representing the rendered parent block. If the child block is empty, `super()` still renders the parent content.

**Beginner-Friendly Explanation:** `super()` lets you say "keep what the parent template has, and add this on top." For example, if the base template includes a default stylesheet, you can use `super()` to keep that stylesheet and add your own.

### Purposes

- To add content to a parent block without completely replacing it.
- To extend the parent's head section with page-specific styles or scripts.
- To include the parent's navigation and add additional links.
- To maintain the parent's default content while customizing.
- To build on existing layouts without duplicating code.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% block block_name %}
    {{ super() }}
    <!-- Additional content -->
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{{ super() }}` | Renders the parent block's content |
| Additional content | Content added after the parent's content |

**Syntax Rules:**

- `{{ super() }}` can be called at any position within a block.
- The parent's content is rendered exactly where `{{ super() }}` is placed.
- `super()` can be called multiple times, though this is unusual.
- `super()` works across multiple levels of inheritance.

**Constraints and Limitations:**

- `super()` cannot be called outside of a block.
- `super()` returns the parent's rendered content as a `Markup` object.
- Calling `super()` in a block that has no parent (base template) returns an empty string.

### Annotated Code Examples

**Example 1: Extending the Head Block**

```jinja
{# templates/base.html #}
<head>
    {% block head %}
        <link rel="stylesheet" href="{{ url_for('static', filename='base.css') }}">
    {% endblock %}
</head>
```

```jinja
{# templates/products.html #}
{% extends "base.html" %}
{% block head %}
    {{ super() }}
    <link rel="stylesheet" href="{{ url_for('static', filename='products.css') }}">
{% endblock %}
```

**Expected Output:**

```html
<head>
    <link rel="stylesheet" href="/static/base.css">
    <link rel="stylesheet" href="/static/products.css">
</head>
```

**Why this output:** `{{ super() }}` renders the parent's `head` block, which includes `base.css`. Then the child adds `products.css`. Both stylesheets are included.

**Example 2: Extending Navigation**

```jinja
{# templates/base.html #}
<nav>
    {% block nav %}
        <a href="/">Home</a>
        <a href="/about">About</a>
    {% endblock %}
</nav>
```

```jinja
{# templates/admin.html #}
{% extends "base.html" %}
{% block nav %}
    {{ super() }}
    <a href="/admin">Admin</a>
{% endblock %}
```

**Expected Output:**

```html
<nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/admin">Admin</a>
</nav>
```

**Why this output:** `{{ super() }}` renders the parent's navigation links, then the child adds the admin link. The result is the complete navigation with all links.

**Example 3: Multiple Levels of `super()`**

```jinja
{# base.html #}
{% block content %}Base content{% endblock %}
```

```jinja
{# level2.html #}
{% extends "base.html" %}
{% block content %}{{ super() }} | Level 2{% endblock %}
```

```jinja
{# level3.html #}
{% extends "level2.html" %}
{% block content %}{{ super() }} | Level 3{% endblock %}
```

**Expected Output:**

```
Base content | Level 2 | Level 3
```

**Why this output:** Each level calls `super()` to include the parent's content, then appends its own. The result is a chain of content from all levels.

### Real-World Cases

- **Adding page-specific CSS/JS:** Using `super()` in the head block.
- **Extending navigation:** Adding context-specific links to the main nav.
- **Adding meta tags:** Including page-specific SEO meta tags.
- **Breadcrumbs:** Adding breadcrumb navigation to a base breadcrumb block.

### References

- Jinja2 Super Blocks — https://jinja.palletsprojects.com/en/stable/templates/#super-blocks
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/

---

## 7. Dynamic Inheritance (Conditional `extends`)

### Definitions

**Core Definition:** Dynamic inheritance is a Jinja2 feature that allows the `{% extends %}` tag to be placed inside an `{% if %}` block, enabling a template to conditionally inherit from different parent templates.

**Technical Definition:** Jinja supports dynamic inheritance and does not distinguish between parent and child template as long as no `extends` tag is visited. It is possible to put the `extends` tag into an `if` tag to only extend from the layout template if a variable (e.g., `standalone`) evaluates to false, which it does by default if it's not defined. This allows the same template to be rendered either as a standalone page or as part of a larger layout.

**Beginner-Friendly Explanation:** Dynamic inheritance lets a template decide which base template to extend based on a condition. For example, the same template could extend a full layout when rendered as a normal page, or a minimal layout when rendered as a standalone widget.

### Purposes

- To support multi-theme applications where the base template depends on a theme variable.
- To allow the same template to be rendered with different layouts.
- To implement a "null-default fallback" pattern (standalone rendering).
- To support AJAX/partial rendering where the layout differs.
- To enable theme switching without duplicating templates.

### Syntax Rules and Structure

**Complete General Syntax:**

```jinja
{% if not standalone %}{% extends 'default.html' %}{% endif -%}
<!DOCTYPE html>
<title>{% block title %}The Page Title{% endblock %}</title>
<link rel="stylesheet" href="style.css" type="text/css">
{% block body %}
    <p>This is the page body.</p>
{% endblock %}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{% if not standalone %}` | Condition for extending |
| `{% extends 'default.html' %}` | Conditional extend |
| `{% endif -%}` | Whitespace control after condition |
| Fallback skeleton | Rendered when `standalone` is True |

**Syntax Rules:**

- The `{% extends %}` tag can be placed inside an `{% if %}` block.
- Only one `{% extends %}` tag may be executed per rendering.
- Everything before the first `extends` tag (including whitespace) is printed out instead of being ignored.
- The `standalone` variable defaults to undefined (falsy) if not provided.

**Constraints and Limitations:**

- Only one `extends` tag may be executed at a time.
- The behavior before the `extends` tag is "surprising" — whitespace is printed.
- Dynamic inheritance can make templates harder to understand.

### Annotated Code Examples

**Example 1: Null-Default Fallback**

```jinja
{# templates/page.html #}
{% if not standalone %}{% extends 'base.html' %}{% endif -%}
<!DOCTYPE html>
<title>{% block title %}The Page Title{% endblock %}</title>
<link rel="stylesheet" href="style.css" type="text/css">
{% block body %}
    <p>This is the page body.</p>
{% endblock %}
```

**Expected Output (when `standalone` is True):**

```html
<!DOCTYPE html>
<title>The Page Title</title>
<link rel="stylesheet" href="style.css" type="text/css">
<p>This is the page body.</p>
```

**Expected Output (when `standalone` is False or undefined):**

```html
<!-- Rendered through base.html -->
```

**Why this output:** When `standalone` is True, the `extends` tag is skipped, and the fallback skeleton is rendered. When `standalone` is False or undefined, the template extends `base.html` and the fallback skeleton is ignored.

**Example 2: Multi-Theme Inheritance**

```jinja
{# templates/page.html #}
{% if theme == 'dark' %}
    {% extends 'dark_base.html' %}
{% elif theme == 'light' %}
    {% extends 'light_base.html' %}
{% else %}
    {% extends 'default_base.html' %}
{% endif %}

{% block content %}
    <h1>Page Content</h1>
{% endblock %}
```

**Expected Output (when `theme = 'dark'`):**
- The page extends `dark_base.html` and uses the dark theme layout.

**Why this output:** The `extends` tag is inside an `if/elif/else` block, allowing the template to inherit from different base templates based on the `theme` variable.

### Real-World Cases

- **Multi-theme applications:** Switching between dark and light themes.
- **AJAX partial rendering:** Rendering the same template with or without the full layout.
- **Email templates:** Using a minimal layout for email clients.
- **PDF generation:** Using a print-specific layout.

### References

- Jinja2 Null-Default Fallback — https://jinja.palletsprojects.com/en/stable/tricks/#null-default-fallback
- Jinja2 Dynamic Inheritance — https://jinja.palletsprojects.com/en/stable/tricks/

---

## References

- Jinja2 Template Inheritance — https://jinja.palletsprojects.com/en/stable/templates/#template-inheritance
- Jinja2 Blocks — https://jinja.palletsprojects.com/en/stable/templates/#blocks
- Jinja2 Super Blocks — https://jinja.palletsprojects.com/en/stable/templates/#super-blocks
- Jinja2 Null-Default Fallback — https://jinja.palletsprojects.com/en/stable/tricks/#null-default-fallback
- Jinja2 Tips and Tricks — https://jinja.palletsprojects.com/en/stable/tricks/
- Flask Template Inheritance — https://flask.palletsprojects.com/en/stable/patterns/templateinheritance/
- Flask Blueprint Templates — https://flask.palletsprojects.com/en/stable/blueprints/#templates
- Jinja2 Include — https://jinja.palletsprojects.com/en/stable/templates/#include
- Jinja2 Scoped Blocks — https://jinja.palletsprojects.com/en/stable/templates/#block-scoping
- Jinja2 Nested Blocks — https://jinja.palletsprojects.com/en/stable/templates/#nested-blocks