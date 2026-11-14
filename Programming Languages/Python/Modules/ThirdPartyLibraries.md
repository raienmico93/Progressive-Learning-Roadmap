# Python Third-Party Libraries: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Third-party libraries are reusable Python packages created and maintained by the broader community (not the Python core team) that extend the language's capabilities—from web frameworks and data-science toolkits to testing utilities and CLI builders. They are distributed primarily through the **Python Package Index (PyPI)** and installed into isolated environments using tools like `pip`.

**Technical Definition**

The Python packaging ecosystem is built on **PEP 517** (build-system interface), **PEP 518** (`pyproject.toml`), **PEP 621** (project metadata), and **PEP 508** (dependency specification). A third-party library is a distribution package that declares its metadata and dependencies in `pyproject.toml` (or legacy `setup.py`), is uploaded to PyPI or a private index, and is resolved and installed by a package installer. As of mid-2026, PyPI hosts over **604,000 projects** . The ecosystem is target-rich: in 2024 alone, researchers tracked over **7,000 malicious packages** on PyPI, many using typosquatting names like `reqeusts` to catch a single misplaced keystroke . Security, reproducibility, and deliberate evaluation are therefore first-class concerns.

**Beginner-Friendly Explanation**

Python's standard library is powerful, but it cannot cover everything. When you need to build a web API, process scientific data, or test your code, you almost always use a **third-party library**—a package written by someone else that you download and install. The main marketplace for these packages is **PyPI**, and the main tool for installing them is `pip`. The convenience is enormous, but so is the risk: you are running code written by strangers, so you need to evaluate, pin, and scan what you install.

### Key Characteristics

- **Huge ecosystem**: Over 600,000 projects on PyPI, ranging from production-grade frameworks to abandoned experiments.
- **Dependency trees**: Installing one package often pulls in dozens of transitive dependencies, each of which is another trust decision .
- **Reproducibility challenges**: Without lock files, two installs of the same `requirements.txt` can produce different environments.
- **Active attack surface**: Typosquatting, dependency confusion, and compromised maintainer accounts are documented, recurring attack vectors .
- **Tooling diversity**: `pip`, `uv`, `Poetry`, `pipx`, and `conda` each address different parts of the workflow.
- **Standards convergence**: `pyproject.toml` is now the single source of truth for project metadata and dependencies .

### Prerequisites

- Basic Python syntax and familiarity with the command line.
- Understanding of virtual environments and why they matter.
- Awareness of version numbers and semantic versioning concepts.
- A basic security mindset: distrust by default, verify before install.

### Related Programming Areas

- **Dependency management**: Lock files, version pinning, and resolution algorithms.
- **Supply-chain security**: SBOMs, vulnerability scanning, and provenance attestation.
- **DevOps and CI/CD**: Reproducible builds, automated auditing, and Trusted Publishing.
- **Packaging and distribution**: Building and publishing your own packages.

---

## Core Concepts / Features

---

### 1. PyPI (Python Package Index)

#### Definitions

**Core Definition**

PyPI is the official, community-maintained repository where Python packages are published and discovered; it is the primary source for `pip install` .

**Technical Definition**

PyPI is a **PEP 503**-compliant simple index that exposes package metadata via a JSON API and distributes artifacts as source distributions (sdists) and built distributions (wheels). The index does not guarantee the quality or security of hosted code; assessing trustworthiness is the consumer's responsibility . PyPI supports **Trusted Publishing** via OpenID Connect, allowing CI systems to publish with short-lived credentials instead of long-lived API tokens .

**Beginner-Friendly Explanation**

PyPI is the “app store” for Python packages. When you type `pip install requests`, pip goes to PyPI, finds the `requests` package, downloads it, and installs it. It is the first place to look for any third-party library.

#### Purposes

- To provide a central, standardized index for discovering and distributing Python packages.
- To host package metadata, documentation links, and release history.
- To serve as the authoritative source that `pip` queries by default.
- To enable automated publishing through Trusted Publishers and OIDC.
- To offer a JSON API for programmatic package discovery and evaluation .

#### Syntax Rules and Structure

**Direct Web Discovery**

```
https://pypi.org/                     # Main site
https://pypi.org/search/?q=<term>     # Web search
https://pypi.org/project/<name>/      # Project page
```

**JSON API (for tooling)**

```
https://pypi.org/pypi/<package>/json           # Latest release metadata
https://pypi.org/pypi/<package>/<version>/json # Specific version metadata
```

**Component Breakdown**

| Endpoint | Returns |
|----------|---------|
| `/pypi/<package>/json` | Full metadata including releases, dependencies, and project URLs |
| `/simple/<package>/` | PEP 503 simple index HTML listing available files |

**Syntax Rules**

- Package names are case-insensitive and normalized (e.g., `Flask` → `flask`).
- The JSON API requires no authentication for public packages.
- Project pages expose “Last released,” “Requires Python,” wheel tags, and download statistics.

**Constraints and Limitations**

- `pip search` is **disabled** (since 2020); use the web search or third-party tools instead .
- PyPI does not audit or guarantee package safety; it is a distribution service, not a security service .
- Download counts are a popularity signal, not a quality or security signal .

#### Annotated Code Examples

**Example 1: Fetching PyPI Metadata via the JSON API**

```python
import json
import urllib.request

url = "https://pypi.org/pypi/requests/json"
with urllib.request.urlopen(url) as response:
    data = json.load(response)

info = data["info"]
print("Name:", info["name"])
print("Version:", info["version"])
print("Requires Python:", info["requires_python"])
print("Homepage:", info.get("home_page"))
```

**Expected Output:**

```
Name: requests
Version: 2.32.3
Requires Python: >=3.8
Homepage: https://requests.readthedocs.io
```

**Step-by-Step Breakdown:**

1. The JSON API is queried for the `requests` package.
2. The `info` dictionary contains the latest release metadata.
3. `requires_python` and project URLs are extracted for evaluation.

**Why the Output is Produced:**

PyPI's JSON API returns structured metadata about the package's current release, enabling programmatic inspection.

#### Real-World Cases

- **Automated dependency auditing**: Tools like `pip-audit` query PyPI metadata and advisory databases to flag vulnerable versions.
- **Package discovery scripts**: Custom tooling fetches project metadata to rank candidates.
- **CI integration**: Pipelines query PyPI to detect available updates.

---

### 2. Package Discovery

#### Definitions

**Core Definition**

Package discovery is the process of finding candidate third-party libraries that solve a given problem, using PyPI search, curated lists, and specialized discovery tools.

**Technical Definition**

Discovery combines **keyword search** on PyPI, **topic classifiers** (e.g., `Topic :: Internet :: WWW/HTTP`), **community signals** (GitHub stars, download trends), and **specialized search tools** that rank packages by relevance, maintenance health, and adoption. The `pip search` command is deprecated, so modern discovery relies on the PyPI web interface, JSON APIs, and third-party CLIs such as `pipscout`, `pse`, and `pip-search-ex` .

**Beginner-Friendly Explanation**

Discovery is how you find the right library. Instead of guessing names, you search PyPI for keywords related to your task, look at the results' release dates and download counts, and check whether they are actively maintained. Tools like `pipscout` can even suggest the “best pick” based on download data and release freshness .

#### Purposes

- To identify candidate libraries that match a functional need.
- To compare candidates on popularity, maintenance, and compatibility.
- To avoid abandoned, insecure, or obsolete packages.
- To discover alternatives that may be better maintained or more secure.
- To reduce the risk of choosing a package that will become a liability.

#### Syntax Rules and Structure

**PyPI Web Search**

```
https://pypi.org/search/?q=<keywords>
```

**Topic-based browsing**

```
https://pypi.org/search/?q=&classifiers=Topic+%3A%3A+...
```

**Third-party discovery tools**

```
pip install pipscout
pipscout <task description> --deep
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| PyPI web search | Free-text search over names and summaries |
| Topic classifiers | Structured categories such as “Database” or “Scientific/Engineering” |
| `pipscout` | Ranks packages by downloads and freshness for a described task |
| `pse` | Cross-index search across PyPI and Conda channels |

**Syntax Rules**

- Use specific, functional keywords (e.g., `pdf generation`, not `Python library`) .
- Prefer topic-classifier browsing over free-text search for precision .
- Check “Last released” — packages older than 12 months deserve scrutiny .
- Verify wheel availability for your platform and Python version .

**Constraints and Limitations**

- PyPI search is not semantic; it matches words, not meaning .
- Download counts measure popularity, not quality or security .
- Third-party discovery tools may not be maintained; verify their own credibility.

#### Annotated Code Examples

**Example 1: Using `pipscout` for Discovery**

```bash
pip install pipscout
pipscout "HTTP client with connection pooling"
```

**Expected Output (illustrative):**

```
Top pick: httpx (monthly downloads: ..., last release: ...)
Runners-up: requests, aiohttp
Install: pip install httpx
```

**Step-by-Step Breakdown:**

1. `pipscout` searches PyPI for packages matching the task description.
2. It ranks candidates by download volume and release recency.
3. It prints the best pick, runners-up, and the exact `pip install` line.

**Why the Output is Produced:**

`pipscout` aggregates PyPI download statistics and release metadata to produce a ranked shortlist, reducing manual comparison effort .

#### Real-World Cases

- **Starting a new project**: Finding the current best HTTP client, CLI framework, or testing library.
- **Replacing deprecated dependencies**: Discovering actively maintained alternatives.
- **Technology evaluation**: Comparing `httpx` vs. `requests` vs. `aiohttp` for a specific use case.

---

### 3. Package Installation

#### Definitions

**Core Definition**

Package installation is the process of downloading a package and its dependencies from an index and making them available to your Python environment, typically using `pip`.

**Technical Definition**

`pip install <specifier>` resolves the package's dependencies according to **PEP 508** version specifiers, downloads the appropriate artifacts (preferring wheels over sdists), and installs them into the active environment. `pip` supports **hash-checking mode** (`--require-hashes`), **constraints files** (`-c`), and **index configuration**. `pipx` is the recommended tool for installing command-line applications in isolated environments . `uv` is a modern, faster alternative that generates hash-pinned lock files .

**Beginner-Friendly Explanation**

Installing a package means telling `pip` to fetch it from PyPI and put it in your environment. You can install a single package (`pip install requests`), a specific version (`pip install requests==2.32.3`), or a whole list from a `requirements.txt` file. Always install into a virtual environment, never globally .

#### Purposes

- To add third-party functionality to a Python project.
- To install specific versions or version ranges for compatibility.
- To install from a curated, hash-pinned lock file for reproducibility.
- To install CLI applications in isolated environments with `pipx`.
- To configure private or alternative indexes for internal packages.

#### Syntax Rules and Structure

**Basic Installation**

```bash
pip install <package>
pip install <package>==<version>
pip install "<package>>=<min>,<<max>"
```

**From a requirements file**

```bash
pip install -r requirements.txt
```

**Hash-checking mode**

```bash
pip install --require-hashes -r requirements.txt
```

**Constraints file**

```bash
pip install -c constraints.txt -r requirements.txt
```

**Component Breakdown**

| Specifier | Meaning |
|-----------|---------|
| `package` | Latest compatible version |
| `package==1.2.3` | Exact version |
| `package>=1.2,<2.0` | Version range |
| `package~=1.2.3` | Compatible release (`>=1.2.3, ==1.2.*`) |
| `-r file` | Read requirements from file |
| `-c file` | Apply version constraints without installing |
| `--require-hashes` | Verify cryptographic hashes of all artifacts |

**Syntax Rules**

- Always install into a virtual environment (`venv`, `virtualenv`, `uv venv`) .
- Use `pipx install <app>` for standalone CLI applications .
- Use `--require-hashes` only with a fully pinned, hashed requirements file .
- Do not use `python setup.py install`; it is deprecated .

**Constraints and Limitations**

- Installing without pinning can silently pull in newer, incompatible, or malicious versions .
- `pip` does not automatically verify hashes unless `--require-hashes` is used.
- Global installs can break system tools and create conflicts.

#### Annotated Code Examples

**Example 1: Reproducible Installation with Hashes**

```bash
# Generate a fully pinned, hashed requirements file
pip-compile --generate-hashes requirements.in -o requirements.txt

# Install with hash verification
pip install --require-hashes -r requirements.txt
```

**Expected Output:**

```
(All packages installed after hash verification succeeds.)
```

**Step-by-Step Breakdown:**

1. `pip-compile` resolves the full dependency tree and records exact versions with SHA-256 hashes.
2. `pip install --require-hashes` refuses to install any artifact whose hash does not match.

**Why the Output is Produced:**

Hash-checking closes the tampering vector: even if an attacker replaces a package file on the index, the hash mismatch prevents installation .

#### Real-World Cases

- **CI pipelines**: Installing from a lock file with `--require-hashes` for deterministic builds.
- **Developer onboarding**: A single `pip install -r requirements.txt` sets up the project.
- **CLI tools**: `pipx install ruff` installs Ruff in its own isolated environment.

---

### 4. Package Versions

#### Definitions

**Core Definition**

Package versions are identifiers that distinguish releases of a package and allow installers to select compatible versions according to declared constraints.

**Technical Definition**

Python versions follow **PEP 440** (version identification and dependency specification). Version specifiers include `==` (exact), `>=`, `<=`, `>`, `<`, `~=` (compatible release), and `!=` (exclusion). **Libraries** should declare ranges with sensible lower and upper bounds; **applications** should pin exact versions in a lock file . The `~=` operator, for example, `~=1.4.2` is equivalent to `>=1.4.2, ==1.4.*`.

**Beginner-Friendly Explanation**

Versions are how packages say “this is release 2.1.3.” When you install, you can ask for a specific version (`==2.1.3`), a minimum version (`>=2.0`), or a compatible range (`~=2.1`). For applications, you want to **pin exact versions** so every machine gets the same code. For libraries, you want **ranges** so your users are not forced into an old version .

#### Purposes

- To ensure compatibility between packages and Python versions.
- To reproduce environments exactly across machines and CI.
- To allow libraries to express compatible version ranges without over-constraining.
- To enable safe upgrades through deliberate version-range updates.
- To prevent silent breakage from unexpectedly new releases.

#### Syntax Rules and Structure

**Version Specifiers**

```
==1.2.3          # exact
>=1.2, <2.0      # range
~=1.2.3          # compatible release (>=1.2.3, ==1.2.*)
!=1.3.0          # exclusion
```

**Declaring in `pyproject.toml` (library)**

```toml
dependencies = [
    "requests>=2.28,<3.0",
    "click~=8.1",
]
```

**Pinning in a lock file (application)**

```
requests==2.32.3
click==8.1.7
```

**Component Breakdown**

| Context | Recommended Strategy |
|---------|---------------------|
| Library | Ranges with lower and upper bounds |
| Application | Exact pinning in a lock file |
| CLI tool | `pipx` isolation handles versions automatically |

**Syntax Rules**

- Libraries should not pin exact versions in `install_requires` .
- Applications should commit a lock file with exact `==` versions .
- Use `~=` for compatible releases when you want patch updates but not minor breaking changes .
- Never rely on an unpinned `>=` alone; it invites supply-chain drift .

**Constraints and Limitations**

- Pinning exact versions in libraries is discouraged because it can cause dependency conflicts for downstream users .
- Version ranges can still allow malicious or broken releases within the range; use a release cooldown for defense .

#### Annotated Code Examples

**Example 1: Library vs. Application Versioning**

```toml
# pyproject.toml (library — use ranges)
[project]
dependencies = [
    "httpx>=0.27,<0.29",
    "pydantic~=2.5",
]
```

```
# requirements.txt (application — pin exactly)
httpx==0.27.2
pydantic==2.5.3
```

**Expected Output:**

```
(Library declares compatible ranges; application pins exact versions.)
```

**Step-by-Step Breakdown:**

1. The library uses ranges so downstream projects can combine it with other dependencies.
2. The application locks exact versions for reproducibility.

**Why the Output is Produced:**

This separation follows the consensus: libraries express flexibility, applications enforce determinism .

#### Real-World Cases

- **Open-source libraries**: Declaring `requests>=2.28,<3.0` so users are not forced into an old release.
- **Production deployments**: Locking exact versions to guarantee identical behavior across environments.
- **Security patching**: Updating the lower bound to exclude vulnerable versions.

---

### 5. Package Dependencies

#### Definitions

**Core Definition**

Package dependencies are other packages that a library requires to function; they are resolved transitively and installed automatically by the package installer.

**Technical Definition**

Dependencies are declared in `pyproject.toml` (PEP 621) or `setup.py` and are resolved by `pip` according to PEP 508 specifiers. The resolver computes a consistent set of versions satisfying all constraints. **Direct dependencies** are those you declare; **transitive dependencies** are those pulled in by your dependencies. A single `pip install flask` pulls in **seven packages** , and each is an implicit trust decision. Dependency confusion attacks exploit this by publishing public packages with the same names as internal private packages .

**Beginner-Friendly Explanation**

When you install one package, it often needs other packages to work. Those are its dependencies, and they get installed automatically. The problem is that you did not choose them—they are strangers your stranger brought along. You need to know what they are, pin them, and scan them just like your direct dependencies.

#### Purposes

- To enable packages to build on shared functionality without duplicating code.
- To express compatibility constraints between packages.
- To allow the resolver to compute a consistent environment.
- To support optional extras (e.g., `requests[security]`) for feature-specific dependencies.
- To enable reproducible dependency trees via lock files.

#### Syntax Rules and Structure

**Declaring dependencies in `pyproject.toml`**

```toml
[project]
dependencies = [
    "requests>=2.28,<3.0",
    "click~=8.1",
]

[project.optional-dependencies]
security = ["cryptography>=41"]
dev = ["pytest>=7", "ruff>=0.5"]
```

**Inspecting the resolved tree**

```bash
pip freeze
pipdeptree
```

**Component Breakdown**

| Concept | Description |
|---------|-------------|
| Direct dependency | Declared by you |
| Transitive dependency | Pulled in by your dependencies |
| Extras | Optional dependency groups (`package[extra]`) |
| Constraints file | Controls versions without forcing installation |

**Syntax Rules**

- Use `pyproject.toml` as the single source of truth for dependencies .
- Declare ranges with sensible bounds for libraries; lock exact versions for applications .
- Use `pipdeptree` or `pip freeze` to audit the full transitive tree .
- Use constraints files to override transitive versions without adding new direct dependencies .

**Constraints and Limitations**

- Deep dependency trees exponentially increase the attack surface.
- Conflicting constraints can make resolution fail; tools like `depwhy` help diagnose conflicts .
- Transitive dependencies can introduce vulnerabilities you never chose directly.

#### Annotated Code Examples

**Example 1: Auditing the Dependency Tree**

```bash
pip install pipdeptree
pipdeptree
```

**Expected Output (illustrative):**

```
flask==3.0.0
├── click [required: >=8.1.3, installed: 8.1.7]
├── itsdangerous [required: >=2.1.2, installed: 2.2.0]
├── jinja2 [required: >=3.1.2, installed: 3.1.4]
├── markupsafe [required: >=2.1.1, installed: 2.1.5]
├── werkzeug [required: >=3.0.0, installed: 3.1.3]
...
```

**Step-by-Step Breakdown:**

1. `pipdeptree` walks the installed environment and prints the full dependency graph.
2. Each top-level package is listed with its requirements and installed versions.

**Why the Output is Produced:**

The tree reveals every transitive dependency, enabling you to audit what you are implicitly trusting .

#### Real-World Cases

- **Supply-chain auditing**: Reviewing transitive dependencies for known vulnerabilities.
- **License compliance**: Ensuring all transitive dependencies have compatible licenses.
- **Conflict resolution**: Using `pip check` and `depwhy` to diagnose resolution failures.

---

### 6. Package Security

#### Definitions

**Core Definition**

Package security is the practice of defending against malicious, compromised, or vulnerable third-party packages through pinning, hash verification, vulnerability scanning, provenance verification, and supply-chain hardening.

**Technical Definition**

Python supply-chain security encompasses: **dependency pinning** with hashes (`pip install --require-hashes`), **vulnerability scanning** (`pip-audit`, `safety`), **lock file generation** (`pip-tools`, `uv`, `Poetry`), **Trusted Publishing** via OIDC, **SBOM generation** (`CycloneDX`), **release cooldowns** (`--exclude-newer`), and **provenance verification** (`trustcheck`, `sigstore`). The threat model includes typosquatting, dependency confusion, account takeover, and malicious new releases . PyPI itself explicitly states that assessing the trustworthiness of a package's source code is the consumer's responsibility .

**Beginner-Friendly Explanation**

Package security means not blindly trusting strangers. You pin exact versions so an attacker cannot swap in a malicious new release. You add hashes so tampering is detected. You scan for known vulnerabilities before deploying. You use short-lived publishing credentials so a stolen token cannot be used to poison your packages. And you wait a few days before installing brand-new releases, giving the community time to spot malware.

#### Purposes

- To prevent malicious packages from executing code in your environment.
- To detect known vulnerabilities in your dependency tree before deployment.
- To guarantee that installed artifacts have not been tampered with.
- To eliminate long-lived publishing tokens that attackers can steal.
- To enable rapid response when a supply-chain incident is disclosed.

#### Syntax Rules and Structure

**Hash-pinned lock file**

```bash
pip-compile --generate-hashes requirements.in
pip install --require-hashes -r requirements.txt
```

**Vulnerability scanning**

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

**Release cooldown**

```bash
pip install --exclude-newer "2026-11-01" -r requirements.txt
```

**Trusted Publishing (for maintainers)**

```
# Configure OIDC in GitHub Actions; no long-lived PyPI token needed.
```

**Component Breakdown**

| Defense | Tool / Mechanism | Threat Mitigated |
|---------|------------------|------------------|
| Exact pinning | `==` in lock file | Silent version drift |
| Hash verification | `--require-hashes` | Artifact tampering |
| Vulnerability scanning | `pip-audit` | Known CVEs |
| Release cooldown | `--exclude-newer` | Malicious new releases |
| Trusted Publishing | OIDC | Stolen API tokens |
| SBOM | CycloneDX | Incident response |

**Syntax Rules**

- Always use `--require-hashes` with a fully pinned lock file .
- Run `pip-audit` in CI on every pull request, not just nightly .
- Use `--exclude-newer` to skip very recent releases .
- Prefer Trusted Publishing (OIDC) over long-lived API tokens when publishing .

**Constraints and Limitations**

- Hash-checking requires **all** dependencies (including transitive) to be pinned with `==` and hashed .
- `pip-audit` only detects known vulnerabilities; it cannot detect backdoors in legitimate packages.
- Release cooldowns add latency to security updates; balance against the need for urgent patches.

#### Annotated Code Examples

**Example 1: CI Security Pipeline**

```bash
# In CI:
pip install pip-audit
pip-audit -r requirements.txt
# Fail the build if any known vulnerability is found.
```

**Expected Output:**

```
No known vulnerabilities found.
```

**Step-by-Step Breakdown:**

1. `pip-audit` reads the pinned requirements and cross-references the OSV and PyPI Advisory databases.
2. If any package matches a known vulnerability, the command exits non-zero, failing the build.

**Why the Output is Produced:**

`pip-audit` uses the Python Packaging Advisory Database as its data source to identify vulnerable versions .

#### Real-World Cases

- **LiteLLM/Telnyx attacks**: Supply-chain incidents that exploited stolen PyPI tokens and unpinned dependencies .
- **Typosquatting campaigns**: Thousands of malicious packages with names like `reqeusts` detected in 2024 .
- **Dependency confusion**: Internal package names published publicly to hijack CI installs .

---

### 7. Package Evaluation

#### Definitions

**Core Definition**

Package evaluation is the process of assessing a package's trustworthiness, maintenance health, security posture, and suitability before adopting it as a dependency.

**Technical Definition**

Evaluation combines **metadata signals** (release recency, download trends, verified project links), **security signals** (`SECURITY.md`, vulnerability response history, advisory presence), **dependency depth** (fewer dependencies is generally safer), and **source-code inspection** (repository activity, CI status, test coverage). Automated tools such as `trustcheck`, `PyPI Package Inspector`, and `pyroma` score packages on these dimensions. Manual review is warranted when a package is new (<90 days old), recently changed maintainers, has a thin repository history, or ships compiled binaries .

**Beginner-Friendly Explanation**

Evaluation is asking “should I trust this package?” before you install it. You look at when it was last updated, how many people use it, whether it has a security policy, how many dependencies it drags in, and whether the repository looks active. If something feels off—a brand-new package, a mysterious maintainer change, or a package with dozens of tiny dependencies—you dig deeper or choose an alternative.

#### Purposes

- To avoid adopting abandoned, insecure, or low-quality packages.
- To distinguish popularity from quality and security.
- To assess the trustworthiness of transitive dependencies.
- To prioritize review effort on the highest-risk candidates.
- To make informed decisions between competing alternatives.

#### Syntax Rules and Structure

**Manual evaluation checklist**

```
[ ] Last released < 12 months ago
[ ] Requires Python compatible with your version
[ ] Wheels available for your platform
[ ] SECURITY.md present in repository
[ ] Repository active (commits, merged PRs, CI status)
[ ] Dependencies are few and reputable
[ ] No known critical vulnerabilities
[ ] Maintainer account history is stable
```

**Automated evaluation**

```bash
pip install trustcheck
trustcheck scan <package>
```

**Component Breakdown**

| Signal | What It Indicates |
|--------|-------------------|
| Last released | Maintenance activity |
| Download trends | Adoption (not quality) |
| `SECURITY.md` | Formal vulnerability process |
| Dependency count | Attack surface size |
| Verified project links | Metadata authenticity |
| Advisory history | Known vulnerability pattern |

**Syntax Rules**

- Treat download counts as a **popularity** signal, not a security guarantee .
- Escalate review when a package is **less than 90 days old** .
- Investigate if the maintainer account changed within the last **30 days** .
- Be cautious of sdists containing compiled binaries you cannot reproduce .

**Constraints and Limitations**

- No single metric proves a package is safe.
- Automated scores (e.g., from `trustcheck`) are heuristics, not malware verdicts.
- Small, niche packages may have low downloads but be perfectly sound.

#### Annotated Code Examples

**Example 1: Automated Trust Evaluation with `trustcheck`**

```bash
pip install trustcheck
trustcheck scan requests
```

**Expected Output (illustrative):**

```
Package: requests
Version: 2.32.3
Provenance: verified
Vulnerabilities: none known
Repository: matches declared URL
Risk: low
```

**Step-by-Step Breakdown:**

1. `trustcheck` fetches PyPI metadata, provenance attestations, and vulnerability records.
2. It compares the declared repository against the attested source.
3. It emits a consolidated risk report.

**Why the Output is Produced:**

`trustcheck` combines multiple trust signals—provenance, repository matching, and advisory data—into a single operator-friendly report .

#### Real-World Cases

- **Security review gates**: Approving a new dependency only after evaluating its maintenance and security signals.
- **Incident response**: Using SBOMs and dependency trees to determine whether a newly disclosed vulnerability affects your project.
- **Replacing abandoned packages**: Discovering that a package has not been released in 18 months and lacks a security policy, prompting a search for alternatives.

---

### 8. Package Best Practices

#### Definitions

**Core Definition**

Package best practices are the repeatable habits that keep third-party dependency usage reproducible, secure, and maintainable—isolation, pinning, auditing, and deliberate adoption.

**Technical Definition**

Best practices include: (1) use **virtual environments** for every project ; (2) declare dependencies in **`pyproject.toml`** as the single source of truth ; (3) generate and commit a **lock file** with exact versions ; (4) **pin with hashes** and install with `--require-hashes` ; (5) run **`pip-audit` in CI** on every pull request ; (6) use **Trusted Publishing** for any package you publish ; (7) apply a **release cooldown** to avoid brand-new malicious releases ; and (8) keep the dependency set **as small as possible** .

**Beginner-Friendly Explanation**

Best practices are the habits that keep you safe. Every project gets its own virtual environment. Every dependency is declared and locked. Every install is hash-checked. Every pull request is scanned for vulnerabilities. And you do not install a package the moment it is released—you wait a few days for the community to spot problems.

#### Purposes

- To guarantee reproducible installs across machines, CI, and production.
- To minimize the attack surface from transitive dependencies.
- To detect vulnerabilities before they reach production.
- To eliminate long-lived credentials that attackers can steal.
- To keep the dependency set lean and maintainable.

#### Syntax Rules and Structure

**Project isolation**

```bash
python -m venv .venv
source .venv/bin/activate   # Linux/macOS
.venv\Scripts\activate      # Windows
```

**Declare and lock**

```bash
# Declare in pyproject.toml, then:
pip-compile --generate-hashes pyproject.toml -o requirements.txt
```

**Audit in CI**

```bash
pip-audit -r requirements.txt
```

**Component Breakdown**

| Practice | Tool / Command |
|----------|----------------|
| Isolation | `venv`, `virtualenv`, `uv venv` |
| Declaration | `pyproject.toml` |
| Locking | `pip-compile`, `poetry lock`, `uv lock` |
| Hash verification | `--require-hashes` |
| Auditing | `pip-audit`, `safety` |
| Cooldown | `--exclude-newer` |
| Trusted Publishing | OIDC in GitHub Actions / GitLab CI |

**Syntax Rules**

- Never install packages globally; always use an isolated environment .
- Commit lock files to version control and validate them in CI .
- Install from the lock file in CI; fail if the lock file is out of date .
- Keep direct dependencies minimal; remove unused packages regularly .

**Constraints and Limitations**

- Lock files can become stale; schedule periodic regeneration and review the diff .
- Cooldowns delay legitimate security patches; override selectively for critical fixes.
- Hash-checking requires discipline: every dependency must be pinned and hashed.

#### Annotated Code Examples

**Example 1: Complete Project Setup**

```bash
# 1. Create and activate an isolated environment
python -m venv .venv
source .venv/bin/activate

# 2. Declare dependencies in pyproject.toml, then generate a hashed lock file
pip install pip-tools
pip-compile --generate-hashes pyproject.toml -o requirements.txt

# 3. Install from the lock file with hash verification
pip install --require-hashes -r requirements.txt

# 4. Audit for known vulnerabilities
pip-audit -r requirements.txt
```

**Expected Output:**

```
(Environment created, dependencies locked and verified, audit reports no known vulnerabilities.)
```

**Step-by-Step Breakdown:**

1. Isolation prevents conflicts with other projects.
2. `pip-compile` resolves the full tree and records exact versions with hashes.
3. `--require-hashes` guarantees artifact integrity.
4. `pip-audit` checks the resolved tree against vulnerability databases.

**Why the Output is Produced:**

Each step addresses a distinct failure mode: drift, tampering, and known vulnerabilities .

#### Real-World Cases

- **Production services**: Locked, audited dependencies deployed through CI with hash verification.
- **Open-source contributions**: `pyproject.toml` as the single source of truth for metadata and dependencies.
- **Security-conscious teams**: Release cooldowns and Trusted Publishing as default policy .

---

## References

- Python Packaging Authority – Tool recommendations - https://packaging.python.org/en/latest/guides/tool-recommendations/
- Python Packaging Authority – install_requires vs requirements files - https://packaging.python.org/en/latest/discussions/install-requires-vs-requirements/
- Real Python – Dependency management (Python Best Practices) - https://realpython.com/ref/best-practices/dependency-management/
- Real Python – Third-party libraries (Python Best Practices) - https://realpython.com/ref/best-practices/third-party-libraries/
- Safeguard.sh – The Ultimate Guide to Creating a Secure Python Package - https://safeguard.sh/resources/blog/the-ultimate-guide-to-creating-a-secure-python-package
- Safeguard.sh – Securing Python Packages on PyPI (2026 Guide) - https://safeguard.sh/resources/blog/securing-python-packages-on-pypi
- PyBeach 2026 – Securing Your Python Supply Chain: From Dependencies to Deployment - https://pretalx.com/pybeach2026/talk/9BUFWV/
- PyPI Docs – Assessing Source Code Security - https://docs.pypi.org/using-pypi/package-security/source-code-security/
- PyPI – trustcheck (package trust evaluation CLI) - https://pypi.org/project/trustcheck/
- PyPI – pipscout (package discovery) - https://pypi.org/project/pipscout/
- PyPI – pse (Python Search Engine) - https://pypi.org/project/pse/
- PyPI – pip-search-ex (replacement for pip search) - https://pypi.org/project/pip-search-ex/
- pip Docs – Repeatable Installs - https://pip.pypa.io/en/stable/topics/repeatable-installs/
- pip Docs – Secure Installs - https://pip.pypa.io/en/stable/topics/secure-installs/
- PyPI – PyPI Package Inspector - https://github.com/.../pypi-package-inspector
- PyPI – pyroma (package rating tool) - https://pypi.org/project/pyroma/
- PHP.cn – Python第三方库去哪找_PyPI官网搜索与包质量评估技巧 - https://www.php.cn/faq/2260779.html
- GitHub – supply-chain-hardening (pip, uv, poetry, conda hardening) - https://github.com/jlevy/supply-chain-hardening
- GitHub – Python Best Practices: Dependency management skill - https://github.com/.../dependency-management
- GitHub – Dependency conflict resolution skill - https://github.com/.../dependency-conflict-resolution