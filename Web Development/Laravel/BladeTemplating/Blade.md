# Laravel Blade Fundamentals & Syntax: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Blade is Laravel's built-in templating engine that compiles template files into plain PHP code, providing an expressive syntax for displaying data, control structures, and template inheritance while maintaining separation between application logic and presentation.

**Technical Definition:** Blade is a template compiler, not a template engine in the traditional sense. Blade template files use the `.blade.php` extension and are stored in `resources/views`. When a Blade template is first requested, the Blade compiler (`Illuminate\View\Compilers\BladeCompiler`) parses the template, converts Blade directives and echo statements into corresponding PHP code, and caches the compiled result. On subsequent requests, the cached PHP file is used directly, meaning Blade adds essentially zero overhead to the application . The compiler uses regular expressions to identify Blade constructs (`{{ }}`, `{!! !!}`, `@directive`) and transform them into PHP statements such as `echo htmlspecialchars(...)`, `<?php if(...): ?>`, and `<?php endif; ?>`.

**Beginner-Friendly Explanation:** Blade is a tool that lets you write HTML with special shortcuts for PHP logic. Instead of writing `<?php echo htmlspecialchars($name); ?>`, you write `{{ $name }}`. Instead of `<?php if ($condition): ?>`, you write `@if($condition)`. Laravel converts these shortcuts into plain PHP the first time the page is viewed, saves the converted version, and reuses it for subsequent visitors. This means you get clean, readable templates without sacrificing performance.

### Key Characteristics

- **Automatic Escaping:** `{{ }}` expressions are automatically escaped via `htmlspecialchars` to prevent XSS attacks .
- **Raw Output:** `{!! !!}` renders unescaped HTML, useful for trusted content but dangerous for user input .
- **Template Inheritance:** `@extends`, `@section`, `@yield`, and `@include` enable layout reuse and modular views.
- **Control Structures:** `@if`, `@foreach`, `@for`, `@while`, and `@switch` provide clean shortcuts for PHP control structures .
- **Zero Overhead:** Blade compiles to plain PHP and caches the result; it adds no runtime overhead beyond the initial compilation .
- **JavaScript-Friendly:** `@json`, `@js`, and `@{{ }}` handle data passing and escaping for JavaScript frameworks .
- **Native PHP Support:** Blade does not restrict the use of plain PHP code within templates .

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (Blade syntax is stable across versions; `@js` directive available from Laravel 9).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `resources/views` directory.
- Basic understanding of PHP and HTML.

### Related Programming Areas

- **MVC Architecture:** Blade is the View layer in Laravel's MVC pattern.
- **View Composers:** Blade views can receive data from view composers and shared data.
- **JavaScript Frameworks:** Blade integrates with Vue, React, and Alpine.js through directives like `@json` and `@{{ }}`.
- **Component Systems:** Blade components (`x-component`) provide reusable UI elements.
- **Localization:** Blade supports `@lang` and `__()` for multilingual applications.

### Core Concepts / Features

1. Core Syntax: Expressions and Escaping
2. Directives & Comments
3. Blade Compiler
4. JavaScript Integration

---

## 1. Core Syntax: Expressions and Escaping

### Definitions

**Core Definition:** Blade's core syntax consists of echo statements enclosed in `{{ }}` for escaped output and `{!! !!}` for raw unescaped output, along with directives prefixed with `@` for control structures and template inheritance.

**Technical Definition:** The `{{ $variable }}` syntax is compiled by Blade into `<?php echo e($variable); ?>`, where `e()` is Laravel's helper that calls `htmlspecialchars()` with `ENT_QUOTES | ENT_SUBSTITUTE` flags to prevent XSS attacks . The `{!! $variable !!}` syntax compiles to `<?php echo $variable; ?>` without escaping, allowing raw HTML to be rendered . The `{{ $variable }}` and `{!! $variable !!}` syntaxes can be used with any PHP expression, including function calls, ternary operators, and method chains.

**Beginner-Friendly Explanation:** `{{ }}` is the "safe" way to output data—it automatically converts special characters like `<` and `>` into harmless HTML entities, preventing malicious scripts from running. `{!! !!}` is the "raw" way—it outputs whatever you give it, including HTML tags. You should use `{{ }}` for almost everything, especially user-provided data. Only use `{!! !!}` for content you trust completely, like HTML you've written yourself or sanitized through a tool like HTMLPurifier .

### Purposes

- To display data from PHP variables and expressions within HTML templates.
- To prevent Cross-Site Scripting (XSS) attacks by automatically escaping user-provided content.
- To render trusted HTML content (e.g., from a WYSIWYG editor) as actual HTML.
- To provide a clean, readable syntax for outputting data without PHP tags.
- To support complex expressions including method calls, array access, and ternary operators.

### Syntax Rules and Structure

**Complete General Syntax — Escaped Output:**

```blade
{{ $variable }}
{{ $user->name }}
{{ $items[0] }}
{{ $condition ? 'Yes' : 'No' }}
{{ ucfirst($name) }}
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `{{ $var }}` | `<?php echo e($var); ?>` | Escaped output (default, safe) |
| `{!! $var !!}` | `<?php echo $var; ?>` | Raw, unescaped output |
| `{{ $var ?? 'default' }}` | `<?php echo e($var ?? 'default'); ?>` | Null coalescing |
| `{{ $var ?: 'default' }}` | `<?php echo e($var ?: 'default'); ?>` | Ternary fallback |

**Complete General Syntax — Raw Output:**

```blade
{!! $htmlContent !!}
{!! Purifier::clean($userContent) !!}
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `{!! $var !!}` | `<?php echo $var; ?>` | Raw HTML output |
| `{!! csrf_field() !!}` | `<?php echo csrf_field(); ?>` | Output form helper result |

**Syntax Rules:**

- `{{ }}` is the default and safest syntax—always use it for user-provided data.
- `{!! !!}` bypasses escaping—only use it for trusted content or after sanitization.
- The `e()` helper used by `{{ }}` calls `htmlspecialchars()` with `ENT_QUOTES | ENT_SUBSTITUTE` and double-encoding enabled by default .
- Double encoding can be disabled via `Blade::withoutDoubleEncoding()` in `AppServiceProvider::boot()` .
- Both syntaxes support any valid PHP expression, not just variables.

**Constraints and Limitations:**

- **XSS Risk:** `{!! !!}` is dangerous with user-generated content. Never use it with unsanitized user input .
- **HTMLPurifier:** For user-generated HTML, sanitize with a package like HTMLPurifier before using `{!! !!}` .
- **Double Encoding:** By default, Blade double-encodes HTML entities. Disable via `Blade::withoutDoubleEncoding()` if needed .
- **Performance:** Escaping adds a negligible overhead per echo statement.

### Multiple Annotated Complete Code Examples

**Example 1: Escaped Output with User Data**

```blade
{{-- resources/views/users/show.blade.php --}}

<div class="user-profile">
    <h1>{{ $user->name }}</h1>
    <p>Email: {{ $user->email }}</p>
    <p>Bio: {{ $user->bio }}</p>
</div>
```

```php
// Controller
return view('users.show', ['user' => $user]);
```

**Expected Output:** If `$user->name` is `<script>alert("XSS")</script>`, the output is `<h1>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</h1>`—the script is rendered as text, not executed.

**Why:** The `{{ }}` syntax escapes all HTML special characters, preventing XSS attacks. The user's name is displayed as text, not interpreted as HTML.

---

**Example 2: Raw Output with Trusted HTML**

```blade
{{-- resources/views/posts/show.blade.php --}}

<div class="post-content">
    {!! $post->body !!}
</div>
```

```php
// Controller
return view('posts.show', ['post' => $post]);
```

**Expected Output:** If `$post->body` contains `<p>Hello <strong>World</strong></p>`, the output is `<p>Hello <strong>World</strong></p>`—the HTML is rendered as actual HTML.

**Why:** The `{!! !!}` syntax does not escape HTML, allowing the trusted content (e.g., from a WYSIWYG editor or admin) to render as intended.

---

**Example 3: Escaping with HTMLPurifier for User Content**

```blade
{{-- resources/views/comments/show.blade.php --}}

<div class="comment-body">
    {!! Purifier::clean($comment->body) !!}
</div>
```

```php
// Controller
use Mews\Purifier\Facades\Purifier;

return view('comments.show', [
    'comment' => $comment,
    'body' => Purifier::clean($comment->body),
]);
```

**Expected Output:** The user's comment HTML is sanitized, removing dangerous tags like `<script>` while preserving safe formatting tags like `<p>`, `<strong>`, and `<em>`.

**Why:** HTMLPurifier removes dangerous HTML elements before the content is rendered with `{!! !!}`, balancing the need for rich text with security.

### Real-World Cases

- **User Profiles:** Display user names, emails, and bios with `{{ }}` to prevent XSS.
- **Blog Posts:** Render trusted post content with `{!! !!}` when the content is authored by admins.
- **Comments:** Sanitize user comments with HTMLPurifier before rendering with `{!! !!}`.
- **WYSIWYG Editors:** Render rich text content from editors like TinyMCE or CKEditor.
- **Form Helpers:** Use `{!! csrf_field() !!}` to output hidden CSRF token fields.

### References

- Laravel Blade Templates: Displaying Data — https://laravel.com/docs/12.x/blade#displaying-data
- Laravel Blade: Escaping Output — https://laravel.com/docs/12.x/blade#displaying-data
- Laravel Blade: HTML Entity Encoding — https://laravel.com/docs/12.x/blade#html-entity-encoding
- OWASP: XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

---

## 2. Directives & Comments

### Definitions

**Core Definition:** Blade directives are special `@`-prefixed keywords that provide shortcuts for PHP control structures, template inheritance, authentication checks, and other common tasks; Blade comments (`{{-- --}}`) are non-rendering comments that are removed during compilation.

**Technical Definition:** Blade directives are compiled into corresponding PHP code by the `BladeCompiler`. For example, `@if($condition)` compiles to `<?php if($condition): ?>` and `@endif` compiles to `<?php endif; ?>`. Blade comments (`{{-- --}}`) are stripped from the template entirely—they do not appear in the compiled PHP or the rendered HTML . HTML comments (`<!-- -->`) are preserved in the rendered output. The `@json` directive compiles to `<?php echo json_encode($var); ?>` and is used to safely pass PHP data to JavaScript .

**Beginner-Friendly Explanation:** Directives are like shortcuts for common PHP tasks. Instead of writing `<?php if ($user->isAdmin()): ?>`, you write `@if($user->isAdmin())`. Instead of `<?php endif; ?>`, you write `@endif`. Blade comments are like notes to yourself that never appear in the final page—they're removed entirely during compilation. HTML comments, on the other hand, are sent to the browser and can be seen by anyone who views the page source.

### Purposes

- To provide clean, readable shortcuts for PHP control structures (`@if`, `@foreach`, `@while`).
- To enable template inheritance and layout reuse (`@extends`, `@section`, `@yield`, `@include`).
- To handle authentication checks concisely (`@auth`, `@guest`).
- To pass PHP data to JavaScript safely (`@json`).
- To document template logic without sending comments to the browser (`{{-- --}}`).
- To conditionally include content based on environment or user state (`@env`, `@production`).

### Syntax Rules and Structure

**Complete General Syntax — Blade Comments:**

```blade
{{-- This comment will NOT appear in the rendered HTML --}}

{{--
    Multi-line Blade comment
    with multiple lines
--}}
```

**Component Breakdown:**

| Syntax | Appears in HTML? | Purpose |
|--------|------------------|---------|
| `{{-- --}}` | No | Blade comment (removed during compilation) |
| `<!-- -->` | Yes | HTML comment (visible in page source) |
| `//` or `/* */` | In `@php` blocks | PHP comments |

**Complete General Syntax — Control Structure Directives:**

```blade
@if($condition)
    <p>Condition is true</p>
@elseif($otherCondition)
    <p>Other condition is true</p>
@else
    <p>All conditions are false</p>
@endif

@foreach($items as $item)
    <p>{{ $item->name }}</p>
@endforeach

@for($i = 0; $i < 10; $i++)
    <p>{{ $i }}</p>
@endfor

@while($condition)
    <p>Looping...</p>
@endwhile
```

**Complete General Syntax — Authentication Directives:**

```blade
@auth
    <p>Welcome, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <p>Please log in.</p>
@endguest

@auth('admin')
    <p>Admin panel</p>
@endauth
```

**Complete General Syntax — `@json` Directive:**

```blade
<script>
    var app = @json($array);
    var user = @json($user->toArray());
    var config = @json($config, JSON_PRETTY_PRINT);
</script>
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `@json($var)` | `<?php echo json_encode($var); ?>` | Encode PHP data as JSON |
| `@json($var, JSON_PRETTY_PRINT)` | `<?php echo json_encode($var, JSON_PRETTY_PRINT); ?>` | JSON with formatting |

**Complete General Syntax — `@js` Directive (Laravel 9+):**

```blade
<script>
    let posts = @js($posts);
    let user = @js($user);
</script>
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `@js($var)` | `JSON.parse('...')` with proper escaping | Safely pass PHP data to JavaScript |

**Syntax Rules:**

- Blade comments (`{{-- --}}`) are removed entirely during compilation—they do not appear in compiled PHP or rendered HTML .
- HTML comments (`<!-- -->`) are preserved and visible in the page source.
- Directives are case-sensitive and must be written as `@if`, `@endif`, etc.
- The `@json` directive is equivalent to `json_encode()` but more readable .
- The `@js` directive (Laravel 9+) provides additional XSS protection by escaping the JSON output .
- Control structure directives compile to PHP alternative syntax (`if(): endif;`).

**Constraints and Limitations:**

- **Blade comments cannot be used inside `@php` blocks**—use PHP comments there.
- **`@json` inside HTML attributes:** When using `@json` inside an HTML attribute, wrap the attribute value in single quotes to avoid conflicts with the JSON double quotes.
- **`@js` vs. `@json`:** `@js` is preferred for passing data to JavaScript as it includes additional escaping to prevent XSS .
- **Comments in `@verbatim`:** Blade comments inside `@verbatim` blocks are not processed.

### Multiple Annotated Complete Code Examples

**Example 1: Blade Comments vs. HTML Comments**

```blade
{{-- This Blade comment will NOT appear in the rendered HTML --}}

<!-- This HTML comment WILL appear in the rendered HTML -->

<div class="content">
    {{-- Loop through users --}}
    @foreach($users as $user)
        <p>{{ $user->name }}</p>
    @endforeach
</div>
```

**Rendered HTML:**

```html
<!-- This HTML comment WILL appear in the rendered HTML -->

<div class="content">
    <p>Alice</p>
    <p>Bob</p>
    <p>Charlie</p>
</div>
```

**Why:** The Blade comment is stripped during compilation, while the HTML comment is preserved. This makes Blade comments ideal for template documentation that should not be visible to end users.

---

**Example 2: Control Structure Directives**

```blade
@if(count($records) === 1)
    <p>I have one record!</p>
@elseif(count($records) > 1)
    <p>I have multiple records!</p>
@else
    <p>I don't have any records!</p>
@endif

@unless(Auth::check())
    <p>You are not signed in.</p>
@endunless

@isset($records)
    <p>Records is defined and not null.</p>
@endisset

@empty($records)
    <p>Records is empty.</p>
@endempty
```

**Expected Output:** Depending on the values of `$records` and the authentication state, the appropriate messages are displayed.

**Why:** Blade directives provide clean shortcuts for PHP control structures. `@unless` is the inverse of `@if`, `@isset` checks if a variable is defined and not null, and `@empty` checks if a variable is empty .

---

**Example 3: `@json` and `@js` for JavaScript Integration**

```blade
<script>
    // Using @json
    var app = @json($array);

    // Using @js (Laravel 9+)
    let posts = @js($posts);
    let user = @js($user);

    // Using @json with options
    var config = @json($config, JSON_PRETTY_PRINT);
</script>
```

**Expected Output:**

```html
<script>
    var app = {"name":"Laravel","version":"12"};

    let posts = JSON.parse('{"id":1,"title":"Hello"}');
    let user = JSON.parse('{"id":1,"name":"Alice"}');

    var config = {
        "debug": true,
        "locale": "en"
    };
</script>
```

**Why:** `@json` uses `json_encode()` to convert PHP data to JSON. `@js` uses `JSON.parse()` with additional escaping to prevent XSS when embedding data in JavaScript contexts .

### Real-World Cases

- **Layout Documentation:** Use `{{-- --}}` to document template sections without exposing them to users.
- **Conditional Rendering:** Use `@auth`, `@guest`, `@if` to conditionally display content based on user state.
- **Data Initialization:** Use `@json` or `@js` to initialize JavaScript variables with server-side data.
- **Looping:** Use `@foreach` to iterate over collections for rendering lists.
- **Environment Checks:** Use `@env('production')` to conditionally include analytics scripts.

### References

- Laravel Blade: Comments — https://laravel.com/docs/12.x/blade#comments
- Laravel Blade: Control Structures — https://laravel.com/docs/12.x/blade#control-structures
- Laravel Blade: Rendering JSON — https://laravel.com/docs/12.x/blade#rendering-json
- Laravel Blade: The @js Directive — https://laravel.com/docs/12.x/blade#the-js-directive
- Laravel 9 Release Notes: @js Directive — https://laravel.com/docs/9.x/releases

---

## 3. Blade Compiler

### Definitions

**Core Definition:** The Blade compiler is the component that transforms Blade template files (`.blade.php`) containing Blade syntax into plain PHP files, which are then cached and executed by the PHP interpreter.

**Technical Definition:** The `BladeCompiler` class (`Illuminate\View\Compilers\BladeCompiler`) extends the `Compiler` class and uses a series of regular expression-based replacements to convert Blade syntax into PHP. The compilation process occurs when a view is first rendered. The compiled PHP file is stored in `storage/framework/views/` with a hashed filename. On subsequent requests, the compiled file is used directly if the original Blade file has not been modified . The `isExpired()` method compares the modification timestamps of the source and compiled files to determine if recompilation is needed.

**Beginner-Friendly Explanation:** Blade templates aren't executed directly—they're first converted into plain PHP. This conversion happens automatically the first time a view is rendered. The resulting PHP file is saved so that future visitors don't have to wait for the conversion again. If you change the Blade template, Laravel detects the change and recompiles it. It's like translating a document from one language to another: you do the translation once, then everyone can read the translated version without waiting for a new translation.

### Purposes

- To transform Blade's expressive syntax into executable PHP code.
- To cache compiled templates for performance, avoiding recompilation on every request.
- To automatically detect changes to source templates and recompile when needed.
- To enable template inheritance and directives without runtime overhead .
- To provide a clean separation between template authoring (Blade) and execution (PHP).

### Syntax Rules and Structure

**Complete General Syntax — Compilation Process:**

```
Blade Template (.blade.php)
    ↓
BladeCompiler::compile()
    ↓
Compiled PHP (storage/framework/views/hash.php)
    ↓
PHP Interpreter executes compiled file
    ↓
Rendered HTML sent to browser
```

**Component Breakdown:**

| Step | Description |
|------|-------------|
| Blade Template | Source file in `resources/views` with `.blade.php` extension. |
| BladeCompiler | Compiles directives and echo statements to PHP. |
| Compiled PHP | Cached file in `storage/framework/views`. |
| PHP Interpreter | Executes the compiled PHP and generates HTML. |

**Complete General Syntax — Compilation Example:**

```blade
{{-- Source Blade template --}}
<h1>{{ $title }}</h1>
@if($user)
    <p>Welcome, {{ $user->name }}!</p>
@endif
```

```php
<?php
// Compiled PHP output
?>
<h1><?php echo e($title); ?></h1>
<?php if($user): ?>
    <p>Welcome, <?php echo e($user->name); ?>!</p>
<?php endif; ?>
```

**Syntax Rules:**

- Blade templates use the `.blade.php` extension and are stored in `resources/views`.
- The compiler runs automatically when a view is first rendered.
- Compiled files are stored in `storage/framework/views` with hashed filenames.
- Recompilation occurs when the source template's modification time is newer than the compiled file's.
- The `BladeCompiler` class can be resolved from the container for custom compilation.
- The `Blade::compileString()` method can compile a string without a file.

**Constraints and Limitations:**

- **Write permissions:** `storage/framework/views` must be writable by the web server.
- **Cache clearing:** `php artisan view:clear` removes all compiled views.
- **Performance:** Compilation adds overhead on the first render only; subsequent renders use the cached compiled file .
- **Debugging:** Errors in Blade templates may reference compiled file line numbers, not the original Blade file.

### Multiple Annotated Complete Code Examples

**Example 1: Simple Compilation**

```blade
{{-- resources/views/greeting.blade.php --}}
<h1>Hello, {{ $name }}!</h1>

@if($isAdmin)
    <p>You are an admin.</p>
@endif
```

```php
<?php
// Compiled output in storage/framework/views/{hash}.php
?>
<h1>Hello, <?php echo e($name); ?>!</h1>

<?php if($isAdmin): ?>
    <p>You are an admin.</p>
<?php endif; ?>
```

**Expected Output:** The rendered HTML displays the greeting and admin message (if `$isAdmin` is true).

**Why:** The Blade compiler converts `{{ $name }}` to `<?php echo e($name); ?>` and `@if`/`@endif` to `<?php if(): ?>`/`<?php endif; ?>`.

---

**Example 2: Compiling a String with `Blade::compileString()`**

```php
use Illuminate\Support\Facades\Blade;

$bladeString = 'Hello, {{ $name }}!';
$compiled = Blade::compileString($bladeString);

// $compiled = 'Hello, <?php echo e($name); ?>!';
```

**Expected Output:** The string `Hello, <?php echo e($name); ?>!` is returned.

**Why:** The `compileString()` method compiles Blade syntax without requiring a file, useful for testing or dynamic template generation.

---

**Example 3: Clearing Compiled Views**

```bash
php artisan view:clear
```

**Expected Output:** `Compiled views cleared!` — all files in `storage/framework/views` are deleted.

**Why:** Clearing compiled views forces Laravel to recompile all Blade templates on the next request, which is useful after making changes or troubleshooting rendering issues.

### Real-World Cases

- **Template Inheritance:** `@extends` and `@section` are compiled into PHP `include` statements and variable assignments.
- **Component Rendering:** Blade components (`<x-alert>`) are compiled into PHP calls to the component renderer.
- **Custom Directives:** `Blade::directive()` registers custom directives that the compiler transforms into PHP.
- **Performance Optimization:** Compiled views avoid parsing overhead on every request .
- **Development Workflow:** `view:clear` is run after template changes during development.

### References

- Laravel Blade: Blade & JavaScript Frameworks — https://laravel.com/docs/12.x/blade#blade-and-javascript-frameworks
- Laravel Blade: Introduction — https://laravel.com/docs/12.x/blade#introduction
- Laravel View: Compiling Views — https://laravel.com/docs/12.x/views
- Laravel API: BladeCompiler — https://api.laravel.com/docs/12.x/Illuminate/View/Compilers/BladeCompiler.html

---

## 4. JavaScript Integration

### Definitions

**Core Definition:** JavaScript integration in Blade refers to the mechanisms for safely passing PHP data to JavaScript, preventing conflicts between Blade and JavaScript framework syntax, and handling variable interpolation in JavaScript contexts.

**Technical Definition:** Blade provides three primary mechanisms for JavaScript integration: (1) `@json($var)` compiles to `json_encode($var)`, rendering PHP data as a JavaScript object literal; (2) `@js($var)` (Laravel 9+) compiles to `JSON.parse('...')` with additional escaping to prevent XSS; (3) `@{{ }}` tells Blade to leave the expression untouched so that JavaScript frameworks like Vue, React, or Alpine.js can process it . The `@verbatim` directive wraps large blocks of JavaScript template code to prevent Blade from processing `{{ }}` expressions .

**Beginner-Friendly Explanation:** When you have data in PHP (like a list of posts) and you want to use it in JavaScript, you can't just write `{{ $posts }}` because that would output PHP's array representation, not JavaScript. Blade's `@json` and `@js` directives convert PHP data into proper JavaScript objects. And when you're using a JavaScript framework like Vue that also uses `{{ }}` syntax, Blade's `@{{ }}` tells Laravel "don't touch this—it's for JavaScript."

### Purposes

- To render PHP data as JSON for JavaScript initialization via `@json` and `@js`.
- To prevent Blade from processing JavaScript framework expressions via `@{{ }}`.
- To wrap large blocks of JavaScript template code in `@verbatim` to avoid `@` prefixing every expression.
- To safely escape JSON output to prevent XSS in JavaScript contexts.
- To pass complex data structures (arrays, Eloquent models) to JavaScript frameworks.

### Syntax Rules and Structure

**Complete General Syntax — `@json` Directive:**

```blade
<script>
    var app = @json($array);
    var user = @json($user->toArray());
    var config = @json($config, JSON_PRETTY_PRINT);
</script>
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `@json($var)` | `<?php echo json_encode($var); ?>` | Encode PHP data as JSON |
| `@json($var, JSON_PRETTY_PRINT)` | `json_encode()` with options | Formatted JSON |
| `@json($var, JSON_HEX_TAG)` | `json_encode()` with hex encoding | Additional XSS protection |

**Complete General Syntax — `@js` Directive (Laravel 9+):**

```blade
<script>
    let posts = @js($posts);
    let user = @js($user);
    let settings = @js($settings);
</script>
```

**Component Breakdown:**

| Syntax | Compiled To | Purpose |
|--------|-------------|---------|
| `@js($var)` | `JSON.parse('...')` with escaping | Safe JavaScript data transfer |
| `@js($var)` | Handles `</script>` escaping | Prevents script injection |

**Complete General Syntax — `@{{ }}` for JavaScript Frameworks:**

```blade
<div id="app">
    <h1>@{{ title }}</h1>
    <ul>
        <li v-for="item in items">@{{ item.name }}</li>
    </ul>
</div>
```

**Component Breakdown:**

| Syntax | Rendered As | Purpose |
|--------|-------------|---------|
| `@{{ expression }}` | `{{ expression }}` | Leave expression for JavaScript framework |
| `@` prefix | Removed by Blade | Escape Blade processing |

**Complete General Syntax — `@verbatim` Directive:**

```blade
@verbatim
<div id="app">
    <h1>{{ title }}</h1>
    <ul>
        <li v-for="item in items">{{ item.name }}</li>
    </ul>
</div>
@endverbatim
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@verbatim` | Prevents Blade from processing `{{ }}` expressions |
| `@endverbatim` | Closes the verbatim block |

**Syntax Rules:**

- `@json` is equivalent to `json_encode()` and is useful for embedding JSON data in JavaScript .
- `@js` (Laravel 9+) provides additional escaping to prevent XSS and handles `</script>` sequences .
- `@{{ }}` tells Blade to leave the expression untouched—the `@` is removed, and the `{{ }}` remains for JavaScript .
- `@verbatim` wraps large blocks of JavaScript template code to avoid prefixing every expression with `@` .
- The `@js` directive is preferred over `@json` for JavaScript contexts because of its additional security escaping.

**Constraints and Limitations:**

- **`@json` in HTML attributes:** When using `@json` inside HTML attributes, wrap the attribute value in single quotes: `data-config='@json($config)'` .
- **`@js` XSS protection:** `@js` escapes the JSON output to prevent XSS, making it safer than `@json` for user-provided data .
- **Vue/React compatibility:** `@{{ }}` is required when using Blade alongside JavaScript frameworks that use `{{ }}` syntax.
- **`@verbatim` scope:** `@verbatim` blocks cannot contain Blade directives that need to be processed.

### Multiple Annotated Complete Code Examples

**Example 1: `@json` for Data Initialization**

```blade
{{-- resources/views/dashboard.blade.php --}}
<script>
    var stats = @json($stats);
    var user = @json($user->toArray());
</script>
```

```php
// Controller
return view('dashboard', [
    'stats' => ['views' => 1500, 'likes' => 320],
    'user' => $user,
]);
```

**Rendered HTML:**

```html
<script>
    var stats = {"views":1500,"likes":320};
    var user = {"id":1,"name":"Alice","email":"alice@example.com"};
</script>
```

**Why:** The `@json` directive converts PHP arrays and objects into JavaScript object literals, making the data available for client-side scripts.

---

**Example 2: `@js` for Safer Data Transfer**

```blade
{{-- resources/views/posts/index.blade.php --}}
<script>
    let posts = @js($posts);
    let currentUser = @js(auth()->user());
</script>
```

```php
// Controller
return view('posts.index', ['posts' => $posts]);
```

**Rendered HTML:**

```html
<script>
    let posts = JSON.parse('[...]');
    let currentUser = JSON.parse('{"id":1,"name":"Alice"}');
</script>
```

**Why:** The `@js` directive uses `JSON.parse()` with proper escaping, providing additional protection against XSS when the data contains user-provided content .

---

**Example 3: `@{{ }}` and `@verbatim` for Vue.js Integration**

```blade
{{-- Using @{{ }} for individual expressions --}}
<div id="app">
    <h1>@{{ title }}</h1>
    <p>@{{ message }}</p>
</div>

{{-- Using @verbatim for large blocks --}}
@verbatim
<div id="app">
    <h1>{{ title }}</h1>
    <ul>
        <li v-for="item in items">{{ item.name }}</li>
    </ul>
</div>
@endverbatim
```

**Rendered HTML:**

```html
<!-- @{{ }} result -->
<div id="app">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
</div>

<!-- @verbatim result -->
<div id="app">
    <h1>{{ title }}</h1>
    <ul>
        <li v-for="item in items">{{ item.name }}</li>
    </ul>
</div>
```

**Why:** The `@{{ }}` syntax and `@verbatim` directive prevent Blade from processing `{{ }}` expressions, allowing Vue.js to handle them .

### Real-World Cases

- **SPA Initialization:** Use `@json` or `@js` to pass initial state to Vue, React, or Alpine.js applications.
- **Chart Data:** Pass data to Chart.js or D3.js via `@json`.
- **Vue/React Templates:** Use `@{{ }}` and `@verbatim` when Blade and Vue/React share a template.
- **Configuration:** Embed application configuration in JavaScript for client-side use.
- **User Data:** Pass the authenticated user's data to JavaScript for client-side rendering.

### References

- Laravel Blade: Rendering JSON — https://laravel.com/docs/12.x/blade#rendering-json
- Laravel Blade: The @js Directive — https://laravel.com/docs/12.x/blade#the-js-directive
- Laravel Blade: Blade & JavaScript Frameworks — https://laravel.com/docs/12.x/blade#blade-and-javascript-frameworks
- Laravel Blade: The @verbatim Directive — https://laravel.com/docs/12.x/blade#the-verbatim-directive

---

## References

- Laravel Blade Templates Documentation (12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade: Displaying Data — https://laravel.com/docs/12.x/blade#displaying-data
- Laravel Blade: Control Structures — https://laravel.com/docs/12.x/blade#control-structures
- Laravel Blade: Comments — https://laravel.com/docs/12.x/blade#comments
- Laravel Blade: Rendering JSON — https://laravel.com/docs/12.x/blade#rendering-json
- Laravel Blade: The @js Directive — https://laravel.com/docs/12.x/blade#the-js-directive
- Laravel Blade: Blade & JavaScript Frameworks — https://laravel.com/docs/12.x/blade#blade-and-javascript-frameworks
- Laravel Blade: The @verbatim Directive — https://laravel.com/docs/12.x/blade#the-verbatim-directive
- Laravel Blade: HTML Entity Encoding — https://laravel.com/docs/12.x/blade#html-entity-encoding
- Laravel Views Documentation — https://laravel.com/docs/12.x/views
- Laravel API: BladeCompiler — https://api.laravel.com/docs/12.x/Illuminate/View/Compilers/BladeCompiler.html
- Laravel 9 Release Notes: @js Directive — https://laravel.com/docs/9.x/releases
- OWASP: XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html