# Production Autoloading & Performance Optimization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Production autoloading is the mechanism by which PHP automatically locates and loads class files on demand, eliminating the need for manual `require` or `include` statements. Composer generates a standards-compliant autoloader based on PSR-4 (the modern standard), PSR-0 (legacy), classmaps, or file inclusions. Performance optimization involves converting these dynamic resolution rules into static, pre-computed lookup tables that eliminate filesystem checks at runtime.

**Technical Definition**

Composer's autoloader is a PHP class (`Composer\Autoload\ClassLoader`) that registers itself with SPL via `spl_autoload_register()`. It maps fully-qualified class names to file paths using one or more strategies: PSR-4 namespace prefixes, PSR-0 legacy namespace prefixes, classmap directories (which scan for all `.php` and `.inc` files), and file-level inclusions. The autoloader's default behaviour performs a filesystem check (`file_exists()`) for every class lookup under PSR-4/PSR-0 rules. Production optimization flags (`--optimize`, `--classmap-authoritative`) convert the dynamic PSR-4/PSR-0 rules into a static classmap that returns file paths instantly without filesystem I/O. The classmap is also cached in opcache on PHP 5.6+, making initialization nearly instant.

**Beginner-Friendly Explanation**

Imagine you have a library with thousands of books (classes) arranged by a complex filing system (PSR-4 namespaces). Every time someone asks for a book, you walk to the shelf, check if it exists, and then retrieve it. That walk takes time. Production optimization is like creating a master index at the front desk: "Book X is on shelf 42, row 3." Now you can find any book instantly without walking the shelves. The trade-off is that if someone adds a new book, you must update the index — which is why optimizations are for production, not development.

### Key Characteristics

- **PSR-4 as the Modern Standard:** Maps fully-qualified class names to file paths deterministically, with the namespace prefix consumed and the remaining sub-namespaces mapping to sub-directories.
- **Classmap as the Performance Fallback:** Scans directories for all `.php` and `.inc` files and builds a static `class => file` array, used for libraries that do not follow PSR-0/4.
- **File Inclusions for Functions and Constants:** The `files` autoloading mechanism explicitly requires files on every request, used for non-class code such as helper functions.
- **Dynamic vs. Static Resolution:** Development favours dynamic PSR-4 resolution (new classes are discovered immediately); production favours static classmaps (no filesystem checks).
- **Opcache Integration:** The generated classmap is cached in opcache, eliminating file reads on subsequent requests.
- **Authoritative Mode:** `--classmap-authoritative` tells the autoloader to skip PSR-4/PSR-0 fallback entirely when a class is not in the classmap, providing the fastest possible lookup at the cost of flexibility.

### Prerequisites

- **PHP 8.1+** with Composer 2.x installed.
- **A project with a `composer.json`** file containing an `autoload` section.
- **Understanding of PHP namespaces** and the PSR-4 naming convention (namespace path mirrors directory path).
- **A deployment pipeline** where `composer install` or `composer dump-autoload` is run as a build step.
- **OPcache enabled** in production (`opcache.enable=1` in `php.ini`) for maximum performance gains.

### Related Programming Areas

- **PSR Standards (PHP-FIG):** PSR-4 (autoloading), PSR-0 (legacy autoloading), and PSR-1/PSR-12 (coding style).
- **Composer Dependency Management:** Autoloading is configured alongside dependency declarations in `composer.json`.
- **PHP OPcache and Preloading:** Classmap optimization is most effective when combined with opcache and preloading.
- **CI/CD Pipelines:** Production optimization flags are standard steps in deployment pipelines.
- **Framework Bootstrapping:** Laravel, Symfony, and other frameworks rely on Composer's optimized autoloader for fast application boot.

### Core Concepts / Features

1. **The PSR-4 Autoloading Standard** — Mapping namespaces to directory paths.
2. **Alternative Autoloading Schemas** — PSR-0 (legacy), classmaps, and file-level inclusions.
3. **Production Optimization Flags** — `--optimize` (-o) and `--classmap-authoritative` (-a).

---

## Core Concept 1: The PSR-4 Autoloading Standard

### Definitions

**Core Definition**

PSR-4 is the PHP-FIG interoperability specification that maps a fully-qualified class name to a file path, so the interpreter can load classes on demand without a single manual `require`. 

**Technical Definition**

PSR-4 defines a deterministic transformation from a fully-qualified class name to a file path. A fully-qualified class name has the form `<NamespacePrefix>\<SubNamespaces...>\<ClassName>`. The specification states: the FQN must have a top-level namespace prefix (a "vendor namespace"), e.g. `App\`. A prefix maps to a base directory. Contiguous sub-namespaces after the prefix map to sub-directories under that base directory, with the namespace separator `\` becoming the directory separator `/`. The terminating class name maps to a file named `<ClassName>.php`. Autoloader implementations must not throw exceptions, raise errors, or return a value — they simply return silently when the name is not their responsibility.  Only the trailing portion after a matched prefix becomes a path. The prefix `App\` mapped to `src/` means `App\Http\Logger` → `src/Http/Logger.php` — the `App` segment is consumed by the prefix, not repeated in the path. 

**Beginner-Friendly Explanation**

Think of PSR-4 as a postal address system for PHP classes. The "vendor namespace" (`App\`) is like a country code. The sub-namespaces (`Http\Controller\`) are like province, city, and street. The class name (`UserController`) is like a house number. The post office (the autoloader) reads the address and delivers the letter (loads the file) from exactly one location. There is no ambiguity: every valid address maps to exactly one house.

### Purposes

- To provide a deterministic, interoperable mapping from class names to file paths across all PHP packages.
- To eliminate manual `require` and `include` statements, allowing classes to be referenced by name and loaded lazily on first use.
- To enable thousands of independent packages to share a single autoloader in one process.
- To establish a predictable project structure where the namespace hierarchy mirrors the directory hierarchy.
- To serve as the modern replacement for the deprecated PSR-0 standard.

### Syntax Rules and Structure

**Complete General Syntax: PSR-4 Configuration in `composer.json`**

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "Acme\\Blog\\": "packages/blog/src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/"
        }
    }
}
```

**Component Breakdown:**

- `"psr-4"` — The autoloading strategy key. Composer generates PSR-4-compliant rules.
- `"App\\": "src/"` — Maps the namespace prefix `App\` to the base directory `src/`. When a class like `App\Http\Controller\UserController` is referenced, Composer looks for `src/Http/Controller/UserController.php`. 
- `"Acme\\Blog\\": "packages/blog/src/"` — Maps a deeper namespace prefix (`Acme\Blog\`) to a nested base directory. A class like `Acme\Blog\Comment` resolves to `packages/blog/src/Comment.php`.
- `"autoload-dev"` — Development-only autoloading rules. These are excluded when `composer install --no-dev` is run. Used for test namespaces like `App\Tests\`.

**PSR-4 Mapping Examples:**

| Fully-Qualified Class Name | Prefix | Base Directory | Resolved File |
|---|---|---|---|
| `App\Logger` | `App\` | `src/` | `src/Logger.php` |
| `App\Http\Controller\UserController` | `App\` | `src/` | `src/Http/Controller/UserController.php` |
| `Acme\Blog\Comment` | `Acme\Blog\` | `packages/blog/src/` | `packages/blog/src/Comment.php` |
| `App\Tests\UserTest` | `App\Tests\` | `tests/` | `tests/UserTest.php` |

**Syntax Rules:**

- The namespace prefix must end with a double backslash (`\\`) in JSON, which represents a single backslash (`\`) in PHP.
- The namespace prefix must be a top-level namespace (vendor namespace), e.g., `App\`, not just `App` without the separator.
- The directory path is relative to the project root (where `composer.json` resides).
- PSR-4 allows the base directory to be omitted: `"App\\": ""` maps `App\Foo` to `Foo.php` in the project root.
- PSR-4 is case-sensitive; the namespace path must match the directory path exactly. 

**Constraints and Limitations:**

- **Case sensitivity:** PSR-4 requires the fully qualified class name to match the filesystem path, and is case-sensitive. Mismatches cause "class not found" errors on case-sensitive file systems (Linux).
- **No filesystem check for known classes:** By default, Composer's PSR-4 autoloader still performs a `file_exists()` check before loading. Production optimization eliminates this check for classmapped classes.
- **Deprecated PSR-0 co-existence:** PSR-4 can co-exist with PSR-0, but PSR-0 is deprecated and should not be used for new code.
- **Prefix length matters:** Longer prefixes (e.g., `Acme\Blog\`) are matched before shorter ones. Composer sorts prefixes by length to ensure correct resolution.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Complete PSR-4 Setup and Usage**

```bash
# Step 1: Create the project structure
mkdir -p my-app/src/Http/Controller
cd my-app

# Step 2: Create a class
cat > src/Http/Controller/UserController.php << 'EOF'
<?php
namespace App\Http\Controller;

class UserController
{
    public function index(): string
    {
        return "UserController::index() was called.";
    }
}
EOF

# Step 3: Create composer.json with PSR-4 autoloading
cat > composer.json << 'EOF'
{
    "name": "acme/my-app",
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
EOF

# Step 4: Generate the autoloader
composer dump-autoload

# Step 5: Create the entry point
cat > index.php << 'EOF'
<?php
require __DIR__ . '/vendor/autoload.php';

use App\Http\Controller\UserController;

$controller = new UserController();
echo $controller->index() . "\n";
EOF

# Step 6: Run the script
php index.php
```

**Expected Output:**

```
UserController::index() was called.
```

**Why:** Composer reads the `psr-4` configuration, maps `App\` to `src/`, and generates a PSR-4 autoloader in `vendor/autoload.php`. When `new UserController()` is executed, PHP calls the registered autoloader with the fully-qualified class name `App\Http\Controller\UserController`. The autoloader strips the `App\` prefix, converts the remaining namespace separators to directory separators, appends `.php`, and loads `src/Http/Controller/UserController.php`. 

**Example 2: Multiple Namespace Prefixes**

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "Acme\\Blog\\": "packages/blog/src/",
            "Acme\\Shop\\": "packages/shop/src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/"
        }
    }
}
```

**Expected File Structure:**

```
project/
├── composer.json
├── src/
│   └── Http/
│       └── Controller/
│           └── UserController.php      # App\Http\Controller\UserController
├── packages/
│   ├── blog/
│   │   └── src/
│   │       └── Comment.php             # Acme\Blog\Comment
│   └── shop/
│       └── src/
│           └── Product.php             # Acme\Shop\Product
└── tests/
    └── UserTest.php                    # App\Tests\UserTest
```

**Why:** Each PSR-4 prefix maps to its own base directory. The `autoload-dev` section is separate so that test classes are not autoloaded in production when `--no-dev` is used. 

### Real-World Cases

**Case 1: Laravel Application Structure**

Laravel's default `composer.json` maps `"App\\": "app/"` and `"Database\\Factories\\": "database/factories/"`. All application classes live under the `App\` namespace, and the directory structure mirrors the namespace hierarchy exactly. Laravel's production optimization runs `composer install --no-dev --optimize-autoloader` as part of its deployment process.

**Case 2: Symfony Bundles**

Symfony bundles declare PSR-4 autoloading in their `composer.json`. A bundle named `Acme\BlogBundle` maps to `src/` within the bundle package. When installed via Composer, the bundle's classes are merged into the application's autoloader, allowing `use Acme\BlogBundle\Controller\PostController` to work without manual includes.

**Case 3: Yii Framework**

Yii uses Composer to manage packages and generates a PSR-4-compatible autoloader. Developers configure `"psr-4": { "App\\": "src/" }` in `composer.json`, run `composer dump-autoload`, and all classes under the `App\` namespace are loaded automatically. 

---

## Core Concept 2: Alternative Autoloading Schemas

### Definitions

**Core Definition**

Alternative autoloading schemas are Composer's other class-loading strategies: PSR-0 (the legacy namespace convention), classmap (scanning directories for all classes), and files (including specific files on every request).

**Technical Definition**

**PSR-0** is the deprecated predecessor to PSR-4. It maps namespace prefixes and underscores in class names to directory separators. Unlike PSR-4, PSR-0 includes the namespace prefix in the path (e.g., `Acme\Log\Writer\File_Writer` maps to `./acme-log-writer/lib/File/Writer/File/Writer.php` under PSR-0, whereas PSR-4 consumes the prefix). **Classmap** autoloading scans specified directories and files for `.php` and `.inc` files, parses them to extract class, interface, and trait declarations, and builds a static `class => file` array stored in `vendor/composer/autoload_classmap.php`. **Files** autoloading explicitly requires the specified files on every request, regardless of whether any class is used; this is used for packages that contain functions or constants that cannot be autoloaded by class name.

**Beginner-Friendly Explanation**

PSR-0 is the old address system — it works, but it is more complicated and less flexible than PSR-4. Classmap is a master index of every class in your project; it is fast but must be rebuilt when classes change. Files autoloading is like leaving a note on the front door that everyone reads before entering — it is used for helper functions that are not classes.

### Purposes

- To provide autoloading for legacy code that follows PSR-0 instead of PSR-4.
- To support libraries and packages that do not follow any namespace standard, by scanning directories with classmap.
- To load non-class code (functions, constants) that cannot be autoloaded by class name, using file-level inclusions.
- To offer flexibility during migration from legacy standards to PSR-4.
- To serve as the foundation for production optimization (classmap generation).

### Sub-Feature 2.1: PSR-0 (Legacy)

#### Definitions

**Core Definition**

PSR-0 is the deprecated PHP-FIG standard for autoloading that maps namespace prefixes and underscores to directory separators, including the prefix in the resolved path.

**Technical Definition**

Under PSR-0, a fully-qualified class name is transformed by replacing namespace separators (`\`) and underscores (`_`) in the class name with directory separators (`/`). The entire namespace prefix is included in the path. For example, `\Acme\Log\Writer\File_Writer` maps to `./acme-log-writer/lib/File/Writer/File/Writer.php` when the prefix `Acme\Log\Writer` is mapped to `./acme-log-writer/lib/`. PSR-0 also allows non-namespaced classes with underscores in the class name (e.g., `PEAR_Text_Diff` maps to `PEAR/Text/Diff.php`). 

**Beginner-Friendly Explanation**

PSR-0 was the first attempt at a universal autoloading standard. It works, but it has a quirk: it includes the entire namespace prefix in the file path. PSR-4 fixed this by consuming the prefix, making paths shorter and directory structures cleaner. New projects should never use PSR-0.

#### Syntax Rules and Structure

```json
{
    "autoload": {
        "psr-0": {
            "Acme\\Log\\Writer\\": "acme-log-writer/lib/",
            "PEAR_": "pear/"
        }
    }
}
```

**Component Breakdown:**

- `"Acme\\Log\\Writer\\": "acme-log-writer/lib/"` — Maps the PSR-0 prefix to a base directory. The full namespace prefix is included in the resolved path.
- `"PEAR_": "pear/"` — Maps a non-namespaced prefix (using underscores) to a directory.

#### Constraints and Limitations

- **Deprecated:** PSR-0 is deprecated in favour of PSR-4. New code should not use it.
- **Deeper directory trees:** Because the prefix is included in the path, PSR-0 produces longer, deeper directory structures than PSR-4.
- **Underscore ambiguity:** PSR-0 treats underscores in class names as directory separators, which conflicts with modern naming conventions.

---

### Sub-Feature 2.2: Classmap Autoloading

#### Definitions

**Core Definition**

Classmap autoloading builds a static array mapping every class name to its file path by scanning specified directories for `.php` and `.inc` files.

**Technical Definition**

The `classmap` key in `composer.json` points to directories or files. During `composer install` or `composer dump-autoload`, Composer scans these locations, parses every `.php` and `.inc` file, extracts class, interface, and trait declarations, and generates a `class => file` array stored in `vendor/composer/autoload_classmap.php`. This map is returned instantly for known classes, with no filesystem check.  Classmap generation is particularly useful for libraries that do not follow PSR-0 or PSR-4 standards. 

**Beginner-Friendly Explanation**

Classmap is like creating a telephone directory for every class in your project. Instead of searching the filesystem every time you need a class, the autoloader looks up the class in the directory and goes directly to the file. It is faster than PSR-4 because there is no `file_exists()` check, but the directory must be rebuilt whenever a new class is added.

#### Syntax Rules and Structure

```json
{
    "autoload": {
        "classmap": [
            "src/",
            "legacy/",
            "vendor/old-library/"
        ]
    }
}
```

**Component Breakdown:**

- `"classmap"` — An array of directories or file paths to scan.
- `"src/"` — Composer scans all `.php` and `.inc` files in `src/` recursively, extracting class declarations.
- The generated classmap is stored in `vendor/composer/autoload_classmap.php`.

#### Constraints and Limitations

- **Must be rebuilt on class addition:** If a new class is added to a scanned directory, the classmap must be regenerated with `composer dump-autoload` for the class to be autoloadable.
- **Performance vs. flexibility trade-off:** Classmap is faster than PSR-4 for known classes, but misses (classes not in the map) fall back to PSR-4 rules, which still incur filesystem checks.
- **Scanning overhead:** Initial classmap generation scans every file, which can be slow for very large directories. This is a one-time cost during deployment.

---

### Sub-Feature 2.3: Files Autoloading (File-Level Inclusions)

#### Definitions

**Core Definition**

The `files` autoloading mechanism explicitly requires specified files on every request, used for non-class code such as functions and constants.

**Technical Definition**

The `files` key in `composer.json` points to specific files (not directories). These files are required automatically by Composer's autoloader on every request, regardless of whether any class from them is used. This is necessary for packages that define global functions, constants, or other non-class code that cannot be autoloaded by class name. 

**Beginner-Friendly Explanation**

Files autoloading is like a doorbell that rings every time you enter the house — whether you need it or not. It is used for helper functions (like `array_flatten()` or `str_contains()`) that are not part of any class. Since PHP can only autoload classes, functions must be explicitly included.

#### Syntax Rules and Structure

```json
{
    "autoload": {
        "files": [
            "src/helpers.php",
            "src/constants.php"
        ]
    }
}
```

**Component Breakdown:**

- `"files"` — An array of file paths (not directories) to require on every request.
- Each file is required exactly once via `require_once` when the autoloader is included.

#### Constraints and Limitations

- **Performance overhead:** Every file in the `files` list is loaded on every request, even if its functions are never called. Use sparingly.
- **No lazy loading:** Unlike classes, functions cannot be lazily loaded. The file must be present and loaded at bootstrap.
- **Global namespace pollution:** Functions defined in `files` occupy the global namespace (unless namespaced), potentially causing naming conflicts.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Classmap for Legacy Code**

```json
{
    "autoload": {
        "classmap": ["legacy/"]
    }
}
```

```php
<?php
// legacy/OldUser.php
class OldUser
{
    public function getName(): string
    {
        return 'Legacy User';
    }
}
```

```bash
# Generate the classmap
composer dump-autoload

# Verify the classmap contains OldUser
php -r "var_export(require 'vendor/composer/autoload_classmap.php');" | grep OldUser
```

**Expected Output (excerpt):**

```
'OldUser' => __DIR__ . '/../legacy/OldUser.php',
```

**Why:** Composer scans `legacy/`, finds `OldUser.php`, extracts the `OldUser` class declaration, and adds it to the classmap. When `new OldUser()` is called, the autoloader returns the file path instantly from the classmap.

**Example 2: Files Autoloading for Helper Functions**

```json
{
    "autoload": {
        "files": ["src/helpers.php"]
    }
}
```

```php
<?php
// src/helpers.php
if (!function_exists('format_price')) {
    function format_price(float $amount): string
    {
        return '$' . number_format($amount, 2);
    }
}
```

```php
<?php
// index.php
require __DIR__ . '/vendor/autoload.php';

echo format_price(49.99) . "\n";
```

**Expected Output:**

```
$49.99
```

**Why:** The `helpers.php` file is required automatically by the Composer autoloader. `format_price()` is available without any explicit `require` in `index.php`. The `function_exists` guard prevents fatal errors if the file is loaded twice.

### Real-World Cases

**Case 1: PHPUnit's Legacy Autoloading**

PHPUnit historically used a classmap to autoload its internal classes, which were not strictly PSR-4 compliant in older versions. Even today, PHPUnit's `composer.json` includes a classmap for `src/` to ensure all test framework classes are discovered.

**Case 2: Symfony Polyfills**

The `symfony/polyfill-*` packages use the `files` autoloading mechanism to define global functions (e.g., `mb_strlen`, `intdiv`) when the corresponding PHP extension is not available. These functions must be loaded before any code that uses them.

**Case 3: WordPress Plugin Libraries**

Legacy WordPress plugin libraries often do not follow PSR-4. Plugin developers use classmap autoloading to scan the library directory and make all classes available without manually requiring each file.

---

## Core Concept 3: Production Optimization Flags

### Definitions

**Core Definition**

Production optimization flags are Composer command-line options (`--optimize`, `--classmap-authoritative`) that convert the autoloader's dynamic PSR-4/PSR-0 resolution rules into static classmaps, eliminating filesystem checks and dramatically reducing class-loading overhead.

**Technical Definition**

Composer's default autoloader performs filesystem checks (`file_exists()`) for every class lookup under PSR-4 and PSR-0 rules. **Level 1 optimization** (`--optimize-autoloader` / `-o` / `dump-autoload -o`) converts PSR-4/PSR-0 rules into classmap rules. For known classes, the classmap returns the file path instantly, and Composer can guarantee the class is in the classmap without a filesystem check. On PHP 5.6+, the classmap is also cached in opcache, improving initialization time greatly. **Level 2/A optimization** (`--classmap-authoritative` / `-a` / `dump-autoload -a`) enables Level 1 and additionally instructs the autoloader that if a class is not found in the classmap, it does not exist. The autoloader skips the PSR-4/PSR-0 fallback entirely, providing the fastest possible lookup. 

**Beginner-Friendly Explanation**

Think of the default autoloader as a librarian who checks every shelf to see if a book exists before handing it to you. Level 1 optimization gives the librarian a master index: "If the book is in the index, go straight to the shelf." Level 2 optimization says: "If the book is not in the index, do not bother checking the shelves — it does not exist." Level 2 is the fastest, but it requires that the index (classmap) is complete and up to date.

### Purposes

- To eliminate filesystem I/O during class resolution in production, reducing latency.
- To leverage PHP's opcache for caching the classmap in memory across requests.
- To reduce the initialization time of the autoloader on every request.
- To provide a clear, deterministic class resolution path with no filesystem checks.
- To optimise application boot time, particularly for frameworks with hundreds of classes.

### Syntax Rules and Structure

**Complete General Syntax: Level 1 Optimization (Classmap Generation)**

```bash
# Via CLI flag
composer dump-autoload --optimize
composer dump-autoload -o

# Via composer install/update
composer install --optimize-autoloader
composer install -o

# Via composer.json config
```

```json
{
    "config": {
        "optimize-autoloader": true
    }
}
```

**Complete General Syntax: Level 2/A Optimization (Authoritative Classmap)**

```bash
# Via CLI flag
composer dump-autoload --classmap-authoritative
composer dump-autoload -a

# Via composer install/update
composer install --classmap-authoritative
composer install -a

# Via composer.json config
```

```json
{
    "config": {
        "classmap-authoritative": true
    }
}
```

**Component Breakdown:**

- `--optimize` / `-o` — Enables Level 1 classmap generation. Converts PSR-4/PSR-0 rules into classmap rules. For known classes, no filesystem check is performed. On PHP 5.6+, the classmap is cached in opcache. 
- `--classmap-authoritative` / `-a` — Enables Level 1 and additionally treats the classmap as authoritative. If a class is not found in the classmap, the autoloader does not fall back to PSR-4/PSR-0 rules. This eliminates all filesystem checks, including for classes that do not exist. 
- `config.optimize-autoloader: true` — Permanently enables Level 1 optimization for all `composer install` and `composer dump-autoload` commands.
- `config.classmap-authoritative: true` — Permanently enables Level 2/A optimization.

**Syntax Rules:**

- **Development warning:** None of these optimizations should be enabled in development, as they all cause problems when adding or removing classes. The performance gains are not worth the trouble in a development setting. 
- **Deployment requirement:** The classmap must be regenerated on every deployment (`composer dump-autoload -o -a`) because new classes added between deploys will not be in the classmap.
- **OPcache required for maximum benefit:** Ensure OPcache is enabled (`opcache.enable=1`) so the classmap is cached in memory. 
- **Class existence assumption:** With `--classmap-authoritative`, Composer assumes that any class not in the classmap does not exist. This is safe in production but dangerous in development where classes are added dynamically.

**Constraints and Limitations:**

- **Authoritative mode and dynamic classes:** If your application uses `eval()` or dynamically generated classes, `--classmap-authoritative` may cause "class not found" errors because those classes are not in the classmap.
- **Classmap size:** For very large projects with tens of thousands of classes, the classmap file can become large (several megabytes). Opcache mitigates this, but memory usage must be monitored.
- **No fallback in authoritative mode:** If a class is missing from the classmap (e.g., due to a failed deployment), there is no fallback mechanism. The application will throw a "class not found" fatal error.
- **APCu alternative:** Some deployments use APCu caching (`--apcu`) instead of opcache for classmap caching. APCu is a userland cache and may be preferable in shared hosting environments where opcache configuration is restricted.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Comparing Autoloader Performance (Default vs. Optimized)**

```bash
# Step 1: Create a project with many classes
mkdir -p perf-test/src && cd perf-test

# Generate 500 classes
for i in $(seq 1 500); do
    mkdir -p src/Entity
    cat > "src/Entity/Entity$i.php" << EOF
<?php
namespace App\Entity;
class Entity$i {}
EOF
done

# Step 2: Create composer.json
cat > composer.json << 'EOF'
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
EOF

# Step 3: Generate default autoloader
composer dump-autoload
cp vendor/composer/autoload_classmap.php /tmp/default_classmap.php

# Step 4: Generate optimized autoloader
composer dump-autoload -o
cp vendor/composer/autoload_classmap.php /tmp/optimized_classmap.php

# Step 5: Generate authoritative autoloader
composer dump-autoload -a

# Step 6: Benchmark class loading
php -r "
require 'vendor/autoload.php';
\$start = microtime(true);
for (\$i = 1; \$i <= 500; \$i++) {
    class_exists('App\\\\Entity\\\\Entity' . \$i);
}
echo 'Optimized: ' . round((microtime(true) - \$start) * 1000, 2) . ' ms' . PHP_EOL;
"
```

**Expected Output (illustrative):**

```
Optimized: 2.34 ms
```

**Why:** With `-o` (and especially `-a`), the autoloader returns file paths directly from the classmap without filesystem checks. The 500 `class_exists()` calls complete in ~2 ms. Without optimization, the same loop would perform 500 `file_exists()` calls, taking significantly longer (typically 15–30 ms depending on the filesystem).

**Example 2: Configuring Permanent Optimization in `composer.json`**

```json
{
    "name": "acme/production-app",
    "require": {
        "php": ">=8.2"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "config": {
        "optimize-autoloader": true,
        "classmap-authoritative": true,
        "sort-packages": true
    }
}
```

```bash
# Deploy to production
composer install --no-dev

# The autoloader is automatically optimized and authoritative
```

**Expected Output:**

```
Generating optimized autoload files (authoritative)
...
```

**Why:** Setting `optimize-autoloader: true` and `classmap-authoritative: true` in `config` ensures that every `composer install` or `composer dump-autoload` in production automatically applies the optimizations, even if the deployment script forgets to pass the flags.

**Example 3: Production Deployment Script**

```bash
#!/bin/bash
set -e

# Step 1: Install production dependencies only
composer install --no-dev --no-interaction --prefer-dist

# Step 2: Regenerate the autoloader with maximum optimization
composer dump-autoload --optimize --classmap-authoritative

# Step 3: Clear any application-level caches
php artisan cache:clear  # Laravel example
php artisan config:cache
php artisan route:cache
php artisan view:cache

echo "Production deployment complete. Autoloader optimized."
```

**Expected Output:**

```
Generating optimized autoload files (authoritative)
...
Production deployment complete. Autoloader optimized.
```

**Why:** The deployment script explicitly runs `composer dump-autoload -o -a` after `composer install` to ensure the classmap is regenerated with the latest application classes. The `--no-dev` flag excludes development dependencies, and the framework cache commands warm the application caches. 

### Real-World Cases

**Case 1: Laravel Production Deployment**

Laravel's deployment documentation recommends running `composer install --no-dev --optimize-autoloader` as part of the deploy process. For maximum performance, `php artisan optimize` is also run, which internally calls `composer dump-autoload -o`. Laravel's `config/autoload.php` does not exist; the optimization is applied via CLI flags in the deployment pipeline. 

**Case 2: Symfony Production Optimization**

Symfony's `composer.json` includes `"optimize-autoloader": true` and `"classmap-authoritative": true` in the `config` section. The `composer install --no-dev` command in the Symfony Flex recipe automatically generates an authoritative classmap, reducing class-loading overhead to near-zero. Symfony also recommends enabling OPcache with `opcache.validate_timestamps=0` in production.

**Case 3: High-Traffic API with 10,000 Classes**

A high-traffic REST API has over 10,000 classes across its application and vendor directories. Without optimization, the default autoloader performs a filesystem check for each class, adding ~5–10 ms per request. With `composer dump-autoload -o -a`, the classmap is loaded once into opcache, and class resolution becomes a direct array lookup. The request latency decreases measurably, and CPU usage drops. 

**Case 4: Shared Hosting with APCu**

A project deployed on shared hosting where OPcache configuration is restricted uses `composer dump-autoload -o --apcu` to cache the classmap in APCu instead of opcache. APCu is a userland cache that does not require server-level PHP configuration changes. 

---

## References

- Composer: Autoloader optimization — https://getcomposer.org/doc/articles/autoloader-optimization.md 
- Composer: The composer.json Schema — https://getcomposer.org/doc/04-schema.md
- PHP-FIG: PSR-4: Autoloader — https://www.php-fig.org/psr/psr-4/ 
- Secure PHP Development: PSR-4 Autoloading Standard — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/PSR-Standards/PSR-4-Autoloading-Standard.md 
- Secure PHP Development: Alternative Autoloading Schemas — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Alternative-Autoloading-Schemas.md
- Composer: Autoloading — https://getcomposer.org/doc/01-basic-usage.md
- PHP Manual: Autoloading Classes — https://www.php.net/manual/en/language.oop5.autoload.php
- PHP Manual: Preloading — https://www.php.net/manual/en/opcache.preloading.php
- Yii Framework: Class Autoloading — https://yiisoft.github.io/docs/es/guide/concept/autoloading.html 
- Packagist: piotrpress/composer-classmapper — https://packagist.org/packages/piotrpress/composer-classmapper 
- Stack Overflow: Composer dump-autoload difference — https://stackoverflow.com/questions/21247314/composer-update-vs-composer-dump-autoload 