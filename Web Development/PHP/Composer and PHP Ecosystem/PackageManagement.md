# Package Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Package management in PHP is the practice of declaring, resolving, installing, and maintaining third-party libraries and developer tooling through Composer, using a declarative manifest (`composer.json`) and a locked resolution (`composer.lock`) to guarantee reproducible environments across development, CI, and production.

**Technical Definition**

Composer is PHP's de-facto dependency manager. It resolves, downloads, and autoloads third-party libraries per project based on a declarative `composer.json` manifest, and pins exact resolved versions in a `composer.lock` file for reproducible installs. The manifest declares runtime dependencies (`require`) and development-only dependencies (`require-dev`), each with version constraints that follow Semantic Versioning (SemVer). Composer's SAT-based resolver explores the transitive dependency graph and produces a deterministic resolution. Security auditing (`composer audit`) compares locked versions against the PHP Security Advisories Database and reports known vulnerabilities and abandoned packages.

**Beginner-Friendly Explanation**

Imagine you are building a house. Instead of manufacturing every nail, plank, and pane of glass yourself, you buy them from suppliers. **Package management** is the system that tracks which materials you need (`composer.json`), orders them, checks that they fit together, and records exactly what was delivered (`composer.lock`) so that every builder on every site gets the same materials. Some materials are for the finished house (runtime dependencies); others are scaffolding and tools used only during construction (development dependencies). **Semantic versioning** tells you whether an update is safe. **Dependency auditing** is the building inspector who checks for faulty materials. **Package maintenance** is the ongoing process of replacing worn-out parts and keeping everything up to date.

### Key Characteristics

- **Declarative Manifest:** `composer.json` expresses intent (which packages, which version ranges), while `composer.lock` records fact (exact versions, commit hashes, content checksums). Commit both to version control so every environment gets byte-identical dependencies.
- **Runtime vs. Development Separation:** `require` holds packages shipped to production; `require-dev` holds tooling (PHPUnit, PHPStan, PHP-CS-Fixer) excluded when `composer install --no-dev` is used.
- **Semantic Versioning Awareness:** Composer understands SemVer and provides operators (`^`, `~`, `*`, comparison operators) that translate version constraints into acceptable ranges.
- **Transitive Dependency Coverage:** Auditing and resolution operate on the full transitive graph, not just direct dependencies.
- **Lock-File-Driven Security:** `composer audit` reads `composer.lock` and checks exact pinned versions, ensuring the audit reflects what is actually deployed.
- **Abandonment Detection:** `composer audit` surfaces packages marked abandoned by their maintainers, a leading indicator of future unpatched vulnerabilities.

### Prerequisites

- **PHP 8.1+** with Composer 2.x installed.
- **A project directory** with a `composer.json` file (or the ability to run `composer init`).
- **Internet access** to Packagist (`packagist.org`) or configured private repositories.
- **Git** (recommended) for version control and understanding the lock file's role.
- **Basic understanding of PHP namespaces and PSR-4 autoloading** for configuring the `autoload` section.

### Related Programming Areas

- **PHP Composer Foundations & Dependency Architecture:** Version constraints, `composer.lock`, and platform requirements determine what is installed before maintenance and auditing can occur.
- **Production Autoloading & Performance Optimization:** The `post-autoload-dump` event and optimized classmaps are generated after dependency installation.
- **Package Lifecycle, Scripting, & Automation:** Composer scripts hook into install and update events to automate post-dependency tasks.
- **CI/CD Pipeline Engineering:** `composer install --no-dev`, `composer audit --no-dev`, and `composer outdated` are standard CI steps.
- **Software Supply Chain Security:** Package management is a primary control surface for preventing vulnerable or abandoned code from reaching production.

### Core Concepts / Features

1. **Third-party packages** — Declaring and resolving external dependencies via `require`.
2. **Development dependencies** — Isolating tooling in `require-dev` and excluding it from production.
3. **Semantic versioning** — Caret, tilde, wildcard, and strict version constraints.
4. **Dependency auditing** — Scanning locked dependencies for known vulnerabilities and abandoned packages.
5. **Package maintenance** — Identifying outdated packages, planning updates, and managing upgrades safely.

---

## Core Concept 1: Third-party Packages

### Definitions

**Core Definition**

Third-party packages are external libraries created by other developers and published to Packagist or private repositories, which a project declares as dependencies and installs into its local `vendor/` directory.

**Technical Definition**

A third-party package is addressed as `vendor/package-name` (e.g., `monolog/monolog`, `guzzlehttp/guzzle`) and is installed via `composer require vendor/package`. Composer reads the package's own `composer.json` to discover its transitive dependencies, resolves a compatible set of versions across the entire graph, and installs all packages into `vendor/`. The project's `composer.json` records the constraint (e.g., `"monolog/monolog": "^3.5"`), while `composer.lock` records the exact resolved version. Packages are never installed globally by default; each project maintains its own isolated dependency tree.

**Beginner-Friendly Explanation**

Think of Packagist as a massive public library of PHP code. When you need a feature (logging, HTTP requests, image processing), you do not write it yourself — you borrow a book (package) from the library. You tell Composer which books you want, and it fetches them along with every book those books depend on. The books are stored on your project's own shelf (`vendor/`), not in a shared community shelf, so different projects can borrow different versions without conflict.

### Purposes

- To add proven, tested functionality to a project without reinventing common utilities.
- To leverage the collective maintenance and security patching of the open-source community.
- To reduce development time by assembling applications from composable, well-documented components.
- To enable reproducible installations by recording exact package versions in the lock file.
- To isolate each project's dependencies from other projects on the same machine.

### Syntax Rules and Structure

**Complete General Syntax: Declaring a Third-party Package in `composer.json`**

```json
{
    "name": "acme/blog",
    "description": "Internal blog engine",
    "type": "project",
    "require": {
        "php": ">=8.2",
        "guzzlehttp/guzzle": "^7.8",
        "monolog/monolog": "^3.5"
    }
}
```

**Component Breakdown:**

- `"require"` — The key that declares runtime dependencies. Every package listed here is installed in all environments (development and production).
- `"guzzlehttp/guzzle": "^7.8"` — A third-party package with a caret version constraint. The package name consists of a vendor name (`guzzlehttp`) and a project name (`guzzle`), separated by a slash.
- `"monolog/monolog": "^3.5"` — A logging library, also constrained with a caret.
- `"php": ">=8.2"` — The PHP platform requirement, treated as a virtual package.

**Complete General Syntax: Installing a Third-party Package via CLI**

```bash
# Add a runtime dependency (edits composer.json, updates lock, installs)
composer require monolog/monolog

# Add a specific version constraint
composer require monolog/monolog:^3.5

# Install all dependencies from the lock file (CI/production)
composer install --no-dev --optimize-autoloader
```

**Syntax Rules:**

- Package names are always lowercase and follow the `vendor/package` format.
- The `require` key is the first thing specified in `composer.json`; it tells Composer which packages the project depends on.
- If no custom repository is registered, Composer assumes the package is on Packagist.org, the default package repository.
- `composer install` reads the lock file and reproduces the exact versions; `composer update` re-resolves from `composer.json` and rewrites the lock file.

**Constraints and Limitations:**

- **No global installation by default:** Packages are installed per-project in `vendor/`. Global installation is reserved for CLI tools, not runtime libraries.
- **Transitive dependency conflicts:** If two direct dependencies require incompatible versions of a shared transitive dependency, resolution fails and Composer reports the conflict.
- **Packagist availability:** Private or internal packages must be declared in a `repositories` section; otherwise, Composer cannot find them.
- **Abandoned packages:** Third-party packages may be abandoned by their maintainers. `composer audit` detects abandonment and reports it as a finding.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Adding and Using a Third-party Package**

```bash
# Step 1: Create a project
mkdir -p blog-app && cd blog-app
composer init --name=acme/blog --no-interaction

# Step 2: Add a runtime dependency
composer require monolog/monolog
```

**Expected Output:**

```
./composer.json has been updated
Running composer update monolog/monolog
Loading composer repositories with package information
Updating dependencies
Lock file operations: 2 installs, 0 updates, 0 removals
  - Locking monolog/monolog (3.7.0)
  - Locking psr/log (3.0.2)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 2 installs, 0 updates, 0 removals
  - Installing psr/log (3.0.2): Extracting archive
  - Installing monolog/monolog (3.7.0): Extracting archive
Generating autoload files
```

```php
<?php
// Step 3: Use the package via the autoloader
require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Level;

$log = new Logger('blog');
$log->pushHandler(new StreamHandler('app.log', Level::Debug));
$log->info('Application started.');
echo "Log entry written.\n";
```

**Expected Output:**

```
Log entry written.
```

**Why:** `composer require monolog/monolog` adds the package to `require` in `composer.json`, resolves Monolog and its transitive dependency (`psr/log`), writes both to `composer.lock`, and installs them into `vendor/`. The generated autoloader maps the `Monolog\` namespace to the correct files, so `use Monolog\Logger` works without any manual `require`.

**Example 2: Installing from a Lock File (Reproducible Environment)**

```bash
# Step 1: Clone the repository (composer.lock is committed)
git clone https://github.com/acme/blog.git && cd blog

# Step 2: Install exactly what the lock file pins
composer install --no-dev --optimize-autoloader
```

**Expected Output:**

```
Installing dependencies from lock file (excluding dev dependencies)
Verifying lock file contents can be installed on current platform.
Package operations: 8 installs, 0 updates, 0 removals
  - Installing psr/log (3.0.2): Extracting archive
  - Installing monolog/monolog (3.7.0): Extracting archive
  ... (6 more)
Generating optimized autoload files
```

**Why:** Because `composer.lock` is committed to version control, `composer install` reproduces the exact same dependency tree on every machine. The `--no-dev` flag excludes development dependencies, and `--optimize-autoloader` generates a static classmap for production performance.

### Real-World Cases

**Case 1: Laravel Application Dependencies**

A Laravel application declares `laravel/framework`, `guzzlehttp/guzzle`, and `monolog/monolog` in `require`. These are all third-party packages that ship with the application to production. The `composer.lock` file pins the exact versions, ensuring that every deployment runs the same framework and library code.

**Case 2: Symfony Bundle Ecosystem**

A Symfony application requires `symfony/console`, `symfony/http-client`, and `doctrine/orm`. Each of these is a third-party package with its own transitive dependencies. Composer resolves the full graph and installs all packages into `vendor/`, where Symfony's autoloader maps the `Symfony\` and `Doctrine\` namespaces to the correct directories.

**Case 3: Private Repository Integration**

An enterprise project requires a private package hosted on a company Git server. The `composer.json` declares a `vcs` repository entry for the private package, and `composer require acme/private-lib` resolves and installs it alongside public Packagist packages. The lock file records the private package's commit hash for reproducibility.

---

## Core Concept 2: Development Dependencies

### Definitions

**Core Definition**

Development dependencies are packages required only during development, testing, linting, and static analysis — not at runtime — and are declared in the `require-dev` section of `composer.json`.

**Technical Definition**

Composer separates dependencies into two sections: `require` (runtime dependencies shipped to production) and `require-dev` (development-only dependencies). Packages in `require-dev` are installed when `composer install` or `composer update` is run without `--no-dev`, and are skipped when `--no-dev` is passed. The `autoload-dev` section similarly provides autoloading rules for development-only classes (e.g., test namespaces). This separation ensures that test frameworks, static analysers, and debug tools never reach production environments, reducing disk usage, memory consumption, and attack surface.

**Beginner-Friendly Explanation**

Think of building a house. The finished house needs walls, windows, and a roof (runtime dependencies). But during construction, you also need scaffolding, power tools, and safety equipment (development dependencies). When the house is finished, the scaffolding is removed and the tools go back to the workshop. Development dependencies are the scaffolding and tools — essential for building, irrelevant for living in.

### Purposes

- To keep production installations lean by excluding test frameworks, static analysers, and debug tools.
- To reduce the security attack surface by preventing development-only code from reaching production.
- To accelerate production deployment by installing and autoloading fewer packages.
- To allow CI pipelines to install the full toolchain for testing while production installs only runtime dependencies.
- To ensure that development tooling versions are locked and reproducible across the development team.

### Syntax Rules and Structure

**Complete General Syntax: Declaring Development Dependencies**

```json
{
    "name": "acme/blog",
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.5"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0",
        "phpstan/phpstan": "^1.11",
        "friendsofphp/php-cs-fixer": "^3.50",
        "mockery/mockery": "^1.6"
    },
    "autoload": {
        "psr-4": { "Acme\\Blog\\": "src/" }
    },
    "autoload-dev": {
        "psr-4": { "Acme\\Blog\\Tests\\": "tests/" }
    }
}
```

**Component Breakdown:**

- `"require-dev"` — The key that declares development-only dependencies. These packages are installed by default but excluded when `--no-dev` is used.
- `"phpunit/phpunit": "^11.0"` — The test framework. Required for running tests but never for serving production requests.
- `"phpstan/phpstan": "^1.11"` — A static analysis tool. Used in CI and local development.
- `"friendsofphp/php-cs-fixer": "^3.50"` — A coding standards fixer.
- `"autoload-dev"` — Autoloading rules for development-only classes. The `Acme\Blog\Tests\` namespace maps to the `tests/` directory and is excluded when `--no-dev` is used.

**Complete General Syntax: Installing with and without Development Dependencies**

```bash
# Development installation (includes require-dev)
composer install

# Production installation (excludes require-dev)
composer install --no-dev

# Via environment variable
COMPOSER_NO_DEV=1 composer install
```

**Syntax Rules:**

- `--no-dev` only affects `install` and `update`. It does not uninstall already-installed development packages; it skips them during resolution and installation.
- `autoload-dev` rules are omitted when `--no-dev` is used, so test classes under `Acme\Blog\Tests\` are not autoloaded in production.
- Development dependencies should never be referenced by runtime code. If production code depends on a class from `require-dev`, the application fails with a "class not found" error.
- Use `composer require --dev vendor/package` to add a package to `require-dev` and install it in one step.

**Constraints and Limitations:**

- **Runtime dependency leakage:** If application code accidentally depends on a package in `require-dev`, production will fail. Static analysis tools like PHPStan can detect such leaks.
- **Lock file size:** `composer.lock` contains both runtime and development packages. Production installations with `--no-dev` still read the lock file but skip the `packages-dev` entries.
- **CI cache invalidation:** CI pipelines that cache `vendor/` must invalidate the cache when switching between `--dev` and `--no-dev` installations, because the two directories have different contents.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Adding and Using Development Dependencies**

```bash
# Step 1: Add PHPUnit as a development dependency
composer require --dev phpunit/phpunit
```

**Expected Output:**

```
./composer.json has been updated
Running composer update phpunit/phpunit
...
  - Locking phpunit/phpunit (11.0.0)
  ... (12 more dev packages)
Writing lock file
Installing dependencies from lock file (including require-dev)
```

```bash
# Step 2: Run the test framework
vendor/bin/phpunit --version
```

**Expected Output:**

```
PHPUnit 11.0.0 by Sebastian Bergmann and contributors.
```

```bash
# Step 3: Simulate a production installation (no dev packages)
rm -rf vendor/
composer install --no-dev --quiet

# Step 4: Verify that PHPUnit is absent
ls vendor/bin/phpunit 2>/dev/null || echo "PHPUnit is not installed in production."
```

**Expected Output:**

```
PHPUnit is not installed in production.
```

**Why:** `composer require --dev phpunit/phpunit` adds PHPUnit to `require-dev` in `composer.json` and installs it into `vendor/`. The binary is exposed in `vendor/bin`. When `--no-dev` is used, PHPUnit is skipped entirely — it is not in `vendor/`, and no binary is created. This ensures that the test framework never reaches production.

**Example 2: Common Development Dependencies**

```json
{
    "require-dev": {
        "phpunit/phpunit": "^11.0",
        "phpstan/phpstan": "^1.11",
        "friendsofphp/php-cs-fixer": "^3.50",
        "mockery/mockery": "^1.6",
        "fakerphp/faker": "^1.23"
    }
}
```

**Why these packages:** PHPUnit is the test framework; PHPStan performs static analysis; PHP-CS-Fixer enforces coding standards; Mockery provides test doubles; Faker generates realistic test data. None of these should be installed in production because they add disk usage, memory overhead, and security surface area without providing runtime value.

### Real-World Cases

**Case 1: CI Pipeline with Full Toolchain**

A CI pipeline runs `composer install` (without `--no-dev`) to install PHPUnit, PHPStan, and PHP-CS-Fixer. It then runs `vendor/bin/phpunit`, `vendor/bin/phpstan analyse`, and `vendor/bin/php-cs-fixer fix --dry-run`. After tests pass, a separate production build step runs `composer install --no-dev --optimize-autoloader` to produce a lean artifact.

**Case 2: Laravel Development Dependencies**

A Laravel application's `composer.json` includes `laravel/tinker`, `laravel/pint`, `phpunit/phpunit`, and `nunomaduro/collision` in `require-dev`. These tools are essential for local development but are excluded from production deployments via `composer install --no-dev`.

**Case 3: Docker Multi-Stage Build**

A Dockerfile uses a two-stage build. The first stage runs `composer install` (with dev dependencies) and executes the test suite. The second stage copies only the application source and runs `composer install --no-dev --optimize-autoloader`, producing a minimal production image that contains only runtime dependencies.

---

## Core Concept 3: Semantic Versioning (SemVer)

### Definitions

**Core Definition**

Semantic Versioning is a version-numbering contract that uses a `MAJOR.MINOR.PATCH` format to communicate the nature of changes between releases, allowing Composer to resolve compatible updates automatically while respecting backward-compatibility boundaries.

**Technical Definition**

Under SemVer, the `MAJOR` version increments on incompatible API changes, `MINOR` on backward-compatible functionality additions, and `PATCH` on backward-compatible bug fixes. Composer translates version constraints into ranges of acceptable versions using operators: the caret (`^`) allows non-breaking updates (`^3.5` → `>=3.5.0 <4.0.0`); the tilde (`~`) allows more restrictive updates (`~3.5` → `>=3.5.0 <3.6.0` when a minor is given, or `~3.5.2` → `>=3.5.2 <3.6.0`); the wildcard (`*`) matches any version; and comparison operators (`>=`, `<`) define explicit ranges. Composer's resolver selects the highest version that satisfies all constraints in the project's dependency graph.

**Beginner-Friendly Explanation**

Think of version numbers as a promise from the package maintainer. `3.5.2` means: "This is version 3 (major), the 5th minor update, with 2 patches." The promise is: if you upgrade within version 3 (e.g., `3.7.0`), your code should still work. If you upgrade to version 4, the maintainer is warning you that something might break. Composer's operators let you say "give me any version 3.x" (`^3.5`), "give me only 3.5.x" (`~3.5.0`), or "give me exactly 3.5.2" (`3.5.2`).

### Purposes

- To communicate the backward-compatibility of a release through its version number.
- To allow Composer to automatically select the newest version that does not introduce breaking changes.
- To provide a clear, unambiguous way to express version constraints in `composer.json`.
- To prevent accidental upgrades that could break the application by respecting the major version boundary.
- To enable safe, automated dependency updates within a defined compatibility range.

### Syntax Rules and Structure

**Complete General Syntax: Version Constraints in `composer.json`**

```json
{
    "require": {
        "vendor/package": "^1.2.3",
        "vendor/other": "~2.1",
        "vendor/third": "3.5.*",
        "vendor/pinned": "4.0.1",
        "vendor/range": ">=1.0 <2.0"
    }
}
```

**Component Breakdown and Constraint Table:**

| Constraint | Example | Matches | Notes |
|---|---|---|---|
| Caret (`^`) | `^3.5` | `>=3.5.0 <4.0.0` | Allows non-breaking updates. Preferred default. |
| Tilde (`~`) | `~3.5` | `>=3.5.0 <3.6.0` | Only patch updates when a minor is given. |
| Tilde with patch | `~3.5.2` | `>=3.5.2 <3.6.0` | Patch updates within minor 3.5. |
| Wildcard (`*`) | `3.5.*` | Any `3.5.x` | Wildcard on patch. |
| Strict pin | `3.5.2` | Exactly `3.5.2` | Reproducible but never receives patches automatically. |
| Comparison | `>=3.5` | Anything `3.5+` | Dangerous open upper bound; can silently jump major versions. |
| Range | `>=1.0 <2.0` | `1.0` to `<2.0` | Explicit range. |

**Syntax Rules:**

- The caret operator (`^`) is the preferred default for most dependencies because it respects SemVer's major version boundary. Composer documentation explicitly recommends `^3.4` over `>=3.4` because the latter has no upper bound.
- The tilde operator (`~`) is more restrictive when a patch digit is specified: `~1.2.3` allows only patch updates (`<1.3.0`), while `~1.2` allows minor updates (`<2.0.0`).
- Wildcards (`*`) should be used sparingly. They are acceptable only for platform extensions like `ext-pdo`, never for real libraries.
- Mixing comparison operators with wildcards (e.g., `>=2.*`) is invalid and causes Composer to throw an error.

**Constraints and Limitations:**

- **SemVer is a social contract, not a technical guarantee:** Packages that do not follow SemVer strictly can introduce breaking changes in minor or patch releases. Composer cannot prevent this.
- **Caret operator and major version 0:** In SemVer, `0.x` versions are considered unstable. Composer's caret operator on `^0.3` allows `>=0.3.0 <0.4.0` (only patch updates), reflecting the instability of pre-1.0 versions.
- **Open upper bounds (`>=3.5`):** Allowing any version above a threshold can silently jump major versions and break the application. Use `^` instead.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Caret vs. Tilde in Practice**

```bash
# Step 1: Create a project with a caret constraint
composer init --name=acme/caret-demo --require="monolog/monolog:^3.5" --no-interaction
composer install
composer show monolog/monolog | grep versions
```

**Expected Output:**

```
versions : * 3.7.0
```

```bash
# Step 2: Create a project with a tilde constraint
composer init --name=acme/tilde-demo --require="monolog/monolog:~3.5" --no-interaction
composer install
composer show monolog/monolog | grep versions
```

**Expected Output:**

```
versions : * 3.5.0
```

**Why:** `^3.5` allows any version from 3.5.0 up to (but not including) 4.0.0, so the resolver selects the newest available version (3.7.0). `~3.5` allows only patch updates within minor version 3.5, so the resolver selects 3.5.0 (or the latest 3.5.x patch), not 3.7.0.

**Example 2: The Danger of Open Upper Bounds**

```json
{
    "require": {
        "vendor/package": ">=1.0"
    }
}
```

**Why this is dangerous:** If `vendor/package` releases version 2.0.0 with breaking changes, `composer update` will install it because the constraint `>=1.0` has no upper bound. The correct constraint is `^1.0` (allowing `>=1.0.0 <2.0.0`) or `~1.0` (allowing `>=1.0.0 <1.1.0`). Composer's documentation explicitly warns against unbound version constraints.

### Real-World Cases

**Case 1: Framework Dependency Constraints**

A Laravel application uses `"laravel/framework": "^11.0"`. This allows any Laravel 11.x release but prevents accidental upgrades to Laravel 12.0, which may introduce breaking changes. The application receives bug fixes and minor features automatically.

**Case 2: Library Development with Permissive Constraints**

A PSR-7 implementation library declares `"psr/http-message": "^1.0 || ^2.0"` to indicate compatibility with both major versions of the PSR-7 interface. This allows the library to be used by applications that have locked either version, maximising compatibility.

**Case 3: Strict Pinning in Production**

A production deployment uses `composer install` with a committed lock file. Although `composer.json` declares `^3.5`, the lock file pins `3.7.0` exactly. Production never receives an unexpected update; updates are applied deliberately via `composer update` followed by testing.

---

## Core Concept 4: Dependency Auditing

### Definitions

**Core Definition**

Dependency auditing is the practice of scanning a project's locked dependencies for known security vulnerabilities and abandoned packages, using `composer audit` against the PHP Security Advisories Database.

**Technical Definition**

`composer audit`, built into Composer since 2.4, reads the exact package versions pinned in `composer.lock` and checks them against the PHP Security Advisories Database, which aggregates advisories from the PHP security community and the GitHub Advisory Database. Because it works from the lock file, it covers the full transitive dependency graph, not just direct requirements. The command exits with a non-zero code when advisories are found, making it suitable as a CI gate. It also surfaces abandoned packages — dependencies whose maintainers have marked them as no longer maintained — and can be configured to treat abandonment as a failure.

**Beginner-Friendly Explanation**

Imagine you are assembling a product from parts supplied by dozens of vendors. Some parts might have known safety defects (vulnerabilities). **Dependency auditing** is the quality inspector who checks every part against a global database of known defects. It also flags parts whose manufacturers have gone out of business (abandoned packages), because those parts will never receive safety updates. The inspector works from the exact list of parts you actually installed (the lock file), not just the parts you ordered.

### Purposes

- To identify dependencies with known security vulnerabilities before they reach production.
- To surface abandoned packages that will not receive future security patches.
- To provide a CI-friendly gate that fails the build when vulnerabilities are detected.
- To cover the full transitive dependency graph, including packages never directly required.
- To shift security checks left, flagging vulnerabilities in the same command that introduces them.

### Syntax Rules and Structure

**Complete General Syntax: Running a Dependency Audit**

```bash
composer audit [options]
```

**Component Breakdown:**

- `composer audit` — Runs the audit against `composer.lock`. Reports all known vulnerabilities in installed packages.
- `--format=<plain|table|json|summary>` — Output format. `table` is the default for humans; `json` is for tooling; `summary` provides a one-line count.
- `--locked` — Audits `composer.lock` even if `vendor/` is not installed. This is the default source anyway.
- `--no-dev` — Ignores `require-dev` packages, matching a production installation.
- `--abandoned=<report|fail|ignore>` — Controls how abandoned packages are treated. `fail` makes an abandoned dependency break the audit.
- `--ignore-severity` — Ignores advisories of one or more severity levels.

**Complete General Syntax: Automatic Auditing During Install/Update**

```bash
# Auditing runs automatically after update
composer update

# Disable automatic auditing
composer update --no-audit

# Run an audit as part of install
composer install --audit
```

**Syntax Rules:**

- `composer audit` exits with code `0` when no advisories are found and a non-zero code when at least one vulnerability is reported. This makes it usable as a CI gate.
- Automatic auditing runs by default after `composer update` and can be overridden with `--no-audit`.
- The `audit.abandoned` config setting controls whether abandoned packages are reported (`report`, the current default), ignored (`ignore`), or treated as failures (`fail`, the future default in Composer 2.7).
- The `audit > block-insecure` config setting controls whether updates to package versions with known security advisories are blocked (defaults to `true`).

**Constraints and Limitations:**

- **No reachability analysis:** `composer audit` tells you a vulnerable version is installed. It cannot tell you whether your application actually invokes the vulnerable class or method. A flaw in an unused code path is reported the same as one in an active request path.
- **Advisory latency:** The audit is only as current as the advisories it consumes. Freshly published vulnerabilities may not yet have an advisory and will pass silently.
- **Snapshot only:** Each run is a point-in-time check. There is no SLA tracking, no accepted-risk record with an expiry, and no consolidated view across multiple projects.
- **Not a code scanner:** `composer audit` finds known-vulnerable dependencies, not bugs in your own code, and not zero-days.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Running a Basic Audit**

```bash
# Step 1: Install dependencies
composer install

# Step 2: Run the audit
composer audit
```

**Expected Output (when a vulnerability is found):**

```
Found 1 security vulnerability advisory affecting 1 package:

+-------------------+----------------------------------------------------------+
| Package           | guzzlehttp/guzzle                                        |
| CVE               | CVE-2022-31090                                            |
| Title             | Cross-domain cookie leakage                              |
| Affected versions | >=7.0.0,<7.4.5                                            |
| Reported at       | 2022-06-20T15:00:00+00:00                                 |
+-------------------+----------------------------------------------------------+
```

**Why:** Composer reads `composer.lock`, extracts every installed package and version, and queries the advisories database. It reports each advisory with the package name, CVE identifier, title, affected version range, and timestamp.

**Example 2: CI Pipeline with Audit Gate**

```yaml
# .github/workflows/ci.yml (excerpt)
steps:
  - uses: actions/checkout@v4
  - run: composer install --no-interaction
  - run: composer audit --no-dev --abandoned=report
  - run: vendor/bin/phpunit
```

**Expected Output (when no advisories are found):**

```
No security vulnerability advisories found.
```

**Why:** The CI pipeline runs `composer audit --no-dev --abandoned=report` after installation. If a vulnerability is found, the command exits with a non-zero code and the pipeline fails. The `--no-dev` flag limits the audit to production dependencies, and `--abandoned=report` lists abandoned packages without failing the build.

**Example 3: Auditing with JSON Output for Tooling**

```bash
composer audit --format=json --no-dev
```

**Expected Output (excerpt):**

```json
{
    "advisories": {
        "guzzlehttp/guzzle": [
            {
                "advisoryId": "CVE-2022-31090",
                "packageName": "guzzlehttp/guzzle",
                "affectedVersions": ">=7.0.0,<7.4.5",
                "title": "Cross-domain cookie leakage",
                "severity": "high"
            }
        ]
    },
    "abandoned": []
}
```

**Why:** The JSON format is designed for tooling integration. CI systems, dashboards, and security platforms can parse the output and create tickets, send notifications, or generate reports.

### Real-World Cases

**Case 1: Scheduled Nightly Audits**

A security team runs `composer audit --locked --format=json` as a nightly cron job against the default branch. Even when no one changes dependencies, newly disclosed advisories in already-shipped code surface within 24 hours.

**Case 2: Automatic Auditing During Development**

A developer runs `composer update` to add a new package. Composer automatically audits the newly resolved dependencies and reports any advisories in the same terminal session. The developer sees the vulnerability immediately, before committing the change.

**Case 3: Abandoned Package Governance**

A project's audit reports that a dependency is abandoned. The team sets `audit.abandoned` to `fail` in `composer.json`, forcing a conversation about replacing the dead dependency before it becomes a security problem. They fork the package, apply a patch, and point Composer at the fork via a VCS repository entry.

---

## Core Concept 5: Package Maintenance

### Definitions

**Core Definition**

Package maintenance is the ongoing process of identifying outdated dependencies, planning and executing updates, and managing the lifecycle of third-party packages to keep an application secure, performant, and compatible.

**Technical Definition**

Package maintenance encompasses the use of `composer outdated` to list packages with newer versions available, `composer update` to resolve and install newer versions within constraints, and `composer why-not` to diagnose version conflicts. The recommended workflow is to check for outdated packages, read release notes and changelogs, update in a development environment, test thoroughly, and then deploy the updated lock file to staging and production. Updates should never be run directly in production; they should be developed, tested, and deployed through the normal pipeline.

**Beginner-Friendly Explanation**

Think of package maintenance as car maintenance. You check the dashboard for warning lights (`composer outdated`), review the mechanic's recommendations (release notes), replace worn parts in the garage (development environment), test-drive the car, and only then take it on the road (production). You do not replace the engine while driving on the highway.

### Purposes

- To identify dependencies that have newer versions available, including security patches.
- To prioritise security updates over feature updates.
- To plan and execute updates safely, with testing in development and staging before production.
- To diagnose version conflicts that prevent updates.
- To manage the lifecycle of abandoned or deprecated packages by finding replacements or forking.

### Syntax Rules and Structure

**Complete General Syntax: Checking for Outdated Packages**

```bash
composer outdated
composer outdated --direct        # Only direct dependencies
composer outdated --major-only    # Only major version updates
composer outdated --format=json   # Machine-readable output
```

**Component Breakdown:**

- `composer outdated` — Lists all packages with newer versions available, including transitive dependencies.
- `--direct` — Limits the output to direct dependencies (those in `require` and `require-dev`).
- `--major-only` — Shows only packages with major version updates available.
- `--format=json` — Outputs the list in JSON for tooling integration.

**Complete General Syntax: Updating Packages**

```bash
# Update all packages to the newest versions allowed by composer.json
composer update

# Update a single package
composer update monolog/monolog

# Update a single package and its dependencies
composer update monolog/monolog --with-all-dependencies

# Dry-run to see what would change without applying
composer update monolog/monolog --dry-run
```

**Component Breakdown:**

- `composer update` — Re-resolves all dependencies, updates `composer.lock`, and installs the new versions.
- `composer update vendor/package` — Updates only the specified package, leaving others at their locked versions.
- `--with-all-dependencies` (or `-W`) — Also updates the specified package's dependencies, which may be necessary when the package requires newer versions of shared libraries.
- `--dry-run` — Simulates the update and shows what would change without modifying the lock file or installing anything.

**Complete General Syntax: Diagnosing Update Blockers**

```bash
composer why-not vendor/package 3.0.0
```

**Component Breakdown:**

- `composer why-not vendor/package version` — Explains why a specific version of a package cannot be installed. It lists the constraints from other packages that conflict with the requested version.

**Syntax Rules:**

- Never run `composer update` directly in production. Always update in development, test thoroughly, and deploy the updated lock file.
- Update security-critical packages first. Vulnerability fixes are the highest priority.
- Read release notes and changelogs before updating, especially for major version changes.
- Use `--dry-run` to preview the impact of an update before applying it.
- After updating, run the test suite and static analysis to catch regressions.

**Constraints and Limitations:**

- **Update conflicts:** Updating one package may be blocked by another package's constraints. `composer why-not` helps diagnose these conflicts.
- **Major version updates:** Major version updates often contain breaking changes. They require code modifications, testing, and possibly changes to the application's own code.
- **Abandoned packages:** If a package is abandoned and has no successor, the team must decide whether to fork it, replace it, or continue using it with accepted risk.
- **Dependency drift:** Frequent small updates are easier to manage than infrequent large updates. Regular monthly maintenance is recommended over annual "big bang" updates.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Checking for Outdated Packages**

```bash
# Step 1: List outdated packages
composer outdated --direct
```

**Expected Output:**

```
Direct dependencies:
monolog/monolog  3.5.0  3.7.0  A logging library for PHP
guzzlehttp/guzzle  7.4.5  7.9.2  HTTP client
```

```bash
# Step 2: Update a single package
composer update monolog/monolog --with-all-dependencies
```

**Expected Output:**

```
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 1 update, 0 removals
  - Upgrading monolog/monolog (3.5.0 => 3.7.0)
Writing lock file
Installing dependencies from lock file (including require-dev)
  - Upgrading monolog/monolog (3.5.0 => 3.7.0): Extracting archive
Generating autoload files
```

**Why:** `composer outdated --direct` lists direct dependencies with newer versions available. `composer update monolog/monolog --with-all-dependencies` updates Monolog to the newest version allowed by `^3.5` (3.7.0) and also updates any of Monolog's dependencies if necessary. The lock file is rewritten, and the new version is installed.

**Example 2: Diagnosing an Update Blocker**

```bash
# Attempt to update a package that is blocked
composer update vendor/package --dry-run
```

**Expected Output:**

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires vendor/package ^2.0, but it is fixed to 1.5.0 by the lock file.
    - vendor/other 1.0.0 requires vendor/package ^1.0, which conflicts with the requested ^2.0.
```

```bash
# Diagnose why vendor/package 2.0.0 cannot be installed
composer why-not vendor/package 2.0.0
```

**Expected Output:**

```
vendor/other 1.0.0 requires vendor/package ^1.0
```

**Why:** The update is blocked because another package (`vendor/other`) requires an older major version of `vendor/package`. `composer why-not` identifies the conflicting package and its constraint, allowing the developer to decide whether to update `vendor/other` first, find an alternative, or accept the older version.

### Real-World Cases

**Case 1: Monthly Maintenance Schedule**

A team schedules a monthly maintenance window. They run `composer outdated --direct`, review the list, prioritise security updates, update one package at a time with `--dry-run` first, run the test suite after each update, and deploy the updated lock file to staging for a week before production.

**Case 2: Security Patch Emergency**

A critical CVE is published for a dependency. The team runs `composer audit` to confirm the affected version, then `composer update vendor/package --with-all-dependencies` to pull the patched version. They run the test suite, deploy to staging, and promote to production within hours.

**Case 3: Major Version Upgrade Project**

A team plans to upgrade from Laravel 10 to Laravel 11. They use `composer outdated --major-only` to identify major version updates, read the Laravel upgrade guide, update `composer.json` to `^11.0`, run `composer update` in a development branch, fix breaking changes, and deploy through the normal pipeline.

---

## References

- Composer: Basic usage — https://getcomposer.org/doc/01-basic-usage.md
- Composer: Versions and constraints — https://getcomposer.org/doc/articles/versions.md
- Composer: Why are unbound version constraints a bad idea? — https://getcomposer.org/doc/articles/versions.md#why-are-unbound-version-constraints-a-bad-idea
- Composer: Command-line interface / Commands — https://getcomposer.org/doc/03-cli.md
- Composer: The composer.json Schema — https://getcomposer.org/doc/04-schema.md
- Composer: 2.4.0-RC1 Changelog (audit command introduction) — https://getcomposer.org/changelog/2.4.0-RC1
- Composer: 2.7.0 Changelog (audit.abandoned default) — https://getcomposer.org/changelog/2.7.0
- Secure PHP Development: Package Management — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Package-Management.md
- Secure PHP Development: composer.json — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/composer-json.md
- Secure PHP Development: Security Auditing with composer audit — https://github.com/armourinfosec/Secure-PHP-Development/blob/586b887138a30c25fc459684c78dffe0a2296c17/Composer/Security-Auditing-with-composer-audit.md
- Safeguard.sh: Auditing PHP Dependencies with composer audit — https://safeguard.sh/resources/blog/composer-audit-php-dependencies
- Packagist Blog: Discover Security Advisories with Composer's audit command — https://blog.packagist.com/discover-security-advisories-with-composers-audit-command/
- Laracasts: Updating strategies — https://laracasts.com/discuss/channels/general-discussion/updating-strategies
- Packagist: cs278/composer-audit — https://packagist.org/packages/cs278/composer-audit
- Packagist: koeker/composer-audit-guard — https://packagist.org/packages/koeker/composer-audit-guard
- Packagist: satheez/laravel-package-doctor — https://packagist.org/packages/satheez/laravel-package-doctor