# jQuery Dependency Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** jQuery Dependency Security is the discipline of managing the jQuery library, its plugins, and their transitive dependencies in a manner that minimizes the risk of introducing known vulnerabilities into a web application, while ensuring that security patches are applied promptly and that third-party code loaded from external sources is verified for integrity.

**Technical Definition:** jQuery Dependency Security encompasses the lifecycle management of jQuery and its ecosystem — version selection, upgrade planning, vulnerability monitoring, supply-chain auditing, obsolete dependency removal, and Subresource Integrity (SRI) enforcement. It addresses risks arising from: (1) known CVEs in outdated jQuery versions (e.g., CVE-2019-11358 prototype pollution, CVE-2020-11022/11023 XSS); (2) unmaintained or malicious third-party plugins; (3) transitive dependencies introduced by plugins; (4) CDN-hosted assets that can be tampered with; and (5) polyfills and legacy shims that expand the attack surface without providing current value.

**Beginner-Friendly Explanation:** jQuery is a tool that millions of websites use. Like any tool, it gets updated over time — sometimes because new features are added, but often because security holes are discovered and fixed. Dependency security means: using a version of jQuery that has the important fixes, checking that the plugins you add do not bring their own security problems, watching for new vulnerabilities, removing tools you no longer need, and verifying that the copy of jQuery you load from a CDN is exactly what you expect it to be.

### Key Characteristics

- **jQuery has a long CVE history:** Multiple prototype pollution and XSS vulnerabilities have been discovered and patched over the years, and older versions remain widely deployed in legacy systems.
- **The 3.x branch is the current security baseline:** jQuery 4.0 was released in January 2026 with Trusted Types support and modern CSP compatibility; the 3.x branch receives critical security patches only.
- **Plugin dependencies are the weakest link:** jQuery itself has few direct dependencies, but plugins often pull in chains of transitive dependencies that may contain vulnerabilities.
- **CDN loading without SRI is a supply-chain risk:** Roughly three out of four websites load jQuery, often via CDNs with no SRI or version pinning.
- **Obsolete polyfills and plugins increase attack surface:** Removing dead code reduces the number of components that can be exploited.

### Prerequisites

- Proficiency in jQuery fundamentals: selectors, AJAX, DOM manipulation, and plugin usage.
- Understanding of web application security concepts (XSS, prototype pollution, supply-chain attacks).
- Familiarity with package managers (npm, yarn) and dependency management concepts.
- Awareness of browser security features (CSP, SRI, Trusted Types).

### Related Programming Areas

- **Software Supply Chain Security:** Managing third-party components and their transitive dependencies.
- **Vulnerability Management:** Continuous monitoring and patching of known CVEs.
- **Web Application Security (OWASP Top 10):** XSS and injection are the primary risks mitigated by dependency security.
- **Build Tooling and CI/CD:** Automating security audits as part of the build pipeline.
- **Content Security Policy (CSP):** Modern jQuery versions integrate with CSP and Trusted Types.

### Core Concepts / Features

This cheat sheet covers five core concepts and one enhanced topic: keeping jQuery updated, checking plugin dependencies, vulnerability monitoring, removing obsolete libraries, SRI implementation, and automated security auditing.

---

## Core Concept 1: Keeping jQuery Updated — Patching Historical Prototype Pollution and XSS Flaws

### Definitions

**Core Definition:** Keeping jQuery updated is the practice of upgrading the jQuery library to the latest stable version (or at minimum the latest security-patched version of the 3.x branch) to permanently address historical vulnerabilities such as prototype pollution in `$.extend()` and XSS bypasses in DOM manipulation methods.

**Technical Definition:** jQuery has accumulated a series of security vulnerabilities over its long history. The most significant are: **CVE-2019-11358** — prototype pollution in `jQuery.extend(true, ...)` (deep merge) that allows an attacker to inject a `__proto__` key and pollute `Object.prototype`; fixed in jQuery 3.4.0. **CVE-2020-11022** — XSS in the `htmlPrefilter()` method when passing HTML from untrusted sources, even after sanitization; fixed in jQuery 3.5.0. **CVE-2020-11023** — XSS in DOM manipulation methods when passing HTML containing `<option>` elements from untrusted sources; fixed in jQuery 3.5.0. **CVE-2020-7656** — XSS via improper script handling. jQuery 4.0.0, released January 2026, adds Trusted Types support for CSP compliance and removes legacy polyfills and deprecated APIs.

**Beginner-Friendly Explanation:** jQuery has had security bugs over the years, just like any software. The bugs have been fixed in newer versions, but older versions are still running on many websites. Keeping jQuery updated means using a version where these known bugs are already fixed, so attackers cannot exploit them.

### Purposes

- To eliminate known security vulnerabilities that have been patched in newer jQuery versions.
- To benefit from modern security features such as Trusted Types and CSP compatibility introduced in jQuery 4.0.
- To reduce the risk of prototype pollution attacks that can lead to XSS and authorization bypass.
- To comply with security best practices and audit requirements that mandate patching known CVEs.
- To prepare for the eventual removal of deprecated APIs and legacy browser support.

### Syntax Rules and Structure

**Version Upgrade Path:**

| Current Version | Recommended Action |
|-----------------|-------------------|
| jQuery 1.x | Upgrade to 3.7.x, then plan for 4.x |
| jQuery 2.x | Upgrade to 3.7.x, then plan for 4.x |
| jQuery 3.x (< 3.5.0) | Upgrade to 3.5.0+ immediately (critical XSS fixes) |
| jQuery 3.x (≥ 3.5.0) | Upgrade to 3.7.1 (latest 3.x) |
| jQuery 4.x | Current version; monitor for patches |

**Complete General Syntax (CDN Upgrade):**
```html
<!-- Before: vulnerable version -->
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>

<!-- After: patched version with SRI -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"
        integrity="sha384-1H217gwSVyLSIfaLxHbE7dRb3v4mYCKbpQvzx0cegeju1MVsGrX5xXxAvs/HgeFs"
        crossorigin="anonymous"></script>
```

**Syntax Rules:**

- Upgrade to jQuery 3.5.0 or later immediately if running any version between 1.0.3 and 3.5.0, because CVE-2020-11022 and CVE-2020-11023 are critical XSS vulnerabilities.
- Use the jQuery Migrate plugin to identify deprecated API usage before upgrading to 4.x.
- Test thoroughly after upgrading, especially form focus handling, `.toggleClass(true|false)`, and `.css("width", 100)` numeric pixel assumptions.
- For legacy applications that cannot upgrade immediately, use DOMPurify with `SAFE_FOR_JQUERY` to sanitize HTML before passing it to jQuery methods.

**Constraints and Limitations:**

- Upgrading from 1.x or 2.x to 3.x may break plugins that depend on removed APIs (e.g., `jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`).
- jQuery 4.0 removes support for IE 10 and older; IE 11 support is planned for removal in jQuery 5.0.
- Some legacy plugins may not be compatible with jQuery 4.x; audit plugin compatibility before upgrading.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Detecting Vulnerable jQuery Version**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jQuery Version Check</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 1: Read the jQuery version
      var version = $.fn.jquery;
      var parts = version.split(".").map(Number);

      // Step 2: Check if the version is vulnerable
      var isVulnerable = false;
      if (parts[0] === 1 || parts[0] === 2) {
        isVulnerable = true;
      } else if (parts[0] === 3 && parts[1] < 5) {
        isVulnerable = true; // CVE-2020-11022/11023
      }

      // Step 3: Display the result
      $("#log").html(
        "jQuery version: " + version + "<br>" +
        (isVulnerable
          ? "⚠️ VULNERABLE: Upgrade to jQuery 3.5.0+ immediately."
          : "✅ Version is not affected by known critical CVEs.")
      );
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
jQuery version: 3.7.1
✅ Version is not affected by known critical CVEs.
```

**Why this output:** The script reads `$.fn.jquery` (which returns the jQuery version string) and checks whether the major version is 1 or 2 (always vulnerable) or the major version is 3 with a minor version below 5 (vulnerable to CVE-2020-11022/11023). jQuery 3.7.1 passes the check.

### Real-World Cases

- **Enterprise applications:** Auditing all jQuery instances and upgrading to 3.7.x or 4.0.
- **WordPress sites:** Using the Enable jQuery Migrate Helper plugin to identify and fix deprecated API usage.
- **Legacy CMS platforms:** Bundled jQuery copies inside CMS themes and admin panels are a common source of unpatched vulnerabilities.
- **SPA frameworks:** Ensuring the jQuery version used by legacy plugins is not vulnerable.

---

## Core Concept 2: Checking Plugin Dependencies — Auditing the Supply Chain

### Definitions

**Core Definition:** Checking plugin dependencies is the practice of auditing the third-party jQuery plugins and wrappers used in a project to identify their direct and transitive dependencies, assess their maintenance status, and detect any known vulnerabilities introduced through the dependency chain.

**Technical Definition:** jQuery itself has few direct dependencies, but projects using jQuery often pull in plugins (e.g., jQuery UI, DataTables, Select2) that declare their own dependencies. A jQuery plugin such as `jquery-ui-extended` might depend on jQuery (direct), moment.js (for date handling), and timezone-data.js (transitive through moment.js). According to Black Duck's OSSRA report, 64% of open source components are transitive dependencies, and 81% of codebases contain high- or critical-risk vulnerabilities, nearly half of which were introduced by transitive dependencies. Auditing involves: identifying all declared dependencies in `package.json`, scanning for vulnerabilities in transitive dependencies, and checking whether each plugin is actively maintained.

**Beginner-Friendly Explanation:** When you add a jQuery plugin to your project, you are not just adding that plugin — you are also adding everything that plugin depends on. A date picker plugin might depend on a date library, which depends on a timezone database. Any of those could have a security problem. Checking plugin dependencies means tracing that chain and making sure everything is safe and maintained.

### Purposes

- To identify vulnerabilities introduced through transitive dependencies that are not visible in the project's direct `package.json`.
- To assess whether a plugin is actively maintained and receives security updates.
- To detect malicious packages that masquerade as legitimate jQuery plugins.
- To comply with software composition analysis (SCA) requirements and license audits.
- To make informed decisions about whether to keep, replace, or remove a plugin.

### Syntax Rules and Structure

**Complete General Syntax (npm Audit):**
```bash
npm audit
npm audit --audit-level=moderate
npm audit fix
```

**Complete General Syntax (Retire.js):**
```bash
# Install Retire.js
npm install -g retire

# Scan a directory
retire --path ./node_modules

# Scan a URL
retire --path https://code.jquery.com/jquery-1.11.3.min.js
```

**Complete General Syntax (Snyk):**
```bash
# Install Snyk
npm install -g snyk

# Test for vulnerabilities
snyk test

# Monitor for new vulnerabilities
snyk monitor
```

| Tool | Scope | Best For |
|------|-------|----------|
| `npm audit` | npm dependencies | Node.js projects with `package.json` |
| Retire.js | JS libraries (jQuery, Bootstrap, Angular) | Front-end libraries, CDN-loaded scripts |
| Snyk | Multi-language dependencies | CI/CD integration, detailed remediation |
| OWASP Dependency-Check | Multi-language dependencies | Enterprise SCA |

**Syntax Rules:**

- Run `npm audit` after every `npm install` to catch newly discovered vulnerabilities.
- Use Retire.js to scan for outdated client-side libraries that may be loaded via CDN and not tracked by npm.
- Integrate Snyk or `npm audit` into the CI/CD pipeline to fail builds when high-severity vulnerabilities are detected.
- Review the maintenance status of each plugin: check the last commit date, open issues, and release frequency.
- Be cautious of plugins that have not been updated in over a year; they may contain unpatched vulnerabilities.

**Constraints and Limitations:**

- `npm audit` only knows about dependencies in `node_modules`; it does not detect vendored copies of jQuery in a `static/vendor/` folder.
- Retire.js relies on its curated database, which may not cover every plugin.
- Transitive dependencies can be deeply nested and difficult to trace without automated tooling.
- Malicious packages may not be flagged by vulnerability scanners until after they are reported.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Auditing a Project with npm Audit**

```bash
# Step 1: Install dependencies
npm install jquery jquery-ui datatables.net

# Step 2: Run the audit
npm audit

# Expected output (example):
# found 2 vulnerabilities (1 moderate, 1 high)
#   High: Prototype Pollution in jquery@3.4.1
#   Moderate: XSS in datatables.net@1.10.20
```

**Expected Output:** The audit reports vulnerabilities in jQuery 3.4.1 (prototype pollution) and DataTables 1.10.20 (XSS). The developer can then upgrade to patched versions.

**Why this output:** `npm audit` analyses the dependency tree and cross-references against the npm advisory database. It reports the severity, the affected package, and the recommended fix.

### Real-World Cases

- **Enterprise SCA:** Black Duck's report found jQuery to be the most frequent source of vulnerabilities, with eight of the top ten high-risk vulnerabilities found in jQuery.
- **Malicious packages:** The `jquery.currencies` package was removed from npm after being identified as malicious; the `jquery-bindings` package was compromised in a supply chain attack in November 2025.
- **Trojanized jQuery Migrate:** In June 2025, a corrupted version of jQuery Migrate was discovered hiding malware in post-build minified bundles.

---

## Core Concept 3: Vulnerability Monitoring — Continuous Automated Security Audits

### Definitions

**Core Definition:** Vulnerability monitoring is the practice of continuously scanning a project's dependencies for known vulnerabilities using automated tools such as npm audit, Snyk, or Retire.js, and receiving alerts when new vulnerabilities are discovered in packages the project uses.

**Technical Definition:** Vulnerability monitoring operates on two levels: (1) **On-demand scanning** — running tools like `npm audit` or `retire` manually or as part of the build process; and (2) **Continuous monitoring** — using services like Snyk Monitor or GitHub Dependabot to receive alerts when new CVEs are published for dependencies in the project. Retire.js is specifically designed for JavaScript libraries, with a curated database covering jQuery, Bootstrap, Angular, and hundreds of npm packages. Snyk provides detailed vulnerability descriptions and remediation advice, and integrates with CI/CD pipelines.

**Beginner-Friendly Explanation:** New security vulnerabilities are discovered all the time — even in software that was previously considered safe. Vulnerability monitoring means having a system that watches for these discoveries and tells you when a package you use has a problem, so you can update it before attackers exploit it.

### Purposes

- To receive timely alerts when new vulnerabilities are discovered in dependencies.
- To automate the detection of vulnerable jQuery versions and plugins in CI/CD pipelines.
- To maintain an up-to-date inventory of all dependencies and their security status.
- To reduce the window of exposure between vulnerability disclosure and patching.
- To comply with security policies that require continuous monitoring.

### Syntax Rules and Structure

**Complete General Syntax (CI/CD Integration):**
```yaml
# GitHub Actions example
- name: Run security audit
  run: |
    npm audit --audit-level=high
    npx snyk test --severity-threshold=high
```

| Tool | Monitoring Type | Integration |
|------|----------------|-------------|
| `npm audit` | On-demand | CLI, CI/CD |
| Snyk | Continuous | CLI, CI/CD, GitHub, IDE |
| Retire.js | On-demand | CLI, Burp/ZAP extensions, Chrome extension |
| Dependabot | Continuous | GitHub |

**Syntax Rules:**

- Run `npm audit` in CI/CD and fail the build if high-severity vulnerabilities are found.
- Use Snyk Monitor (`snyk monitor`) to continuously track dependencies and receive alerts.
- Use Retire.js with its `--path` option to scan both local files and CDN URLs.
- Configure the audit tool to exclude development-only dependencies if they do not affect production security.

**Constraints and Limitations:**

- Vulnerability databases may have false positives or lag behind disclosures.
- `npm audit` only covers npm-managed dependencies; vendored libraries require manual scanning.
- Continuous monitoring services may require API keys and network access to external services.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Retire.js Scanning a CDN-Loaded jQuery**

```bash
# Step 1: Install Retire.js
npm install -g retire

# Step 2: Scan a CDN-hosted jQuery version
retire --path https://code.jquery.com/jquery-3.3.1.min.js

# Expected output:
# jquery
#   ↳ 3.3.1
#     Severity: High
#     CVE: CVE-2019-11358
#     Summary: Prototype Pollution
#     Info: https://github.com/jquery/jquery/pull/4333
```

**Expected Output:** Retire.js identifies jQuery 3.3.1 as vulnerable to CVE-2019-11358 (prototype pollution) and reports the severity and remediation information.

**Why this output:** Retire.js maintains a database of known vulnerable library versions and their CVEs. It scans the specified path (local file or URL) and matches the detected library version against the database.

### Real-World Cases

- **GitHub Dependabot:** Automatically creates pull requests when a vulnerable dependency is detected.
- **Snyk in CI:** Fails the build when a high-severity vulnerability is introduced by a new dependency.
- **Retire.js Chrome extension:** Scans websites in real time to detect vulnerable JavaScript libraries.
- **Enterprise security dashboards:** Aggregate vulnerability data from multiple projects.

---

## Core Concept 4: Removing Obsolete Libraries — Minimizing Attack Surface

### Definitions

**Core Definition:** Removing obsolete libraries is the practice of decommissioning redundant code bases, dead plugins, unused polyfills, and deprecated shims to reduce the number of components that could contain vulnerabilities and minimize the overall attack surface of the application.

**Technical Definition:** jQuery 4.0 removed polyfills for older JavaScript features, the Sizzle selector engine, JSONP, and many deprecated APIs, relying on native implementations in supported browsers. Projects migrating to jQuery 4.x should audit their codebase for usage of these removed features and either replace them with native alternatives or remove them entirely. Additionally, jQuery Migrate — while useful during upgrades — should be removed once the migration is complete, as it re-introduces deprecated behavior that may have security implications. According to practical migration guidance, the recommended approach is: upgrade to 3.6.x first, add jQuery Migrate 4.0 to surface warnings, fix all warnings, and then remove Migrate before switching to 4.x.

**Beginner-Friendly Explanation:** Over time, websites accumulate extra tools — plugins that are no longer used, polyfills for browsers that no longer exist, and compatibility shims that were only needed during an upgrade. Every extra piece of code is a potential security risk. Removing obsolete libraries is like cleaning out your garage: you get rid of things you no longer need, and there is less clutter to worry about.

### Purposes

- To reduce the number of components that could contain vulnerabilities.
- To eliminate deprecated APIs that may have known security weaknesses.
- To remove polyfills that are no longer needed, reducing code size and potential attack surface.
- To comply with modern browser requirements and prepare for jQuery 4.x compatibility.
- To simplify the codebase and make security auditing more manageable.

### Syntax Rules and Structure

**Complete General Syntax (Removing jQuery Migrate):**
```html
<!-- Before: jQuery Migrate included -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://code.jquery.com/jquery-migrate-3.4.1.min.js"></script>

<!-- After: Migrate removed once warnings are resolved -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
```

**Migration Checklist:**

| Step | Action |
|------|--------|
| 1 | Upgrade to jQuery 3.6.x (latest 3.x) |
| 2 | Add jQuery Migrate 4.0 and fix all console warnings |
| 3 | Remove jQuery Migrate |
| 4 | Upgrade to jQuery 4.x |
| 5 | Remove unused plugins and polyfills |

**Syntax Rules:**

- Never remove jQuery Migrate until all deprecation warnings have been resolved.
- Audit the codebase for usage of removed APIs (`jQuery.trim`, `jQuery.isArray`, `jQuery.parseJSON`, `jQuery.now`, `jQuery.isFunction`).
- Replace `.toggleClass(true|false)` with explicit `.addClass()` / `.removeClass()`.
- Replace numeric `.css("width", 100)` with explicit units (`.css("width", "100px")`).
- Remove polyfills for features that are natively supported in the project's target browsers.

**Constraints and Limitations:**

- Removing a plugin may break functionality if the plugin is still in use; audit usage before removing.
- Some polyfills may still be needed for older browsers that are within the project's support matrix.
- jQuery 4.0 removal of Sizzle and JSONP may break plugins that depend on those features.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Auditing for Deprecated API Usage**

Step 1: Scan the codebase for deprecated APIs
(This would typically be done with a tool like grep or a linter)

Deprecated APIs to search for:
- jQuery.trim()     → String.prototype.trim
- jQuery.isArray()  → Array.isArray
- jQuery.parseJSON() → JSON.parse
- jQuery.now()      → Date.now
- jQuery.isFunction() → typeof

Step 2: Replace deprecated APIs with native equivalents
```js
// Before:
var trimmed = jQuery.trim(input);

// After:
var trimmed = String(input).trim();
```
```js
// Before:
var arr = jQuery.isArray(value);

// After:
var arr = Array.isArray(value);
```
```js
// Before:
var obj = jQuery.parseJSON(jsonString);

// After:
var obj = JSON.parse(jsonString);
```

**Expected Output:** After replacing all deprecated APIs, the codebase is compatible with jQuery 4.x and jQuery Migrate can be removed.

**Why this output:** jQuery 4.0 removed these deprecated utilities. Replacing them with native JavaScript equivalents ensures compatibility and eliminates the dependency on the Migrate shim.

### Real-World Cases

- **WordPress:** The Enable jQuery Migrate Helper plugin can downgrade jQuery versions, but has its own vulnerability (CVE-2026-3279) due to missing capability checks.
- **Splunk:** Provides an Internal Library Settings page to restrict access to jQuery versions older than 3.5.
- **Enterprise migration:** The recommended approach is to use jQuery Migrate to surface warnings, fix them, and then remove Migrate before upgrading to 4.x.

---

## Enhanced Topic: Subresource Integrity (SRI) Implementation — Enforcing Cryptographic Hash Checks

### Definitions

**Core Definition:** Subresource Integrity (SRI) is a browser security feature that allows a script or stylesheet loaded from an external source (typically a CDN) to be verified against a cryptographic hash, ensuring that the resource has not been tampered with. If the hash does not match, the browser refuses to execute the resource.

**Technical Definition:** SRI is implemented via the `integrity` attribute on `<script>` or `<link>` elements. The attribute value is a base64-encoded cryptographic hash (typically SHA-384) prefixed by the algorithm name (e.g., `sha384-`). When the browser fetches the resource, it computes the hash of the response body and compares it to the value in the `integrity` attribute. If they do not match, the browser blocks the resource and reports a network error. CDNs must serve resources with the `Access-Control-Allow-Origin` header to support SRI. SRI is a W3C standard supported by all modern browsers.

**Beginner-Friendly Explanation:** When you load jQuery from a CDN, you are trusting that the CDN will deliver the correct file. If an attacker compromises the CDN, they could modify the jQuery file to include malicious code. SRI lets you specify a "fingerprint" of the correct file. The browser checks the fingerprint before running the script. If the file has been changed, the browser refuses to run it.

### Purposes

- To protect against supply-chain attacks that compromise CDN-hosted resources.
- To ensure that the exact version of jQuery (or any library) that was tested is the version that runs in the user's browser.
- To provide a browser-enforced integrity check that does not depend on the CDN's security.
- To comply with security best practices and CSP requirements that recommend SRI.
- To mitigate the risk of malicious code injection through third-party CDNs.

### Syntax Rules and Structure

**Complete General Syntax:**
```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js"
        integrity="sha384-1H217gwSVyLSIfaLxHbE7dRb3v4mYCKbpQvzx0cegeju1MVsGrX5xXxAvs/HgeFs"
        crossorigin="anonymous"></script>
```

| Component | Description |
|-----------|-------------|
| `src` | The URL of the resource. |
| `integrity` | The base64-encoded hash, prefixed by the algorithm (e.g., `sha384-`). |
| `crossorigin="anonymous"` | Required for cross-origin SRI; tells the browser not to send credentials. |

**Generating a Hash:**
```bash
curl -s https://code.jquery.com/jquery-3.7.1.min.js | \
  openssl dgst -sha384 -binary | \
  openssl base64 -A
```

**Syntax Rules:**

- The `integrity` attribute is required for SRI; the `crossorigin` attribute is required for cross-origin resources.
- Use SHA-384 (recommended) or SHA-256; SHA-1 is not supported.
- The hash must match the exact bytes of the resource; even a whitespace change will cause a mismatch.
- CDNs must set `Access-Control-Allow-Origin` for SRI to work cross-origin.
- Use a hash generator tool like srihash.org to generate the correct `integrity` value.

**Constraints and Limitations:**

- SRI hashes must be updated when the resource is updated; this requires a build process or manual updating.
- SRI does not protect against server-side compromise of the origin server, only CDN tampering.
- Some older browsers do not support SRI; the resource loads without verification in those browsers.
- SRI does not prevent the CDN from serving a different (valid) version of the file if the URL is not version-pinned.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Loading jQuery with SRI**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>SRI Demo</title>
  <!-- Step 1: Load jQuery with SRI from the official CDN -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"
          integrity="sha384-1H217gwSVyLSIfaLxHbE7dRb3v4mYCKbpQvzx0cegeju1MVsGrX5xXxAvs/HgeFs"
          crossorigin="anonymous"></script>
</head>
<body>
  <p id="log"></p>

  <script>
    $(function() {
      // Step 2: Verify that jQuery loaded successfully
      if (typeof jQuery !== "undefined") {
        $("#log").text("jQuery " + $.fn.jquery + " loaded with SRI verification.");
      } else {
        $("#log").text("jQuery failed to load (SRI hash mismatch or network error).");
      }
    });
  </script>
</body>
</html>
```

**Expected Output:**
```
jQuery 3.7.1 loaded with SRI verification.
```

**Why this output:** The browser fetches jQuery 3.7.1 from `code.jquery.com`, computes the SHA-384 hash of the response, and compares it to the value in the `integrity` attribute. If they match, the script executes; if not, the browser blocks it. The script then verifies that jQuery is available and displays its version.

### Real-World Cases

- **Official jQuery CDN:** `code.jquery.com` provides SRI hashes for all jQuery versions.
- **cdnjs and jsDelivr:** Both support SRI and provide hash values for their hosted libraries.
- **Google Hosted Libraries:** Provides SRI hashes for jQuery and other libraries.
- **Enterprise security policies:** Many organizations require SRI for all externally loaded scripts.

---

## References

- jQuery Prototype Pollution: CVE-2019-11358 Re-Emerges — Safeguard — https://safeguard.sh/resources/blog/jquery-prototype-pollution-vulnerability-re-emerges
- jQuery 3.3.1 Prototype Pollution & XSS Exploit — Exploit Database — https://www.exploit-db.com/exploits/52141
- jQuery 3.5.0 Released! — Official jQuery Blog — https://blog.jquery.com/2020/04/10/jquery-3-5-0-released/
- CVE-2020-11023 — CIRCL Vulnerability Lookup — https://vulnerability.circl.lu/vuln/cve-2020-11023
- jQuery 4.0 Released — heise.de — https://www.heise.de
- jQuery End of Life (EOL) Dates & Support Status — EOL.Wiki — https://eol.wiki
- Subresource Integrity (SRI) Implementation — MDN — https://mdn.org.cn/en-US/docs/Web/Security/Practical_implementation_guides/SRI
- Subresource Integrity — developer.typescripts.org — https://developer.typescripts.org/en-US/docs/Web/Security/Practical_implementation_guides/SRI
- jQuery CDN Supply Chain Risk Analysis — Safeguard — https://safeguard.sh/resources/blog/jquery-cdn-supply-chain-risk-analysis
- Hidden Malware Discovered in jQuery Migrate — Trellix — https://www.trellix.com
- GMS-2025-495: jquery-bindings contains malware — GitLab Advisories — https://advisories.gitlab.com
- Black Duck OSSRA Report — Black Duck — https://www.blackduck.com
- JavaScript安全审计 — php.cn — https://www.php.cn/faq/1771922.html
- Retire.js Chrome Extension — Chrome Web Store — https://chromewebstore.google.com
- npm audit — npm Documentation — https://docs.npmjs.com/cli/v10/commands/npm-audit
- Splunk jQuery Upgrade Readiness — Splunk Documentation — https://docs.splunk.com
- jQuery 4.0 Migration Guide — biton.co.jp — https://www.biton.co.jp/blog_61.html
- CVE-2026-3279 Enable jQuery Migrate Helper — VulDB — https://vuldb.com
- jQuery Migrate: A Practical Security Guide — Safeguard — https://safeguard.sh/resources/blog/jquery-migrate-practical-security-guide
- OWASP Dependency-Check — OWASP — https://owasp.org/www-project-dependency-check/