# Laravel Blade Directives: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Blade directives are special keywords prefixed with `@` that provide clean, readable shortcuts for common PHP control structures, template inheritance, authentication checks, asset management, and other tasks within Laravel's Blade templating engine.

**Technical Definition**

Blade directives are compiled by the `BladeCompiler` class into corresponding PHP code. For example, `@if ($condition)` compiles to `<?php if ($condition): ?>`, and `@endif` compiles to `<?php endif; ?>`. Directives are identified via regular expressions and translated according to their registered compilation callbacks. Custom directives can be registered via `Blade::directive()`. Directives cover conditionals, loops, includes, layouts, stacks, authentication, raw PHP, form helpers, asset bundling, and more.

**Beginner-Friendly Explanation**

Blade directives are the "@-words" you write in your `.blade.php` files. Instead of writing messy PHP tags like `<?php if ($x): ?>`, you write `@if($x)`. They make your templates cleaner and easier to read, and they cover everything from "show this if that's true" to "include this form security token."

---

### Key Characteristics

- **Expressive Shortcuts**: Each directive replaces a verbose PHP construct with a concise, readable keyword.
- **Compile-Time Translation**: Directives are compiled into plain PHP during view compilation, adding no runtime overhead.
- **Context-Aware**: Directives like `@auth`, `@guest`, `@can`, and `@error` integrate with Laravel's authentication, authorization, and validation systems.
- **Composable**: Directives can be nested and combined (e.g., `@foreach` inside `@if` inside `@auth`).
- **Extensible**: Custom directives can be registered via `Blade::directive()` in a service provider.
- **Version-Specific**: Some directives (e.g., `@vite`, `@aware`, `@session`) were introduced in specific Laravel versions and may not be available in older applications.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with Blade templates in `resources/views`.
- **Basic knowledge of PHP control structures** (if/else, loops, switch).
- **Familiarity with Laravel's authentication and validation systems** for auth/error directives.
- **Vite** installed for `@vite` directives.

---

### Related Programming Areas

- **Blade Templates**: Directives are the primary syntax for dynamic behaviour in Blade views.
- **Blade Components**: `@props` and `@aware` are used in anonymous component authoring.
- **Authentication**: `@auth`, `@guest`, `@can`, `@cannot` integrate with Laravel's guards and policies.
- **Validation**: `@error` displays validation error messages.
- **Asset Bundling**: `@vite` and `@viteReactRefresh` integrate with Vite.
- **Localization**: `@lang` and `@choice` handle translations.

---

## Core Concepts

---

### 1. Conditional Statements

#### Definitions

**Core Definition**

Conditional directives evaluate boolean expressions and render blocks of template content only when the condition is met.

**Technical Definition**

Conditional directives compile to PHP `if`, `elseif`, `else`, and `endif` statements, or to negated `if` statements for `@unless`. `@isset` and `@empty` compile to their corresponding PHP language constructs. `@env` and `@production` check the application's environment via `app()->environment()`.

**Beginner-Friendly Explanation**

These are the "if/then" directives. They let you show or hide parts of your page based on conditions like whether a variable exists or what environment the app is running in.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@if` / `@endif` | Standard conditional block. |
| `@elseif` | Additional condition within an `@if` block. |
| `@else` | Fallback within an `@if` block. |
| `@unless` / `@endunless` | Negated conditional (runs if condition is false). |
| `@isset` / `@endisset` | Runs if a variable is defined and not null. |
| `@empty` / `@endempty` | Runs if a variable is "empty" (per PHP's `empty()`). |
| `@env` / `@endenv` | Runs if the application environment matches. |
| `@production` / `@endproduction` | Runs if the environment is `production`. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@if ($condition)
    <p>Condition is true</p>
@elseif ($otherCondition)
    <p>Other condition is true</p>
@else
    <p>Both conditions are false</p>
@endif

@unless ($condition)
    <p>Condition is false</p>
@endunless

@isset($records)
    <p>Records are defined</p>
@endisset

@empty($records)
    <p>No records found</p>
@endempty

@env('staging')
    <p>Staging environment</p>
@endenv

@production
    <p>Production environment</p>
@endproduction
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@if (...)` | Opens a conditional block. |
| `@elseif (...)` | Adds an additional condition. |
| `@else` | Fallback branch. |
| `@endif` | Closes the conditional block. |
| `@unless (...)` | Negated conditional. |
| `@isset(...)` | Checks `isset()` on the argument. |
| `@empty(...)` | Checks `empty()` on the argument. |
| `@env('...')` | Checks `app()->environment('...')`. |
| `@production` | Shorthand for `@env('production')`. |

**Syntax Rules**

- `@if` must be closed with `@endif`.
- `@elseif` and `@else` must appear before `@endif`.
- `@unless` must be closed with `@endunless`.
- `@isset` and `@empty` require a variable or expression as an argument.
- `@env` requires the environment name(s) as arguments.

**Constraints and Limitations**

- `@empty` uses PHP's `empty()` semantics: `0`, `"0"`, `""`, `null`, `false`, and empty arrays are all considered empty.
- `@isset` returns `false` for null values.
- `@env` checks the environment name against `APP_ENV`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard Conditional**

**Code**

```blade
@if ($user->isAdmin())
    <p>Admin panel</p>
@elseif ($user->isModerator())
    <p>Moderator panel</p>
@else
    <p>User dashboard</p>
@endif
```

**Expected Output (for an admin user)**

```html
<p>Admin panel</p>
```

**Why This Output Occurs**

The `@if` directive compiles to `<?php if($user->isAdmin()): ?>`. When `isAdmin()` returns `true`, the first branch renders and the others are skipped.

---

**Example 2: `@unless` and `@isset`**

**Code**

```blade
@unless ($user->isSubscribed())
    <p>Subscribe to continue reading.</p>
@endunless

@isset($records)
    <p>Records exist.</p>
@endisset
```

**Expected Output (if user is not subscribed and records are set)**

```html
<p>Subscribe to continue reading.</p>
<p>Records exist.</p>
```

**Why This Output Occurs**

`@unless` renders when the condition is `false`. `@isset` renders when the variable is defined and not null.

---

#### Real-World Cases with Explanation

**Case 1: Role-Based Navigation**

A navigation bar shows different links based on the user's role using `@if`, `@elseif`, and `@else`. This ensures each user sees only the navigation items they are authorized to access.

**Explanation**: Conditional directives keep role-based UI logic clean and readable without raw PHP tags.

---

### 2. Switch Statements

#### Definitions

**Core Definition**

Switch directives provide a clean way to evaluate a single expression against multiple possible values, rendering the matching case block.

**Technical Definition**

`@switch` compiles to PHP's `switch` statement. `@case` compiles to `case:`, `@break` to `break;`, `@default` to `default:`, and `@endswitch` to the closing brace. Switch support was introduced in Laravel 5.5.

**Beginner-Friendly Explanation**

Switch directives are like a multi-way if statement. You give a value, and Laravel picks the matching case to display.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@switch` | Opens the switch block. |
| `@case` | Defines a case value. |
| `@break` | Breaks out of the switch. |
| `@default` | Fallback case. |
| `@endswitch` | Closes the switch block. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@switch($variable)
    @case('value1')
        <p>Value 1</p>
        @break
    @case('value2')
        <p>Value 2</p>
        @break
    @default
        <p>Default case</p>
@endswitch
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@switch($var)` | Evaluates the variable. |
| `@case('value')` | Compares against a value. |
| `@break` | Prevents fall-through. |
| `@default` | Runs if no case matches. |
| `@endswitch` | Closes the switch. |

**Syntax Rules**

- `@break` is required after each `@case` to prevent fall-through.
- `@default` is optional but recommended.
- `@endswitch` must close the block.

**Constraints and Limitations**

- `@break` inside a `@case` breaks the switch; it does not break a surrounding loop.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Status Badge**

**Code**

```blade
@switch($order->status)
    @case('pending')
        <span class="badge badge-warning">Pending</span>
        @break
    @case('shipped')
        <span class="badge badge-info">Shipped</span>
        @break
    @case('delivered')
        <span class="badge badge-success">Delivered</span>
        @break
    @default
        <span class="badge">Unknown</span>
@endswitch
```

**Expected Output (for status `shipped`)**

```html
<span class="badge badge-info">Shipped</span>
```

**Why This Output Occurs**

`@switch` evaluates `$order->status`. The `@case('shipped')` branch matches, renders the badge, and `@break` prevents the `@default` from also rendering.

---

#### Real-World Cases with Explanation

**Case 1: Payment Status Display**

An e-commerce order page uses `@switch` to render different badges and messages for payment statuses (`pending`, `paid`, `failed`, `refunded`), keeping the template clean and exhaustive.

**Explanation**: Switch directives are ideal when a single value maps to multiple distinct UI states.

---

### 3. Looping Statements

#### Definitions

**Core Definition**

Looping directives iterate over arrays and collections, rendering a block of template content for each item, with support for empty-state handling and loop control.

**Technical Definition**

`@foreach`, `@for`, and `@while` compile to their PHP counterparts. `@forelse` compiles to a `foreach` loop with an `if` check for emptiness, providing an `@empty` block when the collection is empty. `@continue` and `@break` compile to PHP's `continue` and `break` statements. The `$loop` variable is automatically available inside loops and provides metadata (index, iteration, first, last, etc.).

**Beginner-Friendly Explanation**

These directives let you repeat parts of your template for each item in a list, with special handling for when the list is empty.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@for` / `@endfor` | Numeric loop. |
| `@foreach` / `@endforeach` | Iterate over an array or collection. |
| `@forelse` / `@endforelse` | Iterate with an `@empty` fallback. |
| `@empty` (within `@forelse`) | Renders when the collection is empty. |
| `@while` / `@endwhile` | Loop while a condition is true. |
| `@continue` | Skip the current iteration. |
| `@break` | Exit the loop. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@foreach ($users as $user)
    <p>{{ $user->name }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users found.</p>
@endforelse

@for ($i = 0; $i < 10; $i++)
    <p>{{ $i }}</p>
@endfor

@while ($condition)
    <p>Looping...</p>
@endwhile

@foreach ($users as $user)
    @continue($user->isBlocked())
    <p>{{ $user->name }}</p>
    @break($loop->iteration >= 5)
@endforeach
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@foreach` | Iterates over each item. |
| `@forelse` | Iterates with an `@empty` fallback. |
| `@for` | Numeric loop with initialisation, condition, and increment. |
| `@while` | Loops while the condition is true. |
| `@continue(condition)` | Skips the iteration if the condition is true. |
| `@break(condition)` | Exits the loop if the condition is true. |
| `$loop` | Automatic loop metadata variable. |

**Syntax Rules**

- `@foreach` must be closed with `@endforeach`.
- `@forelse` must be closed with `@endforelse` and must contain an `@empty` block.
- `@for` must be closed with `@endfor`.
- `@while` must be closed with `@endwhile`.
- `@continue` and `@break` can accept an optional condition.

**Constraints and Limitations**

- `@forelse`'s `@empty` block is mandatory.
- `$loop` is only available inside `@foreach` and `@forelse`.
- Modifying the collection inside the loop can lead to unexpected behaviour.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: `@forelse` with Empty State**

**Code**

```blade
@forelse ($posts as $post)
    <article>{{ $post->title }}</article>
@empty
    <p>No posts yet.</p>
@endforelse
```

**Expected Output (when `$posts` is empty)**

```html
<p>No posts yet.</p>
```

**Why This Output Occurs**

`@forelse` checks if the collection is empty. If so, the `@empty` block renders instead of the loop body.

---

**Example 2: `$loop` Variable and `@break`**

**Code**

```blade
@foreach ($users as $user)
    @continue($user->isBlocked())
    <p>{{ $loop->iteration }}. {{ $user->name }}</p>
    @break($loop->iteration >= 3)
@endforeach
```

**Expected Output (for three non-blocked users)**

```html
<p>1. Alice</p>
<p>2. Bob</p>
<p>3. Charlie</p>
```

**Why This Output Occurs**

`$loop->iteration` starts at 1 and increments each iteration. `@break` exits the loop after the third iteration. `@continue` skips blocked users.

---

#### Real-World Cases with Explanation

**Case 1: Product Listing with Empty State**

A product listing page uses `@forelse` to display products and shows a friendly "No products found" message via `@empty` when the catalogue is empty.

**Explanation**: `@forelse` eliminates the need for a separate `@if` check for emptiness.

---

### 4. Subviews and Fragment Inclusion

#### Definitions

**Core Definition**

Inclusion directives embed the rendered output of another Blade view within the current view, optionally based on conditions or by iterating over a collection.

**Technical Definition**

`@include` renders a view immediately. `@includeIf` renders a view only if it exists. `@includeWhen` and `@includeUnless` conditionally include a view based on a boolean expression. `@each` renders a view for each item in a collection, with an optional empty view.

**Beginner-Friendly Explanation**

These directives let you reuse pieces of templates—like a header or a sidebar—by including them in other templates.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@include` | Includes a view unconditionally. |
| `@includeIf` | Includes a view only if it exists. |
| `@includeWhen` | Includes a view if a condition is true. |
| `@includeUnless` | Includes a view if a condition is false. |
| `@each` | Renders a view for each item in a collection. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@include('partials.header')
@include('partials.user', ['user' => $user])

@includeIf('partials.optional')

@includeWhen($condition, 'partials.conditional')

@includeUnless($condition, 'partials.negated')

@each('partials.post', $posts, 'post', 'partials.empty')
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@include('view')` | Includes the view. |
| `['key' => $value]` | Optional data passed to the included view. |
| `@includeIf('view')` | Includes only if the view file exists. |
| `@includeWhen($bool, 'view')` | Includes if `$bool` is truthy. |
| `@includeUnless($bool, 'view')` | Includes if `$bool` is falsy. |
| `@each('view', $items, 'item', 'empty')` | Renders a view for each item. |

**Syntax Rules**

- The view name uses dot notation for nested directories.
- Data can be passed as an associative array.
- `@each` requires the view name, the collection, the variable name for each item, and optionally an empty view.

**Constraints and Limitations**

- Included views have access to the parent's data by default; additional data can be passed as the second argument.
- `@each` does not support additional data per item beyond the item variable.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Include**

**Code**

```blade
@includeWhen($user->isAdmin(), 'partials.admin-toolbar')

@includeUnless($user->isAdmin(), 'partials.user-toolbar')
```

**Expected Output (for an admin user)**

```html
<!-- partials.admin-toolbar rendered -->
```

**Why This Output Occurs**

`@includeWhen` renders the admin toolbar because `isAdmin()` is `true`. `@includeUnless` is skipped for the same reason.

---

**Example 2: `@each` with Empty View**

**Code**

```blade
@each('partials.post', $posts, 'post', 'partials.no-posts')
```

**Expected Output (when `$posts` has items)**

```html
<!-- partials.post rendered for each post -->
```

**Expected Output (when `$posts` is empty)**

```html
<!-- partials.no-posts rendered once -->
```

**Why This Output Occurs**

`@each` iterates over the collection, rendering the specified view for each item. If the collection is empty, the fourth argument (empty view) is rendered.

---

#### Real-World Cases with Explanation

**Case 1: Reusable Header and Footer**

A layout includes a header and footer via `@include('partials.header')` and `@include('partials.footer')`, allowing these shared elements to be maintained in one place.

**Explanation**: Inclusion directives promote DRY principles by reusing template fragments.

---

### 5. Template Layouts & Inheritance

#### Definitions

**Core Definition**

Layout and inheritance directives allow child views to extend a parent layout, injecting content into named sections and yielding placeholders.

**Technical Definition**

`@extends` declares the parent layout. `@section` defines a named content block. `@yield` outputs a section's content in the layout. `@parent` includes the parent's version of a section in a child view. `@show` defines a section and immediately renders it in the layout.

**Beginner-Friendly Explanation**

These directives let you build a master template and fill in the unique parts for each page.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@extends` | Declares the parent layout. |
| `@section` | Defines a content block. |
| `@show` | Defines and immediately renders a section (in layouts). |
| `@yield` | Outputs a section's content (in layouts). |
| `@parent` | Includes the parent's section content (in child views). |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Child view --}}
@extends('layouts.app')

@section('title', 'Page Title')

@section('content')
    <p>Page content</p>
@endsection

@section('sidebar')
    @parent
    <p>Additional sidebar content</p>
@endsection
```

```blade
{{-- Layout --}}
<html>
<head>
    <title>@yield('title')</title>
</head>
<body>
    @section('sidebar')
        <p>Default sidebar</p>
    @show

    <main>@yield('content')</main>
</body>
</html>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@extends('layout')` | Declares the parent layout. |
| `@section('name')` | Begins a content section. |
| `@endsection` | Ends a section definition. |
| `@yield('name')` | Outputs a section in the layout. |
| `@show` | Defines and renders a section in the layout. |
| `@parent` | Includes the parent's section content. |

**Syntax Rules**

- `@extends` must be the first directive in a child view.
- `@section` must be closed with `@endsection` (or `@stop`).
- `@yield` is used in layouts, not child views.
- `@parent` must be inside a `@section` in a child view.

**Constraints and Limitations**

- A child view can only extend one layout.
- `@parent` only works with sections defined via `@section ... @show` in the layout, not with `@yield`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Layout Inheritance**

**Layout**

```blade
<html>
<head>
    <title>@yield('title', 'My App')</title>
</head>
<body>
    @yield('content')
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@section('title', 'Home')

@section('content')
    <h1>Welcome!</h1>
@endsection
```

**Expected Output**

```html
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h1>Welcome!</h1>
</body>
</html>
```

**Why This Output Occurs**

`@extends` links the child to the layout. `@section('title', 'Home')` fills the `@yield('title')` placeholder, and `@section('content')` fills the `@yield('content')` placeholder.

---

**Example 2: `@parent` in a Section**

**Layout**

```blade
@section('sidebar')
    <p>Default sidebar</p>
@show
```

**Child View**

```blade
@extends('layouts.app')

@section('sidebar')
    @parent
    <p>Page-specific sidebar</p>
@endsection
```

**Expected Output**

```html
<p>Default sidebar</p>
<p>Page-specific sidebar</p>
```

**Why This Output Occurs**

`@parent` includes the layout's default sidebar content, and the child's additional content is appended.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel Layout**

An admin panel uses a layout with a sidebar, header, and footer. Each admin page extends the layout and provides its content in the `content` section, ensuring consistency across all admin pages.

**Explanation**: Template inheritance centralizes the layout structure and avoids duplication.

---

### 6. Stacks

#### Definitions

**Core Definition**

Stack directives collect content from multiple views and render it at a designated location in a layout, commonly used for page-specific CSS and JavaScript.

**Technical Definition**

`@push` appends content to a named stack. `@stack` renders the accumulated content. `@prepend` inserts content at the beginning of a stack. `@pushOnce` and `@prependOnce` ensure content is added only once per rendering cycle.

**Beginner-Friendly Explanation**

Stacks are like collection boxes for scripts and styles. Different pages can drop their own files into the box, and the layout displays everything together in the right place.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@push` | Appends content to a stack. |
| `@stack` | Renders the stack's content. |
| `@prepend` | Prepends content to a stack. |
| `@pushOnce` | Appends content only once. |
| `@prependOnce` | Prepends content only once. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- In a layout --}}
@stack('scripts')

{{-- In a view --}}
@push('scripts')
    <script src="/js/page.js"></script>
@endpush

@prepend('scripts')
    <script src="/js/critical.js"></script>
@endprepend

@pushOnce('scripts')
    <script src="/js/once.js"></script>
@endPushOnce
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@stack('name')` | Renders the named stack. |
| `@push('name')` | Appends content to the stack. |
| `@prepend('name')` | Prepends content to the stack. |
| `@pushOnce('name')` | Appends content once per rendering cycle. |
| `@prependOnce('name')` | Prepends content once per rendering cycle. |

**Syntax Rules**

- The stack name must match between `@push`/`@prepend` and `@stack`.
- `@push` and `@prepend` must be closed with `@endpush` and `@endprepend`.
- `@pushOnce` can accept a unique identifier as a second argument to prevent duplicate content from different templates.

**Constraints and Limitations**

- Stack content is not escaped.
- If no `@stack` is defined for a given name, pushed content is discarded.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Page-Specific Scripts**

**Layout**

```blade
<html>
<body>
    @yield('content')
    @stack('scripts')
</body>
</html>
```

**Child View**

```blade
@extends('layouts.app')

@push('scripts')
    <script src="/js/home.js"></script>
@endpush

@section('content')
    <p>Home</p>
@endsection
```

**Expected Output**

```html
<html>
<body>
    <p>Home</p>
    <script src="/js/home.js"></script>
</body>
</html>
```

**Why This Output Occurs**

The child pushes the script to the `scripts` stack, which is rendered at the bottom of the layout body.

---

**Example 2: `@pushOnce` in a Loop**

**Component**

```blade
@pushOnce('scripts')
    <script src="/js/chart.js"></script>
@endPushOnce

<div class="chart" data-id="{{ $id }}"></div>
```

**Expected Output (when the component is rendered multiple times)**

```html
<script src="/js/chart.js"></script>
<div class="chart" data-id="1"></div>
<div class="chart" data-id="2"></div>
```

**Why This Output Occurs**

`@pushOnce` ensures the script is pushed only on the first rendering of the component, even if the component appears multiple times.

---

#### Real-World Cases with Explanation

**Case 1: Component Asset Injection**

A chart component pushes Chart.js to the `scripts` stack using `@pushOnce`. The layout renders the stack, ensuring the library loads only once regardless of how many charts are on the page.

**Explanation**: `@pushOnce` prevents duplicate script tags when components are reused.

---

### 7. Authentication and Authorization

#### Definitions

**Core Definition**

Authentication and authorization directives conditionally render content based on whether a user is authenticated or authorized to perform an action.

**Technical Definition**

`@auth` and `@guest` check the authentication status via Laravel's guards. `@can` and `@cannot` check authorization via gates and policies. These directives compile to calls on the `Auth` and `Gate` facades.

**Beginner-Friendly Explanation**

These directives show or hide parts of the page based on whether the user is logged in or has permission to do something.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@auth` | Renders if the user is authenticated. |
| `@guest` | Renders if the user is not authenticated. |
| `@can` | Renders if the user is authorized for an ability. |
| `@cannot` | Renders if the user is not authorized. |
| `@canany` | Renders if the user is authorized for any of the abilities. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@auth
    <p>Welcome, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <p>Please log in.</p>
@endguest

@can('update', $post)
    <a href="/posts/{{ $post->id }}/edit">Edit</a>
@endcan

@cannot('delete', $post)
    <p>You cannot delete this post.</p>
@endcannot
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@auth` | Checks if a user is authenticated. |
| `@guest` | Checks if no user is authenticated. |
| `@can('ability', $model)` | Checks authorization via gates/policies. |
| `@cannot('ability', $model)` | Inverse of `@can`. |
| `@canany(['a', 'b'], $model)` | Checks if any ability is allowed. |

**Syntax Rules**

- `@auth` can optionally specify a guard: `@auth('admin')`.
- `@can` and `@cannot` accept an ability name and optionally a model instance.
- All directives have corresponding closing directives (`@endauth`, `@endguest`, `@endcan`, `@endcannot`).

**Constraints and Limitations**

- `@can` relies on gates and policies being defined.
- `@auth` without a guard uses the default guard.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Authentication Check**

**Code**

```blade
@auth
    <p>Hello, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <a href="/login">Log in</a>
@endguest
```

**Expected Output (for a logged-in user)**

```html
<p>Hello, John!</p>
```

**Why This Output Occurs**

`@auth` checks if a user is authenticated. If so, the user's name is displayed. `@guest` is skipped.

---

**Example 2: Authorization Check**

**Code**

```blade
@can('update', $post)
    <a href="/posts/{{ $post->id }}/edit">Edit Post</a>
@endcan
```

**Expected Output (if the user can update the post)**

```html
<a href="/posts/1/edit">Edit Post</a>
```

**Why This Output Occurs**

`@can` checks the `update` policy for the `$post` model. If the user is authorized, the edit link is rendered.

---

#### Real-World Cases with Explanation

**Case 1: Conditional Admin Actions**

A blog post page uses `@can('update', $post)` to show an "Edit" button only to the post's author or an administrator.

**Explanation**: Authorization directives prevent unauthorized users from seeing actions they cannot perform.

---

### 8. Raw PHP, JSON, and Optimization

#### Definitions

**Core Definition**

These directives allow embedding raw PHP, rendering JSON safely, and optimising template rendering by executing content only once.

**Technical Definition**

`@php` and `@endphp` execute a block of plain PHP. `@json` compiles to `json_encode()` with proper escaping. `@once` ensures a block is evaluated only once per rendering cycle. `@use` imports a PHP class without a full `@php` block.

**Beginner-Friendly Explanation**

`@php` lets you write regular PHP code inside Blade. `@json` safely outputs data as JSON. `@once` makes sure something only happens one time.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@php` / `@endphp` | Executes a block of PHP. |
| `@json` | Outputs JSON-encoded data. |
| `@once` | Renders content only once per cycle. |
| `@use` | Imports a PHP class or function. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@php
    $counter = 1;
@endphp

<script>
    var app = @json($array);
</script>

@once
    @push('scripts')
        <script src="/js/once.js"></script>
    @endpush
@endonce

@use('App\Models\Flight')
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@php ... @endphp` | Executes PHP code. |
| `@json($data)` | Compiles to `json_encode($data, JSON_HEX_TAG | ...)`. |
| `@once ... @endonce` | Ensures content is rendered once. |
| `@use('Class')` | Imports a class (Laravel 10.26+). |

**Syntax Rules**

- `@php` must be closed with `@endphp`.
- `@json` accepts any JSON-serialisable data.
- `@once` must be closed with `@endonce`.
- `@use` supports aliases and grouped imports.

**Constraints and Limitations**

- `@php` should be used sparingly; business logic belongs in controllers.
- `@once` is scoped to the current rendering cycle.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: `@php` and `@json`**

**Code**

```blade
@php
    $user = ['name' => 'John', 'email' => 'john@example.com'];
@endphp

<script>
    var user = @json($user);
</script>
```

**Expected Output**

```html
<script>
    var user = {"name":"John","email":"john@example.com"};
</script>
```

**Why This Output Occurs**

`@php` defines the array. `@json` compiles to a safe `json_encode()` call, outputting the JSON directly into the script.

---

**Example 2: `@once` with `@push`**

**Code**

```blade
@once
    @push('scripts')
        <script src="/js/critical.js"></script>
    @endpush
@endonce
```

**Expected Output**

The script is pushed to the `scripts` stack only once, even if the template is rendered multiple times.

**Why This Output Occurs**

`@once` ensures the enclosed block is evaluated only on the first rendering pass.

---

#### Real-World Cases with Explanation

**Case 1: Passing Data to JavaScript**

A page uses `@json($chartData)` to embed a PHP array as a JavaScript variable, avoiding manual `json_encode()` calls and ensuring proper escaping.

**Explanation**: `@json` simplifies passing server-side data to client-side scripts.

---

### 9. HTML Attributes and Styling

#### Definitions

**Core Definition**

These directives conditionally output HTML attributes (class, style, checked, selected, disabled, readonly, required) based on boolean expressions.

**Technical Definition**

`@class` compiles to a conditional class string. `@style` compiles to conditional inline styles. `@checked`, `@selected`, `@disabled`, `@readonly`, and `@required` output their respective attributes when the condition is true.

**Beginner-Friendly Explanation**

These directives let you add or skip HTML attributes (like `checked` or `disabled`) based on conditions, without writing messy `@if` blocks.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@class` | Conditionally adds CSS classes. |
| `@style` | Conditionally adds inline styles. |
| `@checked` | Outputs `checked` if true. |
| `@selected` | Outputs `selected` if true. |
| `@disabled` | Outputs `disabled` if true. |
| `@readonly` | Outputs `readonly` if true. |
| `@required` | Outputs `required` if true. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<div @class(['active' => $isActive, 'disabled' => $isDisabled])>
    Content
</div>

<span @style(['color: red' => $isError])>Error</span>

<input type="checkbox" @checked($isChecked)>
<option @selected($isSelected)>Option</option>
<button @disabled($isDisabled)>Submit</button>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@class([...])` | Array keys are classes; values are conditions. |
| `@style([...])` | Array keys are style declarations; values are conditions. |
| `@checked($bool)` | Outputs `checked` when `$bool` is true. |
| `@selected($bool)` | Outputs `selected` when true. |
| `@disabled($bool)` | Outputs `disabled` when true. |
| `@readonly($bool)` | Outputs `readonly` when true. |
| `@required($bool)` | Outputs `required` when true. |

**Syntax Rules**

- `@class` and `@style` accept an array where keys are the class/style and values are boolean conditions.
- `@checked`, `@selected`, etc. accept a boolean expression.

**Constraints and Limitations**

- `@class` and `@style` were introduced in Laravel 8.51+.
- The boolean attribute directives output the bare attribute name when true.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Class**

**Code**

```blade
<div @class(['alert', 'alert-danger' => $isError, 'alert-success' => !$isError])>
    {{ $message }}
</div>
```

**Expected Output (for `$isError = true`)**

```html
<div class="alert alert-danger">
    Message
</div>
```

**Why This Output Occurs**

`@class` includes `alert` (always) and `alert-danger` (because `$isError` is true). `alert-success` is omitted.

---

**Example 2: Checked and Disabled**

**Code**

```blade
<input type="checkbox" @checked($user->isActive)>
<button @disabled($user->isBlocked)>Submit</button>
```

**Expected Output (for an active, non-blocked user)**

```html
<input type="checkbox" checked>
<button>Submit</button>
```

**Why This Output Occurs**

`@checked` outputs `checked` because `isActive` is true. `@disabled` outputs nothing because `isBlocked` is false.

---

#### Real-World Cases with Explanation

**Case 1: Form State Restoration**

A form restores checkbox and select states after validation failure using `@checked(old('remember'))` and `@selected(old('country') == $country)`.

**Explanation**: These directives simplify form state restoration without conditional blocks.

---

### 10. Component Authoring

#### Definitions

**Core Definition**

These directives are used within anonymous Blade components to declare expected data (props) and access data from parent components.

**Technical Definition**

`@props` declares data variables for an anonymous component, separating them from pass-through attributes. `@aware` accesses data from a parent component that was passed via `@props`.

**Beginner-Friendly Explanation**

`@props` tells a component "here is the data I expect." `@aware` lets a child component access data from its parent component.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@props` | Declares data variables in an anonymous component. |
| `@aware` | Accesses parent component data. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Component: resources/views/components/alert.blade.php --}}
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

```blade
{{-- Child component --}}
@aware(['color' => 'gray'])

<div class="child" style="color: {{ $color }}"></div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@props([...])` | Declares props with optional defaults. |
| `@aware([...])` | Declares inherited props from a parent component. |

**Syntax Rules**

- `@props` must be at the top of an anonymous component file.
- `@aware` is used inside a child component.
- `@aware` can only access props that are explicitly passed to the parent component via HTML attributes.

**Constraints and Limitations**

- `@aware` cannot access default `@props` values that were not explicitly passed.
- `@props` only works in anonymous components.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: `@props` in an Alert Component**

**Component**

```blade
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

**Usage**

```blade
<x-alert type="error" message="Something went wrong." />
```

**Expected Output**

```html
<div class="alert alert-error">
    Something went wrong.
</div>
```

**Why This Output Occurs**

`@props` declares `$type` and `$message`. The `type` attribute is used in the class, and `message` is echoed.

---

**Example 2: `@aware` in a Child Component**

**Parent Component**

```blade
@props(['color' => 'gray'])
<div {{ $attributes->class(['card', 'card-' . $color]) }}>
    {{ $slot }}
</div>
```

**Child Component**

```blade
@aware(['color' => 'gray'])
<span class="text-{{ $color }}">{{ $slot }}</span>
```

**Usage**

```blade
<x-card color="blue">
    <x-card-text>Hello</x-card-text>
</x-card>
```

**Expected Output**

```html
<div class="card card-blue">
    <span class="text-blue">Hello</span>
</div>
```

**Why This Output Occurs**

The parent's `color` prop is `blue`. The child uses `@aware(['color'])` to access the inherited `color` value.

---

#### Real-World Cases with Explanation

**Case 1: Accordion Component**

An accordion parent component passes a `variant` prop to its child items via `@aware`, allowing all items to share the same styling without explicitly passing the prop to each child.

**Explanation**: `@aware` reduces prop drilling in nested component structures.

---

### 11. Form, Security, and Session

#### Definitions

**Core Definition**

These directives handle form security (CSRF), HTTP method spoofing, validation error display, and session data access.

**Technical Definition**

`@csrf` outputs a hidden CSRF token field. `@method('PUT')` outputs a hidden `_method` field for method spoofing. `@error` and `@enderror` check and display validation errors. `@session` checks for a session value.

**Beginner-Friendly Explanation**

These directives handle the behind-the-scenes security and error handling for forms.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@csrf` | Outputs a hidden CSRF token field. |
| `@method('PUT')` | Outputs a hidden `_method` field. |
| `@error('field')` / `@enderror` | Displays validation errors. |
| `@session('key')` / `@endsession` | Checks for a session value. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<form method="POST" action="/profile">
    @csrf

    @method('PUT')

    <input type="text" name="email" value="{{ old('email') }}">

    @error('email')
        <span class="error">{{ $message }}</span>
    @enderror
</form>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@csrf` | Generates `<input type="hidden" name="_token" value="...">`. |
| `@method('PUT')` | Generates `<input type="hidden" name="_method" value="PUT">`. |
| `@error('field')` | Checks if a validation error exists for the field. |
| `$message` | The error message (available inside `@error`). |
| `@session('key')` | Checks for a session flash value. |

**Syntax Rules**

- `@csrf` must be included in every `POST`, `PUT`, `PATCH`, or `DELETE` form.
- `@method` accepts the HTTP verb as a string.
- `@error` must be closed with `@enderror`.

**Constraints and Limitations**

- `@csrf` relies on Laravel's CSRF middleware being enabled.
- `@method` only works with forms submitted via `POST`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: CSRF and Method Spoofing**

**Code**

```blade
<form method="POST" action="/posts/1">
    @csrf
    @method('PUT')

    <input type="text" name="title" value="{{ $post->title }}">
    <button type="submit">Update</button>
</form>
```

**Expected Output**

```html
<form method="POST" action="/posts/1">
    <input type="hidden" name="_token" value="...">
    <input type="hidden" name="_method" value="PUT">

    <input type="text" name="title" value="...">
    <button type="submit">Update</button>
</form>
```

**Why This Output Occurs**

`@csrf` generates the hidden token field. `@method('PUT')` generates the hidden `_method` field, which Laravel uses to spoof the `PUT` verb.

---

**Example 2: Validation Error Display**

**Code**

```blade
<input type="email" name="email" value="{{ old('email') }}">

@error('email')
    <span class="text-red-500">{{ $message }}</span>
@enderror
```

**Expected Output (when there is an email validation error)**

```html
<input type="email" name="email" value="invalid-email">
<span class="text-red-500">The email must be a valid email address.</span>
```

**Why This Output Occurs**

`@error('email')` checks the validation error bag for the `email` field. If an error exists, the `$message` variable contains the error message and is rendered.

---

#### Real-World Cases with Explanation

**Case 1: Secure Form Submission**

Every form in a Laravel application includes `@csrf` to protect against cross-site request forgery. Forms that perform updates or deletions use `@method('PUT')` or `@method('DELETE')` to spoof the correct HTTP verb.

**Explanation**: `@csrf` and `@method` are essential for secure, RESTful form handling.

---

### 12. Assets & Frontend Tooling Integration (Vite)

#### Definitions

**Core Definition**

Vite directives generate the appropriate `<link>` and `<script>` tags for assets compiled by Vite, supporting both development (with hot module replacement) and production (with versioned files).

**Technical Definition**

`@vite` compiles to calls on the `Vite` facade, which reads the manifest file or Vite dev server to generate asset tags. `@viteReactRefresh` injects the React refresh runtime for React applications.

**Beginner-Friendly Explanation**

`@vite` loads your CSS and JavaScript files that were compiled by Vite. `@viteReactRefresh` enables fast refresh for React during development.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@vite` | Loads Vite-compiled assets. |
| `@viteReactRefresh` | Enables React Fast Refresh. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- In the <head> of your layout --}}
@viteReactRefresh
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@vite([...])` | Accepts an array of entry points. |
| `@viteReactRefresh` | Injects the React refresh runtime (development only). |

**Syntax Rules**

- `@vite` accepts a single path or an array of paths.
- `@viteReactRefresh` must be placed before `@vite` for React applications.
- Both directives should be placed in the `<head>` of the root layout.

**Constraints and Limitations**

- Requires Vite to be installed and configured (`npm install`, `vite.config.js`).
- In production, the Vite manifest must be built (`npm run build`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Vite Setup**

**Layout**

```blade
<!DOCTYPE html>
<html>
<head>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    @yield('content')
</body>
</html>
```

**Expected Output (development)**

```html
<link rel="stylesheet" href="http://localhost:5173/resources/css/app.css">
<script type="module" src="http://localhost:5173/resources/js/app.js"></script>
```

**Why This Output Occurs**

In development, `@vite` points to the Vite dev server (usually `localhost:5173`), enabling hot module replacement.

---

**Example 2: React with Fast Refresh**

**Layout**

```blade
<head>
    @viteReactRefresh
    @vite(['resources/js/app.jsx'])
</head>
```

**Expected Output**

The React refresh runtime is injected before the main application script, enabling Fast Refresh during development.

**Why This Output Occurs**

`@viteReactRefresh` adds the necessary preamble for React Fast Refresh, which must load before the React application.

---

#### Real-World Cases with Explanation

**Case 1: SPA with React and Vite**

A Laravel application serving a React SPA uses `@viteReactRefresh` and `@vite(['resources/js/app.jsx'])` in the root Blade layout to load the compiled React application.

**Explanation**: Vite directives integrate Laravel's backend with modern frontend tooling.

---

### 13. Ecosystem Packages (Livewire & Inertia)

#### Definitions

**Core Definition**

These directives integrate Laravel with ecosystem packages: Livewire (for dynamic components) and Inertia.js (for SPA-like experiences without an API).

**Technical Definition**

`@livewireStyles` and `@livewireScripts` inject Livewire's CSS and JavaScript. `@inertia` and `@inertiaHead` render Inertia's root element and head content. `@routes` exposes Laravel routes to JavaScript via the Ziggy package.

**Beginner-Friendly Explanation**

These directives are the "glue" between Laravel and popular frontend packages.

#### Directives Covered

| Directive | Package | Purpose |
|-----------|---------|---------|
| `@livewireStyles` | Livewire | Injects Livewire CSS. |
| `@livewireScripts` | Livewire | Injects Livewire JavaScript. |
| `@inertia` | Inertia.js | Renders the Inertia root element. |
| `@inertiaHead` | Inertia.js | Renders Inertia head content. |
| `@routes` | Ziggy | Exposes Laravel routes to JavaScript. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<head>
    @livewireStyles
    @routes
</head>
<body>
    @inertia
    @livewireScripts
</body>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@livewireStyles` | Injects Livewire's `<style>` tags. |
| `@livewireScripts` | Injects Livewire's `<script>` tags. |
| `@inertia` | Renders the Inertia root `<div id="app">`. |
| `@routes` | Injects the Ziggy route configuration script. |

**Syntax Rules**

- `@livewireStyles` belongs in the `<head>`; `@livewireScripts` belongs before `</body>`.
- `@inertia` replaces the root div in Inertia applications.
- `@routes` requires the Ziggy package.

**Constraints and Limitations**

- These directives only work if the corresponding package is installed.
- Inertia v3 may use `<x-inertia>` components instead of `@inertia`/`@inertiaHead`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Livewire Setup**

**Layout**

```blade
<html>
<head>
    @livewireStyles
</head>
<body>
    @yield('content')
    @livewireScripts
</body>
</html>
```

**Expected Output**

Livewire's CSS is injected in the `<head>` and its JavaScript is injected before `</body>`.

**Why This Output Occurs**

`@livewireStyles` and `@livewireScripts` inject the necessary assets for Livewire components to function.

---

#### Real-World Cases with Explanation

**Case 1: Livewire Dashboard**

A dashboard built with Livewire includes `@livewireStyles` and `@livewireScripts` in the layout, enabling dynamic, reactive components without writing JavaScript.

**Explanation**: Livewire directives are essential for integrating Livewire into a Laravel application.

---

### 14. Localization

#### Definitions

**Core Definition**

Localization directives output translated strings and handle pluralization based on the application's locale.

**Technical Definition**

`@lang` compiles to `__()` (translation helper). `@choice` compiles to `trans_choice()` for pluralized strings.

**Beginner-Friendly Explanation**

`@lang` translates a string into the current language. `@choice` handles plural forms (e.g., "1 day" vs. "2 days").

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@lang` | Outputs a translated string. |
| `@choice` | Outputs a pluralized translated string. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@lang('messages.welcome')

@choice('messages.apples', $count)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@lang('key')` | Looks up the translation for the key. |
| `@choice('key', $count)` | Chooses the plural form based on `$count`. |

**Syntax Rules**

- Translation keys use dot notation for nested files.
- `@choice` requires a count to determine pluralization.

**Constraints and Limitations**

- Translation files must exist in `lang/{locale}`.
- Pluralization rules are locale-specific.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Translation**

**Code**

```blade
@lang('messages.welcome')
```

**Expected Output (for English locale)**

```
Welcome to our application
```

**Why This Output Occurs**

`@lang` looks up the `welcome` key in `lang/en/messages.php` and outputs the translated string.

---

**Example 2: Pluralization**

**Code**

```blade
@choice('messages.apples', 3)
```

**Expected Output**

```
3 apples
```

**Why This Output Occurs**

`@choice` selects the plural form of `messages.apples` based on the count `3`.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Language Application**

A multi-language application uses `@lang` and `@choice` throughout its views to display content in the user's preferred language.

**Explanation**: Localization directives make it easy to support multiple languages.

---

### 15. Debugging

#### Definitions

**Core Definition**

Debugging directives dump variable contents and optionally halt execution, providing a quick way to inspect data during development.

**Technical Definition**

`@dd` compiles to `dd()` (dump and die). `@dump` compiles to `dump()` (dump without halting).

**Beginner-Friendly Explanation**

`@dd` shows you what a variable contains and then stops the page. `@dump` shows the variable but lets the page continue.

#### Directives Covered

| Directive | Purpose |
|-----------|---------|
| `@dd` | Dumps variables and stops execution. |
| `@dump` | Dumps variables without stopping execution. |

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@dd($variable)

@dump($variable)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@dd($var)` | Dumps and dies. |
| `@dump($var)` | Dumps without dying. |

**Syntax Rules**

- Both directives accept one or more variables.
- `@dd` halts execution immediately after dumping.

**Constraints and Limitations**

- These directives should be removed before deploying to production.
- `@dd` can interrupt the rendering of the page.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dumping Without Halting**

**Code**

```blade
@dump($users)
```

**Expected Output**

The contents of `$users` are dumped to the output, and the page continues to render.

**Why This Output Occurs**

`@dump` compiles to `dump()`, which outputs the variable but does not halt execution.

---

**Example 2: Dump and Die**

**Code**

```blade
@dd($user)
```

**Expected Output**

The contents of `$user` are dumped, and the page stops rendering immediately.

**Why This Output Occurs**

`@dd` compiles to `dd()`, which outputs the variable and terminates the script.

---

#### Real-World Cases with Explanation

**Case 1: Debugging Data in a View**

A developer uses `@dump($posts)` to inspect the structure of a collection in a Blade view without interrupting the page.

**Explanation**: Debugging directives provide quick, in-template introspection during development.

---

## References

- Blade Templates — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/blade
- Blade Templates — Laravel master Documentation — https://laravel.com/framework/docs/master/blade
- Blade Templates — Laravel 8.x Documentation (`@aware`) — https://laravel.com/docs/8.x/blade
- Laravel Blade Wrapper — Open VSX Registry — https://marketplace.windsurf.com
- Laravel Expert Skills: Blade Directives Reference — https://github.com/fusengine/agents
- Switch Statements in Blade — Stack Overflow — https://stackoverflow.com/questions/29818687/switch-in-laravel-5-blade
- Looping Through Data in Blade — DEV Community — https://dev.to
- Conditional Attributes in Blade — BookStack (Laravel 11.x) — https://www.bookstack.cn
- Blade `@aware` vs `@props` — Stack Overflow — https://stackoverflow.com/questions/72160400/difference-between-aware-and-props-directives-in-laravel-9
- CSRF and Method Field — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/blade
- Vite and Inertia Setup — Inertia.js Documentation — https://inertiajs.com
- Livewire and Ziggy Directives — Laracasts — https://laracasts.com

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Directives such as `@vite`, `@viteReactRefresh`, `@aware`, `@session`, and `@use` were introduced in specific Laravel versions and may not be available in older applications. `@class` and `@style` require Laravel 8.51+. `@once` and `@pushOnce`/`@prependOnce` were introduced in Laravel 7.25 and 8.x respectively. Developers should always consult the documentation for their specific Laravel version.