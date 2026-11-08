# ASP.NET Core Dependency Security: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **.NET 6–10** and the **NuGet dependency security** ecosystem. It covers the built‑in NuGet audit (introduced in .NET 8, extended to transitive packages in .NET 9/10), the `dotnet list package --vulnerable` CLI, GitHub Dependabot for automated updates, Snyk for software composition analysis (SCA), and the supply‑chain hardening practices (lock files, package source mapping, signing verification, Central Package Management) that defend against dependency confusion, typosquatting, and malicious package injection. A critical framing note: **a clean `dotnet restore` is not proof of a clean supply chain** — restore output, audit sources, and transitive resolution must all be verified independently . Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dependency security is the practice of controlling, auditing, and hardening the third‑party packages an application depends on — direct and transitive — so that known vulnerabilities are remediated, malicious packages are blocked, and the resolved dependency graph is exactly what the team intended. |
| **Technical Definition** | In .NET, dependency security spans **vulnerability detection** (NuGet Audit during restore, `dotnet list package --vulnerable`, Snyk SCA), **automated remediation** (Dependabot, Renovate), **resolution integrity** (lock files with `--locked-mode`, package source mapping to prevent dependency confusion), **artifact trust** (NuGet package signing and `signatureValidationMode`), and **version governance** (Central Package Management, TFM support lifecycle). The threat model includes **dependency confusion** (a public package shadowing an internal name with a higher version), **typosquatting** (`Newtonsoft.Jsoon`), **maintainer takeover**, and **build‑time execution** via MSBuild targets . |
| **Beginner‑Friendly Explanation** | Dependency security is the **“know exactly what you are shipping”** discipline. Your application is not just the code you write — it is every NuGet package you pull, every package those packages pull, and the feeds and build machines in between . If one of those packages has a known vulnerability, or if an attacker slips a malicious package into your build, your application is compromised. Dependency security is about **detecting** known‑vulnerable packages, **pinning** what gets resolved, **automating** updates, and **verifying** that nothing changed silently. |

### Key Characteristics

- **Transitive‑aware** — the dependency tree is typically 2–3× the size of the direct set; transitive packages are the majority of the risk .
- **Resolution‑dependent** — manifests (`.csproj`, `packages.config`) declare *intent*; the actual resolved graph lives in `obj/project.assets.json` after restore .
- **Automation‑driven** — Dependabot and Snyk run on every push/PR to detect and remediate.
- **Defense‑in‑depth** — audit, lock files, source mapping, signing, and version governance are complementary controls, not alternatives.
- **CI‑gate‑ready** — NuGet Audit warnings can be elevated to build errors; `--locked-mode` fails on drift; `dotnet list package --vulnerable` can be parsed by CI.

### Prerequisites

- .NET SDK (6.0+ recommended; .NET 8+ for NuGet Audit, .NET 10 for transitive audit by default).
- A NuGet‑based project (`.csproj` with `PackageReference`, or legacy `packages.config`).
- A source‑control repository on GitHub (for Dependabot) or a Snyk account.
- Familiarity with **NuGet feeds**, **`.csproj`**, **`nuget.config`**, and **CI/CD pipelines**.
- Basic understanding of **CVEs**, **GitHub Advisory Database**, and **software composition analysis**.

### Related Programming Areas

- **Supply chain security** — the broader discipline of securing build pipelines and artifacts.
- **CI/CD** — automated audit, update, and gate enforcement.
- **Package management** — NuGet feeds, source mapping, lock files, and signing.
- **Vulnerability management** — CVE tracking, remediation SLAs, and exception handling.
- **Build reproducibility** — deterministic restores via lock files and locked mode.

### Core Concepts / Features

1. **NuGet Package Security** — the threat model, audit mechanisms, and hardening controls.
2. **`dotnet list package --vulnerable`** — the built‑in CLI vulnerability listing.
3. **Dependabot** — automated dependency update PRs for the `nuget` ecosystem.
4. **Snyk** — SCA scanning that resolves the real dependency graph via `dotnet restore`.
5. **Dependency Security Best Practices** — the review checklist that ties audit, pinning, and automation together.

---

## 1. NuGet Package Security (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | NuGet package security is the practice of ensuring that the packages restored into a .NET build are authentic, free of known vulnerabilities, and resolved exactly as intended — from trusted feeds, with signed artifacts, and with a reproducible dependency graph. |
| **Technical Definition** | NuGet package security combines **vulnerability auditing** (NuGet Audit against the GitHub Advisory Database, `dotnet list package --vulnerable`), **resolution integrity** (lock files with `--locked-mode`, package source mapping), **artifact trust** (author and repository signing, `signatureValidationMode=require`), and **version governance** (Central Package Management, TFM lifecycle). The attack classes are **malicious/backdoored packages**, **dependency confusion**, **typosquatting**, and **build‑time execution** via MSBuild targets and install scripts . |
| **Beginner‑Friendly Explanation** | NuGet package security is the **“trust but verify”** approach to dependencies. Trust: use packages from known authors and feeds. Verify: check them against vulnerability databases, pin exactly what you resolved, and make sure the artifact you downloaded is the artifact the author signed. The goal is to make sure that nothing enters your build that you did not explicitly intend — and that everything you did intend is free of known vulnerabilities. |

### Purposes

- **To** **detect known vulnerabilities** in direct and transitive packages before they reach production.
- **To** **prevent dependency confusion** — stop a public package from shadowing an internal package name with a higher version .
- **To** **make restores reproducible** — lock files record exact versions and content hashes so a compromised mirror cannot silently substitute a package .
- **To** **verify artifact authenticity** — signing and `signatureValidationMode=require` reject unsigned or tampered packages .
- **To** **govern versions consistently** — Central Package Management ensures one version per package across the solution .

### Syntax Rules and Structure

#### NuGet Audit (`.csproj`)

```xml
<PropertyGroup>
  <NuGetAudit>true</NuGetAudit>
  <NuGetAuditLevel>moderate</NuGetAuditLevel>
  <NuGetAuditMode>all</NuGetAuditMode>
</PropertyGroup>
```

**Component breakdown:**

| Property | Purpose |
|----------|---------|
| `NuGetAudit` | Enables audit during restore (default `true` in .NET 8+). |
| `NuGetAuditLevel` | Minimum severity to report (`low`, `moderate`, `high`, `critical`). |
| `NuGetAuditMode` | `direct` (default for net8.0) or `all` (includes transitive; default for net10.0+) . |

#### Package Source Mapping (`nuget.config`)

```xml
<packageSourceMapping>
  <packageSource key="nuget.org">
    <package pattern="*" />
  </packageSource>
  <packageSource key="internal-feed">
    <package pattern="Contoso.*" />
  </packageSource>
</packageSourceMapping>
```

#### Lock Files (`.csproj` + CI)

```xml
<PropertyGroup>
  <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
</PropertyGroup>
```

```bash
dotnet restore --locked-mode
```

#### Signature Validation (`nuget.config`)

```xml
<config>
  <add key="signatureValidationMode" value="require" />
</config>
```

**Syntax rules:**

- **`NuGetAuditMode=all`** should be set explicitly on .NET 8/9 projects; .NET 10+ defaults to `all` for transitive packages .
- **Package source mapping** is the single highest‑value control against dependency confusion — internal package names always resolve from the internal feed .
- **Lock files** must be committed to source control and enforced in CI with `--locked-mode` .
- **`signatureValidationMode=require`** rejects unsigned packages; combine with trusted signer configuration .

**Constraints and limitations:**

- **A clean restore is not a clean supply chain** — audit sources may be unreachable (`NU1900`), may not provide vulnerability data (`NU1905`), or may be skipped for up‑to‑date projects .
- **`packages.config` projects have limited audit** — `NuGetAuditMode` does not apply, and audit support began later (NuGet 6.10) .
- **Lock files do not cover SDK/runtime vulnerabilities** — TFM lifecycle must be checked separately .

### Annotated Complete Step‑by‑Step Code Example

```xml
<!-- Contoso.Api.csproj -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditLevel>moderate</NuGetAuditLevel>
    <NuGetAuditMode>all</NuGetAuditMode>
    <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.0" />
  </ItemGroup>
</Project>
```

```xml
<!-- nuget.config -->
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="internal-feed" value="https://pkgs.contoso.com/nuget/v3/index.json" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="nuget.org">
      <package pattern="*" />
    </packageSource>
    <packageSource key="internal-feed">
      <package pattern="Contoso.*" />
    </packageSource>
  </packageSourceMapping>
  <config>
    <add key="signatureValidationMode" value="require" />
  </config>
</packageSourceMapping>
</configuration>
```

**Step-by-step explanation:**

1. NuGet Audit is enabled with `NuGetAuditMode=all` so transitive packages are checked on .NET 10.
2. Lock files are enabled; `packages.lock.json` will be generated on restore and must be committed.
3. Package source mapping ensures `Contoso.*` packages resolve only from the internal feed — dependency confusion is blocked even if a same‑named package exists on nuget.org.
4. Signature validation is required, so unsigned packages fail the restore.

**Expected Output:** Restore produces audit warnings for vulnerable packages (elevated to errors in CI), resolves `Contoso.*` only from the internal feed, and rejects unsigned artifacts.

**Why this output?** Each control closes a different attack class: audit catches known CVEs, mapping blocks dependency confusion, lock files prevent silent substitution, and signing verifies authenticity.

### Real‑World Cases

- **Enterprise .NET solutions with private feeds** — package source mapping is mandatory to prevent dependency confusion .
- **Open‑source libraries** — lock files and signing verification protect consumers from compromised mirrors .
- **Multi‑project solutions** — Central Package Management ensures consistent versions and simplifies auditing .

### References

- NuGet Supply Chain Security Guide (.NET, 2026) – https://safeguard.sh/resources/blog/nuget-supply-chain-security
- .NET NuGet Package Security Guide – https://safeguard.sh/resources/blog/dotnet-nuget-package-security
- .NET Supply Chain Audit Patterns – https://safeguard.sh/resources/blog/dotnet-supply-chain-audit-patterns
- NuGet packages.config or PackageReference: which gets audited? – https://safeguard.sh/resources/blog/nuget-packages-config-or-packagereference-scan-coverage
- Auditing package dependencies for security vulnerabilities (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore

---

## 2. `dotnet list package --vulnerable` (The CLI Listing)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `dotnet list package --vulnerable` is a .NET CLI command that enumerates the packages in a project or solution and reports any with known vulnerabilities from the configured audit sources. |
| **Technical Definition** | The command was introduced in NuGet 5.9 and checks packages against the **GitHub Advisory Database** (via the default audit source). By default it lists **direct** packages only; `--include-transitive` extends the listing to transitive packages. `--format json` produces machine‑readable output suitable for CI parsing. It is distinct from **NuGet Audit during restore**, which emits warnings automatically; `dotnet list package --vulnerable` is an explicit, on‑demand listing . |
| **Beginner‑Friendly Explanation** | `dotnet list package --vulnerable` is the **“show me what is vulnerable”** command. Run it in a project or solution, and it prints a table of packages with known CVEs, the severity, and the fixed version. It is the fastest way to get a snapshot of your vulnerability posture without parsing restore output. The `--include-transitive` flag is essential — transitive packages are not listed by default . |

### Purposes

- **To** **enumerate vulnerable packages** explicitly, independent of restore warnings.
- **To** **include transitive dependencies** with `--include-transitive` — the majority of the risk lives there.
- **To** **produce machine‑readable output** (`--format json`) for CI gates and dashboards.
- **To** **verify that audit sources are working** — combine with restore warnings (`NU1900`, `NU1905`) to confirm coverage .
- **To** **track deprecation** alongside vulnerabilities with `--deprecated` .

### Syntax Rules and Structure

```bash
dotnet list package --vulnerable --include-transitive
```

```bash
dotnet list package --vulnerable --include-transitive --format json
```

**Component breakdown:**

| Option | Purpose |
|--------|---------|
| `--vulnerable` | Lists packages with known vulnerabilities. |
| `--include-transitive` | Includes transitive packages (essential). |
| `--format json` | Machine‑readable output for CI. |
| `--deprecated` | Lists deprecated packages (separate run). |

**Syntax rules:**

- **Always use `--include-transitive`** — direct‑only listings miss most of the risk .
- **Run against the solution, not individual projects**, to capture the full graph .
- **Use `--format json` in CI** and parse the `vulnerabilities` arrays .

**Constraints and limitations:**

- **A clean result is not proof of a clean supply chain** — it reflects the advisory data at that moment; new advisories can appear later .
- **Coverage depends on the SDK/NuGet version** — older tooling may not audit `packages.config` or transitive packages .
- **It does not replace restore‑time audit** — restore audit runs on every build; the CLI listing is an explicit snapshot.

### Annotated Complete Step‑by‑Step Code Example

```bash
# 1. Restore first so the resolved graph is available
dotnet restore

# 2. List vulnerable packages, including transitive
dotnet list package --vulnerable --include-transitive

# 3. Produce JSON for CI parsing
dotnet list package --vulnerable --include-transitive --format json > vulnerabilities.json
```

```bash
# CI gate: fail if any vulnerable package is found
if jq -e '.projects[].frameworks[].topLevelPackages[] | select(.vulnerabilities != null)' vulnerabilities.json; then
  echo "Vulnerable packages detected"
  exit 1
fi
```

**Step-by-step explanation:**

1. `dotnet restore` resolves the graph and writes `project.assets.json`.
2. The CLI listing reads the resolved graph and checks it against audit sources.
3. JSON output is parsed by `jq`; the build fails if any top‑level or transitive package has vulnerabilities.

**Expected Output:** A table (or JSON) listing each vulnerable package, its severity, and the fixed version; the CI step fails the build when vulnerabilities are present.

**Why this output?** The command reads the resolved dependency graph and matches it against the advisory database.

### Real‑World Cases

- **CI security gates** — run on every PR and fail on high/critical vulnerabilities .
- **Pre‑release audit** — run before tagging a release to confirm the shipped graph is clean.
- **Dependency inventory** — combine with `--include-transitive` to build an SBOM‑like inventory .

### References

- NuGet packages.config or PackageReference: which gets audited? (CLI history and options) – https://safeguard.sh/resources/blog/nuget-packages-config-or-packagereference-scan-coverage
- .NET NuGet Package Security Guide (`dotnet list package --vulnerable`) – https://safeguard.sh/resources/blog/dotnet-nuget-package-security
- .NET Supply Chain Audit Patterns (`--include-transitive --format json`) – https://safeguard.sh/resources/blog/dotnet-supply-chain-audit-patterns

---

## 3. Dependabot (Automated Dependency Updates)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dependabot is GitHub’s automated dependency management service that opens pull requests to update dependencies (including vulnerable ones) and, optionally, automerges them according to configured rules. |
| **Technical Definition** | Dependabot is configured via `.github/dependabot.yml` with `version: 2` and one `updates` entry per ecosystem. For .NET, the `package-ecosystem` value is **`nuget`**, and the `directory` points to the location of the `.csproj`/`.fsproj` files . Dependabot reads the manifest, checks for newer versions and security advisories, and opens PRs that run the project’s CI. It supports schedules, grouping, labels, reviewers, and automerge via GitHub Actions . |
| **Beginner‑Friendly Explanation** | Dependabot is the **“robot that keeps your dependencies up to date”**. You add a small YAML file to your repository, and Dependabot checks for new versions and security advisories. When it finds an update, it opens a pull request with the version bump. Your CI runs on the PR; if everything passes, you merge (or let it automerge). It turns dependency maintenance from a periodic chore into a continuous, reviewable process. |

### Purposes

- **To** **automate dependency updates** — no more manual version checks.
- **To** **surface security advisories** as PRs, not just alerts.
- **To** **run CI on every update** — the PR is validated by the same pipeline as any other change.
- **To** **support grouped updates** — batch minor/patch updates into one PR to reduce noise .
- **To** **enable automerge** for low‑risk updates while keeping major versions manual.

### Syntax Rules and Structure

#### Minimal `.github/dependabot.yml`

```yaml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
```

#### Fuller Configuration (Grouping, Labels, Automerge)

```yaml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/src"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    groups:
      microsoft:
        patterns:
          - "Microsoft.*"
      minor-and-patch:
        applies-to: version-updates
        update-types:
          - "minor"
          - "patch"
    labels:
      - "dependencies"
    reviewers:
      - "security-team"
```

**Component breakdown:**

| Key | Purpose |
|-----|---------|
| `package-ecosystem: "nuget"` | Targets .NET projects (`.csproj`, `.fsproj`) . |
| `directory` | Location of the manifest relative to the repo root . |
| `schedule.interval` | `daily`, `weekly`, or `monthly` . |
| `groups` | Batch related updates into one PR. |
| `labels` / `reviewers` | Route PRs to the right people. |

**Syntax rules:**

- **`version: 2` is required** — the configuration is invalid without it .
- **`package-ecosystem: "nuget"`** is the value for .NET; `directory` must point to the folder containing the `.csproj`/`.fsproj` .
- **GitHub Actions must also be monitored** — add a separate `github-actions` entry with `directory: "/"` .
- **Grouping and automerge** reduce PR noise; automerge should be limited to low‑risk update types.

**Constraints and limitations:**

- **Dependabot opens PRs, not fixes** — the PR still must pass CI and review.
- **Automerge requires a workflow** — a GitHub Actions job that approves and merges qualifying PRs .
- **Dependabot does not replace SCA** — it surfaces updates; Snyk or NuGet Audit surfaces the underlying vulnerabilities.

### Annotated Complete Step‑by‑Step Code Example

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/New_York"
    open-pull-requests-limit: 5
    groups:
      microsoft-extensions:
        patterns:
          - "Microsoft.Extensions.*"
    labels:
      - "dependencies"
      - "nuget"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

**Step-by-step explanation:**

1. Dependabot monitors NuGet packages in the repository root every Monday at 09:00 Eastern.
2. `Microsoft.Extensions.*` updates are grouped into a single PR to reduce noise.
3. GitHub Actions workflows are also monitored weekly — a requirement for repositories using Actions .

**Expected Output:** Weekly PRs for outdated NuGet packages (grouped where configured) and for GitHub Actions updates; each PR triggers CI.

**Why this output?** Dependabot reads the manifests, checks for newer versions and advisories, and opens PRs on the configured schedule.

### Real‑World Cases

- **Security‑critical repositories** — daily schedule with `open-pull-requests-limit` to avoid PR floods.
- **Large monorepos** — grouped updates by package family (e.g., `Microsoft.*`, `Serilog.*`).
- **Automated maintenance** — automerge patch updates that pass CI, with manual review for major versions.

### References

- Dependabot for Dependency Management (ecosystems, `nuget` value) – https://infra.apache.org/dependabot.html
- Dependabot options reference (GitHub Docs) – https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference
- Dependabot configuration examples (GitHub Actions + NuGet) – https://github.com/Georges034302/secure-dotnet-api-codeql

---

## 4. Snyk (Software Composition Analysis)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Snyk is a commercial software composition analysis (SCA) platform that scans .NET projects for vulnerable open‑source dependencies, using the .NET SDK’s own resolution machinery to build an accurate dependency graph. |
| **Technical Definition** | Snyk’s NuGet plugin (`snyk/snyk-nuget-plugin`) parses `project.json`, `packages.config`, and **`project.assets.json`** — the restore output that contains every direct and transitive package with its exact resolved version per target framework . For SDK‑style `PackageReference` projects, **`dotnet restore` must run first** so `project.assets.json` exists; Snyk reads the graph the SDK resolved rather than reimplementing NuGet’s solver . It matches the graph against Snyk’s vulnerability database and supports CLI, IDE (Visual Studio extension), and GitHub Actions integration . |
| **Beginner‑Friendly Explanation** | Snyk is the **“deep scanner”** for your dependency tree. Unlike a simple manifest scan, Snyk runs the same restore the build runs, reads the resolved graph, and checks every package — direct and transitive — against its vulnerability database. It gives you a fix PR, a CLI gate for CI, and an IDE view. The catch: for modern .NET projects, you must run `dotnet restore` before `snyk test`, because Snyk reads the SDK’s resolved output, not the project file . |

### Purposes

- **To** **scan the real resolved graph** — `project.assets.json` reflects the SDK’s actual resolution, including Central Package Management and `Directory.Build.props` overrides .
- **To** **detect vulnerabilities in transitive packages** — the majority of the tree.
- **To** **remediate** — Snyk opens fix PRs for vulnerable packages.
- **To** **integrate into CI** via the Snyk CLI or GitHub Action .
- **To** **surface results in the IDE** — the Visual Studio extension shows findings in‑editor .

### Syntax Rules and Structure

#### CLI (Modern `PackageReference` Projects)

```bash
dotnet restore
snyk test
```

#### GitHub Actions

```yaml
- name: Restore
  run: dotnet restore

- name: Snyk test
  uses: snyk/actions/dotnet@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

**Component breakdown:**

| Step | Purpose |
|------|---------|
| `dotnet restore` | Produces `obj/project.assets.json` — **required** before `snyk test` . |
| `snyk test` | Scans the resolved graph against Snyk’s database. |
| `SNYK_TOKEN` | Authenticates the Snyk CLI/action. |

**Syntax rules:**

- **`dotnet restore` must run first** for `PackageReference` projects — Snyk reads the resolved assets file .
- **`packages.config` projects** use `nuget install -OutputDirectory packages` before `snyk test` for accurate results .
- **The Snyk GitHub Action** requires the token as a secret .

**Constraints and limitations:**

- **Snyk does not support `PackageReference` without a version attribute** — add explicit versions or use Central Package Management with versions .
- **Static analysis fallback** — older `packages.config` projects without `project.assets.json` use less accurate static analysis .
- **Commercial licensing** — Snyk is a paid product with a free tier; check plan limits .

### Annotated Complete Step‑by‑Step Code Example

```yaml
name: SCA
on: [push, pull_request]

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Restore
        run: dotnet restore

      - name: Snyk test
        uses: snyk/actions/dotnet@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

**Step-by-step explanation:**

1. The workflow checks out the repository and installs .NET 10.
2. `dotnet restore` produces `project.assets.json` — Snyk’s required input .
3. The Snyk action scans the resolved graph and fails on high‑severity findings.

**Expected Output:** The workflow fails if any high‑severity vulnerability is found in the resolved dependency graph.

**Why this output?** Snyk reads the SDK‑resolved graph, ensuring transitive packages and CPM overrides are included .

### Real‑World Cases

- **Enterprise codebases with legacy and modern projects** — Snyk handles both `packages.config` and `PackageReference` .
- **GitHub‑hosted repositories** — the Snyk GitHub Action integrates with code scanning .
- **IDE‑centric teams** — the Visual Studio extension shows findings while editing .

### References

- How Snyk Scans NuGet and .NET Files for Vulnerabilities – https://safeguard.sh/resources/blog/how-snyk-scans-nuget-and-net-project-files-for-vulnerable-dependencies
- Snyk for .NET (User Docs) – https://docs.snyk.io/~/changes/m2hJeDrttXy2N83AnRjM/scan-application-code/snyk-open-source/snyk-open-source-supported-languages-and-package-managers/snyk-for-.net
- Snyk dotNET action (User Docs) – https://docs.snyk.io/~/changes/m2hJeDrttXy2N83AnRjM/integrations/
- How Snyk’s VS Extension Scans .NET Solutions – https://safeguard.sh/resources/blog/how-snyks-vs-extension-scans-net-solutions

---

## 5. Dependency Security Best Practices (The Review Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dependency security best practices are the reviewable, automatable controls that ensure every package entering the build is audited, pinned, signed, and maintained — and that the resolved graph is exactly what the team intended. |
| **Technical Definition** | Best practices span **audit** (NuGet Audit with `all` mode, `dotnet list package --vulnerable --include-transitive`, Snyk), **resolution integrity** (lock files with `--locked-mode`, package source mapping), **artifact trust** (signing, `signatureValidationMode=require`), **version governance** (Central Package Management, TFM support lifecycle), **automation** (Dependabot, Snyk CI), and **CI gates** (warnings as errors, `RestoreProjectsAuditedCount` checks) . |
| **Beginner‑Friendly Explanation** | These are the **“do not ship without these”** rules for dependencies. The biggest mistakes are: trusting a clean restore without checking audit coverage, listing only direct packages, leaving floating versions, ignoring transitive packages, and never verifying that the package you downloaded is signed. The checklist turns dependency security from a periodic audit into a continuous, enforced property of the build. |

### Purposes

- **To** **make the secure path the default** — audit on every restore, lock files committed, mapping configured.
- **To** **catch transitive vulnerabilities** — `NuGetAuditMode=all` and `--include-transitive` are non‑negotiable.
- **To** **prevent dependency confusion** — package source mapping and Central Package Management.
- **To** **verify artifact authenticity** — signing verification rejects unsigned or tampered packages.
- **To** **automate remediation** — Dependabot and Snyk open fix PRs; CI blocks vulnerable merges.
- **To** **fail loudly on drift** — `--locked-mode` and warnings‑as‑errors turn silent substitutions into build failures.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Audit** | Enable `NuGetAudit` with `NuGetAuditMode=all`; run `dotnet list package --vulnerable --include-transitive` in CI . | Rely on a clean restore as proof of safety — check `NU1900`/`NU1905` . |
| **Transitive packages** | Always include transitive packages in audit and listing . | Audit direct packages only — transitive is the majority of the risk . |
| **Lock files** | Commit `packages.lock.json`; enforce `dotnet restore --locked-mode` in CI . | Use floating versions without a lock file. |
| **Source mapping** | Configure `packageSourceMapping` so internal names resolve only from internal feeds . | Let NuGet resolve from any feed — dependency confusion risk. |
| **Signing** | Set `signatureValidationMode=require` and configure trusted signers . | Accept unsigned packages silently. |
| **Version governance** | Use Central Package Management (`Directory.Packages.props`) for consistent versions . | Let each project pin different versions of the same package. |
| **TFM lifecycle** | Audit target frameworks against Microsoft’s support lifecycle; upgrade out‑of‑support TFMs . | Run .NET 6/7 after end of support — no framework security patches. |
| **Automation** | Enable Dependabot for `nuget` and `github-actions`; configure Snyk in CI . | Rely on manual dependency reviews. |
| **Remediation** | Patch high/critical vulnerabilities immediately; track medium/low with an SLA. | Defer remediation indefinitely. |
| **Pruning** | Remove unused packages; fewer dependencies mean less risk . | Keep transitive cruft “just in case.” |

### Annotated Code Example (Best‑Practice CI Pipeline)

```yaml
# .github/workflows/security.yml
name: Dependency Security
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Restore (locked mode)
        run: dotnet restore --locked-mode

      - name: Vulnerable packages (including transitive)
        run: dotnet list package --vulnerable --include-transitive --format json > vuln.json

      - name: Fail on vulnerabilities
        run: |
          if jq -e '.projects[].frameworks[].topLevelPackages[] | select(.vulnerabilities != null)' vuln.json; then
            echo "Vulnerable packages found"; exit 1
          fi

      - name: Snyk test
        uses: snyk/actions/dotnet@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

**Why this follows best practices:**

- **`--locked-mode`** fails if the lock file and manifests disagree — silent substitution is impossible .
- **`--include-transitive`** ensures transitive packages are audited .
- **JSON parsing** turns the listing into a CI gate.
- **Snyk** adds a second, independent vulnerability database .
- **Dependabot** (configured separately) opens fix PRs for the findings.

### Real‑World Cases

- **Enterprise CI pipelines** — locked restore, vulnerable listing, and Snyk gate on every PR.
- **Open‑source libraries** — lock files, signing verification, and Dependabot for consumers.
- **Regulated industries** — audit coverage verification (`RestoreProjectsAuditedCount`) and remediation SLAs .

### References

- NuGet Supply Chain Security Guide (lock files, source mapping, audit) – https://safeguard.sh/resources/blog/nuget-supply-chain-security
- .NET NuGet Package Security Guide (signing, CPM, NuGetAudit) – https://safeguard.sh/resources/blog/dotnet-nuget-package-security
- .NET Supply Chain Audit Patterns (TFM drift, transitive resolution) – https://safeguard.sh/resources/blog/dotnet-supply-chain-audit-patterns
- NuGet packages.config or PackageReference: which gets audited? (coverage gaps, `NU1900`/`NU1905`) – https://safeguard.sh/resources/blog/nuget-packages-config-or-packagereference-scan-coverage
- Securing the .NET Supply Chain: Best Practices for Managing NuGet Dependencies (M365.FM) – https://www.m365.fm/
- NuGet Audit documentation (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore

---

## Summary Table: Dependency Security at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|----------|--------------|-------------------|-------------|
| **NuGet Package Security** | Audit, pin, sign, and govern dependencies | `NuGetAudit`, `packageSourceMapping`, lock files, `signatureValidationMode` | Every .NET project |
| **`dotnet list package --vulnerable`** | Explicit vulnerability listing | `--include-transitive --format json` | CI gates and pre‑release audits |
| **Dependabot** | Automated dependency update PRs | `.github/dependabot.yml`, `package-ecosystem: "nuget"` | Every GitHub repository |
| **Snyk** | Deep SCA of the resolved graph | `dotnet restore` + `snyk test`, GitHub Action | Enterprise SCA and IDE integration |
| **Best Practices** | The review checklist | Audit all, lock, map, sign, automate, gate | Every production build |

---

## References (Full List)

- NuGet Supply Chain Security Guide (.NET, 2026) – https://safeguard.sh/resources/blog/nuget-supply-chain-security
- .NET NuGet Package Security Guide – https://safeguard.sh/resources/blog/dotnet-nuget-package-security
- .NET Supply Chain Audit Patterns – https://safeguard.sh/resources/blog/dotnet-supply-chain-audit-patterns
- NuGet packages.config or PackageReference: which gets audited? – https://safeguard.sh/resources/blog/nuget-packages-config-or-packagereference-scan-coverage
- How Snyk Scans NuGet and .NET Files for Vulnerabilities – https://safeguard.sh/resources/blog/how-snyk-scans-nuget-and-net-project-files-for-vulnerable-dependencies
- Snyk for .NET (User Docs) – https://docs.snyk.io/~/changes/m2hJeDrttXy2N83AnRjM/scan-application-code/snyk-open-source/snyk-open-source-supported-languages-and-package-managers/snyk-for-.net
- Snyk dotNET action (User Docs) – https://docs.snyk.io/~/changes/m2hJeDrttXy2N83AnRjM/integrations/
- How Snyk’s VS Extension Scans .NET Solutions – https://safeguard.sh/resources/blog/how-snyks-vs-extension-scans-net-solutions
- Dependabot for Dependency Management – https://infra.apache.org/dependabot.html
- Dependabot options reference (GitHub Docs) – https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference
- Dependabot configuration examples (GitHub Actions + NuGet) – https://github.com/Georges034302/secure-dotnet-api-codeql
- NuGet Audit documentation (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore
- Securing the .NET Supply Chain: Best Practices for Managing NuGet Dependencies (M365.FM) – https://www.m365.fm/
- Use dotnet-dependency skill (GitHub) – https://github.com/dotnet/skills
- .NET Supply Chain Audit Patterns (`--include-transitive --format json`) – https://safeguard.sh/resources/blog/dotnet-supply-chain-audit-patterns