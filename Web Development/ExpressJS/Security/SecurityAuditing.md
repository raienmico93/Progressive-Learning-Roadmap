# Express.js Security Auditing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Security auditing in Express.js is the systematic, evidence-based examination of an application's code, configuration, dependencies, and runtime behavior to identify vulnerabilities, verify the effectiveness of security controls, and produce actionable remediation guidance before weaknesses can be exploited.

**Technical Definition**
Express.js security auditing combines **static analysis** (AST-based rule engines such as `express-audit` and `eslint-plugin-express-security`), **software composition analysis** (SCA tools such as `npm audit`, Snyk, and OSV-Scanner), **dynamic analysis** (DAST and penetration testing of the running application), and **configuration review** (Helmet, CORS, body-parser limits, `trust proxy`, and cookie flags). The audit follows a structured methodology—**Discover → Categorize → Remediate → Verify**—and assigns severity using frameworks such as OWASP Top 10, CWE, and CVSS so that findings can be triaged and tracked to closure . Express's minimal design means much of its security posture lives in companion packages (`send`, `serve-static`, `qs`, `path-to-regexp`, `body-parser`, `cookie`), making dependency auditing and application-level code review equally essential .

**Beginner-Friendly Explanation**
A security audit is a thorough checkup for your Express app. It is like a building inspection: you walk through every room (code, dependencies, configuration, runtime), look for weak locks (missing headers, open redirects, over-broad CORS), and write down what needs fixing and how urgent it is. Penetration testing is the “break-in test”—you try to exploit the weaknesses on purpose to see which ones are real. Vulnerability scanning is the “automated smoke detector”—it continuously watches for known problems. Together they give you a complete picture of your security posture.

---

### Key Characteristics

- **Layered evidence** — A credible audit triangulates **static code analysis**, **dependency scanning**, **configuration review**, and **dynamic testing**; relying on any single tool leaves blind spots .
- **Express-specific attack surface** — The highest-impact issues are rarely framework-core bugs; they are **application-level misconfigurations** such as reflecting user input into `res.redirect()`, unbounded `express.json()` parsing, missing Helmet headers, and misconfigured `trust proxy` .
- **Broken access control dominates** — OWASP's 2025 Top 10 reports that **100% of tested applications** showed some form of broken access control, up from 94% in 2021, and Express ships with essentially no access-control primitives out of the box .
- **Severity-driven output** — Findings must be tagged with **CWE**, **OWASP category**, **CVSS score**, and a **concrete fix**, not just a warning, so that engineering teams can prioritize and auto-remediate .
- **Continuous, not one-off** — Modern auditing runs in CI on every pull request, with scheduled deep scans and continuous monitoring for new advisories affecting already-deployed versions .

---

### Prerequisites

- Node.js 18 or higher and an Express application with `package.json` and a committed `package-lock.json`.
- A running instance of the application (for dynamic testing) in an isolated environment.
- Access to source code, configuration files, and environment-variable definitions.
- Installed tooling: `npm audit`, Snyk CLI (`snyk`), `express-audit`, `eslint-plugin-express-security`, and optionally a DAST proxy (Burp Suite, OWASP ZAP).
- Authorization and scope documentation for any penetration testing activity.

---

### Related Programming Areas

- **Software Composition Analysis (SCA)** — Dependency vulnerability scanning (`npm audit`, Snyk, OSV-Scanner).
- **Static Application Security Testing (SAST)** — AST-based code review (`express-audit`, `eslint-plugin-express-security`, Semgrep).
- **Dynamic Application Security Testing (DAST)** — Penetration testing and runtime probing.
- **Configuration hardening** — Helmet, CORS, body-parser limits, `trust proxy`, cookie flags.
- **Compliance** — Mapping findings to SOC 2, PCI-DSS, HIPAA, and OWASP ASVS controls.

---

## Core Concept 1: Security Auditing

### Definitions

**Core Definition**
Security auditing is the structured review of an Express.js application's code, configuration, dependencies, and architecture against a security baseline, producing a prioritized list of findings with evidence and remediation guidance.

**Technical Definition**
A security audit follows the **4-phase workflow**: **Discover** (inventory routes, middleware, dependencies, secrets, and trust boundaries), **Categorize** (map each finding to OWASP Top 10, CWE, and CVSS severity), **Remediate** (provide a concrete fix for each finding), and **Verify** (confirm the fix and check for regressions) . The audit must trace **sensitive operations** from request entry through validation, authorization, business logic, data access, and response or side effect, and must validate candidate findings against reachable code and existing controls . Express-specific audit checkpoints include: missing `helmet()`, over-broad `cors()`, `req.query`/`req.params` flowing into sinks, weak session secrets, missing per-object authorization (IDOR/BOLA), and `app.use()` ordering that bypasses auth middleware .

**Beginner-Friendly Explanation**
A security audit is a checklist-driven walkthrough of your app. You start by mapping every door and window (routes and middleware), then check each one against known weak points. You write down what is wrong, how serious it is, and exactly how to fix it. Then you verify the fix actually works.

---

### Purposes

- To identify vulnerabilities before attackers do, with evidence and severity so remediation can be prioritized.
- To verify that security controls (Helmet, CORS, authentication, authorization) are correctly configured and ordered.
- To satisfy compliance and procurement requirements for evidence of security due diligence.
- To create a repeatable baseline that can be re-audited after every significant change.

---

### Syntax Rules and Structure

#### Complete General Syntax: 4-Phase Audit Workflow

```text
Phase 1: Discover
  → Inventory routes, middleware, dependencies, secrets, trust boundaries
  → Run npm audit, express-audit, eslint-plugin-express-security

Phase 2: Categorize
  → Map each finding to OWASP Top 10, CWE, CVSS
  → Assign severity: Critical / High / Medium / Low

Phase 3: Remediate
  → Provide concrete fix (code, configuration, version bump)
  → Create small PRs targeting one class of issue

Phase 4: Verify
  → Re-run scanners, confirm fix, check for regressions
  → Update audit report and close findings
```

**Component Breakdown**

| Phase | Key Activities | Express-Specific Checks |
|-------|---------------|------------------------|
| Discover | Route inventory, dependency scan, secret scan | `express-audit`, `npm audit`, Semgrep |
| Categorize | OWASP/CWE/CVSS mapping | Broken access control, misconfiguration, injection |
| Remediate | Concrete fixes, small PRs | Helmet, CORS allowlist, body limits, auth middleware |
| Verify | Re-test, regression check | Re-run scanners, confirm headers, confirm auth |

#### Syntax Rules

- **Never report a theoretical vulnerability with no realistic attack path**; validate findings against reachable code and existing controls .
- **Do not ignore medium/low findings**; note them even if they are not blocking .
- **Provide a concrete fix**, not just a warning .
- **Use AST-based tools**, not string matching, to reduce false positives and detect missing options (e.g., `httpOnly` absent from `res.cookie()`) .

#### Constraints and Limitations

- Static analysis cannot follow middleware registered through factories or dependency injection, and will miss cross-file data flows .
- An audit is a point-in-time assessment; continuous scanning is required to catch new CVEs and regressions.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Running the Express-Specific Static Audit

```bash
# 1. Install dependencies exactly as locked
npm ci

# 2. Run dependency audit (SCA)
npm audit --audit-level=high

# 3. Run Express-specific static analysis
npx express-audit

# 4. Run Express security ESLint rules
npx eslint --plugin express-security --rule 'express-security/require-helmet: error' .
```

**Expected Output (express-audit)**

```
src/app.js
  12:3  HIGH   COOKIE001  res.cookie() called without httpOnly: true
       Fix: res.cookie('session', token, { httpOnly: true, secure: true, sameSite: 'strict' })

  18:5  HIGH   JWT001     jwt.sign() called without expiresIn
       Fix: jwt.sign(payload, secret, { expiresIn: '15m' })

  25:3  CRITICAL CORS001  cors() called without origin allowlist
       Fix: cors({ origin: ['https://app.example.com'] })
```

**Why This Output Occurs**

- `express-audit` parses the source into an AST and inspects typed nodes for `res.cookie()`, `jwt.sign()`, and `cors()` calls, flagging the specific missing or unsafe option rather than a generic string match .

---

#### Example 2: Mapping a Finding to OWASP and CWE

```text
Finding: cors() called without origin allowlist
OWASP:   A05:2021 – Security Misconfiguration
CWE:     CWE-942 – Permissive Cross-domain Policy with Untrusted Domains
CVSS:    9.1 (Critical)
Fix:     cors({ origin: ['https://app.example.com'] })
```

**Why This Output Occurs**

- `eslint-plugin-express-security` produces structured messages with CWE, OWASP, CVSS, and a concrete fix in every finding, enabling compliance mapping and AI-assisted auto-remediation .

---

### Real-World Cases

- **Pre-deployment gate** — Running `express-audit` and `npm audit` in CI blocks releases with critical findings.
- **Compliance evidence** — SOC 2 and PCI-DSS audits require documented, repeatable security review processes with severity-mapped findings.
- **Post-incident review** — After a breach, an audit traces the exploited weakness back to a missing control and verifies the fix.

---

## Core Concept 2: Penetration Testing

### Definitions

**Core Definition**
Penetration testing (pentesting) is the authorized, simulated exploitation of an Express.js application to discover vulnerabilities that automated scanners miss—particularly business-logic flaws, authentication bypasses, and chainable weaknesses.

**Technical Definition**
Express pentesting begins with **fingerprinting**: indicators include `X-Powered-By: Express`, stack traces mentioning `express`, `body-parser`, `qs`, `cookie-parser`, `express-session`, or `finalhandler`; cookies prefixed with `s:` (signed) or `j:` (JSON); `connect.sid` session cookies; and error pages leaking `Cannot GET /path` or `URIError` . Once Express is confirmed, the tester focuses on the **middleware chain**—parsers, proxy trust, session handling, and method tunneling—rather than framework core . High-value Express-specific test cases include **cookie secret brute-forcing** (`cookie-monster`), **query-string and URL-encoded parser abuse** for prototype pollution and NoSQL injection, **`trust proxy` misconfiguration** for IP allowlist bypass, **open redirects** via `res.redirect()`/`res.location()`, and **middleware ordering bypass** .

**Beginner-Friendly Explanation**
Pentesting is ethical hacking of your own app. You start by figuring out that it runs Express (the `X-Powered-By` header or an error page gives it away). Then you try the classic Express weak spots: can you guess the cookie-signing secret? Can you sneak nested objects into the query string? Can you bypass an IP allowlist by spoofing a header? The goal is to find the holes before the bad guys do.

---

### Purposes

- To discover vulnerabilities that static and dependency scanners cannot see, especially business-logic and authorization flaws.
- To validate whether a theoretical finding is actually exploitable in the running application.
- To test the effectiveness of the middleware chain—parsers, proxy trust, session handling, and routing order.
- To provide concrete proof-of-concept evidence for remediation prioritization.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Fingerprinting and Test Cases

```text
# Fingerprinting
curl -I https://target.com              # X-Powered-By: Express
curl https://target.com/nonexistent     # Cannot GET /nonexistent

# Cookie secret testing
cookie-monster -c <cookie> -s <secret> -n session
cookie-monster -b -f cookies.json -w custom.lst

# Trust proxy abuse
curl -H "X-Forwarded-For: 127.0.0.1" https://target.com/admin

# Parser abuse (prototype pollution)
curl -X POST https://target.com/api \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"isAdmin": true}}'

# Open redirect
curl -I "https://target.com/redirect?next=https://evil.com"
```

**Component Breakdown**

| Test | Target | Express-Specific Relevance |
|------|--------|---------------------------|
| Fingerprinting | `X-Powered-By`, error pages | Confirms Express and companion packages |
| Cookie secret | `s:` signed cookies | Weak secrets allow forgery of session/identity |
| Trust proxy | `X-Forwarded-For` | Bypasses IP allowlists and rate limits |
| Parser abuse | `req.query`, `express.urlencoded({ extended: true })` | Prototype pollution, NoSQL injection, mass assignment |
| Open redirect | `res.redirect(req.query.next)` | Phishing and token theft |
| Middleware bypass | Route ordering, path traversal | `app.use()` before auth middleware |

#### Syntax Rules

- **Focus on the middleware chain**, not framework core: most interesting Express bugs come from parsers, proxy trust, session handling, and method tunneling .
- **Test cookie secrets** with `cookie-monster`; if the secret is weak, an attacker can re-sign arbitrary cookie values .
- **Test `trust proxy` misconfiguration**: `app.set('trust proxy', true)` without validation lets an attacker set `X-Forwarded-For` to bypass IP allowlists or rate limits .
- **Test parser abuse**: `express.urlencoded({ extended: true })` uses `qs`-style parsing, which can unlock object injection, mass assignment, NoSQL injection, and prototype pollution .

#### Constraints and Limitations

- Pentesting requires written authorization and a defined scope; unauthorized testing is illegal.
- Dynamic testing cannot exhaustively cover all code paths; it complements, but does not replace, static analysis.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Testing Trust Proxy Misconfiguration

```bash
# Step 1: Confirm Express and trust proxy behavior
curl -I https://target.com

# Step 2: Attempt to bypass IP allowlist
curl -H "X-Forwarded-For: 127.0.0.1" https://target.com/admin
```

**Expected Output (vulnerable)**

```
HTTP/1.1 200 OK
X-Powered-By: Express
# /admin accessible from a spoofed localhost IP
```

**Expected Output (secure)**

```
HTTP/1.1 403 Forbidden
# /admin not accessible; trust proxy is configured to a known hop count
```

**Why This Output Occurs**

- With `trust proxy` set to `true` (trust all proxies), Express accepts the attacker-supplied `X-Forwarded-For` as the client IP, bypassing allowlists . Correct configuration sets `trust proxy` to the exact number of proxy hops.

---

#### Example 2: Testing Cookie Secret Strength

```bash
# Step 1: Capture a signed cookie
curl -I https://target.com/login -c cookies.txt

# Step 2: Brute-force the secret
cookie-monster -b -f cookies.json -w rockyou.txt

# Step 3: If secret found, re-sign an admin cookie
cookie-monster -e -f admin_cookie.json -k found_secret
```

**Expected Output (weak secret)**

```
[+] Secret found: "secret"
[+] Signed cookie: s:{"role":"admin"}.<valid_signature>
```

**Expected Output (strong secret)**

```
[-] No secret found in wordlist
```

**Why This Output Occurs**

- Express signs cookies with `cookie-parser` or `express-session`; a weak or hardcoded secret allows an attacker to forge signed cookies and escalate privileges .

---

### Real-World Cases

- **Bug bounty programs** — Express fingerprinting and parser abuse are standard recon and exploitation paths.
- **Red-team engagements** — Testing middleware ordering, `trust proxy`, and cookie secrets reveals privilege-escalation chains.
- **Post-audit validation** — Confirming that a fixed open redirect or CORS misconfiguration is no longer exploitable.

---

## Core Concept 3: Vulnerability Scanning

### Definitions

**Core Definition**
Vulnerability scanning is the automated, repeatable inspection of an Express.js application—its dependencies, source code, configuration, and running surface—against known vulnerability databases and security rules.

**Technical Definition**
Vulnerability scanning spans four layers: **SCA** (Software Composition Analysis) checks dependencies against CVE/GHSA advisories (`npm audit`, Snyk, OSV-Scanner, `vulnfind`); **SAST** (Static Application Security Testing) analyzes source code for insecure patterns (`express-audit`, `eslint-plugin-express-security`, Semgrep, `vulnfind` data-flow analysis); **DAST** (Dynamic Application Security Testing) probes the running application; and **secret scanning** detects credentials committed to source control (`vulnfind --history`) . Modern scanners such as `vulnfind` enrich findings with **CISA KEV** (known exploited vulnerabilities) and **EPSS** (exploit probability), rank results by what to fix first, and trace user input through the application to confirm whether a vulnerable function is actually reachable .

**Beginner-Friendly Explanation**
Vulnerability scanning is the automated smoke detector for your app. It watches your dependencies for known bugs, reads your code for dangerous patterns, scans your running site for exposed files and missing headers, and even checks whether secrets got committed to Git. It tells you what to fix first, not just what is wrong.

---

### Purposes

- To continuously detect known CVEs in dependencies before they are exploited.
- To flag Express-specific code patterns (missing Helmet, permissive CORS, unbounded body parsers, missing `httpOnly`) at development time.
- To verify that secrets are not committed to source control or served to clients.
- To rank findings by actual exploitability and business impact.

---

### Syntax Rules and Structure

#### Complete General Syntax: Scanning Stack

```bash
# SCA — dependencies
npm audit --audit-level=high --omit=dev
snyk test --severity-threshold=high --fail-on=upgradable
vulnfind ~/code/my-app

# SAST — source code
npx express-audit
npx eslint --plugin express-security .

# Secret scanning
vulnfind --history ~/code/my-app

# DAST — running application
vulnfind https://my-site.com --active
```

**Component Breakdown**

| Layer | Tools | What It Finds |
|-------|-------|---------------|
| SCA | `npm audit`, Snyk, OSV-Scanner, `vulnfind` | CVEs, malicious packages, license conflicts |
| SAST | `express-audit`, `eslint-plugin-express-security` | Missing Helmet, permissive CORS, insecure cookies, missing JWT expiry |
| Secret scanning | `vulnfind --history`, TruffleHog | Hardcoded credentials in source and Git history |
| DAST | `vulnfind <url> --active`, OWASP ZAP | Reflected XSS, SQL errors, open redirects, exposed files |

#### Syntax Rules

- **Rank by exploitability**, not just CVSS: `vulnfind` enriches with CISA KEV and EPSS to prioritize what is actually being exploited .
- **Use AST-based SAST**, not regex: `express-audit` inspects typed AST nodes to detect missing options in `res.cookie()`, `jwt.sign()`, and `cors()` .
- **Scan Git history for secrets**, not just the working tree .
- **Run DAST only against systems you own or are authorized to test**.

#### Constraints and Limitations

- SCA reports **presence**, not **reachability**; a CVE in a dependency you never call is less urgent than one in a hot path.
- SAST misses cross-file middleware registration and dependency-injection patterns .
- No scanner replaces human review for business-logic and authorization flaws.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Scanning Pipeline in CI

```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci

      - name: SCA — dependencies
        run: npm audit --audit-level=high --omit=dev

      - name: SAST — Express-specific rules
        run: npx express-audit

      - name: SAST — ESLint security rules
        run: npx eslint --plugin express-security .

      - name: Deep scan with ranked findings
        run: npx vulnfind .
```

**Expected Output**

```
# npm audit
found 0 vulnerabilities

# express-audit
src/app.js:25:3  CRITICAL CORS001  cors() called without origin allowlist

# vulnfind
Rank  Finding                          Severity  Reason
1     CORS misconfiguration            CRITICAL  Exploitable, reachable
2     Express 4.18.2 (CVE-2024-29041) HIGH      CISA KEV listed
3     Hardcoded JWT secret             HIGH      Detectable in source
```

**Why This Output Occurs**

- Each scanner operates at a different layer; `vulnfind` ranks findings by exploitability and reachability, while `express-audit` and ESLint catch application-level Express misconfigurations .

---

### Real-World Cases

- **Every Node.js CI pipeline** — `npm audit` is the zero-cost baseline; Snyk and `vulnfind` add depth and ranking.
- **Pre-commit hooks** — Secret scanning catches credentials before they reach the repository.
- **Runtime DAST** — Scheduled active scans detect open redirects and reflected XSS in deployed applications.

---

## Core Concept 4: Security Auditing Best Practices

### Definitions

**Core Definition**
Security auditing best practices are the operational and methodological patterns that keep Express.js security reviews accurate, actionable, repeatable, and integrated into the development lifecycle.

**Technical Definition**
Best practices include: **auditing at the AST level** for Express-specific patterns; **triangulating SCA, SAST, DAST, and manual review**; **mapping every finding to CWE/OWASP/CVSS** with a concrete fix; **prioritizing by reachability and exploitability** (CISA KEV, EPSS); **running scans on every pull request** with a severity gate; **keeping Express on a maintained version** (4.21.2+ or 5.x) and companion packages patched; **hardening configuration** (Helmet, CORS allowlist, body limits, `trust proxy`, cookie flags); and **verifying fixes** rather than assuming remediation succeeded .

**Beginner-Friendly Explanation**
Best practices are the habits that make audits actually useful. Scan your code at a deep level, not just with a text search. Use several tools because each catches different things. Always say how serious a finding is and exactly how to fix it. Run the scans automatically on every change. And verify that the fix really closed the hole.

---

### Purposes

- To maximize coverage while minimizing false positives and alert fatigue.
- To ensure findings are prioritized by real-world risk, not just theoretical severity.
- To make security review a continuous, automated part of the development workflow.
- To maintain an auditable trail of findings, fixes, and verification for compliance.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Audit Pipeline

```text
1. Version floor: Express 4.21.2+ or 5.x
2. Configuration: Helmet, CORS allowlist, body limits, trust proxy, secure cookies
3. SAST: express-audit + eslint-plugin-express-security on every PR
4. SCA: npm audit --audit-level=high --omit=dev on every PR
5. Deep scan: vulnfind . (AST + data-flow + secrets + config)
6. DAST: vulnfind <url> --active (scheduled)
7. Verify: re-run all scanners after fix; confirm headers and auth behavior
```

**Component Breakdown**

| Practice | Implementation | Express Relevance |
|----------|---------------|-------------------|
| AST-level SAST | `express-audit`, `eslint-plugin-express-security` | Detects missing `httpOnly`, no JWT expiry, permissive CORS  |
| Triangulation | SCA + SAST + DAST + manual | No single layer catches everything  |
| Severity mapping | CWE + OWASP + CVSS | Enables compliance and prioritization  |
| Reachability ranking | `vulnfind` with CISA KEV/EPSS | Fix what is actually exploitable first  |
| Continuous gating | CI on every PR | Prevents regressions |
| Version floor | Express 4.21.2+ or 5.x | Carries redirect, XSS, and path-to-regexp fixes  |
| Verify | Re-run scanners, confirm fixes | Prevents false closure |

#### Syntax Rules

- **Do not report theoretical vulnerabilities with no realistic attack path** .
- **Do not ignore medium/low findings**; document them even if not blocking .
- **Do not recommend security through obscurity** .
- **Do not skip dependency CVE checks** .
- **Always provide a concrete fix**, not just a warning .
- **Keep Express patched**: 4.21.2+ or 5.x; ensure lockfile resolves patched `qs`, `send`, `serve-static`, and `path-to-regexp` .

#### Constraints and Limitations

- Automation cannot replace human judgment for business-logic and authorization flaws; manual review of access-control paths is essential .
- A clean scan does not prove the absence of vulnerabilities; it proves the absence of **known detectable** vulnerabilities.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: End-to-End Hardened Audit Workflow

```yaml
name: Full Security Audit
on:
  pull_request:
  schedule:
    - cron: '0 6 * * 1'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci

      - name: Dependency scan (SCA)
        run: npm audit --audit-level=high --omit=dev

      - name: Express AST audit (SAST)
        run: npx express-audit

      - name: Express security lint (SAST)
        run: npx eslint --plugin express-security --rule 'express-security/require-helmet: error' .

      - name: Deep ranked scan (SCA + SAST + secrets + config)
        run: npx vulnfind .

      - name: Dynamic scan (DAST)
        run: npx vulnfind https://staging.example.com --active
        continue-on-error: true
```

**Expected Output**

```
✓ npm audit: 0 high vulnerabilities in production dependencies
✓ express-audit: 2 findings (1 critical, 1 high) → build fails
✓ eslint: require-helmet passed (helmet() present)
✓ vulnfind: ranked findings with CISA KEV/EPSS
✓ DAST: no reflected XSS or open redirects detected
```

**Why This Output Occurs**

- Each layer contributes unique evidence: SCA catches vulnerable packages, AST-based SAST catches Express misconfigurations, deep scanning ranks by exploitability, and DAST validates the running surface. The build fails on critical SAST findings, enforcing remediation before merge.

---

### Real-World Cases

- **Production Express APIs** — CI-gated SCA and SAST with DAST on staging, plus version-floor enforcement.
- **Compliance-driven organizations** — Severity-mapped findings and verification evidence for SOC 2 and PCI-DSS audits.
- **Security-conscious teams** — AST-level rules and reachability ranking reduce alert fatigue while catching high-impact Express misconfigurations.

---

## References

- Express.js Security Guide (2026): CVEs, Safe Config, SCA — https://safeguard.sh/resources/blog/express-js-security-guide
- Broken Access Control in Express.js: OWASP #1 Fixed — https://safeguard.sh/resources/blog/broken-access-control-express-nodejs
- NodeJS Express — HackTricks — https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/nodejs-express.html
- `express-audit` — Static Security Analysis for Express.js — https://www.npmjs.com/package/express-audit
- `eslint-plugin-express-security` — Express Security ESLint Rules — https://www.npmjs.com/package/eslint-plugin-express-security
- `vulnfind` — Zero-Dependency Security Scanner — https://pypi.org/project/vulnfind/
- Security Audit Skill — 4-Phase Workflow (Discover → Categorize → Remediate → Verify) — https://github.com/gohypergiant/standard-toolkit/tree/main/.agents/skills/accelint-security-best-practices
- OWASP Top 10 Node.js: 12 Steps (2026) — https://shattered.io
- OWASP Node.js Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html
- npm audit Documentation — https://docs.npmjs.com/cli/v11/commands/npm-audit