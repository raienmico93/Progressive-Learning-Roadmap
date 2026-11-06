# Express.js Dependency Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Dependency security in Express.js is the discipline of identifying, monitoring, and mitigating security vulnerabilities in the third-party npm packages that an application depends on—both directly and transitively—to prevent supply-chain breaches, known CVE exploitation, and license compliance issues.

**Technical Definition**
A typical Express.js application declares a few dozen direct dependencies but installs **400–700 transitive packages** through `node_modules`, creating a vast software composition analysis (SCA) attack surface. Dependency security encompasses **vulnerability scanning** (`npm audit`, Snyk, OSV-Scanner), **automated update management** (Dependabot, Renovate), **lockfile integrity enforcement** (`npm ci`), and **supply-chain hardening** (pinning versions, auditing before adding packages, restricting install scripts). Over **80% of codebases contain at least one vulnerability** through open-source dependencies, and Express's own security posture is heavily dependent on companion packages like `send`, `serve-static`, `qs`, `path-to-regexp`, `body-parser`, and `cookie`.

**Beginner-Friendly Explanation**
Every Express app is built on top of hundreds of other people's code. That code is usually safe, but sometimes it has bugs—or worse, an attacker sneaks malicious code into a popular package. Dependency security is how you keep watch: you run scanners to find known problems, you set up bots that automatically propose fixes, and you make sure you're installing exactly the code you reviewed—not a compromised version that appeared overnight.

---

### Key Characteristics

- **Transitive depth is the biggest risk** — You are responsible for packages you never chose and have never heard of; a CVE three levels deep still affects your app.
- **`npm audit` is the baseline, not the complete solution** — It is built-in and free, but it only reports *presence*, not *reachability*; most flagged vulnerabilities are never actually called by your code.
- **Supply-chain attacks target the install step** — Typosquatting, dependency confusion, and maintainer account takeovers are documented npm attack vectors that CVE scanners may miss until an advisory is filed.
- **Lockfile integrity is non-negotiable** — `package-lock.json` records exact versions and SHA-512 integrity hashes; `npm ci` enforces them and fails on tampering.
- **Automation closes the gap** — Dependabot and Snyk provide continuous monitoring and fix PRs, turning a periodic manual task into a standing control.

---

### Prerequisites

- Node.js 18 or higher and an Express application with `package.json` and a committed `package-lock.json`.
- npm 8+ (for `npm audit` and lockfile v2/v3 support).
- A GitHub repository (for Dependabot) and/or a Snyk account (for `snyk test`).
- A CI/CD pipeline (GitHub Actions, GitLab CI, Jenkins) to automate scanning.
- Basic understanding of semantic versioning (major, minor, patch) and lockfiles.

---

### Related Programming Areas

- **Software Composition Analysis (SCA)** — The broader category of tools that scan dependencies for vulnerabilities and license issues.
- **Supply chain security** — Protecting against malicious packages, typosquatting, and account takeovers.
- **CI/CD security** — Gating builds on vulnerability thresholds.
- **Lockfile management** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` integrity.
- **Runtime security** — Express version floors, Helmet, and secure configuration complement dependency scanning.

---

## Core Concept 1: Dependency Auditing

### Definitions

**Core Definition**
Dependency auditing is the process of inspecting an application's resolved dependency tree—including transitive dependencies—against vulnerability databases to identify packages with known security flaws.

**Technical Definition**
Dependency auditing works by building the full resolved dependency graph from the lockfile and comparing each package name and version against advisory databases such as the GitHub Advisory Database (used by `npm audit`) or Snyk's proprietary vulnerability database. Auditing must inspect the **resolved tree**, not just top-level `package.json` declarations, because most vulnerabilities live in transitive dependencies. Modern SCA tools also check for license compliance, malicious package behavior, and reachability of vulnerable code paths.

**Beginner-Friendly Explanation**
Auditing is the process of checking every package in your app—including the ones you didn't directly install—against a database of known problems. It's like a background check for your dependencies.

---

### Purposes

- To identify known vulnerabilities (CVEs) in direct and transitive dependencies before they can be exploited.
- To satisfy compliance and security-audit requirements for open-source software usage.
- To prioritize remediation by severity and exploitability.
- To catch license compliance issues (e.g., GPL dependencies in commercial software).

---

### Syntax Rules and Structure

#### Complete General Syntax: The Auditing Stack

| Tool | Type | Coverage | Cost |
|------|------|----------|------|
| `npm audit` | Built-in CLI | npm advisory database | Free |
| Snyk Open Source | Commercial SCA | Deep transitive analysis, reachability, fix PRs | Free tier + paid |
| OSV-Scanner | Open-source CLI | Broader advisory coverage | Free |
| Socket | Supply-chain SCA | Detects malicious/compromised packages by behavior | Free tier + paid |
| Dependabot | GitHub-native | Automated alerts and security update PRs | Free for GitHub |

#### Syntax Rules

- **Audit the resolved tree, not just `package.json`** — transitive dependencies are where most vulnerabilities hide.
- **Run auditing on every pull request** in CI, not just periodically.
- **Use `--omit=dev`** to exclude development-only dependencies when assessing production risk.
- **Set a severity floor** (`--audit-level=high`) so low-severity noise does not fail builds.

#### Constraints and Limitations

- `npm audit` only knows what the advisory database knows; zero-days, malicious install scripts, and freshly published compromised versions are invisible until an advisory is filed.
- It reports presence, not reachability; most flagged vulnerabilities are never actually called by your code.
- Dev-dependency noise can obscure production-relevant findings unless filtered.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Running a Baseline Audit

```bash
# Install dependencies
npm ci

# Run the audit
npm audit

# Gate the pipeline at high severity
npm audit --audit-level=high
```

**Expected Output**

```
# npm audit report

axios  <0.21.1
Severity: high
Axios Cross-Site Request Forgery - https://github.com/advisories/GHSA-cph5-m8f7-6c5x
fix available via `npm audit fix`
node_modules/axios

3 high severity vulnerabilities

To address all issues, run:
  npm audit fix
```

**Why This Output Occurs**

- `npm audit` reads `package-lock.json`, builds the resolved tree, and queries the npm advisory endpoint; it reports each vulnerable package with severity and a fix path.

---

### Real-World Cases

- **Express.js applications** — A typical Express app pulls in hundreds of transitive dependencies; auditing the resolved tree surfaces CVEs in packages like `lodash` or `qs` that you never directly installed.
- **Commercial software** — License compliance scanning catches GPL dependencies that a pure CVE scanner would miss.
- **Compliance audits** — Demonstrating continuous SCA scanning is a common requirement for SOC 2 and ISO 27001.

---

## Core Concept 2: `npm audit`

### Definitions

**Core Definition**
`npm audit` is the built-in npm command that submits the project's resolved dependency tree to the npm registry's audit endpoint and reports known vulnerabilities from the GitHub Advisory Database.

**Technical Definition**
`npm audit` reads `package-lock.json` (or `npm-shrinkwrap.json`), builds the full resolved dependency graph, and sends the list of package names and versions to the configured registry's audit endpoint. The registry compares these versions against the GitHub Advisory Database and returns vulnerabilities with severity, vulnerable path, and remediation availability. The command exits with a **non-zero code** if any vulnerability is found, making it suitable for CI gating; `--audit-level` changes the failure threshold.

**Beginner-Friendly Explanation**
`npm audit` is the scanner that comes free with npm. You run it in your project folder, and it tells you which of your installed packages have known security holes and what versions fix them.

---

### Purposes

- To provide a zero-cost, zero-configuration vulnerability scan in every Node.js project.
- To gate CI pipelines on high-severity vulnerabilities.
- To automatically apply safe (semver-respecting) fixes with `npm audit fix`.
- To support structured JSON output for integration with other tools.

---

### Syntax Rules and Structure

#### Complete General Syntax: Core Commands

```bash
npm audit                          # human-readable report
npm audit --json                   # machine-readable JSON
npm audit --audit-level=high       # fail only on high+ severity
npm audit --omit=dev               # exclude dev dependencies
npm audit fix                      # auto-fix within semver
npm audit fix --force              # force major bumps (risky)
npm audit fix --dry-run            # preview without applying
npm audit signatures               # verify registry signatures
```

**Component Breakdown**

| Command | Purpose |
|---------|---------|
| `npm audit` | Baseline scan against the advisory database |
| `--audit-level=high` | Set the CI failure threshold |
| `--omit=dev` | Exclude development-only dependencies |
| `npm audit fix` | Apply safe fixes respecting semver ranges |
| `npm audit fix --force` | Force breaking major upgrades (opt-in risk) |
| `npm audit signatures` | Verify registry signatures and provenance attestations |

#### Syntax Rules

- `npm audit` requires a lockfile; bypassing with `--no-package-lock` produces inconsistent results across runs.
- `npm audit fix` runs a full `npm install` under the hood; all installer configs apply.
- `--force` may introduce breaking changes; always preview with `--dry-run` first.
- Use `overrides` in `package.json` to pin transitive dependencies that have no upstream fix.

#### Constraints and Limitations

- Only reports **presence**, not **reachability**; most flagged vulnerabilities are never actually called.
- Cannot detect zero-days, malicious install scripts, or typosquats until an advisory is filed.
- Dev-dependency noise can drown out production-relevant findings unless `--omit=dev` is used.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete CI Audit Workflow

```yaml
# .github/workflows/security.yml
name: Security Scan
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0' # weekly

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - name: Run security audit
        run: npm audit --audit-level=high --omit=dev
```

**Expected Output**

```
# CI output
found 0 vulnerabilities
# or, if high-severity vulnerabilities exist:
npm ERR! audit Found 2 high severity vulnerabilities
Process completed with exit code 1.
```

**Why This Output Occurs**

- `npm ci` installs exactly the lockfile; `npm audit --audit-level=high --omit=dev` fails the build only when a production-relevant high-severity vulnerability is present, keeping the gate actionable.

---

#### Example 2: Using `overrides` for an Unfixed Transitive Dependency

```json
{
  "overrides": {
    "lodash": "4.17.21"
  }
}
```

```bash
npm install
npm audit
```

**Expected Output**

```
found 0 vulnerabilities
```

**Why This Output Occurs**

- The `overrides` field forces npm to resolve `lodash` to the patched version regardless of what intermediate packages request, eliminating the transitive vulnerability.

---

### Real-World Cases

- **Every Node.js CI pipeline** — `npm audit` is the zero-cost first gate recommended for all Node.js projects.
- **Express.js applications** — Auditing catches vulnerable `qs`, `path-to-regexp`, and `body-parser` versions that affect Express's security posture.
- **Compliance reporting** — JSON output (`npm audit --json`) feeds into dashboards and compliance artifacts.

---

## Core Concept 3: Snyk

### Definitions

**Core Definition**
Snyk is a commercial SCA platform that provides deep transitive dependency analysis, reachability analysis, license checks, and automated fix pull requests for Node.js and other ecosystems.

**Technical Definition**
Snyk Open Source uses a proprietary vulnerability database and resolves the full dependency graph—not just the lockfile text—to identify vulnerabilities with upgrade paths. Its **reachability analysis** identifies which vulnerable code paths are actually called by the application, reducing noise compared to presence-only scanners. The Snyk CLI (`snyk test`) runs dependency scanning from the project root; `snyk monitor` snapshots the dependency tree to the Snyk web UI for continuous alerting when new advisories affect already-shipped versions. The `--fail-on=upgradable` flag gates builds only when a fix actually exists.

**Beginner-Friendly Explanation**
Snyk is a more advanced scanner than `npm audit`. It tells you not just “this package has a known bug,” but “this bug is in code your app never calls, so you can deprioritize it”—or, conversely, “this bug is in a function your app calls on every request, so fix it now.” It also opens pull requests with the fix.

---

### Purposes

- To provide deeper transitive dependency analysis than `npm audit`.
- To reduce alert fatigue through reachability analysis.
- To automate fix PRs and continuous monitoring.
- To check license compliance across the dependency tree.

---

### Syntax Rules and Structure

#### Complete General Syntax: Snyk CLI

```bash
# Install
npm install -g snyk

# Authenticate (local)
snyk auth

# CI authentication via token
export SNYK_TOKEN=...

# Dependency scan
snyk test

# Fail only on high/critical
snyk test --severity-threshold=high

# Fail only when a fix exists
snyk test --fail-on=upgradable

# Machine-readable output
snyk test --json > results.json

# Monitor for new advisories (continuous)
snyk monitor
```

**Component Breakdown**

| Command | Purpose |
|---------|---------|
| `snyk test` | Dependency vulnerability scan |
| `--severity-threshold=high` | Only fail on high/critical |
| `--fail-on=upgradable` | Only fail when a fix exists |
| `snyk monitor` | Snapshot for continuous alerting |
| `snyk code test` | SAST on your own source code |

#### Syntax Rules

- Authenticate in CI via `SNYK_TOKEN`; never bake tokens into images or scripts.
- Use the standalone binary or official Snyk Docker image in CI rather than installing Node just for npm.
- `snyk test` tests the **resolved graph**, not the lockfile text; for Node.js it uses the installed tree.
- Run both `snyk test` (gate) and `snyk monitor` (tripwire) in production pipelines.

#### Constraints and Limitations

- Consumes a test from the plan quota on every invocation; free-tier monthly caps apply.
- Requires authentication and network access to Snyk's service.
- `--all-projects` in monorepos changes exit-code semantics; any failing subproject fails the run.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: CI Snyk Gate

```yaml
# .github/workflows/snyk.yml
name: Snyk Security Scan
on: [push, pull_request]

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --fail-on=upgradable
```

**Expected Output**

```
# Snyk test results
✓ Tested 347 dependencies for known issues
✗ Found 1 vulnerable dependency
  - lodash@4.17.20 (high severity)
    Upgrade to lodash@4.17.21 to fix
```

**Why This Output Occurs**

- Snyk resolves the full dependency graph, reports the vulnerable package with its severity and upgrade path, and fails the build because a fix is available and the severity meets the threshold.

---

### Real-World Cases

- **Enterprises with strict compliance** — Snyk's license checks and reachability analysis satisfy procurement and security-review requirements.
- **Monorepos** — `snyk test --all-projects` walks subdirectories and gates the entire repository.
- **Continuous monitoring** — `snyk monitor` alerts when a new advisory affects an already-deployed version.

---

## Core Concept 4: Dependabot

### Definitions

**Core Definition**
Dependabot is GitHub's native automated dependency update service that monitors repositories for vulnerable or outdated dependencies and opens pull requests to update them.

**Technical Definition**
Dependabot provides two core functions: **Dependabot alerts** (detect vulnerable dependencies based on the GitHub Advisory Database) and **Dependabot updates** (automatically create PRs to bump versions). **Security updates** are created immediately regardless of the configured schedule, while **version updates** follow the schedule in `.github/dependabot.yml`. The configuration file specifies the package ecosystem (`npm`), directory, schedule interval, PR limits, labels, reviewers, and grouping rules for minor/patch updates.

**Beginner-Friendly Explanation**
Dependabot is a bot that lives in your GitHub repository. When one of your dependencies gets a security fix, Dependabot immediately opens a pull request with the updated version. You review it, merge it, and you're patched.

---

### Purposes

- To automate the patching of vulnerable dependencies with minimal human effort.
- To keep dependencies current and reduce long-term upgrade debt.
- To provide security alerts directly in the GitHub interface.
- To reduce noise by grouping minor and patch updates into single PRs.

---

### Syntax Rules and Structure

#### Complete General Syntax: `.github/dependabot.yml`

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"
    reviewers:
      - "security-team"
    groups:
      production-minor-patch:
        patterns:
          - "*"
        exclude-patterns:
          - "@types/*"
          - "eslint*"
          - "typescript"
        update-types:
          - "minor"
          - "patch"
      typescript-ecosystem:
        patterns:
          - "typescript"
          - "@types/*"
          - "eslint*"
        update-types:
          - "minor"
          - "patch"
    ignore:
      - dependency-name: "express"
        update-types: ["version-update:semver-major"]
```

**Component Breakdown**

| Setting | Purpose |
|---------|---------|
| `package-ecosystem: "npm"` | Targets npm dependencies |
| `directory: "/"` | Root `package.json` and lockfile |
| `schedule.interval` | Version update cadence (security updates are immediate) |
| `open-pull-requests-limit` | Caps concurrent PRs to avoid noise |
| `groups` | Batch minor/patch updates into a single PR |
| `ignore` | Exclude specific dependency or update types |

#### Syntax Rules

- **Security updates are created immediately** regardless of the schedule; version updates follow the schedule.
- **Group minor and patch updates** to reduce PR noise; keep major updates separate for careful review.
- **Ignore major updates for critical packages** (e.g., Express) until you have a migration plan.
- **Limit Node.js `@types/node` to LTS (even) versions** to avoid churn from odd-numbered Current releases.

#### Constraints and Limitations

- Dependabot only covers GitHub repositories; it does not scan non-GitHub CI environments.
- It relies on the GitHub Advisory Database; proprietary or niche advisories may lag.
- Security update PRs still require review and merge; they are not automatically applied.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Grouped Dependabot Configuration

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 10
    groups:
      dev-dependencies:
        patterns:
          - "@types/*"
          - "eslint*"
          - "prettier*"
        update-types:
          - "minor"
          - "patch"
    ignore:
      - dependency-name: "express"
        update-types: ["version-update:semver-major"]
```

**Expected Output**

```
# Dependabot opens a single PR:
# "chore(deps): bump the dev-dependencies group with 5 updates"
# Instead of 5 separate PRs for @types/node, eslint, prettier, etc.
```

**Why This Output Occurs**

- The `groups` configuration batches all matching minor/patch dev-dependency updates into one PR, reducing review overhead.

---

### Real-World Cases

- **All GitHub-hosted Express apps** — Zero-config security updates for vulnerable dependencies.
- **Monorepos** — Multiple `updates` entries for different directories and ecosystems (npm, Docker, GitHub Actions).
- **Compliance** — Dependabot alerts in the Security tab provide an auditable trail of vulnerability detection and remediation.

---

## Core Concept 5: Dependency Security Best Practices

### Definitions

**Core Definition**
Dependency security best practices are the operational, configuration, and workflow patterns that minimize supply-chain risk across the full dependency lifecycle—from evaluation and installation through monitoring and remediation.

**Technical Definition**
Best practices include: **auditing before adding** a new dependency; **committing the lockfile** and installing with `npm ci` in CI; **running `npm audit` and SCA scans on every pull request** with a severity gate; **enabling Dependabot or Renovate** for automated updates; **pinning production dependencies** to exact or narrow ranges; **minimizing dependencies** to shrink the attack surface; **keeping Express and its companion packages patched**; and **restricting install scripts** where possible.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your dependency chain healthy. Don't install random packages without checking them. Always commit your lockfile and use `npm ci` so CI installs exactly what you reviewed. Run scanners automatically. Let a bot handle the routine updates. And keep your dependency count small.

---

### Purposes

- To prevent supply-chain attacks and known-CVE exploitation.
- To ensure reproducible, tamper-evident builds.
- To reduce alert fatigue through automated, grouped updates.
- To maintain a current, auditable dependency posture.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Dependency Pipeline

```yaml
# 1. Commit the lockfile
# 2. Install with npm ci in CI
# 3. Audit on every PR
# 4. Enable Dependabot
# 5. Pin Express to a maintained version

# CI step
- run: npm ci
- run: npm audit --audit-level=high --omit=dev
- run: snyk test --severity-threshold=high --fail-on=upgradable
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Commit lockfile | `package-lock.json` in version control |
| Enforce lockfile | `npm ci` in CI, not `npm install` |
| Automated audit | `npm audit --audit-level=high` on every PR |
| Deep SCA | `snyk test --fail-on=upgradable` |
| Automated updates | `.github/dependabot.yml` |
| Express version floor | 4.21.2+ or 5.x |
| Minimize dependencies | Evaluate necessity before adding |

#### Syntax Rules

- **Commit the lockfile** — it records exact versions and SHA-512 integrity hashes; `npm ci` verifies every hash and fails on mismatch.
- **Use `npm ci`, never `npm install`, in CI** — `npm install` can silently rewrite the lockfile and introduce unreviewed code.
- **Audit before adding** — run `npm audit` on a fresh install of the candidate package in isolation before merging it into your project.
- **Keep Express on a maintained version** — 4.21.2+ or 5.x; older versions are unsupported and carry known CVEs.
- **Enable Dependabot security alerts and security updates** — they are created immediately and require no schedule.
- **Pin production dependencies** to exact versions or narrow ranges; use `overrides` for unfixed transitive packages.
- **Disable install scripts by default** where possible (`npm ci --ignore-scripts`) to block a common supply-chain attack vector.

#### Constraints and Limitations

- Pinning every dependency to exact versions creates upgrade debt; balance pinning with automated updates.
- `--ignore-scripts` can break packages that legitimately need build steps; test in a staging environment.
- No scanner replaces code review; `npm audit` and Snyk are aids, not guarantees.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Hardened CI Pipeline

```yaml
name: Dependency Security
on:
  pull_request:
  schedule:
    - cron: '0 6 * * 1' # weekly Monday 6am

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - name: Install with lockfile enforcement
        run: npm ci
      - name: npm audit (high+ only, production only)
        run: npm audit --audit-level=high --omit=dev
      - name: Snyk SCA gate
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --fail-on=upgradable
      - name: Verify registry signatures
        run: npm audit signatures
```

**Expected Output**

```
✓ npm ci: installed exactly from lockfile
✓ npm audit: found 0 high vulnerabilities in production dependencies
✓ Snyk: 0 upgradable high-severity vulnerabilities
✓ npm audit signatures: all packages verified
```

**Why This Output Occurs**

- `npm ci` enforces the lockfile; `npm audit --omit=dev` filters dev noise; Snyk adds reachability-aware deep scanning; `npm audit signatures` verifies registry provenance, closing the supply-chain gap.

---

### Real-World Cases

- **Production Express APIs** — Lockfile enforcement, audit gating, and Dependabot security updates form the baseline.
- **Compliance-driven organizations** — Snyk and OSV-Scanner provide the audit trail and license reporting required for SOC 2 and ISO 27001.
- **High-security environments** — Pinning, install-script restriction, and registry signature verification defend against supply-chain tampering.

---

## References

- npm audit Documentation — https://docs.npmjs.com/cli/v11/commands/npm-audit
- npm audit Complete Guide (2026) — https://safeguard.sh/resources/blog/npm-audit-complete-guide
- Best SCA Tools for Node.js (2026) — https://appsecsanta.com/sca-tools/sca-tools-for-nodejs
- Snyk CLI: `snyk test` vs `snyk code test` — https://safeguard.sh/resources/blog/snyk-test-and-snyk-code-test-guide
- Dependabot Configuration (GitHub) — https://github.com/podverse/podverse/blob/main/docs/repo-management/DEPENDABOT.md
- Dependabot 配置指南 — https://github.com/Harzva/everything-in-github/blob/main/tutorials/security/dependabot.md
- Express Dependency Security (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/8-express-security/11-express-dependency-security.mdx
- Express.js Security Guide (2026) — https://safeguard.sh/resources/blog/express-js-security-guide
- package-lock.json Integrity and npm ci — https://safeguard.sh/resources/blog/package-lockjson-integrity-checks-and-the-npm-audit-workflow
- npm Lockfile Security Best Practices (2026) — https://safeguard.sh/resources/blog/npm-lockfile-security
- OWASP Node.js Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html