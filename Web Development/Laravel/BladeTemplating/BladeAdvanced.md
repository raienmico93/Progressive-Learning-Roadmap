# Laravel Blade Components Advanced: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Advanced Blade components extend the basic component system with sophisticated attribute manipulation, slot composition, lifecycle management, and testing capabilities, enabling developers to build highly reusable, testable UI elements.

**Technical Definition**

Advanced Blade component features are implemented through the `ComponentAttributeBag` class, the `ComponentSlot` class, and the component lifecycle methods on `Illuminate\View\Component`. The attribute bag provides over 30 methods for filtering, merging, and manipulating HTML attributes. Slots support named and scoped variants, attribute injection, and content detection. Components support lifecycle hooks (`mount`, `boot`, `hydrate`, etc.) and event dispatching, and can be tested via Laravel's `TestComponent` assertion API.

**Beginner-Friendly Explanation**

Once you have mastered basic Blade components, you can unlock a powerful set of tools. You can control exactly which HTML attributes get passed to your component, decide how classes and styles are merged, create reusable content areas (slots) that can be filled in different ways, and write automated tests to make sure everything works. These features are what make components truly reusable and maintainable in large applications.

---

### Key Characteristics

- **Attribute Bag Power**: The `$attributes` object provides over 30 methods for inspecting, filtering, merging, and rendering HTML attributes.
- **Conditional Class and Style Management**: `->class()` and `->style()` allow conditional application of CSS based on boolean expressions.
- **Prefix-Based Filtering**: `whereStartsWith`, `whereDoesntStartWith`, `thatStartWith`, and `thatDoesntStartWith` enable filtering attributes by prefix (e.g., `wire:model`, `x-data`).
- **Named and Scoped Slots**: Components support multiple named slots and scoped slots that expose component methods and properties to the parent template.
- **Slot Attribute Access**: Slots have their own `$attributes` bag, accessible via `$slotName->attributes`.
- **Testing API**: Laravel provides `TestComponent` with assertions like `assertSee`, `assertSeeText`, `assertSeeInOrder`, and their negative counterparts.
- **Lifecycle Hooks**: Components support `mount`, `boot`, `hydrate`, `updating`, and `updated` lifecycle methods.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Solid understanding of basic Blade components** (anonymous and class-based).
- **Familiarity with Blade layouts, sections, and stacks**.
- **Basic knowledge of PHPUnit or Pest** for component testing.

---

### Related Programming Areas

- **Design Systems**: Advanced attribute and slot features enable building comprehensive design systems with consistent component APIs.
- **Component-Driven Development**: Scoped slots and lifecycle hooks mirror patterns in React, Vue, and Web Components.
- **Testing and Quality Assurance**: The `TestComponent` API integrates with Laravel's testing suite for automated component verification.
- **Accessibility**: Attribute manipulation enables proper ARIA attribute injection and semantic HTML generation.
- **Frontend Framework Integration**: Prefix filtering and colon-escaped attributes facilitate integration with Alpine.js, Vue, and Livewire.

---

## Core Concepts

---

### 1. Component Attributes

#### Definitions

**Core Definition**

Component attributes are HTML attributes passed to a Blade component when it is invoked, which are made available in the component's view via the `$attributes` attribute bag.

**Technical Definition**

When a component is rendered, any attribute specified on the component tag that is not consumed as a constructor parameter (class-based) or declared via `@props` (anonymous) is collected into an `Illuminate\View\ComponentAttributeBag` instance. This bag implements `Arrayable`, `ArrayAccess`, `IteratorAggregate`, `JsonSerializable`, `Htmlable`, and `Stringable`, and provides a fluent API for manipulating and rendering attributes.

**Beginner-Friendly Explanation**

When you write `<x-alert type="error" class="mt-4">`, the `type` attribute is a prop (data for the component), while `class="mt-4"` is a regular attribute. The component can access all regular attributes through the `$attributes` object and decide how to use them in its HTML output.

#### Purposes

- To allow customization of a component from the outside without modifying its code.
- To provide a mechanism for passing through standard HTML attributes (like `class`, `id`, `data-*`) to the component's root element.
- To enable conditional application of attributes based on component state.
- To support integration with JavaScript frameworks via attribute forwarding.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Rendering a component with attributes --}}
<x-alert type="error" class="mt-4" id="alert-1" data-controller="alert">
    Message content
</x-alert>

{{-- Rendering all attributes in the component view --}}
<div {{ $attributes }}>
    {{ $slot }}
</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-alert>` | Component tag. |
| `type="error"` | A declared prop (not in attribute bag). |
| `class="mt-4"` | An undeclared attribute (in attribute bag). |
| `$attributes` | The attribute bag containing all non-prop attributes. |

**Syntax Rules**

- Attributes not declared as props are automatically collected into `$attributes`.
- The attribute bag is available in the component view as `$attributes`.
- `{{ $attributes }}` renders all attributes as a string.
- Attributes can be filtered, merged, and manipulated using the bag's methods.

**Constraints and Limitations**

- Reserved keywords (`data`, `render`, `resolve`, `resolveView`, `shouldRender`, `view`, `withAttributes`, `withName`) cannot be used as prop names or public method names.
- Attributes in the bag are not automatically escaped when rendered with `{{ $attributes }}`; individual values are escaped.
- The attribute bag is only available in class-based components and anonymous components with `@props`.

---

### 2. Attribute Bags (`$attributes`)

#### Definitions

**Core Definition**

An attribute bag is a collection object that holds all non-prop attributes passed to a component, providing methods for inspection, filtering, merging, and rendering.

**Technical Definition**

The `ComponentAttributeBag` class in `Illuminate\View` extends no parent but implements several interfaces for array and string manipulation. It maintains a raw `$attributes` array and provides methods including `all()`, `get()`, `has()`, `hasAny()`, `first()`, `filter()`, `only()`, `except()`, `merge()`, `class()`, `style()`, `prepends()`, `whereStartsWith()`, `whereDoesntStartWith()`, `thatStartWith()`, `thatDoesntStartWith()`, and more.

**Beginner-Friendly Explanation**

An attribute bag is like a toolbox that holds all the extra attributes you passed to a component. It has many useful tools (methods) that let you look inside, pick out specific attributes, combine them with defaults, and render them as HTML.

#### Purposes

- To provide a centralized, method-rich interface for working with component attributes.
- To enable filtering and transformation of attributes before rendering.
- To support default value merging and conditional attribute application.
- To allow prefix-based selection of attributes for framework-specific integration.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Render all attributes --}}
{{ $attributes }}

{{-- Check for attribute presence --}}
@if ($attributes->has('class'))
    <p>Class attribute is present</p>
@endif

{{-- Get attribute value --}}
{{ $attributes->get('class') }}

{{-- Get all attributes as array --}}
$attributes->all()
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `$attributes` | Renders all attributes as a string. |
| `->has('key')` | Returns `true` if the attribute exists. |
| `->hasAny(['a', 'b'])` | Returns `true` if any of the attributes exist. |
| `->get('key', 'default')` | Returns the attribute value or default. |
| `->all()` | Returns all attributes as an array. |
| `->first('default')` | Returns the first attribute's value. |
| `->only(['a', 'b'])` | Returns only the specified attributes. |
| `->except(['a', 'b'])` | Returns all attributes except the specified ones. |
| `->filter($callback)` | Filters attributes using a closure. |
| `->merge([...])` | Merges defaults with passed attributes. |
| `->class([...])` | Conditionally applies CSS classes. |
| `->style([...])` | Conditionally applies inline styles. |
| `->prepends('value')` | Prepends a value to an attribute. |
| `->whereStartsWith('prefix')` | Returns attributes starting with a prefix. |
| `->whereDoesntStartWith('prefix')` | Returns attributes not starting with a prefix. |
| `->thatStartWith('prefix')` | Alias for `whereStartsWith`. |
| `->thatDoesntStartWith('prefix')` | Alias for `whereDoesntStartWith`. |

**Syntax Rules**

- The attribute bag is always available as `$attributes` in component views.
- Method calls are chainable.
- `{{ $attributes }}` renders all attributes with proper escaping.
- `merge()` does not duplicate non-class attributes; it uses the passed values as defaults.

**Constraints and Limitations**

- The attribute bag is read-only in the sense that you cannot add attributes to the original component tag from within the component.
- Some methods (like `prepends`) only make sense within a `merge()` call.
- Attribute keys are case-sensitive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Checking and Retrieving Attributes**

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div {{ $attributes->merge(['class' => 'alert']) }}>
    @if ($attributes->has('id'))
        <span class="alert-id">ID: {{ $attributes->get('id') }}</span>
    @endif
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert id="main-alert" class="mt-4">Error!</x-alert>
```

**Expected Output**

```html
<div class="alert mt-4" id="main-alert">
    <span class="alert-id">ID: main-alert</span>
    Error!
</div>
```

**Why This Output Occurs**

The `$attributes->has('id')` check returns `true`, so the ID span is rendered. `$attributes->get('id')` returns `"main-alert"`. The `merge()` method combines the default `class="alert"` with the passed `class="mt-4"`.

---

**Example 2: Using `all()` and `first()`**

**Component View**

```blade
<!-- resources/views/components/data-list.blade.php -->
<div {{ $attributes->merge(['class' => 'data-list']) }}>
    @php $allAttrs = $attributes->all(); @endphp
    <p>Total attributes: {{ count($allAttrs) }}</p>
    <p>First attribute value: {{ $attributes->first('none') }}</p>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-data-list data-id="1" data-type="user" class="list">
    Content
</x-data-list>
```

**Expected Output**

```html
<div class="data-list list" data-id="1" data-type="user">
    <p>Total attributes: 3</p>
    <p>First attribute value: list</p>
    Content
</div>
```

**Why This Output Occurs**

`$attributes->all()` returns all attributes as an array. The count is 3 (`data-id`, `data-type`, `class`). `$attributes->first()` returns the first attribute's value, which is `"list"` (the class value).

---

#### Real-World Cases with Explanation

**Case 1: Form Input with Conditional Attributes**

A form input component uses `$attributes->has('required')` to conditionally add an asterisk to the label. The component merges default classes with passed classes and forwards `name`, `placeholder`, and other HTML attributes to the underlying `<input>`.

**Explanation**: The attribute bag enables the component to inspect and react to attributes without hardcoding every possible input.

---

### 3. `$attributes->merge()`

#### Definitions

**Core Definition**

The `merge()` method combines a set of default attributes with the attributes passed to the component, giving precedence to passed values for non-class attributes.

**Technical Definition**

`ComponentAttributeBag::merge(array $attributeDefaults)` returns a new attribute bag where each default attribute is merged with the passed attribute of the same name. For the `class` attribute, values are concatenated (passed value appended to default). For other attributes, the passed value takes precedence; if no passed value exists, the default is used. The `prepends()` method can be used within `merge()` to force a value to be prepended rather than appended.

**Beginner-Friendly Explanation**

`merge()` is like saying "use these default attributes, but if the user passed their own values, use those instead." For classes, both the default and the user's classes are kept.

#### Purposes

- To provide sensible defaults for component attributes that can be overridden.
- To ensure that CSS classes are combined rather than replaced.
- To allow other attributes (like `type`, `data-*`) to be defaulted while remaining overridable.
- To support the `prepends()` method for attributes where the default should always come first.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->merge(['class' => 'default-class', 'type' => 'button']) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$attributes->merge([...])` | Merges the passed array with the component's attributes. |
| `'class' => 'default'` | Class values are concatenated. |
| `'type' => 'button'` | Passed value overrides default. |
| `$attributes->prepends('value')` | Used inside `merge()` to prepend a value. |

**Syntax Rules**

- `merge()` returns a new `ComponentAttributeBag`; it does not mutate the original.
- For `class`, values are concatenated with a space separator.
- For other attributes, the passed value takes precedence.
- `prepends()` can be called on `$attributes` within the `merge()` array to force prepending.

**Constraints and Limitations**

- `merge()` does not deep-merge arrays or objects.
- The `prepends()` method only works within a `merge()` call.
- Attribute names are case-sensitive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Merging Classes and Type**

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

The default `class` (`btn btn-primary`) is merged with the passed `class` (`btn-lg`), resulting in `"btn btn-primary btn-lg"`. The `type` attribute defaults to `"button"` but is overridden by the passed `"submit"`.

---

**Example 2: Using `prepends()` within `merge()`**

**Component View**

```blade
<!-- resources/views/components/controller.blade.php -->
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-controller data-controller="user-controller">Content</x-controller>
```

**Expected Output**

```html
<div data-controller="profile-controller user-controller">Content</div>
```

**Why This Output Occurs**

The `prepends('profile-controller')` ensures that `"profile-controller"` always comes first in the `data-controller` attribute. The passed value `"user-controller"` is appended after it.

---

#### Real-World Cases with Explanation

**Case 1: Alert Component with Merge**

An alert component merges a default `class="alert"` with the passed classes and merges a default `role="alert"` with any passed role.

**Explanation**: `merge()` provides sensible defaults while preserving user customization.

---

### 4. `$attributes->class()`

#### Definitions

**Core Definition**

The `class()` method conditionally adds CSS classes to the attribute bag based on boolean values in an array.

**Technical Definition**

`ComponentAttributeBag::class(array $classes)` returns a new attribute bag with the specified classes added. The array keys are the class names, and the values are booleans indicating whether the class should be applied. This method is typically used in conjunction with `merge()` to combine default and conditional classes.

**Beginner-Friendly Explanation**

`class()` lets you say "add this CSS class only if this condition is true." For example, you can add an `active` class only when a navigation item is the current page.

#### Purposes

- To conditionally apply CSS classes based on component state or props.
- To build dynamic class strings without messy string concatenation.
- To combine conditional classes with default and passed classes via `merge()`.
- To support boolean expressions for class application.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->class(['active' => $isActive, 'disabled' => $isDisabled]) }}
{{ $attributes->merge(['class' => 'base'])->class(['active' => $isActive]) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->class([...])` | Applies classes conditionally. |
| `'active' => $isActive` | Applies `active` if `$isActive` is truthy. |
| `->merge([...])->class([...])` | Chains merge and class for combined behaviour. |

**Syntax Rules**

- The array keys are CSS class names.
- The array values are boolean conditions.
- `class()` returns a new attribute bag; it does not mutate the original.
- Can be chained with `merge()`.

**Constraints and Limitations**

- Only boolean conditions are evaluated; non-boolean values are truthy/falsy.
- Class names with spaces (e.g., `"btn btn-primary"`) should be specified as separate array entries or in a `merge()` default.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Active Class**

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
<x-nav-link :active="false" href="/about">About</x-nav-link>
```

**Expected Output**

```html
<a class="active" href="/home">Home</a>
<a href="/about">About</a>
```

**Why This Output Occurs**

The first component receives `$active = true`, so the `active` class is applied. The second receives `$active = false`, so no class is added.

---

**Example 2: Chaining `merge()` and `class()`**

**Component View**

```blade
<!-- resources/views/components/button.blade.php -->
<button {{ $attributes->merge(['class' => 'btn'])->class(['btn-primary' => $primary, 'btn-lg' => $large]) }}>
    {{ $slot }}
</button>
```

**Usage**

```blade
<x-button :primary="true" :large="false">Submit</x-button>
```

**Expected Output**

```html
<button class="btn btn-primary">Submit</button>
```

**Why This Output Occurs**

`merge()` adds the default `btn` class. `class()` conditionally adds `btn-primary` (since `$primary` is `true`) and `btn-lg` (since `$large` is `false`).

---

#### Real-World Cases with Explanation

**Case 1: Tab Component with Active State**

A tab component uses `class(['active' => $isActive])` to apply an `active` class to the currently selected tab.

**Explanation**: Conditional classes enable dynamic UI states without complex string manipulation.

---

### 5. `$attributes->style()`

#### Definitions

**Core Definition**

The `style()` method conditionally adds inline CSS styles to the attribute bag based on boolean values in an array.

**Technical Definition**

`ComponentAttributeBag::style(array $styles)` returns a new attribute bag with the specified inline styles added. The array keys are CSS property declarations (e.g., `"color: red"`), and the values are booleans indicating whether the style should be applied. It follows the same pattern as `class()` but for the `style` attribute.

**Beginner-Friendly Explanation**

`style()` is like `class()` but for inline styles. You can conditionally apply styles like `background-color: red` only when a condition is true.

#### Purposes

- To conditionally apply inline CSS styles based on component state.
- To build dynamic style strings without manual concatenation.
- To combine conditional styles with default and passed styles.
- To support boolean expressions for style application.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->style(['color: red' => $isError, 'font-weight: bold' => $isBold]) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->style([...])` | Applies styles conditionally. |
| `'color: red' => $isError` | Applies `color: red` if `$isError` is truthy. |

**Syntax Rules**

- Array keys are CSS property declarations.
- Array values are boolean conditions.
- Returns a new attribute bag.
- Can be chained with `merge()`.

**Constraints and Limitations**

- CSS declarations should include the property and value (e.g., `"color: red"`).
- The `style` attribute is rendered as a string with semicolons separating declarations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Inline Styles**

**Component View**

```blade
<!-- resources/views/components/status-badge.blade.php -->
<span {{ $attributes->style(['color: green' => $isActive, 'color: red' => !$isActive]) }}>
    {{ $slot }}
</span>
```

**Usage**

```blade
<x-status-badge :isActive="true">Online</x-status-badge>
<x-status-badge :isActive="false">Offline</x-status-badge>
```

**Expected Output**

```html
<span style="color: green">Online</span>
<span style="color: red">Offline</span>
```

**Why This Output Occurs**

The first component applies `color: green` because `$isActive` is `true`. The second applies `color: red` because `$isActive` is `false`.

---

#### Real-World Cases with Explanation

**Case 1: Progress Bar with Dynamic Width**

A progress bar component uses `style(['width: ' . $percentage . '%' => true])` to set the width dynamically.

**Explanation**: `style()` enables dynamic inline styling based on component data.

---

### 6. `$attributes->except()`

#### Definitions

**Core Definition**

The `except()` method returns a new attribute bag containing all attributes except those with the specified keys.

**Technical Definition**

`ComponentAttributeBag::except(array $keys)` filters out attributes whose keys are in the provided array. It returns a new `ComponentAttributeBag` instance containing only the remaining attributes. This is useful for excluding specific attributes from being rendered or forwarded.

**Beginner-Friendly Explanation**

`except()` lets you say "render everything except these specific attributes." For example, you might want to exclude `class` from a wrapper div and apply it elsewhere.

#### Purposes

- To exclude specific attributes from being rendered in a particular location.
- To prevent certain attributes from being forwarded to child elements.
- To filter out framework-specific attributes before rendering standard HTML.
- To support fine-grained control over attribute output.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->except(['class']) }}
{{ $attributes->except(['wire:model', 'x-data']) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->except(['class'])` | Returns all attributes except `class`. |
| `->except(['a', 'b'])` | Excludes multiple attributes. |

**Syntax Rules**

- The argument is an array of attribute keys to exclude.
- Returns a new attribute bag.
- Can be chained with other methods.

**Constraints and Limitations**

- Keys are case-sensitive.
- If a key does not exist, it is silently ignored.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Excluding Class from Wrapper**

**Component View**

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card-wrapper" {{ $attributes->except(['class']) }}>
    <div class="card-content {{ $attributes->get('class') }}">
        {{ $slot }}
    </div>
</div>
```

**Usage**

```blade
<x-card class="shadow-lg" id="my-card">Card content</x-card>
```

**Expected Output**

```html
<div class="card-wrapper" id="my-card">
    <div class="card-content shadow-lg">
        Card content
    </div>
</div>
```

**Why This Output Occurs**

The `except(['class'])` excludes the `class` attribute from the wrapper div, while `id` is preserved. The `class` value is applied to the inner content div via `$attributes->get('class')`.

---

#### Real-World Cases with Explanation

**Case 1: Excluding Framework Attributes**

A component designed for Alpine.js integration excludes `x-data` and `x-init` from the root element and applies them to a different element.

**Explanation**: `except()` provides precise control over attribute placement.

---

### 7. `$attributes->only()`

#### Definitions

**Core Definition**

The `only()` method returns a new attribute bag containing only the attributes with the specified keys.

**Technical Definition**

`ComponentAttributeBag::only(array $keys)` creates a new bag containing only the attributes whose keys are in the provided array. This is the inverse of `except()`.

**Beginner-Friendly Explanation**

`only()` lets you say "render only these specific attributes." It is useful when you want to extract a subset of attributes for a specific purpose.

#### Purposes

- To extract a subset of attributes for rendering in a specific location.
- To isolate framework-specific attributes for a particular element.
- To provide granular control over attribute distribution.
- To complement `except()` for attribute filtering.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->only(['class']) }}
{{ $attributes->only(['wire:model', 'wire:change']) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->only(['class'])` | Returns only the `class` attribute. |
| `->only(['a', 'b'])` | Returns only `a` and `b`. |

**Syntax Rules**

- The argument is an array of attribute keys to include.
- Returns a new attribute bag.
- Can be chained with other methods.

**Constraints and Limitations**

- Keys are case-sensitive.
- If a key does not exist, it is not included in the result.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Extracting Only Class Attributes**

**Component View**

```blade
<!-- resources/views/components/wrapper.blade.php -->
<div class="wrapper" {{ $attributes->only(['class']) }}>
    <div {{ $attributes->except(['class']) }}>
        {{ $slot }}
    </div>
</div>
```

**Usage**

```blade
<x-wrapper class="outer-class" id="wrapper-id" data-role="wrapper">
    Content
</x-wrapper>
```

**Expected Output**

```html
<div class="wrapper outer-class">
    <div id="wrapper-id" data-role="wrapper">
        Content
    </div>
</div>
```

**Why This Output Occurs**

`only(['class'])` extracts the `class` attribute for the outer div. `except(['class'])` passes all other attributes (`id`, `data-role`) to the inner div.

---

#### Real-World Cases with Explanation

**Case 1: Separating Layout and Content Attributes**

A layout component uses `only()` to apply layout-related attributes to a wrapper and `except()` to pass content attributes to an inner element.

**Explanation**: `only()` and `except()` work together to distribute attributes appropriately.

---

### 8. `$attributes->filter()`

#### Definitions

**Core Definition**

The `filter()` method returns a new attribute bag containing only the attributes that satisfy a given closure condition.

**Technical Definition**

`ComponentAttributeBag::filter(callable $callback)` applies the callback to each attribute, where the callback receives the value and key as arguments. Attributes for which the callback returns `true` are retained; those for which it returns `false` are removed. The callback signature is `fn ($value, $key) => bool`.

**Beginner-Friendly Explanation**

`filter()` is like a custom filter for attributes. You write a rule, and only attributes that match the rule are kept.

#### Purposes

- To apply custom, arbitrary conditions for attribute inclusion.
- To filter attributes by their values rather than just their keys.
- To implement complex attribute selection logic that cannot be expressed with prefix-based methods.
- To support dynamic attribute bag manipulation.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}
{{ $attributes->filter(fn ($value, $key) => str_starts_with($key, 'data-')) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->filter($callback)` | Filters attributes using the closure. |
| `$value` | The attribute value. |
| `$key` | The attribute key. |

**Syntax Rules**

- The callback must return a boolean.
- The callback receives `$value` and `$key` as arguments.
- Returns a new attribute bag.
- Can be chained.

**Constraints and Limitations**

- The callback is applied to each attribute individually.
- Complex conditions may impact performance with very large attribute bags.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Filtering by Key Prefix**

**Component View**

```blade
<!-- resources/views/components/form-field.blade.php -->
<div {{ $attributes->filter(fn ($value, $key) => str_starts_with($key, 'wire:')) }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-form-field wire:model="name" wire:change="updateName" class="form-control" id="name-field">
    <input type="text">
</x-form-field>
```

**Expected Output**

```html
<div wire:model="name" wire:change="updateName">
    <input type="text">
</div>
```

**Why This Output Occurs**

The `filter()` callback retains only attributes whose keys start with `"wire:"`. The `class` and `id` attributes are excluded.

---

#### Real-World Cases with Explanation

**Case 1: Filtering Livewire Attributes**

A Livewire-aware component uses `filter()` to separate `wire:` attributes from standard HTML attributes, applying them to different elements.

**Explanation**: `filter()` provides maximum flexibility for custom attribute selection.

---

### 9. `$attributes->whereStartsWith()`

#### Definitions

**Core Definition**

The `whereStartsWith()` method returns a new attribute bag containing only attributes whose keys begin with a specified string.

**Technical Definition**

`ComponentAttributeBag::whereStartsWith(string $prefix)` filters attributes by key prefix, returning a new bag with only matching attributes. This is a convenience method built on top of `filter()` for the common case of prefix-based filtering. It is commonly used for framework-specific attributes like `wire:model` (Livewire) or `x-data` (Alpine.js).

**Beginner-Friendly Explanation**

`whereStartsWith()` lets you grab all attributes that start with a specific prefix. For example, you can get all `wire:` attributes or all `data-` attributes.

#### Purposes

- To isolate framework-specific attributes by prefix.
- To forward `wire:` attributes to Livewire-managed elements.
- To extract `data-` attributes for JavaScript consumption.
- To filter attributes by naming convention.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->whereStartsWith('wire:model') }}
{{ $attributes->whereStartsWith('data-') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->whereStartsWith('prefix')` | Returns attributes starting with `prefix`. |

**Syntax Rules**

- The argument is a string prefix.
- Returns a new attribute bag.
- Case-sensitive matching.
- Can be chained with `first()` or other methods.

**Constraints and Limitations**

- Only exact prefix matches are returned.
- The prefix is not case-insensitive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Forwarding Livewire Attributes**

**Component View**

```blade
<!-- resources/views/components/livewire-input.blade.php -->
<div>
    <input type="text" {{ $attributes->whereStartsWith('wire:') }}>
</div>
```

**Usage**

```blade
<x-livewire-input wire:model="email" wire:blur="validateEmail" class="form-control" />
```

**Expected Output**

```html
<div>
    <input type="text" wire:model="email" wire:blur="validateEmail">
</div>
```

**Why This Output Occurs**

Only attributes starting with `"wire:"` are passed to the input. The `class` attribute is excluded from this element (though it remains in the original `$attributes` bag).

---

#### Real-World Cases with Explanation

**Case 1: Livewire Component Wrapper**

A wrapper component uses `whereStartsWith('wire:')` to forward Livewire directives to an internal input element while applying its own classes to the wrapper div.

**Explanation**: Prefix filtering enables clean separation of framework-specific attributes.

---

### 10. `$attributes->whereDoesntStartWith()`

#### Definitions

**Core Definition**

The `whereDoesntStartWith()` method returns a new attribute bag containing only attributes whose keys do **not** begin with a specified string.

**Technical Definition**

`ComponentAttributeBag::whereDoesntStartWith(string $prefix)` is the inverse of `whereStartsWith()`. It returns a new bag with attributes that do not match the prefix. This is useful for excluding framework-specific attributes from standard HTML elements.

**Beginner-Friendly Explanation**

`whereDoesntStartWith()` lets you say "give me all attributes except those starting with this prefix." It is the opposite of `whereStartsWith()`.

#### Purposes

- To exclude framework-specific attributes from standard HTML rendering.
- To prevent `wire:` or `x-` attributes from appearing on elements that should not have them.
- To filter out internal attributes before passing to a third-party library.
- To complement `whereStartsWith()` for attribute distribution.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->whereDoesntStartWith('wire:') }}
{{ $attributes->whereDoesntStartWith('x-') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->whereDoesntStartWith('prefix')` | Returns attributes not starting with `prefix`. |

**Syntax Rules**

- The argument is a string prefix.
- Returns a new attribute bag.
- Case-sensitive matching.
- Can be chained.

**Constraints and Limitations**

- Only exact prefix matches are excluded.
- The prefix is not case-insensitive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Excluding Livewire Attributes from a Wrapper**

**Component View**

```blade
<!-- resources/views/components/wrapper.blade.php -->
<div {{ $attributes->whereDoesntStartWith('wire:') }}>
    <input type="text" {{ $attributes->whereStartsWith('wire:') }}>
</div>
```

**Usage**

```blade
<x-wrapper wire:model="name" class="wrapper" id="wrapper">
    <span>Content</span>
</x-wrapper>
```

**Expected Output**

```html
<div class="wrapper" id="wrapper">
    <input type="text" wire:model="name">
    <span>Content</span>
</div>
```

**Why This Output Occurs**

The wrapper div receives all attributes except those starting with `wire:`. The input receives only the `wire:` attributes. The `class` and `id` are applied to the wrapper.

---

#### Real-World Cases with Explanation

**Case 1: Clean HTML Output**

A component that wraps a third-party library uses `whereDoesntStartWith('wire:')` to ensure Livewire attributes are not rendered on the library's elements.

**Explanation**: Prefix-based exclusion keeps HTML output clean and semantically correct.

---

### 11. `$attributes->thatStartWith()`

#### Definitions

**Core Definition**

The `thatStartWith()` method is an alias for `whereStartsWith()`, returning attributes whose keys begin with a specified string.

**Technical Definition**

In the current implementation of `ComponentAttributeBag`, `thatStartWith()` is identical to `whereStartsWith()`. Both methods perform the same prefix-based filtering. The alias exists for API convenience and readability in certain contexts.

**Beginner-Friendly Explanation**

`thatStartWith()` is exactly the same as `whereStartsWith()`. You can use either one; they do the same thing.

#### Purposes

- To provide an alternative method name for prefix-based attribute filtering.
- To support different coding style preferences.
- To maintain API consistency with other Laravel methods that use `that` prefixes.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->thatStartWith('wire:') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->thatStartWith('prefix')` | Alias for `whereStartsWith()`. |

**Syntax Rules**

- Identical behaviour to `whereStartsWith()`.
- Returns a new attribute bag.

**Constraints and Limitations**

- Same as `whereStartsWith()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using `thatStartWith()`**

**Component View**

```blade
<!-- resources/views/components/data-wrapper.blade.php -->
<div {{ $attributes->thatStartWith('data-') }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-data-wrapper data-id="1" data-type="test" class="wrapper">Content</x-data-wrapper>
```

**Expected Output**

```html
<div data-id="1" data-type="test">Content</div>
```

**Why This Output Occurs**

`thatStartWith('data-')` returns only attributes whose keys start with `"data-"`. The `class` attribute is excluded.

---

#### Real-World Cases with Explanation

**Case 1: Extracting Data Attributes**

A JavaScript integration component uses `thatStartWith('data-')` to extract all `data-*` attributes for serialization to a JSON config object.

**Explanation**: `thatStartWith()` provides a readable alias for common prefix filtering.

---

### 12. `$attributes->thatDoesntStartWith()`

#### Definitions

**Core Definition**

The `thatDoesntStartWith()` method is an alias for `whereDoesntStartWith()`, returning attributes whose keys do **not** begin with a specified string.

**Technical Definition**

`ComponentAttributeBag::thatDoesntStartWith(string $prefix)` is identical in behaviour to `whereDoesntStartWith()`. It filters out attributes matching the given prefix.

**Beginner-Friendly Explanation**

`thatDoesntStartWith()` is exactly the same as `whereDoesntStartWith()`. It excludes attributes with a certain prefix.

#### Purposes

- To provide an alternative method name for prefix-based attribute exclusion.
- To support different coding style preferences.
- To maintain API consistency with `thatStartWith()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->thatDoesntStartWith('wire:') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->thatDoesntStartWith('prefix')` | Alias for `whereDoesntStartWith()`. |

**Syntax Rules**

- Identical behaviour to `whereDoesntStartWith()`.
- Returns a new attribute bag.

**Constraints and Limitations**

- Same as `whereDoesntStartWith()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using `thatDoesntStartWith()`**

**Component View**

```blade
<!-- resources/views/components/non-livewire.blade.php -->
<div {{ $attributes->thatDoesntStartWith('wire:') }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-non-livewire wire:model="name" class="box" id="non-livewire">
    Content
</x-non-livewire>
```

**Expected Output**

```html
<div class="box" id="non-livewire">Content</div>
```

**Why This Output Occurs**

`thatDoesntStartWith('wire:')` excludes the `wire:model` attribute, leaving `class` and `id`.

---

#### Real-World Cases with Explanation

**Case 1: Standard HTML Wrapper**

A wrapper component that should never expose Livewire attributes uses `thatDoesntStartWith('wire:')` to ensure clean HTML output.

**Explanation**: The alias provides a readable alternative for excluding framework attributes.

---

### 13. `$attributes->get()`

#### Definitions

**Core Definition**

The `get()` method retrieves the value of a specific attribute from the attribute bag, with an optional default value if the attribute is not present.

**Technical Definition**

`ComponentAttributeBag::get(string $key, mixed $default = null)` returns the value of the attribute identified by `$key`. If the attribute does not exist, the `$default` value is returned. This is the primary method for accessing individual attribute values.

**Beginner-Friendly Explanation**

`get()` lets you read a specific attribute's value. If the attribute is not there, you can provide a fallback value.

#### Purposes

- To retrieve the value of a specific attribute for use in component logic.
- To access attribute values with a safe fallback when the attribute is optional.
- To implement conditional behaviour based on attribute values.
- To extract attribute data for use in JavaScript or other integrations.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->get('class') }}
{{ $attributes->get('id', 'default-id') }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->get('key')` | Returns the attribute value or `null`. |
| `->get('key', 'default')` | Returns the attribute value or the default. |

**Syntax Rules**

- The first argument is the attribute key.
- The second argument is optional and provides a default value.
- Returns the attribute value or the default.

**Constraints and Limitations**

- Keys are case-sensitive.
- Returns `null` if the attribute does not exist and no default is provided.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Retrieving Attribute with Default**

**Component View**

```blade
<!-- resources/views/components/user-card.blade.php -->
<div class="user-card" id="{{ $attributes->get('id', 'user-' . $user->id) }}">
    <h3>{{ $user->name }}</h3>
</div>
```

**Usage**

```blade
<x-user-card :user="$user" />
<x-user-card :user="$user" id="custom-id" />
```

**Expected Output (first)**

```html
<div class="user-card" id="user-5">
    <h3>John Doe</h3>
</div>
```

**Expected Output (second)**

```html
<div class="user-card" id="custom-id">
    <h3>John Doe</h3>
</div>
```

**Why This Output Occurs**

In the first case, no `id` is passed, so the default `user-5` (using the user's ID) is used. In the second, the passed `id="custom-id"` is used.

---

#### Real-World Cases with Explanation

**Case 1: Default ID Generation**

A component that renders a form field uses `get('id', 'field-' . Str::random(6))` to generate a unique ID if none is provided.

**Explanation**: `get()` with a default enables graceful fallback for optional attributes.

---

### 14. `$attributes->has()`

#### Definitions

**Core Definition**

The `has()` method checks whether one or more specific attributes exist in the attribute bag.

**Technical Definition**

`ComponentAttributeBag::has(string|array $key)` returns `true` if the attribute (or all attributes in the array) exist. When an array is passed, it returns `true` only if **all** specified attributes are present. The `hasAny()` method can be used to check if **any** of the specified attributes are present.

**Beginner-Friendly Explanation**

`has()` lets you check if an attribute exists. You can check for one attribute or an array of attributes (all must exist).

#### Purposes

- To conditionally render content based on the presence of an attribute.
- To implement component behaviour that depends on specific attributes.
- To validate that required attributes are provided before rendering.
- To support both "all of" and "any of" checks via `has()` and `hasAny()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@if ($attributes->has('class'))
    <p>Class attribute is present</p>
@endif

@if ($attributes->has(['name', 'class']))
    <p>Both name and class are present</p>
@endif

@if ($attributes->hasAny(['href', ':href']))
    <p>At least one is present</p>
@endif
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `->has('key')` | Returns `true` if `key` exists. |
| `->has(['a', 'b'])` | Returns `true` if **all** exist. |
| `->hasAny(['a', 'b'])` | Returns `true` if **any** exist. |

**Syntax Rules**

- `has()` accepts a string or an array of strings.
- With an array, all attributes must be present for `has()` to return `true`.
- `hasAny()` returns `true` if at least one attribute is present.

**Constraints and Limitations**

- Keys are case-sensitive.
- An empty array returns `true` for `has()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Checking for Multiple Attributes**

**Component View**

```blade
<!-- resources/views/components/form-field.blade.php -->
<div {{ $attributes->merge(['class' => 'form-field']) }}>
    @if ($attributes->has(['label', 'name']))
        <label for="{{ $attributes->get('name') }}">{{ $attributes->get('label') }}</label>
    @endif
    <input type="text" name="{{ $attributes->get('name') }}">
</div>
```

**Usage**

```blade
<x-form-field name="email" label="Email Address" class="form-control" />
<x-form-field name="password" class="form-control" />
```

**Expected Output (first)**

```html
<div class="form-field form-control">
    <label for="email">Email Address</label>
    <input type="text" name="email">
</div>
```

**Expected Output (second)**

```html
<div class="form-field form-control">
    <input type="text" name="password">
</div>
```

**Why This Output Occurs**

In the first case, both `label` and `name` are present, so the label is rendered. In the second, `label` is missing, so the label is omitted.

---

#### Real-World Cases with Explanation

**Case 1: Conditional Label Rendering**

A form field component uses `has(['label', 'name'])` to render a `<label>` only when both attributes are provided.

**Explanation**: `has()` enables conditional rendering based on attribute presence.

---

### 15. `$attributes->first()`

#### Definitions

**Core Definition**

The `first()` method returns the value of the first attribute in the attribute bag, with an optional default if the bag is empty.

**Technical Definition**

`ComponentAttributeBag::first(mixed $default = null)` returns the value of the first attribute in the bag's internal `$attributes` array. If the bag is empty, the `$default` value is returned. This method is often used in conjunction with `whereStartsWith()` to get the first matching attribute.

**Beginner-Friendly Explanation**

`first()` gives you the value of the first attribute in the bag. It is useful when you expect only one matching attribute (e.g., after filtering).

#### Purposes

- To retrieve the first attribute's value when only one is expected.
- To combine with prefix filtering to get a specific attribute.
- To provide a default when no attributes are present.
- To support fallback logic in component rendering.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->first() }}
{{ $attributes->first('default-value') }}
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->first()` | Returns the first attribute's value or `null`. |
| `->first('default')` | Returns the first value or the default. |
| `->whereStartsWith('prefix')->first()` | Returns the first matching attribute's value. |

**Syntax Rules**

- Returns the value, not the key.
- If the bag is empty, returns the default.
- Chainable with other methods.

**Constraints and Limitations**

- The order of attributes is determined by the order they were passed to the component.
- After filtering, the "first" attribute is the first in the filtered set.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Getting the First `wire:model` Attribute**

**Component View**

```blade
<!-- resources/views/components/wire-field.blade.php -->
@php
    $wireModel = $attributes->whereStartsWith('wire:model')->first('no-model');
@endphp
<div data-model="{{ $wireModel }}">
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-wire-field wire:model="email" wire:blur="validate" class="field" />
<x-wire-field class="field" />
```

**Expected Output (first)**

```html
<div data-model="email">Content</div>
```

**Expected Output (second)**

```html
<div data-model="no-model">Content</div>
```

**Why This Output Occurs**

In the first case, `whereStartsWith('wire:model')` returns the `wire:model` attribute, and `first()` returns `"email"`. In the second, no matching attribute exists, so the default `"no-model"` is used.

---

#### Real-World Cases with Explanation

**Case 1: Extracting Livewire Model Binding**

A component uses `whereStartsWith('wire:model')->first()` to determine which property is bound to the component, enabling conditional logic based on the binding.

**Explanation**: `first()` combined with prefix filtering provides a targeted way to extract specific attribute values.

---

### 16. `$attributes->all()`

#### Definitions

**Core Definition**

The `all()` method returns all attributes in the bag as a plain associative array.

**Technical Definition**

`ComponentAttributeBag::all(mixed $keys = null)` returns the raw `$attributes` array. If `$keys` is provided (string or array), it returns only the values for the specified keys. Otherwise, it returns all attributes as an associative array where keys are attribute names and values are attribute values.

**Beginner-Friendly Explanation**

`all()` gives you the complete list of attributes as a regular PHP array. This is useful when you need to process attributes programmatically.

#### Purposes

- To retrieve all attributes as an array for programmatic manipulation.
- To extract specific attribute values in bulk.
- To pass attributes to functions or methods that expect an array.
- To support serialization or debugging of attribute data.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@php $allAttrs = $attributes->all(); @endphp
@php $specificAttrs = $attributes->all(['class', 'id']); @endphp
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->all()` | Returns all attributes as an array. |
| `->all(['a', 'b'])` | Returns only the values for keys `a` and `b`. |

**Syntax Rules**

- Returns an associative array.
- When `$keys` is provided, returns an array of values (not key-value pairs).
- Can be used with `@php` directives or in component class methods.

**Constraints and Limitations**

- Returns the raw values, not the attribute bag.
- If a key in the `$keys` array does not exist, it is omitted from the result.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Processing All Attributes**

**Component View**

```blade
<!-- resources/views/components/debug-box.blade.php -->
@php
    $allAttrs = $attributes->all();
@endphp
<div class="debug-box">
    <h4>Attributes ({{ count($allAttrs) }}):</h4>
    <ul>
        @foreach ($allAttrs as $key => $value)
            <li><strong>{{ $key }}:</strong> {{ $value }}</li>
        @endforeach
    </ul>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-debug-box id="debug-1" class="mt-4" data-type="test">Content</x-debug-box>
```

**Expected Output**

```html
<div class="debug-box">
    <h4>Attributes (3):</h4>
    <ul>
        <li><strong>id:</strong> debug-1</li>
        <li><strong>class:</strong> mt-4</li>
        <li><strong>data-type:</strong> test</li>
    </ul>
    Content
</div>
```

**Why This Output Occurs**

`all()` returns all attributes as an associative array, which is then iterated to display each key-value pair.

---

#### Real-World Cases with Explanation

**Case 1: Debugging Component Attributes**

During development, a debug component uses `all()` to display all attributes passed to a component, helping developers understand what data is available.

**Explanation**: `all()` provides a complete view of attribute data for debugging and inspection.

---

### 17. `$attributes->prepends()`

#### Definitions

**Core Definition**

The `prepends()` method returns a value that, when used within `merge()`, ensures the specified value is always placed at the beginning of the merged attribute.

**Technical Definition**

`ComponentAttributeBag::prepends(mixed $value)` is a special method designed to be used inside the `merge()` array. It returns a value that `merge()` recognises as needing to be prepended rather than appended. When merging a non-class attribute (e.g., `data-controller`), the `prepends()` value is placed before any passed values.

**Beginner-Friendly Explanation**

`prepends()` is used inside `merge()` to make sure a default value always comes first. For example, you can ensure that a `data-controller` attribute always starts with a specific controller name.

#### Purposes

- To enforce that a default value always precedes passed values in a merged attribute.
- To support attributes like `data-controller` where order matters.
- To provide a mechanism for controlled concatenation of non-class attributes.
- To complement `merge()` for attributes that need specific ordering.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$attributes->prepends('value')` | Returns a special value for use in `merge()`. |
| `merge(['key' => ...])` | Merges the prepended value with passed values. |

**Syntax Rules**

- `prepends()` must be used inside a `merge()` call.
- The returned value is handled specially by `merge()`.
- Only works for non-class attributes (class values are already concatenated with default first).

**Constraints and Limitations**

- Has no effect if used outside of `merge()`.
- The order of values is: prepended default first, then passed values.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Prepending a Controller Value**

**Component View**

```blade
<!-- resources/views/components/controller-wrapper.blade.php -->
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-controller-wrapper data-controller="user-controller" class="wrapper">
    Content
</x-controller-wrapper>
```

**Expected Output**

```html
<div data-controller="profile-controller user-controller" class="wrapper">
    Content
</div>
```

**Why This Output Occurs**

The `prepends('profile-controller')` ensures that `"profile-controller"` always appears first in the `data-controller` attribute. The passed `"user-controller"` is appended after it.

---

#### Real-World Cases with Explanation

**Case 1: Controller Ordering in Stimulus**

A Stimulus controller wrapper component uses `prepends()` to ensure a base controller is always loaded before any page-specific controllers.

**Explanation**: `prepends()` provides ordering guarantees for attribute values that depend on sequence.

---

### 18. Component Slots

#### Definitions

**Core Definition**

Component slots are named placeholders within a component's view that are filled with content passed between the component's opening and closing tags.

**Technical Definition**

The default slot is available in the component view as `$slot`, an instance of `Illuminate\View\ComponentSlot`. Named slots are defined using `<x-slot:name>` and are accessed as `$name`. Slots can be checked for content using `isEmpty()` and `hasActualContent()`. Slot content is an `HtmlString` instance.

**Beginner-Friendly Explanation**

A slot is like a blank space in a component where you can put your own content. For example, an alert component might have a slot for the main message and another slot for a title.

#### Purposes

- To allow dynamic content to be injected into a component from the parent template.
- To support multiple content areas within a single component via named slots.
- To provide a flexible API for component composition.
- To enable scoped slots where the parent can access component methods and properties.

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
    </x-slot>
    <p>Body content</p>
</x-card>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$slot` | Default slot content. |
| `$header` | Named slot content. |
| `<x-slot:header>` | Defines content for the `header` slot. |
| `$slot->isEmpty()` | Checks if the slot is empty. |
| `$slot->hasActualContent()` | Checks for non-comment content. |

**Syntax Rules**

- Content outside `<x-slot>` tags becomes the default slot.
- Named slots use `<x-slot:name>` syntax.
- The component view must echo the slot variable to render it.
- Slots can have their own attributes.

**Constraints and Limitations**

- A component can have only one default slot.
- Named slot names must be unique within a component.
- Slots cannot be nested within other slots.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Default Slot with Empty Check**

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-info">
    @if ($slot->isEmpty())
        <p>Default alert message</p>
    @else
        {{ $slot }}
    @endif
</div>
```

**Usage**

```blade
<x-alert />
<x-alert>Custom message</x-alert>
```

**Expected Output (first)**

```html
<div class="alert alert-info">
    <p>Default alert message</p>
</div>
```

**Expected Output (second)**

```html
<div class="alert alert-info">
    Custom message
</div>
```

**Why This Output Occurs**

When the slot is empty, `isEmpty()` returns `true`, and the default message is rendered. When content is provided, it is rendered instead.

---

**Example 2: Multiple Named Slots**

**Component View**

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card">
    <div class="card-header">{{ $header }}</div>
    <div class="card-body">{{ $slot }}</div>
    <div class="card-footer">{{ $footer }}</div>
</div>
```

**Usage**

```blade
<x-card>
    <x-slot:header>
        <h2>Card Title</h2>
    </x-slot>
    <p>Card body content.</p>
    <x-slot:footer>
        <p>Footer content</p>
    </x-slot>
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
    <div class="card-footer">
        <p>Footer content</p>
    </div>
</div>
```

**Why This Output Occurs**

Each named slot (`$header`, `$footer`) is rendered in its designated location, and the content outside the named slots becomes the default `$slot`.

---

#### Real-World Cases with Explanation

**Case 1: Modal Component with Header and Footer Slots**

A modal component uses a default slot for the body, a named slot for the title, and a named slot for action buttons. Different pages reuse the modal shell with custom content.

**Explanation**: Named slots provide flexibility while keeping the component's structural markup centralized.

---

### 19. Named Slots

#### Definitions

**Core Definition**

Named slots are slots that are explicitly identified by a name, allowing multiple distinct content areas within a single component.

**Technical Definition**

Named slots are defined using the `<x-slot:name>` tag within a component invocation. In the component view, they are accessed via the variable matching the slot name (e.g., `$header`, `$footer`). Named slots are instances of `ComponentSlot` and support the same methods as the default slot.

**Beginner-Friendly Explanation**

Named slots let you have multiple content areas in a component. For example, a card component might have a "header" slot, a "body" slot, and a "footer" slot.

#### Purposes

- To provide multiple distinct content areas within a single component.
- To enable more expressive component APIs with semantically named content regions.
- To support complex UI patterns like cards, modals, and panels.
- To allow conditional rendering of different slot regions.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<x-card>
    <x-slot:header>
        Header content
    </x-slot>
    Main content
    <x-slot:footer>
        Footer content
    </x-slot>
</x-card>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-slot:header>` | Defines the `header` named slot. |
| `$header` | Variable in component view for header content. |
| `$slot` | Default slot for content outside named slots. |

**Syntax Rules**

- Named slots use `<x-slot:name>` syntax.
- The name must be a valid PHP variable name (kebab-case is converted to camelCase).
- Named slots can appear in any order within the component tags.
- Content outside named slots becomes the default `$slot`.

**Constraints and Limitations**

- Slot names must be unique.
- Named slots cannot be nested.
- The component view must echo the slot variable to render it.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Card with Named Slots**

**Component View**

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card">
    @if ($title)
        <div class="card-title">{{ $title }}</div>
    @endif
    <div class="card-body">{{ $slot }}</div>
    @if ($actions)
        <div class="card-actions">{{ $actions }}</div>
    @endif
</div>
```

**Usage**

```blade
<x-card>
    <x-slot:title>User Profile</x-slot>
    <p>Name: John Doe</p>
    <p>Email: john@example.com</p>
    <x-slot:actions>
        <button>Edit</button>
        <button>Delete</button>
    </x-slot>
</x-card>
```

**Expected Output**

```html
<div class="card">
    <div class="card-title">User Profile</div>
    <div class="card-body">
        <p>Name: John Doe</p>
        <p>Email: john@example.com</p>
    </div>
    <div class="card-actions">
        <button>Edit</button>
        <button>Delete</button>
    </div>
</div>
```

**Why This Output Occurs**

The `title` and `actions` named slots are rendered in their designated locations. The content between them becomes the default `$slot`.

---

#### Real-World Cases with Explanation

**Case 1: Panel Component with Header, Body, and Footer**

A panel component uses three named slots (`header`, `default`, `footer`) to create a structured container for content.

**Explanation**: Named slots enforce a consistent structure while allowing content flexibility.

---

### 20. Scoped Slots

#### Definitions

**Core Definition**

Scoped slots allow the parent template to access data and methods from the component within the slot content, via the `$component` variable.

**Technical Definition**

In Laravel Blade, scoped slots are implemented by accessing the `$component` variable within slot content. This variable provides access to public methods and properties of the component class. The parent template can call methods like `$component->formatAlert('message')` to use component logic within the slot.

**Beginner-Friendly Explanation**

Scoped slots let you use the component's own methods and data from within the slot content you provide. It is like the component shares its tools with the content you put inside it.

#### Purposes

- To allow slot content to use component logic and formatting methods.
- To enable parent templates to access component state for conditional rendering.
- To support advanced composition patterns where slot content depends on component data.
- To provide a Vue-like scoped slot experience in Blade.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$component` | The component instance, available in slot content. |
| `->formatAlert()` | Public method on the component class. |
| `$component->property` | Public property on the component class. |

**Syntax Rules**

- The `$component` variable is automatically available within slot content.
- Any public method or property of the component class can be accessed.
- `$component` can be used in both default and named slots.

**Constraints and Limitations**

- Only public methods and properties are accessible.
- Reserved keywords cannot be used as method or property names.
- `$component` is not available outside of slot content.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using Component Method in Slot**

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

    public function formatAlert(string $message): string
    {
        return strtoupper($message);
    }
}
```

**Component View**

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-{{ $type }}">
    @if (isset($title))
        <span class="alert-title">{{ $title }}</span>
    @endif
    {{ $slot }}
</div>
```

**Usage**

```blade
<x-alert type="error">
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>
    Something went wrong!
</x-alert>
```

**Expected Output**

```html
<div class="alert alert-error">
    <span class="alert-title">SERVER ERROR</span>
    Something went wrong!
</div>
```

**Why This Output Occurs**

The `$component->formatAlert('Server Error')` call invokes the public method on the component class, which returns the uppercased string `"SERVER ERROR"`.

---

#### Real-World Cases with Explanation

**Case 1: Data Table with Formatted Columns**

A data table component provides a `formatCurrency` method that parent templates can use in scoped slots to format monetary values.

**Explanation**: Scoped slots enable parent templates to leverage component-specific formatting logic.

---

### 21. Slot Attributes

#### Definitions

**Core Definition**

Slot attributes are HTML attributes assigned to a slot (e.g., `<x-slot:heading class="font-bold">`), which are accessible in the component view via the slot's `$attributes` property.

**Technical Definition**

Each `ComponentSlot` instance has an `$attributes` property that is itself a `ComponentAttributeBag`. This allows slot attributes to be manipulated using the same methods as component attributes (merge, class, etc.). Slot attributes are accessed as `$slotName->attributes` in the component view.

**Beginner-Friendly Explanation**

You can add attributes to a slot (like a class) just like you can to a component. These slot attributes are then available in the component view for use in rendering.

#### Purposes

- To allow styling and customization of individual slots from the parent template.
- To pass additional data to the component via slot attributes.
- To enable conditional class application on slot content.
- To maintain consistency between component and slot attribute handling.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>
    Content
    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<x-slot:heading class="font-bold">` | Assigns `class` attribute to the heading slot. |
| `$heading->attributes` | Accesses the heading slot's attribute bag. |
| `$heading->attributes->class(['text-lg'])` | Conditionally applies classes to the slot. |

**Syntax Rules**

- Slot attributes are specified on the `<x-slot:name>` tag.
- Access slot attributes via `$slotName->attributes`.
- Slot attributes support the same methods as component attributes.

**Constraints and Limitations**

- Slot attributes are separate from component attributes.
- Slot attributes are only available in the component view, not in the parent.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Applying Slot Attributes**

**Component View**

```blade
<!-- resources/views/components/card.blade.php -->
@props(['heading', 'footer'])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>
    {{ $slot }}
    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

**Usage**

```blade
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Card Heading
    </x-slot>
    Body content
    <x-slot:footer class="mt-4">
        Footer content
    </x-slot>
</x-card>
```

**Expected Output**

```html
<div class="border shadow-sm">
    <h1 class="text-lg font-bold">
        Card Heading
    </h1>
    Body content
    <footer class="text-gray-700 mt-4">
        Footer content
    </footer>
</div>
```

**Why This Output Occurs**

The `heading` slot has the `font-bold` class from its attribute bag, merged with the default `text-lg`. The `footer` slot has `mt-4` merged with `text-gray-700`.

---

#### Real-World Cases with Explanation

**Case 1: Table with Styled Headers**

A table component uses slot attributes to allow different classes on header cells, enabling per-column styling.

**Explanation**: Slot attributes provide granular control over individual slot presentation.

---

### 22. Component Events

#### Definitions

**Core Definition**

Component events are notifications dispatched by a Blade component (or its class) that can be listened to by other parts of the application.

**Technical Definition**

In pure Blade components (non-Livewire), event dispatching is done through Laravel's event system via the `Event` facade or the `event()` helper. Components can dispatch events in their lifecycle methods (e.g., `mount`, `boot`) or in custom methods. These events can be listened to by application event listeners or other components. For Livewire components, the `dispatch()` method provides client-side event broadcasting.

**Beginner-Friendly Explanation**

Component events are like messages a component sends out to say "something happened." Other parts of your application can listen for these messages and react accordingly.

#### Purposes

- To notify other parts of the application when a component performs an action.
- To decouple components from the systems that need to react to their behaviour.
- To enable event-driven architecture within Blade component systems.
- To support logging, analytics, or side effects triggered by component lifecycle events.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Dispatching an event from a component class
use Illuminate\Support\Facades\Event;

public function mount()
{
    Event::dispatch(new ComponentMounted($this));
}
```

```blade
{{-- In Livewire components --}}
<button wire:click="$dispatch('user-created', { id: 1 })">Create</button>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Event::dispatch()` | Dispatches a Laravel event. |
| `$this->dispatch()` | Livewire-specific event dispatch. |
| `wire:click="$dispatch(...)"` | Blade-side Livewire event dispatch. |

**Syntax Rules**

- Pure Blade components use Laravel's event system.
- Livewire components use the `dispatch()` method.
- Events can carry payload data.
- Event listeners must be registered in the application's event service provider.

**Constraints and Limitations**

- Pure Blade components cannot dispatch client-side events without Livewire or JavaScript.
- Event dispatch in `render()` methods is discouraged for performance reasons.
- Events dispatched during component rendering may fire multiple times if the component is rendered multiple times.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dispatching a Laravel Event from a Component**

**Component Class**

```php
// app/View/Components/FormSubmit.php
namespace App\View\Components;

use Illuminate\View\Component;
use App\Events\FormSubmitted;

class FormSubmit extends Component
{
    public function __construct(public string $formId) {}

    public function render()
    {
        return view('components.form-submit');
    }

    public function handleSubmit()
    {
        event(new FormSubmitted($this->formId));
    }
}
```

**Event Listener**

```php
// app/Listeners/LogFormSubmission.php
namespace App\Listeners;

use App\Events\FormSubmitted;

class LogFormSubmission
{
    public function handle(FormSubmitted $event): void
    {
        logger("Form {$event->formId} was submitted.");
    }
}
```

**Expected Output (in log)**

```
Form contact-form was submitted.
```

**Why This Output Occurs**

The component's `handleSubmit()` method dispatches a `FormSubmitted` event with the form ID. The registered listener logs the message.

---

#### Real-World Cases with Explanation

**Case 1: Analytics Tracking on Component Mount**

A component dispatches a `ComponentRendered` event when it is mounted. An analytics listener records which components are used on which pages.

**Explanation**: Component events enable loose coupling between UI rendering and analytics tracking.

---

### 23. Component Lifecycle

#### Definitions

**Core Definition**

The component lifecycle refers to the sequence of states and methods a Blade component goes through from instantiation to rendering.

**Technical Definition**

Pure Blade components (non-Livewire) have a simpler lifecycle than Livewire components. The lifecycle includes: **construction** (class instantiation), **mount** (if defined, called after construction), **boot** (called on every request after mount/hydrate), **render** (returns the view or markup), and **hydration** (for Livewire components, re-initialization on subsequent requests). For class-based Blade components, the primary hooks are the constructor and the `render()` method.

**Beginner-Friendly Explanation**

The component lifecycle is the "journey" a component takes from being created to being displayed on the page. Pure Blade components are relatively simple: they are created, optionally run a setup method, and then render their view.

#### Purposes

- To provide hooks for executing logic at specific stages of a component's existence.
- To enable setup, data fetching, and dependency injection before rendering.
- To support state persistence and re-initialization in Livewire components.
- To allow cleanup or side effects at appropriate lifecycle points.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class Alert extends Component
{
    public function __construct(public string $type = 'info')
    {
        // Construction logic
    }

    public function mount()
    {
        // Called after construction (if defined)
    }

    public function boot()
    {
        // Called on every request
    }

    public function render()
    {
        return view('components.alert');
    }
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `__construct()` | Constructor; receives props and dependencies. |
| `mount()` | Optional; called after construction. |
| `boot()` | Optional; called on every request. |
| `render()` | Required; returns the view or markup. |

**Syntax Rules**

- The constructor receives dependency-injected services and props.
- `mount()` is called after construction but before `render()`.
- `boot()` is called on every request for Livewire components.
- `render()` must return a `View` or a string.

**Constraints and Limitations**

- Pure Blade components do not have `hydrate()`, `updating()`, or `updated()` hooks (these are Livewire-specific).
- The constructor should not perform heavy logic; use `mount()` for that.
- `render()` should not have side effects.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Lifecycle with Mount and Boot**

**Component Class**

```php
// app/View/Components/TrackedAlert.php
namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\Support\Facades\Log;

class TrackedAlert extends Component
{
    public function __construct(public string $message)
    {
        Log::info("TrackedAlert constructed: {$message}");
    }

    public function mount()
    {
        Log::info("TrackedAlert mounted: {$this->message}");
    }

    public function render()
    {
        return view('components.tracked-alert');
    }
}
```

**Expected Output (in log)**

```
TrackedAlert constructed: Hello
TrackedAlert mounted: Hello
```

**Why This Output Occurs**

The constructor logs the message, and the `mount()` method logs it again. `mount()` is called after construction.

---

#### Real-World Cases with Explanation

**Case 1: Data Loading in Mount**

A component that needs to fetch data from an API uses `mount()` to perform the fetch, keeping the constructor lightweight for dependency injection.

**Explanation**: `mount()` is the appropriate place for logic that should run after dependencies are injected.

---

### 24. Component Testing

#### Definitions

**Core Definition**

Component testing is the practice of writing automated tests that verify a Blade component renders correctly with given inputs.

**Technical Definition**

Laravel provides `$this->component(ComponentClass::class, $data)` and `$this->blade('<x-component />')` methods that return a `TestComponent` instance. This instance offers assertions including `assertSee()`, `assertSeeText()`, `assertSeeInOrder()`, `assertSeeTextInOrder()`, `assertDontSee()`, and `assertDontSeeText()`. Tests are typically written as feature tests using PHPUnit or Pest.

**Beginner-Friendly Explanation**

Testing a component means writing code that checks if your component displays the right thing. Laravel makes this easy with special testing methods that render the component and let you assert what appears in the output.

#### Purposes

- To ensure components render correctly with various inputs.
- To catch regressions when component code changes.
- To verify that slots and attributes are handled as expected.
- To document component behaviour through executable tests.

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
| `assertSeeText()` | Asserts text is present (HTML-stripped). |
| `assertSeeInOrder()` | Asserts strings appear in order. |

**Syntax Rules**

- Tests extend `Tests\TestCase`.
- Use `$this->component()` for class-based components.
- Use `$this->blade()` for anonymous components or raw Blade.
- Assertions are chainable.
- `assertSee()` escapes by default; pass `false` as the second argument for raw HTML.

**Constraints and Limitations**

- Component tests require the view engine to be booted.
- `assertSee()` may produce false positives if the string appears in HTML attributes.
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

**Example 2: Testing an Anonymous Component with `blade()`**

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

**Example 3: Testing Slot Content**

**Test Class**

```php
public function test_card_renders_named_slots(): void
{
    $view = $this->blade(
        '<x-card>
            <x-slot:header>Header Content</x-slot>
            Body Content
        </x-card>'
    );

    $view->assertSee('Header Content');
    $view->assertSee('Body Content');
    $view->assertSeeInOrder(['Header Content', 'Body Content']);
}
```

**Expected Output (Test Result)**

```
PASS  Tests\Feature\CardComponentTest
✓ card renders named slots
```

**Why This Output Occurs**

The `blade()` method renders the component with the named slot and default slot. The assertions verify both content areas and their order.

---

#### Real-World Cases with Explanation

**Case 1: UI Library Testing**

A team building a reusable UI library writes tests for each component to ensure it renders correctly with different props and slots. This prevents regressions when updating the library.

**Explanation**: Component testing is essential for maintaining a reliable component library.

---

## References

- Laravel Blade Templates Documentation (Laravel 13.x) — https://laravel.com/docs/13.x/blade
- Laravel Blade Templates Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade Templates Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/blade
- Laravel Blade Templates Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/blade
- Laravel Blade Templates Documentation (Laravel master) — https://laravel.com/framework/docs/master/blade
- Laravel API: ComponentAttributeBag — https://api.laravel.com/docs/master/Illuminate/View/ComponentAttributeBag.html
- Laravel API: TestComponent — https://api.laravel.com/docs/8.x/Illuminate/Testing/TestComponent.html
- Laravel API: ComponentSlot — https://api.laravel.com/docs/12.x/Illuminate/View/ComponentSlot.html
- Laravel Blade Attribute Bag Methods (GitHub) — https://github.com/DevStorm-Team/laravel-book/blob/94af0a42e99bc9fc555bfc0fecb1c69e7ea770d2/laravel-docs-master.pdf
- Laravel Blade Attribute Filtering (GitHub) — https://raw.githubusercontent.com/rogervila/laravel-docs/refs/heads/10.x/blade.md
- Laravel Blade 13.x PDF Documentation — https://github.com/driade/laravel-book/blob/1a41c51c3a826b6aa62296367a972d6b59ce0feb/laravel-docs-13.x.pdf
- Testing Blade/Volt Components (Laracasts) — https://laracasts.com/discuss/channels/laravel/testing-blade-volt-components
- Laravel Events Documentation — https://laravel.com/docs/12.x/events

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. The `ComponentAttributeBag` methods have remained stable across these versions, though the `style()` method and some filtering methods were added in Laravel 9–10. The `TestComponent` API was introduced in Laravel 8. Component lifecycle hooks for pure Blade components are limited to the constructor, `mount()`, and `render()`. Livewire components have a richer lifecycle with additional hooks. Developers should always consult the documentation for their specific Laravel version.