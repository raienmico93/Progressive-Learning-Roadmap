# PHP Namespaces — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Namespaces are a language construct that encapsulates and organizes related classes, interfaces, functions, and constants under a hierarchical, directory-like naming structure, preventing naming collisions and enabling clean, readable code organization.

**Technical Definition**  
A namespace in PHP is a container for a set of related definitions. It is declared using the `namespace` keyword, and it affects four types of code constructs: classes (including abstract classes, traits, and enums), interfaces, functions, and constants. The namespace hierarchy uses a backslash (`\`) as the separator, mirroring filesystem directory structures. Namespace names are case-insensitive, and the namespace `PHP` (and any compound names starting with it) is reserved for internal language use. PHP 5.3.0 introduced namespaces, and they have been refined through PHP 7 and PHP 8, with PHP 8 introducing a new tokenization process that treats namespaced names as single tokens.

**Beginner-Friendly Explanation**  
Imagine you have two files named `report.txt` on your computer — one in a folder called `Work` and another in a folder called `Personal`. Because they are in different folders, you can tell them apart: `Work/report.txt` and `Personal/report.txt`. PHP namespaces work the same way for your code. They let you have a `Logger` class in `App\Http\Logger` and another `Logger` class in `App\Database\Logger` without them conflicting. This makes your code organized, readable, and safe from collisions with other libraries.

---

### Key Characteristics

- **Introduced in PHP 5.3.0**, with refinements in PHP 7 and PHP 8.
- **Only four code types are affected**: classes (including abstract classes, traits, and enums), interfaces, functions, and constants.
- **Declared with the `namespace` keyword** at the top of a file, before any other code except `declare`.
- **Case-insensitive** namespace names, but the convention is to use `StudlyCase` for namespace segments.
- **Reserved namespaces**: `PHP` and any compound names starting with `PHP` are reserved for internal language use.
- **PSR-4 autoloading standard**: namespace prefixes map directly to directory structures, enabling automatic class loading.
- **Global namespace fallback**: functions and constants fall back to the global namespace if not found in the current namespace, but classes do not.
- **PHP 8 tokenization change**: namespaced names are treated as single tokens, allowing reserved keywords to be used in namespace segments without conflict.
- **One namespace per file is the modern convention** (multiple namespaces per file are allowed with bracketed syntax, but discouraged).
- **The `use` keyword imports fully-qualified names** so they can be referenced by short names.

---

### Prerequisites

- Basic PHP syntax: variables, functions, classes.
- Object-oriented PHP: classes, interfaces, traits, enums.
- Basic understanding of filesystem directory structures.
- Composer (for PSR-4 autoloading in real-world projects).

---

### Related Programming Areas

- **Autoloading and PSR standards** (PSR-0, PSR-4).
- **Composer package management**.
- **Object-oriented design and dependency injection**.
- **Framework architecture** (Laravel, Symfony, WordPress).
- **PHP 8 tokenization and language evolution**.

---

## Core Concepts / Features

## 1. Defining Namespaces

### Definitions

**Core Definition**  
A namespace is declared using the `namespace` keyword as the first statement in a PHP file (only `declare` may precede it), grouping related code under a hierarchical name.

**Technical Definition**  
A file containing a namespace must declare the namespace at the top of the file before any other code, with the sole exception of the `declare` keyword. The namespace declaration is a compile-time construct that affects how the interpreter resolves names for classes, interfaces, functions, and constants. Multiple files may share the same namespace, allowing a namespace's contents to be split across the filesystem. No non-PHP code (including whitespace) may precede a namespace declaration.

**Beginner-Friendly Explanation**  
Think of a namespace as a folder for your code. To create a folder, you write `namespace MyProject;` at the very top of your PHP file. Everything below that line belongs to the `MyProject` folder. If you have another file with the same `namespace MyProject;` line, PHP treats it as the same folder, even though they are separate files.

### Purposes

- To prevent name collisions between your code and third-party libraries.
- To organize related classes, interfaces, functions, and constants.
- To enable PSR-4 autoloading by mapping namespaces to directory structures.
- To improve code readability and maintainability.
- To communicate the architectural structure of an application.
- To allow the same class name to exist in different contexts.
- To support modular and scalable application design.

### Syntax Rules and Structure

**General Syntax: Single Namespace Declaration**
```php
<?php
namespace MyProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */ }
```

**Component Breakdown**

- `namespace MyProject;` — declares the namespace; must be the first statement.
- `const CONNECT_OK = 1;` — a constant in the `MyProject` namespace.
- `class Connection` — a class in the `MyProject` namespace.
- `function connect()` — a function in the `MyProject` namespace.

**General Syntax: Sub-Namespaces**
```php
<?php
namespace MyProject\Sub\Level;

class MyClass { /* ... */ }
```

**Component Breakdown**

- `MyProject\Sub\Level` — a hierarchical namespace.
- The backslash separates namespace levels.
- Mirrors a directory structure: `MyProject/Sub/Level/`.

**General Syntax: Multiple Namespaces in One File (Bracketed)**
```php
<?php
namespace MyProject {
    class MyClass { /* ... */ }
}

namespace AnotherProject {
    class MyClass { /* ... */ }
}
```

**Component Breakdown**

- Bracketed syntax allows multiple namespaces per file.
- Global non-namespaced code must use the `{ }` bracket syntax.
- Discouraged in modern PHP; one namespace per file is preferred.

**General Syntax: The `__NAMESPACE__` Constant**
```php
<?php
namespace MyProject;

echo __NAMESPACE__; // outputs "MyProject"
```

**Component Breakdown**

- `__NAMESPACE__` — a magic constant containing the current namespace name.
- Returns an empty string in the global namespace.

**General Syntax: `namespace` Keyword as Operator**
```php
<?php
namespace MyProject;

$a = namespace\MYCONST; // accesses MyProject\MYCONST
```

**Component Breakdown**

- `namespace\` — an operator that explicitly accesses the current namespace.
- Equivalent to `self` in classes.

**Syntax Rules**

- The `namespace` keyword must be the first statement in the file (except `declare`).
- No non-PHP code may precede the declaration, including whitespace.
- Namespace names are case-insensitive but conventionally use `StudlyCase`.
- Namespaces cannot start with a number; use a prefix like `v1` instead.
- Fully qualified names (starting with `\`) are not allowed in namespace declarations.
- The `PHP` namespace and its sub-namespaces are reserved.
- `define()` with a namespaced constant requires the full name or `__NAMESPACE__`.

**Constraints and Limitations**

- A namespace cannot be nested inside another namespace declaration.
- Multiple namespaces per file require bracketed syntax and are discouraged.
- The `namespace` keyword is a compile-time construct; it cannot be conditional.
- Dynamic namespace names (e.g., from variables) are not supported directly.
- PHP 8 changed tokenization, which may break some edge cases with `new\class`.

### Annotated Code Examples and Expected Outputs

#### Example 1: Declaring and Using a Namespace

**File: `MyProject/Connection.php`**
```php
<?php
namespace MyProject;

const CONNECT_OK = 1;

class Connection {
    public function __construct() {
        echo "MyProject\Connection created.";
    }
}

function connect() {
    return "Connected to MyProject.";
}
?>
```

**File: `index.php`**
```php
<?php
require 'MyProject/Connection.php';

// Fully qualified name (from global namespace).
$conn = new \MyProject\Connection();

// Using the namespace operator.
echo \MyProject\connect();
?>
```

**Expected Output**
```
MyProject\Connection created.Connected to MyProject.
```

**Why This Output Occurs**

- `namespace MyProject;` declares the namespace.
- `new \MyProject\Connection()` uses the fully qualified name from the global namespace.
- `\MyProject\connect()` calls the namespaced function.

#### Example 2: Sub-Namespaces

**File: `App/Http/Logger.php`**
```php
<?php
namespace App\Http;

class Logger {
    public function log(string $message): void {
        echo "[App\Http\Logger] $message";
    }
}
?>
```

**File: `index.php`**
```php
<?php
require 'App/Http/Logger.php';

$logger = new \App\Http\Logger();
$logger->log('Hello, namespaces!');
?>
```

**Expected Output**
```
[App\Http\Logger] Hello, namespaces!
```

**Why This Output Occurs**

- `App\Http` is a sub-namespace of `App`.
- The class is instantiated using its fully qualified name.
- The method outputs the class name and message.

### Real-World Cases

- **Laravel:** Uses `App\Http\Controllers`, `App\Models`, etc.
- **Symfony:** Uses `App\Kernel`, `App\Controller\`.
- **WordPress plugins:** Use namespaces like `Advanced_Multi_Block\` with PSR-4 autoloading.
- **Composer packages:** Use vendor-prefixed namespaces like `Monolog\Logger`.

---

## 2. Aliasing and Importing with `use`

### Definitions

**Core Definition**  
The `use` operator imports a fully-qualified name into the current namespace, allowing it to be referenced by a short alias.

**Technical Definition**  
PHP can alias or import constants, functions, classes, interfaces, traits, enums, and namespaces using the `use` operator. Importing is performed at compile-time and affects only unqualified and qualified names, not fully qualified names. The `use` keyword must be declared in the outermost scope of a file or inside a namespace declaration. Importing rules are per-file; included files do not inherit the parent file's importing rules. PHP additionally supports group use statements for importing multiple names in one line.

**Beginner-Friendly Explanation**  
If you have a class with a very long name like `My\Full\Namespace\VeryLongClassName`, typing it every time is annoying. The `use` statement lets you give it a short nickname: `use My\Full\Namespace\VeryLongClassName as ShortName;`. Now you can just write `new ShortName()` instead of the full path. This keeps your code clean and readable.

### Purposes

- To shorten long fully-qualified names for better readability.
- To avoid name collisions by using aliases.
- To import functions and constants explicitly.
- To organize dependencies at the top of a file.
- To improve code maintainability and greppability.
- To enable clean dependency injection.
- To follow PSR coding standards.

### Syntax Rules and Structure

**General Syntax: Importing a Class**
```php
use My\Full\ClassName;
use My\Full\ClassName as Alias;
```

**Component Breakdown**

- `use My\Full\ClassName;` — imports the class for short-name usage.
- `use My\Full\ClassName as Alias;` — imports with a custom alias.

**General Syntax: Importing a Namespace**
```php
use My\Full\NSname;
```

**Component Breakdown**

- Imports the namespace so sub-elements can be referenced via `NSname\Sub`.
- Does not import the namespace itself as a class.

**General Syntax: Importing Functions and Constants**
```php
use function My\Full\functionName;
use function My\Full\functionName as func;
use const My\Full\CONSTANT;
```

**Component Breakdown**

- `use function` — imports a function.
- `use const` — imports a constant.
- Aliases can be applied to both.

**General Syntax: Group Use Statements (PHP 7+)**
```php
use My\Full\Classname as Another, My\Full\NSname;
```

**Component Breakdown**

- Multiple imports on one line, comma-separated.
- Each import can have its own alias.

**General Syntax: Illegal Use (Block Scope)**
```php
<?php
namespace Languages;
function toGreenlandic() {
    use Languages\Danish; // ILLEGAL: use cannot be inside a function
}
?>
```

**Component Breakdown**

- `use` must be at the top-level of the file or inside a namespace declaration block.
- It cannot be used inside a function or method.

**Syntax Rules**

- `use` must be declared at the outermost scope of a file or inside a namespace declaration.
- Importing is compile-time and does not affect dynamic names.
- Fully qualified names (starting with `\`) are not affected by imports.
- Importing rules are per-file; they are not inherited by included files.
- A leading backslash is unnecessary and not recommended in `use` statements.
- The `use` keyword can import classes, interfaces, traits, enums, functions, constants, and namespaces.
- Group use statements are available in PHP 7.0+.

**Constraints and Limitations**

- `use` cannot be used inside functions or methods.
- Importing does not work for dynamic class names (e.g., `$className`).
- Importing rules do not cascade to included files.
- Importing a function or constant does not automatically import the namespace.
- Name conflicts between imported names and local declarations cause fatal errors.

### Annotated Code Examples and Expected Outputs

#### Example 1: Importing Classes with Aliases

**File: `App/Service/UserService.php`**
```php
<?php
namespace App\Service;

use App\Http\Logger;
use App\Http\Client as HttpClient;
use function App\Support\slugify;
use const App\Support\VERSION;

class UserService {
    public function __construct(private Logger $logger) {}

    public function create(string $name): void {
        $slug = slugify($name);
        $this->logger->log("Creating user: $slug (v" . VERSION . ")");
    }
}
?>
```

**File: `index.php`**
```php
<?php
require 'App/Http/Logger.php';
require 'App/Http/Client.php';
require 'App/Support/functions.php';
require 'App/Service/UserService.php';

$logger = new \App\Http\Logger();
$service = new \App\Service\UserService($logger);

$service->create('Alice Smith');
?>
```

**Expected Output**
```
Creating user: alice-smith (v1.0.0)
```

**Why This Output Occurs**

- `use App\Http\Logger;` imports the `Logger` class.
- `use function App\Support\slugify;` imports the `slugify` function.
- `use const App\Support\VERSION;` imports the `VERSION` constant.
- The `UserService` class uses the imported names by their short forms.

#### Example 2: Importing a Global Class

**File: `App/Persistence/Database.php`**
```php
<?php
namespace App\Persistence;

use PDO;

final class Database {
    public function connect(): PDO {
        return new PDO('sqlite:app.db');
    }
}
?>
```

**Expected Output**
```
(PDO connection created successfully)
```

**Why This Output Occurs**

- `use PDO;` imports the global `PDO` class into the `App\Persistence` namespace.
- Without the import, `new PDO()` would look for `App\Persistence\PDO` and fail.
- The imported short name `PDO` refers to the global class.

### Real-World Cases

- **Framework controllers:** `use App\Http\Controllers\Controller;`.
- **Composer packages:** `use Monolog\Logger;` and `use Monolog\Handler\StreamHandler;`.
- **WordPress plugins:** Importing plugin classes with `use Advanced_Multi_Block\Functions\...`.
- **Dependency injection:** Importing interfaces for type-hinting.

---

## 3. Name Resolution Rules

### Definitions

**Core Definition**  
Name resolution rules determine how PHP interprets a name — whether it is fully qualified, qualified, or unqualified — and which namespace it belongs to.

**Technical Definition**  
PHP distinguishes three types of names: **fully qualified names** (starting with `\`) resolve from the global root; **qualified names** (containing `\` but not starting with `\`) are resolved relative to the current namespace or a `use` import; **unqualified names** (no `\`) are resolved against the current namespace and import tables. For classes, there is no fallback to the global namespace; for functions and constants, PHP falls back to the global namespace if the name is not found in the current namespace.

**Beginner-Friendly Explanation**  
When you write a name in PHP, it can be interpreted in three ways. A name starting with `\` is an absolute address, like a full file path: `\PDO` always means the global `PDO` class. A name with `\` in the middle is like a relative path: `App\Http\Logger` means “look in the current namespace for `App`, then `Http`, then `Logger`.” A simple name like `Logger` is like a filename in the current folder: PHP looks in the current namespace, then checks any `use` imports, and for functions and constants, it also checks the global namespace.

### Purposes

- To understand how PHP resolves names in different contexts.
- To avoid unexpected “class not found” errors.
- To correctly use global classes and functions inside namespaces.
- To write predictable, maintainable code.
- To debug namespace-related issues.
- To follow best practices for explicit naming.

### Syntax Rules and Structure

**General Syntax: Fully Qualified Name**
```php
\My\Full\ClassName
\strlen('hello')
\PHP_VERSION
```

**Component Breakdown**

- Starts with `\` — resolved from the global root.
- Ignores the current namespace and any imports.
- Always works, regardless of context.

**General Syntax: Qualified Name**
```php
Sub\ClassName
My\Full\ClassName
```

**Component Breakdown**

- Contains `\` but does not start with `\`.
- First segment is resolved against the current namespace and import tables.
- Remaining segments are appended.

**General Syntax: Unqualified Name**
```php
ClassName
strlen
MYCONST
```

**Component Breakdown**

- No `\` — resolved against the current namespace.
- For classes: no fallback to global namespace.
- For functions/constants: falls back to global namespace if not found locally.

**General Syntax: The `namespace` Operator**
```php
namespace\MyClass
namespace\MYCONST
```

**Component Breakdown**

- Explicitly accesses the current namespace.
- Equivalent to writing the full namespace name.

**Name Resolution Summary Table**

| Name Type | Example | Resolution |
|---|---|---|
| Fully qualified | `\App\Logger` | Global root |
| Qualified | `App\Logger` | Current namespace + `App\Logger` |
| Unqualified (class) | `Logger` | Current namespace, then imports; no global fallback |
| Unqualified (function) | `strlen` | Current namespace, then global fallback |
| Unqualified (constant) | `PHP_VERSION` | Current namespace, then global fallback |
| `namespace\` operator | `namespace\MyClass` | Current namespace |

**Syntax Rules**

- Fully qualified names always resolve from the global root.
- Qualified names are resolved relative to the current namespace unless imported.
- Unqualified class names do not fall back to the global namespace.
- Unqualified function and constant names do fall back to the global namespace.
- The `namespace\` operator explicitly accesses the current namespace.
- Importing affects only unqualified and qualified names, not fully qualified names.

**Constraints and Limitations**

- Class names inside a namespace require `\` or `use` to access global classes.
- Function and constant fallback can cause confusion; explicit `use function` is recommended.
- The `namespace\` operator cannot be used with dynamic names.
- PHP 8’s tokenization change affects how namespaced names are parsed.

### Annotated Code Examples and Expected Outputs

#### Example 1: Accessing Global Classes Inside a Namespace

**File: `App/Logger.php`**
```php
<?php
namespace App;

// Option 1: Use the fully qualified name.
$pdo1 = new \PDO('sqlite::memory:');

// Option 2: Import the global class.
use PDO;
$pdo2 = new PDO('sqlite::memory:');

echo "Both connections created.";
?>
```

**Expected Output**
```
Both connections created.
```

**Why This Output Occurs**

- `\PDO` uses the fully qualified name to access the global `PDO` class.
- `use PDO;` imports the global class, allowing `new PDO()`.
- Without either approach, PHP would look for `App\PDO` and fail.

#### Example 2: Function Fallback to Global Namespace

**File: `App/Helpers.php`**
```php
<?php
namespace App;

// strlen() is not defined in App, so PHP falls back to the global strlen().
$length = strlen('hello');
echo "Length: $length";

// But if you define your own strlen in App, it takes precedence.
function strlen($s) {
    return 999;
}
echo "App strlen: " . strlen('hello');
?>
```

**Expected Output**
```
Length: 5App strlen: 999
```

**Why This Output Occurs**

- The first `strlen('hello')` call falls back to the global `strlen()`.
- After defining `App\strlen()`, the unqualified call resolves to the local function.
- This demonstrates the function fallback behavior.

### Real-World Cases

- **Global classes in namespaced code:** Use `\PDO`, `\Exception`, `\DateTimeImmutable`, or import them with `use`.
- **Global functions:** `\strlen`, `\array_map`, etc., or import with `use function`.
- **Framework code:** Typically imports global classes explicitly at the top of files.
- **Legacy code migration:** Understanding fallback rules helps when converting code to namespaces.

---

## 4. PSR-4 Autoloading with Namespaces

### Definitions

**Core Definition**  
PSR-4 is a PHP Standard Recommendation that defines a specification for autoloading classes from file paths based on namespace prefixes.

**Technical Definition**  
PSR-4 maps a namespace prefix to a base directory. When a class is referenced, the autoloader strips the namespace prefix, converts the remaining namespace segments to directory separators, appends `.php`, and includes the file from the base directory. Composer generates a PSR-4 autoloader from the `autoload.psr-4` section of `composer.json`. This eliminates the need for manual `require` statements and keeps the file structure aligned with the namespace hierarchy.

**Beginner-Friendly Explanation**  
PSR-4 is a rule that says: “If your class is in the `App\Http` namespace, then the file should be in the `src/Http/` folder.” When PHP needs a class, the autoloader figures out the file path from the namespace and loads it automatically. Composer does this for you — you just tell it which namespace maps to which folder, and it handles the rest. No more manual `require` statements.

### Purposes

- To eliminate manual `require`/`include` statements.
- To keep file structure aligned with namespace hierarchy.
- To enable interoperability between libraries and frameworks.
- To improve performance by loading classes only when needed.
- To follow industry-standard PHP practices.
- To simplify dependency management with Composer.
- To support large-scale application architecture.

### Syntax Rules and Structure

**General Syntax: `composer.json` PSR-4 Configuration**
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "Advanced_Multi_Block\\": "Functions/"
        }
    }
}
```

**Component Breakdown**

- `"App\\": "src/"` — maps the `App\` namespace prefix to the `src/` directory.
- `"Advanced_Multi_Block\\": "Functions/"` — maps another namespace prefix.
- The double backslash (`\\`) is JSON escaping for a single backslash.

**General Syntax: Autoloader Usage**
```php
<?php
require __DIR__ . '/vendor/autoload.php';

// Now classes are autoloaded.
$logger = new App\Http\Logger();
?>
```

**Component Breakdown**

- `require 'vendor/autoload.php'` — loads Composer's autoloader.
- Classes are loaded on demand based on the PSR-4 mapping.

**General Syntax: Directory Structure**
```
project/
├── composer.json
├── src/
│   ├── Http/
│   │   └── Logger.php      → App\Http\Logger
│   └── Database/
│       └── Connection.php  → App\Database\Connection
└── vendor/
    └── autoload.php
```

**Component Breakdown**

- `src/` is the base directory for the `App\` namespace.
- `src/Http/Logger.php` corresponds to `App\Http\Logger`.
- `src/Database/Connection.php` corresponds to `App\Database\Connection`.

**General Syntax: Generating the Autoloader**
```
composer dump-autoload
```

**Component Breakdown**

- `composer dump-autoload` — regenerates the autoloader after adding new classes or changing `composer.json`.
- `composer du` — shorthand for the same command.

**Syntax Rules**

- The namespace prefix must end with `\`.
- The base directory is relative to the `composer.json` location.
- The file path mirrors the namespace path after the prefix.
- The class name must match the filename (case-sensitive on Unix).
- Multiple namespace prefixes can map to different directories.
- Run `composer dump-autoload` after adding new classes or changing mappings.

**Constraints and Limitations**

- PSR-4 requires one class per file.
- File names must match class names exactly (case-sensitive).
- The autoloader does not work for functions or constants.
- Performance overhead is minimal but non-zero.
- PSR-0 is the older standard; PSR-4 is preferred for new projects.

### Annotated Code Examples and Expected Outputs

#### Example 1: PSR-4 Autoloading Setup

**File: `composer.json`**
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**File: `src/Http/Logger.php`**
```php
<?php
namespace App\Http;

class Logger {
    public function log(string $message): void {
        echo "[Logger] $message";
    }
}
?>
```

**File: `index.php`**
```php
<?php
require __DIR__ . '/vendor/autoload.php';

$logger = new App\Http\Logger();
$logger->log('PSR-4 autoloading works!');
?>
```

**Expected Output**
```
[Logger] PSR-4 autoloading works!
```

**Why This Output Occurs**

- `composer.json` maps `App\` to `src/`.
- The class `App\Http\Logger` is loaded from `src/Http/Logger.php`.
- The autoloader is required once, and classes load on demand.

#### Example 2: WordPress Plugin with PSR-4

**File: `composer.json`**
```json
{
    "autoload": {
        "psr-4": {
            "Advanced_Multi_Block\\": "Functions/"
        }
    }
}
```

**File: `Functions/Plugin.php`**
```php
<?php
namespace Advanced_Multi_Block;

class Plugin {
    public function init(): void {
        echo "Plugin initialized.";
    }
}
?>
```

**File: `plugin.php`**
```php
<?php
require __DIR__ . '/vendor/autoload.php';

$plugin = new Advanced_Multi_Block\Plugin();
$plugin->init();
?>
```

**Expected Output**
```
Plugin initialized.
```

**Why This Output Occurs**

- The `Advanced_Multi_Block\` namespace maps to the `Functions/` directory.
- The `Plugin` class is autoloaded from `Functions/Plugin.php`.
- The plugin initializes without manual `require` statements.

### Real-World Cases

- **Laravel:** `App\` maps to `app/` directory.
- **Symfony:** `App\` maps to `src/` directory.
- **WordPress plugins:** Custom namespace prefixes map to plugin directories.
- **Composer packages:** Each package defines its own PSR-4 mappings.

---

## 5. PHP 8 Namespace Changes

### Definitions

**Core Definition**  
PHP 8 introduced a new tokenization process for namespaced names, treating them as single tokens instead of multiple tokens, which resolves several parsing ambiguities and enables new syntax.

**Technical Definition**  
Before PHP 8, namespaced names like `Foo\Bar` were tokenized as separate tokens (`T_STRING`, `T_NS_SEPARATOR`, `T_STRING`), causing inconsistent parsing in contexts like `new\class` and `instanceof`. PHP 8’s “Namespaced Names as Single Token” RFC changed this so that namespaced names are treated as a single token, allowing reserved keywords to be used within namespace segments without conflict. This also exposes bad naming practices such as whitespace in namespaces.

**Beginner-Friendly Explanation**  
Before PHP 8, PHP read namespaced names character by character, which sometimes caused confusing errors. PHP 8 reads them as whole units, like reading a word instead of individual letters. This makes the parser simpler and allows you to use words like `new` or `class` as parts of namespace names without triggering errors. It also catches mistakes like putting spaces in namespace names.

### Purposes

- To simplify the PHP parser and tokenizer.
- To resolve parsing ambiguities with `new\class` and similar constructs.
- To allow reserved keywords in namespace segments.
- To expose bad naming practices (e.g., whitespace in namespaces).
- To improve forward compatibility of namespace code.
- To align with modern language design principles.

### Syntax Rules and Structure

**General Syntax: Namespaced Names as Single Token**
```php
<?php
namespace App\New;

class MyClass {
    public function test(): void {
        // Before PHP 8, "new\class" could be ambiguous.
        // In PHP 8, it is a single token.
        $obj = new \App\New\MyClass();
    }
}
?>
```

**Component Breakdown**

- `App\New` — `New` is a reserved keyword but can be used as a namespace segment in PHP 8.
- The entire `\App\New\MyClass` is treated as a single token.
- Parsing is unambiguous.

**General Syntax: Whitespace in Namespaces (Bad Practice)**
```php
<?php
namespace My \ Project; // PHP 8 exposes this as bad naming.
?>
```

**Component Breakdown**

- Whitespace around `\` is invalid or discouraged.
- PHP 8’s tokenization exposes this as a naming error.

**General Syntax: Reserved Keywords in Namespaces**
```php
<?php
namespace App\List\Array; // Valid in PHP 8.
?>
```

**Component Breakdown**

- `List` and `Array` are reserved keywords but can be namespace segments in PHP 8.
- The tokenizer treats them as part of the namespaced name.

**Syntax Rules**

- Namespaced names are single tokens in PHP 8.
- Reserved keywords can be used as namespace segments.
- Whitespace in namespace names is exposed as bad practice.
- The tokenization change may cause backwards-compatible breaks in edge cases.
- The RFC “Namespaced Names as Single Token” was implemented in PHP 8.0.

**Constraints and Limitations**

- Some existing code that relied on the old tokenization may break.
- `new\class` now behaves differently.
- Whitespace in namespaces is no longer silently accepted.
- The change is not backwards-compatible with PHP 7.

### Annotated Code Examples and Expected Outputs

#### Example 1: Reserved Keywords in Namespaces

**File: `App/List/Array/Collection.php`**
```php
<?php
namespace App\List\Array;

class Collection {
    public function __construct() {
        echo "Collection created in App\List\Array namespace.";
    }
}
?>
```

**File: `index.php`**
```php
<?php
require 'App/List/Array/Collection.php';

$collection = new \App\List\Array\Collection();
?>
```

**Expected Output**
```
Collection created in App\List\Array namespace.
```

**Why This Output Occurs**

- `List` and `Array` are reserved keywords, but PHP 8 allows them as namespace segments.
- The namespaced name is treated as a single token.

#### Example 2: `new\class` Parsing Change

**File: `test.php`**
```php
<?php
namespace App;

// In PHP 7, this could be parsed inconsistently.
// In PHP 8, "new\class" is a single token and fails with a clear error.
$obj = new\class();
?>
```

**Expected Output (PHP 8)**
```
Fatal error: Uncaught Error: Class "App\class" not found
```

**Why This Output Occurs**

- In PHP 8, `new\class` is parsed as a single token, so PHP looks for `App\class`.
- In PHP 7, it might have been parsed differently, causing confusing behavior.
- The error message is now clearer.

### Real-World Cases

- **Legacy code migration:** Understanding the tokenization change helps with PHP 7 to PHP 8 upgrades.
- **Framework code:** Modern frameworks already use PHP 8-compatible namespace syntax.
- **Library development:** Ensures compatibility with PHP 8’s stricter parsing.
- **Static analysis tools:** Updated to handle the new tokenization.

---

## Security Considerations

### Namespace Isolation

- **Autoloader boundaries:** PSR-4 autoloading limits code exposure by only loading classes within defined namespace prefixes.
- **PHP-Scoper:** A tool that prefixes all PHP namespaces in a file/directory to isolate code bundled in PHARs.
- **Global namespace pollution:** Avoid placing many variables, functions, or classes in the global namespace to prevent naming conflicts with third-party code.

### Namespace Prefixing for Security

- **Purpose:** Prefixing namespaces isolates your code from other libraries, preventing conflicts and reducing the attack surface.
- **Implementation:** Use a unique vendor prefix (e.g., `Acme\`) for all your code.
- **Tools:** PHP-Scoper automates the prefixing process for PHAR packaging.

### Best Practices

- **One class per file:** Keeps autoloading predictable and code organized.
- **Namespace mirrors directory:** Follow PSR-4 conventions for consistency.
- **Use `StudlyCase` for namespace segments:** e.g., `App\Http\Middleware`.
- **Import at the top:** Keep `use` statements at the top of the file for visibility and greppability.
- **Use `declare(strict_types=1)`:** Enable strict typing for better security and code quality.
- **Avoid deep nesting:** Keep namespace hierarchies shallow for readability.
- **Explicitly import global classes:** Use `use PDO;` instead of `\PDO` for readability.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Notes |
|---|---|---|
| Multiple namespaces per file | Discouraged | Use one namespace per file |
| Global namespace pollution | Discouraged | Use namespaces for all code |
| `PHP` namespace | Reserved | Cannot be used in userland code |
| Whitespace in namespaces | Bad practice | Exposed by PHP 8 tokenization |
| `new\class` parsing | Changed in PHP 8 | May break legacy code |

### References Links

- PHP: Namespaces — https://www.php.net/manual/en/language.namespaces.php
- PHP: Namespaces Overview — https://www.php.net/manual/en/language.namespaces.rationale.php
- PHP: Defining Namespaces — https://www.php.net/manual/en/language.namespaces.definition.php
- PHP: Aliasing/Importing — https://www.php.net/manual/en/language.namespaces.importing.php
- PHP: Name Resolution Rules — https://www.php.net/manual/en/language.namespaces.rules.php
- PHP: Namespace Fallback — https://www.php.net/manual/en/language.namespaces.fallback.php
- PHP: `namespace` Keyword — https://www.php.net/manual/en/language.namespaces.nsconstants.php
- PHP: `__NAMESPACE__` — https://www.php.net/manual/en/language.namespaces.nsconstants.php
- PHP RFC: Namespaced Names as Single Token — https://wiki.php.net/rfc/namespaced_names_as_token
- PHP RFC: PHP Namespace Policy — https://wiki.php.net/rfc/php_namespace_policy
- PHP RFC: Namespace Resolution — https://wiki.php.net/rfc/namespaceresolution
- PSR-4: Autoloader — https://www.php-fig.org/psr/psr-4/
- Composer: Autoloading — https://getcomposer.org/doc/01-basic-usage.md#autoloading
- WordPress: Implementing Namespaces and Coding Standards — https://developer.wordpress.org/news/2025/09/implementing-namespaces-and-coding-standards-in-wordpress-plugin-development/
- Secure PHP Development: Namespaces — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Namespaces.md
- Educative: Understanding Namespace Changes in PHP 8 — https://www.educative.io/courses/php-8-programming-tips-tricks-and-best-practices/lta/understanding-changes-in-namespaces
- PHP-Scoper — https://github.com/humbug/php-scoper