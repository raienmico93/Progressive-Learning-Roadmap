# Laravel Blade Components & Modern Layouts: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Blade components are self-contained, reusable UI building blocks that encapsulate markup, logic, and styling into a single unit, enabling developers to compose complex interfaces from small, maintainable pieces while supporting modern layout architectures that replace or complement traditional template inheritance.

**Technical Definition:** Blade components are implemented through two primary mechanisms: **class-based components** (a PHP class extending `Illuminate\View\Component` paired with a Blade view) and **anonymous components** (a standalone Blade template with no associated PHP class). Both are rendered via the `<x-component-name>` tag syntax. Components receive data through HTML attributes (resolved via constructor parameters or `@props`), expose content through **slots** (`$slot` for default content, `<x-slot:name>` for named content), and pass through arbitrary HTML attributes via the `$attributes` bag. Class-based components leverage Laravel's service container for dependency injection, while anonymous components declare expected props via the `@props` directive.

**Beginner-Friendly Explanation:** Think of Blade components as LEGO bricks for your website. Instead of copy-pasting the same HTML for every button, alert, or card, you build a reusable "brick" once and use it everywhere. If you need to change how all buttons look, you change the brick—not every page. Laravel gives you two ways to build these bricks: you can make a simple one that's just HTML (anonymous component), or a more powerful one with PHP logic behind it (class-based component). Both can accept data and content, making them flexible enough for almost any UI need.

### Key Characteristics

- **Two Component Types:** Anonymous components (Blade-only, no class) and class-based components (PHP class + Blade view), each suited to different complexity levels.
- **Automatic Discovery:** Components in `app/View/Components` and `resources/views/components` are auto-discovered without manual registration.
- **Slot-Based Content Injection:** Default slots (`$slot`) and named slots (`<x-slot:name>`) allow passing content into components.
- **Attribute Bag:** The `$attributes` variable exposes all HTML attributes passed to the component, enabling class merging and attribute passthrough.
- **Dependency Injection:** Class-based components support constructor and method injection via Laravel's service container.
- **Layout Flexibility:** Components can serve as layout wrappers, offering an alternative to `@extends`/`@section`/`@yield` inheritance.
- **Modern Layout Patterns:** Layout components provide a mental model that many developers find easier to understand than traditional template inheritance.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (components are stable across versions; `@props`, `@aware`, and anonymous components available since Laravel 7).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `resources/views` directory.
- Basic understanding of Blade templates, HTML, and PHP classes.

### Related Programming Areas

- **Blade Templating:** Components are built on top of Blade's compilation engine.
- **Service Container:** Class-based components leverage the IoC container for dependency injection.
- **Layout Patterns:** Components enable modern layout architectures that complement or replace `@extends`/`@section`/`@yield`.
- **UI Libraries:** Components are the foundation for UI kits (Blade UI Kit, Tailwind UI, etc.).
- **Frontend Frameworks:** Components can wrap Vue, React, or Alpine.js components.

### Core Concepts / Features

1. Layout Patterns: Class-Based vs. Traditional Inheritance
2. Anonymous Components
3. Class-Based Components
4. Component Attributes
5. Slots Architecture

---

## 1. Layout Patterns: Class-Based vs. Traditional Inheritance

### Definitions

**Core Definition:** Laravel supports two primary layout patterns: **traditional template inheritance** using `@extends`, `@section`, and `@yield` directives, and **component-based layouts** using layout components with slots, each offering different mental models for structuring page templates.

**Technical Definition:** Traditional inheritance uses Blade's `@extends` directive to specify a parent layout, `@section` to define named content blocks in child views, and `@yield` to render those sections in the parent layout. Component-based layouts use a layout component (`<x-layout>`) that receives page content through the default slot and named slots for sections like `title`, `header`, and `footer`. The component-based approach treats layouts as just another component, leveraging the same slot and attribute mechanisms used for UI components.

**Beginner-Friendly Explanation:** Traditional inheritance is like filling out a form—the parent layout has blanks (`@yield`) that child pages fill in (`@section`). Component-based layouts are like putting your content inside a box—the layout component is the box, and your page content goes inside it. Both achieve the same result, but the component approach uses the same pattern you already use for buttons, cards, and alerts, so there's less to learn.

### Purposes

- To provide structured, reusable page templates that maintain consistent layouts across an application.
- To separate page structure (header, footer, navigation) from page content.
- To enable a single mental model (components) for both UI elements and page layouts.
- To support multiple named content areas (title, header, footer) within a layout.
- To allow layouts to receive data (e.g., page title, meta tags) through typed props.
- To integrate seamlessly with class-based components and dependency injection.

### Syntax Rules and Structure

**Complete General Syntax — Traditional Inheritance:**

```blade
{{-- layouts/app.blade.php --}}
<html>
<head>
    <title>@yield('title', 'Default Title')</title>
</head>
<body>
    <header>@yield('header')</header>
    <main>@yield('content')</main>
    <footer>@yield('footer')</footer>
</body>
</html>
```

```blade
{{-- pages/home.blade.php --}}
@extends('layouts.app')

@section('title', 'Home Page')

@section('content')
    <h1>Welcome to the home page</h1>
@endsection

@section('header')
    <nav>...</nav>
@endsection
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@extends('layouts.app')` | Specifies the parent layout. |
| `@section('name')` | Defines a named content block. |
| `@yield('name')` | Renders the content of a named section. |
| `@endsection` | Ends a section definition. |

**Complete General Syntax — Component-Based Layout:**

```blade
{{-- components/layout.blade.php --}}
@props([
    'title' => 'Default Title',
    'metaDescription' => null,
])

<html>
<head>
    <title>{{ $title }}</title>
    @if($metaDescription)
        <meta name="description" content="{{ $metaDescription }}">
    @endif
</head>
<body>
    @if(isset($header))
        <header {{ $header->attributes->class(['site-header']) }}>
            {{ $header }}
        </header>
    @endif

    <main {{ $attributes->class(['site-main']) }}>
        {{ $slot }}
    </main>

    @if(isset($footer))
        <footer {{ $footer->attributes->class(['site-footer']) }}>
            {{ $footer }}
        </footer>
    @endif
</body>
</html>
```

```blade
{{-- pages/home.blade.php --}}
<x-layout title="Home Page" meta-description="Welcome to our site">
    <x-slot:header>
        <nav>...</nav>
    </x-slot>

    <h1>Welcome to the home page</h1>

    <x-slot:footer>
        <p>&copy; 2025</p>
    </x-slot>
</x-layout>
```

**Component Breakdown:**

| Syntax | Purpose |
|--------|---------|
| `<x-layout>` | Renders the layout component. |
| `title="..."` | Passes a prop to the layout. |
| `<x-slot:header>` | Provides content for the `header` named slot. |
| `{{ $slot }}` | Renders the default slot content. |
| `$attributes->class([...])` | Merges classes onto the main element. |

**Syntax Rules:**

- Traditional inheritance uses `@extends` at the top of the child view; component layouts use `<x-layout>` as the root element.
- `@yield` cannot be extended with additional content; it is a placeholder only.
- Component layouts can accept typed props (e.g., `title`, `metaDescription`) through the `@props` directive.
- Named slots in component layouts use `<x-slot:name>` syntax; in traditional inheritance, `@section('name')` is used.
- The `$slot` variable contains all content not within an explicit `<x-slot>` tag.
- Component layouts automatically receive `$attributes` for passthrough HTML attributes.

**Constraints and Limitations:**

- **Directive support:** Blade directives like `@env` cannot be used within component tags (e.g., `<x-alert :live="@env('production')"/>` will not compile).
- **Learning curve:** Traditional inheritance is familiar to developers coming from other PHP frameworks; component layouts require understanding the component mental model.
- **Performance:** Both approaches compile to equivalent PHP; there is no meaningful performance difference.
- **Nested layouts:** Component layouts can be nested, but deeply nested layouts can become difficult to trace.

### Multiple Annotated Complete Code Examples

**Example 1: Traditional Inheritance Layout**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', config('app.name'))</title>
</head>
<body>
    <header>
        @include('partials.navigation')
    </header>

    <main class="container">
        @yield('content')
    </main>

    <footer>
        @include('partials.footer')
    </footer>
</body>
</html>
```

```blade
{{-- resources/views/posts/index.blade.php --}}
@extends('layouts.app')

@section('title', 'All Posts')

@section('content')
    <h1>All Posts</h1>
    @foreach($posts as $post)
        <article>
            <h2>{{ $post->title }}</h2>
        </article>
    @endforeach
@endsection
```

**Expected Output:** The page renders with the navigation header, the list of posts in the main content area, and the footer.

**Why:** The child view extends the parent layout, fills the `title` and `content` sections, and the layout renders those sections via `@yield`.

---

**Example 2: Component-Based Layout**

```blade
{{-- resources/views/components/layout.blade.php --}}
@props([
    'title' => config('app.name'),
    'metaDescription' => null,
])

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ $title }}</title>
    @if($metaDescription)
        <meta name="description" content="{{ $metaDescription }}">
    @endif
</head>
<body>
    <header>
        @include('partials.navigation')
    </header>

    <main {{ $attributes->class(['container']) }}>
        {{ $slot }}
    </main>

    <footer>
        @include('partials.footer')
    </footer>
</body>
</html>
```

```blade
{{-- resources/views/posts/index.blade.php --}}
<x-layout title="All Posts" meta-description="Browse all blog posts">
    <h1>All Posts</h1>
    @foreach($posts as $post)
        <article>
            <h2>{{ $post->title }}</h2>
        </article>
    @endforeach
</x-layout>
```

**Expected Output:** The page renders identically to the traditional inheritance example, but the layout is applied via a component tag with props.

**Why:** The layout component receives `title` and `metaDescription` as props, renders the default slot content inside the `<main>` element, and merges any additional attributes onto the `<main>` tag.

---

**Example 3: Layout with Named Slots**

```blade
{{-- resources/views/components/layout.blade.php --}}
@props(['title' => 'Default'])

<html>
<head>
    <title>{{ $title }}</title>
</head>
<body>
    @if(isset($header))
        <header {{ $header->attributes->class(['site-header']) }}>
            {{ $header }}
        </header>
    @endif

    <main>
        {{ $slot }}
    </main>

    @if(isset($footer))
        <footer {{ $footer->attributes->class(['site-footer']) }}>
            {{ $footer }}
        </footer>
    @endif
</body>
</html>
```

```blade
{{-- resources/views/posts/show.blade.php --}}
<x-layout title="{{ $post->title }}">
    <x-slot:header class="bg-gray-800 text-white">
        <h1>{{ $post->title }}</h1>
    </x-slot>

    <article>
        {!! $post->body !!}
    </article>

    <x-slot:footer>
        <p>Published on {{ $post->created_at->toDateString() }}</p>
    </x-slot>
</x-layout>
```

**Expected Output:** The page renders with a styled header containing the post title, the post body in the main content area, and a footer with the publication date.

**Why:** The layout component uses named slots for the header and footer. Each slot can receive its own attributes (e.g., `class="bg-gray-800"`), which are accessible via the slot's `attributes` property.

### Real-World Cases

- **Application Layouts:** Both patterns are used for the main application shell (header, navigation, footer, content area).
- **Admin Panels:** Layout components with named slots for sidebar, header, and content are common in admin dashboards.
- **Marketing Pages:** Component layouts with props for SEO metadata (title, description, Open Graph tags) are used for landing pages.
- **Multi-Tenant Applications:** Layout components can accept tenant-specific props for branding and navigation.

### References

- Laravel Blade: Layouts Using Components — https://laravel.com/docs/12.x/blade#layouts-using-components
- Laravel Blade: Template Inheritance — https://laravel.com/docs/12.x/blade#template-inheritance
- Stack Overflow: Components vs. Extends with Section & Yield — https://stackoverflow.com/questions/76644433
- Stack Overflow: Laravel 8 Template Layout Slots vs. Old Sections, Yields, and Extends — https://stackoverflow.com/questions/68276871

---

## 2. Anonymous Components

### Definitions

**Core Definition:** Anonymous components are Blade components defined by a single Blade template file with no associated PHP class, using the `@props` directive to declare expected data variables and their default values.

**Technical Definition:** Anonymous components are stored in `resources/views/components` and are rendered using the `<x-component-name>` syntax. Unlike class-based components, they have no PHP class—all logic is contained within the Blade template. The `@props` directive at the top of the template declares which attributes should be extracted as variables (rather than remaining in the `$attributes` bag) and provides default values. All other attributes remain available in the `$attributes` bag for passthrough.

**Beginner-Friendly Explanation:** Anonymous components are the simplest type of component—just a Blade file with no PHP class. You create a file like `alert.blade.php`, declare what data it expects with `@props`, and use it with `<x-alert>`. They're perfect for simple UI elements like buttons, cards, and alerts that don't need complex logic. If you need to query a database or inject a service, use a class-based component instead.

### Purposes

- To create simple, reusable UI components without the overhead of a PHP class.
- To encapsulate presentational markup (buttons, cards, alerts, badges) in a single file.
- To declare expected props and their defaults via the `@props` directive.
- To provide a low-ceremony path to component creation for simple use cases.
- To support nested component directories with automatic discovery.
- To enable `@aware` for accessing parent component data in child components.

### Syntax Rules and Structure

**Complete General Syntax — Creating an Anonymous Component:**

```bash
php artisan make:component alert --view
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `make:component` | Artisan command to generate a component. |
| `alert` | The component name. |
| `--view` | Flag indicating an anonymous (view-only) component. |

**Complete General Syntax — Anonymous Component Template:**

```blade
{{-- resources/views/components/alert.blade.php --}}
@props([
    'type' => 'info',
    'message' => null,
])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    @if($message)
        {{ $message }}
    @else
        {{ $slot }}
    @endif
</div>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@props(['type' => 'info', 'message' => null])` | Declares expected props with defaults. |
| `$type`, `$message` | Variables available in the template. |
| `$attributes` | Remaining HTML attributes. |
| `$slot` | Default slot content. |

**Complete General Syntax — Using an Anonymous Component:**

```blade
<x-alert type="error" class="mt-4">
    Something went wrong!
</x-alert>
```

**Syntax Rules:**

- The `@props` directive must be at the top of the component template.
- Props declared in `@props` are extracted as variables and removed from the `$attributes` bag.
- Props with array keys (`'type' => 'info'`) have default values; props without keys (`'message'`) are required.
- All attributes not declared in `@props` remain in the `$attributes` bag.
- Anonymous components are automatically discovered in `resources/views/components`.
- Components can be nested in subdirectories: `resources/views/components/forms/input.blade.php` → `<x-forms.input />`.
- An `index.blade.php` file can serve as the root template for a component directory.

**Constraints and Limitations:**

- **No logic:** Anonymous components cannot contain PHP class logic—no constructor injection, no methods, no computed properties.
- **No dependency injection:** Services cannot be injected into anonymous components.
- **`@aware` limitation:** The `@aware` directive can only access parent data that is explicitly passed via HTML attributes—default `@props` values are not accessible.
- **Prop declaration required:** All data variables must be declared in `@props`; otherwise they remain in the `$attributes` bag.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Anonymous Alert Component**

```blade
{{-- resources/views/components/alert.blade.php --}}
@props([
    'type' => 'info',
    'message' => null,
])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }} role="alert">
    @if($message)
        <strong>{{ $message }}</strong>
    @else
        {{ $slot }}
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-alert type="error" message="Something went wrong!" />

<x-alert type="success">
    Your changes have been saved.
</x-alert>

<x-alert type="warning" class="mt-4" id="warning-alert">
    Please review your input.
</x-alert>
```

**Expected Output:**
- First usage: `<div class="alert alert-error" role="alert"><strong>Something went wrong!</strong></div>`
- Second usage: `<div class="alert alert-success" role="alert">Your changes have been saved.</div>`
- Third usage: `<div class="alert alert-warning mt-4" id="warning-alert" role="alert">Please review your input.</div>`

**Why:** The `@props` directive extracts `type` and `message` as variables. The `$attributes->merge()` method adds the default `class` while preserving additional classes (`mt-4`) and attributes (`id`).

---

**Example 2: Anonymous Card Component with Named Slots**

```blade
{{-- resources/views/components/card.blade.php --}}
@props([
    'padding' => true,
    'shadow' => 'md',
])

<div {{ $attributes->merge(['class' => 'bg-white rounded-lg shadow-'.$shadow]) }}>
    @if(isset($header))
        <div {{ $header->attributes->class(['px-4 py-5 border-b']) }}>
            {{ $header }}
        </div>
    @endif

    <div @class(['px-4 py-5' => $padding])>
        {{ $slot }}
    </div>

    @if(isset($footer))
        <div {{ $footer->attributes->class(['px-4 py-4 border-t bg-gray-50']) }}>
            {{ $footer }}
        </div>
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-card shadow="lg" class="max-w-sm">
    <x-slot:header class="flex justify-between">
        <h3>Card Title</h3>
    </x-slot>

    <p>This is the card content.</p>

    <x-slot:footer class="flex justify-end">
        <x-button>Save</x-button>
    </x-slot>
</x-card>
```

**Expected Output:** A card with a header containing the title, body content, and a footer with the save button. The card has a large shadow and a max-width of `sm`.

**Why:** The card component uses named slots for `header` and `footer`, allowing flexible content injection. The `$attributes->merge()` handles the card's own classes, while slot attributes (`$header->attributes`, `$footer->attributes`) handle the slot-specific classes.

---

**Example 3: Nested Anonymous Components with `@aware`**

```blade
{{-- resources/views/components/menu/index.blade.php --}}
@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-100']) }}>
    {{ $slot }}
</ul>
```

```blade
{{-- resources/views/components/menu/item.blade.php --}}
@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

```blade
{{-- Usage --}}
<x-menu color="blue">
    <x-menu.item>Dashboard</x-menu.item>
    <x-menu.item>Settings</x-menu.item>
</x-menu>
```

**Expected Output:** A menu with a blue background (`bg-blue-100`) and menu items with dark blue text (`text-blue-800`).

**Why:** The parent `<x-menu>` component declares the `color` prop. The child `<x-menu.item>` uses `@aware(['color' => 'gray'])` to access the parent's `color` value. Without `@aware`, the child would only see its own `$attributes` and would not have access to the parent's prop.

### Real-World Cases

- **UI Kits:** Buttons, badges, alerts, cards, and form inputs are commonly built as anonymous components.
- **Layout Wrappers:** Simple layout components with a default slot and maybe a title prop.
- **Icon Components:** SVG icon components that accept a `name` prop and render the corresponding SVG.
- **Table Components:** Table wrappers with slots for header and body content.
- **Navigation Components:** Navigation menus, breadcrumbs, and tab bars.

### References

- Laravel Blade: Anonymous Components — https://laravel.com/docs/12.x/blade#anonymous-components
- Laravel Blade: Anonymous Component Paths — https://laravel.com/docs/12.x/blade#anonymous-component-paths
- Laravel Blade: The @aware Directive — https://laravel.com/docs/12.x/blade#the-aware-directive
- Laravel Blade: Anonymous Component Props — https://github.com/adrianorsouza/docs/blob/1ba6613f7b4a76a320a8027d6200eedc01038ce7/blade.md#3

---

## 3. Class-Based Components

### Definitions

**Core Definition:** Class-based components are Blade components that pair a PHP class (extending `Illuminate\View\Component`) with a Blade view template, enabling dependency injection, computed properties, methods, and other class-based logic alongside the component's markup.

**Technical Definition:** Class-based components are generated via `php artisan make:component ComponentName`, which creates a PHP class in `app/View/Components` and a Blade view in `resources/views/components`. The component's constructor receives data passed as HTML attributes. The `render()` method returns the view. The class can define public properties (exposed to the template), public methods (callable from the template), and inject dependencies through the constructor. Components can also implement `shouldRender()` for conditional rendering and define `$except` to exclude specific constructor parameters from being exposed to the template.

**Beginner-Friendly Explanation:** Class-based components are like anonymous components with a brain. They have a PHP class behind the scenes that can query databases, call services, compute values, and more. You use them when your component needs to do something before it renders—like fetching a user's avatar from an API, calculating tax, or checking permissions. The class receives data through its constructor and exposes it to the Blade template.

### Purposes

- To create components that require dependency injection (services, repositories, SDK clients).
- To compute derived values (e.g., formatted dates, calculated totals) before rendering.
- To define reusable methods that the Blade template can call (e.g., `formatAlert()`).
- To implement conditional rendering logic via `shouldRender()`.
- To encapsulate complex component logic in a testable PHP class.
- To leverage Laravel's service container for automatic dependency resolution.

### Syntax Rules and Structure

**Complete General Syntax — Generating a Class-Based Component:**

```bash
php artisan make:component Alert
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Alert` | The component class name (creates `app/View/Components/Alert.php`). |
| Generated view | `resources/views/components/alert.blade.php`. |

**Complete General Syntax — Class-Based Component Class:**

```php
<?php
// app/View/Components/Alert.php

namespace App\View\Components;

use App\Services\AlertFormatter;
use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * Create a new component instance.
     */
    public function __construct(
        public string $type = 'info',
        public ?string $message = null,
        private AlertFormatter $formatter
    ) {}

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View|Closure|string
    {
        return view('components.alert');
    }

    /**
     * Format the alert message.
     */
    public function formattedMessage(): string
    {
        return $this->formatter->format($this->message);
    }

    /**
     * Determine if the component should be rendered.
     */
    public function shouldRender(): bool
    {
        return $this->message !== null;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `__construct()` | Receives component data and injected dependencies. |
| `public string $type` | Public property exposed to the template as `$type`. |
| `private AlertFormatter $formatter` | Injected dependency (not exposed to template). |
| `render()` | Returns the component's view. |
| `formattedMessage()` | Public method callable from the template. |
| `shouldRender()` | Determines if the component renders. |

**Complete General Syntax — Class-Based Component View:**

```blade
{{-- resources/views/components/alert.blade.php --}}
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    <span>{{ $formattedMessage() }}</span>
</div>
```

**Complete General Syntax — Using a Class-Based Component:**

```blade
<x-alert type="error" message="Something went wrong!" class="mt-4" />
```

**Syntax Rules:**

- The component class must extend `Illuminate\View\Component`.
- Public properties in the constructor are automatically exposed to the template.
- Private/protected constructor parameters are not exposed to the template.
- The `$except` property can exclude specific public properties from the template.
- The `render()` method must return a view, closure, or string.
- Public methods on the component can be called from the template via `$component->method()`.
- Dependencies type-hinted in the constructor are resolved from the service container.
- The `shouldRender()` method can be overridden to conditionally render the component.

**Constraints and Limitations:**

- **Reserved keywords:** The following cannot be used as property or method names: `data`, `render`, `resolve`, `resolveView`, `shouldRender`, `view`, `withAttributes`, `withName`.
- **Constructor complexity:** Keep constructors simple—they run on every component render.
- **Dependency injection timing:** Constructor-injected dependencies are resolved before the component renders; do not attempt to access request data in the constructor.
- **Performance:** Class-based components add a class instantiation overhead compared to anonymous components.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Class-Based Alert Component**

```php
<?php
// app/View/Components/Alert.php

namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info',
        public ?string $message = null
    ) {}

    public function render(): View|Closure|string
    {
        return view('components.alert');
    }

    public function alertClass(): string
    {
        return match ($this->type) {
            'error' => 'alert-danger',
            'success' => 'alert-success',
            'warning' => 'alert-warning',
            default => 'alert-info',
        };
    }
}
```

```blade
{{-- resources/views/components/alert.blade.php --}}
<div {{ $attributes->merge(['class' => 'alert '.$alertClass()]) }}>
    {{ $message ?? $slot }}
</div>
```

```blade
{{-- Usage --}}
<x-alert type="error" message="Something went wrong!" />

<x-alert type="success">
    Your changes have been saved.
</x-alert>
```

**Expected Output:**
- First usage: `<div class="alert alert-danger">Something went wrong!</div>`
- Second usage: `<div class="alert alert-success">Your changes have been saved.</div>`

**Why:** The `alertClass()` method computes the CSS class based on the `type` prop. The template calls `$alertClass()` to get the class. The `$message` prop is used if provided; otherwise, the default slot is rendered.

---

**Example 2: Class-Based Component with Dependency Injection**

```php
<?php
// app/Services/AvatarService.php

namespace App\Services;

class AvatarService
{
    public function urlFor(string $email): string
    {
        $hash = md5(strtolower(trim($email)));
        return "https://www.gravatar.com/avatar/{$hash}";
    }
}
```

```php
<?php
// app/View/Components/UserAvatar.php

namespace App\View\Components;

use App\Services\AvatarService;
use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class UserAvatar extends Component
{
    public string $url;

    public function __construct(
        public string $email,
        public string $size = 'md',
        AvatarService $avatars
    ) {
        $this->url = $avatars->urlFor($email);
    }

    public function render(): View|Closure|string
    {
        return view('components.user-avatar');
    }
}
```

```blade
{{-- resources/views/components/user-avatar.blade.php --}}
<img src="{{ $url }}"
     alt="User avatar"
     {{ $attributes->class([
         'rounded-full',
         'w-8 h-8' => $size === 'sm',
         'w-12 h-12' => $size === 'md',
         'w-16 h-16' => $size === 'lg',
     ]) }}>
```

```blade
{{-- Usage --}}
<x-user-avatar email="alice@example.com" size="lg" class="border-2" />
```

**Expected Output:** An `<img>` element with the Gravatar URL for the email, large size classes (`w-16 h-16`), rounded-full class, and a `border-2` class.

**Why:** The `AvatarService` is injected into the constructor. The service computes the Gravatar URL, which is stored in the public `$url` property. The template renders the URL and applies size-based classes via `$attributes->class()`.

---

**Example 3: Class-Based Component with `shouldRender()`**

```php
<?php
// app/View/Components/FlashMessage.php

namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class FlashMessage extends Component
{
    public function __construct(
        public string $type = 'status'
    ) {}

    public function render(): View|Closure|string
    {
        return view('components.flash-message');
    }

    public function shouldRender(): bool
    {
        return session()->has($this->type);
    }

    public function message(): string
    {
        return session($this->type);
    }
}
```

```blade
{{-- resources/views/components/flash-message.blade.php --}}
<div {{ $attributes->class(['alert', 'alert-success' => $type === 'status', 'alert-danger' => $type === 'error']) }}>
    {{ $message() }}
</div>
```

```blade
{{-- Usage --}}
<x-flash-message type="status" />
<x-flash-message type="error" />
```

**Expected Output:** If a `status` session value exists, the first component renders a success alert with the message. If an `error` session value exists, the second component renders a danger alert.

**Why:** The `shouldRender()` method checks if the session has a value for the given type. If not, the component does not render at all—no empty wrapper element is output.

### Real-World Cases

- **User Avatars:** Fetching Gravatar URLs or uploaded avatar paths via an injected service.
- **Navigation Menus:** Checking user permissions and rendering different menu items based on roles.
- **Stats Cards:** Computing statistics (total users, revenue, etc.) via injected repositories.
- **Form Components:** Validating and formatting form data before rendering.
- **Conditional Alerts:** Rendering flash messages only when session data exists.

### References

- Laravel Blade: Class-Based Components — https://laravel.com/docs/12.x/blade#class-based-components
- Laravel Blade: Component Methods — https://laravel.com/docs/12.x/blade#component-methods
- Laravel Blade: Component Dependency Injection — https://laravel.com/docs/12.x/blade#component-dependency-injection
- Laravel Blade: ShouldRender — https://laravel.com/docs/12.x/blade#should-render
- Laravel Blade: Reserved Keywords — https://laravel.com/docs/12.x/blade#reserved-keywords

---

## 4. Component Attributes

### Definitions

**Core Definition:** Component attributes are the HTML attributes passed to a component that are not part of its declared props, collected into an `$attributes` bag (an instance of `Illuminate\View\ComponentAttributeBag`) that provides methods for rendering, merging, filtering, and manipulating those attributes.

**Technical Definition:** The `$attributes` variable is automatically available in every Blade component template. It implements `ArrayAccess`, `Htmlable`, and provides fluent methods including `merge()`, `class()`, `only()`, `except()`, `has()`, `get()`, `prepends()`, and `scope()`. The `merge()` method combines default attribute values with passed values (for `class`, values are concatenated; for other attributes, passed values override defaults). The `class()` method conditionally adds classes based on boolean expressions.

**Beginner-Friendly Explanation:** When you use a component like `<x-alert type="error" class="mt-4" id="my-alert">`, the `type` attribute becomes a prop (because it's declared in `@props`), but `class` and `id` stay in the `$attributes` bag. You can then render all these extra attributes on the component's root element with `{{ $attributes }}`. The `merge()` method lets you set default classes (like `alert alert-error`) while allowing the caller to add their own (like `mt-4`). This makes components flexible without requiring every possible class to be a prop.

### Purposes

- To pass through arbitrary HTML attributes (class, id, data-*, aria-*) to the component's root element.
- To merge default CSS classes with caller-provided classes via `$attributes->merge()`.
- To conditionally add classes based on boolean expressions via `$attributes->class()`.
- To filter attributes (only/except) for specific elements within the component.
- To check if an attribute exists via `$attributes->has()`.
- To retrieve an attribute value with a fallback via `$attributes->get()`.
- To prepend values to non-class attributes via `$attributes->prepends()`.

### Syntax Rules and Structure

**Complete General Syntax — Rendering All Attributes:**

```blade
<div {{ $attributes }}>
    {{ $slot }}
</div>
```

**Complete General Syntax — Merging Default Classes:**

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

**Component Breakdown:**

| Method | Purpose |
|--------|---------|
| `$attributes->merge(['class' => 'default'])` | Merges default class with passed classes. |
| `$attributes->class(['p-4', 'bg-red' => $hasError])` | Conditionally adds classes. |
| `$attributes->only(['id', 'class'])` | Keeps only specified attributes. |
| `$attributes->except(['class'])` | Removes specified attributes. |
| `$attributes->has('wire:model')` | Checks if an attribute exists. |
| `$attributes->get('id', 'default')` | Gets an attribute with a fallback. |

**Complete General Syntax — Conditional Classes:**

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

**Complete General Syntax — Non-Class Attribute Merging:**

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

**Syntax Rules:**

- `merge()` concatenates `class` values (defaults first, passed values appended).
- For non-class attributes, `merge()` uses the passed value if provided; otherwise, the default is used.
- `class()` accepts an array where string keys are class names and boolean values determine inclusion; numeric keys always include the class.
- `only()` and `except()` filter the attribute bag for specific elements.
- `has()` checks if an attribute exists without rendering it.
- `get()` retrieves an attribute value with an optional default.

**Constraints and Limitations:**

- **Class merging order:** Default classes are prepended to passed classes. If the caller passes `class="p-4"` and the default is `class="p-2"`, the result is `class="p-2 p-4"`. Tailwind CSS conflict resolution requires a dedicated tool (e.g., `tailwind-merge`).
- **Non-class override:** Non-class attributes are overwritten by passed values, not merged. Use `prepends()` if you need to prepend instead.
- **Attribute bag scope:** The `$attributes` bag contains all attributes not declared in `@props` (for anonymous components) or not in the constructor (for class-based components).

### Multiple Annotated Complete Code Examples

**Example 1: Merging Default Classes**

```blade
{{-- resources/views/components/button.blade.php --}}
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-'.$variant, 'type' => 'button']) }}>
    {{ $slot }}
</button>
```

```blade
{{-- Usage --}}
<x-button>Save</x-button>

<x-button variant="danger" class="w-full">Delete</x-button>

<x-button type="submit" class="mt-4">Submit</x-button>
```

**Expected Output:**
- First: `<button class="btn btn-primary" type="button">Save</button>`
- Second: `<button class="btn btn-danger w-full" type="button">Delete</button>`
- Third: `<button class="btn btn-primary mt-4" type="submit">Submit</button>`

**Why:** The `merge()` method adds the default `class` (`btn btn-primary`) and `type` (`button`). The passed `class="w-full"` is appended to the default classes. The passed `type="submit"` overrides the default `type="button"` (non-class attributes are overwritten).

---

**Example 2: Conditional Classes with `class()`**

```blade
{{-- resources/views/components/alert.blade.php --}}
@props(['type' => 'info', 'dismissible' => false])

<div {{ $attributes->class([
    'alert',
    'alert-'.$type,
    'alert-dismissible' => $dismissible,
]) }}>
    {{ $slot }}

    @if($dismissible)
        <button type="button" class="close" data-dismiss="alert">×</button>
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-alert type="error" :dismissible="true" class="mt-4">
    Something went wrong!
</x-alert>
```

**Expected Output:** `<div class="alert alert-error alert-dismissible mt-4"><button type="button" class="close" data-dismiss="alert">×</button></div>`

**Why:** The `class()` method conditionally adds `alert-dismissible` only when `$dismissible` is true. The `mt-4` class is passed through and appended.

---

**Example 3: Slot Attributes with `$attributes`**

```blade
{{-- resources/views/components/card.blade.php --}}
@props(['padding' => true])

<div {{ $attributes->merge(['class' => 'bg-white rounded-lg shadow']) }}>
    @if(isset($header))
        <div {{ $header->attributes->class(['px-4 py-5 border-b']) }}>
            {{ $header }}
        </div>
    @endif

    <div @class(['px-4 py-5' => $padding])>
        {{ $slot }}
    </div>

    @if(isset($footer))
        <div {{ $footer->attributes->class(['px-4 py-4 border-t bg-gray-50']) }}>
            {{ $footer }}
        </div>
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-card class="max-w-sm shadow-lg">
    <x-slot:header class="flex justify-between items-center">
        <h3>Card Title</h3>
        <x-badge>New</x-badge>
    </x-slot>

    <p>Card content goes here.</p>

    <x-slot:footer class="flex justify-end space-x-2">
        <x-button variant="ghost">Cancel</x-button>
        <x-button>Save</x-button>
    </x-slot>
</x-card>
```

**Expected Output:** A card with `max-w-sm shadow-lg` classes (the `shadow` default is overridden by `shadow-lg`). The header has `flex justify-between items-center` in addition to the default `px-4 py-5 border-b`. The footer has `flex justify-end space-x-2` in addition to `px-4 py-4 border-t bg-gray-50`.

**Why:** The card's `$attributes` handles the outer div's classes. The `$header->attributes` and `$footer->attributes` handle the slot-specific classes, allowing each slot to have its own styling while preserving the component's defaults.

### Real-World Cases

- **Buttons:** Merging default button classes (`btn btn-primary`) with caller-provided classes (`w-full`, `mt-4`).
- **Form Inputs:** Passing through `id`, `name`, `placeholder`, and `required` attributes while merging default form-control classes.
- **Cards:** Merging default card styles with custom width, shadow, and margin classes.
- **Alerts:** Conditionally adding dismissible classes and merging contextual classes.
- **Tables:** Passing through `id`, `class`, and `data-*` attributes for JavaScript hooks.

### References

- Laravel Blade: Component Attributes — https://laravel.com/docs/12.x/blade#component-attributes
- Laravel Blade: Default / Merged Attributes — https://laravel.com/docs/12.x/blade#default-merged-attributes
- Laravel Blade: Conditionally Merge Classes — https://laravel.com/docs/12.x/blade#conditionally-merge-classes
- Laravel Blade: Non-Class Attribute Merging — https://laravel.com/docs/12.x/blade#non-class-attribute-merging
- Laravel Blade: Slot Attributes — https://laravel.com/docs/12.x/blade#slot-attributes

---

## 5. Slots Architecture

### Definitions

**Core Definition:** Slots are the mechanism for injecting content into Blade components, with a **default slot** (`$slot`) for primary content and **named slots** (`<x-slot:name>`) for additional, named content areas within the component.

**Technical Definition:** The default slot is available as the `$slot` variable in the component template and contains all content passed between the opening and closing component tags that is not within an explicit `<x-slot>` tag. Named slots are defined using `<x-slot:name>...</x-slot>` and are available as variables matching the slot name (e.g., `$header`, `$footer`). Each slot is an instance of `Illuminate\View\ComponentSlot`, which provides `isEmpty()`, `isNotEmpty()`, and `hasActualContent()` methods, along with an `attributes` property for slot-specific attributes. Scoped slots allow access to component methods and properties via the `$component` variable.

**Beginner-Friendly Explanation:** Think of a component as a picture frame. The default slot is the main picture—it's what you see when you look at the frame. Named slots are like small inset frames—they hold additional pictures (a title, a caption, a footer). You define where each slot appears in the component, and the person using the component fills them with content. Slot attributes let you add styling to each slot independently.

### Purposes

- To pass content into components through the default slot (`$slot`) for primary content.
- To define multiple named content areas via `<x-slot:name>` for structured components (e.g., card with header, body, footer).
- To conditionally render slot wrappers based on whether content was provided (`$slot->isEmpty()`).
- To attach attributes (classes, IDs) to slots for independent styling.
- To access component methods and properties from within slots via the `$component` variable (scoped slots).
- To check for "actual" content (non-comment) via `$slot->hasActualContent()`.

### Syntax Rules and Structure

**Complete General Syntax — Default Slot:**

```blade
{{-- Component template --}}
<div class="alert">
    {{ $slot }}
</div>
```

```blade
{{-- Usage --}}
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

**Component Breakdown:**

| Syntax | Purpose |
|--------|---------|
| `{{ $slot }}` | Renders the default slot content. |
| Content between tags | Becomes the default slot. |

**Complete General Syntax — Named Slots:**

```blade
{{-- Component template --}}
<div class="card">
    @if(isset($header))
        <div class="card-header">{{ $header }}</div>
    @endif

    <div class="card-body">{{ $slot }}</div>

    @if(isset($footer))
        <div class="card-footer">{{ $footer }}</div>
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-card>
    <x-slot:header>
        Card Title
    </x-slot>

    <p>Card content goes here.</p>

    <x-slot:footer>
        <button>Save</button>
    </x-slot>
</x-card>
```

**Component Breakdown:**

| Syntax | Purpose |
|--------|---------|
| `<x-slot:header>` | Defines content for the `header` slot. |
| `{{ $header }}` | Renders the `header` slot content. |
| `{{ $slot }}` | Renders the default slot content. |

**Complete General Syntax — Slot Attributes:**

```blade
{{-- Component template --}}
<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-sm']) }}>
        {{ $footer }}
    </footer>
</div>
```

```blade
{{-- Usage --}}
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Card Title
    </x-slot>

    Card content.

    <x-slot:footer class="text-gray-500">
        Footer content
    </x-slot>
</x-card>
```

**Component Breakdown:**

| Syntax | Purpose |
|--------|---------|
| `<x-slot:heading class="font-bold">` | Defines a named slot with attributes. |
| `$heading->attributes` | Accesses the slot's attributes. |
| `$heading->attributes->class(['text-lg'])` | Merges classes onto the slot element. |

**Complete General Syntax — Slot Methods:**

```blade
@if($slot->isEmpty())
    <p>This is default content if the slot is empty.</p>
@else
    {{ $slot }}
@endif

@if($slot->hasActualContent())
    <p>The slot has non-comment content.</p>
@endif
```

**Syntax Rules:**

- The default slot is always available as `$slot`.
- Named slots are defined with `<x-slot:name>` and accessed as `$name` in the template.
- Named slots must be checked with `isset($name)` before rendering to avoid errors when not provided.
- Slot attributes are accessed via `$slot->attributes` or `$name->attributes`.
- `isEmpty()` returns true if the slot has no content.
- `hasActualContent()` returns true if the slot contains content other than HTML comments.
- Scoped slots access component methods via `$component->method()`.

**Constraints and Limitations:**

- **Named slot variables:** If a named slot is not provided, its variable is not defined—use `isset()` to check.
- **Slot content location:** Content not within an explicit `<x-slot>` tag is passed to the default slot.
- **Slot attributes on default slot:** The default slot's attributes are accessed via `$slot->attributes`.
- **Scoped slot complexity:** Scoped slots add complexity; use them sparingly.

### Multiple Annotated Complete Code Examples

**Example 1: Default Slot with Fallback Content**

```blade
{{-- resources/views/components/alert.blade.php --}}
@props(['type' => 'info'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    @if($slot->isEmpty())
        <p>This is a default alert message.</p>
    @else
        {{ $slot }}
    @endif
</div>
```

```blade
{{-- Usage --}}
<x-alert type="info" />

<x-alert type="error">
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

**Expected Output:**
- First: `<div class="alert alert-info"><p>This is a default alert message.</p></div>`
- Second: `<div class="alert alert-error"><strong>Whoops!</strong> Something went wrong!</div>`

**Why:** The `$slot->isEmpty()` method checks if content was provided. If not, the fallback message is rendered. If content was provided, the slot is rendered normally.

---

**Example 2: Card Component with Named Slots**

```blade
{{-- resources/views/components/card.blade.php --}}
@props(['padding' => true])

<div {{ $attributes->merge(['class' => 'bg-white rounded-lg shadow']) }}>
    @if(isset($header))
        <div {{ $header->attributes->class(['px-4 py-5 border-b border-gray-200']) }}>
            {{ $header }}
        </div>
    @endif

    <div @class(['px-4 py-5' => $padding])>
        {{ $slot }}
    </div>

    @if(isset($footer))
        <div {{ $footer->attributes->class(['px-4 py-4 border-t border-gray-200 bg-gray-50']) }}>
            {{ $footer }}
        </div>
    @endif
</div>
```

```blade
{{-- Usage: Simple card with only content --}}
<x-card>
    <p>Simple card with just content.</p>
</x-card>

{{-- Usage: Card with header and footer --}}
<x-card>
    <x-slot:header class="flex justify-between items-center">
        <h3 class="text-lg font-medium">Card Title</h3>
        <x-badge color="green">Active</x-badge>
    </x-slot>

    <p class="text-gray-600">This is the main content of the card.</p>

    <x-slot:footer class="flex justify-end space-x-3">
        <x-button variant="ghost" size="sm">Cancel</x-button>
        <x-button size="sm">Save</x-button>
    </x-slot>
</x-card>
```

**Expected Output:**
- Simple card: A card with just the content (no header or footer elements rendered).
- Full card: A card with a header containing the title and badge, body content, and a footer with Cancel and Save buttons.

**Why:** The `@if(isset($header))` check ensures the header wrapper is only rendered when the header slot is provided. The same applies to the footer. This makes the component flexible—it works with or without named slots.

---

**Example 3: Scoped Slot Accessing Component Methods**

```php
<?php
// app/View/Components/Alert.php

namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info'
    ) {}

    public function render(): View|Closure|string
    {
        return view('components.alert');
    }

    public function formatAlert(string $message): string
    {
        return "[{$this->type}] " . strtoupper($message);
    }
}
```

```blade
{{-- resources/views/components/alert.blade.php --}}
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    @if(isset($title))
        <span class="alert-title">{{ $title }}</span>
    @endif

    {{ $slot }}
</div>
```

```blade
{{-- Usage --}}
<x-alert type="error">
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

**Expected Output:** `<div class="alert alert-error"><span class="alert-title">[error] SERVER ERROR</span><strong>Whoops!</strong> Something went wrong!</div>`

**Why:** The `$component` variable in the slot provides access to the component instance. The `formatAlert()` method is called with `'Server Error'`, producing `[error] SERVER ERROR`. This demonstrates scoped slots—accessing component methods from within slot content.

### Real-World Cases

- **Cards:** Default slot for body content, named slots for header, image, and footer.
- **Modals:** Default slot for body content, named slots for title and footer buttons.
- **Alerts:** Default slot for message content, named slot for title.
- **Layouts:** Default slot for page content, named slots for title, header, and footer.
- **Form Groups:** Default slot for input elements, named slot for label and help text.
- **Tables:** Default slot for table body, named slots for header and footer.

### References

- Laravel Blade: Slots — https://laravel.com/docs/12.x/blade#slots
- Laravel Blade: Named Slots — https://laravel.com/docs/12.x/blade#named-slots
- Laravel Blade: Scoped Slots — https://laravel.com/docs/12.x/blade#scoped-slots
- Laravel Blade: Slot Attributes — https://laravel.com/docs/12.x/blade#slot-attributes
- Laravel Blade: Slot Methods (isEmpty, hasActualContent) — https://laravel.com/docs/12.x/blade#slot-methods

---

## References

- Laravel Blade Templates Documentation (12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade: Components — https://laravel.com/docs/12.x/blade#components
- Laravel Blade: Class-Based Components — https://laravel.com/docs/12.x/blade#class-based-components
- Laravel Blade: Anonymous Components — https://laravel.com/docs/12.x/blade#anonymous-components
- Laravel Blade: Component Attributes — https://laravel.com/docs/12.x/blade#component-attributes
- Laravel Blade: Slots — https://laravel.com/docs/12.x/blade#slots
- Laravel Blade: Named Slots — https://laravel.com/docs/12.x/blade#named-slots
- Laravel Blade: Scoped Slots — https://laravel.com/docs/12.x/blade#scoped-slots
- Laravel Blade: Slot Attributes — https://laravel.com/docs/12.x/blade#slot-attributes
- Laravel Blade: Layouts Using Components — https://laravel.com/docs/12.x/blade#layouts-using-components
- Laravel Blade: Template Inheritance — https://laravel.com/docs/12.x/blade#template-inheritance
- Laravel Blade: The @aware Directive — https://laravel.com/docs/12.x/blade#the-aware-directive
- Laravel Blade: Reserved Keywords — https://laravel.com/docs/12.x/blade#reserved-keywords
- Laravel Blade: Anonymous Component Paths — https://laravel.com/docs/12.x/blade#anonymous-component-paths
- Stack Overflow: Components vs. Extends with Section & Yield — https://stackoverflow.com/questions/76644433
- Stack Overflow: Laravel 8 Template Layout Slots vs. Old Sections, Yields, and Extends — https://stackoverflow.com/questions/68276871
- Stack Overflow: Some Laravel Anonymous Component Props Always Use Default Values — https://stackoverflow.com/questions/73656768
- GitHub: Laravel Blade Component Patterns — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-blade/references/components.md
- GitHub: Slots & Attributes Reference — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-blade/references/slots-attributes.md
- GitHub: Card with Named Slots Template — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-blade/references/templates/CardWithSlots.blade.md
- Laravel API: ComponentAttributeBag — https://api.laravel.com/docs/12.x/Illuminate/View/ComponentAttributeBag.html
- Laravel API: ComponentSlot — https://api.laravel.com/docs/12.x/Illuminate/View/ComponentSlot.html
- Laravel API: Component — https://api.laravel.com/docs/12.x/Illuminate/View/Component.html