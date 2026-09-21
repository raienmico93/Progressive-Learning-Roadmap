# Laravel Blade Control Structures: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Blade control structures are the `@`-prefixed directives that provide expressive shortcuts for PHP conditionals, loops, form handling, authentication checks, environment detection, and session/error management within Laravel's templating engine.

**Technical Definition:** Blade control structures are compiled by the `BladeCompiler` into native PHP control structures using alternative syntax (`if(): endif;`, `foreach(): endforeach;`, etc.). Directives such as `@if`, `@foreach`, and `@while` are transformed into their PHP equivalents during compilation, while form directives like `@csrf` and `@method` generate hidden input fields. Authentication directives (`@auth`, `@guest`, `@can`) integrate with Laravel's guard and gate systems, environment directives (`@env`, `@production`) check `APP_ENV`, and session directives (`@session`, `@error`) interact with session data and validation error bags.

**Beginner-Friendly Explanation:** Blade control structures are shortcuts that make your templates cleaner and more readable. Instead of writing `<?php if ($user): ?>`, you write `@if($user)`. Instead of manually adding a CSRF token to every form, you write `@csrf`. These directives handle the boilerplate so you can focus on building your page. They compile down to regular PHP, so there's no performance penalty.

### Key Characteristics

- **PHP-Equivalent Behaviour:** Every Blade control directive compiles to its PHP counterpart, meaning they function identically to native PHP control structures.
- **Automatic CSRF Handling:** `@csrf` generates the hidden token field required for Laravel's CSRF protection middleware.
- **HTTP Method Spoofing:** `@method` generates a hidden `_method` field to emulate PUT, PATCH, and DELETE requests from HTML forms.
- **Conditional HTML Attributes:** Directives like `@checked`, `@selected`, `@disabled`, `@readonly`, and `@required` conditionally render boolean HTML attributes.
- **Authorization Integration:** `@can`, `@cannot`, and `@canany` integrate with Laravel's Gate and Policy system.
- **The `$loop` Variable:** Inside `@foreach` and `@forelse` loops, a `$loop` variable provides iteration metadata (index, first, last, depth, parent).
- **Error Handling:** `@error` checks for validation error messages on specific attributes; `$errors` provides access to the full error bag.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (Blade control structures are stable across versions; `@session` available from Laravel 10.38).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with Blade templates in `resources/views`.
- Basic understanding of PHP control structures, HTML forms, and Laravel's authentication system.

### Related Programming Areas

- **MVC Architecture:** Blade control structures are used in the View layer of Laravel's MVC pattern.
- **Form Handling:** `@csrf` and `@method` are essential for secure form submissions.
- **Authentication & Authorization:** `@auth`, `@guest`, `@can`, and `@cannot` integrate with Laravel's guard and gate systems.
- **Validation:** `@error` and `$errors` handle validation error display.
- **Session Management:** `@session` checks for session values without verbose `@if` statements.

### Core Concepts / Features

1. Conditionals (`@if`, `@unless`, `@isset`, `@empty`, Ternary Shortcut)
2. Form Directives (`@csrf`, `@method`, Conditional Attributes)
3. Loops (`@for`, `@foreach`, `@forelse`, `@while`, `$loop` Variable)
4. Environment & Auth Directives (`@env`, `@production`, `@auth`, `@guest`, `@can`, `@cannot`)
5. Session Directives (`@session`, `@error`)

---

## 1. Conditionals

### Definitions

**Core Definition:** Blade conditional directives provide expressive shortcuts for PHP conditional statements, allowing templates to conditionally render content based on variable values, authentication state, or other conditions.

**Technical Definition:** The `@if`, `@elseif`, `@else`, and `@endif` directives compile to PHP's `if`, `elseif`, `else`, and `endif` statements. The `@unless` directive is the inverse of `@if`, compiling to `if (! condition)`. The `@isset` and `@empty` directives are shortcuts for PHP's `isset()` and `empty()` functions. Blade also supports the ternary operator within `{{ }}` expressions for inline conditionals.

**Beginner-Friendly Explanation:** These are the "if-then-else" shortcuts for your templates. Use `@if` when you want to show something only under certain conditions. Use `@unless` when you want to show something only when a condition is *not* true. Use `@isset` to check if a variable exists and isn't null. Use `@empty` to check if a variable is empty (empty string, empty array, null, etc.).

### Purposes

- To conditionally render content based on variable values or application state.
- To provide the inverse of a condition via `@unless` without negation operators.
- To check if a variable is defined and not null via `@isset`.
- To check if a variable is empty via `@empty`.
- To write inline conditional expressions via the ternary operator within `{{ }}`.
- To combine multiple conditions with `@elseif` for complex branching logic.

### Syntax Rules and Structure

**Complete General Syntax — `@if`/`@elseif`/`@else`:**

```blade
@if($condition)
    <p>Condition is true</p>
@elseif($otherCondition)
    <p>Other condition is true</p>
@else
    <p>All conditions are false</p>
@endif
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@if($cond)` | `<?php if($cond): ?>` | Standard conditional |
| `@elseif($cond)` | `<?php elseif($cond): ?>` | Additional condition |
| `@else` | `<?php else: ?>` | Fallback |
| `@endif` | `<?php endif; ?>` | End of conditional |

**Complete General Syntax — `@unless`:**

```blade
@unless(Auth::check())
    <p>You are not signed in.</p>
@endunless
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@unless($cond)` | `<?php if (! $cond): ?>` | Negated conditional |
| `@endunless` | `<?php endif; ?>` | End of unless block |

**Complete General Syntax — `@isset`:**

```blade
@isset($records)
    <p>Records is defined and not null.</p>
@endisset
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@isset($var)` | `<?php if(isset($var)): ?>` | Check defined and not null |
| `@endisset` | `<?php endif; ?>` | End of isset block |

**Complete General Syntax — `@empty`:**

```blade
@empty($records)
    <p>Records is empty.</p>
@endempty
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@empty($var)` | `<?php if(empty($var)): ?>` | Check if empty |
| `@endempty` | `<?php endif; ?>` | End of empty block |

**Complete General Syntax — Ternary Shortcut:**

```blade
{{ $posts->isEmpty() ? 'No posts' : $posts->count() . ' posts' }}
{{ $name ?: 'Guest' }}
```

**Component Breakdown:**

| Syntax | Purpose |
|--------|---------|
| `{{ $cond ? 'A' : 'B' }}` | Full ternary |
| `{{ $var ?: 'default' }}` | Short ternary (elvis operator) |

**Syntax Rules:**

- `@if` blocks must be closed with `@endif`.
- `@unless` is equivalent to `@if (! condition)`.
- `@isset` checks if a variable is defined and not null—it does not check for empty values.
- `@empty` checks if a variable is empty (empty string, `0`, `"0"`, `null`, `false`, empty array).
- The ternary operator within `{{ }}` is useful for simple inline conditionals.
- `@elseif` and `@else` must appear before `@endif`.

**Constraints and Limitations:**

- **`@isset` vs. `@empty`:** Use `@isset` to check if a variable exists; use `@empty` to check if its value is empty. They are not interchangeable.
- **Negation preference:** `@unless` is preferred over `@if(! $condition)` for readability.
- **Ternary complexity:** Avoid complex nested ternaries in templates—they reduce readability.

### Multiple Annotated Complete Code Examples

**Example 1: Conditional Display with `@if`/`@elseif`/`@else`**

```blade
{{-- resources/views/users/show.blade.php --}}

@if($user->role === 'admin')
    <span class="badge badge-danger">Administrator</span>
@elseif($user->role === 'editor')
    <span class="badge badge-warning">Editor</span>
@else
    <span class="badge badge-secondary">Member</span>
@endif
```

**Expected Output:** If `$user->role` is `'admin'`, the badge `Administrator` is displayed. If `'editor'`, the `Editor` badge is displayed. Otherwise, `Member` is displayed.

**Why:** The `@if`/`@elseif`/`@else` directives provide clean branching logic for multiple conditions, compiling to PHP's native `if`/`elseif`/`else` statements.

---

**Example 2: `@unless` for Negated Conditions**

```blade
{{-- resources/views/layouts/navigation.blade.php --}}

@unless(Auth::check())
    <a href="{{ route('login') }}">Login</a>
    <a href="{{ route('register') }}">Register</a>
@endunless
```

**Expected Output:** If the user is not authenticated, login and register links are displayed. If authenticated, nothing is displayed.

**Why:** `@unless` is the inverse of `@if`—it renders its contents only when the condition is false. This is more readable than `@if(! Auth::check())`.

---

**Example 3: `@isset` and `@empty`**

```blade
{{-- resources/views/products/show.blade.php --}}

@isset($product->discount)
    <p>Discount: {{ $product->discount }}%</p>
@endisset

@empty($product->reviews)
    <p>No reviews yet.</p>
@endempty
```

**Expected Output:** If `$product->discount` is set, the discount is displayed. If `$product->reviews` is empty, the "No reviews yet" message is displayed.

**Why:** `@isset` checks if the discount property is defined and not null. `@empty` checks if the reviews collection is empty. Each directive is used for its specific purpose.

---

**Example 4: Ternary Shortcut**

```blade
{{-- resources/views/posts/index.blade.php --}}

<p>{{ $posts->count() > 0 ? 'Posts available' : 'No posts found' }}</p>
<p>Welcome, {{ $user->name ?: 'Guest' }}!</p>
```

**Expected Output:** If there are posts, "Posts available" is displayed; otherwise, "No posts found". The user's name is displayed, or "Guest" if the name is empty.

**Why:** The ternary operator provides a concise inline conditional within `{{ }}` expressions, avoiding the need for full `@if` blocks for simple conditions.

### Real-World Cases

- **User Roles:** Display role-specific badges or menus using `@if`/`@elseif`/`@else`.
- **Authentication State:** Show login/register links only for guests using `@unless`.
- **Optional Data:** Display optional fields (discounts, metadata) only when present using `@isset`.
- **Empty States:** Show "no results" messages when collections are empty using `@empty`.
- **Inline Labels:** Use the ternary shortcut for simple binary labels (e.g., "Active" vs. "Inactive").

### References

- Laravel Blade: If Statements — https://laravel.com/docs/12.x/blade#if-statements
- Laravel Blade: Unless — https://laravel.com/docs/12.x/blade#if-statements
- Laravel Blade: Isset and Empty — https://laravel.com/docs/12.x/blade#if-statements

---

## 2. Form Directives

### Definitions

**Core Definition:** Blade form directives provide shortcuts for generating hidden form fields (`@csrf`, `@method`) and conditionally rendering boolean HTML attributes (`@disabled`, `@readonly`, `@required`, `@checked`, `@selected`).

**Technical Definition:** The `@csrf` directive generates a hidden `<input type="hidden" name="_token" value="...">` field containing the CSRF token, which the `ValidateCsrfToken` middleware validates on POST, PUT, PATCH, and DELETE requests. The `@method` directive generates a hidden `<input type="hidden" name="_method" value="PUT">` field, enabling HTTP method spoofing for forms that cannot natively send PUT, PATCH, or DELETE requests. The conditional attribute directives compile to `<?php if($condition): echo 'disabled'; endif; ?>` and similar, rendering the boolean attribute only when the condition is true.

**Beginner-Friendly Explanation:** HTML forms can only send GET and POST requests, but Laravel's RESTful routing uses PUT, PATCH, and DELETE for updates and deletions. The `@method` directive lets you "spoof" those methods. The `@csrf` directive adds a security token that proves the form was submitted from your own site, preventing cross-site request forgery attacks. The conditional attribute directives let you check/uncheck checkboxes, select dropdown options, and disable/readonly inputs based on data, without writing messy `@if` blocks inside your HTML tags.

### Purposes

- To generate the hidden CSRF token field required for form security via `@csrf`.
- To emulate PUT, PATCH, and DELETE requests via `@method`.
- To conditionally render the `checked` attribute on checkboxes and radio buttons via `@checked`.
- To conditionally render the `selected` attribute on dropdown options via `@selected`.
- To conditionally render the `disabled` attribute on form elements via `@disabled`.
- To conditionally render the `readonly` attribute on form elements via `@readonly`.
- To conditionally render the `required` attribute on form elements via `@required`.

### Syntax Rules and Structure

**Complete General Syntax — `@csrf`:**

```blade
<form method="POST" action="/profile">
    @csrf
    <!-- Other form fields -->
</form>
```

**Component Breakdown:**

| Directive | Generated HTML | Purpose |
|-----------|---------------|---------|
| `@csrf` | `<input type="hidden" name="_token" value="...">` | CSRF token field |

**Complete General Syntax — `@method`:**

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')
    <!-- Other form fields -->
</form>
```

**Component Breakdown:**

| Directive | Generated HTML | Purpose |
|-----------|---------------|---------|
| `@method('PUT')` | `<input type="hidden" name="_method" value="PUT">` | HTTP method spoofing |
| `@method('DELETE')` | `<input type="hidden" name="_method" value="DELETE">` | HTTP method spoofing |
| `@method('PATCH')` | `<input type="hidden" name="_method" value="PATCH">` | HTTP method spoofing |

**Complete General Syntax — Conditional Attributes:**

```blade
<input type="checkbox" name="active" @checked($user->active)>

<select name="version">
    <option value="1" @selected($version === 1)>Version 1</option>
    <option value="2" @selected($version === 2)>Version 2</option>
</select>

<input type="text" name="email" @disabled($user->isSuspended())>
<input type="text" name="id" @readonly($user->isVerified())>
<input type="text" name="name" @required($isRequired)>
```

**Component Breakdown:**

| Directive | Rendered When True | Purpose |
|-----------|-------------------|---------|
| `@checked($cond)` | `checked` | Checkbox/radio checked state |
| `@selected($cond)` | `selected` | Dropdown option selected state |
| `@disabled($cond)` | `disabled` | Disabled form element |
| `@readonly($cond)` | `readonly` | Read-only form element |
| `@required($cond)` | `required` | Required form element |

**Syntax Rules:**

- `@csrf` must be included in every POST, PUT, PATCH, and DELETE form.
- `@method` must be included in forms that spoof PUT, PATCH, or DELETE methods.
- The conditional attribute directives accept a boolean expression and render the attribute only when true.
- The attribute directives can be used on any HTML element that supports the attribute.
- For select options, `@selected` compares the current value with the option value.

**Constraints and Limitations:**

- **`@csrf` required for all state-changing forms:** Omitting `@csrf` results in a 419 Page Expired error.
- **`@method` only works with POST forms:** The `_method` field is only read when the form's method is POST.
- **Attribute directives are boolean:** They render the attribute name without a value (e.g., `disabled`, not `disabled="disabled"`).
- **`@checked` and `@selected` accept expressions:** The condition can be any boolean expression, not just a variable.

### Multiple Annotated Complete Code Examples

**Example 1: CSRF and Method in an Update Form**

```blade
{{-- resources/views/posts/edit.blade.php --}}

<form action="{{ route('posts.update', $post) }}" method="POST">
    @csrf
    @method('PUT')

    <label for="title">Title</label>
    <input type="text" name="title" id="title" value="{{ old('title', $post->title) }}">

    <label for="body">Body</label>
    <textarea name="body" id="body">{{ old('body', $post->body) }}</textarea>

    <button type="submit">Update Post</button>
</form>
```

**Expected Output:** The form submits a PUT request to `/posts/{id}` with the CSRF token and form data.

**Why:** The `@csrf` directive generates the hidden token field. The `@method('PUT')` directive generates the hidden `_method` field, allowing the POST form to be treated as a PUT request by Laravel's router.

---

**Example 2: Conditional Checkbox and Select**

```blade
{{-- resources/views/settings/edit.blade.php --}}

<label>
    <input type="checkbox" name="notifications" value="1" @checked($settings->notifications)>
    Enable notifications
</label>

<select name="theme">
    <option value="light" @selected($settings->theme === 'light')>Light</option>
    <option value="dark" @selected($settings->theme === 'dark')>Dark</option>
    <option value="system" @selected($settings->theme === 'system')>System</option>
</select>
```

**Expected Output:** The checkbox is checked if `$settings->notifications` is true. The dropdown option matching `$settings->theme` is selected.

**Why:** `@checked` and `@selected` render the boolean attributes conditionally, avoiding manual `@if` blocks inside HTML tags.

---

**Example 3: Conditional Disabled and Readonly**

```blade
{{-- resources/views/users/edit.blade.php --}}

<input type="email" name="email" value="{{ $user->email }}"
    @disabled($user->isSuspended())
    @readonly($user->isVerified())>

<input type="text" name="username" value="{{ $user->username }}"
    @required($isRequired)>
```

**Expected Output:** If the user is suspended, the email input is disabled. If the user is verified, the email input is readonly. If `$isRequired` is true, the username input is required.

**Why:** The conditional attribute directives render the appropriate HTML attribute only when the condition is true, providing clean, readable form control logic.

### Real-World Cases

- **All Forms:** `@csrf` is included in every form that submits data via POST, PUT, PATCH, or DELETE.
- **Resource Controllers:** `@method('PUT')` is used in edit forms for resource controllers.
- **Settings Pages:** `@checked` and `@selected` are used for preference toggles and dropdown selections.
- **User Management:** `@disabled` and `@readonly` are used to restrict editing based on user status.
- **Admin Panels:** `@required` is used for mandatory fields in administrative forms.

### References

- Laravel Blade: CSRF Field — https://laravel.com/docs/12.x/blade#csrf-field
- Laravel Blade: Method Field — https://laravel.com/docs/12.x/blade#method-field
- Laravel Blade: Additional Attributes — https://laravel.com/docs/12.x/blade#additional-attributes

---

## 3. Loops

### Definitions

**Core Definition:** Blade loop directives provide expressive shortcuts for PHP loop structures (`@for`, `@foreach`, `@forelse`, `@while`) and introduce a `$loop` variable that provides iteration metadata inside `@foreach` and `@forelse` loops.

**Technical Definition:** The `@for`, `@foreach`, and `@while` directives compile to their PHP counterparts using alternative syntax (`for(): endfor;`, `foreach(): endforeach;`, `while(): endwhile;`). The `@forelse` directive is a Blade-specific construct that combines `foreach` with an empty check, compiling to a `foreach` loop with a conditional check for emptiness. Inside `@foreach` and `@forelse` loops, a `$loop` variable is automatically available, providing properties such as `index`, `iteration`, `remaining`, `count`, `first`, `last`, `even`, `odd`, `depth`, and `parent`.

**Beginner-Friendly Explanation:** Loops let you repeat a block of HTML for each item in a collection. `@foreach` is the most common—it loops over an array or collection. `@forelse` is like `@foreach` but includes an `@empty` block that runs when the collection is empty, so you don't need a separate `@if` check. The `$loop` variable is a special helper that tells you things like "is this the first item?" or "what's the current index?"—useful for adding CSS classes to alternating rows or separators between items.

### Purposes

- To iterate over arrays and collections with `@foreach`.
- To provide an `@empty` fallback when a collection is empty with `@forelse`.
- To execute a loop a fixed number of times with `@for`.
- To loop while a condition is true with `@while`.
- To access iteration metadata (index, first, last, depth) via the `$loop` variable.
- To skip the current iteration with `@continue` or break out of the loop with `@break`.
- To access the parent loop's `$loop` variable in nested loops.

### Syntax Rules and Structure

**Complete General Syntax — `@foreach`:**

```blade
@foreach($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@foreach($items as $item)` | `<?php foreach($items as $item): ?>` | Loop over collection |
| `@endforeach` | `<?php endforeach; ?>` | End of loop |

**Complete General Syntax — `@forelse`:**

```blade
@forelse($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users found.</p>
@endforelse
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@forelse($items as $item)` | Loop with empty fallback |
| `@empty` | Executed when collection is empty |
| `@endforelse` | End of forelse block |

**Complete General Syntax — `@for`:**

```blade
@for($i = 0; $i < 10; $i++)
    <p>The current value is {{ $i }}</p>
@endfor
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@for($i = 0; $i < 10; $i++)` | `<?php for($i = 0; $i < 10; $i++): ?>` | Fixed iteration loop |
| `@endfor` | `<?php endfor; ?>` | End of loop |

**Complete General Syntax — `@while`:**

```blade
@while($condition)
    <p>I'm looping forever.</p>
@endwhile
```

**Component Breakdown:**

| Directive | Compiled To | Purpose |
|-----------|-------------|---------|
| `@while($cond)` | `<?php while($cond): ?>` | Conditional loop |
| `@endwhile` | `<?php endwhile; ?>` | End of loop |

**Complete General Syntax — `$loop` Variable Properties:**

```blade
@foreach($users as $user)
    @if($loop->first)
        <p>This is the first iteration.</p>
    @endif

    @if($loop->last)
        <p>This is the last iteration.</p>
    @endif

    <p>Index: {{ $loop->index }}, Iteration: {{ $loop->iteration }}</p>
    <p>Remaining: {{ $loop->remaining }}, Total: {{ $loop->count }}</p>
    <p>Depth: {{ $loop->depth }}</p>
@endforeach
```

**Complete General Syntax — Nested Loops with `$loop->parent`:**

```blade
@foreach($users as $user)
    @foreach($user->posts as $post)
        @if($loop->parent->first)
            <p>This is the first iteration of the parent loop.</p>
        @endif
        <p>{{ $post->title }}</p>
    @endforeach
@endforeach
```

**Complete General Syntax — `@break` and `@continue`:**

```blade
@foreach($users as $user)
    @continue($user->type == 1)
    <li>{{ $user->name }}</li>
    @break($user->number == 5)
@endforeach
```

**Syntax Rules:**

- `@foreach` and `@forelse` automatically provide the `$loop` variable.
- `@for` and `@while` do **not** provide the `$loop` variable.
- `@forelse` requires an `@empty` block—it is not optional.
- `@break` and `@continue` can include a condition: `@continue($condition)`.
- The `$loop->parent` property provides access to the parent loop's `$loop` variable in nested loops.
- `@for` and `@while` do not have an `@empty` equivalent.

**Constraints and Limitations:**

- **`$loop` only in `@foreach`/`@forelse`:** The `$loop` variable is not available in `@for` or `@while` loops.
- **`@forelse` requires `@empty`:** The `@empty` block is mandatory in `@forelse`.
- **`@break` and `@continue`:** These directives cannot be used outside of loops.
- **Nested loop depth:** The `$loop->depth` property indicates the nesting level (1 for the outermost loop).

### Multiple Annotated Complete Code Examples

**Example 1: Basic `@foreach` with `$loop` Variable**

```blade
{{-- resources/views/users/index.blade.php --}}

@foreach($users as $user)
    <div class="user-row {{ $loop->even ? 'bg-gray-100' : 'bg-white' }}">
        <span>{{ $loop->iteration }}.</span>
        <span>{{ $user->name }}</span>
        @if($loop->last)
            <span class="badge">Last user</span>
        @endif
    </div>
@endforeach
```

**Expected Output:** Each user is displayed with their iteration number. Even rows have a gray background; the last row has a "Last user" badge.

**Why:** The `$loop->iteration` property provides the 1-based iteration number. `$loop->even` alternates row colors. `$loop->last` identifies the final iteration.

---

**Example 2: `@forelse` with Empty Fallback**

```blade
{{-- resources/views/posts/index.blade.php --}}

@forelse($posts as $post)
    <article>
        <h2>{{ $post->title }}</h2>
        <p>{{ $post->excerpt }}</p>
    </article>
@empty
    <div class="alert alert-info">
        No posts have been published yet.
    </div>
@endforelse
```

**Expected Output:** If there are posts, each post is displayed. If the collection is empty, the "No posts" message is displayed.

**Why:** `@forelse` combines the loop and the empty check into a single construct, eliminating the need for a separate `@if($posts->isEmpty())` block.

---

**Example 3: Nested Loops with `$loop->parent`**

```blade
{{-- resources/views/categories/index.blade.php --}}

@foreach($categories as $category)
    <h3>{{ $category->name }}</h3>
    <ul>
        @foreach($category->products as $product)
            <li>
                {{ $product->name }}
                @if($loop->parent->first)
                    <span class="text-muted">(first category)</span>
                @endif
            </li>
        @endforeach
    </ul>
@endforeach
```

**Expected Output:** Each category is displayed with its products. Products in the first category have a "(first category)" label.

**Why:** The `$loop->parent` property provides access to the outer loop's `$loop` variable, allowing inner loop logic to reference the parent iteration state.

---

**Example 4: `@break` and `@continue`**

```blade
{{-- resources/views/tasks/index.blade.php --}}

@foreach($tasks as $task)
    @continue($task->isCompleted())
    <li>{{ $task->title }}</li>
    @break($loop->index >= 10)
@endforeach
```

**Expected Output:** Only incomplete tasks are displayed, up to a maximum of 10 tasks.

**Why:** `@continue($condition)` skips the current iteration if the condition is true. `@break($condition)` exits the loop entirely when the condition is true.

### Real-World Cases

- **User Lists:** `@foreach` iterates over user collections to build user tables.
- **Empty States:** `@forelse` provides "no results" messages when collections are empty.
- **Alternating Rows:** `$loop->even` and `$loop->odd` are used for zebra-striping table rows.
- **First/Last Items:** `$loop->first` and `$loop->last` are used for separators and border styling.
- **Nested Comments:** `$loop->parent` is used when rendering nested comment threads.

### References

- Laravel Blade: Loops — https://laravel.com/docs/12.x/blade#loops
- Laravel Blade: The Loop Variable — https://laravel.com/docs/12.x/blade#the-loop-variable
- Laravel Blade: Additional Attributes — https://laravel.com/docs/12.x/blade#additional-attributes

---

## 4. Environment & Auth Directives

### Definitions

**Core Definition:** Environment and authentication directives are Blade shortcuts for conditionally rendering content based on the application's environment (`@env`, `@production`) or the current user's authentication and authorization state (`@auth`, `@guest`, `@can`, `@cannot`).

**Technical Definition:** The `@env` directive checks the application's `APP_ENV` value against a given environment name (or array of names) and renders its contents if there is a match. The `@production` directive is a shortcut for `@env('production')`. The `@auth` and `@guest` directives check the authentication status of the current user via Laravel's guard system, optionally accepting a guard name as a parameter. The `@can` and `@cannot` directives integrate with Laravel's Gate and Policy system, checking whether the current user is authorized to perform a given action on a given model.

**Beginner-Friendly Explanation:** These directives help you show or hide parts of your page based on *where* your application is running (development, staging, production) or *who* is viewing it (logged in, guest, authorized to do something). For example, you might show Google Analytics code only in production (`@production`), show an admin menu only to admins (`@can`), or show login links only to guests (`@guest`).

### Purposes

- To conditionally render content based on the application environment via `@env`.
- To render content exclusively in production via `@production`.
- To display content only to authenticated users via `@auth`.
- To display content only to guests (unauthenticated users) via `@guest`.
- To display content only when the user is authorized to perform an action via `@can`.
- To display content only when the user is not authorized via `@cannot`.
- To check multiple abilities at once via `@canany`.

### Syntax Rules and Structure

**Complete General Syntax — Environment Directives:**

```blade
@production
    <script>/* Google Analytics */</script>
@endproduction

@env('staging')
    <p>Staging environment</p>
@endenv

@env(['staging', 'production'])
    <p>Staging or production</p>
@endenv
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@production` | Content visible only in production |
| `@endproduction` | End of production block |
| `@env('staging')` | Content visible in staging |
| `@env(['staging', 'production'])` | Content visible in staging or production |
| `@endenv` | End of env block |

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

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@auth` | Content visible to authenticated users |
| `@endauth` | End of auth block |
| `@guest` | Content visible to unauthenticated users |
| `@endguest` | End of guest block |
| `@auth('admin')` | Content visible to users authenticated via the `admin` guard |

**Complete General Syntax — Authorization Directives:**

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@cannot('delete', $post)
    <p>You cannot delete this post.</p>
@endcannot

@canany(['update', 'delete'], $post)
    <p>You have some permissions on this post.</p>
@endcanany
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@can('ability', $model)` | Content visible if user is authorized |
| `@endcan` | End of can block |
| `@cannot('ability', $model)` | Content visible if user is not authorized |
| `@endcannot` | End of cannot block |
| `@canany(['ability1', 'ability2'], $model)` | Content visible if user has any of the abilities |

**Syntax Rules:**

- `@env` accepts a string or an array of environment names.
- `@production` is equivalent to `@env('production')`.
- `@auth` and `@guest` optionally accept a guard name as a parameter.
- `@can` and `@cannot` require an ability name and optionally a model instance.
- `@canany` accepts an array of abilities and optionally a model.
- These directives integrate with Laravel's Gate and Policy system for authorization.

**Constraints and Limitations:**

- **`@env` is environment-specific:** The environment name must match the `APP_ENV` value in the `.env` file.
- **`@can` requires gates/policies:** The ability must be defined via `Gate::define()` or a policy class.
- **Guard-specific auth:** `@auth('admin')` only works if the `admin` guard is configured in `config/auth.php`.
- **Authorization caching:** `@can` checks are not cached—they execute on every render.

### Multiple Annotated Complete Code Examples

**Example 1: Environment-Specific Analytics**

```blade
{{-- resources/views/layouts/app.blade.php --}}

@production
    <!-- Google Analytics -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
    <script>
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', 'GA_MEASUREMENT_ID');
    </script>
@endproduction

@env('local')
    <div class="alert alert-warning">
        Local development environment
    </div>
@endenv
```

**Expected Output:** In production, the Google Analytics script is included. In the local environment, a warning banner is displayed.

**Why:** The `@production` directive ensures analytics are only loaded in production. The `@env('local')` directive shows a development notice only in the local environment.

---

**Example 2: Authentication-Based Navigation**

```blade
{{-- resources/views/layouts/navigation.blade.php --}}

@auth
    <nav>
        <a href="{{ route('dashboard') }}">Dashboard</a>
        <a href="{{ route('profile.show') }}">{{ auth()->user()->name }}</a>
        <form method="POST" action="{{ route('logout') }}">
            @csrf
            <button type="submit">Logout</button>
        </form>
    </nav>
@endauth

@guest
    <nav>
        <a href="{{ route('login') }}">Login</a>
        <a href="{{ route('register') }}">Register</a>
    </nav>
@endguest
```

**Expected Output:** Authenticated users see the dashboard, profile, and logout links. Guests see login and register links.

**Why:** `@auth` and `@guest` conditionally render navigation based on the user's authentication status.

---

**Example 3: Authorization with `@can` and `@cannot`**

```blade
{{-- resources/views/posts/show.blade.php --}}

@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}" class="btn btn-primary">Edit Post</a>
@endcan

@cannot('delete', $post)
    <p class="text-muted">You do not have permission to delete this post.</p>
@endcannot

@canany(['update', 'delete'], $post)
    <p>You have some management permissions on this post.</p>
@endcanany
```

**Expected Output:** If the user can update the post, the "Edit Post" link is displayed. If the user cannot delete the post, the permission message is displayed. If the user can either update or delete, the management message is displayed.

**Why:** `@can`, `@cannot`, and `@canany` check the user's authorization against the post's policy, rendering content only when the user has (or does not have) the required permission.

### Real-World Cases

- **Analytics:** Load Google Analytics or Tag Manager only in production via `@production`.
- **Debug Toolbars:** Show debug information only in local/staging environments via `@env`.
- **Admin Menus:** Display admin navigation only to users with admin privileges via `@can`.
- **Login/Register Links:** Show authentication links only to guests via `@guest`.
- **Edit/Delete Buttons:** Display edit and delete actions only when the user is authorized via `@can`.
- **Feature Flags:** Show or hide features based on environment or user permissions.

### References

- Laravel Blade: Environment Directives — https://laravel.com/docs/12.x/blade#environment-directives
- Laravel Blade: Authentication Directives — https://laravel.com/docs/12.x/blade#authentication-directives
- Laravel Blade: Authorization Directives — https://laravel.com/docs/12.x/blade#authorization-directives
- Laravel Authorization Documentation — https://laravel.com/docs/12.x/authorization

---

## 5. Session Directives

### Definitions

**Core Definition:** Session directives in Blade (`@session` and `@error`) provide shortcuts for checking the existence of session values and validation error messages, rendering content conditionally based on session state.

**Technical Definition:** The `@session` directive (introduced in Laravel 10.38) checks whether a given session key exists and has a value, rendering its contents if so. Within the `@session` block, the `$value` variable contains the session value. The `@error` directive checks whether a validation error message exists for a given attribute in the `$errors` MessageBag, rendering its contents if so. Within the `@error` block, the `$message` variable contains the error message. The `$errors` variable is automatically shared with all views by the `ShareErrorsFromSession` middleware.

**Beginner-Friendly Explanation:** After a form submission fails validation, Laravel redirects the user back to the form with error messages stored in the session. The `@error` directive lets you check if a specific field has an error and display the error message next to that field. The `@session` directive is a newer, cleaner way to check if a session value exists—useful for flash messages like "Profile updated successfully!" that appear once after a redirect.

### Purposes

- To check if a session value exists and render content conditionally via `@session`.
- To display flash messages (success, error, status) stored in the session.
- To check for validation error messages on specific attributes via `@error`.
- To display validation error messages next to the corresponding form fields.
- To support named error bags when multiple forms exist on the same page.
- To provide a cleaner alternative to `@if(session('key'))` blocks.

### Syntax Rules and Structure

**Complete General Syntax — `@session`:**

```blade
@session('status')
    <div class="alert alert-success">
        {{ $value }}
    </div>
@endsession
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@session('status')` | Checks if the `status` session key exists |
| `$value` | The session value |
| `@endsession` | End of session block |

**Complete General Syntax — `@error`:**

```blade
@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

**Component Breakdown:**

| Directive | Purpose |
|-----------|---------|
| `@error('title')` | Checks if a validation error exists for `title` |
| `$message` | The validation error message |
| `@enderror` | End of error block |

**Complete General Syntax — Named Error Bags:**

```blade
@error('title', 'post')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

**Component Breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `'title'` | The field name |
| `'post'` | The error bag name |

**Complete General Syntax — Displaying All Errors:**

```blade
@if($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

**Syntax Rules:**

- `@session` checks if the session key exists and has a non-null value.
- Within `@session`, the `$value` variable contains the session value.
- `@error` checks the `$errors` MessageBag for the specified attribute.
- Within `@error`, the `$message` variable contains the error message.
- Named error bags are passed as the second argument to `@error`.
- The `$errors` variable is always available in views (shared by `ShareErrorsFromSession` middleware).

**Constraints and Limitations:**

- **`@session` requires session middleware:** The directive only works on routes that have session middleware (typically the `web` group).
- **`@error` requires validation:** The directive only renders when validation errors exist for the specified attribute.
- **Named error bags:** When using multiple forms on the same page, each form should use a named error bag to avoid conflicts.
- **`$errors` always defined:** The `$errors` variable is always defined in views, so you can safely call `$errors->any()` without checking if it exists.

### Multiple Annotated Complete Code Examples

**Example 1: Flash Message with `@session`**

```blade
{{-- resources/views/layouts/app.blade.php --}}

@session('status')
    <div class="alert alert-success" role="alert">
        {{ $value }}
    </div>
@endsession

@session('error')
    <div class="alert alert-danger" role="alert">
        {{ $value }}
    </div>
@endsession
```

```php
// Controller
return redirect()->route('profile.show')
    ->with('status', 'Profile updated successfully!');
```

**Expected Output:** After a redirect with a `status` flash message, the success alert is displayed. The message appears once and is then removed from the session.

**Why:** The `@session('status')` directive checks if the `status` key exists in the session. The `$value` variable contains the message. This is cleaner than `@if(session('status'))`.

---

**Example 2: Validation Error with `@error`**

```blade
{{-- resources/views/posts/create.blade.php --}}

<form method="POST" action="{{ route('posts.store') }}">
    @csrf

    <label for="title">Title</label>
    <input type="text" name="title" id="title" value="{{ old('title') }}"
        class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror

    <label for="body">Body</label>
    <textarea name="body" id="body" class="@error('body') is-invalid @enderror">{{ old('body') }}</textarea>

    @error('body')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror

    <button type="submit">Create Post</button>
</form>
```

**Expected Output:** If validation fails for the `title` field, the input receives the `is-invalid` class and the error message is displayed below it. The same applies to the `body` field.

**Why:** The `@error` directive conditionally adds the `is-invalid` CSS class and displays the error message. The `$message` variable contains the specific error message for that field.

---

**Example 3: Named Error Bag**

```blade
{{-- resources/views/auth/login.blade.php --}}

<form method="POST" action="{{ route('login') }}">
    @csrf

    <label for="email">Email</label>
    <input type="email" name="email" id="email" value="{{ old('email') }}"
        class="@error('email', 'login') is-invalid @enderror">

    @error('email', 'login')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror

    <button type="submit">Login</button>
</form>

{{-- A separate registration form on the same page --}}
<form method="POST" action="{{ route('register') }}">
    @csrf

    <label for="email">Email</label>
    <input type="email" name="email" id="email" value="{{ old('email') }}"
        class="@error('email', 'register') is-invalid @enderror">

    @error('email', 'register')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror

    <button type="submit">Register</button>
</form>
```

**Expected Output:** Errors from the login form are displayed only under the login form, and errors from the registration form are displayed only under the registration form, even though both use the `email` field name.

**Why:** The named error bag (`'login'` and `'register'`) isolates the error messages for each form, preventing conflicts when multiple forms share the same field names.

### Real-World Cases

- **Flash Messages:** Display success, error, or status messages after redirects using `@session`.
- **Form Validation:** Display validation errors next to the corresponding form fields using `@error`.
- **Multi-Form Pages:** Use named error bags when multiple forms exist on the same page.
- **Login/Register:** Show validation errors for authentication forms.
- **Profile Updates:** Display validation errors for profile edit forms.

### References

- Laravel Blade: Session Directive — https://laravel.com/docs/12.x/blade#session-directive
- Laravel Blade: Validation Errors — https://laravel.com/docs/12.x/blade#validation-errors
- Laravel News: Laravel 10.38 Released with a New Blade @session Directive — https://laravel-news.com/laravel-10-38

---

## References

- Laravel Blade Templates Documentation (12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade: If Statements — https://laravel.com/docs/12.x/blade#if-statements
- Laravel Blade: CSRF Field — https://laravel.com/docs/12.x/blade#csrf-field
- Laravel Blade: Method Field — https://laravel.com/docs/12.x/blade#method-field
- Laravel Blade: Additional Attributes — https://laravel.com/docs/12.x/blade#additional-attributes
- Laravel Blade: Loops — https://laravel.com/docs/12.x/blade#loops
- Laravel Blade: The Loop Variable — https://laravel.com/docs/12.x/blade#the-loop-variable
- Laravel Blade: Environment Directives — https://laravel.com/docs/12.x/blade#environment-directives
- Laravel Blade: Authentication Directives — https://laravel.com/docs/12.x/blade#authentication-directives
- Laravel Blade: Authorization Directives — https://laravel.com/docs/12.x/blade#authorization-directives
- Laravel Blade: Session Directive — https://laravel.com/docs/12.x/blade#session-directive
- Laravel Blade: Validation Errors — https://laravel.com/docs/12.x/blade#validation-errors
- Laravel News: Laravel 10.38 Released with a New Blade @session Directive — https://laravel-news.com/laravel-10-38
- Laravel Authorization Documentation — https://laravel.com/docs/12.x/authorization
- Laravel Validation Documentation — https://laravel.com/docs/12.x/validation