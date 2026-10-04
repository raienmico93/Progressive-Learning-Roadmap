# Package Lifecycle, Scripting, & Automation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Package lifecycle management in Composer is the practice of controlling how dependencies are installed, which environments they target, and what automated actions are triggered at each stage of the install, update, or deployment process. It encompasses three complementary mechanisms: environment separation via `--dev`/`--no-dev`, event-driven scripting via `composer.json` hooks, and executable binary orchestration via `vendor/bin`.

**Technical Definition**

Composer's lifecycle is governed by a finite state machine of named events fired during `install`, `update`, `dump-autoload`, and `create-project` commands. Scripts defined in the root package's `composer.json` under the `"scripts"` key are registered as callbacks to these events and executed synchronously during the process. Dependency separation is achieved through the `require-dev` section, which is excluded from installation when `--no-dev` is passed. Vendor binaries are command-line executables declared in a package's `"bin"` key and symlinked into the consuming project's `vendor/bin` directory, providing a stable invocation path regardless of the package's physical location in the dependency tree.

**Beginner-Friendly Explanation**

Think of Composer as a construction crew building a house (your application). **Development vs. production separation** is like deciding which tools go to the construction site (development) and which stay in the architect's office (production). **Scripts and hooks** are the crew's checklist: "After pouring the foundation (post-install-cmd), run the plumbing inspection." **Vendor binaries** are the power tools in the site's tool shed (`vendor/bin`), available by a short name no matter which aisle of the warehouse they came from.

### Key Characteristics

- **Environment-Specific Dependency Sets:** `require` holds runtime dependencies; `require-dev` holds tooling (PHPUnit, PHPStan, PHP-CS-Fixer) that must never reach production.
- **Event-Driven Automation:** Composer fires named events at precise points in its execution, allowing scripts to run before or after installation, updates, autoloader dumps, and package-level operations.
- **Root-Package Script Authority:** Only scripts defined in the root package's `composer.json` are executed; dependency scripts are ignored, preventing third-party packages from running arbitrary code during installation.
- **Binary Proxy Architecture:** Composer creates proxy files in `vendor/bin` that resolve the correct path to each package's binary, handling Windows/WSL compatibility and autoloader discovery via `$_composer_autoload_path`.
- **Composability:** Scripts can be PHP static methods, shell commands, or Symfony Console Command classes, and can reference each other using `@script-name` syntax.

### Prerequisites

- **PHP 8.1+** with Composer 2.x installed.
- **A project with a valid `composer.json`** containing at least one dependency and an `autoload` section.
- **Familiarity with CLI execution** (`composer install`, `composer update`, `composer dump-autoload`).
- **Understanding of environment variables** for conditional script behaviour (e.g., `COMPOSER_NO_DEV`).
- **A basic deployment pipeline** (CI/CD or manual) where production installation flags are applied.

### Related Programming Areas

- **Production Autoloading & Performance Optimization:** The `post-autoload-dump` event is where optimized classmaps are generated, directly impacting production boot time.
- **PHP Composer Foundations & Dependency Architecture:** Version constraints, `composer.lock`, and platform requirements determine what is installed before scripts run.
- **CI/CD Pipeline Engineering:** Deployment scripts rely on `--no-dev`, `post-install-cmd`, and `vendor/bin` invocations.
- **Testing and Static Analysis:** Developer tooling (PHPUnit, PHPStan, PHP-CS-Fixer) is installed via `require-dev` and invoked via `vendor/bin/`.

### Core Concepts / Features

1. **Development vs. Production Separation** — `--dev` and `--no-dev` flags, `require-dev`, and `COMPOSER_NO_DEV`.
2. **Composer Scripts & Hooks** — Event names (`post-install-cmd`, `pre-update-cmd`), script definition, and PHP callbacks.
3. **Custom Binaries Orchestrating** — The `bin` key, `vendor/bin` proxies, and cross-platform binary invocation.

---

## Core Concept 1: Development vs. Production Separation

### Definitions

**Core Definition**

Development vs. production separation is the practice of installing different sets of dependencies depending on the target environment, ensuring that developer tooling (test frameworks, static analysers, debuggers) never bloats or compromises production deployments.

**Technical Definition**

Composer distinguishes between runtime dependencies (`require`) and development dependencies (`require-dev`). When `composer install` or `composer update` is executed with the `--no-dev` flag, packages listed in `require-dev` are skipped, and the autoloader generation omits the `autoload-dev` rules. The `COMPOSER_NO_DEV` environment variable provides an alternative mechanism: setting it to `1` achieves the same effect without passing the flag on every command. This separation is critical because development packages often have larger dependency trees, introduce security surface area, and are unnecessary for runtime operation.

**Beginner-Friendly Explanation**

Imagine you are shipping a product to a customer. You include the product itself, its power cable, and a user manual. You do not include the factory's welding equipment, the engineer's testing rig, or the QA team's checklist. Development dependencies are the welding equipment — essential for building the product, irrelevant for using it. The `--no-dev` flag tells Composer to ship only the product and its power cable.

### Purposes

- To prevent development tooling from consuming production disk space, memory, and autoloader resources.
- To reduce the security attack surface by excluding test frameworks and debug tools from production environments.
- To accelerate production deployment by installing and autoloading fewer packages.
- To ensure that CI pipelines can install the full development toolchain while production deployments install only runtime dependencies.
- To enable reproducible environments where the same `composer.lock` yields different installations based on the environment flag.

### Syntax Rules and Structure

**Complete General Syntax: Separating Dependencies in `composer.json`**

```json
{
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.5",
        "guzzlehttp/guzzle": "^7.8"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0",
        "phpstan/phpstan": "^1.11",
        "friendsofphp/php-cs-fixer": "^3.50"
    },
    "autoload": {
        "psr-4": { "App\\": "src/" }
    },
    "autoload-dev": {
        "psr-4": { "App\\Tests\\": "tests/" }
    }
}
```

**Complete General Syntax: Production Installation**

```bash
# Via CLI flag
composer install --no-dev

# Via environment variable
COMPOSER_NO_DEV=1 composer install

# Via configuration in composer.json (not recommended for development machines)
```

```json
{
    "config": {
        "no-dev": true
    }
}
```

**Component Breakdown:**

- `"require"` — Runtime dependencies. Always installed, regardless of `--no-dev`.
- `"require-dev"` — Development-only dependencies. Skipped when `--no-dev` is used.
- `"autoload-dev"` — Autoloading rules for development-only classes (e.g., test namespaces). Skipped when `--no-dev` is used.
- `--no-dev` — CLI flag that excludes `require-dev` packages from installation and omits `autoload-dev` rules from the generated autoloader.
- `COMPOSER_NO_DEV=1` — Environment variable that achieves the same effect as `--no-dev` without requiring the flag on every command.

**Syntax Rules:**

- `--no-dev` only affects the `install` and `update` commands. It does not uninstall already-installed development packages; it simply skips them during resolution and installation.
- The `autoload-dev` section is ignored when `--no-dev` is used, so test classes under `App\Tests\` are not autoloaded in production.
- `COMPOSER_NO_DEV` can be overridden by explicitly passing `--dev` (if supported by the Composer version) or by setting it to `0`.
- Development dependencies should never be required at runtime; if production code references a class from a `require-dev` package, the application will fail with a "class not found" error.

**Constraints and Limitations:**

- **Runtime dependency on dev packages:** If application code (not test code) accidentally depends on a package in `require-dev`, production will fail. Use static analysis to detect such leaks.
- **Lock file consistency:** The `composer.lock` file contains both runtime and development packages. Production installations with `--no-dev` still read the lock file but skip the `packages-dev` entries.
- **CI cache invalidation:** CI pipelines that cache `vendor/` must invalidate the cache when switching between `--dev` and `--no-dev` installations, because the two directories have different contents.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Comparing Development and Production Installations**

```bash
# Step 1: Create a project with both runtime and dev dependencies
mkdir -p env-separation-demo && cd env-separation-demo
composer init --name=acme/env-demo --no-interaction

# Step 2: Add a runtime dependency
composer require monolog/monolog

# Step 3: Add a development dependency
composer require --dev phpunit/phpunit

# Step 4: Inspect the vendor directory (development install)
ls vendor/ | head -20
echo "--- Development install: PHPUnit present ---"
ls vendor/bin/
```

**Expected Output:**

```
--- Development install: PHPUnit present ---
phpunit
```

```bash
# Step 5: Simulate production installation
rm -rf vendor/
composer install --no-dev --quiet

# Step 6: Inspect the vendor directory (production install)
echo "--- Production install: PHPUnit absent ---"
ls vendor/bin/ 2>/dev/null || echo "(vendor/bin is empty)"
```

**Expected Output:**

```
--- Production install: PHPUnit absent ---
(vendor/bin is empty)
```

**Why:** In the development installation, `phpunit/phpunit` is installed and its binary is exposed in `vendor/bin`. In the production installation with `--no-dev`, PHPUnit is excluded entirely — it is not in `vendor/`, and no binary is created in `vendor/bin`. This reduces the production artifact size and attack surface.

**Example 2: Using `COMPOSER_NO_DEV` in CI**

```bash
# Step 1: CI pipeline installs full toolchain for testing
composer install
vendor/bin/phpunit

# Step 2: After tests pass, build the production artifact
COMPOSER_NO_DEV=1 composer install --optimize-autoloader --classmap-authoritative

# Step 3: Verify no dev packages are present
composer show --no-dev
```

**Expected Output (Step 3):**

```
monolog/monolog 3.7.0 A logging library for PHP
psr/log          3.0.2 Common interface for logging libraries
```

**Why:** The first `composer install` installs all dependencies, including PHPUnit, so the test suite can run. The second `composer install` with `COMPOSER_NO_DEV=1` regenerates the vendor directory with only runtime dependencies, producing a lean production artifact. `composer show --no-dev` confirms that only runtime packages are listed.

### Real-World Cases

**Case 1: Docker Multi-Stage Build**

A Dockerfile uses a two-stage build. The first stage installs all dependencies (`composer install`) and runs tests. The second stage copies only the application source and runs `composer install --no-dev --optimize-autoloader`, producing a minimal production image. This reduces the final image size from ~500 MB to ~80 MB.

**Case 2: Laravel Deployment**

Laravel's deployment scripts run `composer install --no-dev --optimize-autoloader` on production servers. Development tools such as Laravel Debugbar, Laravel Telescope, and PHPUnit are in `require-dev` and are never installed in production. This prevents debug information from leaking to end users and eliminates the performance overhead of these packages.

**Case 3: Shared Hosting Constraints**

A project deployed on shared hosting has a strict disk quota. The hosting provider's deployment hook runs `COMPOSER_NO_DEV=1 composer install`. Development dependencies are excluded, and the vendor directory remains within the quota.

---

## Core Concept 2: Composer Scripts & Hooks

### Definitions

**Core Definition**

Composer scripts are callbacks — PHP static methods, shell commands, or Symfony Console Command classes — that are automatically executed at specific points during Composer's lifecycle, such as before or after installation, updates, or autoloader dumps.

**Technical Definition**

Scripts are defined in the root package's `composer.json` under the `"scripts"` key, which maps named events to one or more executable commands or PHP callbacks. Composer fires a predefined set of events during `install`, `update`, `dump-autoload`, `create-project`, and other commands. Only scripts defined in the root package are executed; dependency scripts are ignored. As of Composer 2.5, scripts can also be Symfony Console Command classes, allowing options to be passed, though this is not recommended for event handling.

**Beginner-Friendly Explanation**

Think of Composer scripts as the "if this, then that" rules of your dependency lifecycle. "After installing dependencies, clear the application cache." "Before updating, run a database backup." "After dumping the autoloader, warm the route cache." You write these rules once in `composer.json`, and Composer follows them automatically every time the corresponding command runs.

### Purposes

- To automate repetitive setup tasks that must run after dependency installation (cache clearing, asset compilation, database migration).
- To enforce pre-flight checks before updates (backup, compatibility validation).
- To integrate Composer into larger build pipelines without separate orchestration scripts.
- To centralise environment-specific automation in a single, version-controlled file.
- To provide a standardised way for frameworks (Laravel, Symfony, Drupal) to hook into Composer's lifecycle.

### Syntax Rules and Structure

**Complete General Syntax: Script Definition**

```json
{
    "scripts": {
        "post-install-cmd": [
            "@php artisan cache:clear",
            "@php artisan config:cache"
        ],
        "pre-update-cmd": [
            "@php artisan down --render=\"errors::503\""
        ],
        "post-update-cmd": [
            "@php artisan up",
            "@php artisan migrate --force"
        ],
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump"
        ],
        "test": [
            "vendor/bin/phpunit"
        ]
    }
}
```

**Component Breakdown:**

- `"post-install-cmd"` — The event name. Occurs after the `install` command has been executed with a lock file present.
- `"pre-update-cmd"` — Occurs before the `update` command is executed, or before the `install` command when no lock file is present.
- `"post-autoload-dump"` — Occurs after the autoloader has been dumped, either during `install`/`update` or via `dump-autoload`.
- `"@php"` — A special Composer placeholder that resolves to the current PHP interpreter, ensuring the script uses the same PHP binary as Composer.
- `"@script-name"` — References another named script defined in the same `scripts` block, allowing script composition.
- `"Illuminate\\Foundation\\ComposerScripts::postAutoloadDump"` — A PHP static method callback. The class and method are resolved via the autoloader.
- `"vendor/bin/phpunit"` — A shell command that invokes a vendor binary directly.

**Complete Event Reference (Command Events):**

| Event | Trigger |
|---|---|
| `pre-install-cmd` | Before `install` with a lock file present |
| `post-install-cmd` | After `install` with a lock file present |
| `pre-update-cmd` | Before `update`, or before `install` without a lock file |
| `post-update-cmd` | After `update`, or after `install` without a lock file |
| `pre-autoload-dump` | Before the autoloader is dumped |
| `post-autoload-dump` | After the autoloader is dumped |
| `post-root-package-install` | After the root package is installed during `create-project` |
| `post-create-project-cmd` | After `create-project` completes |

**Complete Event Reference (Package Events):**

| Event | Trigger |
|---|---|
| `pre-package-install` | Before a package is installed |
| `post-package-install` | After a package is installed |
| `pre-package-update` | Before a package is updated |
| `post-package-update` | After a package is updated |
| `pre-package-uninstall` | Before a package is uninstalled |
| `post-package-uninstall` | After a package is uninstalled |

**Syntax Rules:**

- Scripts are executed in the order they appear in the array for a given event.
- The `@php` placeholder ensures the script runs with the same PHP interpreter that Composer uses.
- Scripts can reference each other using `@script-name` syntax, enabling composition.
- Only the root package's scripts are executed. Dependencies' scripts are ignored.
- Scripts requiring Composer-managed dependencies should not be placed in `pre-update-cmd` or `pre-install-cmd` because the dependencies may not yet be installed at that point.

**Constraints and Limitations:**

- **No dependency scripts:** Third-party packages cannot execute scripts during installation of a consuming project. This is a security feature, not a limitation.
- **Pre-event autoloader unavailability:** Scripts in `pre-install-cmd` and `pre-update-cmd` cannot rely on classes from dependencies that have not yet been installed. Use `post-*` events for dependency-dependent logic.
- **Process timeout:** Composer enforces a default 300-second timeout on script execution. Long-running scripts (e.g., full database migrations) must either complete within this window or have their timeout disabled using `Composer\Config::disableProcessTimeout`.
- **No interactive input:** Scripts should not prompt for user input, as they run non-interactively in CI environments. Use `--no-interaction` compatible commands.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Framework-Style Script Hooks**

```json
{
    "name": "acme/framework-app",
    "scripts": {
        "post-install-cmd": [
            "@php artisan key:generate --force",
            "@php artisan storage:link",
            "@php artisan optimize:clear"
        ],
        "post-update-cmd": [
            "@php artisan migrate --force",
            "@php artisan optimize"
        ],
        "post-autoload-dump": [
            "@php artisan package:discover --ansi"
        ]
    }
}
```

```bash
# Step 1: Run composer install
composer install --no-dev --optimize-autoloader

# Expected: The scripts run automatically after installation
```

**Expected Output (excerpt):**

```
Generating optimized autoload files
> @php artisan package:discover --ansi
Discovered Package: laravel/sanctum
Discovered Package: laravel/tinker
> @php artisan key:generate --force
Application key set successfully.
> @php artisan storage:link
The [public/storage] directory has been linked.
> @php artisan optimize:clear
Compiled views cleared successfully.
Application cache cleared successfully.
```

**Why:** The `post-autoload-dump` script runs first because autoloader generation occurs before the `post-install-cmd` scripts. The `package:discover` command registers service providers. Then `key:generate`, `storage:link`, and `optimize:clear` run as part of `post-install-cmd`. Each script is a shell command prefixed with `@php` to ensure the correct PHP interpreter is used.

**Example 2: PHP Callback Script**

```json
{
    "scripts": {
        "post-install-cmd": [
            "Acme\\Composer\\SetupScript::postInstall"
        ]
    }
}
```

```php
<?php
// src/Composer/SetupScript.php
namespace Acme\Composer;

use Composer\Script\Event;

class SetupScript
{
    public static function postInstall(Event $event): void
    {
        $io = $event->getIO();
        $io->write('<info>Running custom post-install setup...</info>');

        // Example: create a required directory
        $dir = __DIR__ . '/../../var/cache';
        if (!is_dir($dir)) {
            mkdir($dir, 0775, true);
            $io->write("Created directory: $dir");
        }

        // Example: copy a default config file if absent
        $configFile = __DIR__ . '/../../config/local.php';
        if (!file_exists($configFile)) {
            copy(__DIR__ . '/../../config/local.php.dist', $configFile);
            $io->write('Created default local configuration.');
        }
    }
}
```

```bash
# Step 1: Run composer install
composer install
```

**Expected Output:**

```
> Acme\Composer\SetupScript::postInstall
Running custom post-install setup...
Created directory: /var/www/app/var/cache
Created default local configuration.
```

**Why:** The script is defined as a PHP static method and receives a `Composer\Script\Event` object, which provides access to the IO interface and Composer's configuration. This pattern is used by frameworks like Drupal to perform environment-specific setup after installation.

**Example 3: Custom Runnable Script**

```json
{
    "scripts": {
        "test": "vendor/bin/phpunit",
        "lint": "vendor/bin/php-cs-fixer fix --dry-run --diff",
        "analyse": "vendor/bin/phpstan analyse src --level=max",
        "ci": [
            "@lint",
            "@analyse",
            "@test"
        ]
    }
}
```

```bash
# Step 1: Run the custom CI script
composer ci
```

**Expected Output:**

```
> @lint
> vendor/bin/php-cs-fixer fix --dry-run --diff
... (linting output) ...
> @analyse
> vendor/bin/phpstan analyse src --level=max
... (analysis output) ...
> @test
> vendor/bin/phpunit
... (test output) ...
```

**Why:** The `ci` script composes three other scripts using `@script-name` references. Running `composer ci` executes linting, static analysis, and tests in sequence. This provides a single, memorable command for the entire CI workflow, defined entirely in `composer.json`.

### Real-World Cases

**Case 1: Laravel Package Discovery**

Laravel's `post-autoload-dump` script calls `Illuminate\Foundation\ComposerScripts::postAutoloadDump`, which triggers `php artisan package:discover`. This scans installed packages for service providers and registers them automatically, eliminating manual registration in `config/app.php`.

**Case 2: Symfony Flex Recipes**

Symfony Flex hooks into `post-install-cmd` and `post-update-cmd` to apply "recipes" — automated configuration changes for newly installed packages. When a developer runs `composer require symfony/mailer`, Flex's script copies configuration files, updates `.env`, and registers bundles without manual intervention.

**Case 3: Drupal Scaffold**

Drupal's `drupal-scaffold` plugin uses scripts to manage scaffolding files (`.htaccess`, `robots.txt`, `index.php`) in the project root. The `post-install-cmd` and `post-update-cmd` events trigger the scaffold update, ensuring that these files match the installed Drupal version.

---

## Core Concept 3: Custom Binaries Orchestrating

### Definitions

**Core Definition**

Vendor binaries are command-line executables exposed by Composer packages and made available in the consuming project's `vendor/bin` directory through proxy files, providing a stable, cross-platform invocation path.

**Technical Definition**

A package declares its command-line executables in the `"bin"` key of its `composer.json`. When another project depends on that package, Composer creates proxy files in the consuming project's `vendor/bin` directory (the `bin-dir`). These proxies resolve the correct path to the original binary within `vendor/vendor-name/package-name/bin/` and execute it. On Unix-like systems, the proxy is a symlink or a shell script; on Windows, Composer creates `.bat` and `.php` proxy files to handle platform differences. The `bin-dir` location is configurable via the `config.bin-dir` key in `composer.json`.

**Beginner-Friendly Explanation**

Imagine you install a package that comes with a command-line tool, like a screwdriver. Composer does not make you dig through the warehouse (`vendor/vendor-name/package-name/bin/`) to find it. Instead, it places a "shortcut" in a special tool shed called `vendor/bin`. You just type `vendor/bin/screwdriver` (or `vendor/bin/screwdriver`), and the shortcut takes you to the real tool. This works the same way no matter where the original package is installed.

### Purposes

- To provide a consistent, predictable path for invoking package-provided CLI tools regardless of the package's installation location.
- To abstract platform differences (Unix vs. Windows) so that the same command works across operating systems.
- To enable project-local tooling that does not depend on global installations, ensuring version consistency across team members and CI.
- To allow packages to expose build tools, code generators, test runners, and other utilities to their consumers.
- To integrate with IDE task runners, Makefiles, and CI pipelines that reference `vendor/bin/` paths.

### Syntax Rules and Structure

**Complete General Syntax: Declaring Binaries in a Package**

```json
{
    "name": "acme/code-generator",
    "bin": [
        "bin/generate-controller",
        "bin/generate-migration"
    ]
}
```

**Complete General Syntax: Consuming Binaries in a Project**

```bash
# Run a vendor binary directly
vendor/bin/phpunit

# Run a vendor binary from a Composer script
composer test  # where "test": "vendor/bin/phpunit"

# Add vendor/bin to PATH for the current session
export PATH="$PWD/vendor/bin:$PATH"
phpunit  # Now invocable without the vendor/bin/ prefix
```

**Complete General Syntax: Custom Bin Directory**

```json
{
    "config": {
        "bin-dir": "bin"
    }
}
```

**Component Breakdown:**

- `"bin"` — An array of file paths, relative to the package root, that should be exposed as executable binaries.
- `vendor/bin/` — The default directory where Composer creates proxy files for all declared binaries from all installed dependencies.
- `config.bin-dir` — Overrides the default `vendor/bin` location. Setting it to `bin` places proxies in the project root's `bin/` directory.
- Proxy files — Auto-generated executables that resolve the correct path to the package's binary. On Unix-like systems, these are symlinks or shell scripts; on Windows, `.bat` and `.php` proxies are created.
- `$_composer_autoload_path` — A global variable defined by the bin proxy (Composer 2.2+) that allows the binary to locate the project's autoloader.

**Syntax Rules:**

- Binary paths must be relative to the package root and must not contain `..` segments or special characters (`*`, `$`, `` ` ``, `"`, `&`, `^`, `|`, `<`, `>`, `(`, `)`, `%`, `!`, `;`).
- Wildcards are not expanded: `"bin/*"` is treated as a literal file path that does not exist.
- Only binaries from packages that the root project depends on are installed to `vendor/bin`. A package's own binaries are not installed for itself.
- The `vendor/bin` directory is typically added to `.gitignore` because it is regenerated on every `composer install`.

**Constraints and Limitations:**

- **Filename collisions:** If two packages declare binaries with the same filename, Composer's behaviour depends on installation order. The last installed package's binary overwrites the previous one. To avoid this, use unique binary names in your packages.
- **Missing autoloader in root binaries:** When running a binary defined by the root package itself (not a dependency), `$_composer_autoload_path` is not defined. The binary must fall back to locating the autoloader manually, typically via `__DIR__ . '/../vendor/autoload.php'`.
- **Windows/WSL complexity:** On Windows and WSL, Composer creates additional proxy files (`.bat` and `.php`) to handle path translation. These are transparent to the user but can cause confusion when debugging.
- **Binary execution permissions:** Composer sets executable permissions on the proxy files it creates. If the original binary in the package is not executable, the proxy still works because it invokes the PHP interpreter directly.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Installing and Invoking a Vendor Binary**

```bash
# Step 1: Create a project and install PHPUnit
mkdir -p vendor-bin-demo && cd vendor-bin-demo
composer init --name=acme/bin-demo --no-interaction
composer require --dev phpunit/phpunit

# Step 2: Inspect the vendor/bin directory
ls -la vendor/bin/
```

**Expected Output:**

```
total 8
drwxr-xr-x 2 user user 4096 Jan 15 10:00 .
drwxr-xr-x 5 user user 4096 Jan 15 10:00 ..
lrwxrwxrwx 1 user user   25 Jan 15 10:00 phpunit -> ../phpunit/phpunit/phpunit
```

```bash
# Step 3: Run the binary
vendor/bin/phpunit --version
```

**Expected Output:**

```
PHPUnit 11.0.0 by Sebastian Bergmann and contributors.
```

**Why:** PHPUnit's `composer.json` declares `"bin": ["phpunit"]`. Composer creates a symlink `vendor/bin/phpunit` pointing to `vendor/phpunit/phpunit/phpunit`. The consumer invokes the binary through the proxy, which works regardless of the package's physical location.

**Example 2: Creating a Custom Binary in a Package**

```json
{
    "name": "acme/project-scaffolder",
    "bin": ["bin/scaffold"]
}
```

```php
<?php
// bin/scaffold
#!/usr/bin/env php
<?php

// Locate the autoloader (works whether installed as a dependency or run directly)
if (isset($_composer_autoload_path)) {
    require $_composer_autoload_path;
} else {
    require __DIR__ . '/../vendor/autoload.php';
}

echo "Project scaffolder running...\n";

// Example: create a directory structure
$directories = ['src/Controller', 'src/Entity', 'templates'];
foreach ($directories as $dir) {
    if (!is_dir($dir)) {
        mkdir($dir, 0775, true);
        echo "Created: $dir\n";
    }
}

echo "Scaffolding complete.\n";
```

```bash
# Step 3: Install the package in a consuming project
cd ../consumer-project
composer require acme/project-scaffolder

# Step 4: Run the binary via the vendor/bin proxy
vendor/bin/scaffold
```

**Expected Output:**

```
Project scaffolder running...
Created: src/Controller
Created: src/Entity
Created: templates
Scaffolding complete.
```

**Why:** The binary uses `$_composer_autoload_path` (defined by the proxy) to locate the project's autoloader, with a fallback for direct execution. The `bin` key in the package's `composer.json` exposes `scaffold` to the consuming project's `vendor/bin` directory.

**Example 3: Adding `vendor/bin` to `PATH` for Convenience**

```bash
# Step 1: Add vendor/bin to PATH for the current shell session
export PATH="$PWD/vendor/bin:$PATH"

# Step 2: Verify
which phpunit
# Expected: /path/to/project/vendor/bin/phpunit

# Step 3: Run without the vendor/bin/ prefix
phpunit --version
```

**Expected Output:**

```
PHPUnit 11.0.0 by Sebastian Bergmann and contributors.
```

**Why:** Adding `vendor/bin` to `PATH` allows vendor binaries to be invoked by their short names, matching the convenience of globally installed tools while preserving project-local version isolation. This is a common pattern in development environments and CI scripts.

### Real-World Cases

**Case 1: PHPUnit in CI**

A CI pipeline runs `vendor/bin/phpunit` after `composer install --dev`. Because PHPUnit is installed locally from the lock file, the CI runner uses exactly the same version as the development team. The `vendor/bin` proxy ensures the correct binary is invoked regardless of the CI runner's operating system.

**Case 2: PHP-CS-Fixer as a Composer Script**

A project defines `"lint": "vendor/bin/php-cs-fixer fix --dry-run --diff"` in its `scripts` section. Developers run `composer lint` to check coding standards. The binary is invoked through the `vendor/bin` proxy, and the Composer script provides a memorable, short command.

**Case 3: Drupal's Drush**

Drupal projects use Drush (`vendor/bin/drush`) for command-line site management. Drush is installed as a project dependency, and its binary is exposed in `vendor/bin`. CI scripts and deployment hooks invoke `vendor/bin/drush` to run database updates, clear caches, and import configuration.

**Case 4: Monorepo with Multiple Packages**

A monorepo contains several packages, each with its own binaries. The root `composer.json` uses path repositories to link the packages. Composer creates proxies in the root `vendor/bin` for all binaries declared by all linked packages, providing a unified tool shed for the entire monorepo.

---

## References

- Composer: Scripts — https://getcomposer.org/doc/articles/scripts.md
- Composer: Vendor binaries and the `vendor/bin` directory — https://getcomposer.org/doc/articles/vendor-binaries.md
- Composer: Command-line interface / Commands — https://getcomposer.org/doc/03-cli.md
- Composer: The composer.json Schema — https://getcomposer.org/doc/04-schema.md
- Composer: Config — https://getcomposer.org/doc/06-config.md
- Secure PHP Development: Composer Scripts and Hooks — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Scripts-and-Hooks.md
- Secure PHP Development: Vendor Binaries — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Vendor-Binaries.md
- Packagist: composer-package-updater — https://packagist.org/packages/werbfred/composer-package-updater
- Packagist: comodojo/composer-events-handler — https://packagist.org/packages/comodojo/composer-events-handler
- Packagist: composer-git-hooks — https://packagist.org/packages/php-core/composer-git-hooks
- Stack Overflow: Composer scripts and event hooks — https://stackoverflow.com/questions/28778156/composer-scripts-and-event-hooks
- Symfony Flex Documentation — https://symfony.com/doc/current/setup/flex.html
- Laravel: Package Development — https://laravel.com/docs/packages