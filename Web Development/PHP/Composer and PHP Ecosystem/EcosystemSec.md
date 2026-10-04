# Ecosystem Security, Auditing, & Compliance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Ecosystem security, auditing, and compliance in PHP is the discipline of systematically verifying that a project's dependency graph — every direct and transitive package resolved by Composer — is free of known vulnerabilities, legally compatible with the project's licensing model, and internally consistent between its declarative manifest (`composer.json`) and its locked resolution (`composer.lock`).

**Technical Definition**

Composer provides three built-in or ecosystem-supported mechanisms for security and compliance verification: `composer audit`, which cross-references the exact versions pinned in `composer.lock` against the PHP Security Advisories Database and reports known CVEs and abandoned packages; license compliance tooling (both third-party and Composer-plugin-based), which scans dependency licenses and enforces allow/deny policies; and `composer validate --strict`, which verifies that the `content-hash` stored in `composer.lock` matches the relevant fields of `composer.json`, detecting drift between the manifest and the lock file.

**Beginner-Friendly Explanation**

Imagine you are assembling a product from hundreds of parts supplied by different vendors. **Vulnerability analysis** is the safety inspector who checks every part against a database of known defects. **License compliance checking** is the legal team that verifies you are allowed to use each part in your product. **Lockfile integrity** is the quality control clerk who confirms that the parts you ordered (the manifest) match the parts that were actually delivered and recorded (the lock file). Together, these three practices ensure that what you ship is safe, legal, and reproducible.

### Key Characteristics

- **Lockfile-Driven Analysis:** All three practices operate on `composer.lock`, not `composer.json`, because the lock file records the exact versions actually deployed.
- **CI/CD Gate Integration:** Each practice provides a non-zero exit code on failure, making it suitable as a build pipeline gate that blocks merges or deployments.
- **Full Transitive Coverage:** Vulnerability and license checks cover the entire dependency graph, including packages that were never directly required.
- **SPDX-Aware License Matching:** License compliance tools use SPDX identifiers and support wildcard patterns (e.g., `GPL-*`) for allow/deny policies.
- **Content-Hash Integrity:** The `content-hash` in `composer.lock` is a cryptographic digest of the relevant `composer.json` fields; a mismatch indicates the lock is stale.

### Prerequisites

- **Composer 2.4+** for `composer audit` (2.7+ for the `--abandoned` flag, 2.8+ for `--ignore-severity`).
- **A committed `composer.lock` file** — auditing and validation are meaningless without it.
- **A CI/CD pipeline** (GitHub Actions, GitLab CI, Jenkins) with PHP and Composer provisioned.
- **A license policy** defining which SPDX licenses are acceptable for the project (e.g., MIT, Apache-2.0, BSD-3-Clause).
- **Basic understanding of SemVer and Composer constraints** to interpret update advisories correctly.

### Related Programming Areas

- **Package Management:** Third-party packages, development dependencies, and SemVer constraints determine what enters the dependency graph.
- **PHP Composer Foundations & Dependency Architecture:** The resolver produces the lock file that all three practices audit.
- **Production Autoloading & Performance Optimization:** The optimized classmap is generated after dependencies are installed and verified.
- **Package Lifecycle, Scripting, & Automation:** Composer scripts can hook into install/update events to run audits and validations automatically.
- **CI/CD Pipeline Engineering:** Security gates, license gates, and lockfile validation are standard CI steps.

### Core Concepts / Features

1. **Vulnerability Analysis** — Running `composer audit` in CI/CD pipelines to scan installed dependencies for known, published security risks.
2. **License Compliance Checking** — Scanning and restricting project package licenses to ensure legal compatibility before shipping code to production.
3. **Lockfile Integrity & Freshness** — Utilizing automated linters (`composer validate`) to verify that `composer.json` and `composer.lock` remain perfectly synchronized.

---

## Core Concept 1: Vulnerability Analysis

### Definitions

**Core Definition**

Vulnerability analysis is the automated process of cross-referencing the exact package versions pinned in `composer.lock` against a database of published security advisories to identify dependencies with known CVEs or abandoned status.

**Technical Definition**

`composer audit`, built into Composer since version 2.4, reads the resolved dependency graph from `composer.lock` and queries the PHP Security Advisories Database, which aggregates advisories from the PHP security community and the GitHub Advisory Database. Because it works from the lock file, it covers the full transitive graph, not just direct requirements. Every advisory names the affected package, the vulnerable version range, and the linked CVE/GHSA identifier. The command exits with a non-zero code when at least one advisory is found, making it suitable as a CI gate. Composer 2.7 changed the default of the `audit.abandoned` config setting to `fail`, meaning abandoned packages now break the audit unless explicitly configured otherwise. Composer 2.8 added the `--abandoned` flag to override the config setting per-invocation, and the `--ignore-severity` flag to filter advisories by severity.

**Beginner-Friendly Explanation**

Think of `composer audit` as a health check for your project's dependencies. It looks at the exact list of packages you have installed (the lock file) and checks each one against a global database of known problems. If a package has a known security flaw, the audit tells you which package, which version, and what the flaw is. If a package's maintainer has stopped maintaining it (abandoned), the audit warns you too, because abandoned packages will never receive security fixes.

### Purposes

- To detect dependencies with known security vulnerabilities before they reach production.
- To surface abandoned packages that will not receive future security patches.
- To provide a CI-friendly gate that fails the build when vulnerabilities are found.
- To cover the full transitive dependency graph, including packages never directly required.
- To shift security checks left, flagging vulnerabilities in the same command that introduces them.

### Syntax Rules and Structure

**Complete General Syntax: Running a Dependency Audit**

```bash
composer audit [options]
```

**Component Breakdown:**

- `composer audit` — Runs the audit against `composer.lock`. Reports all known vulnerabilities in installed packages.
- `--format=<plain|table|json|summary>` — Output format. `table` is the default for humans; `json` is for tooling.
- `--locked` — Audits `composer.lock` even if `vendor/` is not installed. This is the default source anyway.
- `--no-dev` — Ignores `require-dev` packages, matching a production installation.
- `--abandoned=<report|fail|ignore>` — Controls how abandoned packages are treated (Composer 2.8+). `fail` makes an abandoned dependency break the audit.
- `--ignore-severity` — Ignores advisories of one or more severity levels (Composer 2.8+).
- `--ignore-unreachable` — Allows running audit in environments without access to some repositories (Composer 2.9+).

**Complete General Syntax: Automatic Auditing During Install/Update**

```bash
# Auditing runs automatically after update
composer update

# Disable automatic auditing
composer update --no-audit

# Run an audit as part of install
composer install --audit
```

**Exit Codes:**

| Code | Meaning |
|---|---|
| `0` | No advisories found |
| `1` | Vulnerabilities found |
| `2` | Abandoned packages found (Composer 2.8.4+) |
| `3` | Both vulnerabilities and abandoned packages found |

**Syntax Rules:**

- `composer audit` exits with a non-zero code when at least one advisory is found. This makes it usable as a CI gate.
- Automatic auditing runs by default after `composer update` and can be overridden with `--no-audit`.
- The `audit.abandoned` config setting controls whether abandoned packages are reported (`report`), ignored (`ignore`), or treated as failures (`fail`). Composer 2.7 changed the default to `fail`.
- The `audit > block-insecure` config setting controls whether updates to package versions with known security advisories are blocked (defaults to `true`).

**Constraints and Limitations:**

- **No reachability analysis:** `composer audit` tells you a vulnerable version is installed. It cannot tell you whether your application actually invokes the vulnerable class or method.
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

**Example 2: CI Pipeline with Audit Gate (GitHub Actions)**

```yaml
name: Security Audit
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 06:00 UTC

jobs:
  composer-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          tools: composer
      - name: Install dependencies
        run: composer install --no-interaction --no-progress
      - name: Run composer audit
        run: composer audit --format=json | tee audit-results.json
      - name: Upload audit results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: composer-audit
          path: audit-results.json
```

**Expected Output (when no advisories are found):**

```
No security vulnerability advisories found.
```

**Why:** The pipeline runs `composer audit` on every push, pull request, and on a weekly schedule. The scheduled run ensures that newly disclosed advisories in already-shipped code surface even when no one is changing dependencies. The JSON output is uploaded as an artifact for further analysis.

**Example 3: Auditing with Abandoned Package Handling**

```bash
# Report abandoned packages without failing
composer audit --abandoned=report

# Fail the audit if any abandoned package is found
composer audit --abandoned=fail
```

**Expected Output (with abandoned packages, `--abandoned=report`):**

```
Found 1 abandoned package:

+---------------------+----------------------------------------------------------+
| Package             | vendor/legacy-lib                                        |
| Abandoned           | Yes                                                      |
| Replacement         | vendor/new-lib                                           |
+---------------------+----------------------------------------------------------+
```

**Why:** The `--abandoned` flag overrides the `audit.abandoned` config setting. `report` lists abandoned packages without failing; `fail` makes an abandoned dependency break the audit. In a strict pipeline, `--abandoned=fail` forces a conversation about replacing dead dependencies before they become a security problem.

### Real-World Cases

**Case 1: Scheduled Nightly Audits**

A security team runs `composer audit --locked --format=json` as a nightly cron job against the default branch. Even when no one changes dependencies, newly disclosed advisories in already-shipped code surface within 24 hours.

**Case 2: Automatic Auditing During Development**

A developer runs `composer update` to add a new package. Composer automatically audits the newly resolved dependencies and reports any advisories in the same terminal session. The developer sees the vulnerability immediately, before committing the change.

**Case 3: Laravel Warden Integration**

A Laravel application uses the Warden package, which performs security audits on Composer dependencies and provides automated notifications for vulnerabilities. Warden integrates with CI pipelines to send reports with affected packages, affected versions, and more from the `composer audit` command.

---

## Core Concept 2: License Compliance Checking

### Definitions

**Core Definition**

License compliance checking is the automated process of scanning a project's dependency licenses and enforcing a policy that ensures every package's license is legally compatible with the project's intended use and distribution model.

**Technical Definition**

License compliance tooling reads the license metadata from `composer.lock` (or the installed packages' `composer.json` files) and compares each package's declared license against a configured allow-list or deny-list. Tools like `lendable/composer-license-checker` provide a configuration file (`.allowed-licenses.php`) where developers declare acceptable licenses using SPDX identifiers (e.g., `MIT`, `Apache-2.0`, `BSD-3-Clause`) and can allow specific vendors or packages regardless of license. `dominikb/composer-license-checker` offers both `check` and `report` commands with `--allowlist` and `--blocklist` options. `k2gl/composer-license-gate` is a Composer plugin that checks licenses against an allow/deny policy during the install/update lifecycle and can fail the install on a violation when in `enforce` mode.

**Beginner-Friendly Explanation**

Think of license compliance as a legal checkpoint for your project's parts. Every open-source package comes with a license that says what you are allowed to do with it — some are permissive (MIT, Apache-2.0), and some are restrictive (GPL, AGPL). If you are building a proprietary product, using a GPL-licensed package could legally require you to open-source your entire product. License compliance tooling scans every package you depend on and checks whether its license is on your company's approved list. If it finds a problem, it tells you before you ship.

### Purposes

- To ensure that all dependency licenses are legally compatible with the project's distribution model.
- To prevent restrictive licenses (e.g., GPL, AGPL) from entering a proprietary codebase.
- To provide a CI-friendly gate that fails the build when a non-compliant license is detected.
- To maintain an auditable record of every dependency's license for legal and compliance reviews.
- To automate the detection of dependencies with no declared license (which defaults to "all rights reserved" under copyright law).

### Syntax Rules and Structure

**Complete General Syntax: Configuring `lendable/composer-license-checker`**

```php
<?php
// .allowed-licenses.php
declare(strict_types=1);

use Lendable\ComposerLicenseChecker\LicenseConfigurationBuilder;

return (new LicenseConfigurationBuilder())
    ->addLicenses(
        'MIT',
        'BSD-2-Clause',
        'BSD-3-Clause',
        'Apache-2.0',
    )
    ->addAllowedVendor('acme')          // Allow any license from your own company
    ->addAllowedPackage('acme/legacy')  // Allow a specific package regardless of licensing
    ->build();
```

```bash
# Run the checker
./vendor/bin/composer-license-checker
```

**Complete General Syntax: Configuring `dominikb/composer-license-checker`**

```bash
# Check against an allow-list and deny-list
./vendor/bin/composer-license-checker check \
    --allowlist MIT \
    --allowlist Apache-2.0 \
    --blocklist GPL \
    --blocklist AGPL

# Generate a report of all licenses
./vendor/bin/composer-license-checker report
```

**Complete General Syntax: Configuring `k2gl/composer-license-gate` (Composer Plugin)**

```json
{
    "extra": {
        "k2gl-license-gate": {
            "mode": "enforce",
            "allow": ["MIT", "Apache-2.0", "BSD-2-Clause", "BSD-3-Clause", "ISC"],
            "deny": ["GPL-*", "AGPL-*"],
            "allow-packages": ["acme/legacy-thing"],
            "require-license": false
        }
    }
}
```

**Component Breakdown:**

- `mode` — `warn` (default, report violations but don't stop), `enforce` (fail the install on violation), or `off`.
- `allow` — An allow-list of SPDX license identifiers. If set, a package is accepted only when every declared license matches one of these patterns. Takes precedence over `deny`.
- `deny` — A deny-list. A package is rejected if any declared license matches one of these patterns.
- `allow-packages` — Package names exempt from the check, for dependencies that have been reviewed and accepted.
- `require-license` — Treat a package that declares no license as a violation.
- Patterns are SPDX identifiers, matched case-insensitively, with a trailing `*` as a prefix wildcard (e.g., `GPL-*` matches `GPL-2.0-only`, `GPL-3.0-or-later`).

**Syntax Rules:**

- License identifiers must use SPDX format (e.g., `MIT`, `Apache-2.0`, `BSD-3-Clause`, `GPL-3.0-or-later`).
- Dual-licensed packages (e.g., `MIT OR GPL-3.0`) are read conservatively: if any of the licenses is disallowed, the package is flagged. Accept it deliberately with `allow-packages` once you have confirmed you are using it under the allowed option.
- The Composer plugin (`k2gl/composer-license-gate`) runs inside Composer's install/update lifecycle, so violations are caught automatically in local installs and CI without a separate command.
- Standalone checkers (`lendable`, `dominikb`) are invoked manually or as CI steps.

**Constraints and Limitations:**

- **No license means "all rights reserved":** Packages without a declared license in `composer.json` default to a restrictive legal position. `require-license: true` in `k2gl/composer-license-gate` treats these as violations.
- **SPDX accuracy:** Some packages declare non-standard license strings. The checker may not map them correctly, requiring manual review.
- **Legal interpretation:** License compliance tooling identifies licenses; it does not provide legal advice. A lawyer should interpret the results for your specific use case.
- **Transitive dependencies:** License checks cover the full dependency graph, including transitive packages that the developer may not be aware of.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Setting Up `lendable/composer-license-checker`**

```bash
# Step 1: Install the checker as a development dependency
composer require --dev lendable/composer-license-checker
```

**Expected Output:**

```
./composer.json has been updated
Running composer update lendable/composer-license-checker
...
  - Locking lendable/composer-license-checker (1.0.0)
Writing lock file
```

```php
<?php
// Step 2: Create .allowed-licenses.php in the project root
declare(strict_types=1);

use Lendable\ComposerLicenseChecker\LicenseConfigurationBuilder;

return (new LicenseConfigurationBuilder())
    ->addLicenses('MIT', 'BSD-2-Clause', 'BSD-3-Clause', 'Apache-2.0')
    ->addAllowedVendor('acme')
    ->build();
```

```bash
# Step 3: Run the checker
./vendor/bin/composer-license-checker
```

**Expected Output (when all licenses are compliant):**

```
All licenses are compliant.
```

**Why:** The checker reads `composer.lock`, extracts each package's license, and compares it against the allow-list. If every package's license is in the allow-list (or belongs to an allowed vendor), the checker exits with code 0. Otherwise, it reports the offending packages and exits with a non-zero code.

**Example 2: CI Pipeline with License Gate**

```yaml
- name: License Compliance Check
  run: |
    ./vendor/bin/composer-license-checker
  # Exits non-zero if any license violates the policy
```

**Expected Output (when a violation is found):**

```
Found 1 package with disallowed license:
  - vendor/gpl-package: GPL-3.0-or-later (not in allow-list)
```

**Why:** The CI step runs the license checker after installation. If a dependency has a license outside the allow-list, the command exits non-zero and the pipeline fails, preventing the non-compliant code from being merged or deployed.

**Example 3: Composer Plugin License Gate (`k2gl/composer-license-gate`)**

```bash
# Step 1: Install the plugin
composer require --dev k2gl/composer-license-gate
```

```json
// Step 2: Configure the policy in composer.json
{
    "extra": {
        "k2gl-license-gate": {
            "mode": "enforce",
            "allow": ["MIT", "Apache-2.0", "BSD-2-Clause", "BSD-3-Clause", "ISC"],
            "deny": ["GPL-*", "AGPL-*"],
            "allow-packages": ["acme/legacy-thing"],
            "require-license": true
        }
    }
}
```

```bash
# Step 3: Run composer install — the gate runs automatically
composer install
```

**Expected Output (when a violation is found):**

```
! license policy: acme/widget — license "GPL-3.0-or-later" is denied by policy
Your requirements could not be resolved to an installable set of packages.
```

**Why:** The plugin hooks into Composer's install/update lifecycle. When a package's license violates the policy in `enforce` mode, the install is aborted with a clear error message. This provides the earliest possible feedback — the violation is caught during installation, not after.

### Real-World Cases

**Case 1: Proprietary SaaS Product**

A company building a proprietary SaaS product uses `lendable/composer-license-checker` with an allow-list of only permissive licenses (MIT, Apache-2.0, BSD). When a developer attempts to add a GPL-licensed package, the CI pipeline fails, preventing the restrictive license from entering the codebase and potentially requiring the company to open-source its entire product.

**Case 2: Open-Source Project with Compatibility Requirements**

An open-source project distributed under the MIT license needs to ensure all dependencies are compatible with MIT. The project uses `dominikb/composer-license-checker` with `--allowlist MIT` to verify that every dependency can be redistributed under MIT-compatible terms.

**Case 3: Enterprise Compliance Audit**

An enterprise prepares for a legal compliance audit. The compliance team runs `lendable/composer-license-checker` and exports a report of every dependency's license. The report is included in the audit documentation, providing evidence that all third-party code was reviewed and approved.

---

## Core Concept 3: Lockfile Integrity & Freshness

### Definitions

**Core Definition**

Lockfile integrity and freshness is the practice of verifying that `composer.lock` is synchronized with `composer.json` and that the locked dependency graph is internally consistent, using `composer validate --strict` as an automated check.

**Technical Definition**

Composer stores a `content-hash` field in `composer.lock` that is a cryptographic digest of the relevant fields of `composer.json` (including `require`, `require-dev`, `autoload`, and other sections that affect dependency resolution). When `composer.json` is edited without running `composer update`, the `content-hash` in the lock file no longer matches, and `composer validate` reports the lock as out of date. `composer validate --strict` enforces this check as a non-zero exit code, making it suitable for CI gates. The `--no-check-lock` option bypasses the lock file check, and `--no-check-all` reduces the strictness of the validation.

**Beginner-Friendly Explanation**

Think of `composer.json` as your shopping list and `composer.lock` as the receipt from the store. The `content-hash` is like a fingerprint of the shopping list. If you change the list (add an item, remove an item) but do not go back to the store to get a new receipt, the fingerprint on the receipt no longer matches the list. `composer validate --strict` checks whether the fingerprint matches. If it does not, it tells you: "Your list has changed, but your receipt is old. Go to the store and get a new receipt" — which means running `composer update` to regenerate the lock file.

### Purposes

- To detect drift between `composer.json` and `composer.lock` before it causes installation failures.
- To ensure that every environment installs the same dependency versions by verifying the lock file is current.
- To prevent developers from committing changes to `composer.json` without regenerating the lock file.
- To provide a CI-friendly gate that fails the build when the lock file is stale.
- To validate the structural correctness of both `composer.json` and `composer.lock`.

### Syntax Rules and Structure

**Complete General Syntax: Running `composer validate`**

```bash
# Basic validation
composer validate

# Strict validation (recommended for CI)
composer validate --strict

# Skip lock file check
composer validate --no-check-lock

# Skip all checks (only validate composer.json syntax)
composer validate --no-check-all
```

**Component Breakdown:**

- `composer validate` — Checks that `composer.json` is syntactically valid and that `composer.lock` is in sync with it. Reports warnings and errors.
- `--strict` — Treats warnings as errors and exits with a non-zero code if any issues are found. This is the recommended mode for CI pipelines.
- `--no-check-lock` — Skips the lock file synchronization check. Useful when validating `composer.json` alone.
- `--no-check-all` — Disables all optional checks, performing only basic syntax validation.
- `--no-check-publish` — Skips checks related to publishing packages to Packagist (useful for applications, not libraries).

**Exit Codes:**

| Code | Meaning |
|---|---|
| `0` | `composer.json` is valid, and `composer.lock` is in sync |
| `1` | `composer.json` is invalid |
| `2` | `composer.lock` is out of date or invalid |

**Syntax Rules:**

- `composer validate` should be run before committing `composer.json` and before tagging a release.
- The lock file freshness check is based on the `content-hash` stored in `composer.lock`.
- `composer validate --strict` catches the classic drift where someone edited `composer.json` and forgot to update the lock file.
- Deploy with `composer install --no-dev --prefer-dist --no-scripts` and always run `composer validate --strict` as part of the deployment pipeline.

**Constraints and Limitations:**

- **`--strict` treats warnings as errors:** This is appropriate for CI but may be overly aggressive for development environments where minor warnings (e.g., missing `description`) are acceptable.
- **Content-hash does not detect semantic changes:** The hash only detects changes to the relevant fields of `composer.json`. It does not detect changes in the environment or in the package repository.
- **Lock file merge conflicts:** `composer.lock` cannot be merged cleanly without conflicts. Teams must resolve conflicts by running `composer update` after merging.
- **Platform differences:** If `composer.lock` was generated on a different platform (e.g., PHP 8.2 vs. PHP 8.3), `composer validate` may report platform-related issues.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Detecting Lock File Drift**

```bash
# Step 1: Start with a valid project
composer install

# Step 2: Edit composer.json without running composer update
# (e.g., add a new dependency to the "require" section)

# Step 3: Run composer validate
composer validate
```

**Expected Output (when the lock is out of date):**

```
./composer.json is valid, but with a few warnings
See https://getcomposer.org/doc/04-schema.md for details on the schema
The lock file is not up to date with the latest changes in composer.json, it is recommended that you run `composer update`.
```

**Why:** The `content-hash` stored in `composer.lock` no longer matches the relevant fields of `composer.json`. `composer validate` detects this mismatch and warns that the lock file is out of date.

**Example 2: CI Pipeline with Strict Validation Gate**

```yaml
name: Validate Composer Files
on:
  pull_request:
    paths:
      - 'composer.json'
      - 'composer.lock'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          tools: composer
      - name: Validate composer.json and composer.lock
        run: composer validate --strict
```

**Expected Output (when valid):**

```
./composer.json is valid
```

**Expected Output (when invalid):**

```
./composer.json is valid, but with a few warnings
See https://getcomposer.org/doc/04-schema.md for details on the schema
The lock file is not up to date with the latest changes in composer.json, it is recommended that you run `composer update`.
Error: Process completed with exit code 1.
```

**Why:** The CI step runs `composer validate --strict` on every pull request that touches `composer.json` or `composer.lock`. If the lock file is out of date, the build fails with exit code 1, forcing the developer to run `composer update` and commit the updated lock file.

**Example 3: Comparing `--strict` vs. Non-Strict Validation**

```bash
# Non-strict: warnings do not cause failure
composer validate
# Exit code: 0 (unless there are errors)

# Strict: warnings are treated as errors
composer validate --strict
# Exit code: 1 (if any warning is present)
```

**Why:** In development, `composer validate` (non-strict) is useful for catching syntax errors without failing on warnings. In CI, `--strict` ensures that even minor issues (e.g., missing `description`, unbound version constraints) are caught and fixed before merge.

### Real-World Cases

**Case 1: Drupal's Composer Validation**

Drupal's CI pipeline runs `composer validate` on all contributed modules and themes. When a module's `composer.json` is edited, the validation job checks that the lock file is in sync and that the manifest follows the expected schema. This prevents broken dependency declarations from reaching Drupal.org's package repository.

**Case 2: Leantime Lock File Synchronization**

The Leantime project's pull request history shows a fix for a `content-hash` mismatch in `composer.lock`. The lock file was out of sync with `composer.json`, and the CI validation job caught the drift. The fix was a single-line change to the `content-hash` field, restoring synchronization.

**Case 3: Deployment Pipeline with Strict Validation**

A deployment pipeline runs `composer validate --strict` as the first step after checkout. If the lock file is stale, the pipeline fails immediately, before any dependencies are installed. This prevents deployments with inconsistent dependency graphs and ensures that the exact versions recorded in the lock file are what is deployed.

---

## References

- webconsulting-skills: CI Security Pipeline for PHP Projects — https://github.com/dirnbauer/webconsulting-skills/blob/main/skills/security-audit/references/ci-security-pipeline.md
- Safeguard.sh: Auditing PHP Dependencies with composer audit — https://safeguard.sh/resources/blog/composer-audit-php-dependencies
- Armour Infosec: Composer Audit in CI — https://github.com/armourinfosec/Secure-PHP-Development/blob/586b887138a30c25fc459684c78dffe0a2296c17/CI-CD/Composer-Audit-in-CI.md
- Packagist Blog: Discover Security Advisories with Composer's audit command — https://blog.packagist.com/discover-security-advisories-with-composers-audit-command/
- Composer Changelog 2.8.0 — https://getcomposer.org/changelog/2.8.0
- Composer Changelog 2.7.0 — https://getcomposer.org/changelog/2.7.0
- Lendable: composer-license-checker — https://packagist.org/packages/lendable/composer-license-checker
- Dominikb: composer-license-checker — https://packagist.org/packages/dominikb/composer-license-checker
- k2gl: composer-license-gate — https://packagist.org/packages/k2gl/composer-license-gate
- Safeguard.sh: PHP Composer Security — Lockfiles & Abandoned Packages — https://safeguard.sh/resources/blog/php-composer-security-lockfiles-packagist-and-abandoned-packages
- Armour Infosec: composer.lock — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Composer/composer-lock.md
- Composer: validate command — https://getcomposer.org/doc/03-cli.md#validate
- Packagist: cs278/composer-audit — https://packagist.org/packages/cs278/composer-audit
- Packagist: koeker/composer-audit-guard — https://packagist.org/packages/koeker/composer-audit-guard
- Packagist: selfphp/composer-license-audit — https://packagist.org/packages/selfphp/composer-license-audit