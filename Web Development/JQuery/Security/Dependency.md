# jQuery Dependency Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery dependency security is the discipline of ensuring that the jQuery library and every plugin, shim, and vendored copy it depends on are free from known vulnerabilities, are obtained from trustworthy sources, are updated to patched versions, and are continuously monitored through software composition analysis (SCA) and static analysis (SAST) so that the application does not inherit exploitable flaws from its third-party code.

**Technical Definition**
Dependency security for jQuery operates across four domains. **Vulnerability awareness** requires knowing the CVEs that affect jQuery core and its ecosystem — chiefly prototype pollution in `$.extend()` (CVE-2019-11358, fixed in 3.4.0) and XSS via `htmlPrefilter` (CVE-2020-11022 and CVE-2020-11023, fixed in 3.5.0). **Update management** requires moving declared dependencies to patched versions (the current stable 3.x line is 3.7.x, and jQuery 4.0.0 was released in January 2026). **Migration tooling** — the jQuery Migrate plugin — re-adds deprecated behaviours so legacy code can run on newer jQuery, but it must be treated as temporary scaffolding, not a permanent substitute for upgrading. **Dependency scanning** uses SCA tools (`npm audit`, OSV-Scanner, Snyk) to match lockfile versions against advisory databases, and SAST tools (Semgrep) to detect insecure usage patterns in application code. The persistent challenge is that vulnerable jQuery copies survive in bundled CMS themes, vendored admin panels, and transitive npm dependencies that package-manager scanners often do not see.

**Beginner-Friendly Explanation**
jQuery is a dependency — a piece of software your application relies on. Like any dependency, it can have security holes. When a hole is discovered, the jQuery team releases a fixed version. Dependency security is about three things: **knowing** which holes affect the version you are using, **updating** to a version that has the fixes, and **scanning** your project regularly to make sure no old, vulnerable copy has sneaked in unnoticed. The tricky part is that jQuery copies hide everywhere — inside WordPress themes, in `vendor/` folders, behind CDN links, and buried in other packages. A vulnerability fixed in 2019 can still show up in a scan in 2026 because someone’s theme still ships jQuery 3.3.1. Dependency security is the process of finding and fixing those copies before an attacker does.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Two CVEs dominate real scan results** | CVE-2019-11358 (prototype pollution in `$.extend()`, fixed in 3.4.0) and CVE-2020-11022/11023 (XSS via `htmlPrefilter`, fixed in 3.5.0) account for most jQuery findings in SCA reports. |
| **Patched versions exist** | Upgrading to jQuery ≥ 3.5.0 (and preferably the latest 3.7.x or 4.0.0) remediates both CVE families. |
| **Bundled copies evade SCA** | Vendored jQuery files inside CMS themes and admin panels are invisible to `npm audit` and often missed by Snyk unless the scanner inventories static assets. |
| **jQuery Migrate signals debt, not safety** | Teams that rely on jQuery Migrate frequently remain on jQuery 1.12 or 2.x, which carry the XSS flaws fixed in 3.5.0. |
| **SCA and SAST are complementary** | SCA matches dependency versions against CVEs; SAST finds insecure usage (e.g., `.html()` with untrusted data) that no version bump can fix. |
| **jQuery 4.0 adds Trusted Types support** | The 2026 major release improves CSP compatibility and removes long-deprecated APIs, but requires migration effort. |

---

### Prerequisites

- Working knowledge of `package.json`, lockfiles (`package-lock.json`, `yarn.lock`), and semantic versioning.
- Familiarity with jQuery’s role in the application and how it is loaded (npm, CDN, or vendored file).
- Awareness of CVE identifiers and severity scoring (CVSS).
- Access to a terminal for running SCA tools and a code repository for SAST.

---

### Related Programming Areas

- **Plugin security** — plugins carry their own CVEs and may bundle vulnerable jQuery copies.
- **XSS prevention** — the jQuery CVEs are XSS-class vulnerabilities; remediation requires both upgrading and safe coding practices.
- **Software supply chain security** — SBOM generation, provenance verification, and dependency pinning.
- **CI/CD security** — automated SCA and SAST gates on every build.
- **Content Security Policy (CSP)** — defence in depth that limits the impact of a compromised dependency.

---

### Core Concepts / Features (Index)

1. Dependency Vulnerabilities
2. jQuery Vulnerabilities
3. jQuery Updates
4. jQuery Migrate
5. Dependency Scanning
6. Dependency Security Best Practices


## Core Concept 1: Dependency Vulnerabilities

### Definitions

**Core Definition**
A dependency vulnerability is a security flaw in a third-party library, framework, or plugin that an application relies on, which can be exploited to compromise the application even though the application’s own code is not directly at fault.

**Technical Definition**
Dependency vulnerabilities are tracked as CVEs (Common Vulnerabilities and Exposures) and scored with CVSS (Common Vulnerability Scoring System). In the jQuery ecosystem, the most consequential dependency vulnerabilities are **prototype pollution** (CVE-2019-11358) and **DOM-based XSS** via `htmlPrefilter` (CVE-2020-11022 and CVE-2020-11023). A dependency vulnerability is **reachable** if the application actually invokes the vulnerable code path with attacker-influenced data; SCA tools report the vulnerability, but reachability analysis determines whether it is exploitable in context. Dependency vulnerabilities persist because vulnerable versions are **transitive** (pulled in by other packages), **vendored** (copied into static assets), or **abandoned** (no patched release will ever be issued).

**Beginner-Friendly Explanation**
A dependency vulnerability is a hole in someone else’s code that you inherited by using their library. You did not write the hole, but you are exposed to it. The two jQuery holes that matter most are: (1) a bug in `$.extend()` that lets an attacker poison shared objects (prototype pollution), and (2) a bug in how jQuery cleans up HTML strings that lets an attacker sneak in a script even after you sanitised it (XSS). Both were fixed years ago, but old copies still exist. The key insight is that a vulnerability only matters if the vulnerable code is actually used with attacker-controlled data — so knowing **which** version you have and **whether** you call the vulnerable function is the foundation of dependency security.

---

### Purposes

- **To recognise that inherited code carries inherited risk**.
- **To distinguish declared dependencies from transitive and vendored copies**.
- **To assess reachability** — whether the vulnerable code path is actually invoked.
- **To prioritise remediation** based on severity, reachability, and exploitability.
- **To motivate continuous scanning** rather than one-time audits.

---

### Syntax Rules and Structure

| Dependency Type | Visibility to SCA | Typical Example |
|---|---|---|
| **Direct npm dependency** | Visible in `package.json` / lockfile | `"jquery": "3.3.1"` |
| **Transitive npm dependency** | Visible in lockfile; may not be declared | `some-ui-lib` depends on `jquery@<3.4.0` |
| **Vendored / bundled copy** | Invisible to package-manager SCA | `themes/legacy/js/jquery-3.3.1.min.js` |
| **CDN-pinned script tag** | Invisible to SCA | `<script src=".../jquery-3.3.1.min.js">` |

---

### Constraints and Limitations

- **Not every reported vulnerability is exploitable** in your context; reachability matters.
- **Transitive dependencies can pin vulnerable versions** even when your direct dependency is current.
- **Vendored copies require manual inventory**; no package manager will flag them.

---

### Annotated Code Example (Identifying a Vulnerable Dependency)

```bash
# Step 1: Check the declared version and resolved lockfile version
npm ls jquery
# my-app@1.0.0
# └── jquery@3.3.1          ← vulnerable to CVE-2019-11358

# Step 2: Search for vendored copies that SCA cannot see
grep -rn "jQuery v3\." --include="*.js" vendor/ themes/ static/
# themes/legacy/js/jquery.js:1:/*! jQuery v3.3.1

# Step 3: Confirm the version at runtime (browser console)
# console.log(jQuery.fn.jquery);  →  "3.3.1"
```

**Expected Output**
The declared dependency resolves to 3.3.1, and a vendored copy of 3.3.1 exists in the legacy theme. Both are below the 3.4.0 fix threshold for prototype pollution.

**Why this output occurs**: `npm ls` shows the resolved version from the lockfile; `grep` discovers files that are outside the package manager’s view. Both must be remediated.

---

### Real-World Cases

- **Enterprise codebases in 2026**: CVE-2019-11358 resurfaces in SCA scans because bundled jQuery copies never left production.
- **WordPress themes and plugins**: A large share of plugin-bundled jQuery copies predate 3.4.0 and are rarely updated independently.
- **Transitive npm dependencies**: Build tooling and older UI libraries pin `jquery@<3.4.0`, propagating the vulnerability invisibly.

---

## Core Concept 2: jQuery Vulnerabilities

### Definitions

**Core Definition**
jQuery vulnerabilities are specific, published security flaws in the jQuery library itself — as opposed to flaws in plugins or application code — that can be exploited when the application uses the vulnerable version in an affected way.

**Technical Definition**
Two CVE families account for the overwhelming majority of jQuery core findings. **CVE-2019-11358** (prototype pollution) affects jQuery versions before 3.4.0. When `jQuery.extend(true, target, source)` performs a deep merge, it does not reject `__proto__` or `constructor.prototype` keys. An attacker who can influence the source object — via a JSON body, query string, or config payload — can inject a `__proto__` key and pollute `Object.prototype`, corrupting behaviour for every object in the application. Impact ranges from denial of service to client-side XSS to authorisation bypass in Node.js code that imports jQuery. **CVE-2020-11022 and CVE-2020-11023** (XSS via `htmlPrefilter`) affect jQuery versions from 1.2 up to but not including 3.5.0. The `htmlPrefilter` routine normalises HTML strings before they are inserted by `.html()`, `.append()`, `.prepend()`, `.before()`, `.after()`, and `.replaceWith()`. Under specific conditions, HTML that was already sanitised could be re-interpreted by the browser to execute a script. CISA added CVE-2020-11023 to its Known Exploited Vulnerabilities catalog. Both families are fixed in jQuery 3.5.0 and later.

**Beginner-Friendly Explanation**
jQuery has had two serious security problems. The first is **prototype pollution**: jQuery has a function called `$.extend()` that merges objects together. If an attacker can control one of the objects being merged, they can sneak in a special key called `__proto__` that changes the behaviour of every object in the application — like changing the rules of the game for everyone. The second is **XSS via `htmlPrefilter`**: jQuery cleans up HTML strings before inserting them into the page, but the cleaning was imperfect. An attacker could craft HTML that looked safe after cleaning but still executed a script when the browser parsed it. Both were fixed in jQuery 3.5.0. If you are on an older version, you are exposed — even if you sanitise your HTML, because the bug is in jQuery’s own processing.

---

### Purposes

- **To know exactly which jQuery versions are affected** by each CVE.
- **To understand the exploit mechanism** so that reachability can be assessed.
- **To identify the fixed versions** (3.4.0 for prototype pollution; 3.5.0 for XSS).
- **To recognise that sanitisation alone does not fix the XSS CVEs** — upgrading is mandatory.
- **To communicate the risk** to stakeholders using CVE identifiers and severity scores.

---

### Syntax Rules and Structure

| CVE | Vulnerability | Affected Range | Fixed In | Impact |
|---|---|---|---|---|
| **CVE-2019-11358** | Prototype pollution in `$.extend(true, …)` | `< 3.4.0` | **3.4.0** | DoS, XSS, authorisation bypass |
| **CVE-2020-11022** | XSS via `htmlPrefilter` (option/style) | `1.2 – 3.4.x` | **3.5.0** | DOM-based XSS |
| **CVE-2020-11023** | XSS via `htmlPrefilter` (self-closing tags) | `1.2 – 3.4.x` | **3.5.0** | Persistent XSS (CISA KEV) |

---

### Constraints and Limitations

- **The XSS CVEs affect sanitised HTML**; upstream sanitisation does not mitigate the flaw because the bug is in jQuery’s internal rewriting.
- **Prototype pollution’s impact depends on downstream usage**; a polluted property only causes harm if the application reads it in a security-sensitive context.
- **jQuery 4.0 does not introduce new core CVEs**; it adds Trusted Types support and removes deprecated APIs.

---

### Annotated Code Example (Prototype Pollution Demonstration)

```javascript
// Step 1: Simulate vulnerable jQuery.extend deep merge (< 3.4.0)
function vulnerableExtend(target, source) {
  for (var key in source) {
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] = vulnerableExtend(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// Step 2: Attacker-controlled payload (e.g., from JSON.parse of request body)
var payload = JSON.parse('{"__proto__": {"isAdmin": true}}');

// Step 3: Trigger the merge
vulnerableExtend({}, payload);

// Step 4: Every object now inherits the polluted property
console.log({}.isAdmin);                    // true
console.log(Object.prototype.isAdmin);      // true
```

**Expected Output**
```
true
true
```

**Why this output occurs**: The merge copies the `__proto__` key, which refers to `Object.prototype`. The assignment pollutes the prototype, making `isAdmin` visible on every object. The fix in jQuery 3.4.0 adds a check that rejects `__proto__` keys during merges.

---

### Real-World Cases

- **Node.js applications importing jQuery**: Prototype pollution can lead to authorisation bypass in server-side code.
- **Comment fields and WYSIWYG editors**: CVE-2020-11022/11023 enable stored XSS even when input is sanitised.
- **Drupal and Backdrop CMS**: CVE-2019-11358 affected these platforms through bundled jQuery.

---

## Core Concept 3: jQuery Updates

### Definitions

**Core Definition**
jQuery updates are the controlled process of moving the jQuery dependency from a vulnerable or outdated version to a patched, supported version — typically the latest 3.7.x release or jQuery 4.0.0 — while verifying compatibility with the application’s plugins and code.

**Technical Definition**
jQuery updates address three concerns. **Security updates** remediate known CVEs: upgrading to ≥ 3.4.0 fixes prototype pollution, and upgrading to ≥ 3.5.0 fixes the `htmlPrefilter` XSS flaws. **Compatibility updates** align jQuery with the application’s plugin ecosystem; jQuery 4.0.0 removes long-deprecated APIs (`jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`, `jQuery.now`, `jQuery.isFunction`) and drops support for IE10 and earlier, so plugins that rely on those APIs may break. **Lifecycle updates** move the application onto a supported release line: the 3.x line remains supported for applications that need older-browser compatibility, while 4.x is the forward path. The update should be performed in a branch, with the lockfile committed, the test suite run, and staging verification completed before production deployment.

**Beginner-Friendly Explanation**
Updating jQuery is not just pressing “upgrade.” You need to move to a version that has the security fixes — 3.5.0 at minimum, and preferably 3.7.x or 4.0.0. But you also need to make sure your plugins still work. jQuery 4.0 removed some old functions that plugins might still use, so an update can break things if you are not careful. The safe approach is: read the changelog, update in a branch, run your tests, check your plugins, and deploy gradually. The goal is to get off the vulnerable versions without breaking the application.

---

### Purposes

- **To remediate CVE-2019-11358 and CVE-2020-11022/11023** by moving to ≥ 3.5.0.
- **To move onto a supported release line** (3.7.x or 4.0.0).
- **To verify plugin compatibility** before and after the update.
- **To maintain a reproducible dependency graph** through lockfile discipline.
- **To reduce accumulated technical debt** from outdated dependencies.

---

### Syntax Rules and Structure

#### npm / Yarn Update Commands

```bash
# Check the current and latest versions
npm outdated jquery

# Update to the latest 3.x release
npm install jquery@3.7.1

# Or update to jQuery 4.0.0 (requires compatibility testing)
npm install jquery@4.0.0

# Run tests and audit
npm test
npm audit
```

#### Runtime Version Verification

```javascript
// In the browser console
console.log(jQuery.fn.jquery);   // e.g. "3.7.1"
```

---

### Constraints and Limitations

- **jQuery 4.0.0 is a major upgrade** with breaking changes for deprecated APIs and older browsers; test thoroughly.
- **Plugins may not yet support jQuery 4.0**; the 3.7.x line is the safer choice for applications with heavy plugin dependencies.
- **Updates do not fix vendored copies**; each copy must be updated separately.

---

### Annotated Code Example (Update Workflow)

```bash
# Step 1: Identify current version and CVEs
npm ls jquery
# jquery@3.3.1  →  CVE-2019-11358 (prototype pollution)

# Step 2: Read the changelog for the target version
# 3.4.0: prototype pollution fix
# 3.5.0: htmlPrefilter XSS fixes (CVE-2020-11022/11023)
# 3.7.1: current stable 3.x release

# Step 3: Update to the latest 3.x
npm install jquery@3.7.1

# Step 4: Run the test suite
npm test

# Step 5: Verify in the browser
# console.log(jQuery.fn.jquery);  →  "3.7.1"

# Step 6: Audit the lockfile
npm audit
```

**Expected Output**
The dependency is updated to 3.7.1, which includes the fixes for both CVE families. Tests pass, and `npm audit` reports no direct jQuery vulnerabilities.

**Why this output occurs**: Version 3.7.1 is above the 3.5.0 threshold for both the prototype pollution and XSS fixes. The test suite verifies that the application still works.

---

### Real-World Cases

- **WordPress core**: Upgrading from jQuery 3.6.x to 3.7.x or 4.0.0 requires verifying plugin compatibility with deprecated APIs.
- **Enterprise applications**: Staged rollouts with lockfile pinning ensure that updates are reproducible and reversible.
- **CDN-pinned applications**: The `<script>` tag must be updated to point to the new version; a lockfile alone does not help.

---

## Core Concept 4: jQuery Migrate

### Definitions

**Core Definition**
jQuery Migrate is a compatibility shim that re-adds deprecated or removed jQuery APIs so that older code can continue to run on newer jQuery releases, logging console warnings when deprecated behaviour is used.

**Technical Definition**
jQuery Migrate is designed as a **temporary migration aid**, not a permanent dependency. When jQuery removed or changed APIs across major versions (1.x → 3.x → 4.x), existing code broke. jQuery Migrate re-adds those behaviours and emits warnings such as `JQMIGRATE: jQuery.fn.attr('checked') might be used instead of .prop()`. The intended workflow is: add the shim, watch the warnings, fix each call site, then remove the shim. The security risk is that teams use Migrate as an excuse to stay on outdated jQuery versions. Projects that ship `jquery-migrate` are frequently the same projects still running jQuery 1.12 or 2.x, which carry the XSS flaws fixed in 3.5.0. The WordPress **Enable jQuery Migrate Helper** plugin has its own vulnerability, CVE-2026-3279, which allows unauthorised modification of data due to a missing capability check on its `downgrade_jquery_version()` function, enabling a downgrade from 3.7.1 to the legacy 1.12.4-wp release.

**Beginner-Friendly Explanation**
jQuery Migrate is a temporary translator. When jQuery changes, old code that uses removed functions stops working. Migrate brings those functions back and prints a warning each time they are used. The idea is to use the warnings as a to-do list: fix each one, then remove Migrate. But many teams never finish. They leave Migrate in place and keep using old jQuery — which means they keep the old security holes. Migrate itself is not malicious, but it is a red flag: if you see it in your dependency tree, it usually means the upgrade was never completed. The right end state is modern jQuery with no Migrate, or no jQuery at all.

---

### Purposes

- **To provide a temporary bridge** from old jQuery code to a newer release.
- **To surface deprecated API usage** through console warnings.
- **To guide a staged migration** rather than a risky big-bang upgrade.
- **To serve as a diagnostic signal** that an application may be on a vulnerable jQuery version.
- **To enable incremental remediation** of plugin and application code.

---

### Syntax Rules and Structure

#### Including jQuery Migrate (Development Only)

```html
<!-- Development: include Migrate to surface warnings -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://code.jquery.com/jquery-migrate-3.5.2.min.js"></script>

<!-- Production: remove Migrate after warnings are fixed -->
```

#### Checking Resolved Versions

```bash
npm ls jquery jquery-migrate
# my-app@1.0.0
# ├── jquery@3.7.1
# └── jquery-migrate@3.5.2   ← investigate whether this is still needed
```

---

### Constraints and Limitations

- **Migrate should not ship to production** long-term; it masks the need to upgrade.
- **Migrate’s presence correlates with outdated jQuery**; treat it as a signal for review.
- **The WordPress Enable jQuery Migrate Helper plugin has its own CVE** (CVE-2026-3279).

---

### Annotated Code Example (Migration Workflow)

```javascript
// Step 1: Run the application with Migrate in development
// Console output:
// JQMIGRATE: jQuery.fn.attr('checked') might be used instead of .prop()

// Step 2: Fix the deprecated usage
// ❌ OLD
$("input").attr("checked", true);
// ✅ NEW
$("input").prop("checked", true);

// Step 3: Repeat until no warnings remain

// Step 4: Remove Migrate from the page and from package.json
// npm uninstall jquery-migrate

// Step 5: Verify the application still works
// $ npm test
```

**Expected Output**
After fixing all warnings and removing Migrate, the console is clean, and the application runs on modern jQuery without the shim.

**Why this output occurs**: Migrate’s warnings identify the exact call sites that need remediation. Once those are fixed, the shim is no longer required.

---

### Real-World Cases

- **WordPress sites**: The Enable jQuery Migrate Helper plugin is widely used; its CVE-2026-3279 illustrates the risk of relying on migration tooling that itself becomes a vulnerability.
- **Legacy enterprise applications**: Migrate enables staged upgrades, but the end state must be Migrate-free.
- **Plugin ecosystems**: Plugins that log Migrate warnings need updates before jQuery 4.0 adoption.

---

## Core Concept 5: Dependency Scanning

### Definitions

**Core Definition**
Dependency scanning is the automated process of identifying the open-source components an application uses, matching their versions against known vulnerability databases, and producing a prioritised list of findings with remediation guidance.

**Technical Definition**
Dependency scanning is implemented through **Software Composition Analysis (SCA)** tools. The main scanner families are: **native audits** (`npm audit`, `yarn audit`, `pip-audit`) that use the ecosystem’s own advisory feed; **OSV-Scanner** (Google’s open-source scanner using the OSV.dev database, which aggregates advisories across ecosystems); and **Snyk** (commercial, with its own vulnerability database and monitoring alerts). Scanners should read **lockfiles**, not loose manifests, because the lockfile captures the exact resolved versions including transitive dependencies. **Reachability analysis** determines whether the vulnerable code path is actually invoked, reducing false positives. **SAST** (e.g., Semgrep) complements SCA by finding insecure usage patterns — such as `.html()` with untrusted data — that no version bump can fix.

**Beginner-Friendly Explanation**
Dependency scanning is the automated version of “checking the recall list.” You give a tool your lockfile, and it tells you which of your dependencies have known vulnerabilities. There are several tools: `npm audit` is built into npm and uses the npm advisory database; OSV-Scanner is free and covers many ecosystems; Snyk is commercial but has more detailed fix advice. The tools do not agree perfectly — each database carries some advisories the others miss — so the recommended approach is to run one native audit plus one cross-ecosystem scanner. Scanning is not a one-time event: new CVEs are published continuously, so it must run on every build and on a schedule.

---

### Purposes

- **To detect known CVEs in declared and transitive dependencies**.
- **To inventory bundled and vendored copies** that package managers cannot see.
- **To prioritise remediation** using severity, reachability, and fix availability.
- **To enforce dependency policy** in CI/CD pipelines.
- **To generate SBOMs** for supply chain transparency.

---

### Syntax Rules and Structure

#### Scanner Commands

```bash
# Native audit (Node.js)
npm audit

# OSV-Scanner (cross-ecosystem)
osv-scanner --lockfile=package-lock.json

# Snyk (commercial)
snyk test

# Semgrep (SAST for jQuery patterns)
semgrep --config "p/jquery" .
```

#### Inventorying Vendored Copies

```bash
# Find jQuery files anywhere in the repository
grep -rn "jQuery v" --include="*.js" .

# Search for CDN-pinned script tags
grep -rn "jquery-[0-9]" --include="*.html" .
```

---

### Constraints and Limitations

- **SCA tools miss vendored files**; manual inventory is required.
- **Native audits and OSV/Snyk have different coverage**; run more than one.
- **Reachability analysis is not universally available**; triage findings manually when needed.
- **Scanning is point-in-time**; new CVEs require re-scanning.

---

### Annotated Code Example (SCA Workflow)

```bash
# Step 1: Run native audit
npm audit
# High  Prototype Pollution in jquery  <3.4.0
# High  XSS in jquery htmlPrefilter   <3.5.0

# Step 2: Run cross-ecosystem scanner
osv-scanner --lockfile=package-lock.json
# jquery@3.3.1  CVE-2019-11358  fixed in 3.4.0
# jquery@3.3.1  CVE-2020-11022  fixed in 3.5.0

# Step 3: Inventory vendored copies
grep -rn "jQuery v3\." --include="*.js" themes/ vendor/
# themes/legacy/js/jquery.js:1:/*! jQuery v3.3.1  ← remediate separately

# Step 4: Triage and remediate
# - Upgrade npm jquery to 3.7.1
# - Replace vendored 3.3.1 copy in legacy theme
# - Re-run scans
```

**Expected Output**
```
jquery@3.3.1  →  CVE-2019-11358  →  fixed in 3.4.0
jquery@3.3.1  →  CVE-2020-11022  →  fixed in 3.5.0
Vendored copy: themes/legacy/js/jquery.js (3.3.1)
```

**Why this output occurs**: The native audit and OSV-Scanner both match the lockfile version against their respective databases. The `grep` step discovers the vendored copy that SCA could not see.

---

### Real-World Cases

- **Enterprise CI/CD**: Automated SCA on every pull request catches regressions before deployment.
- **WordPress security plugins**: Scanners flag jQuery Colorbox and jQuery Hover Footnotes CVEs when those plugins are in the inventory.
- **SBOM generation**: OSV-Scanner and Syft produce CycloneDX or SPDX reports for supply chain transparency.

---

## Core Concept 6: Dependency Security Best Practices

### Definitions

**Core Definition**
jQuery dependency security best practices are the consolidated conventions — knowing your versions, upgrading past the CVE thresholds, treating Migrate as temporary, running complementary SCA and SAST, inventorying vendored copies, and pinning production dependencies — that together minimise the risk inherited from jQuery and its ecosystem.

**Technical Definition**
Best practices synthesise the individual controls: (1) **know the resolved versions** of jQuery and jQuery Migrate from the lockfile, not the manifest range; (2) **upgrade to ≥ 3.5.0** (preferably the latest 3.7.x or 4.0.0) to remediate both CVE families; (3) **use jQuery Migrate only as a temporary development aid** and remove it before production; (4) **run SCA on every build** — one native audit plus one cross-ecosystem scanner; (5) **run SAST** to find insecure jQuery usage patterns that version updates cannot fix; (6) **inventory vendored and CDN-pinned copies** that SCA cannot see; (7) **pin production dependencies** and use `npm ci` for reproducible installs; and (8) **generate an SBOM** for supply chain transparency.

**Beginner-Friendly Explanation**
Best practices are the habits that keep dependency risk under control. Know what version you are actually running — not what `package.json` says, but what the lockfile resolved. Upgrade past 3.5.0 so both jQuery security holes are fixed. Use Migrate to help you migrate, not as a permanent crutch. Scan your dependencies on every build with more than one tool. Scan your code for unsafe patterns. Find the copies of jQuery hiding in themes and vendor folders. Pin your production versions so you know exactly what ships. And keep an inventory (SBOM) so you can respond quickly when the next CVE is announced.

---

### Purposes

- **To provide a single actionable checklist** for jQuery dependency security.
- **To prevent the recurrence of CVE-2019-11358 and CVE-2020-11022/11023**.
- **To detect vendored copies that SCA misses**.
- **To integrate dependency security into CI/CD** rather than treating it as a one-time audit.
- **To establish a baseline for code reviews and supply chain audits**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Version awareness** | `npm ls jquery` from lockfile | Trusting `package.json` ranges |
| **Upgrade target** | jQuery ≥ 3.5.0 (3.7.x or 4.0.0) | Staying below 3.5.0 |
| **jQuery Migrate** | Temporary development aid | Shipping Migrate to production long-term |
| **SCA** | `npm audit` + OSV-Scanner or Snyk | No dependency scanning |
| **SAST** | Semgrep jQuery rules | Manual review only |
| **Vendored copies** | Manual `grep` inventory | Assuming SCA sees everything |
| **Lockfile** | Commit, use `npm ci` | Loose manifests in CI |
| **SBOM** | CycloneDX or SPDX | No component inventory |

---

### Constraints and Limitations

- **Best practices are context-dependent**; a small static site may not need a full SBOM.
- **jQuery 4.0 adoption requires plugin compatibility testing**; the 3.7.x line remains valid for legacy ecosystems.
- **Automation is essential**; manual scans decay.

---

### Annotated Code Example (Consolidated)

```bash
# 1. Know the resolved versions
npm ls jquery jquery-migrate
# jquery@3.3.1  → vulnerable
# jquery-migrate@3.5.2  → investigate

# 2. Upgrade past the CVE thresholds
npm install jquery@3.7.1

# 3. Remove Migrate if no longer needed
npm uninstall jquery-migrate

# 4. Inventory vendored copies
grep -rn "jQuery v3\." --include="*.js" themes/ vendor/
# → update or replace any copy below 3.5.0

# 5. Run SCA and SAST in CI
npm audit
osv-scanner --lockfile=package-lock.json
semgrep --config "p/jquery" .

# 6. Commit the lockfile and use npm ci in CI
npm ci
```

**Expected Output**
```
jquery@3.7.1          ← remediated
jquery-migrate removed
Vendored 3.3.1 copy   ← flagged for manual update
SCA/SAST              ← no high-severity findings
```

**Why this output occurs**: Upgrading to 3.7.1 clears CVE-2019-11358 and CVE-2020-11022/11023. Removing Migrate eliminates the compatibility shim. The `grep` step finds the vendored copy that SCA cannot see. CI enforcement prevents regressions.

---

### Real-World Cases

- **Code reviews**: A checklist ensures that new jQuery dependencies meet version thresholds.
- **CI/CD pipelines**: Automated SCA and SAST gates block vulnerable dependencies from reaching production.
- **Legacy modernization**: Replacing bundled jQuery 1.x copies with 3.7.x or 4.0.0 is a common remediation project.

---

## References

- jQuery 4.0.0 Released | jQuery Blog — https://blog.jquery.com/2026/01/17/jquery-4-0-0/
- jQuery Prototype Pollution: CVE-2019-11358 Re-Emerges | Safeguard — https://safeguard.sh/resources/blog/jquery-prototype-pollution-vulnerability-re-emerges
- CVE-2020-11022: XSS in jQuery via htmlPrefilter | Safeguard — https://safeguard.sh/resources/blog/cve-2020-11022-xss-in-jquery-via-htmlprefilter
- jQuery 3.6.0 Vulnerabilities: What Scanners Flag & Fixes | Safeguard — https://safeguard.sh/resources/blog/jquery-3-6-0-vulnerabilities-guide
- jQuery Migrate: A Practical Security Guide | Safeguard — https://safeguard.sh/resources/blog/jquery-migrate
- CVE-2026-3279 (Enable jQuery Migrate Helper) | NVD — https://nvd.nist.gov/vuln/detail/CVE-2026-3279
- OSV-Scanner — https://google.github.io/osv-scanner/
- Snyk CLI — https://github.com/snyk/snyk
- OWASP Dependency-Check — https://owasp.org/www-project-dependency-check/
- OWASP SCA Audit Play — https://github.com/owasp/secure-agent-playbook
- npm audit documentation — https://docs.npmjs.com/cli/commands/audit
- Semgrep jQuery rules — https://semgrep.dev
- CISA Known Exploited Vulnerabilities (CVE-2020-11023) — https://www.cisa.gov/known-exploited-vulnerabilities-catalog