# Laravel Blade Components: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Blade components are reusable, self-contained UI elements in the Laravel framework that encapsulate markup, logic, and styling into a single unit, invoked through custom HTML-like tags (e.g., `<x-alert>`).

**Technical Definition**

Blade components are implemented as either class-based components (extending `Illuminate\View\Component`) or anonymous components (Blade files in `resources/views/components`). They are resolved by the `BladeCompiler`, which transforms `<x-…>` tags into calls to the component's `render()` method or view. Components support dependency injection, slots, attribute bags, props, and public methods, and are automatically discovered within `app/View/Components` and `resources/views/components`.

**Beginner-Friendly Explanation**

Blade components are like custom HTML tags that you build yourself. For example, you can create an `<x-alert>` component that shows a styled alert box. You can pass content into it (called a "slot") and give it attributes like `type="error"`. This lets you build a library of reusable UI pieces for your application, keeping your code organized and consistent.

---

### Key Characteristics

- **Two Implementation Approaches**: Class-based components (with PHP logic) and anonymous components (Blade template only).
- **Automatic Discovery**: Components are automatically found in `app/View/Components` and `resources/views/components` without manual registration.
- **Attribute Bag**: Non-prop attributes are collected in an `$attributes` object, supporting merging, filtering, and class manipulation.
- **Slots**: Content passed between component tags is available as `$slot` (default) or named slots via `<x-slot:name>`.
- **Props**: Class-based components receive data through constructor properties; anonymous components declare expected data variables using `@props`.
- **Public Methods**: Public methods on component classes are accessible within the component's view.
- **Inline Components**: Components can return their markup directly from `render()` without a separate view file.
- **Dynamic Components**: The `<x-dynamic-component>` tag allows runtime selection of which component to render.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic knowledge of HTML, PHP, and Blade templates**.
- **Familiarity with Artisan commands** for component generation.
- **A text editor or IDE** with Blade syntax support.

---

### Related Programming Areas

- **View Composition**: Components parallel layouts, includes, and sections but offer a more encapsulated model.
- **Design Systems**: Components enable consistent UI libraries across applications.
- **Component-Driven Development**: Mirrors patterns in React, Vue, and Web Components.
- **Dependency Injection**: Class-based components can receive services via constructor injection.
- **Testing**: Laravel provides dedicated testing methods for components (`component()`, `blade()`).

---

## Core Concepts

---

### 1. Blade Components

#### Definitions

**Core Definition**

Blade components are reusable, encapsulated UI units invoked through `<x-component-name>` tags within Blade templates.

**Technical Definition**

When the Blade compiler encounters an `<x-…>` tag, it resolves the component by looking for a class in `App\View\Components` or a Blade file in `resources/views/components`. Class-based components are instantiated with resolved dependencies, and their `render()` method returns a view or markup. Anonymous components are rendered directly from their Blade file. Components can accept attributes, slots, and props, and are automatically discovered without manual registration for application-level components.

**Beginner-Friendly Explanation**

A Blade component is like a custom LEGO block. You build it once, and then you can use it anywhere in your application by writing a simple tag like `<x-alert>`. You can pass information into it and put content inside it, just like you would with regular HTML tags.

#### Purposes

- To encapsulate reusable UI markup and logic into self-contained units.
- To promote consistency across an application by centralizing component behaviour and styling.
- To reduce duplication of HTML and Blade code across multiple views.
- To enable composition of complex interfaces from smaller, testable pieces.
- To provide a mental model that is easier to understand than sections, layouts, and includes for many developers.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Rendering a component --}}
<x-alert type="error" :message="$message">
    <p>Slot content</p>
</x-alert>

{{-- Component with no content (self-closing) --}}
<x-alert type="info" />

{{-- Nested component (dot notation) --}}
<x-inputs.button />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-alert>` | Component tag; name is kebab-case of the class name. |
| `type="error"` | Literal attribute passed to the component. |
| `:message="$message"` | Bound attribute (PHP variable). |
| `{{ $slot }}` | Default slot content inside the component view. |
| `<x-slot:title>` | Named slot definition. |

**Syntax Rules**

- Component tags start with `x-` followed by the kebab-case name of the component class or file.
- Attributes can be literal (`type="error"`) or bound (`:message="$message"`).
- The default slot is available as `$slot`; named slots are accessed by their name (e.g., `$title`).
- Components can be nested using dot notation (e.g., `<x-inputs.button />` for `inputs/button.blade.php`).
- Reserved keywords (`data`, `render`, `resolve`, `resolveView`, `shouldRender`, `view`, `withAttributes`, `withName`) cannot be used as public property or method names.

**Constraints and Limitations**

- Component names must be unique within the application.
- Class-based components require both a class and a view file (unless inline).
- Anonymous components cannot contain PHP logic (no class).
- Attribute names cannot conflict with reserved keywords.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Anonymous Alert Component**

**Setup Guide**

1. Create `resources/views/components/alert.blade.php`.
2. Use the component in a view.

**Component Definition**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-{{ $type }}">
    {{ $slot }}
</div>
```

**Usage**

```blade
<!-- resources/views/demo.blade.php -->
<x-alert type="error">
    <strong>Error!</strong> Something went wrong.
</x-alert>
```

**Expected Output**

```html
<div class="alert alert-error">
    <strong>Error!</strong> Something went wrong.
</div>
```

**Why This Output Occurs**

The `type` attribute is passed to the component and interpolated into the class. The content between the tags becomes the `$slot`, which is echoed inside the `div`.

---

**Example 2: Class-Based Component**

**Setup Guide**

1. Run `php artisan make:component Alert`.
2. This creates `app/View/Components/Alert.php` and `resources/views/components/alert.blade.php`.

**Component Class**

```php
// app/View/Components/Alert.php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(public string $type = 'info') {}

    public function render()
    {
        return view('components.alert');
    }
}
```

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-{{ $type }}">
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert type="warning">
    <strong>Warning!</strong> Check your input.
</x-alert>
```

**Expected Output**

```html
<div class="alert alert-warning">
    <strong>Warning!</strong> Check your input.
</div>
```

**Why This Output Occurs**

The `Alert` class constructor receives `type="warning"`. The `render()` method returns the `components.alert` view with the public property `$type` available. The slot content is rendered inside the `div`.

---

#### Real-World Cases with Explanation

**Case 1: Design System Button Component**

A design system defines a `<x-button>` component with variants (primary, secondary, danger) and sizes. All buttons across the application use this component, ensuring visual consistency.

**Explanation**: Components enforce consistency and make it easy to update styles globally.

**Case 2: Form Input Component**

A form input component wraps an `<input>` element with a label, error display, and styling. It accepts props like `name`, `label`, and `type`, and uses the attribute bag to forward additional HTML attributes.

**Explanation**: Components reduce boilerplate for repetitive form markup.

---

### 2. Component Creation

#### Definitions

**Core Definition**

Component creation is the process of generating the necessary files (class and/or view) for a new Blade component using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:component` command generates a class in `app/View/Components` and a view in `resources/views/components` (or a subdirectory). The `--view` flag creates only an anonymous component (no class). The `--inline` flag creates a component that returns its markup directly from `render()`.

**Beginner-Friendly Explanation**

Creating a component is easy—just run a command in your terminal. Laravel will make the files you need, and you fill them in.

#### Purposes

- To automate the generation of the boilerplate files required for a component.
- To ensure that component classes and views are placed in the correct directories.
- To support both class-based and anonymous component workflows.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:component Alert
php artisan make:component Forms/Input
php artisan make:component forms.input --view
php artisan make:component Alert --inline
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:component Alert` | Creates `app/View/Components/Alert.php` and `resources/views/components/alert.blade.php` |
| `make:component Forms/Input` | Creates component in `Forms` subdirectory |
| `--view` | Creates only an anonymous component (Blade file, no class) |
| `--inline` | Creates a class-based component with inline `render()` |

**Syntax Rules**

- The component name should be in PascalCase (e.g., `Alert`).
- Subdirectories are specified with `/` or dot notation.
- The `--view` flag creates an anonymous component.
- The `--inline` flag creates an inline component.

**Constraints and Limitations**

- The component name must be unique.
- The `--view` and `--inline` flags are mutually exclusive.
- Generated files must be edited to add logic and markup.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Creating a Class-Based Component**

**Command**

```bash
php artisan make:component Alert
```

**Generated Files**

`app/View/Components/Alert.php`:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function render()
    {
        return view('components.alert');
    }
}
```

`resources/views/components/alert.blade.php`:

```blade
<div>
    {{ $slot }}
</div>
```

**Expected Output (Terminal)**

```
INFO  Component [app/View/Components/Alert.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the class and view, placing them in the conventional directories for automatic discovery.

---

**Example 2: Creating an Anonymous Component**

**Command**

```bash
php artisan make:component alert --view
```

**Generated File**

`resources/views/components/alert.blade.php`:

```blade
<div>
    {{ $slot }}
</div>
```

**Expected Output (Terminal)**

```
INFO  Component [resources/views/components/alert.blade.php] created successfully.
```

**Why This Output Occurs**

The `--view` flag instructs Artisan to create only the Blade template, with no associated class.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Component Prototyping**

A developer needs to build a card component quickly. Running `php artisan make:component Card` generates the class and view, which the developer then fills with markup and logic.

**Explanation**: Artisan reduces setup time and enforces Laravel conventions.

---

### 3. Component Classes

#### Definitions

**Core Definition**

Component classes are PHP classes extending `Illuminate\View\Component` that provide the logic, data, and rendering behaviour for class-based Blade components.

**Technical Definition**

A component class defines public properties (automatically passed to the view), a `render()` method returning a view or markup, and optionally a constructor for dependency injection. Public methods are also exposed to the view. The class is resolved by Laravel's service container, allowing constructor injection of services.

**Beginner-Friendly Explanation**

A component class is the "brain" behind a component. It decides what data to pass to the view and can contain PHP logic. The view (the HTML) is separate.

#### Purposes

- To encapsulate component logic and data in a dedicated PHP class.
- To leverage Laravel's service container for dependency injection.
- To provide a clean separation between component logic and presentation.
- To enable the use of public methods and computed properties within component views.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info',
        public string $message = ''
    ) {}

    public function render()
    {
        return view('components.alert');
    }

    public function isError(): bool
    {
        return $this->type === 'error';
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends Component` | Required inheritance. |
| `__construct()` | Optional constructor; supports dependency injection. |
| `public $property` | Public properties are automatically passed to the view. |
| `render()` | Required method returning a view or markup string. |
| `public function method()` | Public methods are accessible in the view. |

**Syntax Rules**

- The class must extend `Illuminate\View\Component`.
- `render()` must return a `View` or a string.
- Public properties are automatically extracted and passed to the view.
- Constructor parameters are resolved by the service container.
- Reserved method names (`data`, `render`, `resolve`, `resolveView`, `shouldRender`, `view`, `withAttributes`, `withName`) cannot be used.

**Constraints and Limitations**

- Public properties cannot have the same name as reserved keywords.
- Constructor injection requires type-hinting the dependency.
- The `render()` method cannot return `null`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Component with Constructor and Public Method**

**Component Class**

```php
// app/View/Components/Alert.php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info'
    ) {}

    public function render()
    {
        return view('components.alert');
    }

    public function isError(): bool
    {
        return $this->type === 'error';
    }
}
```

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-{{ $type }}">
    @if ($isError())
        <strong>Error:</strong>
    @endif
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert type="error">Something went wrong.</x-alert>
```

**Expected Output**

```html
<div class="alert alert-error">
    <strong>Error:</strong>
    Something went wrong.
</div>
```

**Why This Output Occurs**

The constructor receives `type="error"`, making `$type` available in the view. The `isError()` method returns `true`, so the `<strong>Error:</strong>` prefix is rendered.

---

#### Real-World Cases with Explanation

**Case 1: Component with Database Dependency**

A `RecentPosts` component injects a `PostRepository` via its constructor and fetches the latest posts in the constructor. The view iterates over the posts.

**Explanation**: Class-based components can leverage dependency injection to access services.

---

### 4. Component Views

#### Definitions

**Core Definition**

Component views are the Blade templates that define the HTML markup rendered by a component.

**Technical Definition**

For class-based components, the view is typically stored in `resources/views/components/{name}.blade.php`. The `render()` method returns this view via `view('components.name')`. Anonymous components consist solely of this view file. The view has access to public properties and methods of the component class, as well as the `$slot` and `$attributes` variables.

**Beginner-Friendly Explanation**

The component view is the HTML part of the component. It's where you write the markup and use `{{ $slot }}` to insert content passed by the user.

#### Purposes

- To define the HTML structure and presentation of a component.
- To provide a separation between component logic (class) and markup (view).
- To enable the use of Blade directives and echoes within the component's output.
- To support slot injection and attribute rendering.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<!-- resources/views/components/alert.blade.php -->
<div {{ $attributes->merge(['class' => 'alert alert-' . $type]) }}>
    {{ $slot }}
</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$attributes` | Attribute bag containing all non-prop attributes. |
| `$slot` | Default slot content. |
| `{{ $type }}` | Public property from the component class. |
| `$isError()` | Public method call. |

**Syntax Rules**

- The view file should be placed in `resources/views/components` (or a subdirectory).
- Use `{{ $slot }}` to render default slot content.
- Use `{{ $attributes }}` or `{{ $attributes->merge([...]) }}` to render attributes.
- Public properties and methods are available as variables/functions.

**Constraints and Limitations**

- The view cannot access private or protected properties/methods of the component class.
- The view is rendered in the context of the component, not the parent view.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: View with Attribute Merging**

**Component View**

```blade
<!-- resources/views/components/button.blade.php -->
<button {{ $attributes->merge(['class' => 'btn btn-primary', 'type' => 'button']) }}>
    {{ $slot }}
</button>
```

**Usage**

```blade
<x-button class="btn-lg" type="submit">Click Me</x-button>
```

**Expected Output**

```html
<button class="btn btn-primary btn-lg" type="submit">
    Click Me
</button>
```

**Why This Output Occurs**

`$attributes->merge()` combines the default `class` (`btn btn-primary`) with the passed `class` (`btn-lg`). The `type` attribute defaults to `button` but is overridden by `submit`.

---

#### Real-World Cases with Explanation

**Case 1: Card Component View**

A card component view renders a `div` with a header slot, body slot, and footer slot. The view uses `$attributes->merge()` to apply default classes while allowing customization.

**Explanation**: The view encapsulates the card's structure and styling.

---

### 5. Component Attributes

#### Definitions

**Core Definition**

Component attributes are the HTML attributes passed to a component, accessible via the `$attributes` attribute bag.

**Technical Definition**

The `$attributes` variable is an instance of `Illuminate\View\ComponentAttributeBag`. It provides methods for rendering, merging, filtering, and conditionally adding attributes. Attributes not consumed as constructor parameters (class-based) or declared via `@props` (anonymous) are automatically added to this bag.

**Beginner-Friendly Explanation**

Attributes are extra pieces of information you give to a component, like `class="big"` or `id="submit-btn"`. The `$attributes` object lets you decide how to use them in your component's HTML.

#### Purposes

- To allow customization of a component from the outside.
- To provide a mechanism for merging default attributes with passed attributes.
- To support conditional class application.
- To forward attributes to underlying HTML elements.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Rendering all attributes --}}
<div {{ $attributes }}>

{{-- Merging defaults --}}
<div {{ $attributes->merge(['class' => 'alert']) }}>

{{-- Conditional classes --}}
<div {{ $attributes->class(['active' => $isActive]) }}>

{{-- Filtering --}}
{{ $attributes->only(['class']) }}
{{ $attributes->except(['class']) }}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `$attributes` | Renders all attributes as a string. |
| `merge([...])` | Merges default attributes with passed ones. |
| `class([...])` | Conditionally adds CSS classes. |
| `only([...])` | Returns only specified attributes. |
| `except([...])` | Returns all except specified attributes. |

**Syntax Rules**

- Use `{{ $attributes }}` to render all attributes.
- `merge()` accepts an array of default attributes.
- `class()` accepts an array of class names (key = class, value = condition).
- Attributes are automatically escaped.

**Constraints and Limitations**

- Merging non-class attributes may cause duplication if not handled carefully.
- The `$attributes` bag cannot be used in anonymous components without `@props` for data attributes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Merging Classes**

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div {{ $attributes->merge(['class' => 'alert alert-danger']) }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert class="mt-4">Error!</x-alert>
```

**Expected Output**

```html
<div class="alert alert-danger mt-4">
    Error!
</div>
```

**Why This Output Occurs**

The passed `class="mt-4"` is merged with the default `class="alert alert-danger"`, resulting in a combined class string.

---

**Example 2: Conditional Classes**

**Component View**

```blade
<!-- resources/views/components/nav-link.blade.php -->
<a {{ $attributes->class(['active' => $active]) }}>
    {{ $slot }}
</a>
```

**Usage**

```blade
<x-nav-link :active="true" href="/home">Home</x-nav-link>
```

**Expected Output**

```html
<a class="active" href="/home">
    Home
</a>
```

**Why This Output Occurs**

The `active` class is added because the `$active` variable is `true`. The `href` attribute is passed through via the attribute bag.

---

#### Real-World Cases with Explanation

**Case 1: Form Input Attributes**

A form input component forwards `name`, `placeholder`, `required`, and other HTML attributes to the underlying `<input>` element using `{{ $attributes }}`.

**Explanation**: The component acts as a wrapper, letting users pass standard HTML attributes.

---

### 6. Component Slots

#### Definitions

**Core Definition**

Component slots are placeholders within a component's view that are filled with content passed between the component's opening and closing tags.

**Technical Definition**

The default slot is available as `$slot`. Named slots are defined using `<x-slot:name>` and accessed in the component view via the variable matching the slot name (e.g., `$title`). Slots are instances of `Illuminate\View\ComponentSlot` and support methods like `isEmpty()` and `hasActualContent()`.

**Beginner-Friendly Explanation**

A slot is like a blank space in a component where you can put your own content. For example, an alert component might have a slot for the main message and a named slot for the title.

#### Purposes

- To allow dynamic content to be injected into a component.
- To support multiple content areas within a single component via named slots.
- To provide a flexible API for component composition.
- To enable scoped slots where the parent can access component methods via `$component`.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Default slot --}}
<x-alert>
    This is the slot content.
</x-alert>

{{-- Named slots --}}
<x-card>
    <x-slot:header>
        <h2>Title</h2>
    </x-slot:header>
    <p>Body content</p>
</x-card>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$slot` | Default slot content. |
| `$header` | Named slot content. |
| `<x-slot:header>` | Defines content for the `header` slot. |
| `$slot->isEmpty()` | Checks if slot is empty. |
| `$slot->hasActualContent()` | Checks for non-comment content. |

**Syntax Rules**

- Content outside `<x-slot>` tags becomes the default slot.
- Named slots use `<x-slot:name>` syntax.
- The component view must echo the slot variable to render it.
- Slots can have their own attributes, accessible via `$slotName->attributes`.

**Constraints and Limitations**

- A component can have only one default slot.
- Named slot names must be unique within a component.
- Slots cannot be nested within other slots.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Default Slot**

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-info">
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert>
    <strong>Note:</strong> This is a default slot.
</x-alert>
```

**Expected Output**

```html
<div class="alert alert-info">
    <strong>Note:</strong> This is a default slot.
</div>
```

**Why This Output Occurs**

The content between the component tags is assigned to `$slot` and echoed inside the `div`.

---

**Example 2: Named Slots**

**Component View**

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card">
    <div class="card-header">
        {{ $header }}
    </div>
    <div class="card-body">
        {{ $slot }}
    </div>
</div>
```

**Usage**

```blade
<x-card>
    <x-slot:header>
        <h2>Card Title</h2>
    </x-slot:header>
    <p>Card body content.</p>
</x-card>
```

**Expected Output**

```html
<div class="card">
    <div class="card-header">
        <h2>Card Title</h2>
    </div>
    <div class="card-body">
        <p>Card body content.</p>
    </div>
</div>
```

**Why This Output Occurs**

The `<x-slot:header>` defines the `$header` variable, which is rendered in the header div. The remaining content becomes `$slot`, rendered in the body div.

---

#### Real-World Cases with Explanation

**Case 1: Modal Component**

A modal component uses a default slot for the body content and named slots for the title and footer (with action buttons). Different pages reuse the modal shell with custom content.

**Explanation**: Slots provide flexibility while keeping the component's structural markup centralized.

---

### 7. Component Props

#### Definitions

**Core Definition**

Component props are the named data variables that a component explicitly declares as inputs, distinguishing them from generic attributes.

**Technical Definition**

For class-based components, props are constructor parameters with `public` visibility. For anonymous components, props are declared using the `@props` directive at the top of the Blade file. Props are extracted from attributes and made available as variables in the component view. Attributes not declared as props remain in the `$attributes` bag.

**Beginner-Friendly Explanation**

Props are the specific pieces of data a component expects, like `type` or `message`. They're different from other attributes because the component uses them directly in its logic or markup.

#### Purposes

- To define a clear, typed interface for component inputs.
- To distinguish data variables from pass-through attributes.
- To provide default values for optional data.
- To enable validation and type safety in class-based components.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Anonymous component --}}
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

```php
// Class-based component
public function __construct(
    public string $type = 'info',
    public string $message = ''
) {}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@props([...])` | Declares data variables for anonymous components. |
| Constructor parameter | Declares props for class-based components. |
| Default values | Specified as array values in `@props` or parameter defaults. |

**Syntax Rules**

- In anonymous components, `@props` must be at the top of the file.
- In class-based components, props are constructor parameters.
- Props with defaults become optional.
- Props without defaults are required (though PHP may allow `null`).

**Constraints and Limitations**

- Reserved keywords cannot be used as prop names.
- Props cannot be accessed via `$attributes` once declared.
- `@props` only works in anonymous components.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Anonymous Component with Props**

**Component Definition**

```blade
<!-- resources/views/components/alert.blade.php -->
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

The `@props` directive declares `$type` and `$message` as data variables. The `type` attribute is used in the class, and `message` is echoed.

---

**Example 2: Class-Based Component with Constructor Props**

**Component Class**

```php
// app/View/Components/Alert.php
public function __construct(
    public string $type = 'info',
    public string $message = ''
) {}
```

**Usage**

```blade
<x-alert type="warning" message="Low disk space." />
```

**Expected Output**

```html
<div class="alert alert-warning">
    Low disk space.
</div>
```

**Why This Output Occurs**

The constructor receives the attributes as parameters, making them available as public properties in the view.

---

#### Real-World Cases with Explanation

**Case 1: Button Component with Variant Prop**

A button component declares a `variant` prop with a default of `primary`. Users can override it with `variant="danger"` to get a red button.

**Explanation**: Props provide a controlled API for component customization.

---

### 8. Component Methods

#### Definitions

**Core Definition**

Component methods are public methods defined on a class-based component that can be invoked from within the component's view.

**Technical Definition**

Any public method on a component class is automatically made available as a variable-like callable in the component's view. For example, `$component->formatAlert('Error')` or `$isError()` can be used. Methods can accept parameters and return values, enabling computed logic within the view.

**Beginner-Friendly Explanation**

Component methods are functions you define in the component class that you can call from the HTML view. They're useful for calculating values or formatting data without cluttering the view.

#### Purposes

- To encapsulate view-specific logic in the component class rather than the view.
- To provide computed values based on props or component state.
- To enable reusable formatting or condition-checking functions.
- To support scoped slots where the parent can call component methods.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function formatAlert(string $message): string
{
    return strtoupper($message) . '!';
}
```

**Usage in View**

```blade
{{ $formatAlert('Error') }}
{{-- or --}}
{{ $component->formatAlert('Error') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public function method()` | Method definition. |
| `$method()` | Direct call in view. |
| `$component->method()` | Call via the `$component` variable. |

**Syntax Rules**

- Methods must be `public` to be accessible in the view.
- Methods can accept parameters.
- Methods can return any value.
- Methods can be called directly or via `$component`.

**Constraints and Limitations**

- Only public methods are exposed.
- Methods cannot be named after reserved keywords.
- Methods cannot access the view's scope directly.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Method**

**Component Class**

```php
public function isError(): bool
{
    return $this->type === 'error';
}
```

**Component View**

```blade
@if ($isError())
    <strong>Error:</strong>
@endif
{{ $slot }}
```

**Expected Output (when type="error")**

```html
<strong>Error:</strong>
<!-- slot content -->
```

**Why This Output Occurs**

The `isError()` method returns `true`, so the conditional block renders the error prefix.

---

**Example 2: Formatting Method in Scoped Slot**

**Component Class**

```php
public function formatAlert(string $message): string
{
    return strtoupper($message);
}
```

**Usage**

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot:title>
    Something went wrong!
</x-alert>
```

**Expected Output**

```html
<span class="alert-title">SERVER ERROR</span>
<div class="alert alert-danger">
    Something went wrong!
</div>
```

**Why This Output Occurs**

The `$component->formatAlert()` call invokes the public method, which returns the uppercased string.

---

#### Real-World Cases with Explanation

**Case 1: Status Badge Component**

A status badge component has a `getStatusColor()` method that returns a CSS class based on the `$status` prop. The view uses this method to apply the correct colour.

**Explanation**: Methods encapsulate logic that depends on component state.

---

### 9. Anonymous Components

#### Definitions

**Core Definition**

Anonymous components are Blade components defined solely by a Blade template file, without an associated PHP class.

**Technical Definition**

An anonymous component is a `.blade.php` file placed in `resources/views/components`. It has no class, so data variables must be declared using `@props`. Attributes not declared in `@props` are collected in `$attributes`. Anonymous components are automatically discovered and rendered via `<x-…>` tags. They are simpler to create but lack the logic and dependency injection capabilities of class-based components.

**Beginner-Friendly Explanation**

Anonymous components are the simplest kind of component. You just write the HTML and declare what data you expect using `@props`. No PHP class needed.

#### Purposes

- To provide a lightweight way to create reusable UI pieces without a class.
- To reduce boilerplate for components that are purely presentational.
- To enable rapid prototyping and simple component creation.
- To lower the barrier to entry for component-based development.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<!-- resources/views/components/alert.blade.php -->
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@props([...])` | Declares data variables and defaults. |
| `$attributes` | Contains all non-prop attributes. |
| `$slot` | Default slot content. |

**Syntax Rules**

- The component file must be in `resources/views/components`.
- `@props` must be at the top of the file.
- Attributes not declared in `@props` are available in `$attributes`.
- Anonymous components cannot have methods or constructor logic.

**Constraints and Limitations**

- No dependency injection.
- No computed properties or complex logic (except what can be done with Blade directives).
- `@props` must be at the top of the file.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Simple Anonymous Component**

**Component Definition**

```blade
<!-- resources/views/components/alert.blade.php -->
@props(['type' => 'info'])

<div {{ $attributes->merge(['class' => 'alert alert-' . $type]) }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert type="error" class="mt-4">
    Error message
</x-alert>
```

**Expected Output**

```html
<div class="alert alert-error mt-4">
    Error message
</div>
```

**Why This Output Occurs**

The `type` prop is declared and used in the class. The `class="mt-4"` is merged into the attribute bag.

---

#### Real-World Cases with Explanation

**Case 1: UI Kit Components**

A design system ships with a set of anonymous components (buttons, alerts, badges) that are purely presentational and require no PHP logic.

**Explanation**: Anonymous components are ideal for simple, reusable UI elements.

---

### 10. Inline Components

#### Definitions

**Core Definition**

Inline components are class-based components that return their markup directly from the `render()` method as a string, without a separate view file.

**Technical Definition**

Instead of returning a view, the `render()` method returns a Blade string (using a heredoc). This is useful for very small components where managing a separate view file is cumbersome. Inline components are created with `php artisan make:component Alert --inline`.

**Beginner-Friendly Explanation**

An inline component is a component that puts its HTML directly in the PHP class, instead of in a separate file. It's great for tiny components.

#### Purposes

- To reduce the overhead of managing two files for very small components.
- To keep the component's markup and logic together when they are trivial.
- To simplify the component structure for simple use cases.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function render(): string
{
    return <<<'blade'
    <div class="alert alert-danger">
        {{ $slot }}
    </div>
    blade;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `render(): string` | Return type must be `string`. |
| `<<<'blade'` | Heredoc syntax for multi-line strings. |
| `{{ $slot }}` | Slot content is still available. |

**Syntax Rules**

- The `render()` method must return a string.
- The string is treated as a Blade template.
- The `--inline` flag creates an inline component.
- Slot and attributes are available in the inline template.

**Constraints and Limitations**

- Inline components cannot use `@include` or other view-based features that require a file.
- The heredoc syntax can be less readable for complex markup.
- Inline components are best for very small components.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Inline Alert Component**

**Command**

```bash
php artisan make:component Alert --inline
```

**Component Class**

```php
// app/View/Components/Alert.php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(public string $type = 'info') {}

    public function render(): string
    {
        return <<<'blade'
        <div class="alert alert-{{ $type }}">
            {{ $slot }}
        </div>
        blade;
    }
}
```

**Usage**

```blade
<x-alert type="success">Saved successfully!</x-alert>
```

**Expected Output**

```html
<div class="alert alert-success">
    Saved successfully!
</div>
```

**Why This Output Occurs**

The `render()` method returns the Blade string, which is compiled and rendered with the `$type` and `$slot` variables.

---

#### Real-World Cases with Explanation

**Case 1: Tiny Status Indicator**

A status indicator component that simply renders a coloured dot and a label is a good candidate for an inline component.

**Explanation**: Inline components reduce file overhead for trivial UI elements.

---

### 11. Component Nesting

#### Definitions

**Core Definition**

Component nesting is the practice of rendering one component within another, either directly in a view or within another component's markup.

**Technical Definition**

Blade components can be nested by using their `<x-…>` tags inside another component's slot or view. Nested components are resolved independently. Component classes can also be nested in subdirectories, referenced using dot notation (e.g., `<x-inputs.button>` for `app/View/Components/Inputs/Button.php`).

**Beginner-Friendly Explanation**

You can put components inside other components. For example, you can put a button component inside a card component.

#### Purposes

- To build complex interfaces from smaller, reusable components.
- To enable composition of UI elements.
- To support hierarchical component structures.
- To promote maximum reusability.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Nested component usage --}}
<x-card>
    <x-slot:header>
        <x-icon name="user" />
    </x-slot:header>
    <x-button>Submit</x-button>
</x-card>

{{-- Nested component with subdirectory --}}
<x-inputs.button />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-card>` | Outer component. |
| `<x-icon>` | Inner component. |
| `<x-inputs.button>` | Component in subdirectory. |

**Syntax Rules**

- Nested components are resolved independently.
- Subdirectory components use dot notation.
- Components can be nested to any depth.

**Constraints and Limitations**

- Deep nesting can reduce readability.
- Component names must be unique within their namespace.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Card with Nested Button**

**Card Component View**

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card">
    <div class="card-body">
        {{ $slot }}
    </div>
</div>
```

**Button Component View**

```blade
<!-- resources/views/components/button.blade.php -->
<button class="btn">
    {{ $slot }}
</button>
```

**Usage**

```blade
<x-card>
    <p>Card content</p>
    <x-button>Click Me</x-button>
</x-card>
```

**Expected Output**

```html
<div class="card">
    <div class="card-body">
        <p>Card content</p>
        <button class="btn">
            Click Me
        </button>
    </div>
</div>
```

**Why This Output Occurs**

The `<x-button>` is rendered inside the card's slot, and its own `$slot` content is "Click Me".

---

#### Real-World Cases with Explanation

**Case 1: Navigation Menu with Dropdowns**

A navigation component nests dropdown components, which in turn nest link components. This creates a hierarchical menu structure.

**Explanation**: Nesting allows complex UI structures to be built from simple, reusable parts.

---

### 12. Component Testing

#### Definitions

**Core Definition**

Component testing is the practice of writing automated tests that verify a Blade component renders correctly with given inputs.

**Technical Definition**

Laravel provides `$this->component(ComponentClass::class, $data)` and `$this->blade('<x-component />')` methods that return a `TestComponent` instance. This instance offers assertions like `assertSee()`, `assertSeeText()`, `assertDontSee()`, and `assertSeeInOrder()`. Tests are typically written as feature tests using PHPUnit or Pest.

**Beginner-Friendly Explanation**

Testing a component means writing code that checks if your component displays the right thing. Laravel makes this easy with special testing methods.

#### Purposes

- To ensure components render correctly with various inputs.
- To catch regressions when component code changes.
- To verify that slots and attributes are handled as expected.
- To document component behaviour through tests.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Testing a class-based component
$component = $this->component(Alert::class, ['type' => 'error']);
$component->assertSee('Error');

// Testing a Blade string
$view = $this->blade('<x-alert type="error">Test</x-alert>');
$view->assertSee('Test');
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `$this->component()` | Renders a component class. |
| `$this->blade()` | Renders a Blade string. |
| `assertSee()` | Asserts a string is present. |
| `assertDontSee()` | Asserts a string is absent. |
| `assertSeeText()` | Asserts text is present (stripped of HTML). |

**Syntax Rules**

- Tests extend `Tests\TestCase`.
- Use `$this->component()` for class-based components.
- Use `$this->blade()` for anonymous components or raw Blade.
- Assertions are chainable.

**Constraints and Limitations**

- Component tests require the view engine to be booted.
- `assertSee()` escapes by default; use `escape: false` for raw HTML.
- Some assertions may not work in package testing without additional configuration.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Testing a Class-Based Component**

**Test Class**

```php
// tests/Feature/AlertComponentTest.php
namespace Tests\Feature;

use App\View\Components\Alert;
use Tests\TestCase;

class AlertComponentTest extends TestCase
{
    public function test_alert_renders_message(): void
    {
        $component = $this->component(Alert::class, [
            'type' => 'error',
            'message' => 'Something went wrong.',
        ]);

        $component->assertSee('Something went wrong.');
        $component->assertSee('alert-error');
    }
}
```

**Expected Output (Test Result)**

```
PASS  Tests\Feature\AlertComponentTest
✓ alert renders message
```

**Why This Output Occurs**

The `component()` method renders the `Alert` class with the provided data. The `assertSee()` method verifies that the output contains the expected strings.

---

**Example 2: Testing an Anonymous Component with Blade**

**Test Class**

```php
public function test_anonymous_alert(): void
{
    $view = $this->blade(
        '<x-alert type="error">Error!</x-alert>'
    );

    $view->assertSee('Error!');
    $view->assertSee('alert-error');
}
```

**Expected Output (Test Result)**

```
PASS  Tests\Feature\AnonymousComponentTest
✓ anonymous alert
```

**Why This Output Occurs**

The `blade()` method compiles and renders the Blade string. The assertions verify the output.

---

#### Real-World Cases with Explanation

**Case 1: UI Library Testing**

A team building a reusable UI library writes tests for each component to ensure it renders correctly with different props and slots. This prevents regressions when updating the library.

**Explanation**: Component testing is essential for maintaining a reliable component library.

---

### 13. Component Best Practices

#### Definitions

**Core Definition**

Component best practices are the recommended guidelines for designing, implementing, and using Blade components effectively and maintainably.

**Technical Definition**

Best practices encompass naming conventions, prop design, slot usage, attribute handling, testing strategies, and architectural decisions (e.g., when to use anonymous vs. class-based components). They aim to maximize reusability, consistency, and maintainability while minimizing complexity and duplication.

**Beginner-Friendly Explanation**

Best practices are the "do's and don'ts" that experienced developers have learned. Following them makes your components easier to use, understand, and maintain.

#### Purposes

- To ensure components are reusable and maintainable.
- To promote consistency across a codebase.
- To avoid common pitfalls such as attribute conflicts or excessive logic in views.
- To guide the choice between anonymous and class-based components.

#### Key Best Practices

1. **Prefer Anonymous Components for Simple UI**: Use anonymous components for purely presentational elements with no logic. Use class-based components when you need dependency injection, computed properties, or complex logic.

2. **Use `@props` to Declare Data**: In anonymous components, always declare expected data variables with `@props` to distinguish them from pass-through attributes.

3. **Merge Attributes with `$attributes->merge()`**: Always merge default classes and attributes to allow customization while providing sensible defaults.

4. **Use Named Slots for Readability**: When a component has multiple content areas, use named slots instead of a single default slot with conditional logic.

5. **Keep Components Small and Focused**: Each component should have a single responsibility. Extract small, reusable atoms rather than giant organisms.

6. **Name Components Descriptively**: Use clear, kebab-case names that describe the component's purpose (e.g., `x-alert`, `x-user-profile`).

7. **Test Components**: Write tests for components to ensure they render correctly with various inputs.

8. **Avoid Business Logic in Views**: Move complex logic to the component class or a dedicated service. Views should be as simple as possible.

9. **Use Reserved Keywords Carefully**: Avoid using reserved keywords (`data`, `render`, `view`, etc.) as public property or method names.

10. **Document Component APIs**: Provide clear documentation for props, slots, and attributes, especially for shared component libraries.

#### Real-World Cases with Explanation

**Case 1: Choosing Between Anonymous and Class-Based**

A team creates a simple `x-badge` component that only renders a span with a class. They use an anonymous component. For a `x-data-table` component that needs to fetch and format data, they use a class-based component with dependency injection.

**Explanation**: The choice depends on whether the component needs PHP logic or external services.

---

## References

- Laravel Blade Templates Documentation (Laravel 13.x) — https://laravel.com/docs/13.x/blade
- Laravel Blade Templates Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade Templates Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/blade
- Laravel Blade Templates Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/blade
- Laravel TestComponent API Reference — https://api.laravel.com/docs/12.x/Illuminate/Testing/TestComponent.html
- Laravel ComponentAttributeBag API Reference — https://api.laravel.com/docs/12.x/Illuminate/View/ComponentAttributeBag.html
- Laravel HTTP Tests: Rendering Blade and Components — https://laravel.com/docs/12.x/http-tests#rendering-blade-and-components

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Blade components have remained stable across these versions, but developers should always consult the documentation for their specific Laravel version. Features such as `@props`, `$attributes->class()`, and inline components have been available since Laravel 7–8. The `Blade::withoutDoubleEncoding()` method was introduced in Laravel 9.