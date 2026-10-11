# Laravel Blade Layouts: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Blade Layouts are master templates that define the shared structure of multiple web pages—such as headers, footers, navigation, and sidebars—which child views inherit and populate with page-specific content through a system called template inheritance.

**Technical Definition**

Blade layouts implement a template inheritance system where a master layout defines named "sections" using `@yield` placeholders and `@section` blocks. Child views use the `@extends` directive to declare which layout they inherit, then inject content into the layout's sections using `@section` directives. The Blade compiler resolves this inheritance at compile time, producing a single PHP file that renders the complete page. Additional directives such as `@parent`, `@show`, `@push`, and `@stack` provide mechanisms for section extension, immediate rendering, and asset collection.

**Beginner-Friendly Explanation**

Imagine you are building a website with 50 pages. Every page needs the same header, footer, and sidebar. Instead of copying and pasting that code 50 times, you create one "master template" that contains all the common parts. Then, for each page, you just write the unique content and tell Laravel "use the master template." Laravel automatically combines the common parts with your unique content. This is called template inheritance, and it saves you from repeating yourself.

---

### Key Characteristics

- **Template Inheritance**: A master layout defines the common structure; child views extend it and provide page-specific content.
- **Section-Based Content Injection**: Named sections allow child views to inject content into specific areas of the layout.
- **Compile-Time Resolution**: Inheritance is resolved during compilation, producing a single optimized PHP file.
- **Zero Runtime Overhead**: The compiled output is plain PHP with no templating engine overhead.
- **Layered Content**: The `@parent` directive allows child sections to append to (rather than overwrite) parent content.
- **Asset Management**: The `@push` and `@stack` directives collect scripts and styles from multiple views into designated locations in the layout.
- **Component Alternative**: Modern Laravel offers a component-based layout approach using `<x-layout>` tags as an alternative to template inheritance.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic knowledge of HTML and PHP**.
- **Familiarity with Blade templates and the `.blade.php` extension**.
- **Understanding of the `resources/views` directory structure**.

---

### Related Programming Areas

- **Template Inheritance**: A design pattern shared by many templating engines (Twig, Jinja2, Django Templates).
- **MVC Architecture**: Blade layouts serve as the "View" layer in Laravel's MVC pattern.
- **Component-Based UI**: The newer component approach offers an alternative mental model to template inheritance.
- **Asset Pipeline**: The `@push`/`@stack` system complements Vite and Laravel Mix for asset management.
- **DRY Principle**: Layouts directly implement the "Don't Repeat Yourself" principle in view code.

---

## Core Concepts

---

### 1. Layouts

#### Definitions

**Core Definition**

A Blade layout is a master template file that defines the shared HTML structure (header, footer, navigation, sidebars) used across multiple pages of an application.

**Technical Definition**

A layout is a `.blade.php` file typically stored in `resources/views/layouts/` that contains `@yield` directives as placeholders for dynamic content. Child views extend the layout via `@extends` and fill the placeholders with `@section` directives. The layout may also contain `@section ... @show` blocks that define default content which child views can override or extend.

**Beginner-Friendly Explanation**

A layout is like a picture frame. You build one frame with a header, footer, and sidebar, and then you put different pictures (page content) inside it for each page. You don't have to rebuild the frame every time—you just tell Laravel "use this frame and put my content here."

#### Purposes

- To eliminate duplication of common page elements (header, footer, navigation) across multiple views.
- To provide a consistent structure and appearance for all pages in an application.
- To separate the definition of page structure from the definition of page content.
- To make it easy to update the overall layout without touching individual pages.
- To reduce maintenance burden by centralizing shared markup in one file.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<!-- resources/views/layouts/app.blade.php -->
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'Default Title')</title>
</head>
<body>
    <header>
        @include('partials.nav')
    </header>

    <main>
        @yield('content')
    </main>

    <footer>
        @include('partials.footer')
    </footer>
</body>
</html>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@yield('title', 'Default')` | Placeholder for a named section with a default value. |
| `@yield('content')` | Placeholder for the main content section. |
| `@include('partials.nav')` | Includes a reusable partial view. |
| `@section('sidebar') ... @show` | Defines and immediately renders a default section. |

**Syntax Rules**

- Layout files should be stored in `resources/views/layouts/` by convention.
- `@yield` accepts a section name as the first argument and an optional default value as the second.
- `@section ... @show` defines a section and renders it immediately with default content.
- Layouts can include partials via `@include` for further modularity.

**Constraints and Limitations**

- A layout can define any number of `@yield` and `@section` blocks.
- Section names must be unique within a layout.
- Layouts cannot extend other layouts directly (no multi-level inheritance in the traditional sense).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Layout with Multiple Sections**

**Setup Guide**

1. Create `resources/views/layouts/app.blade.php`.
2. Create a child view that extends it.

**Layout File**

```blade
<!-- resources/views/layouts/app.blade.php -->
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'My Application')</title>
</head>
<body>
    <nav>
        <a href="/">Home</a> | <a href="/about">About</a>
    </nav>

    <div class="container">
        @yield('content')
    </div>

    <footer>
        <p>&copy; 2026 My Application</p>
    </footer>
</body>
</html>
```

**Child View**

```blade
<!-- resources/views/home.blade.php -->
@extends('layouts.app')

@section('title', 'Home Page')

@section('content')
    <h1>Welcome!</h1>
    <p>This is the home page content.</p>
@endsection
```

**Route**

```php
Route::get('/', function () {
    return view('home');
});
```

**Expected Output**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Home Page</title>
</head>
<body>
    <nav>
        <a href="/">Home</a> | <a href="/about">About</a>
    </nav>

    <div class="container">
        <h1>Welcome!</h1>
        <p>This is the home page content.</p>
    </div>

    <footer>
        <p>&copy; 2026 My Application</p>
    </footer>
</body>
</html>
```

**Why This Output Occurs**

The child view extends `layouts.app` and provides content for the `title` and `content` sections. The `@yield('title', 'My Application')` uses the value "Home Page" from the child because the child defines a `title` section. The `@yield('content')` is replaced with the child's content section.

---

**Example 2: Layout with Default Section Content**

**Layout File**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    @section('styles')
        <link rel="stylesheet" href="/css/base.css">
    @show
</head>
<body>
    @section('sidebar')
        <p>Default sidebar content</p>
    @show

    <div class="content">
        @yield('content')
    </div>
</body>
</html>
```

**Child View (without sidebar override)**

```blade
@extends('layouts.app')

@section('content')
    <h1>Page Content</h1>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <link rel="stylesheet" href="/css/base.css">
</head>
<body>
    <p>Default sidebar content</p>

    <div class="content">
        <h1>Page Content</h1>
    </div>
</body>
</html>
```

**Why This Output Occurs**

The `@section('sidebar') ... @show` block in the layout defines default content. Since the child view does not override the `sidebar` section, the default content is rendered. This contrasts with `@yield`, where no default content is available for extension.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Site Layout**

An e-commerce application uses a main layout with a header (logo, cart, search bar) and footer (links, newsletter signup). Product pages, cart pages, and checkout pages all extend this layout and fill the `content` section with their specific markup.

**Explanation**: Template inheritance ensures that the header and footer are consistent across all pages and need only be updated in one place.

**Case 2: Admin Dashboard Layout**

An admin layout includes a sidebar navigation and a top bar with user info. Each admin page extends this layout and provides its content in the main area. The active navigation item is highlighted based on the current route.

**Explanation**: Layouts provide a consistent admin experience while allowing each page to focus on its unique content.

---

### 2. Template Inheritance

#### Definitions

**Core Definition**

Template inheritance is the mechanism by which a child view extends a parent layout, inheriting its structure and filling in content through named sections.

**Technical Definition**

Template inheritance in Blade is implemented through a combination of directives: `@extends` (declares the parent layout), `@section` (defines content blocks), and `@yield` (outputs section content). The Blade compiler processes these directives and produces a single PHP file that renders the complete page. The inheritance chain is resolved at compile time, and the resulting PHP file is cached.

**Beginner-Friendly Explanation**

Template inheritance is like a family tree for your web pages. The parent layout is the "ancestor" that contains all the common features. Child pages "inherit" from this ancestor and can add their own unique features. Laravel automatically combines the parent's features with the child's additions.

#### Purposes

- To provide a structured, hierarchical approach to building web pages.
- To enable code reuse without duplication of common page elements.
- To separate the definition of page structure (layout) from page content (child view).
- To support the DRY (Don't Repeat Yourself) principle in view code.
- To facilitate consistent updates to the overall site structure.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Parent Layout --}}
<html>
<body>
    @yield('content')
</body>
</html>

{{-- Child View --}}
@extends('layouts.parent')

@section('content')
    <p>Child content</p>
@endsection
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@extends('layout')` | Declares the parent layout to inherit. |
| `@section('name')` | Defines a content block. |
| `@yield('name')` | Outputs a content block. |
| `@endsection` | Closes a section definition. |

**Syntax Rules**

- `@extends` must be the first Blade directive in the child view (comments are allowed before it).
- Section names must match between the layout and the child view.
- A child view can only extend one layout.
- Sections can be defined in any order in the child view; the layout determines the output order.

**Constraints and Limitations**

- `@extends` cannot be used conditionally.
- Multi-level inheritance (a layout extending another layout) is not directly supported in the traditional template inheritance system.
- Sections defined in the child but not present in the layout are ignored.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Two-Level Inheritance**

**Layout**

```blade
<!-- resources/views/layouts/master.blade.php -->
<html>
<head>
    <title>@yield('title')</title>
</head>
<body>
    @yield('content')
</body>
</html>
```

**Child View**

```blade
<!-- resources/views/page.blade.php -->
@extends('layouts.master')

@section('title', 'Page Title')

@section('content')
    <h1>Page Content</h1>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <title>Page Title</title>
</head>
<body>
    <h1>Page Content</h1>
</body>
</html>
```

**Why This Output Occurs**

The child view extends `layouts.master` and provides content for the `title` and `content` sections. The `@yield` directives in the layout are replaced with the corresponding section content.

---

#### Real-World Cases with Explanation

**Case 1: Blog with Multiple Layouts**

A blog application has two layouts: one for the main site (with a sidebar) and one for full-width pages (without a sidebar). Article pages extend the main layout, while landing pages extend the full-width layout.

**Explanation**: Multiple layouts allow different page types to have appropriate structures while sharing common elements like headers and footers.

---

### 3. `@extends`

#### Definitions

**Core Definition**

The `@extends` directive declares that a child view inherits from a specified parent layout.

**Technical Definition**

`@extends('layout.name')` is compiled by the Blade compiler to register the parent layout for the current view. It must be the first directive in the child view. The compiler resolves the layout path and ensures that the child's sections are injected into the parent's `@yield` and `@section` placeholders during rendering.

**Beginner-Friendly Explanation**

`@extends` is how you tell Laravel "this page uses the master template." You put it at the very top of your view file, and Laravel knows to wrap your content in the layout you specified.

#### Purposes

- To declare the parent layout that a child view inherits from.
- To establish the inheritance relationship between a layout and its child views.
- To enable the child view to inject content into the layout's sections.
- To provide a clear, readable declaration of a view's layout dependency.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@extends('layouts.app')
@extends('layouts.admin')
@extends('vendor.package::layouts.custom')
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@extends` | The directive keyword. |
| `'layouts.app'` | The dot-notation path to the layout view (resolves to `resources/views/layouts/app.blade.php`). |

**Syntax Rules**

- `@extends` must be the first Blade directive in the child view.
- The layout name uses dot notation corresponding to the directory structure.
- Comments (`{{-- --}}`) may appear before `@extends`.
- A view can only extend one layout.

**Constraints and Limitations**

- Cannot be used conditionally (e.g., inside an `@if` block).
- The specified layout must exist; otherwise, a view not found exception is thrown.
- `@extends` does not support passing data to the layout; data is passed through the view's data array.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic `@extends` Usage**

**Layout File**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<body>
    @yield('content')
</body>
</html>
```

**Child View**

```blade
<!-- resources/views/dashboard.blade.php -->
@extends('layouts.app')

@section('content')
    <h1>Dashboard</h1>
@endsection
```

**Route**

```php
Route::get('/dashboard', function () {
    return view('dashboard');
});
```

**Expected Output**

```html
<html>
<body>
    <h1>Dashboard</h1>
</body>
</html>
```

**Why This Output Occurs**

The `@extends('layouts.app')` directive tells the Blade compiler to use `layouts/app.blade.php` as the parent template. The child's `content` section is injected into the `@yield('content')` placeholder.

---

**Example 2: `@extends` with Nested Layout Paths**

**Layout File**

```blade
<!-- resources/views/admin/layouts/master.blade.php -->
<html>
<body>
    @yield('content')
</body>
</html>
```

**Child View**

```blade
<!-- resources/views/admin/users.blade.php -->
@extends('admin.layouts.master')

@section('content')
    <h1>User Management</h1>
@endsection
```

**Expected Output**

```html
<html>
<body>
    <h1>User Management</h1>
</body>
</html>
```

**Why This Output Occurs**

The dot notation `admin.layouts.master` resolves to `resources/views/admin/layouts/master.blade.php`. The child's content is injected into the layout.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Section Application**

A SaaS application has a public-facing site and an admin panel. Public pages extend `layouts.public`, while admin pages extend `layouts.admin`. Each layout has different navigation and styling.

**Explanation**: `@extends` allows different sections of the application to use different layouts while sharing the same Blade inheritance mechanism.

---

### 4. `@section`

#### Definitions

**Core Definition**

The `@section` directive defines a named block of content within a child view or layout that can be injected into a corresponding `@yield` or `@section` placeholder in the parent layout.

**Technical Definition**

`@section('name')` begins a content block, which is terminated by `@endsection` or `@stop`. The content is stored in the view's section stack and later rendered by the parent layout's `@yield` directive. In a layout, `@section ... @show` defines default content that child views can override or extend using `@parent`.

**Beginner-Friendly Explanation**

`@section` is how you say "here is the content for a specific part of the layout." For example, `@section('title', 'Home Page')` sets the page title, and `@section('content') ... @endsection` provides the main body content.

#### Purposes

- To define a named block of content for injection into a layout.
- To provide content for a specific `@yield` placeholder.
- To define default content in a layout that can be overridden or extended.
- To support inline section definitions with a single value.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Standard section --}}
@section('content')
    <p>Content here</p>
@endsection

{{-- Inline section --}}
@section('title', 'Page Title')

{{-- Section with @show (renders immediately) --}}
@section('sidebar')
    <p>Sidebar content</p>
@show

{{-- Section with @parent (extends parent content) --}}
@section('sidebar')
    @parent
    <p>Additional sidebar content</p>
@endsection
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@section('name')` | Begins a content section. |
| `@endsection` | Ends a section (defines only). |
| `@stop` | Alias for `@endsection`. |
| `@show` | Defines and immediately renders the section. |
| `@parent` | Includes the parent's section content. |

**Syntax Rules**

- Section names must be unique within a view.
- `@section` can be used with `@endsection`, `@stop`, or `@show` as terminators.
- Inline syntax: `@section('name', 'value')`.
- `@parent` must be inside a `@section` in a child view.
- `@show` is used in layouts to define and immediately render default content.

**Constraints and Limitations**

- Sections cannot be nested within other sections.
- The order of `@section` definitions in a child view does not affect the output order (determined by the layout).
- `@section` cannot be used outside of a view that extends a layout (except in layouts themselves).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multiple Sections in a Child View**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    <title>@yield('title')</title>
</head>
<body>
    <header>@yield('header')</header>
    <main>@yield('content')</main>
    <footer>@yield('footer')</footer>
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@section('title', 'My Page')

@section('header')
    <h1>Page Header</h1>
@endsection

@section('content')
    <p>Main content here.</p>
@endsection

@section('footer')
    <p>Footer content.</p>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <title>My Page</title>
</head>
<body>
    <header><h1>Page Header</h1></header>
    <main><p>Main content here.</p></main>
    <footer><p>Footer content.</p></footer>
</body>
</html>
```

**Why This Output Occurs**

Each `@section` in the child view provides content for the corresponding `@yield` in the layout. The inline `@section('title', 'My Page')` sets the title directly.

---

**Example 2: `@section` with `@show` in Layout**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    @section('styles')
        <link rel="stylesheet" href="/css/app.css">
    @show
</head>
<body>
    @yield('content')
</body>
</html>
```

**Child View (no styles override)**

```blade
@extends('layouts.app')

@section('content')
    <p>Content</p>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <link rel="stylesheet" href="/css/app.css">
</head>
<body>
    <p>Content</p>
</body>
</html>
```

**Why This Output Occurs**

The `@section('styles') ... @show` in the layout defines default content and renders it immediately. Since the child does not override the `styles` section, the default CSS link is output.

---

#### Real-World Cases with Explanation

**Case 1: Page-Specific Styles**

A layout defines a `styles` section with default CSS. Individual pages override this section to add their own stylesheets while using `@parent` to retain the base styles.

**Explanation**: `@section` with `@show` provides default content that child views can extend.

---

### 5. `@yield`

#### Definitions

**Core Definition**

The `@yield` directive is a placeholder in a layout that outputs the content of a named section defined in a child view.

**Technical Definition**

`@yield('section')` is compiled to PHP code that retrieves the content of the specified section from the view's section stack and echoes it. An optional second argument provides a default value if the section is not defined. Unlike `@section ... @show`, `@yield` does not define default content that can be extended with `@parent`.

**Beginner-Friendly Explanation**

`@yield` is like an empty picture frame in your layout. When a child page provides content for that frame (using `@section`), it appears there. If the child doesn't provide anything, you can specify what shows up by default.

#### Purposes

- To output the content of a named section in a layout.
- To provide a placeholder for dynamic content in a master template.
- To support optional default values when a section is not defined.
- To enable clean separation between layout structure and page content.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@yield('content')
@yield('title', 'Default Title')
@yield('sidebar', View::make('partials.default-sidebar'))
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@yield('content')` | Outputs the `content` section; empty if undefined. |
| `@yield('title', 'Default')` | Outputs the `title` section or "Default". |
| `@yield('section', $default)` | Second argument can be any PHP expression. |

**Syntax Rules**

- `@yield` is used in layouts, not in child views.
- The first argument is the section name.
- The optional second argument is the default value.
- Default values can be strings, View instances, or any PHP expression.

**Constraints and Limitations**

- `@yield` cannot be extended with `@parent` (unlike `@section ... @show`).
- If a section is not defined and no default is provided, nothing is output.
- `@yield` can be used multiple times for the same section (content is duplicated).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: `@yield` with Default Value**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    <title>@yield('title', 'My Application')</title>
</head>
<body>
    @yield('content', '<p>Default content</p>')
</body>
</html>
```

**Child View (no title or content override)**

```blade
@extends('layouts.app')
```

**Expected Output**

```html
<html>
<head>
    <title>My Application</title>
</head>
<body>
    <p>Default content</p>
</body>
</html>
```

**Why This Output Occurs**

Since the child view does not define `title` or `content` sections, the default values provided in the `@yield` directives are used.

---

**Example 2: `@yield` with Dynamic Default**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<body>
    @yield('sidebar', view('partials.default-sidebar')->render())
</body>
</html>
```

**Expected Output**

The default sidebar partial is rendered if the child does not define a `sidebar` section.

**Why This Output Occurs**

The second argument to `@yield` can be any expression, including a rendered view. This allows for dynamic default content.

---

#### Real-World Cases with Explanation

**Case 1: Optional Page Title**

A layout uses `@yield('title', 'Default Page Title')`. Pages that define a `title` section override the default; pages that don't use the default.

**Explanation**: `@yield` with defaults provides a clean fallback for optional content.

---

### 6. `@parent`

#### Definitions

**Core Definition**

The `@parent` directive, when used inside a `@section` in a child view, includes the content of the same-named section from the parent layout.

**Technical Definition**

`@parent` is compiled to PHP code that retrieves and echoes the parent's section content at that point in the child's section. It enables child views to append to (rather than completely overwrite) the default content defined in the layout's `@section ... @show` blocks.

**Beginner-Friendly Explanation**

`@parent` is like saying "add my content after the parent's content." If the layout already has some default sidebar content, `@parent` lets you keep that content and add more to it.

#### Purposes

- To append content to a parent layout's section rather than overwriting it.
- To retain default content while adding page-specific additions.
- To enable layered composition of section content across inheritance levels.
- To support incremental customization of layout sections.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@section('sidebar')
    @parent
    <p>Additional sidebar content</p>
@endsection
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@parent` | Placeholder for the parent's section content. |
| `@section('sidebar')` | The child's override of the `sidebar` section. |

**Syntax Rules**

- `@parent` must be inside a `@section` in a child view.
- It can appear anywhere within the section (beginning, middle, or end).
- The parent layout must define the section with `@section ... @show` (not `@yield`).
- `@parent` is replaced by the parent's section content at render time.

**Constraints and Limitations**

- `@parent` only works with sections defined via `@section ... @show` in the layout, not with `@yield`.
- If the parent section is not defined, `@parent` produces no output.
- Multiple `@parent` calls in the same section duplicate the parent content.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Appending to Parent Sidebar**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<body>
    @section('sidebar')
        <p>Default sidebar content</p>
    @show
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@section('sidebar')
    @parent
    <p>Page-specific sidebar content</p>
@endsection
```

**Expected Output**

```html
<html>
<body>
    <p>Default sidebar content</p>
    <p>Page-specific sidebar content</p>
</body>
</html>
```

**Why This Output Occurs**

The `@parent` directive includes the layout's default sidebar content, and the child's additional content is appended after it.

---

**Example 2: Prepending to Parent Content**

**Child View**

```blade
@extends('layouts.app')

@section('sidebar')
    <p>Page-specific content (before)</p>
    @parent
@endsection
```

**Expected Output**

```html
<html>
<body>
    <p>Page-specific content (before)</p>
    <p>Default sidebar content</p>
</body>
</html>
```

**Why This Output Occurs**

The child's content is placed before `@parent`, so it appears before the parent's default content.

---

#### Real-World Cases with Explanation

**Case 1: Adding Page-Specific Scripts**

A layout defines a `scripts` section with common JavaScript. Individual pages use `@parent` to include the common scripts and add their own page-specific scripts.

**Explanation**: `@parent` allows incremental customization of sections.

---

### 7. `@show`

#### Definitions

**Core Definition**

The `@show` directive, used in a layout, defines a section and immediately renders its default content.

**Technical Definition**

`@section('name') ... @show` is compiled to code that both defines the section (making its content available for extension via `@parent`) and immediately outputs it. This contrasts with `@section ... @endsection`, which only defines the section without rendering it.

**Beginner-Friendly Explanation**

`@show` is like saying "here is the default content for this section, and also display it right here." It's used in layouts to provide default content that child pages can add to.

#### Purposes

- To define a section with default content in a layout.
- To immediately render the section content at its location in the layout.
- To enable child views to extend the default content using `@parent`.
- To provide a fallback when a child view does not override the section.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@section('sidebar')
    <p>Default sidebar content</p>
@show
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@section('sidebar')` | Begins the section definition. |
| `@show` | Ends the section, defines it, and renders it immediately. |

**Syntax Rules**

- `@show` is used in layouts, not in child views.
- It replaces `@endsection` or `@stop` in the layout's section.
- The section content is both defined (for `@parent` access) and rendered.
- `@show` can be used with any section name.

**Constraints and Limitations**

- `@show` cannot be used in child views (only in layouts).
- Sections defined with `@show` cannot be left empty without outputting nothing.
- The `@show` directive is specific to layouts and is not interchangeable with `@yield`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Layout with `@show` Section**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<body>
    @section('sidebar')
        <p>Default sidebar</p>
    @show

    <div class="content">
        @yield('content')
    </div>
</body>
</html>
```

**Child View (no sidebar override)**

```blade
@extends('layouts.app')

@section('content')
    <p>Main content</p>
@endsection
```

**Expected Output**

```html
<html>
<body>
    <p>Default sidebar</p>

    <div class="content">
        <p>Main content</p>
    </div>
</body>
</html>
```

**Why This Output Occurs**

The `@section('sidebar') ... @show` defines and immediately renders the default sidebar content. Since the child does not override it, the default appears.

---

**Example 2: `@show` with `@parent` in Child**

**Child View**

```blade
@extends('layouts.app')

@section('sidebar')
    @parent
    <p>Extra sidebar</p>
@endsection
```

**Expected Output**

```html
<html>
<body>
    <p>Default sidebar</p>
    <p>Extra sidebar</p>

    <div class="content">
        <p>Main content</p>
    </div>
</body>
</html>
```

**Why This Output Occurs**

The child overrides the sidebar section and uses `@parent` to include the default content from the layout's `@show` section, then appends its own content.

---

#### Real-World Cases with Explanation

**Case 1: Default Navigation with Page Additions**

A layout defines a `nav` section with `@show` containing the main navigation. Some pages extend this section with `@parent` to add breadcrumbs or sub-navigation.

**Explanation**: `@show` provides extendable default content.

---

### 8. `@push`

#### Definitions

**Core Definition**

The `@push` directive appends content to a named stack, which is later rendered in the layout using `@stack`.

**Technical Definition**

`@push('stack-name') ... @endpush` collects content from multiple views and appends it to a named stack. The stack is rendered at a designated location in the layout via `@stack('stack-name')`. Unlike sections, stacks can receive content from multiple views and components, and the content is concatenated in the order it was pushed.

**Beginner-Friendly Explanation**

`@push` is like putting notes into a shared mailbox. Different pages can each drop in their own notes (scripts, styles), and the layout collects all of them and displays them in the right place.

#### Purposes

- To collect scripts and styles from multiple views into a single location.
- To allow child views and components to contribute assets to a layout.
- To avoid duplicate script/style tags by centralizing asset collection.
- To support modular asset management in complex applications.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- In child views --}}
@push('scripts')
    <script src="/js/page-specific.js"></script>
@endpush

@push('styles')
    <style>
        .custom-class { color: red; }
    </style>
@endpush
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@push('stack-name')` | Begins a push block. |
| `@endpush` | Ends the push block. |
| `@stack('stack-name')` | Renders the collected content (in layout). |

**Syntax Rules**

- Stack names must match between `@push` and `@stack`.
- Multiple `@push` calls to the same stack are concatenated in order.
- `@push` can be used in views, components, and includes.
- The content is appended in the order the `@push` directives are executed.

**Constraints and Limitations**

- The `@stack` directive must exist in the layout for pushed content to render.
- Pushed content is not escaped; use with trusted data.
- `@push` cannot be used to push to sections; it is a separate system.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Pushing Scripts from Multiple Views**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    @stack('styles')
</head>
<body>
    @yield('content')

    @stack('scripts')
</body>
</html>
```

**Child View 1**

```blade
@extends('layouts.app')

@push('scripts')
    <script src="/js/jquery.js"></script>
@endpush

@section('content')
    <p>Page 1</p>
@endsection
```

**Child View 2**

```blade
@extends('layouts.app')

@push('scripts')
    <script src="/js/chart.js"></script>
@endpush

@section('content')
    <p>Page 2</p>
@endsection
```

**Expected Output (when rendering Child View 1)**

```html
<html>
<head>
</head>
<body>
    <p>Page 1</p>

    <script src="/js/jquery.js"></script>
</body>
</html>
```

**Expected Output (when rendering Child View 2)**

```html
<html>
<head>
</head>
<body>
    <p>Page 2</p>

    <script src="/js/chart.js"></script>
</body>
</html>
```

**Why This Output Occurs**

Each child view pushes its own script to the `scripts` stack. The layout renders the stack at the bottom of the body, outputting the pushed script.

---

**Example 2: Pushing Styles**

**Layout**

```blade
<html>
<head>
    @stack('styles')
</head>
<body>
    @yield('content')
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@push('styles')
    <link rel="stylesheet" href="/css/page.css">
@endpush

@section('content')
    <p>Content</p>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <link rel="stylesheet" href="/css/page.css">
</head>
<body>
    <p>Content</p>
</body>
</html>
```

**Why This Output Occurs**

The child pushes the CSS link to the `styles` stack, which is rendered in the `<head>` of the layout.

---

#### Real-World Cases with Explanation

**Case 1: Page-Specific JavaScript**

A complex application has a layout with a `scripts` stack at the bottom of the body. Each page pushes its own JavaScript libraries and inline scripts to this stack, avoiding conflicts and ensuring scripts load in the correct order.

**Explanation**: `@push` enables modular script management without requiring every page to include the same script tags.

**Case 2: Component Asset Injection**

A reusable chart component pushes the Chart.js library to the `scripts` stack when it is rendered. The layout automatically includes the library only when the component is used.

**Explanation**: `@push` allows components to declare their asset dependencies without requiring manual inclusion in the layout.

---

### 9. `@stack`

#### Definitions

**Core Definition**

The `@stack` directive renders all content that has been pushed to a named stack from various views and components.

**Technical Definition**

`@stack('stack-name')` is compiled to code that outputs the concatenated content of all `@push` calls to the specified stack. The stack is typically rendered in a layout's `<head>` (for styles) or before `</body>` (for scripts). The content is rendered in the order it was pushed.

**Beginner-Friendly Explanation**

`@stack` is the place in your layout where all the pushed content is displayed. It's like an inbox that collects all the scripts and styles from different pages and shows them together.

#### Purposes

- To render collected assets (scripts, styles) at a designated location in the layout.
- To centralize asset output while allowing distributed contribution.
- To support modular asset management across views and components.
- To avoid duplicate asset loading by consolidating pushes into one location.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@stack('scripts')
@stack('styles')
@stack('modals')
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@stack('scripts')` | Renders all content pushed to the `scripts` stack. |
| `@stack('styles')` | Renders all content pushed to the `styles` stack. |

**Syntax Rules**

- `@stack` is used in layouts (or any view).
- The stack name must match the name used in `@push` calls.
- If no content has been pushed to a stack, `@stack` outputs nothing.
- `@stack` can be used multiple times for the same stack (content is duplicated).

**Constraints and Limitations**

- Stack names are case-sensitive.
- The order of output is the order in which `@push` directives were executed.
- `@stack` does not provide default content if the stack is empty.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Rendering Multiple Stacks**

**Layout**

```blade
<!-- resources/views/layouts/app.blade.php -->
<html>
<head>
    @stack('styles')
</head>
<body>
    @yield('content')

    @stack('scripts')
    @stack('modals')
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@push('styles')
    <link rel="stylesheet" href="/css/page.css">
@endpush

@push('scripts')
    <script src="/js/page.js"></script>
@endpush

@section('content')
    <p>Content</p>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <link rel="stylesheet" href="/css/page.css">
</head>
<body>
    <p>Content</p>

    <script src="/js/page.js"></script>
    
</body>
</html>
```

**Why This Output Occurs**

The `styles` stack is rendered in the `<head>`, and the `scripts` stack is rendered before `</body>`. The `modals` stack is empty, so nothing is output for it.

---

#### Real-World Cases with Explanation

**Case 1: Script and Style Stacks**

A layout defines `@stack('styles')` in the `<head>` and `@stack('scripts')` before `</body>`. Pages and components push their assets to these stacks, and the layout renders them in the correct locations.

**Explanation**: `@stack` provides a clean separation between asset collection and asset rendering.

---

### 10. Layout Components

#### Definitions

**Core Definition**

Layout components are Blade components (typically anonymous) that use the `$slot` variable to render page content, providing a component-based alternative to traditional template inheritance.

**Technical Definition**

Instead of using `@extends` and `@section`, a layout component is defined as a Blade component (e.g., `resources/views/components/layouts/app.blade.php`) that renders `{{ $slot }}` where page content should appear. Child views use the component tag `<x-layouts.app>` and place their content inside. Named slots can be used for multiple content areas. Laravel's official starter kits (Breeze, Jetstream) use this approach.

**Beginner-Friendly Explanation**

Layout components are a newer way to build layouts. Instead of "extending" a layout, you "wrap" your content in a layout component. It's like putting your page content inside a special box that adds the header and footer.

#### Purposes

- To provide a component-based alternative to template inheritance.
- To offer a simpler mental model for developers familiar with component-based frameworks.
- To enable layout switching without changing the `@extends` directive.
- To support named slots for multiple content areas in a layout.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Layout Component: resources/views/components/layouts/app.blade.php --}}
<html>
<head>
    <title>{{ $title ?? 'Default Title' }}</title>
</head>
<body>
    <header>
        {{ $header ?? '' }}
    </header>

    <main>
        {{ $slot }}
    </main>

    <footer>
        {{ $footer ?? '' }}
    </footer>
</body>
</html>
```

```blade
{{-- Child View --}}
<x-layouts.app>
    <x-slot:title>Page Title</x-slot:title>
    <x-slot:header>
        <h1>Page Header</h1>
    </x-slot:header>

    <p>Main content here.</p>

    <x-slot:footer>
        <p>Footer content</p>
    </x-slot:footer>
</x-layouts.app>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-layouts.app>` | Component tag for the layout. |
| `{{ $slot }}` | Default slot for main content. |
| `{{ $title }}` | Named slot for title. |
| `<x-slot:title>` | Defines content for the title slot. |

**Syntax Rules**

- Layout components are placed in `resources/views/components/layouts/`.
- The component tag uses dot notation for nested directories.
- Named slots are accessed as variables in the layout component.
- The default slot (`$slot`) contains the content between the component tags.

**Constraints and Limitations**

- Layout components cannot use `@extends` (they are an alternative).
- Named slots must be defined with `<x-slot:name>`.
- The component approach may have slightly different performance characteristics than template inheritance (though negligible in practice).
- Layout components require Laravel 7+ (when components were introduced).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Layout Component**

**Layout Component**

```blade
<!-- resources/views/components/layouts/app.blade.php -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title ?? 'My App' }}</title>
</head>
<body>
    <nav>
        <a href="/">Home</a> | <a href="/about">About</a>
    </nav>

    <main>
        {{ $slot }}
    </main>

    <footer>
        <p>&copy; 2026 My App</p>
    </footer>
</body>
</html>
```

**Child View**

```blade
<!-- resources/views/home.blade.php -->
<x-layouts.app>
    <x-slot:title>Home Page</x-slot:title>

    <h1>Welcome!</h1>
    <p>This is the home page.</p>
</x-layouts.app>
```

**Expected Output**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Home Page</title>
</head>
<body>
    <nav>
        <a href="/">Home</a> | <a href="/about">About</a>
    </nav>

    <main>
        <h1>Welcome!</h1>
        <p>This is the home page.</p>
    </main>

    <footer>
        <p>&copy; 2026 My App</p>
    </footer>
</body>
</html>
```

**Why This Output Occurs**

The `<x-layouts.app>` component wraps the child content. The `$title` named slot is used for the page title, and the default `$slot` contains the main content.

---

**Example 2: Layout Component with Multiple Named Slots**

**Layout Component**

```blade
<!-- resources/views/components/layouts/app.blade.php -->
<html>
<head>
    <title>{{ $title ?? 'Default' }}</title>
    @stack('styles')
</head>
<body>
    <header>
        {{ $header ?? 'Default Header' }}
    </header>

    <aside>
        {{ $sidebar ?? 'Default Sidebar' }}
    </aside>

    <main>
        {{ $slot }}
    </main>

    @stack('scripts')
</body>
</html>
```

**Child View**

```blade
<x-layouts.app>
    <x-slot:title>Dashboard</x-slot:title>
    <x-slot:header>
        <h1>Dashboard Header</h1>
    </x-slot:header>
    <x-slot:sidebar>
        <p>Dashboard Sidebar</p>
    </x-slot:sidebar>

    <p>Main dashboard content.</p>
</x-layouts.app>
```

**Expected Output**

```html
<html>
<head>
    <title>Dashboard</title>
</head>
<body>
    <header>
        <h1>Dashboard Header</h1>
    </header>

    <aside>
        <p>Dashboard Sidebar</p>
    </aside>

    <main>
        <p>Main dashboard content.</p>
    </main>
</body>
</html>
```

**Why This Output Occurs**

The child provides content for the `title`, `header`, and `sidebar` named slots. The main content becomes the default `$slot`.

---

#### Real-World Cases with Explanation

**Case 1: Laravel Breeze/Jetstream Layouts**

Laravel's official starter kits (Breeze, Jetstream) use layout components in `resources/views/layouts/app.blade.php` rendered via the `App\View\Components\AppLayout` component class. This provides a modern, component-based approach to layouts.

**Explanation**: Layout components are the recommended approach in newer Laravel applications.

**Case 2: Multiple Layout Variants**

An application uses `<x-layouts.app>` for the main site and `<x-layouts.admin>` for the admin panel. Switching layouts is as simple as changing the component tag, with no need to modify `@extends` directives.

**Explanation**: Layout components make it easy to switch between different layout structures.

---

### 11. Layout Best Practices

#### Definitions

**Core Definition**

Layout best practices are the recommended guidelines for designing, organizing, and maintaining Blade layouts to maximize maintainability, performance, and developer experience.

**Technical Definition**

Best practices encompass directory organization, naming conventions, section and stack usage, the choice between template inheritance and layout components, performance considerations, and testing strategies. They aim to balance flexibility, readability, and efficiency.

**Beginner-Friendly Explanation**

Best practices are the "do's and don'ts" that experienced Laravel developers have learned. Following them makes your layouts easier to understand, maintain, and extend.

#### Key Best Practices

1. **Organize Layouts in `resources/views/layouts/`**: Keep all layout files in a dedicated directory. Use subdirectories for context-specific layouts (e.g., `layouts/admin/`, `layouts/public/`).

2. **Use Descriptive Section Names**: Name sections after their purpose (e.g., `content`, `sidebar`, `scripts`) rather than their position (e.g., `left`, `right`).

3. **Prefer `@yield` for Required Content, `@section ... @show` for Optional Content**: Use `@yield` when the section must be provided by the child. Use `@show` when the layout should provide default content that can be extended.

4. **Use `@parent` Sparingly**: While `@parent` is powerful, overuse can make it difficult to trace where content originates. Prefer overriding sections completely when possible.

5. **Leverage `@push`/`@stack` for Assets**: Use stacks for scripts and styles to avoid duplicating asset tags across pages. Place `@stack('styles')` in the `<head>` and `@stack('scripts')` before `</body>`.

6. **Consider Layout Components for New Projects**: For new Laravel applications, layout components (using `<x-layouts.app>`) offer a more modern, component-based approach that aligns with Laravel's direction. The official starter kits use this approach.

7. **Avoid Business Logic in Layouts**: Keep layouts focused on presentation. Move complex logic to controllers, view composers, or dedicated service classes.

8. **Use Partial Views for Reusable Blocks**: Extract headers, footers, and navigation into partial views included via `@include`. This keeps layouts clean and modular.

9. **Cache Compiled Views in Production**: Run `php artisan view:cache` during deployment to precompile all Blade templates, reducing the overhead of first-request compilation.

10. **Test Layouts with Different Content Scenarios**: Write tests that verify layouts render correctly with and without optional sections. Use Laravel's `blade()` testing method to render views with various data sets.

11. **Keep Inheritance Shallow**: Avoid deeply nested inheritance chains. A maximum of one or two levels of inheritance is usually sufficient. For more complex compositions, consider layout components or partials.

12. **Document Layout Sections and Stacks**: Provide clear documentation (in comments or a README) listing the sections and stacks available in each layout, especially for shared layout libraries.

#### Real-World Cases with Explanation

**Case 1: Organizing Layouts by Context**

A project with public and admin sections organizes layouts as `layouts/public/app.blade.php` and `layouts/admin/app.blade.php`. Views are grouped by feature, and reusable UI pieces (navbar, footer) are Blade components.

**Explanation**: Context-based organization keeps layouts together and makes it clear which layout applies to which part of the application.

**Case 2: Choosing Between Template Inheritance and Components**

A team building a new Laravel 12 application uses layout components (`<x-layouts.app>`) because they align with the component-based architecture of modern Laravel. A legacy application uses traditional `@extends`/`@section` inheritance, which continues to work without changes.

**Explanation**: The choice depends on project age, team familiarity, and architectural direction.

---

## References

- Laravel Blade Templates Documentation (Laravel 13.x) — https://laravel.com/docs/13.x/blade
- Laravel Blade Templates Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade Templates Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/blade
- Laravel Blade Templates Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/blade
- Laravel Blade Templates Documentation (Laravel 7.x) — https://laravel.com/framework/docs/7.x/blade
- Laravel Blade Templates Documentation (Laravel 5.4) — https://laravel.com/framework/docs/5.4/blade
- Laravel Livewire Layout Documentation — https://livewire.laravel.com/docs/layouts
- How to Create Layouts in Laravel Blade (Beginner Guide) — https://www.cloudways.com/blog/laravel-blade-layouts/
- Structuring Blade resources in a project with public and admin sections (Laracasts) — https://laracasts.com/discuss/channels/laravel/structuring-blade-resources-in-a-project-with-public-and-admin-sections
- Blade @stack directive usage (Laracasts) — https://laracasts.com/discuss/channels/laravel/how-trickly-use-blade-at-stack-directive
- @prepend directive for stack ordering (Laravel GitHub) — https://github.com/laravel/framework/blob/12.x/src/Illuminate/View/Compilers/Concerns/CompilesStacks.php

---

**Version Note**: This cheat sheet is based on Laravel 7.x through 13.x documentation. Template inheritance (`@extends`, `@section`, `@yield`) has remained stable across all versions. The `@parent` and `@show` directives have been available since early Laravel versions. Layout components were introduced with Blade components in Laravel 7. The `@prepend` directive was added in Laravel 5.5. Developers should always consult the documentation for their specific Laravel version.