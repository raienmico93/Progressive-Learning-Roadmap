# Dependency Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dependency security is the discipline of protecting applications from vulnerabilities, compromises, and malicious code introduced through third-party packages in the software supply chain.

**Technical Definition:** Dependency security encompasses the practices, tools, and controls used to manage the risk of third-party dependencies. It includes dependency auditing (static vulnerability detection in the dependency tree), lock files (deterministic, checksum-verified builds), vulnerability scanning (third-party threat analysis), automated updates (Dependabot, Renovate), supply-chain security (integrity verification, maintainer compromise detection), package provenance (cryptographic attestations of build origin), and runtime guardrails (execution-time interception of malicious behaviour). The discipline aligns with SLSA (Supply-chain Levels for Software Artifacts), NIST SP 800-161, and the OWASP Top 10 (A06:2021 — Vulnerable and Outdated Components; A08:2021 — Software and Data Integrity Failures).

**Beginner-Friendly Explanation:** Think of your application as a house built from LEGO bricks. You buy bricks from thousands of different sellers (npm packages). Most are legitimate, but some are defective (vulnerabilities), and a few are booby-trapped (malicious packages). Dependency security is the set of practices that ensures every brick is genuine, unmodified, and safe to use — and that you know immediately if a seller is compromised or a brick is defective. It's the security of your supply chain.

### Key Characteristics

- **Transitive by nature:** Most dependencies are indirect — you depend on packages that depend on other packages, sometimes hundreds deep.
- **Ecosystem-specific:** npm, pnpm, and Yarn each have distinct lock files and auditing tools.
- **Continuous:** New vulnerabilities are discovered daily; dependency security is an ongoing process.
- **Multi-layered:** Auditing, lock files, scanning, updates, provenance, and runtime guardrails each address different risks.
- **Standards-aligned:** SLSA, NIST SP 800-161, OWASP A06/A08.
- **Automated:** Modern tools (Dependabot, Renovate, Socket) automate much of the work.

### Prerequisites

- **Node.js fundamentals:** `package.json`, `npm`, `pnpm`, `yarn`.
- **Lock files:** `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`.
- **Semantic versioning:** Major, minor, patch; `^`, `~`, exact.
- **Supply chain concepts:** Provenance, attestation, SBOM.
- **CI/CD pipelines:** GitHub Actions, GitLab CI, Jenkins.
- **Cryptography basics:** Signatures, checksums, transparency logs.

### Related Programming Areas

- **Web security:** Vulnerable dependencies are a top attack vector.
- **Node.js-specific security:** npm ecosystem risks.
- **DevSecOps:** SAST, DAST, SCA, secret scanning.
- **Compliance:** SOC 2, PCI DSS, ISO 27001, NIST.
- **Container security:** Base image scanning, SBOM generation.

### Core Concepts

1. **Dependency Auditing** — running static security checks using `npm audit`, `pnpm audit`, or `yarn audit`.
2. **Lock Files** — enforcing deterministic, checksum-verified builds via `package-lock.json`, `pnpm-lock.yaml`, or `yarn.lock`.
3. **Vulnerability Scanning** — integrating third-party threat analysis tools like Snyk or OWASP Dependency-Check.
4. **Dependency Updates** — automating continuous infrastructure upgrades using Dependabot or Renovate.
5. **Supply-Chain Security** — verifying package integrity and monitoring downstream maintainer account compromises.
6. **Package Provenance & Signing** — verifying cryptographically signed attestations.
7. **Automated Runtime Guardrails** — leveraging execution-time scanning tools like Socket.

---

## Core Concept 1: Dependency Auditing

### Definitions

**Core Definition:** Dependency auditing is the process of scanning the dependency tree for known vulnerabilities using built-in package manager tools.

**Technical Definition:** Dependency auditing compares the installed dependency tree against a vulnerability database (GitHub Advisory Database, npm Advisory Database) and reports packages with known CVEs. Tools include `npm audit`, `pnpm audit`, `yarn audit`, and `bun audit`. Audits return severity levels (low, moderate, high, critical), affected packages, remediation advice, and dependency paths. Auditing must be integrated into CI/CD to fail builds on high/critical vulnerabilities. Auditing detects *known* vulnerabilities only — it does not detect zero-days, malicious packages, or logic flaws.

**Beginner-Friendly Explanation:** Dependency auditing is like a health check for your LEGO bricks. You run a scanner, and it tells you "this brick has a known defect (CVE-2024-1234), and it came from this seller." You can then decide whether to replace it, patch it, or accept the risk.

### Purposes

- To detect known vulnerabilities in direct and transitive dependencies.
- To prioritise remediation based on severity and exploitability.
- To comply with security standards and policies.
- To fail CI/CD builds on high/critical vulnerabilities.
- To track remediation progress over time.

### Syntax Rules and Structure

#### Audit Commands

| Tool | Command | Notes |
|------|---------|-------|
| **npm** | `npm audit` | Built-in; uses npm Advisory Database |
| **npm** | `npm audit --production` | Ignore dev dependencies |
| **npm** | `npm audit --audit-level=high` | Fail on high or critical |
| **npm** | `npm audit fix` | Auto-fix compatible updates |
| **npm** | `npm audit fix --force` | Force breaking updates |
| **pnpm** | `pnpm audit` | Built-in |
| **pnpm** | `pnpm audit --prod` | Production only |
| **yarn** | `yarn audit` | Yarn 1.x |
| **yarn** | `yarn npm audit` | Yarn 2+/Berry |

#### Audit Output Format (JSON)

```json
{
  "auditReportVersion": 2,
  "vulnerabilities": {
    "lodash": {
      "name": "lodash",
      "severity": "high",
      "isDirect": false,
      "via": [
        {
          "source": "GHSA-35jh-r3h4-6jhm",
          "name": "lodash",
          "dependency": "lodash",
          "title": "Command Injection in lodash",
          "url": "https://github.com/advisories/GHSA-35jh-r3h4-6jhm",
          "severity": "high",
          "cvss": { "score": 7.2, "vectorString": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N" },
          "range": "<4.17.21"
        }
      ],
      "effects": ["my-app"],
      "range": "<4.17.21",
      "nodes": ["node_modules/lodash"],
      "fixAvailable": { "name": "lodash", "version": "4.17.21", "isSemVerMajor": false }
    }
  },
  "metadata": {
    "vulnerabilities": { "info": 0, "low": 0, "moderate": 0, "high": 1, "critical": 0, "total": 1 },
    "dependencies": { "prod": 245, "dev": 512, "optional": 42, "peer": 8, "total": 807 }
  }
}
```

#### Syntax Rules

- **Run `npm audit` regularly** — in CI and locally.
- **Use `--production`** — ignore dev-only vulnerabilities.
- **Use `--audit-level=high`** — fail builds on high/critical.
- **Review before fixing** — `npm audit fix` may introduce breaking changes.
- **Use `npm audit fix --force` with caution** — may break the app.
- **Pin versions in `package.json`** — avoid `^` for critical dependencies.
- **Use `overrides`** — force transitive dependencies to safe versions.
- **Document accepted risks** — vulnerabilities that cannot be fixed.
- **Monitor continuously** — daily audits in CI.
- **Use `--json`** — for programmatic processing.

#### Constraints and Limitations

- **Only detects known vulnerabilities** — zero-days and malicious packages are missed.
- **False positives** — some advisories may not apply to your usage.
- **False negatives** — the database may lag behind disclosures.
- **`npm audit fix` can break the app** — always test.
- **Transitive dependencies** — may be difficult to fix without upstream updates.
- **No context** — the audit does not know if the vulnerable code path is used.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: CI Audit Workflow (GitHub Actions)

```yaml
# .github/workflows/audit.yml
name: Dependency Audit
on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 6 * * *' # Daily at 6 AM UTC

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Audit production dependencies
        run: npm audit --production --audit-level=high

      - name: Audit JSON report
        if: always()
        run: npm audit --json > audit-report.json || true

      - name: Upload audit report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: audit-report
          path: audit-report.json
```

```json
// package.json — overrides for transitive vulnerabilities
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "audit": "npm audit --production --audit-level=high",
    "audit:fix": "npm audit fix",
    "audit:json": "npm audit --json"
  },
  "overrides": {
    "lodash": "^4.17.21",
    "semver": "^7.6.0",
    "minimist": "^1.2.8"
  }
}
```

**Expected behaviour:** CI fails if high/critical vulnerabilities are found. The JSON report is uploaded as an artifact for review. `overrides` force transitive dependencies to safe versions.

**Why this works:** The audit runs on every push and daily. `--production` ignores dev-only issues. `--audit-level=high` fails the build. `overrides` fix transitive vulnerabilities that cannot be patched upstream.

### Real-World Cases

- **Log4Shell (2021):** A critical vulnerability in Log4j was detected by audits.
- **Prototype pollution in lodash:** Audits detect vulnerable versions.
- **event-stream (2018):** Malicious package — audits would not have detected it (no CVE).
- **ReDoS in `path-to-regexp`:** Detected by audits.

---

## Core Concept 2: Lock Files

### Definitions

**Core Definition:** A lock file records the exact versions and integrity hashes of every dependency (direct and transitive), ensuring deterministic, reproducible builds.

**Technical Definition:** Lock files (`package-lock.json` for npm, `pnpm-lock.yaml` for pnpm, `yarn.lock` for Yarn) pin the resolved version and integrity hash (SHA-512) of every package in the dependency tree. They enable `npm ci` (clean install) to install exactly the same versions on every machine. Lock files must be committed to version control and updated intentionally (via `npm install <package>` or `npm update`). They prevent "works on my machine" issues and ensure that the same code is deployed everywhere.

**Beginner-Friendly Explanation:** A lock file is like a recipe with exact measurements. `package.json` says "I need flour" — the lock file says "I need 250g of King Arthur bread flour, lot #12345, from this exact mill." Without the lock file, everyone might use a different flour. With it, everyone uses the exact same one — ensuring the cake (your app) turns out the same every time.

### Purposes

- To ensure deterministic, reproducible builds.
- To prevent supply chain attacks via version substitution.
- To verify package integrity via checksums.
- To enable `npm ci` for fast, clean installs.
- To document the exact dependency tree for auditing.

### Syntax Rules and Structure

#### Lock File Comparison

| Tool | Lock File | Integrity Hash |
|------|-----------|----------------|
| **npm** | `package-lock.json` | SHA-512 |
| **pnpm** | `pnpm-lock.yaml` | SHA-512 |
| **Yarn 1.x** | `yarn.lock` | SHA-1/SHA-512 |
| **Yarn 2+ (Berry)** | `yarn.lock` | SHA-512 |
| **Bun** | `bun.lockb` | SHA-512 |

#### package-lock.json Structure (excerpt)

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "my-app",
      "version": "1.0.0",
      "dependencies": {
        "express": "^4.19.2"
      }
    },
    "node_modules/express": {
      "version": "4.19.2",
      "resolved": "https://registry.npmjs.org/express/-/express-4.19.2.tgz",
      "integrity": "sha512-5T6nhjsT+EOMzuck8JjBHARTHfMht0POzlA60WV2pMD3gyXw2LZnZ+ueGdNxzzxUhp5Zp6n4F0uN6eQ2dOcx3Q==",
      "dependencies": {
        "accepts": "~1.3.8"
      }
    }
  }
}
```

#### Syntax Rules

- **Always commit the lock file** — never add to `.gitignore`.
- **Always use `npm ci` in CI/CD** — never `npm install`.
- **Never edit the lock file manually** — use package manager commands.
- **Update intentionally** — `npm update` or `npm install <package>@<version>`.
- **Review lock file changes in PRs** — detect unexpected version changes.
- **Use `npm ci` for reproducibility** — it deletes `node_modules` and installs exactly.
- **Use `npm ci --ignore-scripts`** — prevent malicious install scripts.
- **Verify integrity** — `npm ci` checks SHA-512 hashes.
- **Use a registry proxy** — Artifactory, Verdaccio, or npm Enterprise.
- **Monitor lock file changes** — unexpected changes may indicate an attack.

#### Constraints and Limitations

- **Lock files can be large** — thousands of lines for large projects.
- **Merge conflicts** — lock files are hard to merge manually.
- **Platform differences** — optional dependencies may differ per platform.
- **Lock files can become stale** — require periodic updates.
- **`npm ci` deletes `node_modules`** — slower than `npm install`.
- **Lock file version changes** — npm upgrades may change the format.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reproducible Builds with npm ci

```yaml
# .github/workflows/build.yml
name: Build
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Verify lock file is in sync
        run: |
          npm ci
          if ! git diff --quiet package-lock.json; then
            echo "::error::package-lock.json is out of sync. Run 'npm install' and commit."
            exit 1
          fi

      - name: Install dependencies (deterministic)
        run: npm ci --ignore-scripts

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

```bash
# Local workflow — update dependencies safely
# 1. Check for outdated packages
npm outdated

# 2. Update a specific package
npm install express@4.19.2

# 3. Review the lock file diff
git diff package-lock.json

# 4. Run tests
npm test

# 5. Commit the change
git add package.json package-lock.json
git commit -m "chore(deps): update express to 4.19.2"
```

**Expected behaviour:**
- CI fails if `package-lock.json` is out of sync with `package.json`.
- `npm ci` installs exactly the locked versions.
- `--ignore-scripts` prevents malicious install scripts.
- Tests verify the build works with the locked versions.

**Why this works:** Lock files ensure deterministic builds. `npm ci` enforces the lock file. `--ignore-scripts` prevents a common attack vector. The sync check prevents drift between `package.json` and the lock file.

### Real-World Cases

- **event-stream (2018):** A malicious `flatmap-stream` dependency was added; lock files would have pinned the safe version.
- **ua-parser-js (2021):** Compromised versions could be avoided with locked, reviewed dependencies.
- **Dependency confusion:** Lock files and scoped registries prevent accidental public installs.

---

## Core Concept 3: Vulnerability Scanning

### Definitions

**Core Definition:** Vulnerability scanning is the use of third-party tools to analyse dependencies for known vulnerabilities, licence issues, and security risks beyond what built-in audits detect.

**Technical Definition:** Vulnerability scanning tools (Snyk, OWASP Dependency-Check, Trivy, Grype, GitHub Advisory Database) analyse the dependency tree against multiple vulnerability databases (NVD, GitHub Advisory, OSV). They provide: severity ratings, exploitability scores (CVSS), remediation advice, licence compliance, and fix PRs. Tools differ in coverage, speed, false-positive rates, and integration capabilities. Best practice: use multiple tools (defence in depth) and integrate them into CI/CD with severity thresholds.

**Beginner-Friendly Explanation:** Vulnerability scanning is like having multiple doctors examine your LEGO bricks. Each doctor has a different set of medical knowledge (vulnerability databases). By consulting multiple doctors, you catch more defects. Some doctors can even write prescriptions (automated fix PRs) to replace defective bricks.

### Purposes

- To detect vulnerabilities that `npm audit` may miss.
- To provide exploitability context (CVSS, EPSS).
- To detect licence compliance issues.
- To automate fix PRs.
- To integrate with CI/CD and IDE.
- To provide SBOM generation.

### Syntax Rules and Structure

#### Tool Comparison

| Tool | Type | Databases | Fix PRs | Licence Scanning |
|------|------|-----------|---------|------------------|
| **Snyk** | Commercial (free tier) | Snyk DB, NVD, GitHub | ✅ Yes | ✅ Yes |
| **OWASP Dependency-Check** | Open source | NVD | ❌ No | ⚠️ Partial |
| **Trivy** | Open source | NVD, OSV, GitHub | ❌ No | ✅ Yes |
| **Grype** | Open source | NVD, OSV | ❌ No | ❌ No |
| **GitHub Advisory** | Free | GitHub Advisory DB | ✅ Dependabot | ❌ No |
| **Socket** | Commercial | Behavioural analysis | ✅ Yes | ❌ No |
| **npm audit** | Built-in | npm Advisory DB | ✅ Yes | ❌ No |

#### Snyk CLI Usage

```bash
# Install
npm install -g snyk

# Authenticate
snyk auth

# Test for vulnerabilities
snyk test

# Test with severity threshold
snyk test --severity-threshold=high

# Monitor (continuous)
snyk monitor

# Fix (creates PRs)
snyk fix

# Test container image
snyk container test my-app:latest

# Generate SBOM
snyk sbom --format=cyclonedx1.4+json > sbom.json
```

#### Trivy Usage

```bash
# Scan filesystem
trivy fs --severity HIGH,CRITICAL .

# Scan container image
trivy image my-app:latest

# Scan with SBOM output
trivy fs --format cyclonedx --output sbom.json .

# Scan with ignore file
trivy fs --ignorefile .trivyignore .
```

#### Syntax Rules

- **Use multiple scanners** — different databases catch different issues.
- **Fail CI on high/critical** — enforce a severity threshold.
- **Generate SBOMs** — for compliance and incident response.
- **Monitor continuously** — not just at build time.
- **Review false positives** — do not blindly accept or dismiss.
- **Prioritise by exploitability** — CVSS, EPSS, and reachability.
- **Integrate with IDEs** — catch issues during development.
- **Scan containers** — base images contain vulnerabilities.
- **Document accepted risks** — with justification and expiry.
- **Automate fix PRs** — Snyk, Dependabot, Renovate.

#### Constraints and Limitations

- **False positives** — tools may flag unused code paths.
- **False negatives** — no tool catches everything.
- **Database lag** — new CVEs may take days to appear.
- **Cost** — commercial tools charge per developer or per project.
- **Performance** — scanning large projects can be slow.
- **Licence scanning is not vulnerability scanning** — both matter.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Tool Scanning in CI (GitHub Actions)

```yaml
# .github/workflows/scan.yml
name: Vulnerability Scan
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --all-projects

      - name: Snyk Monitor (continuous)
        if: github.ref == 'refs/heads/main'
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          command: monitor

  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy filesystem scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'HIGH,CRITICAL'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      - name: Generate SBOM
        run: npx @cyclonedx/cyclonedx-npm --output-file sbom.json

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json
```

**Expected behaviour:** Snyk fails the build on high/critical vulnerabilities. Trivy scans the filesystem and uploads results to GitHub Security. An SBOM is generated and uploaded for compliance.

**Why this works:** Multiple scanners provide defence in depth. Snyk offers fix PRs and continuous monitoring. Trivy is fast and open source. The SBOM provides a complete inventory of dependencies.

### Real-World Cases

- **Log4Shell:** Snyk and Trivy detected vulnerable Log4j versions.
- **Spring4Shell:** Scanners detected vulnerable Spring Framework versions.
- **PyPI typosquatting:** Socket detects malicious packages.
- **Licence compliance:** Snyk and FOSSA detect GPL violations.

---

## Core Concept 4: Dependency Updates

### Definitions

**Core Definition:** Dependency updates are the automated or manual process of upgrading dependencies to newer versions, including security patches, bug fixes, and feature updates.

**Technical Definition:** Dependency update tools (Dependabot, Renovate, npm-check-updates) monitor dependencies for new versions, create pull requests with the updates, run CI, and merge when tests pass. Updates are categorised as patch (bug fixes), minor (features, backwards-compatible), and major (breaking changes). Security updates should be prioritised and applied quickly. Non-security updates should be applied regularly to avoid "dependency debt." Best practice: update frequently in small batches, with comprehensive tests, and review lock file changes.

**Beginner-Friendly Explanation:** Dependency updates are like replacing old bricks with newer, better ones. The manufacturer (maintainer) releases a new version that fixes a defect or adds a feature. You want to upgrade, but you need to make sure the new brick fits (doesn't break your app). Tools like Dependabot automatically fetch the new brick, test it, and propose the swap.

### Purposes

- To patch security vulnerabilities quickly.
- To benefit from bug fixes and performance improvements.
- To stay current with the ecosystem.
- To reduce dependency debt.
- To comply with security policies (e.g., "no dependencies older than 1 year").
- To enable automation of routine maintenance.

### Syntax Rules and Structure

#### Update Tools Comparison

| Tool | Platform | Auto-merge | Grouping | Schedule |
|------|----------|------------|----------|----------|
| **Dependabot** | GitHub | ✅ Yes | ✅ Yes | Daily/weekly |
| **Renovate** | Multi-platform | ✅ Yes | ✅ Yes | Highly configurable |
| **npm-check-updates** | CLI | ❌ No | ❌ No | Manual |
| **Snyk** | Multi-platform | ✅ Yes | ✅ Yes | Continuous |

#### Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: 'npm'
    directory: '/'
    schedule:
      interval: 'weekly'
      day: 'monday'
      time: '06:00'
      timezone: 'UTC'
    open-pull-requests-limit: 10
    versioning-strategy: 'increase'
    labels:
      - 'dependencies'
      - 'security'
    reviewers:
      - 'security-team'
    groups:
      production-dependencies:
        dependency-type: 'production'
        update-types:
          - 'minor'
          - 'patch'
      development-dependencies:
        dependency-type: 'development'
        update-types:
          - 'minor'
          - 'patch'
    ignore:
      - dependency-name: 'aws-sdk'
        versions: ['3.x'] # Pin major version
    allow:
      - dependency-type: 'direct'
```

#### Renovate Configuration

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":dependencyDashboard",
    ":semanticCommits",
    "group:monorepos",
    "group:recommended"
  ],
  "schedule": ["before 6am on monday"],
  "timezone": "UTC",
  "labels": ["dependencies"],
  "rangeStrategy": "bump",
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true,
      "automergeType": "pr",
      "platformAutomerge": true
    },
    {
      "matchUpdateTypes": ["major"],
      "automerge": false,
      "labels": ["major-update", "needs-review"]
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    },
    {
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["patch"],
      "automerge": true
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "automerge": true
  },
  "osvVulnerabilityAlerts": true
}
```

#### Syntax Rules

- **Update regularly** — weekly, not quarterly.
- **Automate security updates** — auto-merge when tests pass.
- **Review major updates manually** — breaking changes require attention.
- **Group updates** — reduce PR noise.
- **Run full CI on update PRs** — tests, lint, build, audit.
- **Pin major versions** — avoid unexpected breaking changes.
- **Use semantic versioning** — `^` for minor/patch, exact for critical.
- **Monitor for abandoned packages** — replace them proactively.
- **Test in staging before production** — especially for major updates.
- **Document the update policy** — who approves, what is auto-merged.

#### Constraints and Limitations

- **Auto-merge can introduce bugs** — tests may not catch everything.
- **Breaking changes require manual work** — major updates are not automatic.
- **PR noise** — many updates create many PRs; grouping helps.
- **CI cost** — each PR runs the full pipeline.
- **Maintainer abandonment** — some packages are not updated.
- **Transitive dependencies** — updates may require upstream changes.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dependabot with Auto-Merge (GitHub Actions)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: 'npm'
    directory: '/'
    schedule:
      interval: 'weekly'
    open-pull-requests-limit: 10
    groups:
      production:
        dependency-type: 'production'
        update-types: ['minor', 'patch']
      development:
        dependency-type: 'development'
        update-types: ['minor', 'patch']
    labels: ['dependencies']
    commit-message:
      prefix: 'chore(deps)'
      include: 'scope'
```

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: Dependabot Auto-Merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Fetch Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Auto-approve patch and minor updates
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Enable auto-merge
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Label major updates for review
        if: steps.metadata.outputs.update-type == 'version-update:semver-major'
        run: gh pr edit "$PR_URL" --add-label "needs-review"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Expected behaviour:** Patch and minor updates are auto-approved and auto-merged when CI passes. Major updates are labelled for manual review. Security updates are prioritised.

**Why this works:** Automation handles routine updates. CI ensures tests pass. Major updates require human review. The metadata action provides update type context.

### Real-World Cases

- **Log4Shell:** Organisations with automated updates patched within hours.
- **Prototype pollution in lodash:** Dependabot PRs fixed the issue.
- **ReDoS in `path-to-regexp`:** Renovate PRs updated the package.
- **Node.js security releases:** Regular updates keep the runtime patched.

---

## Core Concept 5: Supply-Chain Security

### Definitions

**Core Definition:** Supply-chain security is the protection of the entire software supply chain — from maintainers and build systems to distribution and consumption — against tampering, compromise, and malicious insertion.

**Technical Definition:** Supply-chain security (SLSA, NIST SP 800-161) addresses risks at every stage: **source** (maintainer account compromise, malicious commits), **build** (compromised CI/CD, injected build steps), **distribution** (registry compromise, typosquatting), and **consumption** (dependency confusion, malicious packages). Controls include: verified publishers, 2FA enforcement, signed commits, reproducible builds, provenance attestations (SLSA), SBOMs, and runtime monitoring. Notable incidents (SolarWinds, Codecov, event-stream, ua-parser-js) demonstrate the severity of supply-chain attacks.

**Beginner-Friendly Explanation:** Supply-chain security is like ensuring that the food you buy is safe from farm to table. You need to trust the farmer (maintainer), the truck driver (distribution), the grocery store (registry), and the packaging (build). If any link in the chain is compromised, the food (your app) can be poisoned. Supply-chain security verifies every link.

### Purposes

- To prevent malicious code from entering the dependency tree.
- To detect maintainer account compromises.
- To verify package integrity and provenance.
- To enable rapid response to supply-chain incidents.
- To comply with regulations (Executive Order 14028, SLSA).

### Syntax Rules and Structure

#### SLSA Levels

| Level | Requirements | Protection |
|-------|--------------|------------|
| **SLSA 1** | Build process documented | Basic |
| **SLSA 2** | Hosted build, signed provenance | Tamper resistance |
| **SLSA 3** | Hardened builds, non-falsifiable provenance | Strong |
| **SLSA 4** | Hermetic, reproducible builds | Maximum |

#### npm Provenance

```bash
# Publish with provenance (GitHub Actions)
npm publish --provenance --access public

# Verify provenance
npm audit signatures

# View provenance on npmjs.com
# Visit the package page and click "Provenance"
```

#### Syntax Rules

- **Enable 2FA on npm accounts** — `npm profile enable-2fa auth-and-writes`.
- **Use scoped packages** — `@mycompany/package`.
- **Configure `.npmrc`** — scoped registries for private packages.
- **Verify provenance** — `npm audit signatures`.
- **Use lock files** — deterministic installs.
- **Monitor maintainer changes** — Socket, Phylum, Snyk.
- **Pin dependencies** — exact versions for critical packages.
- **Audit dependencies** — regularly.
- **Use SBOMs** — for incident response.
- **Subscribe to security advisories** — GitHub, npm, Snyk.
- **Review new dependencies** — check maintainers, downloads, age.
- **Use a private registry proxy** — Artifactory, Verdaccio.

#### Constraints and Limitations

- **Trust is required** — you cannot verify every line of every dependency.
- **Maintainer compromise is hard to detect** — 2FA helps but is not foolproof.
- **Provenance adoption is low** — many packages do not publish provenance.
- **SBOMs are static** — they do not reflect runtime behaviour.
- **Zero-days exist** — no control catches everything.
- **Cost** — commercial tools charge for advanced features.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Provenance Verification and SBOM Generation

```yaml
# .github/workflows/publish.yml — publish with provenance
name: Publish
on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write # Required for provenance
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm run build
      - run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

```yaml
# .github/workflows/verify-provenance.yml — verify on install
name: Verify Provenance
on:
  push:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - name: Verify package signatures
        run: npm audit signatures

      - name: Generate SBOM
        run: npx @cyclonedx/cyclonedx-npm --output-file sbom.json

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json

      - name: Scan SBOM for vulnerabilities
        run: npx @cyclonedx/cyclonedx-npm --output-format json | npx osv-scanner --sbom=-
```

**Expected behaviour:** Published packages include provenance attestations. CI verifies signatures on install. SBOMs are generated and scanned for vulnerabilities.

**Why this works:** Provenance verifies that packages were built on trusted CI/CD. Signature verification detects tampering. SBOMs provide a complete inventory for incident response.

### Real-World Cases

- **SolarWinds (2020):** Build system compromise injected malicious code into updates.
- **Codecov (2021):** Bash uploader script was modified to exfiltrate environment variables.
- **event-stream (2018):** Maintainer added a malicious dependency.
- **ua-parser-js (2021):** Maintainer account was compromised.
- **colors and faker (2022):** Maintainer intentionally broke both packages.

---

## Core Concept 6: Package Provenance & Signing

### Definitions

**Core Definition:** Package provenance is a cryptographically signed attestation that verifies where, when, and how a package was built, ensuring it originated from a trusted source.

**Technical Definition:** Provenance (npm, SLSA) is a signed statement (in-toto attestation) that links a package to its source repository, build system, build steps, and dependencies. npm provenance uses Sigstore's transparency log (Rekor) and short-lived signing certificates (Fulcio) tied to the CI/CD identity (GitHub Actions OIDC). Consumers verify provenance with `npm audit signatures`, which checks the signature against the transparency log. Provenance prevents: registry tampering, build system compromise, and package substitution. SLSA levels 2–4 require provenance.

**Beginner-Friendly Explanation:** Provenance is like a certificate of authenticity for a LEGO set. It says "This set was manufactured at the LEGO factory in Denmark on this date, using these specific moulds, and it was inspected by this inspector." If you have the certificate, you know the set is genuine. Without it, you might have a counterfeit. npm provenance provides this certificate for packages.

### Purposes

- To verify that a package was built from a specific source repository.
- To detect registry tampering and package substitution.
- To verify the build system and build steps.
- To comply with SLSA and Executive Order 14028.
- To enable rapid incident response.

### Syntax Rules and Structure

#### npm Provenance Fields

```json
{
  "predicateType": "https://slsa.dev/provenance/v1",
  "subject": [
    {
      "name": "pkg:npm/my-package@1.0.0",
      "digest": { "sha512": "..." }
    }
  ],
  "predicate": {
    "buildDefinition": {
      "buildType": "https://github.com/npm/cli/gha/v2",
      "externalParameters": {
        "workflow": {
          "ref": "refs/heads/main",
          "repository": "https://github.com/myorg/my-package",
          "path": ".github/workflows/publish.yml"
        }
      },
      "internalParameters": {
        "github": {
          "event_name": "release",
          "repository_id": "123456789",
          "repository_owner_id": "987654321"
        }
      },
      "resolvedDependencies": [
        {
          "uri": "git+https://github.com/myorg/my-package@refs/heads/main",
          "digest": { "gitCommit": "abc123..." }
        }
      ]
    },
    "runDetails": {
      "builder": {
        "id": "https://github.com/actions/runner/github-hosted"
      },
      "metadata": {
        "invocationId": "https://github.com/myorg/my-package/actions/runs/123456"
      }
    }
  }
}
```

#### Verification Commands

```bash
# Verify all package signatures
npm audit signatures

# Verify a specific package
npm audit signatures --json

# View provenance on npmjs.com
# Visit https://www.npmjs.com/package/<package> and click "Provenance"
```

#### Syntax Rules

- **Publish with `--provenance`** — requires GitHub Actions OIDC.
- **Use `id-token: write`** — for OIDC in GitHub Actions.
- **Verify on install** — `npm audit signatures`.
- **Use SLSA-compliant build systems** — GitHub Actions, GitLab CI.
- **Sign commits** — GPG or Sigstore.
- **Use transparency logs** — Rekor (Sigstore).
- **Monitor provenance** — for unexpected changes.
- **Document the verification policy** — what to do if verification fails.
- **Combine with SBOMs** — for full supply-chain visibility.
- **Educate developers** — provenance is only useful if verified.

#### Constraints and Limitations

- **Low adoption** — many packages do not publish provenance.
- **Requires GitHub Actions** — npm provenance is tied to GitHub OIDC.
- **Verification is not automatic** — `npm audit signatures` must be run.
- **Complexity** — provenance verification adds a step to CI.
- **Trust in the transparency log** — Sigstore must be trusted.
- **No protection against malicious maintainers** — a maintainer with valid credentials can publish malicious code.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Publishing and Verifying Provenance

```yaml
# .github/workflows/publish.yml
name: Publish with Provenance
on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write # Required for npm provenance
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm test
      - run: npm run build
      - run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

```yaml
# .github/workflows/verify.yml
name: Verify Provenance
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci

      - name: Verify package signatures
        run: npm audit signatures

      - name: Verify specific package provenance
        run: |
          npm view express@4.19.2 --json | jq '.dist.signatures'
```

**Expected behaviour:** Published packages include provenance. CI verifies signatures on install. The transparency log provides a public record.

**Why this works:** Provenance links the package to its source and build. Signature verification detects tampering. The transparency log provides auditability.

### Real-World Cases

- **npm package:** Many popular packages (Express, Next.js) publish provenance.
- **Sigstore:** Transparency log for software signing.
- **SLSA:** Framework for supply-chain security levels.
- **Executive Order 14028:** US government mandate for SBOMs and provenance.

---

## Core Concept 7: Automated Runtime Guardrails

### Definitions

**Core Definition:** Runtime guardrails are execution-time controls that intercept and block malicious behaviour (unauthorized network calls, file system access, environment variable reads) from dependencies before damage occurs.

**Technical Definition:** Runtime guardrails (Socket, Capslock, LavaMoat, seccomp) monitor and restrict what packages can do at runtime. Socket, for example, uses behavioural analysis to detect malicious packages and blocks them at install time (via `socket npm install`). Capslock analyses the capabilities a package requires (network, file system, shell) and alerts if they exceed expectations. LavaMoat uses SES (Secure ECMAScript) to sandbox dependencies with per-package capability policies. These tools complement static analysis (auditing, scanning) by catching behaviour that static analysis misses.

**Beginner-Friendly Explanation:** Runtime guardrails are like security cameras and motion sensors in your house. Static analysis (auditing) is checking the blueprint of the house before you buy it. Runtime guardrails watch what actually happens — if a package tries to open a window (network call) when it shouldn't, the alarm goes off and the action is blocked.

### Purposes

- To detect and block malicious behaviour at install time.
- To restrict what dependencies can do at runtime.
- To catch zero-day attacks that static analysis misses.
- To provide visibility into dependency behaviour.
- To enforce least privilege for dependencies.

### Syntax Rules and Structure

#### Socket CLI

```bash
# Install Socket CLI
npm install -g @socketsecurity/cli

# Install with Socket (analyses packages before install)
socket npm install express

# Scan a project
socket scan create --repo my-app

# Optimize dependencies (removes unused)
socket optimize
```

#### Capslock

```bash
# Install Capslock
npm install -g capslock

# Generate a capability report
capslock --audit

# Generate a policy file
capslock --generate-policy

# Lint a package
capslock --lint
```

#### LavaMoat

```typescript
// lavamoat.config.js
module.exports = {
  policies: {
    'my-app': {
      packages: {
        express: {
          // Express can access the network but not the file system
          network: true,
          fs: false,
          child_process: false,
        },
      },
    },
  },
};
```

#### Syntax Rules

- **Use Socket for install-time analysis** — `socket npm install`.
- **Use Capslock for capability auditing** — understand what packages can do.
- **Use LavaMoat for sandboxing** — enforce per-package capabilities.
- **Combine with static analysis** — runtime guardrails complement audits.
- **Monitor outbound network calls** — detect data exfiltration.
- **Restrict file system access** — for packages that should not read files.
- **Restrict `child_process`** — for packages that should not spawn.
- **Log all blocked actions** — for incident response.
- **Review alerts** — false positives are possible.
- **Educate developers** — runtime guardrails are only useful if understood.

#### Constraints and Limitations

- **Performance overhead** — runtime monitoring adds latency.
- **False positives** — legitimate packages may be flagged.
- **Complexity** — sandboxing requires careful configuration.
- **Adoption** — LavaMoat and Capslock are not widely adopted.
- **Commercial tools** — Socket charges for advanced features.
- **Cannot catch everything** — a determined attacker may bypass guardrails.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Socket CLI Integration in CI

```yaml
# .github/workflows/socket.yml
name: Socket Security
on:
  push:
    branches: [main]
  pull_request:

jobs:
  socket:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Socket Security Scan
        uses: SocketDev/action@v1
        with:
          mode: firewall
          repo: ${{ github.repository }}
        env:
          SOCKET_SECURITY_API_KEY: ${{ secrets.SOCKET_API_KEY }}

      - name: Install with Socket protection
        run: |
          npm install -g @socketsecurity/cli
          socket npm ci --ignore-scripts
```

```json
// .socketrc.json
{
  "issueRules": {
    "unmaintained": "error",
    "malware": "error",
    "telemetry": "warn",
    "networkAccess": "warn",
    "shellAccess": "error",
    "filesystemAccess": "warn",
    "envVars": "warn",
    "installScripts": "error",
    "nativeCode": "warn",
    "obfuscatedCode": "error"
  },
  "excludedPackages": [
    "esbuild", // Known to use postinstall
    "sharp"    // Native code
  ]
}
```

**Expected behaviour:** Socket scans all dependencies before install. Malicious packages are blocked. Suspicious behaviour (network access, shell access) is flagged. CI fails on errors.

**Why this works:** Socket analyses packages behaviourally, catching malicious code that static analysis misses. The firewall mode blocks installation of malicious packages.

### Real-World Cases

- **event-stream:** Socket would have detected the malicious `flatmap-stream` dependency.
- **ua-parser-js:** Socket detects compromised versions.
- **node-ipc:** Socket detects protestware behaviour.
- **colors/faker:** Socket detects intentionally broken packages.

---

## References

- OWASP Top 10 — A06:2021 Vulnerable and Outdated Components — https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/
- OWASP Top 10 — A08:2021 Software and Data Integrity Failures — https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/
- OWASP Cheat Sheet Series — Vulnerable Dependency Management — https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html
- OWASP Dependency-Check — https://owasp.org/www-project-dependency-check/
- NIST SP 800-161 — Cybersecurity Supply Chain Risk Management — https://csrc.nist.gov/publications/detail/sp/800-161/rev-1/final
- SLSA — Supply-chain Levels for Software Artifacts — https://slsa.dev/
- Executive Order 14028 — Improving the Nation's Cybersecurity — https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/
- npm Documentation — `npm audit` — https://docs.npmjs.com/cli/v10/commands/npm-audit
- npm Documentation — `npm ci` — https://docs.npmjs.com/cli/v10/commands/npm-ci
- npm Documentation — Package Provenance — https://docs.npmjs.com/generating-provenance-statements
- npm Documentation — `package-lock.json` — https://docs.npmjs.com/cli/v10/configuring-npm/package-lock-json
- pnpm Documentation — `pnpm audit` — https://pnpm.io/cli/audit
- Yarn Documentation — `yarn audit` — https://classic.yarnpkg.com/en/docs/cli/audit
- Snyk Documentation — https://docs.snyk.io/
- Trivy Documentation — https://aquasecurity.github.io/trivy/
- Grype Documentation — https://github.com/anchore/grype
- GitHub Advisory Database — https://github.com/advisories
- GitHub Dependabot — https://docs.github.com/en/code-security/dependabot
- Renovate Documentation — https://docs.renovatebot.com/
- Socket Documentation — https://docs.socket.dev/
- Capslock Documentation — https://github.com/google/capslock
- LavaMoat Documentation — https://github.com/LavaMoat/LavaMoat
- Sigstore — https://www.sigstore.dev/
- in-toto Attestation Framework — https://in-toto.io/
- CycloneDX — https://cyclonedx.org/
- SPDX — https://spdx.dev/
- OSV — Open Source Vulnerabilities — https://osv.dev/
- CVSS — Common Vulnerability Scoring System — https://www.first.org/cvss/
- EPSS — Exploit Prediction Scoring System — https://www.first.org/epss/