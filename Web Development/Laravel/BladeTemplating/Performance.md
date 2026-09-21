# Laravel Performance & Modern Rendering (New): A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Performance & Modern Rendering encompasses the framework's mechanisms for optimising view rendering in production (view caching), enabling partial page updates for modern frontend workflows (Blade fragments), and dynamically rendering Blade templates from strings (inline Blade rendering).

**Technical Definition:** View caching precompiles all Blade templates into plain PHP files via `php artisan view:cache`, eliminating on-demand compilation overhead. Blade fragments (`@fragment` / `@endfragment`) allow controllers to return only a specific section of a rendered view, reducing payload size for HTMX and Turbo-driven partial updates. Inline Blade rendering (`Blade::render()`) compiles a Blade template string into HTML at runtime, writing temporary compiled files to `storage/framework/views` and optionally deleting them after rendering.

**Beginner-Friendly Explanation:** When your application serves many users, every millisecond counts. View caching pre-builds all your page templates so they don't have to be compiled on every request. Blade fragments let you send back just a small piece of a page when only part of it needs to change—like refreshing a notification list without reloading the whole dashboard. Inline Blade rendering lets you turn a string of Blade syntax into HTML on the fly, which is useful for generating emails, reports, or dynamic content that doesn't live in a `.blade.php` file.

### Key Characteristics

- **Precompilation:** `view:cache` compiles all Blade templates during deployment, not on the first request.
- **Partial Rendering:** `@fragment` directives mark sections of a view that can be returned independently.
- **Runtime String Compilation:** `Blade::render()` compiles Blade syntax from a string without requiring a dedicated view file.
- **Framework Integration:** Blade fragments are designed for HTMX and Turbo workflows where the server returns HTML fragments instead of JSON.
- **Cache Management:** `view:clear` removes compiled views, and `view:cache` regenerates them.
- **Temporary File Cleanup:** `Blade::render()` supports `deleteCachedView: true` to remove temporary compiled files after rendering.

### Prerequisites

- **Laravel 9.x+** for Blade fragments and `Blade::render()` (fragments introduced in Laravel 9.39).
- **Laravel 10.x or 12.x** recommended for the latest performance improvements.
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with Blade templates in `resources/views`.
- For HTMX/Turbo workflows: basic familiarity with those frameworks' request/response model.

### Related Programming Areas

- **Deployment Pipelines:** View caching is a standard step in production deployment scripts.
- **Frontend Frameworks:** Blade fragments integrate with HTMX, Turbo, and Alpine.js.
- **Email Generation:** `Blade::render()` is commonly used to generate HTML emails from database-stored templates.
- **Caching Strategy:** View caching complements config caching, route caching, and event caching.
- **Blade Compiler:** All three features operate on the Blade compilation layer.

### Core Concepts / Features

1. Blade Fragments (`@fragment`)
2. View Caching (`view:cache`, `view:clear`)
3. In-line Blade Views (`Blade::render`)

---

## 1. Blade Fragments

### Definitions

**Core Definition:** Blade fragments allow a controller to return only a specific section of a rendered Blade view, identified by a `@fragment` directive, enabling partial page updates without full page reloads.

**Technical Definition:** The `@fragment('name')` and `@endfragment` directives mark a section of a Blade template. When the view is rendered, the `View` instance's `fragment('name')` method can be invoked to return only that section's HTML. The method `fragmentIf($condition, 'name')` conditionally returns the fragment or the full view. Fragments are compiled into the same PHP file as the parent view, and the fragment content is extracted at render time via output buffering. This is ideal for HTMX (using the `HX-Request` header) and Turbo workflows where the server responds with HTML fragments instead of JSON.

**Beginner-Friendly Explanation:** Imagine a dashboard with a notification panel. When a new notification arrives, you don't want to reload the entire dashboard—you just want to update the notification list. With Blade fragments, you mark the notification list with `@fragment('notification-list')`, and when the AJAX request comes in, the controller returns only that fragment. The frontend swaps out the old list for the new one. It's like replacing one puzzle piece instead of the whole puzzle.

### Purposes

- To return specific portions of a Blade template in HTTP responses for partial page updates.
- To reduce payload size when only a section of a page needs to change.
- To integrate with HTMX and Turbo frameworks that expect HTML over the wire.
- To avoid creating separate view partials for every dynamically updatable section.
- To conditionally return a fragment based on request headers (e.g., `HX-Request`).
- To support real-time features (notifications, live feeds) with minimal server-side overhead.

### Syntax Rules and Structure

**Complete General Syntax — Defining a Fragment:**

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@fragment('user-list')` | Marks the beginning of a named fragment. |
| `@endfragment` | Marks the end of the fragment. |
| `'user-list'` | The fragment identifier used when rendering. |

**Complete General Syntax — Returning a Fragment from a Controller:**

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `->fragment('user-list')` | Returns only the `user-list` fragment. |
| `->fragmentIf($condition, 'user-list')` | Returns the fragment only if `$condition` is true; otherwise returns the full view. |

**Complete General Syntax — Conditional Fragment Rendering:**

```php
return view('notifications.index', [
    'notifications' => $notifications
])->fragmentIf(
    $request->hasHeader('HX-Request'),
    'notification-list'
);
```

**Syntax Rules:**

- Fragment names must be unique within a single view file.
- The `@fragment` directive can be placed anywhere within a Blade template.
- Fragments are rendered by calling `->fragment('name')` on the `View` instance.
- The `fragmentIf()` method accepts a boolean condition, a fragment name, and optionally a fallback view.
- Fragments work with both `@extends` layouts and component-based layouts.
- The fragment's content is extracted via output buffering during the full view render.

**Constraints and Limitations:**

- **Full view still renders:** When a fragment is requested, Laravel still renders the entire view and extracts the fragment—it does not skip rendering the rest of the template.
- **Fragment nesting:** Fragments cannot be nested within other fragments.
- **Layout interaction:** If the fragment is inside a section that is normally yielded in a layout, the fragment extraction still works because the full view is rendered first.
- **Performance:** For very large views, rendering the full view to extract a small fragment may be inefficient—consider using dedicated partials for large views.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Fragment with HTMX**

```blade
{{-- resources/views/notifications/index.blade.php --}}
<div class="container">
    <h1>Notifications</h1>

    @fragment('notification-list')
        <div class="notification-wrapper">
            @forelse($notifications as $notification)
                <div class="alert alert-{{ $notification->type }}">
                    {{ $notification->message }}
                    <span class="timestamp">{{ $notification->created_at->diffForHumans() }}</span>
                </div>
            @empty
                <p>No notifications</p>
            @endforelse
        </div>
    @endfragment
</div>
```

```php
<?php
// app/Http/Controllers/NotificationController.php

namespace App\Http\Controllers;

use App\Models\Notification;
use Illuminate\Http\Request;

class NotificationController extends Controller
{
    public function store(Request $request)
    {
        $notification = Notification::create([
            'user_id' => auth()->id(),
            'message' => $request->message,
            'type' => $request->type,
        ]);

        if ($request->hasHeader('HX-Request')) {
            return view('notifications.index', [
                'notifications' => auth()->user()->notifications()->latest()->get()
            ])->fragmentIf(
                $request->hasHeader('HX-Request'),
                'notification-list'
            );
        }

        return back();
    }
}
```

**Expected Output:** When HTMX sends a request with the `HX-Request` header, the response contains only the `notification-list` fragment HTML, not the full page. The frontend swaps the fragment into the page.

**Why:** The `fragmentIf()` method checks for the `HX-Request` header and returns only the marked fragment. This reduces the payload size and allows HTMX to update just the notification list.

---

**Example 2: Fragment for Search Results**

```blade
{{-- resources/views/products/index.blade.php --}}
<div class="products-page">
    <h1>Products</h1>

    <form hx-get="/products/search" hx-target="#product-list" hx-swap="innerHTML">
        <input type="text" name="q" placeholder="Search products...">
    </form>

    @fragment('product-list')
        <div id="product-list">
            @foreach($products as $product)
                <div class="product">{{ $product->name }}</div>
            @endforeach
        </div>
    @endfragment
</div>
```

```php
<?php
// app/Http/Controllers/ProductController.php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function search(Request $request)
    {
        $products = Product::where('name', 'like', "%{$request->q}%")->get();

        return view('products.index', [
            'products' => $products
        ])->fragment('product-list');
    }
}
```

**Expected Output:** When HTMX submits the search form, the controller returns only the `product-list` fragment with the filtered products. HTMX swaps the content of `#product-list` with the new HTML.

**Why:** The fragment isolates the product list, allowing the search results to be updated without reloading the entire page. The `hx-target` and `hx-swap` attributes control how HTMX inserts the fragment.

---

**Example 3: Fragment with Alpine.js**

```blade
{{-- resources/views/cart/index.blade.php --}}
<div x-data="{ count: {{ $cartCount }} }">
    <h1>Shopping Cart</h1>

    @fragment('cart-items')
        <div id="cart-items">
            @foreach($items as $item)
                <div class="cart-item">
                    {{ $item->name }} × {{ $item->quantity }}
                </div>
            @endforeach
        </div>
    @endfragment

    <button @click="
        fetch('/cart/add', { method: 'POST' })
            .then(r => r.text())
            .then(html => document.getElementById('cart-items').innerHTML = html)
    ">Add Item</button>
</div>
```

```php
<?php
// app/Http/Controllers/CartController.php

namespace App\Http\Controllers;

use App\Models\CartItem;

class CartController extends Controller
{
    public function add()
    {
        // Add item to cart...
        $items = CartItem::where('user_id', auth()->id())->get();

        return view('cart.index', ['items' => $items])->fragment('cart-items');
    }
}
```

**Expected Output:** Clicking "Add Item" fetches the `cart-items` fragment from the server and replaces the `#cart-items` element's inner HTML with the updated cart items.

**Why:** The fragment allows the cart items to be updated via a simple `fetch` call without a full page reload, demonstrating the flexibility of fragments beyond HTMX and Turbo.

### Real-World Cases

- **Live Notifications:** HTMX or Turbo requests return only the notification list fragment for real-time updates.
- **Search Results:** Dynamic search forms return only the results fragment.
- **Shopping Carts:** Cart item lists are updated without reloading the page.
- **Comment Feeds:** New comments are appended via fragment responses.
- **Dashboard Widgets:** Individual widgets are refreshed independently using fragments.

### References

- Dynamic Page Updates with Laravel Blade Fragments (Laravel News) — https://laravel-news.com/dynamic-page-updates-with-laravel-blade-fragments
- Laravel Blade: Rendering Blade Fragments — https://laravel.com/docs/12.x/blade#rendering-blade-fragments
- Laravel 9.39 Released (Blade Fragments) — https://laravel-news.com/laravel-9-39-released
- Blade Fragments for HTMX/Turbo (GitCode Blog) — https://blog.gitcode.com/37ae997e076dd4cb03133c63159b0d29.html
- Laravel HTMX Package — https://mail.packalyst.com

---

## 2. View Caching

### Definitions

**Core Definition:** View caching is the process of precompiling all Blade templates into plain PHP files during deployment, so that views are not compiled on-demand for every request.

**Technical Definition:** Laravel's Blade compiler normally compiles a `.blade.php` template into a PHP file in `storage/framework/views` when the view is first rendered. The `php artisan view:cache` command triggers this compilation for all views ahead of time, so the first request after deployment does not incur compilation overhead. The `php artisan view:clear` command removes all compiled view files, forcing recompilation on the next request. View caching is a standard step in production deployment pipelines alongside `config:cache` and `route:cache`.

**Beginner-Friendly Explanation:** When a user visits a page that uses a Blade template, Laravel normally has to "compile" that template into PHP first. This takes a small amount of time. If you precompile all your templates during deployment—before any users visit—then every visitor gets the already-compiled version instantly. It's like preparing all the ingredients before the restaurant opens instead of chopping vegetables for every customer.

### Purposes

- To eliminate on-demand Blade compilation overhead for every request.
- To improve response times for all views in production.
- To ensure consistent performance across the first and subsequent requests after deployment.
- To complement other production caching strategies (config, routes, events).
- To reduce disk I/O and CPU usage associated with template compilation.
- To provide a deterministic deployment process where all views are known to be compiled.

### Syntax Rules and Structure

**Complete General Syntax — Caching Views:**

```bash
php artisan view:cache
```

**Component Breakdown:**

| Command | Description |
|---------|-------------|
| `php artisan view:cache` | Compiles all Blade templates to `storage/framework/views`. |

**Complete General Syntax — Clearing View Cache:**

```bash
php artisan view:clear
```

**Component Breakdown:**

| Command | Description |
|---------|-------------|
| `php artisan view:clear` | Removes all compiled view files. |

**Complete General Syntax — Optimizing All Caches:**

```bash
php artisan optimize
```

**Component Breakdown:**

| Command | Description |
|---------|-------------|
| `php artisan optimize` | Runs `config:cache`, `route:cache`, `view:cache`, and `event:cache`. |

**Complete General Syntax — Clearing All Caches:**

```bash
php artisan optimize:clear
```

**Component Breakdown:**

| Command | Description |
|---------|-------------|
| `php artisan optimize:clear` | Clears all cached files, including views, config, routes, and events. |

**Syntax Rules:**

- `view:cache` should be run during deployment, after `composer install` and before the application receives traffic.
- `view:clear` must be run after modifying Blade templates in development if changes are not reflected.
- The `optimize` command runs all caching commands in sequence.
- View caching is environment-agnostic—it works in any environment but is primarily beneficial in production.

**Constraints and Limitations:**

- **Development workflow:** Running `view:cache` in development can mask template changes—use `view:clear` after modifying templates.
- **Deployment order:** `view:cache` must be run after all Blade templates are deployed and `composer install` is complete.
- **Storage permissions:** The `storage/framework/views` directory must be writable by the web server.
- **Cache invalidation:** If templates change without running `view:clear` or `view:cache` again, stale compiled views may be served.
- **`Blade::render()` temporary files:** Inline Blade rendering creates temporary compiled files that may accumulate if not cleaned up.

### Multiple Annotated Complete Code Examples

**Example 1: Production Deployment Script**

```bash
#!/bin/bash
# deploy.sh — Production deployment script

# Pull latest code
git pull origin main

# Install dependencies without dev packages
composer install --optimize-autoloader --no-dev

# Run database migrations
php artisan migrate --force

# Cache all framework resources
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

# Restart queue workers
php artisan queue:restart
```

**Expected Output:** After running this script, all views, routes, configs, and events are cached. The first user request after deployment does not trigger any compilation.

**Why:** The script follows Laravel's recommended production deployment process. `view:cache` precompiles all Blade templates, `config:cache` caches configuration, and `route:cache` caches routes. The `--no-dev` flag excludes development dependencies.

---

**Example 2: Clearing View Cache During Development**

```bash
# After modifying a Blade template, clear the view cache
php artisan view:clear

# The next request will recompile the template
```

**Expected Output:** `Compiled views cleared!` — the next page load recompiles all templates.

**Why:** During development, Blade templates are compiled on first render. If a compiled version exists, changes to the source template may not be reflected. Clearing the view cache forces recompilation.

---

**Example 3: Using `optimize` for Full Caching**

```bash
# Run all optimization commands at once
php artisan optimize
```

**Expected Output:** `Configuration cached successfully!`, `Routes cached successfully!`, `Views cached successfully!`, `Events cached successfully!`

**Why:** The `optimize` command runs all caching commands in a single step, simplifying deployment scripts. It is equivalent to running `config:cache`, `route:cache`, `view:cache`, and `event:cache` separately.

### Real-World Cases

- **Production Deployments:** `view:cache` is run on every deployment to ensure fast first-request performance.
- **Load-Balanced Environments:** All servers in a load-balanced cluster run `view:cache` to ensure consistent performance.
- **CI/CD Pipelines:** View caching is part of the build step in continuous deployment pipelines.
- **Staging Environments:** View caching is used in staging to mirror production performance characteristics.
- **Performance Audits:** `view:cache` is one of the first optimisations applied when auditing Laravel application performance.

### References

- Laravel Views: Optimizing Views — https://laravel.com/docs/12.x/views#optimizing-views
- Laravel Deployment: Optimization — https://laravel.com/docs/12.x/deployment#optimization
- How does `php artisan view:cache` work? (Laracasts) — https://laracasts.com/discuss/channels/laravel/how-does-php-artisan-viewcache-work
- Laravel Production Performance Checklist — https://github.com/iliaal/whetstone
- Customizing Laravel Optimization with `--except` (Laravel News) — https://laravel-news.com/customizing-laravel-optimization-with-except

---

## 3. In-line Blade Views

### Definitions

**Core Definition:** In-line Blade rendering is the process of compiling a Blade template string (not a file) into HTML at runtime using the `Blade::render()` method.

**Technical Definition:** The `Blade::render($string, $data, $deleteCachedView)` method compiles a Blade template string into PHP, renders it with the provided data, and returns the resulting HTML. Laravel writes the compiled template to a temporary file in `storage/framework/views` with a hashed filename. If `deleteCachedView` is `true`, the temporary file is deleted after rendering. This method is useful for generating HTML emails from database-stored templates, rendering dynamic content, or any scenario where the template is not a static file.

**Beginner-Friendly Explanation:** Normally, Blade templates live in files. But sometimes your template is stored in a database—like an email template that admins can edit. `Blade::render()` lets you take that string, compile it as if it were a Blade file, and get HTML back. It's like having a portable Blade compiler that works on any string, anywhere.

### Purposes

- To transform a raw Blade template string into valid HTML.
- To render database-stored email templates with dynamic data.
- To generate HTML content dynamically from user-defined templates.
- To render Blade syntax without creating a dedicated view file.
- To support CMS systems where templates are stored in the database.
- To provide a lightweight way to render small snippets of Blade in CLI commands or queue jobs.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `$string` | The Blade template string. |
| `$data` | Associative array of data to pass to the template. |
| `$deleteCachedView` | Optional boolean; if `true`, deletes the temporary compiled file after rendering. |

**Complete General Syntax — With Cache Deletion:**

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

**Complete General Syntax — Rendering Components in Inline Views:**

```php
return Blade::render('<x-alert type="error">{{ $message }}</x-alert>', [
    'message' => 'Something went wrong!'
]);
```

**Syntax Rules:**

- `Blade::render()` accepts a string, an optional data array, and an optional `deleteCachedView` boolean.
- The compiled template is written to `storage/framework/views` with a hashed filename.
- If `deleteCachedView` is `true`, the compiled file is deleted after rendering.
- Inline templates can use any Blade directive, including components, conditionals, and loops.
- The data array is extracted into variables within the template's scope.

**Constraints and Limitations:**

- **Temporary file accumulation:** If `deleteCachedView` is not used, compiled files accumulate in `storage/framework/views`. This is intentional for performance if the same template is rendered repeatedly.
- **Race conditions:** With `deleteCachedView: true`, concurrent requests rendering the same inline template may cause errors if one process deletes the file while another is using it.
- **`@extends` limitation:** Inline templates that use `@extends` may not work correctly with `@include` statements for child templates.
- **Performance:** Inline rendering compiles the template on every call unless the compiled file is cached. For frequently rendered templates, store them as view files instead.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Inline Blade Rendering**

```php
<?php
// app/Http/Controllers/EmailController.php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Blade;

class EmailController extends Controller
{
    public function preview()
    {
        $template = 'Hello, {{ $name }}! Your order #{{ $orderId }} has shipped.';

        return Blade::render($template, [
            'name' => 'Julian Bashir',
            'orderId' => '12345',
        ]);
    }
}
```

**Expected Output:** `Hello, Julian Bashir! Your order #12345 has shipped.`

**Why:** The `Blade::render()` method compiles the string template, extracts the provided data into variables, and returns the rendered HTML. No view file is required.

---

**Example 2: Rendering Database-Stored Email Templates**

```php
<?php
// app/Services/EmailTemplateService.php

namespace App\Services;

use App\Models\EmailTemplate;
use Illuminate\Support\Facades\Blade;

class EmailTemplateService
{
    public function render(string $slug, array $data = []): string
    {
        $template = EmailTemplate::where('slug', $slug)->firstOrFail();

        return Blade::render(
            $template->body,
            $data,
            deleteCachedView: true
        );
    }
}
```

```php
<?php
// Usage in a controller

$html = app(EmailTemplateService::class)->render('order-shipped', [
    'customerName' => $order->customer->name,
    'orderNumber' => $order->number,
    'trackingUrl' => $order->tracking_url,
]);

// Send email with $html...
```

**Expected Output:** The email template stored in the database is rendered with the order data and returned as HTML, with the temporary compiled file deleted after rendering.

**Why:** The service retrieves the template body from the database and uses `Blade::render()` with `deleteCachedView: true` to compile and render it. The compiled file is cleaned up automatically.

---

**Example 3: Inline Rendering with Components**

```php
<?php
// app/Http/Controllers/NotificationController.php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Blade;

class NotificationController extends Controller
{
    public function generate()
    {
        $notification = Blade::render(
            '<x-alert type="{{ $type }}">{{ $message }}</x-alert>',
            [
                'type' => 'success',
                'message' => 'Your profile has been updated.',
            ]
        );

        return response()->json(['html' => $notification]);
    }
}
```

**Expected Output:** The JSON response contains `{"html": "<div class=\"alert alert-success\">Your profile has been updated.</div>"}`.

**Why:** Inline Blade rendering supports Blade components (`<x-alert>`), allowing dynamic HTML generation that leverages the component system. The `type` and `message` variables are interpolated into the component tag.

### Real-World Cases

- **Email Templates:** Rendering database-stored email bodies with dynamic user data.
- **CMS Systems:** Rendering user-defined page templates from the database.
- **Notification Systems:** Generating dynamic HTML notifications for in-app or email delivery.
- **Report Generation:** Rendering HTML reports from templates stored in configuration.
- **CLI Tools:** Generating HTML output from Blade strings in Artisan commands.

### References

- Laravel Blade: Rendering Inline Blade Templates — https://laravel.com/docs/12.x/blade#rendering-inline-blade-templates
- Laravel Blade: Rendering Inline Blade Templates (Laravel 9) — https://laravel.com/docs/9.x/blade#rendering-inline-blade-templates
- Blade::render() with deleteCachedView Issue (GitHub) — https://github.com/laravel/framework/issues/53113
- Blade::render() unlink error (Laracasts) — https://laracasts.com/discuss/channels/laravel/bladerender-unlink-error
- How To Render A String As A Blade Template (Eddy Mens) — https://www.eddymens.com/blog/how-to-render-a-string-as-a-blade-template-in-laravel

---

## References

- Laravel Blade Templates Documentation (12.x) — https://laravel.com/docs/12.x/blade
- Laravel Blade: Rendering Blade Fragments — https://laravel.com/docs/12.x/blade#rendering-blade-fragments
- Laravel Blade: Rendering Inline Blade Templates — https://laravel.com/docs/12.x/blade#rendering-inline-blade-templates
- Laravel Views: Optimizing Views — https://laravel.com/docs/12.x/views#optimizing-views
- Laravel Deployment: Optimization — https://laravel.com/docs/12.x/deployment#optimization
- Dynamic Page Updates with Laravel Blade Fragments (Laravel News) — https://laravel-news.com/dynamic-page-updates-with-laravel-blade-fragments
- Laravel 9.39 Released (Blade Fragments) — https://laravel-news.com/laravel-9-39-released
- Laravel Production Performance Checklist — https://github.com/iliaal/whetstone
- How does `php artisan view:cache` work? (Laracasts) — https://laracasts.com/discuss/channels/laravel/how-does-php-artisan-viewcache-work
- Customizing Laravel Optimization with `--except` (Laravel News) — https://laravel-news.com/customizing-laravel-optimization-with-except
- Blade::render() with deleteCachedView Issue (GitHub) — https://github.com/laravel/framework/issues/53113
- Blade Fragments for HTMX/Turbo (GitCode Blog) — https://blog.gitcode.com/37ae997e076dd4cb03133c63159b0d29.html
- Laravel HTMX Package — https://mail.packalyst.com
- Enhancing Frontend Interactivity with Laravel Blade Fragments (Medium) — https://medium.com
- How To Render A String As A Blade Template (Eddy Mens) — https://www.eddymens.com/blog/how-to-render-a-string-as-a-blade-template-in-laravel