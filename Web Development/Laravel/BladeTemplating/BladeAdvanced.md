# Laravel Advanced Blade Features: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Advanced Blade Features encompass the templating engine's most powerful mechanisms for modularising views, managing asset pipelines, dynamically rendering components, automatically binding data to views, and extending the Blade syntax itself.

**Technical Definition:** Advanced Blade features operate at various stages of the view lifecycle: `@include` directives and their conditional variants (`@includeIf`, `@includeWhen`, `@includeUnless`) compile into PHP `include` statements for partial view rendering; `@stack`, `@push`, and `@prepend` manage named content stacks that are rendered in layout files; `<x-dynamic-component>` resolves and renders a component based on a runtime value; View Composers and View Creators use the `View` facade to bind data to views before or during rendering; and custom directives registered via `Blade::directive()` and `Blade::if()` extend the Blade compiler with application-specific syntax.

**Beginner-Friendly Explanation:** Basic Blade lets you write HTML with PHP shortcuts. Advanced Blade features let you build a modular, maintainable front-end architecture. You can split templates into reusable partials, push JavaScript and CSS from child views into the layout, render components dynamically based on data, automatically inject data into views without cluttering controllers, and even create your own Blade directives. These features are what separate a simple templating setup from a production-grade view layer.

### Key Characteristics

- **Partial Reusability:** `@include` and its variants enable DRY (Don't Repeat Yourself) view composition without the overhead of full components.
- **Asset Pipeline Management:** Stacks allow child views to inject assets (scripts, styles) into designated layout regions without coupling.
- **Runtime Component Selection:** `<x-dynamic-component>` renders components based on variable values, enabling data-driven UI composition.
- **Automatic Data Binding:** View Composers and Creators bind data to views automatically, removing that responsibility from controllers.
- **Compiler Extensibility:** `Blade::directive()` and `Blade::if()` allow developers to add custom directives that compile to PHP.
- **Performance Awareness:** Nested includes can impact performance; understanding compilation and caching is essential.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (features stable across versions; `@includeUnless` available since Laravel 8, `@prepend` since Laravel 5.4.10, View Creators since Laravel 5.5).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `resources/views` directory.
- Basic understanding of Blade syntax, service providers, and the view lifecycle.

### Related Programming Areas

- **Service Providers:** View Composers and custom directives are registered in service providers' `boot()` methods.
- **Service Container:** View Composers and Creators are resolved from the container, supporting dependency injection.
- **Component System:** `<x-dynamic-component>` integrates with the Blade component discovery and rendering system.
- **Blade Compiler:** Custom directives extend the `BladeCompiler` class.
- **View Lifecycle:** Composers and Creators hook into different stages of the view rendering lifecycle.

### Core Concepts / Features

1. Template Splitting: `@include`, `@includeIf`, `@includeWhen`
2. Stacks: `@stack`, `@push`, `@prepend`
3. Dynamic Rendering: `<x-dynamic-component>`
4. View Composers & Creators
5. Custom Directives: `Blade::directive` and `Blade::if`

---

## 1. Template Splitting: @include, @includeIf, @includeWhen

### Definitions

**Core Definition:** Template splitting via `@include` directives allows one Blade view to embed another view (a partial), promoting reusability and separation of concerns by breaking large templates into smaller, focused pieces.

**Technical Definition:** The `@include` directive compiles into a PHP `include` statement that renders the specified view within the current template's scope. All variables available to the parent view are automatically made available to the included view, and additional data can be passed as an array. The `@includeIf` variant checks if the view exists before including it, `@includeWhen` includes the view only if a boolean expression is true, `@includeUnless` includes it only if the expression is false, and `@includeFirst` includes the first view that exists from an array of candidates. The `@each` directive combines a loop with an include for rendering collections.

**Beginner-Friendly Explanation:** Think of `@include` as copy-pasting the contents of another file into your current file—but without actually copy-pasting. If you have a navigation bar that appears on every page, you write it once in a `navigation.blade.php` file and `@include` it wherever you need it. If you change the navigation, it updates everywhere. The conditional variants (`@includeIf`, `@includeWhen`) are like saying "include this only if it exists" or "include this only if a certain condition is true."

### Purposes

- To reuse common view fragments (headers, footers, sidebars, form fields) across multiple templates.
- To pass data to partials from the parent view, either by inheriting all parent variables or by passing an explicit array.
- To conditionally include views using `@includeIf`, `@includeWhen`, and `@includeUnless`.
- To include the first existing view from a list of candidates using `@includeFirst`.
- To render collections efficiently using `@each`.
- To reduce duplication and improve maintainability of large templates.

### Syntax Rules and Structure

**Complete General Syntax — `@include`:**

```blade
@include('shared.errors')
@include('view.name', ['some' => 'data'])
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@include('view.name')` | Includes the specified view. |
| `['some' => 'data']` | Optional array of additional data passed to the view. |
| Inherited variables | All parent view variables are automatically available. |

**Complete General Syntax — Conditional Includes:**

```blade
@includeIf('view.name', ['some' => 'data'])
@includeWhen($boolean, 'view.name', ['some' => 'data'])
@includeUnless($boolean, 'view.name', ['some' => 'data'])
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@includeIf` | Includes the view only if it exists. |
| `@includeWhen` | Includes the view if `$boolean` is true. |
| `@includeUnless` | Includes the view if `$boolean` is false. |
| `@includeFirst` | Includes the first existing view from the array. |

**Complete General Syntax — `@each`:**

```blade
@each('view.name', $jobs, 'job')
@each('view.name', $jobs, 'job', 'view.empty')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'view.name'` | The partial to render for each element. |
| `$jobs` | The collection to iterate over. |
| `'job'` | The variable name for the current element. |
| `'view.empty'` | The view rendered if the collection is empty. |

**Syntax Rules:**

- Included views inherit all variables from the parent view unless explicitly overridden.
- The `@include` directive throws an error if the view does not exist; use `@includeIf` for optional views.
- `@includeWhen` and `@includeUnless` evaluate a boolean expression to determine inclusion.
- `@includeFirst` accepts an array of view names and includes the first one that exists.
- `@each` does not inherit parent variables—the partial receives only the current element and key.
- Views rendered via `@each` do not have access to the parent view's variables.

**Constraints and Limitations:**

- **Performance:** Nested includes can impact performance—each include adds a function call and file lookup. Deeply nested includes should be flattened where possible.
- **Variable scope:** Included views inherit all parent variables, which can lead to variable name collisions. Be careful with variable naming in partials.
- **`@each` scope:** Views rendered via `@each` do not inherit parent variables—use `@foreach` with `@include` if the partial needs parent data.
- **Error handling:** `@include` throws an error for missing views; `@includeIf` silently skips missing views.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `@include` with Data Passing**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<div class="container">
    @include('partials.header', ['title' => 'Welcome'])

    <main>
        @yield('content')
    </main>

    @include('partials.footer')
</div>
```

```blade
{{-- resources/views/partials/header.blade.php --}}
<header>
    <h1>{{ $title }}</h1>
    <nav>@include('partials.navigation')</nav>
</header>
```

**Expected Output:** The layout renders the header partial with the `$title` variable set to `'Welcome'`. The header partial includes the navigation partial, which inherits all variables from the header partial (including `$title`).

**Why:** The `@include` directive renders the partial within the parent view's scope. The explicit `['title' => 'Welcome']` array passes additional data to the header partial.

---

**Example 2: Conditional Includes**

```blade
{{-- resources/views/dashboard.blade.php --}}

@includeIf('partials.welcome-banner')

@includeWhen($user->isNew(), 'partials.onboarding-tips')

@includeUnless($user->isVerified(), 'partials.verify-email')

@includeFirst(['custom.navigation', 'navigation'])
```

**Expected Output:**
- The welcome banner is included only if `partials/welcome-banner.blade.php` exists.
- The onboarding tips are included only if the user is new.
- The email verification prompt is included only if the user is not verified.
- The custom navigation is included if it exists; otherwise, the default navigation is included.

**Why:** Each conditional include directive evaluates its condition or existence check before rendering, allowing the view to adapt to different states without cluttering the template with `@if` blocks.

---

**Example 3: `@each` for Rendering Collections**

```blade
{{-- resources/views/jobs/index.blade.php --}}
<h1>Job Listings</h1>

@each('partials.job', $jobs, 'job', 'partials.no-jobs')
```

```blade
{{-- resources/views/partials/job.blade.php --}}
<div class="job">
    <h3>{{ $job->title }}</h3>
    <p>{{ $job->description }}</p>
</div>
```

```blade
{{-- resources/views/partials/no-jobs.blade.php --}}
<p class="text-muted">No job listings found.</p>
```

**Expected Output:** If `$jobs` has items, each job is rendered using the `partials.job` view. If the collection is empty, the `partials.no-jobs` view is rendered.

**Why:** The `@each` directive combines the loop and the include into a single, readable statement. The fourth argument provides the empty state view.

### Real-World Cases

- **Navigation Bars:** `@include('partials.navigation')` on every page.
- **Form Fields:** `@include('partials.form.input', ['name' => 'email', 'type' => 'email'])` for reusable form inputs.
- **Error Messages:** `@includeIf('partials.errors')` to conditionally display validation errors.
- **Empty States:** `@each('partials.product', $products, 'product', 'partials.empty-products')` for product listings.
- **Widgets:** `@includeWhen($showSidebar, 'partials.sidebar')` for conditionally displayed sidebars.

### References

- Laravel Blade: Including Subviews — https://laravel.com/docs/12.x/blade#including-subviews
- Laravel Blade: Rendering Views for Collections — https://laravel.com/docs/12.x/blade#rendering-views-for-collections
- Laravel Blade: Aliasing Includes — https://laravel.com/docs/12.x/blade#aliasing-includes
- Blade includes vs components: how we decide (No Compromises) — https://share.transistor.fm

---

## 2. Stacks: @stack, @push, @prepend

### Definitions

**Core Definition:** Stacks are named content buffers that allow child views to inject content (typically JavaScript or CSS) into specific regions of a layout, with `@push` appending to the end of the stack and `@prepend` prepending to the beginning, rendered via `@stack` in the layout.

**Technical Definition:** The `@push` directive compiles to a call that appends its content to a named stack in the `ViewFactory`'s stack registry. The `@prepend` directive prepends to the same stack. The `@stack` directive renders all accumulated content for the named stack at the point where it appears in the layout. Stacks are resolved at render time, and content can be pushed from any view that is rendered before or during the layout render. Unlike `@section`/`@yield` (which is one-to-one content replacement), `@push`/`@stack` collects multiple pieces from multiple views.

**Beginner-Friendly Explanation:** Imagine your layout has a `<head>` section and a `<body>` section. A child view might need to add a specific JavaScript file or CSS style. Instead of restructuring the layout to add `@yield` sections everywhere, the child view can `@push` content onto a named stack (like `scripts`), and the layout renders that stack with `@stack('scripts')`. Multiple views can push to the same stack, and all their content is collected and rendered together. It's like having a collection basket that different views contribute to, and the layout decides where to empty it.

### Purposes

- To allow child views to inject JavaScript and CSS assets into designated layout regions without modifying the layout.
- To collect content from multiple views into a single rendered location.
- To control the order of stacked content using `@push` (append) and `@prepend` (prepend).
- To decouple asset management from layout structure, improving maintainability.
- To support conditional asset loading based on view-specific needs.

### Syntax Rules and Structure

**Complete General Syntax — `@push`:**

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@push('scripts')` | Starts pushing content to the `scripts` stack. |
| `@endpush` | Ends the push block. |

**Complete General Syntax — `@prepend`:**

```blade
@prepend('scripts')
    <script src="/important.js"></script>
@endprepend
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@prepend('scripts')` | Prepends content to the beginning of the `scripts` stack. |
| `@endprepend` | Ends the prepend block. |

**Complete General Syntax — `@stack`:**

```blade
<head>
    @stack('styles')
</head>
<body>
    @yield('content')
    @stack('scripts')
</body>
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@stack('styles')` | Renders all content pushed to the `styles` stack. |
| `@stack('scripts')` | Renders all content pushed to the `scripts` stack. |

**Syntax Rules:**

- Stacks are rendered where the `@stack` directive appears in the layout.
- Content pushed to a stack can come from any view rendered before or during the layout render.
- `@push` appends to the end of the stack; `@prepend` prepends to the beginning.
- The same stack can be pushed to multiple times from multiple views.
- Stacks are resolved at render time, so order matters—views pushed earlier are rendered first.

**Constraints and Limitations:**

- **Render order:** Content pushed to a stack is rendered in the order it was pushed. If a child view pushes content before the layout renders the stack, it appears in the correct position.
- **`@prepend` availability:** The `@prepend` directive was introduced in Laravel 5.4.10—earlier versions only support `@push`.
- **Stack naming:** Stack names must be unique and descriptive (e.g., `scripts`, `styles`, `modals`).
- **No conditional rendering:** Stacks are always rendered if they have content; there is no built-in "empty" fallback.

### Multiple Annotated Complete Code Examples

**Example 1: Pushing Scripts to the Layout**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title')</title>
    @stack('styles')
</head>
<body>
    @yield('content')

    @stack('scripts')
</body>
</html>
```

```blade
{{-- resources/views/dashboard.blade.php --}}
@extends('layouts.app')

@section('title', 'Dashboard')

@push('styles')
    <link rel="stylesheet" href="/css/dashboard.css">
@endpush

@section('content')
    <h1>Dashboard</h1>
@endsection

@push('scripts')
    <script src="/js/chart.js"></script>
    <script>
        new Chart(document.getElementById('chart'), { type: 'bar', data: @json($chartData) });
    </script>
@endpush
```

**Expected Output:** The layout's `<head>` includes the dashboard CSS, and the bottom of the `<body>` includes the chart JavaScript. Both are injected from the child view without modifying the layout.

**Why:** The child view pushes CSS to the `styles` stack and JavaScript to the `scripts` stack. The layout renders both stacks at the appropriate positions. The layout does not need to know which specific assets the child view requires.

---

**Example 2: Using `@prepend` to Control Order**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<head>
    @stack('scripts')
</head>
```

```blade
{{-- resources/views/child.blade.php --}}
@push('scripts')
    <script src="/js/second.js"></script>
@endpush

@prepend('scripts')
    <script src="/js/first.js"></script>
@endprepend
```

**Expected Output:** The rendered `<head>` contains `/js/first.js` followed by `/js/second.js`. The prepended script appears before the pushed script.

**Why:** `@prepend` adds content to the beginning of the stack, while `@push` adds to the end. This allows critical scripts to be loaded before non-critical ones.

---

**Example 3: Multiple Views Pushing to the Same Stack**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<footer>
    @stack('modals')
</footer>
```

```blade
{{-- resources/views/partials/modal-a.blade.php --}}
@push('modals')
    <div id="modal-a" class="modal">Modal A content</div>
@endpush
```

```blade
{{-- resources/views/partials/modal-b.blade.php --}}
@push('modals')
    <div id="modal-b" class="modal">Modal B content</div>
@endpush
```

```blade
{{-- resources/views/page.blade.php --}}
@extends('layouts.app')

@section('content')
    @include('partials.modal-a')
    @include('partials.modal-b')
@endsection
```

**Expected Output:** The footer contains both modal A and modal B, in the order they were pushed (modal A first, then modal B).

**Why:** Each partial pushes its modal content to the `modals` stack. The layout renders all accumulated content at the `@stack('modals')` location, preserving the push order.

### Real-World Cases

- **JavaScript Libraries:** Child views push chart libraries, form validators, or map scripts to the `scripts` stack.
- **CSS Overrides:** Page-specific CSS is pushed to the `styles` stack.
- **Modals:** Modal components are pushed to a `modals` stack rendered at the end of the body.
- **Meta Tags:** SEO-specific meta tags are pushed to a `meta` stack rendered in the `<head>`.
- **Analytics:** Tracking scripts are prepended to the `scripts` stack to ensure they load first.

### References

- Laravel Blade: Stacks — https://laravel.com/docs/12.x/blade#stacks
- Laravel Blade: @prepend Directive (Laravel News) — https://laravel-news.com
- Laravel Blade Directives Clarified: @section, @yield, @push, @stack, @prepend (LinkedIn) — https://www.linkedin.com

---

## 3. Dynamic Rendering: `<x-dynamic-component>`

### Definitions

**Core Definition:** The `<x-dynamic-component>` component renders a Blade component based on a runtime value, allowing the component to be selected dynamically rather than being hardcoded in the template.

**Technical Definition:** The `<x-dynamic-component>` built-in component accepts a `:component` attribute whose value is a string (or expression resolving to a string) containing the component name. At render time, Laravel resolves the component name against the registered component namespace and renders the corresponding component, passing any additional attributes and slots to it. This enables data-driven UI composition where the component to render is determined at runtime based on user role, configuration, or data state.

**Beginner-Friendly Explanation:** Normally, you write `<x-alert>` to render a specific component. But what if you want to render different components depending on a variable? For example, an admin might see a `<x-admin-panel>` while a regular user sees a `<x-user-panel>`. Instead of writing a giant `@if` block, you use `<x-dynamic-component :component="$panelComponent">` and set `$panelComponent` to the appropriate component name. It's like having a universal remote that can control any TV—you just tell it which one.

### Purposes

- To render components dynamically based on runtime values (user role, configuration, data type).
- To avoid verbose `@if`/`@else` blocks when the component selection is data-driven.
- To enable plugin or module systems where the component to render is determined by configuration.
- To support polymorphic rendering where different data types map to different UI components.
- To facilitate testing by allowing component selection to be injected.

### Syntax Rules and Structure

**Complete General Syntax:**

```blade
<x-dynamic-component :component="$componentName" class="mt-4" />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `:component="$componentName"` | The expression resolving to the component name (e.g., `'alert'`, `'user-profile'`). |
| `class="mt-4"` | Additional attributes passed to the rendered component. |
| Slots | Default and named slots are passed through. |

**Complete General Syntax — With Slots:**

```blade
<x-dynamic-component :component="$componentName">
    <x-slot:header>
        Dynamic Header
    </x-slot>

    Dynamic content goes here.
</x-dynamic-component>
```

**Complete General Syntax — With Attribute Bag:**

```blade
@php
    use Illuminate\View\ComponentAttributeBag;
@endphp

<x-dynamic-component
    :component="$componentName"
    :attributes="new ComponentAttributeBag($props)"
/>
```

**Syntax Rules:**

- The `:component` attribute must be a string or an expression resolving to a string.
- The component name should use kebab-case (e.g., `'user-profile'` for `UserProfile` component).
- Additional attributes (e.g., `class`, `id`) are passed through to the rendered component.
- Slots (default and named) are passed through to the rendered component.
- The component must be discoverable by Laravel's component resolver.

**Constraints and Limitations:**

- **Component discovery:** The dynamic component name must resolve to a registered component. Anonymous components and class-based components are both supported.
- **Attribute bag:** When passing an array of attributes, you must wrap it in a `ComponentAttributeBag` instance.
- **Performance:** Dynamic component resolution adds a small overhead compared to static components.
- **Security:** Never pass user input directly as the component name without validation—this could allow rendering of unintended components.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Dynamic Component Rendering**

```blade
{{-- resources/views/dashboard.blade.php --}}
@php
    $widgetComponent = match($user->role) {
        'admin' => 'admin-dashboard',
        'editor' => 'editor-dashboard',
        default => 'user-dashboard',
    };
@endphp

<x-dynamic-component :component="$widgetComponent" :user="$user" />
```

**Expected Output:** If the user is an admin, the `admin-dashboard` component is rendered with the `$user` prop. If an editor, the `editor-dashboard` component is rendered. Otherwise, the `user-dashboard` component is rendered.

**Why:** The `match` expression determines the component name based on the user's role. The `<x-dynamic-component>` tag renders the appropriate component and passes the `$user` attribute to it.

---

**Example 2: Dynamic Component with Slots**

```blade
{{-- resources/views/notifications.blade.php --}}
@foreach($notifications as $notification)
    <x-dynamic-component :component="$notification->type">
        <x-slot:title>{{ $notification->title }}</x-slot>
        {{ $notification->message }}
    </x-dynamic-component>
@endforeach
```

```blade
{{-- resources/views/components/success-notification.blade.php --}}
<div class="alert alert-success">
    <strong>{{ $title }}</strong>
    {{ $slot }}
</div>
```

```blade
{{-- resources/views/components/error-notification.blade.php --}}
<div class="alert alert-danger">
    <strong>{{ $title }}</strong>
    {{ $slot }}
</div>
```

**Expected Output:** Each notification is rendered with the component matching its `type` (`success-notification` or `error-notification`). The title slot and message content are passed through.

**Why:** The `$notification->type` value determines which component is rendered. The slot content is passed to the dynamic component and rendered within the component's template.

---

**Example 3: Dynamic Component with Attribute Bag**

```blade
{{-- resources/views/flexible-widget.blade.php --}}
@php
    use Illuminate\View\ComponentAttributeBag;

    $props = [
        'class' => 'widget widget-large',
        'data-id' => $widget->id,
        'title' => $widget->title,
    ];
@endphp

<x-dynamic-component
    :component="$widget->component"
    :attributes="new ComponentAttributeBag($props)"
/>
```

**Expected Output:** The component specified by `$widget->component` is rendered with the attributes from the `$props` array.

**Why:** The `ComponentAttributeBag` wraps the attribute array so it can be passed to the dynamic component, allowing complex attribute sets to be constructed programmatically.

### Real-World Cases

- **Role-Based Dashboards:** Different dashboard components for admins, editors, and regular users.
- **Polymorphic Notifications:** `success-notification`, `error-notification`, `warning-notification` components rendered dynamically.
- **Plugin Systems:** Widgets or modules rendered based on plugin configuration.
- **Multi-Tenant Applications:** Different UI components for different tenant tiers.
- **Form Field Types:** Dynamic form field components based on field type (text, select, radio).

### References

- Laravel Blade: Dynamic Components — https://laravel.com/docs/12.x/blade#dynamic-components
- Laravel Blade: Manually Registering Components — https://laravel.com/docs/12.x/blade#manually-registering-components
- GitHub Discussion: Passing Props from x-dynamic-component — https://github.com/laravel/framework/discussions/51921

---

## 4. View Composers & Creators

### Definitions

**Core Definition:** View Composers are callbacks or class methods that bind data to views automatically when they are rendered, while View Creators execute immediately when views are instantiated, providing an earlier injection point for data preparation.

**Technical Definition:** View Composers are registered via the `View::composer()` method in a service provider's `boot()` method. They receive a `View` instance and can call `$view->with('key', $value)` to bind data. Composers execute just before the view is rendered. View Creators, registered via `View::creator()`, execute immediately after the view is instantiated, before any rendering begins. Both are resolved through the service container, supporting constructor dependency injection. They can target specific views, view patterns (`*`), or multiple views at once.

**Beginner-Friendly Explanation:** Normally, your controller passes data to a view. But sometimes multiple views need the same data—like a sidebar that shows the same categories on every page. Instead of repeating the data-binding logic in every controller, you create a View Composer that automatically injects that data whenever a particular view (or group of views) is rendered. View Creators are similar but run even earlier—as soon as the view object is created. This keeps your controllers clean and ensures consistent data availability.

### Purposes

- To bind data to specific views or partials automatically across the application.
- To remove data-binding logic from controllers, keeping them focused on request handling.
- To share data across multiple views (e.g., sidebar widgets, navigation menus) without repetition.
- To prepare data earlier in the lifecycle via View Creators (before rendering begins).
- To use dependency injection in view data preparation via the service container.
- To target all views (`*`) for global data sharing.

### Syntax Rules and Structure

**Complete General Syntax — View Composer (Closure):**

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\View;

View::composer('profile', function (View $view) {
    $view->with('user', Auth::user());
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `View::composer('profile', ...)` | Registers a composer for the `profile` view. |
| `function (View $view)` | Callback receives the view instance. |
| `$view->with('user', ...)` | Binds data to the view. |

**Complete General Syntax — View Composer (Class-Based):**

```php
// app/View/Composers/ProfileComposer.php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    public function __construct(
        private UserRepository $users
    ) {}

    public function compose(View $view): void
    {
        $view->with('user', $this->users->find(auth()->id()));
    }
}
```

```php
// In AppServiceProvider::boot()
use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;

View::composer('profile', ProfileComposer::class);
```

**Complete General Syntax — View Creator:**

```php
// In AppServiceProvider::boot()
View::creator('dashboard', function (View $view) {
    $view->with('metrics', app(MetricsService::class)->all());
});

// Or class-based
View::creator('dashboard', UserDataCreator::class);
```

**Complete General Syntax — Multiple Views:**

```php
View::composer(['profile', 'dashboard'], ProfileComposer::class);
View::composer(['profile', 'dashboard'], function (View $view) { /* ... */ });
```

**Complete General Syntax — All Views:**

```php
View::composer('*', function (View $view) {
    $view->with('currentUser', Auth::user());
});
```

**Syntax Rules:**

- Composers and Creators are registered in a service provider's `boot()` method.
- `View::composer()` accepts a view name, array of view names, or `*` for all views.
- Class-based composers/creators are resolved from the service container, supporting constructor injection.
- The `compose()` method (for classes) or the closure receives the `View` instance.
- View Creators execute immediately after view instantiation; View Composers execute just before rendering.
- Both use `$view->with()` to bind data.

**Constraints and Limitations:**

- **Timing difference:** View Creators execute earlier (after instantiation) than View Composers (before rendering). Use Creators when data must be available throughout the view's lifecycle.
- **Wildcard performance:** Using `*` for all views can impact performance. Target specific views where possible.
- **Service provider registration:** If using a dedicated service provider for composers, it must be registered in `config/app.php` (Laravel 10) or `bootstrap/providers.php` (Laravel 11+).
- **Testing:** Composers and Creators can make view testing more complex due to their automatic execution.

### Multiple Annotated Complete Code Examples

**Example 1: Closure-Based View Composer**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Models\Category;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Share categories with the sidebar partial
        View::composer('partials.sidebar', function ($view) {
            $view->with('categories', Category::withCount('posts')->get());
        });
    }
}
```

```blade
{{-- resources/views/partials/sidebar.blade.php --}}
<aside class="sidebar">
    <h3>Categories</h3>
    <ul>
        @foreach($categories as $category)
            <li>{{ $category->name }} ({{ $category->posts_count }})</li>
        @endforeach
    </ul>
</aside>
```

**Expected Output:** Whenever the `partials.sidebar` view is rendered, the `$categories` variable is automatically available with all categories and their post counts.

**Why:** The View Composer binds the `categories` data to the `partials.sidebar` view. The view does not need to receive this data from the controller—it is injected automatically.

---

**Example 2: Class-Based View Composer with Dependency Injection**

```php
<?php
// app/View/Composers/DashboardComposer.php

namespace App\View\Composers;

use App\Services\MetricsService;
use Illuminate\View\View;

class DashboardComposer
{
    public function __construct(
        private MetricsService $metrics
    ) {}

    public function compose(View $view): void
    {
        $view->with([
            'totalUsers' => $this->metrics->totalUsers(),
            'totalRevenue' => $this->metrics->totalRevenue(),
        ]);
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\View\Composers\DashboardComposer;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        View::composer('dashboard', DashboardComposer::class);
    }
}
```

**Expected Output:** The dashboard view automatically receives `$totalUsers` and `$totalRevenue` variables from the injected `MetricsService`.

**Why:** The composer class is resolved from the service container, so the `MetricsService` is injected automatically. The `compose()` method binds the metrics data to the view.

---

**Example 3: View Creator for Early Data Injection**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\SidebarService;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Creator runs immediately after view instantiation
        View::creator('layouts.app', function ($view) {
            $view->with('sidebarWidgets', app(SidebarService::class)->widgets());
        });
    }
}
```

```blade
{{-- resources/views/layouts/app.blade.php --}}
<div class="layout">
    <aside>
        @foreach($sidebarWidgets as $widget)
            @include($widget['view'], $widget['data'])
        @endforeach
    </aside>

    <main>@yield('content')</main>
</div>
```

**Expected Output:** The `$sidebarWidgets` data is available as soon as the layout view is instantiated, before rendering begins. The sidebar renders all widgets.

**Why:** The View Creator executes earlier than a View Composer would, ensuring the data is available throughout the view's lifecycle. This is useful when the view's constructor or other early methods need the data.

### Real-World Cases

- **Sidebar Widgets:** Categories, popular posts, or recent comments injected into sidebars via composers.
- **Navigation Menus:** Dynamic menu items based on user permissions via composers.
- **Dashboard Metrics:** Aggregated statistics injected into dashboards via creators.
- **Global Notifications:** System-wide notifications injected into all views via `View::composer('*', ...)`.
- **User Context:** Current user data injected into profile-related views.

### References

- Laravel Views: View Composers — https://laravel.com/docs/12.x/views#view-composers
- Laravel Views: View Creators — https://laravel.com/docs/12.x/views#view-creators
- Early View Data Preparation with Laravel View Creators (Laravel News) — https://laravel-news.com/view-creators
- Stack Overflow: Difference between View Composer and Creator — https://stackoverflow.com/questions/19052921

---

## 5. Custom Directives: Blade::directive and Blade::if

### Definitions

**Core Definition:** Custom Blade directives are application-specific syntax extensions registered via `Blade::directive()` (for arbitrary directives) or `Blade::if()` (for conditional directives), allowing developers to add reusable, expressive shortcuts to the Blade compiler.

**Technical Definition:** The `Blade::directive($name, $callback)` method registers a custom directive. When the Blade compiler encounters `@name(expression)`, it calls the callback with the raw expression string and replaces the directive with the PHP code returned by the callback. The `Blade::if($name, $callback)` method registers a custom conditional directive that generates `@name`, `@elseName`, and `@endName` directives, with the callback's boolean return value determining the conditional outcome. Both methods are registered in a service provider's `boot()` method and require clearing the view cache (`php artisan view:clear`) after changes.

**Beginner-Friendly Explanation:** Blade comes with directives like `@if`, `@foreach`, and `@auth`. Custom directives let you create your own. For example, you could create `@datetime($var)` to format a date, or `@admin` to check if the user is an admin. The `Blade::directive()` method is for general-purpose directives that output PHP code. The `Blade::if()` method is specifically for conditional directives—it automatically generates the `@if`, `@else`, and `@endif` equivalents. It's like teaching Blade a new word that it understands.

### Purposes

- To extend the Blade compiler with application-specific directives.
- To encapsulate repetitive view logic into reusable, expressive directives.
- To create custom conditional directives via `Blade::if()` that generate `@if`/`@else`/`@endif` variants.
- To improve template readability by replacing verbose PHP with concise directives.
- To centralise view helper logic in service providers for maintainability.

### Syntax Rules and Structure

**Complete General Syntax — `Blade::directive()`:**

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Blade;

Blade::directive('datetime', function (string $expression) {
    return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Blade::directive('datetime', ...)` | Registers a directive named `datetime`. |
| `function (string $expression)` | Receives the raw expression from the template. |
| Return value | PHP code that replaces the directive. |

**Usage:**

```blade
@datetime($user->created_at)
```

**Compiled Output:**

```php
<?php echo ($user->created_at)->format('m/d/Y H:i'); ?>
```

**Complete General Syntax — `Blade::if()`:**

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Blade;

Blade::if('admin', function () {
    return auth()->check() && auth()->user()->isAdmin();
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Blade::if('admin', ...)` | Registers a conditional directive named `admin`. |
| `function () { return ...; }` | Returns a boolean value. |
| Generated directives | `@admin`, `@elseadmin`, `@endadmin`. |

**Usage:**

```blade
@admin
    <p>Admin panel</p>
@elseadmin
    <p>User panel</p>
@endadmin
```

**Complete General Syntax — `Blade::if()` with Parameters:**

```php
Blade::if('role', function (string $role) {
    return auth()->check() && auth()->user()->hasRole($role);
});
```

```blade
@role('admin')
    <p>Admin area</p>
@endrole
```

**Syntax Rules:**

- Custom directives are registered in a service provider's `boot()` method.
- `Blade::directive()` callbacks receive the raw expression string and must return PHP code.
- `Blade::if()` automatically generates `@name`, `@elseName`, and `@endName` directives.
- The `Blade::if()` callback should return a boolean value.
- After registering or modifying a directive, run `php artisan view:clear` to clear cached views.
- Directives can accept parameters, which are passed to the callback.

**Constraints and Limitations:**

- **Raw expression:** `Blade::directive()` callbacks receive the expression as a raw string (including quotes)—you must handle it appropriately in the returned PHP code.
- **No variable scope:** Custom directives do not have access to the view's variables directly—they operate on the expression passed to them.
- **Cache clearing required:** After changing directive logic, the view cache must be cleared.
- **Security:** Be cautious when using user input in custom directives—validate and sanitize appropriately.

### Multiple Annotated Complete Code Examples

**Example 1: Custom `@datetime` Directive**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

```blade
{{-- Usage --}}
<p>Created: @datetime($post->created_at)</p>
<p>Updated: @datetime($post->updated_at)</p>
```

**Expected Output:** The post's creation date is rendered as `01/15/2025 14:30`, and the update date similarly formatted.

**Why:** The `@datetime` directive compiles to PHP code that calls `format()` on the passed expression. The expression is the raw string `$post->created_at`, which is inserted into the PHP code.

---

**Example 2: Custom `@admin` Conditional Directive**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Blade::if('admin', function () {
            return auth()->check() && auth()->user()->isAdmin();
        });

        Blade::if('role', function (string $role) {
            return auth()->check() && auth()->user()->hasRole($role);
        });
    }
}
```

```blade
{{-- Usage --}}
@admin
    <a href="/admin">Admin Panel</a>
@elseadmin
    <a href="/dashboard">Dashboard</a>
@endadmin

@role('editor')
    <a href="/editor">Editor Tools</a>
@endrole
```

**Expected Output:** If the user is an admin, the admin panel link is displayed. Otherwise, the dashboard link is displayed. If the user has the `editor` role, the editor tools link is displayed.

**Why:** The `Blade::if()` method creates the `@admin`/`@elseadmin`/`@endadmin` and `@role`/`@endrole` directives. The callback's boolean return value determines which branch renders.

---

**Example 3: Custom `@money` Directive**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Blade::directive('money', function (string $expression) {
            return "<?php echo '$' . number_format($expression / 100, 2); ?>";
        });
    }
}
```

```blade
{{-- Usage --}}
<p>Total: @money($order->total_in_cents)</p>
```

**Expected Output:** If `$order->total_in_cents` is `12500`, the output is `Total: $125.00`.

**Why:** The directive divides the cents value by 100 and formats it as a dollar amount with two decimal places.

### Real-World Cases

- **Date Formatting:** `@datetime($timestamp)` for consistent date rendering across views.
- **Currency Display:** `@money($cents)` for formatting prices from integer cents.
- **Role Checks:** `@admin`, `@editor`, `@subscriber` for role-based content visibility.
- **Environment Checks:** `@staging` for showing staging-specific debug information.
- **Feature Flags:** `@feature('new-dashboard')` for conditionally rendering features.

### References

- Laravel Blade: Custom Directives — https://laravel.com/docs/12.x/blade#extending-blade
- Laravel Blade: Custom If Statements — https://laravel.com/docs/12.x/blade#custom-if-statements
- Laravel News: Blade::if() Directives — https://laravel-news.com
- GitHub Discussion: Blade Directive Expression Handling — https://github.com/laravel/framework/discussions/45026

---

## References

- Laravel Blade Templates Documentation (12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade: Including Subviews — https://laravel.com/docs/12.x/blade#including-subviews
- Laravel Blade: Rendering Views for Collections — https://laravel.com/docs/12.x/blade#rendering-views-for-collections
- Laravel Blade: Stacks — https://laravel.com/docs/12.x/blade#stacks
- Laravel Blade: Dynamic Components — https://laravel.com/docs/12.x/blade#dynamic-components
- Laravel Views: View Composers — https://laravel.com/docs/12.x/views#view-composers
- Laravel Views: View Creators — https://laravel.com/docs/12.x/views#view-creators
- Laravel Blade: Extending Blade (Custom Directives) — https://laravel.com/docs/12.x/blade#extending-blade
- Laravel Blade: Custom If Statements — https://laravel.com/docs/12.x/blade#custom-if-statements
- Early View Data Preparation with Laravel View Creators (Laravel News) — https://laravel-news.com/view-creators
- Laravel News: Blade::if() Directives — https://laravel-news.com
- Laravel Blade: @prepend Directive (Laravel News) — https://laravel-news.com
- Laravel Blade Directives Clarified: @section, @yield, @push, @stack, @prepend (LinkedIn) — https://www.linkedin.com
- Blade includes vs components: how we decide (No Compromises) — https://share.transistor.fm
- Stack Overflow: Difference between View Composer and Creator — https://stackoverflow.com/questions/19052921
- GitHub Discussion: Passing Props from x-dynamic-component — https://github.com/laravel/framework/discussions/51921
- GitHub Discussion: Blade Directive Expression Handling — https://github.com/laravel/framework/discussions/45026
- Stack Overflow: Laravel 5.0 Blade including tag causes slow render time — https://stackoverflow.com/questions/46715280