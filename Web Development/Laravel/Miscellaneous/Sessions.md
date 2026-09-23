# Sessions and Flash State: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Sessions provide a way to store information about a user across multiple HTTP requests, since HTTP itself is stateless. Flash state is a special category of session data that persists only for the next request (or a few requests if explicitly re-flashed).

**Technical Definition:** Laravel's session system is managed by the `SessionManager` class, which resolves session drivers from `config/session.php`. Each driver implements `SessionHandlerInterface`, providing `read()`, `write()`, `destroy()`, and `gc()` methods. The `Store` class (accessed via `$request->session()` or the `session()` helper) provides the fluent API for reading, writing, and flashing data. Flash data is stored under the `_flash` key with `new` and `old` arrays; data in `old` is available during the current request, while `new` is promoted to `old` on the next request before being aged out.

**Beginner-Friendly Explanation:** Sessions are like a temporary storage locker that follows a user around your application. When a user logs in, their user ID is stored in the session so they don't have to log in on every page. Flash data is even more temporary—it's like a note that self-destructs after the next page load. This is perfect for showing "Success! Your post was saved" messages that disappear once the user navigates away.

### Key Characteristics

- **Driver-Based Architecture:** Sessions can be stored in files, cookies, databases, Redis, Memcached, or arrays.
- **Horizontally Scalable:** Database and Redis drivers support multi-server deployments where file-based sessions fail .
- **Flash Data Lifetimes:** Data persists for exactly one additional request unless re-flashed.
- **Session ID Regeneration:** Regenerating the session ID prevents session fixation attacks.
- **Security Attributes:** `same_site`, `secure`, and `http_only` cookie attributes harden session security.
- **Unified API:** The `session()` helper and `$request->session()` provide identical functionality.

### Prerequisites

- A Laravel application with session middleware (`StartSession`) active in the `web` middleware group.
- A configured session driver in `.env` (`SESSION_DRIVER`).
- For database driver: a `sessions` table migration.
- For Redis driver: the `predis/predis` package or the PHP Redis extension .

### Related Programming Areas

- **Authentication:** Session guards store the authenticated user ID in the session.
- **CSRF Protection:** CSRF tokens are stored in the session and validated on POST requests.
- **Validation:** Failed validation flashes errors and old input to the session.
- **Caching:** Session data is distinct from application cache but both use similar drivers.

---

## 1. Session Drivers: Configurations for Horizontal Scaling

### Definitions

**Core Definition:** Session drivers determine where session data is physically stored between requests, ranging from local files to distributed caches.

**Technical Definition:** Each driver implements `SessionHandlerInterface` with `open()`, `close()`, `read()`, `write()`, `destroy()`, and `gc()` methods. The `file` driver stores serialized session data in `storage/framework/sessions`. The `cookie` driver encrypts session data and stores it client-side. The `database` driver uses the query builder to read/write the `sessions` table. `Redis` and `Memcached` use their respective clients for high-performance key-value storage. The `array` driver stores data in memory only (for testing) .

**Beginner-Friendly Explanation:** Where should Laravel keep the "sticky notes" that follow a user? It can keep them in files on the server (simple, one server only), in the user's browser cookies (no server storage, but 4KB limit), in a database table (works across many servers), or in a fast in-memory store like Redis (works across many servers and is very fast).

### Purposes

- To persist session state across HTTP requests.
- To enable horizontal scaling by using shared storage (database, Redis, Memcached) across multiple application servers .
- To provide high-performance session access for traffic-heavy applications using in-memory stores.
- To support testing isolation with the `array` driver.
- To allow custom session backends via the `extend` method .

### Syntax Rules and Structure

**Complete General Syntax — Configuration:**

```php
// config/session.php
return [
    'driver' => env('SESSION_DRIVER', 'file'),
    'lifetime' => env('SESSION_LIFETIME', 120),
    'expire_on_close' => false,
    'encrypt' => false,
    'files' => storage_path('framework/sessions'),
    'connection' => env('SESSION_CONNECTION'),
    'table' => 'sessions',
    'store' => env('SESSION_STORE'),
    'lottery' => [2, 100],
    'cookie' => env('SESSION_COOKIE', 'laravel_session'),
    'path' => '/',
    'domain' => env('SESSION_DOMAIN'),
    'secure' => env('SESSION_SECURE_COOKIE'),
    'http_only' => true,
    'same_site' => 'lax',
];
```

**Component Breakdown:**

| Driver | Storage Location | Scaling Suitability |
|--------|-----------------|---------------------|
| `file` | `storage/framework/sessions` | Single server only  |
| `cookie` | Encrypted client cookie | Limited to 4KB; not recommended for production  |
| `database` | `sessions` table | Multi-server via shared DB  |
| `redis` | Redis server | Multi-server, high performance |
| `memcached` | Memcached server | Multi-server, high performance |
| `array` | PHP memory only | Testing only |

**Complete General Syntax — Database Driver Setup:**

```php
// Create sessions table migration
php artisan session:table
php artisan migrate
```

```php
// Example sessions table schema
Schema::create('sessions', function (Blueprint $table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->longText('payload');
    $table->integer('last_activity')->index();
});
```

**Complete General Syntax — Redis Driver Setup:**

```bash
# Install predis (or use phpredis extension)
composer require predis/predis
```

```env
# .env
SESSION_DRIVER=redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

**Syntax Rules:**
- The `driver` key in `config/session.php` determines the storage backend.
- Database driver requires the `sessions` table migration.
- Redis/Memcached require their respective PHP extensions or client libraries.
- For load-balanced applications, file and cookie drivers are not recommended .
- The `array` driver prevents session persistence across requests, ideal for tests .

**Constraints and Limitations:**
- **File driver:** Cannot scale across multiple servers without shared storage (NFS) .
- **Cookie driver:** 4KB browser limit; no chunking; not suitable for large sessions .
- **Database driver:** Requires a database connection for every request; adds database load.
- **Redis/Memcached:** Requires external service configuration and monitoring.
- **Session locking:** Concurrent requests may experience blocking with some drivers.

### Multiple Annotated Complete Code Examples

**Example 1: Database Session Driver for Multi-Server Deployment**

```env
# .env
SESSION_DRIVER=database
DB_CONNECTION=mysql
```

```php
<?php
// database/migrations/xxxx_create_sessions_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('sessions');
    }
};
```

**Expected Output:** Session data is stored in the `sessions` table and shared across all application servers connected to the same database.

**Why:** The database driver uses a shared database, enabling horizontal scaling. Each server reads and writes session data from the same table .

---

**Example 2: Redis Session Driver for High Performance**

```env
# .env
SESSION_DRIVER=redis
REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

```php
<?php
// config/database.php (redis connection)
'redis' => [
    'client' => env('REDIS_CLIENT', 'phpredis'),
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],
],
```

**Expected Output:** Session data is stored in Redis with sub-millisecond read/write performance, suitable for high-traffic applications.

**Why:** Redis is an in-memory data store. Sessions are read and written without disk I/O, making it ideal for performance-critical applications .

---

**Example 3: Array Driver for Testing**

```php
<?php
// phpunit.xml or config/session.php during tests
'environment' => [
    'SESSION_DRIVER' => 'array',
],
```

**Expected Output:** Session data is stored in memory and not persisted across requests. Tests run in isolation.

**Why:** The `array` driver prevents session data from leaking between test methods, ensuring test isolation .

### Real-World Cases

- **Single Server:** `file` driver is sufficient for small applications on one server.
- **Load Balanced:** `database` or `redis` driver is required for multiple application servers .
- **High Traffic:** `redis` or `memcached` driver provides the fastest session access.
- **Testing:** `array` driver ensures session isolation between tests.
- **Legacy Compatibility:** The `kevinsmith/laravel-samesite-none-compat` package provides fallback cookies for older browsers when using `SameSite=None` .

### References

- Laravel Session Documentation (12.x) — https://laravel.com/docs/12.x/session 
- Laravel Lumen Session Documentation — https://lumen.laravel.com/docs/5.1/session 
- Auth0 Laravel SDK: Sessions — https://github.com/auth0/laravel-auth0/blob/HEAD/docs/Sessions.md 
- Laravel 5.1 Session Documentation — https://laravel.com/docs/5.1/session 

---

## 2. Reading, Writing, and Removing Session Data

### Definitions

**Core Definition:** The Session facade and global `session()` helper provide a unified API for storing, retrieving, and deleting session data.

**Technical Definition:** The `Store` class implements `Illuminate\Contracts\Session\Session` and provides methods including `get()`, `put()`, `has()`, `exists()`, `missing()`, `pull()`, `forget()`, `flush()`, `push()`, `increment()`, and `decrement()`. The `session()` helper returns the `Store` instance or sets values when passed an array. The `$request->session()` method returns the same instance.

**Beginner-Friendly Explanation:** You can interact with the session using the `session()` helper function or by calling methods on `$request->session()`. The API is simple: `put()` to store, `get()` to retrieve, `forget()` to remove, and `flush()` to clear everything.

### Purposes

- To store user-specific state across requests (preferences, cart contents).
- To retrieve stored values with optional defaults.
- To check for the existence of session keys.
- To remove specific keys or clear the entire session.
- To push to array-based session values.
- To increment/decrement numeric session values.

### Syntax Rules and Structure

**Complete General Syntax — Reading Data:**

```php
// Via request instance
$value = $request->session()->get('key', 'default');
$value = $request->session()->get('key', function () {
    return 'default';
});

// Via global helper
$value = session('key');
$value = session('key', 'default');

// Retrieve all
$data = $request->session()->all();

// Retrieve subset
$data = $request->session()->only(['username', 'email']);
$data = $request->session()->except(['password']);
```

**Complete General Syntax — Writing Data:**

```php
// Via request instance
$request->session()->put('key', 'value');

// Via global helper
session(['key' => 'value']);

// Push to array
$request->session()->push('user.teams', 'developers');

// Increment/decrement
$request->session()->increment('count');
$request->session()->increment('count', 2);
$request->session()->decrement('count');
$request->session()->decrement('count', 2);
```

**Complete General Syntax — Removing Data:**

```php
// Retrieve and delete in one statement
$value = $request->session()->pull('key', 'default');

// Remove single key
$request->session()->forget('key');

// Remove multiple keys
$request->session()->forget(['key1', 'key2']);

// Remove all session data
$request->session()->flush();
```

**Complete General Syntax — Checking Existence:**

```php
// Returns true if present and not null
if ($request->session()->has('users')) { /* ... */ }

// Returns true if present, even if null
if ($request->session()->exists('users')) { /* ... */ }

// Returns true if not present
if ($request->session()->missing('users')) { /* ... */ }
```

**Component Breakdown:**

| Method | Purpose |
|--------|---------|
| `get($key, $default)` | Retrieve value with optional default . |
| `put($key, $value)` | Store a value. |
| `has($key)` | Check existence (not null). |
| `exists($key)` | Check existence (including null). |
| `missing($key)` | Check non-existence. |
| `pull($key, $default)` | Retrieve and delete. |
| `forget($key)` | Delete a key. |
| `flush()` | Delete all session data. |
| `push($key, $value)` | Append to array value. |
| `increment($key, $amount)` | Increment numeric value. |

**Syntax Rules:**
- The `session()` helper can be used for both getting (`session('key')`) and setting (`session(['key' => 'value'])`).
- `has()` returns `false` if the value is `null`; `exists()` returns `true` .
- `pull()` combines `get()` and `forget()` in one operation .
- `flush()` removes all session data, including flash data.

**Constraints and Limitations:**
- **Reserved key:** Do not use the `flash` session key directly; Laravel uses it internally .
- **`has()` vs `exists()`:** `has()` returns `false` for `null` values; `exists()` returns `true` .
- **`flush()` scope:** Removes all session data, not just specific keys.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Session CRUD**

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function show(Request $request, $id)
    {
        // Store a value
        $request->session()->put('user_id', $id);

        // Retrieve a value
        $userId = $request->session()->get('user_id');

        // Check existence
        if ($request->session()->has('user_id')) {
            // ...
        }

        // Remove a value
        $request->session()->forget('user_id');
    }
}
```

**Expected Output:** The `user_id` value is stored, retrieved, checked, and removed from the session.

**Why:** `put()` stores data, `get()` retrieves it, `has()` checks existence, and `forget()` removes it .

---

**Example 2: Session Array Operations**

```php
<?php
// Push to an array
session(['user.teams' => ['engineering']]);
$request->session()->push('user.teams', 'developers');
// Result: ['engineering', 'developers']

// Increment/decrement
session(['count' => 5]);
$request->session()->increment('count'); // 6
$request->session()->decrement('count', 2); // 4

// Pull (retrieve and delete)
$value = $request->session()->pull('count'); // 4
// 'count' is now removed from session
```

**Expected Output:** Array values are appended; numeric values are incremented/decremented; `pull()` retrieves and removes in one operation.

**Why:** `push()` appends to array values. `increment()`/`decrement()` modify numeric values. `pull()` combines retrieval and deletion .

### Real-World Cases

- **Shopping Cart:** `session(['cart' => [...]])` stores cart items across requests.
- **User Preferences:** `session(['theme' => 'dark'])` remembers user choices.
- **Multi-Step Forms:** Session stores intermediate form data between steps.
- **Rate Limiting:** Session counters track user actions within a time window.

### References

- Laravel HTTP Session (12.x) — https://laravel.com/docs/12.x/session 
- Laravel Session (Vectorial1024/docs) — https://github.com/Vectorial1024/docs/blob/12.x/session.md 
- Laravel Lumen Session — https://lumen.laravel.com/docs/5.1/session 

---

## 3. Flash Data: Managing Transient, Single-Request Lifetimes

### Definitions

**Core Definition:** Flash data is session data that persists only for the next request (or a few requests if explicitly re-flashed), designed for short-lived status messages and form input preservation.

**Technical Definition:** The `Store::flash()` method stores data under the `_flash.new` key. After the request completes, the `ageFlashData()` method promotes `_flash.new` to `_flash.old` and removes data from `_flash.old`. On the next request, data in `_flash.old` is still available via `get()` but will be removed after that request. The `reflash()` method moves all `_flash.old` data back to `_flash.new`, extending its lifetime. The `keep()` method does the same for specific keys .

**Beginner-Friendly Explanation:** Flash data is like a message that self-destructs after the next page load. You use it for things like "Success! Your post was saved"—the user sees it once, and it's gone. If you need the message to survive another redirect (like after a validation error), you call `reflash()` to keep it alive.

### Purposes

- To display status messages that appear once and then disappear.
- To preserve form input after validation errors for form re-population.
- To carry data across multi-step redirects.
- To avoid polluting the session with permanent data.
- To provide temporary state for "flash" notifications.

### Syntax Rules and Structure

**Complete General Syntax — Flashing Data:**

```php
// Store flash data (available this request and next)
$request->session()->flash('status', 'Task was successful!');

// Retrieve flash data
$status = $request->session()->get('status');
```

**Complete General Syntax — Reflashing:**

```php
// Keep ALL flash data for another request
$request->session()->reflash();

// Keep SPECIFIC flash data for another request
$request->session()->keep(['username', 'email']);
```

**Complete General Syntax — Current Request Only:**

```php
// Available only in the current request (not the next)
$request->session()->now('status', 'Task was successful!');
```

**Component Breakdown:**

| Method | Lifetime |
|--------|----------|
| `flash($key, $value)` | Current request + next request  |
| `reflash()` | Extends ALL flash data for one more request |
| `keep($keys)` | Extends SPECIFIC flash data for one more request |
| `now($key, $value)` | Current request only |

**Syntax Rules:**
- Flash data is stored under the `_flash` key with `new` and `old` sub-arrays.
- `flash()` data is available immediately and during the next request.
- After the next request, flash data is automatically removed unless `reflash()` or `keep()` is called.
- `now()` data is available immediately but not in the next request.
- The `reflash()` method keeps **all** current flash data for one more request.

**Constraints and Limitations:**
- **One-request lifetime:** Flash data is removed after the next request unless explicitly kept.
- **`now()` vs `flash()`:** `now()` is for current request only; `flash()` is for current + next request .
- **Reserved key:** Do not use `flash` as a session key; Laravel uses it internally .

### Multiple Annotated Complete Code Examples

**Example 1: Flash Status Message**

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostController extends Controller
{
    public function store(Request $request)
    {
        // Validate and create post
        $post = Post::create($request->validated());

        // Flash success message
        $request->session()->flash('status', 'Post created successfully!');

        return redirect()->route('posts.index');
    }

    public function index(Request $request)
    {
        // The status flash data is available here
        return view('posts.index', [
            'status' => $request->session()->get('status'),
        ]);
    }
}
```

**Expected Output:** After creating a post, the user is redirected to the index page, where the success message is displayed once. On the next page load, the message is gone.

**Why:** `flash()` stores the message for the current request and the next. The `index` method retrieves it. After that request, it is automatically removed .

---

**Example 2: Reflashing for Multi-Step Redirects**

```php
<?php
// Step 1: Store flash data
session()->flash('status', 'Step 1 complete.');

// Step 2: Reflash ALL data for another request
session()->reflash();

// Step 3: Reflash SPECIFIC data only
session()->keep(['status']);

// The status message will survive additional requests
```

**Expected Output:** The `status` message persists across multiple redirects because `reflash()` or `keep()` extends its lifetime.

**Why:** `reflash()` moves all `_flash.old` data back to `_flash.new`, giving it another request of life. `keep()` does the same for specific keys .

---

**Example 3: Preserving Form Input with Flash Data**

```php
<?php
// Controller: Flash input on validation failure
public function store(Request $request)
{
    $request->validate([
        'title' => 'required',
        'body' => 'required',
    ]);

    // ... create post
}

// Laravel automatically flashes input on validation failure
// The next request can retrieve it via old()
```

```blade
{{-- Blade: Re-populate form with old input --}}
<input type="text" name="title" value="{{ old('title') }}">
<textarea name="body">{{ old('body') }}</textarea>
```

**Expected Output:** After validation failure, the form is re-populated with the user's previous input.

**Why:** Laravel's validation system automatically flashes input to the session using `flash()`. The `old()` helper retrieves it on the next request .

### Real-World Cases

- **Success/Error Messages:** Flash "Post created!" or "Invalid credentials" messages.
- **Form Repopulation:** Preserve user input after validation errors.
- **Multi-Step Wizards:** Carry data between wizard steps.
- **Newsletter Subscriptions:** Flash "You're subscribed!" after form submission.

### References

- Laravel HTTP Session (12.x) — https://laravel.com/docs/12.x/session 
- Laravel Session (Vectorial1024/docs) — https://github.com/Vectorial1024/docs/blob/12.x/session.md 
- Laravel 4.2 Session Documentation — https://laravel.com/framework/docs/4.2/session 
- Laravel HTTP Requests: Old Input — https://laravel.com/docs/13.x/requests 

---

## 4. Session Security: Session ID Regeneration

### Definitions

**Core Definition:** Session ID regeneration creates a new session identifier while preserving session data, preventing session fixation attacks where an attacker sets a known session ID before the user logs in.

**Technical Definition:** The `Store::regenerate()` method generates a new session ID using `SessionIdInterface::generate()` and updates the session storage. The `migrate()` method performs regeneration while preserving data. The `invalidate()` method regenerates the ID **and** removes all data, used on logout. Laravel automatically regenerates the session ID during authentication when using starter kits or Fortify .

**Beginner-Friendly Explanation:** Session fixation is when a bad actor tricks you into using a session ID they already know. When you log in, they can hijack your session. Regenerating the session ID on login gives you a fresh ID that the attacker doesn't know, protecting your account.

### Purposes

- To prevent session fixation attacks.
- To refresh the session ID after privilege escalation (login).
- To invalidate the old session on logout.
- To maintain session data while changing the identifier.

### Syntax Rules and Structure

**Complete General Syntax — Regenerate:**

```php
// Regenerate session ID (preserves data)
$request->session()->regenerate();
```

**Complete General Syntax — Invalidate (Regenerate + Flush):**

```php
// Regenerate ID and remove all data
$request->session()->invalidate();
```

**Complete General Syntax — Migrate (Custom Destroys):**

```php
// Regenerate ID and destroy old session explicitly
$request->session()->migrate(true);
```

**Component Breakdown:**

| Method | Behaviour |
|--------|-----------|
| `regenerate()` | New ID, preserves data  |
| `invalidate()` | New ID, removes all data  |
| `migrate($destroy)` | New ID, optionally destroys old  |

**Syntax Rules:**
- Call `regenerate()` after login to prevent fixation.
- Call `invalidate()` on logout to clear all session data.
- Laravel's starter kits and Fortify handle this automatically .
- The `regenerate()` method preserves all session data by default.

**Constraints and Limitations:**
- **Timing:** Regenerate **after** authentication, not before.
- **Data preservation:** `regenerate()` keeps data; `invalidate()` removes it.
- **CSRF token:** Regenerating the session ID does **not** regenerate the CSRF token. Call `regenerateToken()` separately if needed .

### Multiple Annotated Complete Code Examples

**Example 1: Login Session Regeneration**

```php
<?php
namespace App\Http\Controllers\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function store(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (Auth::attempt($credentials)) {
            // Regenerate session ID to prevent fixation
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'Invalid credentials.',
        ]);
    }
}
```

**Expected Output:** On successful login, the session ID changes. An attacker who knew the pre-login session ID cannot use it to access the authenticated session.

**Why:** `regenerate()` creates a new session ID while preserving the authentication state. This defeats session fixation attacks .

---

**Example 2: Logout Session Invalidation**

```php
<?php
namespace App\Http\Controllers\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LogoutController extends Controller
{
    public function destroy(Request $request)
    {
        Auth::logout();

        // Regenerate ID and remove all data
        $request->session()->invalidate();

        // Generate fresh CSRF token
        $request->session()->regenerateToken();

        return redirect('/');
    }
}
```

**Expected Output:** The user is logged out, all session data is removed, and a new session ID and CSRF token are generated.

**Why:** `invalidate()` regenerates the session ID and removes all data. `regenerateToken()` creates a fresh CSRF token for the now-unauthenticated session .

### Real-World Cases

- **Login:** Always regenerate session ID after successful authentication.
- **Logout:** Always invalidate the session and regenerate the CSRF token.
- **Privilege Escalation:** Regenerate when a user's role changes (e.g., sudo mode).
- **Password Change:** Invalidate other sessions when the password changes.

### References

- Laravel HTTP Session (12.x) — https://laravel.com/docs/12.x/session 
- Laravel Session (Vectorial1024/docs) — https://github.com/Vectorial1024/docs/blob/12.x/session.md 
- Basic Authentication: Login/Logout — https://laravel.com/index.php/learn/getting-started-with-laravel/basic-authentication-loginlogout 

---

## 5. Customizing Lifetimes, Cookie Paths, and same_site

### Definitions

**Core Definition:** Session configuration settings in `config/session.php` control session lifetime, cookie attributes (path, domain, secure, http_only, same_site), and driver-specific options.

**Technical Definition:** The `lifetime` key (minutes) determines how long session data is considered valid. The `expire_on_close` boolean controls whether the session cookie expires when the browser closes. The `cookie` key sets the cookie name. The `path` and `domain` keys control cookie scope. The `secure` key requires HTTPS. The `http_only` key prevents JavaScript access. The `same_site` key (`lax`, `strict`, `none`) controls cross-site cookie behaviour .

**Beginner-Friendly Explanation:** These settings control how long sessions last, where cookies are valid, and how they behave across different websites. The `same_site` setting is especially important for security—it prevents other websites from using your session cookies to make requests on your behalf.

### Purposes

- To control how long a user stays logged in (`lifetime`).
- To restrict cookie access to specific paths and domains (`path`, `domain`).
- To enforce HTTPS for session cookies (`secure`).
- To prevent JavaScript access to session cookies (`http_only`).
- To control cross-site cookie behaviour (`same_site`) for CSRF protection and cross-origin SPAs .

### Syntax Rules and Structure

**Complete General Syntax — config/session.php:**

```php
<?php
return [
    'driver' => env('SESSION_DRIVER', 'file'),
    'lifetime' => env('SESSION_LIFETIME', 120),
    'expire_on_close' => false,
    'encrypt' => false,
    'files' => storage_path('framework/sessions'),
    'connection' => env('SESSION_CONNECTION'),
    'table' => 'sessions',
    'store' => env('SESSION_STORE'),
    'lottery' => [2, 100],
    'cookie' => env('SESSION_COOKIE', 'laravel_session'),
    'path' => '/',
    'domain' => env('SESSION_DOMAIN'),
    'secure' => env('SESSION_SECURE_COOKIE'),
    'http_only' => true,
    'same_site' => 'lax',
];
```

**Component Breakdown:**

| Key | Purpose | Default |
|-----|---------|---------|
| `lifetime` | Session validity in minutes | 120 |
| `expire_on_close` | Cookie expires when browser closes | `false` |
| `cookie` | Session cookie name | `laravel_session` |
| `path` | Cookie path scope | `/` |
| `domain` | Cookie domain scope | `null` |
| `secure` | HTTPS-only cookie | `null` (false) |
| `http_only` | Prevent JavaScript access | `true` |
| `same_site` | Cross-site cookie policy | `lax` |

**Complete General Syntax — Cross-Origin SPA Configuration:**

```php
// config/session.php
'domain' => '.your-app.test',
'secure' => true,
'same_site' => 'none',
```

```php
// config/cors.php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => ['https://localhost:3000'],
'supports_credentials' => true,
```

**Complete General Syntax — same_site Values:**

| Value | Behaviour |
|-------|-----------|
| `lax` | Cookie sent on top-level navigation and same-site requests (default)  |
| `strict` | Cookie sent only on same-site requests |
| `none` | Cookie sent on all requests (requires `secure => true`)  |

**Syntax Rules:**
- `lifetime` is in **minutes**. 120 = 2 hours.
- `secure` must be `true` when `same_site` is `none` (browser requirement) .
- `domain` should be `.yourdomain.com` (with leading dot) to share cookies across subdomains.
- `same_site => 'none'` is required for cross-origin SPAs (frontend on different domain) .
- `http_only` should remain `true` for security (prevents XSS access).

**Constraints and Limitations:**
- **`same_site => 'none'` requires HTTPS:** Browsers reject `SameSite=None` cookies without `Secure` .
- **`null` vs `'none'`:** Setting `same_site => null` omits the attribute (browser defaults to Lax). Use `'none'` explicitly .
- **Cookie domain mismatch:** If `domain` doesn't match the application's domain, cookies won't be set or read.
- **CSRF token regeneration:** Changing `same_site` may require clearing cookies and regenerating CSRF tokens.

### Multiple Annotated Complete Code Examples

**Example 1: Standard Web Application Configuration**

```php
<?php
// config/session.php
return [
    'driver' => env('SESSION_DRIVER', 'file'),
    'lifetime' => 120, // 2 hours
    'expire_on_close' => false,
    'cookie' => 'myapp_session',
    'path' => '/',
    'domain' => null,
    'secure' => env('SESSION_SECURE_COOKIE', false),
    'http_only' => true,
    'same_site' => 'lax', // CSRF protection, allows top-level navigation
];
```

**Expected Output:** Sessions last 2 hours, cookies are accessible on all paths, HTTP-only, and use `SameSite=Lax` for CSRF protection.

**Why:** The default configuration works for most web applications. `SameSite=Lax` protects against CSRF while allowing normal navigation .

---

**Example 2: Cross-Origin SPA with Sanctum**

```env
# .env
SESSION_DRIVER=database
SESSION_DOMAIN=.localhost
SESSION_SECURE_COOKIE=true
SANCTUM_STATEFUL_DOMAINS=localhost:3000
```

```php
<?php
// config/session.php
return [
    'domain' => '.localhost',
    'secure' => true,
    'same_site' => 'none',
    'http_only' => true,
];
```

```javascript
// Frontend (Axios)
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

**Expected Output:** The SPA on `localhost:3000` can authenticate with the API on `localhost:8000` using cookies with `SameSite=None; Secure`.

**Why:** Cross-origin cookies require `same_site => 'none'` and `secure => true`. The CORS configuration must also allow credentials .

---

**Example 3: High-Security Configuration**

```php
<?php
// config/session.php
return [
    'lifetime' => 30, // 30 minutes for sensitive applications
    'expire_on_close' => true, // Session ends when browser closes
    'secure' => true, // HTTPS only
    'http_only' => true, // No JavaScript access
    'same_site' => 'strict', // Strict CSRF protection
];
```

**Expected Output:** Sessions expire after 30 minutes or when the browser closes. Cookies are HTTPS-only, HTTP-only, and use `SameSite=Strict`.

**Why:** High-security applications benefit from shorter lifetimes and stricter cookie policies. `SameSite=Strict` prevents cookies from being sent on cross-site navigation .

### Real-World Cases

- **Standard Web App:** `lifetime => 120`, `same_site => 'lax'`.
- **Banking/Healthcare:** `lifetime => 15`, `expire_on_close => true`, `same_site => 'strict'`.
- **Cross-Origin SPA:** `same_site => 'none'`, `secure => true`, `domain => '.domain.com'` .
- **Multi-Subdomain:** `domain => '.domain.com'` shares cookies across subdomains.
- **Legacy Browser Support:** The `kevinsmith/laravel-samesite-none-compat` package provides fallback cookies for browsers that don't support `SameSite=None` .

### References

- Laravel HTTP Session (12.x) — https://laravel.com/docs/12.x/session 
- OWASP AASVS 3.4.3: SameSite Attribute — https://owasp-aasvs4.readthedocs.io/en/latest/3.4.3.html 
- Laravel API CSRF-cookie Issue (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/laravel-api-csrf-cookie-is-not-set-on-the-nextjs-project 
- SameSite=None Compat Package — https://root.packagist.org/packages/kevinsmith/laravel-samesite-none-compat 
- Laravel SaaS Package Configuration — https://root.packagist.org/packages/helori/laravel-saas 

---

## References

- Laravel HTTP Session (12.x) — https://laravel.com/docs/12.x/session 
- Laravel Session (Vectorial1024/docs) — https://github.com/Vectorial1024/docs/blob/12.x/session.md 
- Laravel Lumen Session Documentation — https://lumen.laravel.com/docs/5.1/session 
- Laravel 4.2 Session Documentation — https://laravel.com/framework/docs/4.2/session 
- Laravel 5.1 Session Documentation — https://laravel.com/docs/5.1/session 
- Laravel HTTP Requests: Old Input — https://laravel.com/docs/13.x/requests 
- Basic Authentication: Login/Logout — https://laravel.com/index.php/learn/getting-started-with-laravel/basic-authentication-loginlogout 
- Auth0 Laravel SDK: Sessions — https://github.com/auth0/laravel-auth0/blob/HEAD/docs/Sessions.md 
- OWASP AASVS 3.4.3: SameSite Attribute — https://owasp-aasvs4.readthedocs.io/en/latest/3.4.3.html 
- Laravel API CSRF-cookie Issue (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/laravel-api-csrf-cookie-is-not-set-on-the-nextjs-project 
- SameSite=None Compat Package — https://root.packagist.org/packages/kevinsmith/laravel-samesite-none-compat 
- Laravel SaaS Package Configuration — https://root.packagist.org/packages/helori/laravel-saas 
- Filament 419 Session Driver Database (Laracasts) — https://laracasts.com/discuss/channels/filament/filament-419-on-every-request-only-when-session-driverdatabase 
- Laravel Session 5.6 PDF — https://github.com/DevStorm-Team/laravel-book/blob/26c221f636bd61e3aff94d5bc2754fc86bfc44fc/laravel-docs-5.6.pdf 
- Laravel 5.4 Session PDF — https://github.com/IsmailShurrab/laravel-book/blob/master/laravel-docs-5.4.pdf 
- Migrate from Redis to Valkey (Laravel Cloud) — https://laravel.com/cloud/docs/knowledge-base/migrate-upstash-to-valkey 