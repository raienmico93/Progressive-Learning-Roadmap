# PHP Composer Foundations & Dependency Architecture — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Composer is PHP's de-facto dependency manager: a per-project tool that reads a declarative manifest (`composer.json`), resolves a compatible set of package versions from Packagist and other repositories, installs them into a `vendor/` directory, and generates a PSR-4 autoloader so that application code never needs manual `require` statements.

**Technical Definition**

Composer is a package management system for PHP that provides a SAT-based dependency resolver (ported from openSUSE's libzypp), a manifest schema for declaring dependencies and version constraints, a lock file for reproducible installations, and a runtime autoloader. It treats the PHP interpreter, PHP extensions, and system libraries as virtual platform packages (`php`, `ext-*`, `lib-*`) that can be required, conflicted with, or provided by packages. The dependency resolution engine translates version constraints into boolean satisfiability clauses and uses conflict-driven clause learning (CDCL) to find a set of package versions that simultaneously satisfies all constraints in the dependency graph.

**Beginner-Friendly Explanation**

Imagine you are building a piece of furniture (your PHP application). Instead of manufacturing every screw, bolt, and panel yourself, you buy them from hardware stores (packages). Composer is your personal shopping assistant: you give it a list of what you need ("a 3.5-compatible screwdriver set"), and it finds all the compatible parts, checks that they fit together, orders them, and delivers them to your workshop (the `vendor/` directory). It even remembers the exact models you received (the lock file) so you get the same parts every time you rebuild.

### Key Characteristics

- **Per-Project Dependency Management:** Dependencies are declared and installed per project, not globally, ensuring isolation between projects.
- **SAT-Based Resolution:** Composer uses a boolean satisfiability solver to resolve complex dependency trees with nested constraints.
- **Two-File Architecture:** `composer.json` (human-authored intent) and `composer.lock` (machine-generated exact versions) work together to balance flexibility and reproducibility.
- **Semantic Versioning Awareness:** Composer understands SemVer and provides operators (`^`, `~`, `*`) that translate version constraints into acceptable ranges.
- **Platform Package Virtualization:** PHP itself, extensions, and system libraries are treated as packages that can be required with version constraints.
- **Autoloading:** Composer generates a PSR-4-compatible autoloader, eliminating manual `require`/`include` calls.

### Prerequisites

- **PHP 7.2.5+** (Composer 2.x requires PHP 7.2.5 or later; Composer 1.x supports older versions but is deprecated).
- **A project directory** with write permissions for the `vendor/` directory.
- **Internet access** to Packagist or configured private repositories (or a Composer cache for offline installations).
- **Basic understanding of PHP namespaces and PSR-4 autoloading** for configuring the `autoload` section.
- **Git** (recommended) for version control and understanding the lock file's role.

### Related Programming Areas

- **Semantic Versioning (SemVer):** Composer's version constraint system is built on SemVer principles.
- **PSR Standards:** PSR-4 (autoloading) and PSR-0 (legacy) define how Composer maps namespaces to file paths.
- **CI/CD Pipelines:** `composer install --no-dev` is a standard step in deployment pipelines for reproducible production builds.
- **Software Supply Chain Security:** Composer manages the packages that run with your application's privileges, making dependency auditing a core security control.
- **Monorepo and Multi-Package Development:** Path repositories and workspace configurations allow local package development within a monorepo.

### Core Concepts / Features

1. **The Dependency Resolution Engine** — SAT-based solving of complex dependency trees with nested constraints.
2. **Manifest vs. Lockfiles** — `composer.json` (declarative intent) vs. `composer.lock` (exact resolved versions).
3. **Semantic Versioning (SemVer)** — Caret, tilde, wildcard, and strict version constraints.
4. **Global vs. Project Dependencies** — CLI tools vs. runtime libraries and the risks of global installs.
5. **Platform Requirements & Constraints** — Enforcing PHP versions and extensions via `require` and `config.platform`.

---

## Core Concept 1: The Dependency Resolution Engine

### Definitions

**Core Definition**

The dependency resolution engine is the component of Composer that takes a set of version constraints from `composer.json` and the transitive constraints from all dependencies, and finds a set of concrete package versions that satisfies every constraint simultaneously.

**Technical Definition**

Composer's resolver translates the dependency graph into a boolean satisfiability (SAT) problem. Each package version is a variable; constraints become clauses that must be satisfied. The solver uses conflict-driven clause learning (CDCL) — the same algorithmic family used in modern SAT solvers — to efficiently explore the version space, learn from conflicts, and backtrack to find a valid assignment. The algorithm was ported from openSUSE's `libzypp` package manager. For nested dependencies, Composer attempts to determine a union constraint that loads all potential required versions without loading every version of packages that are dependencies of dependencies of the root package or deeper.

**Beginner-Friendly Explanation**

Imagine you are planning a dinner party and each guest has dietary restrictions. Guest A says "I need a gluten-free dish." Guest B says "I need a dairy-free dish." Guest C says "I need a nut-free dish." You need to find one menu that satisfies all three restrictions. Now imagine 50 guests with overlapping and sometimes conflicting restrictions. Composer's resolver is the algorithm that finds a menu (a set of package versions) that satisfies everyone, and if it cannot, it tells you exactly which guests' restrictions conflict.

### Purposes

- To resolve a compatible set of package versions from a complex graph of transitive dependencies.
- To detect and report conflicts when no valid combination of versions exists.
- To prioritise the root package's constraints over transitive constraints (the root package's requirements take precedence).
- To allow the solver to discard incompatible versions early, reducing the search space.
- To produce a deterministic resolution that can be locked for reproducibility.

### Syntax Rules and Structure

The resolution engine is not directly invoked via syntax; it operates behind the scenes when running `composer install` or `composer update`. The constraints it processes are defined in the `require` and `require-dev` sections of `composer.json`.

**Complete General Syntax: Declaring Nested Dependencies**

```json
{
    "name": "acme/application",
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.5",
        "guzzlehttp/guzzle": "^7.8"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    }
}
```

**Component Breakdown:**

- `"monolog/monolog": "^3.5"` — The root package requires Monolog 3.5 or later, but below 4.0. If Monolog's `composer.json` requires `psr/log: ^2.0 || ^3.0`, the solver must find a Monolog version whose `psr/log` constraint is compatible with any other packages requiring `psr/log`.
- `"guzzlehttp/guzzle": "^7.8"` — Guzzle may require `psr/http-client: ^1.0`. The solver must find a version of `psr/http-client` that satisfies both Guzzle and any other package that depends on it.
- The solver explores the intersection of all constraints. If Package A requires `psr/log: ^2.0` and Package B requires `psr/log: ^1.0`, no single version satisfies both, and Composer reports a conflict.

**Syntax Rules:**

- The root package's constraints are authoritative: Composer will not select a version that violates the root's `require` section, even if a transitive dependency requests it.
- For nested dependencies, the solver attempts to load a union of potential versions rather than all versions of deep dependencies, optimising resolution performance.
- Repository nesting is not resolved transitively: if a dependency declares a custom repository, that repository is not available for resolving the root project's dependencies.

**Constraints and Limitations:**

- **SAT solving is NP-complete in the worst case:** For very large dependency graphs with many conflicting constraints, resolution can be slow. Composer's CDCL-based solver mitigates this but does not eliminate the theoretical complexity.
- **Version constraint combination errors:** Composer rejects invalid combinations such as `>=2.*` or `>=1.1.*`, which mix comparison operators with wildcards.
- **Circular dependencies:** Composer detects circular dependencies but cannot resolve them; packages must be designed without circular references.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: A Successful Resolution**

```bash
# Step 1: Create a project
mkdir my-app && cd my-app
composer init --name=acme/my-app --require="monolog/monolog:^3.5" --no-interaction

# Step 2: Run install
composer install
```

**Expected Output:**

```
Loading composer repositories with package information
Updating dependencies
Lock file operations: 3 installs, 0 updates, 0 removals
  - Locking psr/log (3.0.2)
  - Locking monolog/monolog (3.7.0)
  - Locking acme/my-app (dev-main)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 3 installs, 0 updates, 0 removals
  - Installing psr/log (3.0.2): Extracting archive
  - Installing monolog/monolog (3.7.0): Extracting archive
Generating optimized autoload files
```

**Why:** The resolver reads `monolog/monolog: ^3.5`, fetches Monolog's metadata, discovers that Monolog 3.7.0 requires `psr/log: ^2.0 || ^3.0`, and selects `psr/log` 3.0.2 because it satisfies both the root constraint (no direct constraint on `psr/log`) and Monolog's constraint. The solver explores multiple candidate versions before settling on this combination.

**Example 2: A Resolution Conflict**

```json
{
    "name": "acme/conflicting-app",
    "require": {
        "package-a/package-a": "^1.0",
        "package-b/package-b": "^1.0"
    }
}
```

**Expected Output (when running `composer update`):**

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires package-a/package-a ^1.0, package-b/package-b ^1.0
    - package-a/package-a 1.0.0 requires psr/log ^1.0
    - package-b/package-b 1.0.0 requires psr/log ^2.0
    - Found psr/log[1.0.0, ..., 1.1.4] but the package is fixed to 2.0.0 by the lock file.
    - Root composer.json requires psr/log ^2.0, but package-a/package-a 1.0.0 requires psr/log ^1.0.
```

**Why:** Package A requires `psr/log: ^1.0` (versions 1.x), and Package B requires `psr/log: ^2.0` (versions 2.x). No single version of `psr/log` can satisfy both constraints. The solver detects the conflict and reports the exact chain of constraints that caused it, allowing the developer to adjust the requirements.

### Real-World Cases

**Case 1: Framework Dependency Convergence**

A Laravel application requires `laravel/framework: ^11.0`, `laravel/sanctum: ^4.0`, and `spatie/laravel-permission: ^6.0`. Each of these packages has its own dependencies on `illuminate/*` components at specific versions. The resolver must find a set of `illuminate/*` versions that satisfies the Laravel framework, Sanctum, and Spatie's permission package simultaneously. The SAT solver explores the intersection of all constraints and produces a locked set of versions.

**Case 2: Monorepo with Path Repositories**

A monorepo contains multiple packages (`acme/core`, `acme/admin`, `acme/api`) that depend on each other. The root `composer.json` declares path repositories. The resolver treats the local packages as first-class dependencies, and the SAT solver ensures that the local version constraints are compatible with any external packages that also depend on `acme/core`.

---

## Core Concept 2: Manifest vs. Lockfiles

### Definitions

**Core Definition**

`composer.json` is the human-authored manifest that declares which packages a project wants and the version constraints it accepts. `composer.lock` is the machine-generated record of the exact versions that were actually installed, including source references and integrity hashes.

**Technical Definition**

`composer.json` contains the project's metadata (`name`, `description`, `type`), its dependencies (`require`, `require-dev`), autoloading configuration, and script definitions. When `composer install` or `composer update` is run, Composer's resolver reads the constraints from `composer.json`, explores the dependency graph, selects concrete versions, and writes the fully resolved snapshot to `composer.lock`. The lock file stores each package's `version`, VCS source reference (commit hash), distribution URL, and `shasum` integrity hash. It also records a `content-hash` of the relevant parts of `composer.json`; if the manifest drifts out of sync with the lock file, Composer warns that the lock is out of date.

**Beginner-Friendly Explanation**

Think of `composer.json` as your shopping list ("I need milk, bread, and eggs — any brand is fine"). Think of `composer.lock` as the receipt from the store ("I bought Brand X milk at $3.99, Brand Y bread at $2.50, and Brand Z eggs at $4.00"). The next time you go shopping (`composer install`), you use the receipt to buy the exact same items, not whatever happens to be on the shelf today. This ensures that your code works the same way on every machine.

### Purposes

- To declare the project's direct dependencies with flexible version constraints in `composer.json`.
- To pin exact resolved versions for all dependencies (direct and transitive) in `composer.lock`.
- To ensure byte-for-byte identical dependency trees across development machines, CI runners, and production servers.
- To provide an auditable record of exactly which code is running in production, including commit hashes and integrity hashes.
- To enable reproducible builds by allowing `composer install` to bypass resolution entirely when a lock file is present.

### Syntax Rules and Structure

**Complete General Syntax: `composer.json`**

```json
{
    "name": "acme/blog",
    "type": "project",
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.5",
        "vlucas/phpdotenv": "^5.6"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0",
        "phpstan/phpstan": "^1.11"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "config": {
        "sort-packages": true
    }
}
```

**Complete General Syntax: `composer.lock` (Trimmed)**

```json
{
    "_readme": [
        "This file locks the dependencies of your project to a known state"
    ],
    "content-hash": "a3f5c9e2b1d47f80c6e2a9b4d5f81c30",
    "packages": [
        {
            "name": "monolog/monolog",
            "version": "3.7.0",
            "source": {
                "type": "git",
                "url": "https://github.com/Seldaek/monolog.git",
                "reference": "d4d80a2d115f9c9b41f28db0f76c5e2f3b6b7c2a"
            },
            "dist": {
                "type": "zip",
                "url": "https://api.github.com/repos/Seldaek/monolog/zipball/d4d80a2d",
                "shasum": "9f8e7d6c5b4a3f2e1d0c9b8a7654321fedcba098"
            },
            "require": {
                "php": ">=8.1",
                "psr/log": "^2.0 || ^3.0"
            }
        }
    ],
    "packages-dev": [],
    "platform": {
        "php": ">=8.1"
    },
    "plugin-api-version": "2.6.0"
}
```

**Component Breakdown:**

- `content-hash` — A hash of the relevant parts of `composer.json`. If the manifest is edited (e.g., a new dependency is added) without running `composer update`, the lock file is out of date and `composer install` warns.
- `packages` — Runtime dependencies (those in `require`). Each entry pins a `version`, a `source` (VCS reference), a `dist` (distribution archive with `shasum`), and the package's own `require` constraints.
- `packages-dev` — Development-only dependencies (those in `require-dev`). These are excluded when `composer install --no-dev` is run.
- `platform` — The PHP version constraint recorded in the lock file, derived from the `php` requirement in `composer.json`.
- `plugin-api-version` — The version of the Composer plugin API used during resolution.

**Syntax Rules:**

- `composer.json` is human-edited; `composer.lock` is machine-managed and should never be edited by hand.
- For applications, `composer.lock` must be committed to version control. For reusable libraries, it is conventionally ignored because the consuming application's lock file governs the final versions.
- `composer install` reads `composer.lock` and reproduces the exact versions. If no lock file exists, it falls back to resolving from `composer.json`.
- `composer update` re-resolves all dependencies, updates the lock file, and installs the new versions. It ignores the lock file during resolution.

**Constraints and Limitations:**

- **Lock file drift:** If a developer modifies `composer.json` but does not run `composer update`, the lock file becomes out of sync. `composer install` may warn or fail.
- **Merge conflicts in lock files:** `composer.lock` cannot be merged cleanly without conflicts because it contains hashes and references. Teams must resolve conflicts by running `composer update` after merging.
- **Stale locks and security:** A pinned lock file ensures reproducibility but may pin a version with a known vulnerability. Regular `composer update` (with security auditing) is necessary to receive patches.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Manifest with Flexible Constraints and Generated Lock File**

```bash
# Step 1: Create composer.json
cat > composer.json << 'EOF'
{
    "name": "acme/blog",
    "require": {
        "php": ">=8.2",
        "monolog/monolog": "^3.5",
        "vlucas/phpdotenv": "^5.6"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    }
}
EOF

# Step 2: Run composer install (resolves and creates lock)
composer install --no-interaction
```

**Expected Output:**

```
Loading composer repositories with package information
Updating dependencies
Lock file operations: 12 installs, 0 updates, 0 removals
  - Locking phpunit/phpunit (11.0.0)
  - Locking monolog/monolog (3.7.0)
  - Locking vlucas/phpdotenv (5.6.0)
  ... (9 more)
Writing lock file
Installing dependencies from lock file (including require-dev)
Generating optimized autoload files
```

**Why:** The resolver reads the constraints from `composer.json`, selects concrete versions for all direct and transitive dependencies, and writes them to `composer.lock`. The lock file now contains the exact versions, commit references, and integrity hashes.

**Example 2: Reproducible Installation from a Lock File**

```bash
# Step 1: Clone the repository (composer.lock is committed)
git clone https://github.com/acme/blog.git
cd blog

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

**Why:** `composer install` reads `composer.lock` and installs the exact versions recorded there, ignoring the version ranges in `composer.json`. The `--no-dev` flag excludes development dependencies, and `--optimize-autoloader` generates a classmap for production performance.

### Real-World Cases

**Case 1: CI/CD Reproducibility**

A CI pipeline runs `composer install --no-dev --optimize-autoloader` on every build. Because `composer.lock` is committed, every build uses the exact same dependency versions, eliminating "works on CI but not locally" discrepancies. If a developer adds a new package to `composer.json` without updating the lock, the CI build fails with a clear warning.

**Case 2: Library Development**

A reusable library (e.g., a PSR-7 HTTP client) commits `composer.json` but does not commit `composer.lock`. The library's dependencies are constraints, not pinned versions, because the consuming application's lock file is what ultimately governs the final installed versions.

**Case 3: Security Auditing**

A security team runs `composer audit` on a project's `composer.lock` to identify dependencies with known CVEs. Because the lock file records exact versions, the audit is precise and actionable. After updating vulnerable packages, `composer update` regenerates the lock file with patched versions.

---

## Core Concept 3: Semantic Versioning (SemVer)

### Definitions

**Core Definition**

Semantic Versioning (SemVer) is a version-numbering contract that uses a `MAJOR.MINOR.PATCH` format to communicate the nature of changes between releases, allowing dependency managers like Composer to resolve updates automatically while indicating whether an upgrade is safe.

**Technical Definition**

Under SemVer, the `MAJOR` version increments when incompatible API changes are made, `MINOR` when functionality is added in a backward-compatible manner, and `PATCH` when backward-compatible bug fixes are made. Composer translates version constraints expressed with operators (`^`, `~`, `*`, comparison operators) into ranges of acceptable versions. The caret operator (`^3.5`) allows non-breaking updates: `>=3.5.0 <4.0.0`. The tilde operator (`~3.5`) allows only patch updates when a minor is given: `>=3.5.0 <3.6.0`. The wildcard (`3.5.*`) matches any patch within the specified minor. A strict version (`3.5.2`) pins exactly one release.

**Beginner-Friendly Explanation**

Think of version numbers as a promise from the package maintainer. `1.2.3` means: "This is version 1 (major), the 2nd minor update, with 3 patches." The promise is: if you upgrade within version 1 (e.g., `1.5.0`), your code should still work. If you upgrade to version 2, the maintainer is warning you that something might break. Composer's operators let you say "give me any version 1.x" (`^1.0`), "give me only 1.2.x" (`~1.2.0`), or "give me exactly 1.2.3" (`1.2.3`).

### Purposes

- To communicate the backward-compatibility of a release through its version number.
- To allow dependency managers to automatically select the newest version that does not introduce breaking changes.
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

- The caret operator (`^`) is the preferred default for most dependencies because it respects SemVer's major version boundary.
- The tilde operator (`~`) is more restrictive when a patch digit is specified: `~1.2.3` allows only patch updates (`<1.3.0`), while `~1.2` allows minor updates (`<2.0.0`).
- Wildcards (`*`) should be used sparingly; they can match versions that introduce unexpected changes if the package maintainer does not follow SemVer strictly.
- Mixing comparison operators with wildcards (e.g., `>=2.*`) is invalid and causes Composer to throw an error.
- For reusable libraries, use permissive constraints (`^1.0`) to avoid restricting consumers. For applications, more restrictive constraints reduce the risk of unexpected changes.

**Constraints and Limitations:**

- **SemVer is a social contract, not a technical guarantee:** Packages that do not follow SemVer strictly can introduce breaking changes in minor or patch releases. Composer cannot prevent this.
- **Caret operator and major version 0:** In SemVer, `0.x` versions are considered unstable. Composer's caret operator on `^0.3` allows `>=0.3.0 <0.4.0` (only patch updates), reflecting the instability of pre-1.0 versions.
- **Open upper bounds (`>=3.5`):** Allowing any version above a threshold can silently jump major versions and break the application. Use `^` instead.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Demonstrating Caret vs. Tilde**

```bash
# Step 1: Create a project with a caret constraint
composer init --name=acme/caret-demo --require="monolog/monolog:^3.5" --no-interaction
composer install

# Step 2: Check the resolved version
composer show monolog/monolog | grep versions
```

**Expected Output:**

```
versions : * 3.7.0
```

```bash
# Step 3: Create a project with a tilde constraint
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

**Why this is dangerous:** If `vendor/package` releases version 2.0.0 with breaking changes, `composer update` will install it because the constraint `>=1.0` has no upper bound. The correct constraint is `^1.0` (allowing `>=1.0.0 <2.0.0`) or `~1.0` (allowing `>=1.0.0 <1.1.0`).

### Real-World Cases

**Case 1: Framework Dependency Constraints**

A Laravel application uses `"laravel/framework": "^11.0"`. This allows any Laravel 11.x release (11.0, 11.1, 11.2, etc.) but prevents accidental upgrades to Laravel 12.0, which may introduce breaking changes. The application receives bug fixes and minor features automatically.

**Case 2: Library Development with Permissive Constraints**

A PSR-7 implementation library declares `"psr/http-message": "^1.0 || ^2.0"` to indicate compatibility with both major versions of the PSR-7 interface. This allows the library to be used by applications that have locked either version, maximising compatibility.

**Case 3: Strict Pinning in Production**

A production deployment uses `composer install` with a committed lock file. Although `composer.json` declares `^3.5`, the lock file pins `3.7.0` exactly. Production never receives an unexpected update; updates are applied deliberately via `composer update` followed by testing.

---

## Core Concept 4: Global vs. Project Dependencies

### Definitions

**Core Definition**

Project dependencies are installed in the project's local `vendor/` directory and are isolated from other projects. Global dependencies are installed in a shared system directory and are available across all projects, primarily for CLI tools.

**Technical Definition**

`composer require` installs a package into the current project's `vendor/` directory, writes it to `composer.json`, and includes it in the project's autoloader. `composer global require` installs a package into the Composer home directory (typically `~/.composer` or `%APPDATA%\Composer` on Windows), where it is available as a CLI tool but is not autoloaded by any project. Global installs share the same pool of dependencies, meaning that updating one globally installed tool can affect other tools that depend on the same packages.

**Beginner-Friendly Explanation**

Think of project dependencies as the ingredients in your own kitchen — they are specific to the dish you are cooking and do not affect your neighbour's kitchen. Global dependencies are like a shared spice rack in a communal kitchen: anyone can use it, but if someone uses up the cumin or replaces it with a different brand, everyone's cooking is affected.

### Purposes

- To isolate project dependencies from each other, preventing version conflicts between projects.
- To ensure that every project has its own reproducible dependency tree via its lock file.
- To provide CLI tools (PHPUnit, PHP-CS-Fixer, Laravel Installer) that are accessible from any directory via global installation.
- To avoid polluting the system with per-project installations of developer tools.
- To enable multiple projects to use different versions of the same tool without conflict.

### Syntax Rules and Structure

**Complete General Syntax: Project Dependency**

```bash
composer require monolog/monolog
```

**Complete General Syntax: Global Dependency**

```bash
composer global require laravel/installer
```

**Component Breakdown:**

- `composer require` — Adds a package to the current project's `composer.json` and installs it into `./vendor/`. The package is available for autoloading within the project.
- `composer global require` — Adds a package to the global Composer configuration (typically `~/.composer/composer.json`) and installs it into the global `vendor/` directory. The package's binaries are available via the global `vendor/bin` directory, which should be added to the system `PATH`.

**Syntax Rules:**

- Project dependencies are declared in the project's `composer.json` and locked in its `composer.lock`.
- Global dependencies are declared in the global `composer.json` and are not part of any project's lock file.
- Global packages are not autoloaded by any project; they are only available as CLI tools.
- To use a globally installed package in a project, it must be installed locally as well.

**Constraints and Limitations:**

- **Dependency hell in global installs:** Because globally installed packages share the same dependency pool, updating one tool can break another tool that depends on a different version of a shared dependency.
- **Version inconsistency:** A developer may have a globally installed version of PHPUnit that differs from the version required by a project. Running the global PHPUnit on a project can produce misleading results.
- **Not reproducible:** Global installations are not recorded in any project's lock file, so they cannot be reproduced on CI or in production.
- **PATH configuration required:** Global binaries must be added to the system `PATH` to be accessible from any directory.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Project vs. Global Installation**

```bash
# Step 1: Install a package locally (project dependency)
cd /var/www/my-project
composer require --dev phpunit/phpunit
# This installs PHPUnit into /var/www/my-project/vendor/bin/phpunit

# Step 2: Install a package globally (CLI tool)
composer global require laravel/installer
# This installs the Laravel installer into ~/.composer/vendor/bin/laravel

# Step 3: Verify local installation
./vendor/bin/phpunit --version

# Step 4: Verify global installation (if ~/.composer/vendor/bin is in PATH)
laravel --version
```

**Expected Output:**

```
# Step 3 output:
PHPUnit 11.0.0 by Sebastian Bergmann and contributors.

# Step 4 output:
Laravel Installer 5.0.0
```

**Why:** PHPUnit is installed locally because it is a project-specific development tool whose version must match the project's requirements. The Laravel installer is installed globally because it is a CLI tool used to create new projects, not a runtime dependency of any existing project.

**Example 2: The Risk of Global Dependencies**

```bash
# Developer A installs a global tool
composer global require phpunit/phpunit:^11.0

# Developer B installs a different global tool that depends on an incompatible version
composer global require phpunit/phpunit:^10.0

# Composer's global dependency pool now has a conflict
```

**Expected Output:**

```
Your requirements could not be resolved to an installable set of packages.
  Problem 1
    - Root composer.json requires phpunit/phpunit ^10.0, but it is fixed to 11.0.0 by the lock file.
```

**Why:** Global installs share a single dependency pool. Installing two versions of PHPUnit globally creates a conflict because Composer cannot satisfy both constraints simultaneously. This is why project-local installations are recommended for anything that is not a standalone CLI tool.

### Real-World Cases

**Case 1: Laravel Installer**

The Laravel installer (`laravel/installer`) is commonly installed globally via `composer global require laravel/installer`. It is a standalone CLI tool that creates new Laravel projects. It does not need to be installed per-project because it is not a runtime dependency.

**Case 2: PHPUnit in CI**

A CI pipeline uses `composer install --dev` to install PHPUnit locally from the project's lock file. This ensures that the CI runner uses exactly the same PHPUnit version as the development team, preventing "works locally but fails in CI" issues caused by version mismatches.

**Case 3: PHP-CS-Fixer as a Project Tool**

A team installs PHP-CS-Fixer as a project-local development dependency (`composer require --dev friendsofphp/php-cs-fixer`) rather than globally. This ensures that every developer and CI runner uses the same version with the same rules, and the version is locked in `composer.lock`.

---

## Core Concept 5: Platform Requirements & Constraints

### Definitions

**Core Definition**

Platform requirements allow a project to declare dependencies on the PHP interpreter version, specific PHP extensions (`ext-*`), and system libraries (`lib-*`) directly in `composer.json`, ensuring that the runtime environment satisfies these requirements before the application runs.

**Technical Definition**

Composer treats the PHP interpreter, PHP extensions, and system libraries as virtual platform packages. The PHP interpreter is available as the `php` package (with subtypes `php-64bit`, `php-ipv6`, `php-zts`, `php-debug`); extensions are available as `ext-*` packages (e.g., `ext-mbstring`, `ext-pdo`); and libraries are available as `lib-*` packages (e.g., `lib-curl`). When a package requires one of these virtual packages, Composer checks the requirement against the current environment. The `config.platform` section allows overriding the detected platform values for the purposes of dependency resolution, simulating a target environment even if the current machine does not match.

**Beginner-Friendly Explanation**

Imagine you are ordering a custom computer online. You specify "I need Windows 11, a USB-C port, and 16 GB of RAM." The manufacturer checks your order against their available components before building the machine. Composer's platform requirements are the same idea: you tell Composer "this project needs PHP 8.2 or later and the `mbstring` and `pdo_mysql` extensions," and Composer checks whether your system has them before installing dependencies.

### Purposes

- To enforce a minimum PHP version for the application, preventing runtime errors on older interpreters.
- To ensure that required PHP extensions are enabled before the application runs, avoiding "Class 'PDO' not found" or "mbstring missing" errors.
- To allow CI and deployment environments to validate platform requirements before installing dependencies.
- To simulate a target production environment during dependency resolution (e.g., resolving for PHP 8.1 even when developing on PHP 8.3).
- To document the runtime requirements of an application in a machine-readable, enforceable format.

### Syntax Rules and Structure

**Complete General Syntax: Platform Requirements in `composer.json`**

```json
{
    "require": {
        "php": ">=8.2",
        "ext-mbstring": "*",
        "ext-pdo": "*",
        "ext-json": "*",
        "lib-curl": "*"
    },
    "config": {
        "platform": {
            "php": "8.2.15",
            "ext-mbstring": "8.2.15",
            "ext-pdo": "8.2.15"
        }
    }
}
```

**Component Breakdown:**

- `"php": ">=8.2"` — Requires PHP 8.2 or later. Composer checks this against the currently running PHP version.
- `"ext-mbstring": "*"` — Requires the `mbstring` extension to be enabled. The version value is typically `"*"` because most PHP extensions do not follow SemVer; the requirement is simply that the extension is present.
- `"ext-pdo": "*"` — Requires the PDO extension.
- `"lib-curl": "*"` — Requires the system `libcurl` library.
- `"config": { "platform": { ... } }` — Overrides the detected platform values for resolution purposes. This is useful when developing on PHP 8.3 but deploying to PHP 8.2, or when a required extension is not enabled locally but will be on the target server.

**Syntax Rules:**

- Extension names must match the names shown by `php -m` exactly (e.g., `mbstring`, not `mb_string`).
- In the `require` section, extension version constraints are conventionally `"*"` because PHP extensions do not have independent version numbers in most cases.
- The `config.platform` section is for resolution only; it does not install extensions or change the running PHP version. `composer install` will still fail if the actual environment does not satisfy the requirements (unless `--ignore-platform-reqs` is used).
- Use `composer show --platform` to see all available platform packages in the current environment.

**Constraints and Limitations:**

- **`--ignore-platform-reqs` bypasses checks:** Running `composer install --ignore-platform-reqs` skips platform requirement validation. This can lead to runtime failures if the environment is not actually compatible.
- **Platform config does not install extensions:** Setting `config.platform.ext-mbstring` does not enable the `mbstring` extension; it only tells the resolver to assume it is available. The actual extension must still be installed and enabled.
- **Extension version constraints are rarely useful:** Most PHP extensions report a version (often matching the PHP version or a fixed value like `1.0`), but this version is not meaningful for SemVer-based constraints. Use `"*"` unless you have a specific reason to constrain.
- **Development vs. production mismatch:** If `config.platform` specifies PHP 8.2 but the development machine runs PHP 8.3, the resolver will select packages compatible with 8.2. If a package requires a feature introduced in 8.3, it will be rejected even though the development machine could run it.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Declaring and Enforcing Platform Requirements**

```json
{
    "name": "acme/api",
    "require": {
        "php": ">=8.2",
        "ext-pdo": "*",
        "ext-pdo_mysql": "*",
        "ext-mbstring": "*",
        "ext-json": "*"
    }
}
```

```bash
# Step 1: Run composer install on a system with all extensions
composer install
```

**Expected Output:**

```
Loading composer repositories with package information
Updating dependencies
Nothing to install, update or remove
Generating optimized autoload files
```

```bash
# Step 2: Simulate a missing extension
php -d extension= -r 'echo "mbstring disabled\n";' 2>/dev/null
# Or run composer with --ignore-platform-req=ext-mbstring to see the error
composer install --ignore-platform-req=ext-mbstring
```

**Expected Output (without ignoring):**

```
Your requirements could not be resolved to an installable set of packages.
  Problem 1
    - Root composer.json requires ext-mbstring * -> it is missing from your system. Install or enable PHP's mbstring extension.
```

**Why:** Composer checks each `ext-*` requirement against the currently loaded extensions. If `mbstring` is not enabled, the installation fails with a clear message before any code is written. This prevents runtime errors such as `Call to undefined function mb_strlen()`.

**Example 2: Using `config.platform` to Simulate a Target Environment**

```json
{
    "name": "acme/legacy-migration",
    "require": {
        "php": ">=8.1",
        "ext-pdo": "*"
    },
    "config": {
        "platform": {
            "php": "8.1.27"
        }
    }
}
```

```bash
# Developer is running PHP 8.3 locally but deploying to PHP 8.1
composer update
```

**Expected Output:**

```
Loading composer repositories with package information
Updating dependencies
Resolving dependencies for platform php 8.1.27
Lock file operations: 5 installs, 0 updates, 0 removals
  - Locking vendor/package (2.1.0)  # Version compatible with PHP 8.1
Writing lock file
Generating optimized autoload files
```

**Why:** Even though the developer's machine runs PHP 8.3, the `config.platform` setting tells the resolver to assume PHP 8.1.27. The resolver selects package versions that are compatible with PHP 8.1, preventing the developer from accidentally installing a package that requires PHP 8.2+ and would fail in production.

### Real-World Cases

**Case 1: Requiring PDO and mbstring**

A web application requires `ext-pdo` (for database access) and `ext-mbstring` (for multibyte string handling). By declaring these in `composer.json`, the CI pipeline fails early if the extensions are not installed, rather than failing at runtime when the application tries to connect to the database or process Unicode text.

**Case 2: Simulating a Production PHP Version**

A team develops on PHP 8.3 but deploys to PHP 8.2. They set `config.platform.php` to `"8.2.15"` in `composer.json` to ensure that all resolved dependencies are compatible with PHP 8.2. This prevents the "it works on my machine" problem caused by inadvertently requiring a PHP 8.3-only feature.

**Case 3: Docker Build Validation**

A Dockerfile includes `composer install --no-dev --optimize-autoloader` as a build step. If the base PHP image lacks a required extension (e.g., `pdo_mysql`), the build fails immediately with a clear error, rather than producing an image that crashes at runtime.

---

## References

- Composer: Platform Dependencies — https://getcomposer.org/doc/articles/composer-platform-dependencies.md
- Composer: Version Constraints — https://getcomposer.org/doc/articles/versions.md
- Composer: The composer.json Schema — https://getcomposer.org/doc/04-schema.md
- Composer: Config — https://getcomposer.org/doc/06-config.md
- Composer: Runtime API — https://getcomposer.org/doc/07-runtime.md
- Secure PHP Development: composer.lock — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/composer-lock.md
- Secure PHP Development: Dependency Management — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Dependency-Management.md
- Secure PHP Development: Installing Composer — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/Installing-Composer.md
- SitePoint: Composer Global Require Considered Harmful? — https://www.sitepoint.com/composer-global-require-considered-harmful/
- Laravel News: What are the asterisk, tilde, and caret used for in Composer? — https://laravel-news.com/composer-version-constraints
- Stack Overflow: Version numbers with caret and tilde in composer.json — https://stackoverflow.com/questions/22362452/version-numbers-with-caret-and-tilde-in-composer-json
- EasyEngine: Composer Version Constraints — https://easyengine.io/tutorials/composer/version-constraints/
- Composer GitHub Issue #7630: Pool/Solver/Repo/Installer Tasks — https://github.com/composer/composer/issues/7630
- Composer GitHub Issue #7354: Dependency resolution on local packagist — https://github.com/composer/composer/issues/7354
- Composer: Why are version constraints combining comparisons and wildcards a bad idea? — https://getcomposer.org/doc/articles/versions.md