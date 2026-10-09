# Node.js-Specific Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Node.js-specific security encompasses the vulnerabilities, attack vectors, and defensive controls unique to the Node.js runtime, its package ecosystem (npm), and its asynchronous, single-threaded execution model.

**Technical Definition:** Node.js-specific security addresses risks that arise from: (1) the npm dependency ecosystem (supply chain attacks, typosquatting, transitive vulnerabilities); (2) JavaScript's prototype-based object model (prototype pollution); (3) the `child_process` module (command injection); (4) the single-threaded event loop (ReDoS, blocking operations); (5) environment variable and secret management; and (6) the runtime's permission model (introduced experimentally in Node.js v20, stable in v23). These risks are distinct from generic web vulnerabilities because they exploit Node.js-specific APIs, idioms, and ecosystem characteristics.

**Beginner-Friendly Explanation:** Think of Node.js as a workshop with power tools. The tools (npm packages) are incredibly useful, but some might be defective or even booby-trapped (**malicious packages**). The workshop's electrical system (**event loop**) can be overloaded if one tool jams (**ReDoS**). The blueprints (**prototypes**) that all tools share can be tampered with (**prototype pollution**). If you let a stranger operate the forklift (**child_process**), they might drive it somewhere dangerous (**command injection**). And if you leave the keys to the safe (**environment secrets**) lying around, anyone can take them. This cheat sheet teaches you how to secure each of these Node.js-specific risks.

### Key Characteristics

- **Ecosystem-driven:** The npm ecosystem's size and transitive dependencies create unique supply chain risks.
- **Language-specific:** Prototype pollution exploits JavaScript's prototype chain.
- **Runtime-specific:** The event loop, `child_process`, and the permission model are Node.js-specific.
- **Transitive risk:** Most vulnerabilities come from indirect dependencies, not direct ones.
- **Mitigable:** npm audit, lockfiles, provenance, and the permission model provide defenses.
- **Evolving:** Node.js adds new security features with each major release.

### Prerequisites

- **Node.js fundamentals:** Modules, `require`/`import`, `package.json`, npm.
- **JavaScript object model:** Prototypes, `__proto__`, `Object.create()`, property descriptors.
- **Child processes:** `child_process.exec`, `spawn`, `execFile`, `fork`.
- **Regular expressions:** Backtracking, catastrophic backtracking, regex engines.
- **Environment variables:** `process.env`, `.env` files, dotenv.
- **npm ecosystem:** `package.json`, `package-lock.json`, `npm audit`, `npm ci`.
- **Node.js permission model:** `--allow-fs-read`, `--allow-child-process`, etc.

### Related Programming Areas

- **Web security:** Many Node.js vulnerabilities (command injection, ReDoS) are also web vulnerabilities.
- **Supply chain security:** SBOM, provenance, SLSA, sigstore.
- **DevSecOps:** SAST, DAST, SCA (Software Composition Analysis), secret scanning.
- **Container security:** Docker, Kubernetes, and least-privilege containers.
- **Runtime security:** The Node.js permission model, seccomp, and sandboxing.

### Core Concepts

1. **Unsafe Dependency Usage** — mitigating vulnerabilities nested within heavily nested `node_modules` paths.
2. **Prototype Pollution** — safeguarding object blueprints using `Object.create(null)` or deep-freeze libraries.
3. **Command Injection** — sanitizing external variables passed into unsafe child processes like `exec` and `spawn`.
4. **Malicious Packages** — blocking typosquatting and compromised open-source library installations.
5. **Environment-Secret Exposure** — preventing keys from leaking into version control repositories or process logs.
6. **Regular Expression Denial of Service (ReDoS)** — preventing event loop blocking caused by catastrophic backtracking.
7. **The Node.js Permission Model** — leveraging native runtime execution restrictions like `--allow-fs-read` and `--allow-child-process`.

---

## Core Concept 1: Unsafe Dependency Usage

### Definitions

**Core Definition:** Unsafe dependency usage refers to the risk introduced by vulnerable, outdated, or malicious packages in the npm dependency tree, including transitive dependencies that developers do not directly install.

**Technical Definition:** The npm ecosystem has over 2 million packages, and a typical Node.js application has hundreds or thousands of transitive dependencies. Each dependency is a potential attack vector: vulnerable packages (Log4Shell-style), compromised maintainer accounts (event-stream, ua-parser-js), typosquatting (crossenv vs. cross-env), and dependency confusion. Prevention requires: `package-lock.json` for deterministic installs, `npm ci` for CI/CD, `npm audit` for vulnerability detection, SCA tools (Snyk, Dependabot, Socket), provenance verification (`npm audit signatures`), and minimizing dependencies.

**Beginner-Friendly Explanation:** Imagine your app is a house built from LEGO bricks. You buy a big set (your direct dependency), but that set includes dozens of smaller bags (transitive dependencies). If one bag contains a defective or booby-trapped brick, your whole house is at risk. npm's dependency tree can be hundreds of levels deep — you may not even know what's in there. The fix: lock your dependencies, audit them regularly, and use tools that scan the entire tree.

### Purposes

- To prevent vulnerable packages from introducing exploitable flaws.
- To detect and remediate known CVEs in the dependency tree.
- To prevent supply chain attacks (compromised maintainers, malicious updates).
- To ensure deterministic, reproducible builds.
- To comply with security standards (SOC 2, PCI DSS, ISO 27001).

### Syntax Rules and Structure

#### Dependency Management Rules

| Rule | Implementation |
|------|----------------|
| **Commit `package-lock.json`** | Deterministic installs |
| **Use `npm ci` in CI/CD** | Installs exactly from lockfile |
| **Run `npm audit` regularly** | Detect known vulnerabilities |
| **Use `npm audit --production`** | Ignore dev-only vulnerabilities |
| **Use SCA tools** | Snyk, Dependabot, Socket, npm audit |
| **Enable Dependabot alerts** | GitHub-native vulnerability alerts |
| **Verify package provenance** | `npm audit signatures` |
| **Pin exact versions** | Or use `~`/`^` carefully |
| **Minimize dependencies** | Fewer dependencies = smaller attack surface |
| **Review new dependencies** | Check maintainers, downloads, age |

#### npm audit Commands

```bash
# Audit for vulnerabilities
npm audit

# Audit production dependencies only
npm audit --production

# Fix automatically (may break)
npm audit fix

# Fix with breaking changes
npm audit fix --force

# Verify package signatures (provenance)
npm audit signatures

# Check outdated packages
npm outdated
```

#### Syntax Rules

- **Always commit `package-lock.json`** — deterministic installs.
- **Always use `npm ci` in CI/CD** — never `npm install`.
- **Run `npm audit` on every build** — fail on high/critical.
- **Use SCA tools** — Snyk, Dependabot, Socket, GitHub Advisory Database.
- **Enable Dependabot** — automated PRs for vulnerabilities.
- **Verify signatures** — `npm audit signatures` for provenance.
- **Pin versions in production** — avoid `^` for critical dependencies.
- **Review new dependencies** — check npm page, GitHub, maintainers.
- **Minimize dependencies** — every dependency is a risk.
- **Monitor for supply chain attacks** — Socket, Phylum, Snyk.

#### Constraints and Limitations

- **Transitive dependencies are invisible** — you cannot easily audit hundreds of packages.
- **`npm audit fix` can break your app** — test thoroughly.
- **Zero-day vulnerabilities** — may not be in the advisory database yet.
- **Maintainer compromise** — even trusted packages can be hijacked.
- **Dependency confusion** — private package names can be hijacked on public npm.
- **Performance** — SCA tools add build time.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Dependency Workflow

```yaml
# .github/workflows/security.yml
name: Security Audit
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *' # Daily

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies (deterministic)
        run: npm ci

      - name: Audit production dependencies
        run: npm audit --production --audit-level=high

      - name: Verify package signatures
        run: npm audit signatures

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Dependency Review
        uses: actions/dependency-review-action@v4
```

```json
// package.json — pin critical dependencies
{
  "dependencies": {
    "express": "4.19.2",
    "helmet": "7.1.0",
    "prisma": "5.22.0"
  },
  "overrides": {
    "semver": "^7.6.0"
  },
  "scripts": {
    "audit": "npm audit --production --audit-level=high",
    "audit:fix": "npm audit fix",
    "outdated": "npm outdated"
  }
}
```

**Expected behaviour:** CI fails if high/critical vulnerabilities are found. Signatures are verified. Snyk scans for issues. Dependabot creates PRs for updates.

**Why this works:** Multiple layers — lockfile, `npm ci`, `npm audit`, signature verification, and SCA tools. The workflow runs on every push and daily.

### Real-World Cases

- **event-stream (2018):** A malicious maintainer added code to steal cryptocurrency wallets.
- **ua-parser-js (2021):** A popular package was compromised to install cryptominers.
- **colors and faker (2022):** The maintainer intentionally broke both packages.
- **Log4Shell (2021):** A critical vulnerability in Log4j (Java, but the principle applies).
- **node-ipc (2022):** A maintainer added protestware that overwrote files.

---

## Core Concept 2: Prototype Pollution

### Definitions

**Core Definition:** Prototype pollution is a vulnerability where an attacker injects properties into `Object.prototype` (or another prototype), affecting all objects in the application.

**Technical Definition:** Prototype pollution (CWE-1321) exploits recursive merge functions, `Object.assign()`, or `JSON.parse()` with `__proto__` keys to add properties to `Object.prototype`. Once polluted, every object inherits the malicious property, which can lead to: authentication bypass, denial of service, remote code execution (via gadget chains), or property tampering. Prevention requires: `Object.create(null)` for dictionaries, `Object.freeze()` on prototypes, safe merge libraries (lodash ≥4.17.21), validating `__proto__`/`constructor`/`prototype` keys, and using `Map` instead of objects for user-controlled keys.

**Beginner-Friendly Explanation:** Imagine every LEGO brick shares a common blueprint. If someone sneaks into the blueprint room and adds a new instruction ("all bricks must also be red"), every brick in the world turns red. Prototype pollution is the same: JavaScript objects share prototypes (blueprints). If an attacker can modify `Object.prototype`, every object in the application inherits the change. The fix: use `Object.create(null)` for dictionaries, freeze prototypes, and never merge untrusted objects recursively.

### Purposes

- To prevent attackers from injecting properties into shared prototypes.
- To prevent authentication bypass and privilege escalation.
- To prevent denial of service.
- To prevent remote code execution via gadget chains.
- To ensure that user-controlled data cannot alter global object behaviour.

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — recursive merge
function merge(target: any, source: any) {
  for (const key in source) {
    if (typeof source[key] === 'object') {
      target[key] = merge(target[key] ?? {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

const userInput = JSON.parse('{"__proto__": {"isAdmin": true}}');
merge({}, userInput);
console.log({}.isAdmin); // true — Object.prototype polluted!
```

#### Safe Code

```typescript
// ✅ SAFE — Object.create(null) for dictionaries
const dictionary = Object.create(null);
dictionary['key'] = 'value';
// dictionary has no prototype — no pollution possible

// ✅ SAFE — check for dangerous keys
function safeMerge(target: any, source: any): any {
  for (const key of Object.keys(source)) {
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      continue; // Skip dangerous keys
    }
    if (typeof source[key] === 'object' && source[key] !== null) {
      target[key] = safeMerge(target[key] ?? {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// ✅ SAFE — freeze Object.prototype (defense in depth)
Object.freeze(Object.prototype);

// ✅ SAFE — use Map instead of objects
const map = new Map<string, unknown>();
map.set('key', 'value');

// ✅ SAFE — use safe libraries
import { merge } from 'lodash'; // lodash ≥4.17.21 is safe
```

#### Syntax Rules

- **Use `Object.create(null)`** for dictionaries with user-controlled keys.
- **Use `Map`** instead of objects for user-controlled keys.
- **Skip `__proto__`, `constructor`, `prototype`** in merge functions.
- **Freeze `Object.prototype`** — defense in depth.
- **Use safe merge libraries** — lodash ≥4.17.21, or `deepmerge` with `clone: false`.
- **Validate JSON schemas** — reject unexpected properties.
- **Never use `eval` or `Function`** with user input.
- **Use `Object.hasOwn()`** instead of `in` for property checks.
- **Test with prototype pollution payloads** — `__proto__`, `constructor.prototype`.

#### Constraints and Limitations

- **Prototype pollution can be subtle** — often in third-party libraries.
- **Freezing `Object.prototype`** can break some libraries.
- **Gadget chains are hard to predict** — a polluted property may not be exploited immediately.
- **JSON.parse itself is safe** — the vulnerability is in how you use the parsed object.
- **`Object.assign()` is safe** — but recursive merges are not.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing Prototype Pollution (Node.js)

```typescript
// ❌ VULNERABLE — recursive merge without checks
function vulnerableMerge(target: any, source: any): any {
  for (const key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      target[key] = vulnerableMerge(target[key] ?? {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

const malicious = JSON.parse('{"__proto__": {"isAdmin": true}}');
vulnerableMerge({}, malicious);
console.log(({} as any).isAdmin); // true — VULNERABLE

// ✅ SAFE — safe merge with dangerous key filtering
const DANGEROUS_KEYS = new Set(['__proto__', 'constructor', 'prototype']);

function safeMerge(target: Record<string, any>, source: Record<string, any>): Record<string, any> {
  for (const key of Object.keys(source)) {
    if (DANGEROUS_KEYS.has(key)) continue;

    const value = source[key];
    if (value && typeof value === 'object' && !Array.isArray(value)) {
      target[key] = safeMerge(
        Object.prototype.hasOwnProperty.call(target, key) ? target[key] : {},
        value,
      );
    } else {
      target[key] = value;
    }
  }
  return target;
}

const safe = safeMerge({}, malicious);
console.log(({} as any).isAdmin); // undefined — SAFE

// ✅ SAFE — Object.create(null) for user-controlled dictionaries
function createDictionary(input: Record<string, string>): Record<string, string> {
  const dict = Object.create(null);
  for (const key of Object.keys(input)) {
    if (DANGEROUS_KEYS.has(key)) continue;
    dict[key] = input[key];
  }
  return dict;
}

// ✅ SAFE — freeze Object.prototype (defense in depth)
Object.freeze(Object.prototype);
Object.freeze(Object.getPrototypeOf({}));

// ✅ SAFE — Map for user-controlled keys
function createMap(input: Record<string, string>): Map<string, string> {
  const map = new Map<string, string>();
  for (const [key, value] of Object.entries(input)) {
    map.set(key, value);
  }
  return map;
}
```

**Expected behaviour:**
- `vulnerableMerge()` pollutes `Object.prototype.isAdmin = true`.
- `safeMerge()` skips dangerous keys — no pollution.
- `createDictionary()` uses `Object.create(null)` — no prototype to pollute.
- Freezing `Object.prototype` prevents any further pollution.

**Why this works:** Multiple layers — dangerous key filtering, `Object.create(null)`, `Map`, and freezing prototypes. Each layer alone provides some protection; together, they provide defense in depth.

### Real-World Cases

- **lodash (CVE-2019-10744, CVE-2020-8203):** Prototype pollution in `merge`, `defaultsDeep`.
- **jQuery (CVE-2019-11358):** `$.extend(true, ...)` was vulnerable.
- **Express (CVE-2024-29041):** Prototype pollution via `qs`.
- **Kibana (CVE-2019-7609):** Prototype pollution leading to RCE.

---

## Core Concept 3: Command Injection

### Definitions

**Core Definition:** Command injection occurs when an attacker can execute arbitrary system commands by injecting shell metacharacters into input passed to a child process.

**Technical Definition:** Command injection (CWE-78) exploits `child_process.exec()`, `execSync()`, or `spawn()` with `shell: true`, which pass the command to a shell (`/bin/sh` or `cmd.exe`). Shell metacharacters (`;`, `|`, `&`, `$()`, `` ` ``, `\n`) allow attackers to chain additional commands. Prevention requires: using `execFile()` or `spawn()` without `shell: true`, passing arguments as an array (not concatenated), validating input against allow-lists, and avoiding the shell entirely. `execFile()` and `spawn()` (without `shell: true`) do not invoke a shell, so metacharacters are treated as literal arguments.

**Beginner-Friendly Explanation:** Imagine you ask a robot to "fetch the file named [your input]." If you say "report.pdf; also delete all files," the robot executes both commands. Command injection is the same: the application passes user input to a shell command. If the input contains shell metacharacters, the shell executes additional commands. The fix: use `execFile()` or `spawn()` with an argument array — the input is treated as a literal argument, not as shell syntax.

### Purposes

- To prevent arbitrary command execution on the server.
- To prevent data exfiltration, file deletion, and backdoor installation.
- To prevent lateral movement to other systems.
- To comply with OWASP Top 10 (A03:2021 — Injection).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
import { exec } from 'node:child_process';

// ❌ VULNERABLE — shell injection
app.get('/ping', (req, res) => {
  const host = req.query.host as string;
  exec(`ping -c 1 ${host}`, (err, stdout) => {
    res.send(stdout);
  });
  // Attacker sends: ?host=example.com; cat /etc/passwd
});

// ❌ VULNERABLE — spawn with shell: true
import { spawn } from 'node:child_process';
spawn(`ping -c 1 ${host}`, { shell: true });
```

#### Safe Code

```typescript
import { execFile, spawn } from 'node:child_process';
import { promisify } from 'node:util';

const execFileAsync = promisify(execFile);

// ✅ SAFE — execFile with argument array (no shell)
app.get('/ping', async (req, res) => {
  const host = req.query.host as string;

  // Validate input (allow-list)
  if (!/^[a-zA-Z0-9.-]+$/.test(host)) {
    return res.status(400).json({ error: 'Invalid hostname' });
  }

  try {
    const { stdout } = await execFileAsync('ping', ['-c', '1', host]);
    res.send(stdout);
  } catch (err) {
    res.status(500).json({ error: 'Ping failed' });
  }
});

// ✅ SAFE — spawn without shell
const child = spawn('ping', ['-c', '1', host]);
child.stdout.on('data', (data) => console.log(data.toString()));
```

#### Syntax Rules

- **Use `execFile()` or `spawn()` without `shell: true`** — no shell invocation.
- **Pass arguments as an array** — never concatenate into a command string.
- **Validate input against allow-lists** — reject unexpected characters.
- **Never use `exec()` with user input** — it always invokes a shell.
- **Never use `shell: true`** unless absolutely necessary.
- **Use `execFile()` for external commands** — it is the safest option.
- **Drop privileges** — run the application as a non-root user.
- **Use the Node.js permission model** — `--allow-child-process` restricts spawn.
- **Log all child process invocations** — for audit.
- **Sandbox child processes** — containers, seccomp, AppArmor.

#### Constraints and Limitations

- **Some commands require a shell** — pipes, redirects, and globbing.
- **Windows vs. POSIX** — shell metacharacters differ.
- **Argument injection** — even without a shell, arguments starting with `-` can be dangerous.
- **`execFile` on Windows** — `.bat` and `.cmd` files invoke `cmd.exe`, which can be vulnerable.
- **Environment variables** — `PATH` manipulation can redirect commands.
- **TOCTOU** — the file may change between validation and execution.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe Child Process Usage (Express)

```typescript
import express from 'express';
import { execFile, spawn } from 'node:child_process';
import { promisify } from 'node:util';

const execFileAsync = promisify(execFile);
const app = express();

// ❌ VULNERABLE
app.get('/unsafe/ping', (req, res) => {
  const host = req.query.host as string;
  exec(`ping -c 1 ${host}`, (err, stdout) => {
    if (err) return res.status(500).send(err.message);
    res.send(stdout);
  });
});

// ✅ SAFE — execFile with allow-list
app.get('/safe/ping', async (req, res) => {
  const host = req.query.host as string;

  // Allow-list: hostnames only
  if (!/^[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/.test(host)) {
    return res.status(400).json({ error: 'Invalid hostname' });
  }

  try {
    const { stdout } = await execFileAsync('ping', ['-c', '1', '-W', '2', host], {
      timeout: 5000,
      maxBuffer: 1024 * 1024,
    });
    res.send(stdout);
  } catch (err) {
    res.status(500).json({ error: 'Ping failed' });
  }
});

// ✅ SAFE — spawn for streaming
app.get('/safe/traceroute', (req, res) => {
  const host = req.query.host as string;

  if (!/^[a-zA-Z0-9.-]+$/.test(host)) {
    return res.status(400).json({ error: 'Invalid hostname' });
  }

  const child = spawn('traceroute', ['-m', '15', host], {
    timeout: 30000,
  });

  child.stdout.pipe(res);
  child.stderr.on('data', (data) => console.error(data.toString()));
  child.on('error', (err) => {
    if (!res.headersSent) res.status(500).json({ error: err.message });
  });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/unsafe/ping?host=example.com; cat /etc/passwd` executes both commands (vulnerable).
- `/safe/ping?host=example.com` returns the ping output.
- `/safe/ping?host=example.com; cat /etc/passwd` returns `400 Bad Request` (invalid hostname).
- `/safe/ping?host=$(whoami)` returns `400 Bad Request`.

**Why this works:** `execFile` does not invoke a shell, so metacharacters are treated as literal arguments. The allow-list validation rejects anything that is not a valid hostname. Timeouts and buffer limits prevent DoS.

### Real-World Cases

- **Web application firewalls:** Bypassed via command injection in image processing.
- **CI/CD pipelines:** Injected commands in build scripts.
- **Network tools:** Ping, traceroute, and DNS lookups.
- **Notable breaches:** Shellshock (2014), various IoT botnets.

---

## Core Concept 4: Malicious Packages

### Definitions

**Core Definition:** Malicious packages are npm packages that contain intentionally harmful code, often disguised as legitimate libraries or published under names similar to popular packages (typosquatting).

**Technical Definition:** Malicious packages exploit the npm ecosystem's trust model. Attack vectors include: **typosquatting** (names similar to popular packages — `crossenv` vs. `cross-env`), **dependency confusion** (publishing private package names on public npm), **compromised maintainer accounts** (attacker takes over a legitimate package), **protestware** (maintainer intentionally breaks the package), and **starjacking** (fake GitHub stars). Prevention requires: verifying package names, checking maintainers and download counts, using provenance (`npm audit signatures`), reviewing new dependencies, using SCA tools (Socket, Snyk), and configuring `.npmrc` for scoped registries.

**Beginner-Friendly Explanation:** Imagine a library where anyone can publish a book. Most books are legitimate, but some are fake — titled almost identically to popular books (typosquatting) but with malicious content inside. Malicious packages are the same: an attacker publishes a package with a name almost identical to a popular one, hoping developers will install it by mistake. The fix: verify package names carefully, check the author and download count, and use tools that scan for suspicious packages.

### Purposes

- To prevent installation of malicious packages.
- To detect typosquatting and dependency confusion.
- To verify package provenance and integrity.
- To reduce the risk of supply chain attacks.
- To comply with security standards.

### Syntax Rules and Structure

#### Package Verification Checklist

| Check | Why |
|-------|-----|
| **Exact name** | Typosquatting uses similar names |
| **Maintainer** | Check GitHub, npm profile |
| **Download count** | Popular packages are less likely malicious |
| **Age** | New packages are riskier |
| **Repository** | Verify the GitHub link works |
| **Provenance** | `npm audit signatures` |
| **Dependencies** | Fewer dependencies = less risk |
| **Scripts** | `postinstall` scripts are dangerous |
| **Typosquatting patterns** | `cross-env` vs. `crossenv` |

#### .npmrc Configuration

```ini
# .npmrc — security-hardened configuration
# Disable postinstall scripts (prevents malicious install scripts)
ignore-scripts=true

# Use exact versions
save-exact=true

# Enforce lockfile
package-lock=true

# Use a registry proxy for scanning
registry=https://registry.npmjs.org/

# For scoped private packages
@mycompany:registry=https://npm.mycompany.com/
```

#### Syntax Rules

- **Verify exact package names** — use the official docs.
- **Check maintainers and download counts** — popular packages are safer.
- **Use `npm audit signatures`** — verify provenance.
- **Enable `ignore-scripts=true`** — prevents malicious install scripts.
- **Use SCA tools** — Socket, Snyk, Phylum, Dependabot.
- **Configure scoped registries** — prevent dependency confusion.
- **Review new dependencies** — check GitHub, issues, and recent commits.
- **Minimize dependencies** — fewer packages = smaller attack surface.
- **Use `npm ci`** — deterministic installs.
- **Monitor for package compromises** — subscribe to security advisories.

#### Constraints and Limitations

- **Zero-day attacks** — a package can be compromised at any time.
- **Maintainer accounts can be hijacked** — 2FA helps but is not foolproof.
- **Download counts can be faked** — not a reliable signal.
- **`ignore-scripts=true`** can break legitimate packages.
- **Dependency confusion** requires careful registry configuration.
- **Socket and Snyk require accounts** — but have free tiers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure npm Configuration and Verification

```ini
# .npmrc
# Enforce lockfile
package-lock=true

# Use exact versions
save-exact=true

# Disable install scripts (prevents malicious postinstall)
ignore-scripts=true

# Scoped registry for private packages
@mycompany:registry=https://npm.mycompany.com/
```

```json
// package.json — overrides for security
{
  "name": "my-app",
  "version": "1.0.0",
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=10.0.0"
  },
  "scripts": {
    "preinstall": "npx npm-force-resolutions",
    "audit": "npm audit --production --audit-level=high",
    "audit:signatures": "npm audit signatures",
    "verify": "npm ci && npm run audit && npm run audit:signatures"
  },
  "overrides": {
    "semver": "^7.6.0",
    "minimist": "^1.2.8"
  }
}
```

```bash
# Verify package provenance (npm 9.5+)
npm audit signatures

# Check for typosquatting (Socket)
npx @socketsecurity/cli npm install express

# Scan for suspicious packages
npx snyk test

# Check package details before installing
npm view <package> maintainers
npm view <package> repository
npm view <package> time.created
```

**Expected behaviour:** `ignore-scripts=true` prevents malicious postinstall scripts. `npm audit signatures` verifies provenance. Snyk and Socket scan for vulnerabilities and suspicious packages. Scoped registries prevent dependency confusion.

**Why this works:** Multiple layers — configuration, verification, and scanning. `ignore-scripts` is particularly effective against malicious postinstall scripts.

### Real-World Cases

- **event-stream (2018):** Malicious `flatmap-stream` dependency stole cryptocurrency.
- **ua-parser-js (2021):** Compromised to install cryptominers and password stealers.
- **node-ipc (2022):** Protestware overwrote files on systems with certain IPs.
- **colors and faker (2022):** Maintainer intentionally broke both packages.
- **crossenv (2017):** Typosquatting of `cross-env` stole environment variables.

---

## Core Concept 5: Environment-Secret Exposure

### Definitions

**Core Definition:** Environment-secret exposure occurs when sensitive credentials (API keys, database passwords, tokens) are leaked into version control, logs, error messages, or other unintended locations.

**Technical Definition:** Secrets can leak through: **version control** (committing `.env` files), **logs** (`console.log(process.env)`), **error messages** (stack traces containing secrets), **CI/CD** (unmasked secrets in build logs), **client-side bundles** (secrets in frontend code), **Docker images** (secrets in layers), and **process listings** (`ps aux` showing command-line arguments). Prevention requires: `.gitignore` for `.env`, secret scanning (GitHub, GitGuardian, TruffleHog), environment variable masking in CI, secrets management (Vault, AWS Secrets Manager), and never logging secrets.

**Beginner-Friendly Explanation:** Imagine leaving your house key under the doormat. Environment-secret exposure is the same: developers leave API keys in `.env` files that get committed to Git, or in `console.log` statements that end up in logs. The fix: never commit secrets, use secret scanning tools, store secrets in a vault, and redact them from logs.

### Purposes

- To prevent credentials from leaking into version control.
- To prevent secrets from appearing in logs and error messages.
- To prevent secrets from being exposed in client-side bundles.
- To comply with security standards (PCI DSS, SOC 2, HIPAA).
- To enable secure secret rotation and management.

### Syntax Rules and Structure

#### .gitignore Configuration

```gitignore
# .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
*.p12
secrets/
config/secrets.json
```

#### Secret Scanning

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scanning
on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  trufflehog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: TruffleHog Secret Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### Safe Logging

```typescript
// ❌ VULNERABLE — logs secrets
console.log('Config:', process.env);
console.error('Error:', err); // May contain secrets in the message

// ✅ SAFE — redact secrets
const SENSITIVE_KEYS = ['password', 'secret', 'token', 'key', 'authorization', 'cookie'];

function redact(obj: any): any {
  if (typeof obj !== 'object' || obj === null) return obj;

  const redacted: any = Array.isArray(obj) ? [] : {};
  for (const [key, value] of Object.entries(obj)) {
    if (SENSITIVE_KEYS.some((k) => key.toLowerCase().includes(k))) {
      redacted[key] = '[REDACTED]';
    } else if (typeof value === 'object') {
      redacted[key] = redact(value);
    } else {
      redacted[key] = value;
    }
  }
  return redacted;
}

logger.info('Request received', { headers: redact(req.headers), body: redact(req.body) });
```

#### Syntax Rules

- **Never commit `.env` files** — add to `.gitignore`.
- **Use `.env.example`** — with placeholder values.
- **Use secret scanning** — TruffleHog, Gitleaks, GitHub Secret Scanning.
- **Redact secrets in logs** — use a redaction function.
- **Use secrets management** — Vault, AWS Secrets Manager, Azure Key Vault.
- **Rotate secrets regularly** — and immediately after any exposure.
- **Never log `process.env`** — it contains all environment variables.
- **Never include secrets in client-side code** — use a backend proxy.
- **Mask secrets in CI/CD** — GitHub Actions secrets are masked by default.
- **Use `.dockerignore`** — prevent secrets from being copied into images.

#### Constraints and Limitations

- **Secret scanning can have false positives** — review alerts carefully.
- **Git history retains secrets** — removing a file does not remove the secret from history.
- **Secrets in environment variables** are visible to `ps` and child processes.
- **Third-party services** may log secrets — review their policies.
- **Rotation is complex** — requires coordination across services.
- **Backups and snapshots** may contain secrets.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Secret Management (Node.js)

```typescript
// config.ts — centralised configuration with validation
import { z } from 'zod';

const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  REDIS_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
});

const result = EnvSchema.safeParse(process.env);
if (!result.success) {
  console.error('Invalid environment configuration:', result.error.issues);
  process.exit(1);
}

export const config = result.data;

// ❌ VULNERABLE — never log the full config
console.log('Config:', config);

// ✅ SAFE — log only non-sensitive fields
console.log('Config loaded:', {
  NODE_ENV: config.NODE_ENV,
  PORT: config.PORT,
  DATABASE_URL: config.DATABASE_URL.replace(/\/\/.*@/, '//***@'),
});
```

```typescript
// logger.ts — redacting logger
import pino from 'pino';

const REDACT_PATHS = [
  'password',
  'token',
  'secret',
  'apiKey',
  'authorization',
  'req.headers.authorization',
  'req.headers.cookie',
  '*.password',
  '*.token',
  '*.secret',
];

export const logger = pino({
  redact: {
    paths: REDACT_PATHS,
    censor: '[REDACTED]',
  },
  level: process.env.LOG_LEVEL ?? 'info',
});
```

```typescript
// error-handler.ts — safe error responses
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  // Log the full error internally (redacted)
  logger.error({ err, path: req.path }, 'Unhandled error');

  // Return a generic message to the client
  res.status(500).json({
    error: 'Internal Server Error',
    message: process.env.NODE_ENV === 'production'
      ? 'An unexpected error occurred'
      : err.message, // Only in development
  });
});
```

**Expected behaviour:**
- Missing or invalid environment variables cause the application to exit at startup.
- The logger redacts sensitive fields.
- Error responses do not leak secrets.
- Only non-sensitive fields are logged at startup.

**Why this works:** Schema validation ensures all secrets are present. The logger redacts sensitive fields. Error responses are sanitised. Only non-sensitive fields are logged.

### Real-World Cases

- **Uber (2016):** AWS keys committed to a public GitHub repository.
- **Twitch (2021):** Source code and secrets leaked via a server misconfiguration.
- **Samsung (2023):** ChatGPT usage led to secret leakage.
- **Toyota (2023):** Cloud environment misconfiguration exposed vehicle data.
- **Various:** Countless `.env` files committed to public repositories.

---

## Core Concept 6: Regular Expression Denial of Service (ReDoS)

### Definitions

**Core Definition:** ReDoS is a denial-of-service attack that exploits catastrophic backtracking in regular expressions, causing the event loop to block for an extended period.

**Technical Definition:** ReDoS (CWE-1333) occurs when a regex with nested quantifiers (`(a+)+`, `(a|a)*`, `(.*a){x}`) is applied to a specially crafted input. The regex engine explores an exponential number of backtracking paths, consuming CPU until the process becomes unresponsive. Because Node.js is single-threaded, a single ReDoS regex blocks the entire event loop, affecting all concurrent requests. Prevention requires: simplifying regexes (avoid nested quantifiers), using atomic groups (where supported), limiting input length, using `RE2` (a linear-time regex engine), and running regex processing in a worker thread.

**Beginner-Friendly Explanation:** Imagine a maze with exponentially many paths. A normal person finds the exit quickly. But a malicious maze designer creates a maze where every path leads to a dead end except one — and the exit is at the end of the last path. ReDoS is the same: a malicious input causes the regex engine to try exponentially many combinations before giving up. The fix: use simpler regexes, limit input length, or use a regex engine that does not backtrack (RE2).

### Purposes

- To prevent the event loop from being blocked by malicious regex input.
- To prevent denial of service.
- To ensure the application remains responsive under attack.
- To comply with OWASP Top 10 (A06:2021 — Vulnerable and Outdated Components).

### Syntax Rules and Structure

#### Vulnerable Regex Patterns

| Pattern | Problem | Example Input |
|---------|---------|---------------|
| `(a+)+$` | Nested quantifier | `aaaa...a!` |
| `(a\|a)*$` | Alternation overlap | `aaaa...a!` |
| `(.*a){x}` | Nested quantifier | Long string without `a` |
| `(a\|aa)+$` | Alternation overlap | `aaaa...a!` |
| `([a-zA-Z]+)*$` | Nested quantifier | `aaaa...1` |
| `^(\w+\s?)*$` | Nested quantifier | Long string with spaces |

#### Safe Regex Patterns

```typescript
// ❌ VULNERABLE — catastrophic backtracking
const vulnerable = /^(\w+\s?)*$/;
vulnerable.test('a'.repeat(30) + '!'); // Blocks for seconds/minutes

// ✅ SAFE — simplified regex
const safe = /^\w+(\s\w+)*$/;
safe.test('a'.repeat(30) + '!'); // Fast
```

#### RE2 — Linear-Time Regex Engine

```typescript
import RE2 from 're2';

// ✅ SAFE — RE2 does not backtrack
const re2 = new RE2(/^(\w+\s?)*$/);
re2.test('a'.repeat(10000) + '!'); // Fast, no backtracking

// ✅ SAFE — use RE2 for all user-controlled regexes
function safeMatch(pattern: string, input: string): boolean {
  try {
    const re2 = new RE2(pattern);
    return re2.test(input);
  } catch {
    return false;
  }
}
```

#### Syntax Rules

- **Avoid nested quantifiers** — `(a+)+`, `(a*)*`, `(a|a)*`.
- **Avoid overlapping alternations** — `(a|ab)+`.
- **Use anchored regexes** — `^...$` limits backtracking.
- **Limit input length** — `input.slice(0, 1000)`.
- **Use RE2** — linear-time regex engine.
- **Run regex in worker threads** — for untrusted patterns.
- **Test regexes with ReDoS tools** — `safe-regex`, `vuln-regex-detector`.
- **Use `safe-regex`** — static analysis for vulnerable patterns.
- **Timeout regex execution** — not natively supported; use workers.

#### Constraints and Limitations

- **RE2 does not support all regex features** — backreferences, lookaround.
- **`safe-regex` has false negatives** — it is a heuristic.
- **Worker threads add complexity** — message passing overhead.
- **Input length limits may break legitimate use cases** — balance security and functionality.
- **Regex engines differ** — V8's engine is vulnerable; RE2 is not.
- **Timeout is not built-in** — must be implemented with workers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing ReDoS (Node.js)

```typescript
import express from 'express';
import RE2 from 're2';
import safeRegex from 'safe-regex';

const app = express();

// ❌ VULNERABLE — catastrophic backtracking
app.get('/unsafe/validate', (req, res) => {
  const input = req.query.input as string;
  const regex = /^(\w+\s?)*$/;
  const start = Date.now();
  const result = regex.test(input);
  res.json({ result, time: Date.now() - start });
  // Attacker sends: ?input=aaaa...a!
});

// ✅ SAFE — simplified regex
app.get('/safe/validate', (req, res) => {
  const input = (req.query.input as string).slice(0, 1000); // Limit length
  const regex = /^\w+(\s\w+)*$/; // Simplified
  const result = regex.test(input);
  res.json({ result });
});

// ✅ SAFE — RE2 (linear-time)
app.get('/re2/validate', (req, res) => {
  const input = (req.query.input as string).slice(0, 10000);
  try {
    const re2 = new RE2(/^(\w+\s?)*$/);
    const result = re2.test(input);
    res.json({ result });
  } catch (err) {
    res.status(400).json({ error: 'Invalid pattern' });
  }
});

// ✅ SAFE — static analysis with safe-regex
app.post('/patterns/validate', (req, res) => {
  const { pattern } = req.body;

  if (!safeRegex(pattern)) {
    return res.status(400).json({ error: 'Unsafe regex pattern' });
  }

  try {
    const re2 = new RE2(pattern);
    res.json({ safe: true });
  } catch (err) {
    res.status(400).json({ error: 'Invalid pattern' });
  }
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/unsafe/validate?input=aaaaaaaaaaaaaaaaaaaaaaaaaaaaa!` blocks for seconds.
- `/safe/validate?input=...` returns quickly (simplified regex, length limit).
- `/re2/validate?input=...` returns quickly (RE2 does not backtrack).
- `/patterns/validate` rejects unsafe patterns.

**Why this works:** Multiple layers — simplified regex, input length limits, RE2, and static analysis. Each layer alone provides some protection; together, they prevent ReDoS.

### Real-World Cases

- **Cloudflare (2019):** A ReDoS in a WAF rule caused a global outage.
- **Stack Overflow (2016):** A ReDoS in a post caused an outage.
- **Atom (2016):** A ReDoS in a syntax highlighter.
- **Various npm packages:** `moment`, `validator`, `path-to-regexp`.

---

## Core Concept 7: The Node.js Permission Model

### Definitions

**Core Definition:** The Node.js permission model is a native runtime security feature that restricts an application's access to the file system, child processes, worker threads, and native addons.

**Technical Definition:** Introduced experimentally in Node.js v20.0.0 and stabilized in v23.0.0, the permission model (enabled with `--permission`) restricts the application's access to: file system read (`--allow-fs-read`), file system write (`--allow-fs-write`), child process creation (`--allow-child-process`), worker thread creation (`--allow-worker`), native addons (`--allow-addons`), and WebAssembly compilation (`--allow-wasm`). When enabled, all capabilities are denied by default and must be explicitly allowed. The model is not a silver bullet (it does not prevent all attacks), but it significantly reduces the blast radius of a compromise.

**Beginner-Friendly Explanation:** Imagine your application is a child in a house. Normally, the child can go anywhere (read files, spawn processes, load addons). The permission model is like putting locks on all the doors — the child starts locked inside their room, and you explicitly unlock only the doors they need. If an attacker compromises the application, they are still locked inside the room.

### Purposes

- To restrict the application's access to the file system and child processes.
- To reduce the blast radius of a compromise.
- To enforce least privilege at the runtime level.
- To complement container and OS-level security.
- To prevent supply chain attacks from accessing sensitive resources.

### Syntax Rules and Structure

#### Permission Model Flags

| Flag | Purpose |
|------|---------|
| `--permission` | Enable the permission model (deny by default) |
| `--allow-fs-read=<path>` | Allow reading from a path |
| `--allow-fs-write=<path>` | Allow writing to a path |
| `--allow-child-process` | Allow spawning child processes |
| `--allow-worker` | Allow creating worker threads |
| `--allow-addons` | Allow loading native addons |
| `--allow-wasm` | Allow compiling WebAssembly |
| `--allow-fs-read=*` | Allow reading from anywhere (not recommended) |
| `--allow-fs-write=*` | Allow writing anywhere (not recommended) |

#### Running with Permissions

```bash
# Deny by default, allow specific paths
node --permission \
  --allow-fs-read=/app/config \
  --allow-fs-read=/app/public \
  --allow-fs-write=/app/uploads \
  --allow-child-process \
  /app/dist/main.js
```

#### Programmatic Permission Check

```typescript
// Check if permission model is enabled
if (process.permission) {
  // Check if a specific permission is granted
  const canRead = process.permission.has('fs.read', '/app/config');
  const canWrite = process.permission.has('fs.write', '/app/uploads');
  const canSpawn = process.permission.has('child');

  console.log({ canRead, canWrite, canSpawn });
} else {
  console.log('Permission model is not enabled');
}
```

#### Syntax Rules

- **Enable with `--permission`** — all capabilities denied by default.
- **Use `--allow-fs-read=<path>`** — allow specific paths, not `*`.
- **Use `--allow-fs-write=<path>`** — allow specific paths, not `*`.
- **Use `--allow-child-process`** — only if the application spawns processes.
- **Use `--allow-worker`** — only if the application uses worker threads.
- **Use `--allow-addons`** — only if the application loads native addons.
- **Check `process.permission`** — verify permissions at runtime.
- **Combine with containers** — the permission model complements Docker/Kubernetes.
- **Do not use `*`** — always specify exact paths.
- **Test thoroughly** — the permission model may break legitimate functionality.

#### Constraints and Limitations

- **Experimental** — the API may change (stable in v23.0.0).
- **Not a silver bullet** — does not prevent all attacks.
- **May break libraries** — some libraries read files or spawn processes internally.
- **`--allow-fs-read=*`** — allows reading anywhere, defeating the purpose.
- **Does not restrict network access** — use firewall rules or containers.
- **Does not restrict environment variables** — secrets in `process.env` are still accessible.
- **Symlink handling** — the permission model resolves symlinks before checking.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Running a Node.js Application with the Permission Model

```json
// package.json
{
  "scripts": {
    "start": "node --permission --allow-fs-read=/app/config --allow-fs-read=/app/public --allow-fs-write=/app/uploads --allow-child-process /app/dist/main.js",
    "start:strict": "node --permission --allow-fs-read=/app/config --allow-fs-read=/app/public --allow-fs-write=/app/uploads /app/dist/main.js"
  }
}
```

```typescript
// app.ts — checking permissions at runtime
import express from 'express';
import { readFile, writeFile } from 'node:fs/promises';

const app = express();

app.get('/config', async (req, res) => {
  if (!process.permission?.has('fs.read', '/app/config')) {
    return res.status(403).json({ error: 'No permission to read config' });
  }

  const config = await readFile('/app/config/app.json', 'utf8');
  res.json(JSON.parse(config));
});

app.post('/upload', async (req, res) => {
  if (!process.permission?.has('fs.write', '/app/uploads')) {
    return res.status(403).json({ error: 'No permission to write uploads' });
  }

  await writeFile('/app/uploads/file.txt', req.body.content);
  res.json({ success: true });
});

app.post('/spawn', (req, res) => {
  if (!process.permission?.has('child')) {
    return res.status(403).json({ error: 'No permission to spawn' });
  }

  // Spawn child process
  const child = spawn('ls', ['/app']);
  child.stdout.pipe(res);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

```typescript
// test-permissions.ts — test that permissions are enforced
import { readFile, writeFile } from 'node:fs/promises';

// This should succeed (allowed)
try {
  await readFile('/app/config/app.json');
  console.log('✅ Can read config');
} catch (err) {
  console.log('❌ Cannot read config:', (err as Error).message);
}

// This should fail (not allowed)
try {
  await readFile('/etc/passwd');
  console.log('❌ SECURITY ISSUE: Can read /etc/passwd');
} catch (err) {
  console.log('✅ Cannot read /etc/passwd:', (err as Error).message);
}

// This should fail (not allowed)
try {
  await writeFile('/app/config/app.json', 'malicious');
  console.log('❌ SECURITY ISSUE: Can write to config');
} catch (err) {
  console.log('✅ Cannot write to config:', (err as Error).message);
}

// This should succeed (allowed)
try {
  await writeFile('/app/uploads/test.txt', 'hello');
  console.log('✅ Can write uploads');
} catch (err) {
  console.log('❌ Cannot write uploads:', (err as Error).message);
}
```

**Expected behaviour:**
- Reading `/app/config/app.json` succeeds (allowed).
- Reading `/etc/passwd` fails with `ERR_ACCESS_DENIED` (not allowed).
- Writing to `/app/config/app.json` fails (not allowed).
- Writing to `/app/uploads/test.txt` succeeds (allowed).

**Why this works:** The permission model denies all access by default. Only explicitly allowed paths are accessible. This limits the blast radius of a compromise.

### Real-World Cases

- **High-security applications:** Banking, healthcare, and government applications.
- **Multi-tenant SaaS:** Isolating tenant code.
- **Plugin systems:** Running third-party code with restricted permissions.
- **CI/CD runners:** Restricting build scripts.
- **Containers:** Complementing Docker/Kubernetes security.

---

## References

- Node.js Documentation — Security Best Practices — https://nodejs.org/en/learn/getting-started/security-best-practices
- Node.js Documentation — Permissions — https://nodejs.org/api/permissions.html
- Node.js Documentation — `child_process` — https://nodejs.org/api/child_process.html
- Node.js Documentation — `process.permission` — https://nodejs.org/api/process.html#processpermission
- npm Documentation — `npm audit` — https://docs.npmjs.com/cli/v10/commands/npm-audit
- npm Documentation — `npm ci` — https://docs.npmjs.com/cli/v10/commands/npm-ci
- npm Documentation — Package Provenance — https://docs.npmjs.com/generating-provenance-statements
- OWASP Cheat Sheet Series — Node.js Security — https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Prototype Pollution Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Prototype_Pollution_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — OS Command Injection Defense — https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Regular Expression Denial of Service — https://cheatsheetseries.owasp.org/cheatsheets/Regular_Expression_Denial_of_Service_-_ReDoS_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Secrets Management — https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- CWE-1321 — Prototype Pollution — https://cwe.mitre.org/data/definitions/1321.html
- CWE-78 — OS Command Injection — https://cwe.mitre.org/data/definitions/78.html
- CWE-1333 — ReDoS — https://cwe.mitre.org/data/definitions/1333.html
- CWE-1104 — Use of Unmaintained Third Party Components — https://cwe.mitre.org/data/definitions/1104.html
- Socket — Supply Chain Security — https://socket.dev/
- Snyk — Node.js Security — https://snyk.io/learn/node-js-security/
- TruffleHog — Secret Scanning — https://github.com/trufflesecurity/trufflehog
- Gitleaks — Secret Scanning — https://github.com/gitleaks/gitleaks
- safe-regex — npm package — https://www.npmjs.com/package/safe-regex
- RE2 — npm package — https://www.npmjs.com/package/re2
- Zod — npm package — https://www.npmjs.com/package/zod
- Pino — Redacting Logger — https://getpino.io/#/docs/redaction
- Node.js — `--permission` flag — https://nodejs.org/api/cli.html#--permission
- Node.js — Security Releases — https://nodejs.org/en/blog/vulnerability
- npm Advisory Database — https://github.com/advisories
- GitHub Advisory Database — https://github.com/advisories
- SLSA — Supply Chain Levels for Software Artifacts — https://slsa.dev/
- Sigstore — Software Signing — https://www.sigstore.dev/