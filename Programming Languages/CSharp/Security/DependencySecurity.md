# Dependency Security in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Dependency security is the practice of identifying, assessing, and mitigating risks introduced by third-party software components — NuGet packages, their transitive dependencies, and the build and distribution infrastructure that delivers them — so that a .NET application does not inherit vulnerabilities, backdoors, or licence obligations that compromise its security posture.

**Technical Definition**
Dependency security in the .NET ecosystem operates across four control planes. **Discovery** enumerates the complete resolved dependency graph, including transitive packages that the application never references directly. **Vulnerability assessment** matches resolved package versions against advisory databases — the GitHub Advisory Database is the canonical source for NuGet, and NuGet Audit, `dotnet list package --vulnerable`, Dependabot, Snyk, and OWASP Dependency-Check all consume it directly or indirectly. **Supply-chain hardening** ensures that the packages that enter the build are the packages that were intended: package source mapping defeats dependency confusion, lock files make restores reproducible, signed packages verify provenance, and central package management pins versions across a solution. **Continuous monitoring** keeps the assessment current as new CVEs are published and as the dependency graph changes. NuGet Audit, introduced in .NET 8 and extended to transitive dependencies by default in .NET 9, runs automatically on every `dotnet restore` and emits warnings (NU1901–NU1904) when known-vulnerable packages are found. The .NET 9 SDK changed `NuGetAuditMode` from `direct` to `all`, so transitive vulnerabilities are reported by default; .NET 10 adds `dotnet package update --vulnerable` for automatic remediation of direct dependencies.

**Beginner-Friendly Explanation**
Dependency security is "making sure the code you did not write cannot hurt you." Every NuGet package you add — and every package those packages pull in — runs with the same privileges as your application. If one of them has a known vulnerability, or if an attacker manages to publish a malicious version under a name your build trusts, your application is compromised even though none of your own code is wrong. Dependency security is the set of habits and tools that tell you what you are actually shipping, warn you when something in that list becomes dangerous, and make it hard for an attacker to slip something in.

---

### Key Characteristics

- **The dependency tree is the largest attack surface** — A typical ASP.NET Core service pulls in dozens of transitive packages; a single vulnerable one is enough to compromise the application.
- **Transitive-by-default reporting** — .NET 9+ audits transitive packages by default; .NET 8 required explicit configuration.
- **Automated on every restore** — NuGet Audit runs during `dotnet restore` and emits NU1901–NU1904 warnings without any setup.
- **Two complementary reporting modes** — NuGet Audit gives build-time warnings; `dotnet list package --vulnerable` gives an on-demand, machine-readable report suitable for scheduled reviews and evidence.
- **Supply-chain attacks are real and recent** — The SolarWinds SUNBURST compromise was a .NET supply-chain attack; dependency confusion and typosquatting are practical, demonstrated attack classes.
- **Defence requires layered controls** — Auditing alone is insufficient; package source mapping, lock files, signature verification, and CI enforcement are all part of the picture.
- **Tool-specific strengths** — Dependabot is GitHub-native and zero-config; Snyk offers deeper dependency-graph precision and reachability; OWASP Dependency-Check is a free, standalone scanner suitable for air-gapped environments.

---

### Prerequisites

- A .NET solution with `PackageReference` items (or `packages.config` for legacy projects).
- The .NET 8 SDK or later for NuGet Audit; .NET 9 SDK or later for transitive auditing by default.
- A source that provides vulnerability data — `nuget.org` is the standard source, and private feeds must configure `auditSources` explicitly.
- A GitHub repository (for Dependabot) or a Snyk account (for Snyk scanning).
- Familiarity with `nuget.config`, `Directory.Build.props`, and lock files.
- Awareness of OWASP Top 10 **A03:2025 Software Supply Chain Failures**, which widened the vulnerable-components category to include build pipelines and distribution.

---

### Related Programming Areas

- **Common vulnerabilities** — Dependency risk is a root cause of injection, deserialisation, and cryptographic failures.
- **Secure coding** — The code you write must not amplify the risk of the code you consume.
- **CI/CD security** — Dependency scanning belongs in the pipeline, not in a quarterly manual review.
- **Software supply-chain security** — SBOM generation, package signing, and provenance attestation.
- **OWASP Top 10:2025** — A03 Software Supply Chain Failures is the category that covers this discipline.

---

### Core Concepts / Features

1. NuGet Package Security
2. `dotnet list package --vulnerable`
3. Dependabot
4. Snyk
5. Dependency Security Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with "To"), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. NuGet Package Security

### Definitions

**Core Definition**
NuGet package security is the discipline of ensuring that the NuGet packages an application resolves — direct and transitive — are free of known vulnerabilities, are sourced from trusted feeds, are not subject to dependency confusion or typosquatting, and are verified for provenance.

**Technical Definition**
NuGet package security spans four mechanisms. **NuGet Audit** checks resolved package versions against the GitHub Advisory Database during `dotnet restore` and reports warnings NU1901 (low), NU1902 (moderate), NU1903 (high), and NU1904 (critical). **Package source mapping** (NuGet 6.0+) restricts which feeds NuGet will search for which package patterns, defeating dependency confusion by ensuring that `Contoso.*` can only ever come from the internal feed. **Lock files** (`packages.lock.json`) record the exact resolved version and content hash of every package, enabling `--locked-mode` restores that fail on drift. **Signature verification** rejects unsigned or tampered packages and prefers feeds that enforce author or repository signing. Central package management (`Directory.Packages.props`) pins versions solution-wide and is compatible with source mapping and SBOM generation.

**Beginner-Friendly Explanation**
NuGet package security is "keeping your package supply chain clean." It means four things: know what you are shipping (audit every package, including the ones your packages pull in), know where it came from (pin package names to the right feeds so nobody can sneak a same-named package in from nuget.org), know it has not changed (lock files record the exact version and hash), and know it was published by who it says (signed packages). The .NET SDK gives you most of this out of the box; the job is configuring it so the unsafe path is the hard path.

---

### Purposes

- To detect known-vulnerable packages during every restore, including transitive dependencies.
- To prevent dependency confusion by pinning package names to trusted feeds.
- To make restores reproducible so a compromised mirror cannot silently substitute a package.
- To verify that packages were published by their claimed authors.
- To centralise version management across a solution.
- To produce evidence (reports, SBOMs) for compliance and audit.
- To satisfy OWASP A03:2025 Software Supply Chain Failures.

---

### Syntax Rules and Structure

#### NuGet Audit Configuration (`Directory.Build.props`)

```xml
<Project>
  <PropertyGroup>
    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditMode>all</NuGetAuditMode>          <!-- direct | all -->
    <NuGetAuditLevel>low</NuGetAuditLevel>        <!-- low | moderate | high | critical -->
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
</Project>
```

**Component breakdown**

- `NuGetAudit` — Enables or disables auditing. `true` by default since .NET 8.
- `NuGetAuditMode` — `direct` checks only packages you reference; `all` also checks transitive. Default is `all` for .NET 10+ and `direct` for older targets.
- `NuGetAuditLevel` — The minimum severity that produces a warning.
- `TreatWarningsAsErrors` — Promotes audit warnings (NU1901–NU1904) to build failures.

#### Package Source Mapping (`nuget.config`)

```xml
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="internal-feed" value="https://internal.example.com/nuget/index.json" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="nuget.org">
      <package pattern="*" />
    </packageSource>
    <packageSource key="internal-feed">
      <package pattern="Contoso.*" />
    </packageSource>
  </packageSourceMapping>
</configuration>
```

**Component breakdown**

- `<clear />` — Removes machine-level and inherited feeds, ensuring deterministic behaviour across developers and CI.
- `packageSourceMapping` — Declares which patterns resolve from which source. `Contoso.*` can only come from `internal-feed`; everything else from `nuget.org`.
- **Limitation** — Source mapping applies to restore, install, and update, but not to metadata lookups performed by `dotnet list package` (including `--vulnerable`) or the Package Manager UI.

#### Lock Files

```xml
<PropertyGroup>
  <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
</PropertyGroup>
```

```bash
dotnet restore --locked-mode
```

**Component breakdown**

- `RestorePackagesWithLockFile` — Generates `packages.lock.json` alongside the project.
- `--locked-mode` — Restore fails if the lock file and the project's declared dependencies disagree, turning silent substitution into a loud failure.

#### Audit Sources (Private Feed Scenarios)

```xml
<auditSources>
  <clear />
  <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
</auditSources>
```

- When all package sources are private, NuGet cannot reach a vulnerability database. Add `nuget.org` as an explicit audit source separate from `packageSources`.

#### Syntax Rules

- Commit a shared `nuget.config` to the repository root; do not rely on machine-level configuration.
- Set `NuGetAuditMode` to `all` and promote NU1903/NU1904 to errors in CI.
- Use package source mapping for every internal package namespace.
- Commit `packages.lock.json` and enforce `--locked-mode` in CI.
- Prefer signed packages and feeds that require signing.
- Do not combine `NuGetAudit` with `EnsureCreated`-style workarounds; resolve the underlying vulnerability.

#### Constraints and Limitations

- NuGet Audit only reports known vulnerabilities; it does not assess whether your code actually calls the vulnerable function.
- `dotnet list package` metadata lookups bypass source mapping, so package IDs are sent to every configured source.
- Lock files must be regenerated deliberately when dependencies change; `--locked-mode` will otherwise fail.
- Signature verification requires the feed and tooling to support it; not all private feeds do.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Elevating Audit Warnings to Errors

```xml
<Project>
  <PropertyGroup>
    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditMode>all</NuGetAuditMode>
    <NuGetAuditLevel>low</NuGetAuditLevel>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
</Project>
```

```bash
dotnet restore
```

**Expected Output (conceptual)**

```
error NU1903: Package 'Newtonsoft.Json' 12.0.1 has a known high severity vulnerability
  (https://github.com/advisories/GHSA-5crp-9r3c-p9vr)
Build FAILED.
```

**Why this output occurs**

- NuGet Audit matches the resolved version against the GitHub Advisory Database.
- `TreatWarningsAsErrors` promotes the warning to an error, failing the build.

---

#### Example 2: Package Source Mapping Against Dependency Confusion

```xml
<!-- nuget.config -->
<packageSourceMapping>
  <packageSource key="nuget.org">
    <package pattern="*" />
  </packageSource>
  <packageSource key="internal-feed">
    <package pattern="Contoso.*" />
  </packageSource>
</packageSourceMapping>
```

**Expected Output (conceptual)**

```
Contoso.Billing resolves only from internal-feed.
A malicious Contoso.Billing 99.0.0 published to nuget.org is never considered.
```

**Why this output occurs**

- The mapping restricts the search space per package pattern; NuGet will not fall back to `nuget.org` for `Contoso.*`.

---

### Real-World Cases

- **SolarWinds SUNBURST (2020)** — A .NET supply-chain attack in which malicious code was planted in a build pipeline and signed with a legitimate certificate; this is why provenance, not just CVEs, matters.
- **Dependency confusion (Birsan, 2021)** — Demonstrated across multiple ecosystems, including NuGet, by registering internal package names on public feeds with higher version numbers.
- **Typosquatting** — Packages named `Newtonsoft.Jsoon` or `Microsoft.AspNetcore` that a developer installs by mistake.
- **Build-time execution** — MSBuild targets and package install scripts can run code during restore or build, so a malicious dependency does not need to wait until runtime.

---

### References

- NuGet Audit: Stop Shipping Vulnerable .NET Packages — Coherent Solutions - https://www.coherentsolutions.com/insights/how-to-stop-shipping-solutions-with-vulnerable-dot-net-packages
- .NET Security Best Practices (2026 Guide) — Safeguard - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- NuGet Supply Chain Security Guide (.NET, 2026) — Safeguard - https://safeguard.sh/resources/blog/nuget-supply-chain-security
- Package Source Mapping — Microsoft Learn - https://learn.microsoft.com/en-us/nuget/consume-packages/package-source-mapping
- SDK updates in .NET 9 Preview 6 (NuGetAudit transitive) — dotnet/core - https://github.com/dotnet/core/blob/main/release-notes/9.0/preview/preview6/sdk.md
- Strengthening NuGet Supply Chain Security: Reducing API Key Lifetime — .NET Blog - https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/
- Securing the .NET Supply Chain: Best Practices for Managing NuGet Dependencies — Mirko Peters - https://www.m365.fm/community/u/mirko-p/blog/securing-the-net-supply-chain-best-practices-for-managing-nuget-dependencies/

---

## 2. `dotnet list package --vulnerable`

### Definitions

**Core Definition**
`dotnet list package --vulnerable` is a .NET CLI command that produces an on-demand report of packages in a project or solution that have known vulnerabilities, optionally including transitive dependencies and machine-readable JSON output.

**Technical Definition**
The command queries the configured audit sources (from .NET SDK 9.0.300 onward, the `auditSources` section of `nuget.config`; previously the package sources) for advisories affecting the resolved versions of each package. It reports findings grouped by project and target framework. The `--include-transitive` flag adds transitive packages, which are not included by default. The `--format json` option produces structured output suitable for storage, diffing, or feeding downstream tools. The command cannot be combined with `--deprecated` or `--outdated` in the same invocation. Unlike NuGet Audit, which runs on every restore, `dotnet list package --vulnerable` gives a point-in-time report that serves as evidence for compliance and scheduled reviews.

**Beginner-Friendly Explanation**
`dotnet list package --vulnerable` is "the command that tells you which of your packages have known security holes." NuGet Audit warns you during every build; this command gives you a report you can save, compare week to week, and attach to a release. It does not include the packages your packages depend on unless you add `--include-transitive`, so the safe habit is to always pass that flag.

---

### Purposes

- To produce an on-demand inventory of vulnerable packages.
- To include transitive dependencies when `--include-transitive` is specified.
- To generate machine-readable JSON for storage, diffing, and CI artifacts.
- To provide evidence for compliance and audit trails.
- To support scheduled nightly scans independent of build-time warnings.
- To identify the specific packages and advisories that need remediation.

---

### Syntax Rules and Structure

#### General Syntax

```bash
dotnet list package --vulnerable --include-transitive --format json > vulnerable.json
```

**Component breakdown**

- `dotnet list package` — Lists packages for the project or solution in the current directory.
- `--vulnerable` — Filters to packages with known vulnerabilities.
- `--include-transitive` — Includes transitive packages (omitted by default).
- `--format json` — Emits machine-readable JSON.
- `>` — Redirects the report to a file for artifact storage.

#### Related Commands

```bash
# List installed packages
dotnet list package

# List outdated packages
dotnet list package --outdated

# List deprecated packages
dotnet list package --deprecated

# Find out why a transitive package is present
dotnet nuget why .\my.csproj Microsoft.Extensions.Primitives
```

**Component breakdown**

- `dotnet list package` — Shows requested and resolved versions.
- `--outdated` — Shows available updates.
- `--deprecated` — Shows packages marked deprecated by their authors.
- `dotnet nuget why` — Explains the dependency path that pulled in a transitive package.

#### Syntax Rules

- Always use `--include-transitive`; transitive packages are where many advisories live.
- Use `--format json` when the report will be stored or processed by another tool.
- Do not combine `--vulnerable` with `--deprecated` or `--outdated`.
- Run the command in CI as a scheduled job and store the JSON as an artifact.
- Pair the report with NuGet Audit; they answer different questions.

#### Constraints and Limitations

- The command reports known issues but does not tell you whether your code reaches the vulnerable function.
- Source mapping does not apply to metadata lookups; package IDs are sent to every configured source.
- The command is a point-in-time snapshot; it does not monitor for newly published advisories between runs.
- The report cannot be combined with `--deprecated` or `--outdated` in a single invocation.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: On-Demand Vulnerability Report

```bash
dotnet list package --vulnerable --include-transitive
```

**Expected Output (conceptual)**

```
Project 'MyApp' has the following vulnerable packages:
   [net8.0]:
   Top-level Package          Requested   Resolved   Severity   Advisory URL
   > Newtonsoft.Json          12.0.1      12.0.1     High       https://github.com/advisories/GHSA-...
   Transitive Package         Resolved               Severity   Advisory URL
   > System.Text.Json         4.7.0       4.7.0      Moderate   https://github.com/advisories/GHSA-...
```

**Why this output occurs**

- The command matches resolved versions against advisories and reports severity and advisory URL.
- `--include-transitive` surfaces the transitive `System.Text.Json` finding.

---

#### Example 2: Machine-Readable Report for CI

```bash
dotnet list package --vulnerable --include-transitive --format json > artifacts/vulnerable.json
```

**Expected Output (conceptual)**

```json
{
  "version": 1,
  "projects": [
    {
      "path": "MyApp.csproj",
      "frameworks": [
        {
          "framework": "net8.0",
          "vulnerablePackages": [
            {
              "id": "Newtonsoft.Json",
              "resolvedVersion": "12.0.1",
              "severity": "High",
              "advisoryUrl": "https://github.com/advisories/GHSA-..."
            }
          ]
        }
      ]
    }
  ]
}
```

**Why this output occurs**

- `--format json` emits structured data that can be diffed week over week or fed into a dashboard.

---

### Real-World Cases

- **Nightly compliance scans** — Store the JSON report as a CI artifact and open tickets for new findings.
- **Release evidence** — Attach the report to a release record to demonstrate that the shipped dependency graph was scanned.
- **Post-incident investigation** — Determine whether a newly published CVE affects the version you actually resolved.

---

### References

- dotnet list package vulnerable or NuGet Audit: which should I use? — Safeguard - https://safeguard.sh/resources/blog/dotnet-list-package-vulnerable-vs-nuget-audit-which-to-use
- Machine-readable output for dotnet list package — NuGet Home - https://github.com/nuget/home/blob/main/accepted/2020/MachineReadableOutputDotNetListPackage.md
- Upgrade app dependencies — Microsoft Learn - https://github.com/microsoftdocs/learn/blob/main/learn-pr/language/dotnet-dependencies/includes/5-exercise-dependency-management.md
- .NET reachability with dosai — OWASP dep-scan - https://github.com/owasp-dep-scan/dep-scan/blob/main/documentation/docs/languages/dotnet-reachability.mdx

---

## 3. Dependabot

### Definitions

**Core Definition**
Dependabot is GitHub's built-in dependency management tool that alerts on known vulnerabilities, automatically opens pull requests to fix vulnerable dependencies, and optionally keeps dependencies current with version-update PRs.

**Technical Definition**
Dependabot has three capabilities: **Dependabot alerts** (notifications when a dependency has a known CVE, powered by the dependency graph and the GitHub Advisory Database), **Dependabot security updates** (automatic PRs that bump a vulnerable dependency to a fixed version), and **Dependabot version updates** (scheduled PRs that keep dependencies current). For .NET, the `nuget` ecosystem value covers `*.csproj` and `packages.config`; `dotnet-sdk` covers `global.json`. Configuration lives in a single `.github/dependabot.yml` file on the default branch; GitHub does not support multiple Dependabot configuration files per repository. Security updates are triggered by advisories; version updates are driven by the schedule you configure.

**Beginner-Friendly Explanation**
Dependabot is "GitHub's automatic dependency watcher." When a CVE is published for a package you use, Dependabot alerts you and — if you enable security updates — opens a pull request that bumps the package to a safe version. You review and merge the PR like any other change. Version updates go further: they periodically propose updates even when there is no security issue, so you do not fall far behind.

---

### Purposes

- To alert on known vulnerabilities in dependencies without any setup beyond enabling the feature.
- To automatically open PRs that remediate vulnerable dependencies.
- To keep dependencies current through scheduled version-update PRs.
- To integrate dependency hygiene into the normal pull-request workflow.
- To group related updates so review effort stays manageable.
- To satisfy supply-chain monitoring expectations with a GitHub-native, zero-cost tool.

---

### Syntax Rules and Structure

#### General Syntax (`.github/dependabot.yml`)

```yaml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    groups:
      microsoft:
        patterns:
          - "Microsoft.*"
      testing:
        patterns:
          - "xunit*"
          - "Microsoft.NET.Test.Sdk"
```

**Component breakdown**

- `package-ecosystem: "nuget"` — Enables NuGet dependency scanning.
- `directory` — Where the manifests live; use `directories` (plural) for glob patterns in monorepos.
- `schedule.interval` — `daily`, `weekly`, or `monthly`.
- `open-pull-requests-limit` — Caps concurrent PRs.
- `groups` — Batches related packages into a single PR to reduce review noise.

#### Enabling Security Features

- **Dependabot alerts** — Enabled at the repository or organisation level (Settings → Code security and analysis).
- **Dependabot security updates** — A separate toggle; without it, alerts notify but do not remediate.
- **Dependabot version updates** — Driven by `dependabot.yml`.

#### Syntax Rules

- Commit `.github/dependabot.yml` on the default branch; it is the only supported configuration file.
- Use `groups` to keep PR volume manageable.
- Review Dependabot PRs like any other change; they still need CI to pass.
- Enable both security updates and version updates; they serve different purposes.
- Pin GitHub Actions to commit SHAs independently of Dependabot, since `uses:` lines are a separate supply-chain concern.

#### Constraints and Limitations

- Dependabot relies on the GitHub Advisory Database; advisories not in that database are not reported.
- Security updates only fire when a fixed version exists.
- Version-update PRs can be noisy without grouping and scheduling discipline.
- Dependabot does not assess reachability; it reports that a dependency is vulnerable, not that your code calls the vulnerable function.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Grouped Weekly Updates

```yaml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      aspnetcore:
        patterns:
          - "Microsoft.AspNetCore.*"
      efcore:
        patterns:
          - "Microsoft.EntityFrameworkCore.*"
```

**Expected Output (conceptual)**

```
One PR: "Bump Microsoft.AspNetCore.* group from 8.0.1 to 8.0.8"
One PR: "Bump Microsoft.EntityFrameworkCore.* group from 8.0.2 to 8.0.10"
```

**Why this output occurs**

- Grouping batches related packages into a single PR, reducing review overhead.

---

#### Example 2: Security Update PR

**Expected Output (conceptual)**

```
Dependabot opened PR #42: "Bump Newtonsoft.Json from 12.0.1 to 13.0.3"
Reason: GHSA-5crp-9r3c-p9vr (High severity)
CI passed; ready for review.
```

**Why this output occurs**

- A security advisory matched a resolved dependency; Dependabot generated a PR that bumps it to the fixed version.

---

### Real-World Cases

- **Open-source repositories** — Dependabot alerts and security updates are free for public repositories and widely adopted.
- **Enterprise GitHub organisations** — Centralised enablement across repositories, with grouping to keep PR noise manageable.
- **Monorepos** — `directories` with glob patterns covers multiple projects in one configuration.

---

### References

- Dependabot Configuration & Management — GitHub Awesome Copilot - https://github.com/github/awesome-copilot/blob/main/skills/dependabot/SKILL.md
- Setting Up Dependency Scanning on GitHub — Safeguard - https://safeguard.sh/resources/blog/setting-up-dependency-scanning-github
- Dependabot documentation — GitHub Docs - https://docs.github.com/en/code-security/dependabot

---

## 4. Snyk

### Definitions

**Core Definition**
Snyk is a commercial software composition analysis (SCA) platform that scans .NET and NuGet projects for vulnerable dependencies, provides precise dependency graphs, and offers automated fixes.

**Technical Definition**
Snyk supports .NET through SCM integrations (importing repositories from GitHub, GitLab, Bitbucket, and Azure Repos) and through the Snyk CLI. For SCM integrations, NuGet is the only supported package manager, and Snyk builds dependency trees from `*.csproj`, `*.vbproj`, `*.fsproj`, `global.json`, `Directory.Build.props`, and `Directory.Packages.props`. For CLI scans, Snyk reads `project.assets.json`, which is produced by `dotnet restore`; the required sequence is `dotnet restore` followed by `snyk test`. Snyk expanded support for self-hosted and private NuGet packages in 2026 and uses the .NET SDK directly to resolve dependencies for a highly precise dependency graph. When a fix is available, Snyk can update the version in the manifest automatically for projects using `PackageReference` or `packages.config`.

**Beginner-Friendly Explanation**
Snyk is "a dedicated security scanner for your dependencies." Unlike Dependabot, which lives inside GitHub, Snyk is a platform you connect to your repository or run from the command line. It builds a precise picture of your NuGet dependency tree — including transitive packages and private feeds — and tells you which packages are vulnerable and what to upgrade to. It can also open fix pull requests.

---

### Purposes

- To obtain a precise, SDK-resolved dependency graph for .NET projects.
- To scan private and self-hosted NuGet feeds in addition to nuget.org.
- To identify vulnerabilities with remediation advice and automated fixes.
- To integrate scanning into SCM workflows and CI pipelines.
- To separate open-source dependency scanning from source-code scanning.
- To provide a commercial support and policy layer on top of advisory data.

---

### Syntax Rules and Structure

#### General Syntax (CLI)

```bash
# Restore first so project.assets.json exists
dotnet restore

# Scan the project
snyk test

# Scan a specific solution
snyk test --file=MySolution.sln

# Scan all projects in a mixed repository
snyk test --all-projects

# Monitor (continuous) rather than one-off test
snyk monitor
```

**Component breakdown**

- `dotnet restore` — Produces `obj/project.assets.json`, which Snyk reads.
- `snyk test` — Runs the SCA scan.
- `--file=` — Targets a specific `.sln` or project file.
- `--all-projects` — Scans all projects in a multi-language or multi-project repository.
- `snyk monitor` — Sends the dependency snapshot to Snyk for continuous monitoring.

#### SCM Integration Syntax

- Import the repository into Snyk; Snyk builds the dependency tree from the supported manifest files.
- Configure the Group or Organisation .NET settings to include or exclude build/development dependencies.
- Snyk creates a separate project per target framework, isolating dependencies.

#### Syntax Rules

- Always run `dotnet restore` before `snyk test` when using the CLI.
- Use `--all-projects` for solutions with multiple projects.
- Ensure manifest file casing matches the filesystem exactly; Snyk operates on a case-sensitive filesystem.
- Review Snyk's fix PRs like any other dependency update; run the full test suite.

#### Constraints and Limitations

- Snyk does not support projects using Visual Studio Build Tools.
- Windows-specific frameworks (WPF, WCF) are supported only for .NET 10 and later.
- SCM integrations support NuGet as the only package manager for .NET.
- Static analysis can flag vulnerabilities that are already patched in your environment if the manifest does not reflect the resolved version.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CLI Scan of a .NET Project

```bash
dotnet restore
snyk test
```

**Expected Output (conceptual)**

```
✗ High severity vulnerability found in Newtonsoft.Json
  Description: Deserialization of Untrusted Data
  Info: https://security.snyk.io/vuln/SNYK-DOTNET-NEWTONSOFTJSON-...
  Introduced through: MyApp@1.0.0 > Newtonsoft.Json@12.0.1
  Fixed in: 13.0.3
```

**Why this output occurs**

- Snyk reads `project.assets.json` and matches resolved versions against its vulnerability database, showing the introduction path and the fix version.

---

#### Example 2: SCM Import Produces Per-Framework Projects

**Expected Output (conceptual)**

```
Snyk project: MyApp (net8.0) — 3 vulnerable dependencies
Snyk project: MyApp (netstandard2.0) — 1 vulnerable dependency
```

**Why this output occurs**

- Snyk creates a separate dependency list per target framework, isolating findings.

---

### Real-World Cases

- **Enterprise .NET portfolios** — Snyk's SCM integration scales across many repositories and private feeds.
- **Regulated environments** — Commercial support and policy controls supplement the advisory data.
- **Multi-targeted libraries** — Per-framework scanning reveals vulnerabilities that only affect specific targets.

---

### References

- SCM integrations for .NET — Snyk User Docs - https://docs.snyk.io/supported-languages/supported-languages-list/.net/scm-integrations-for-.net
- CLI support for .NET — Snyk User Docs - https://docs.snyk.io/supported-languages/supported-languages-list/.net/snyk-cli-for-.net
- Improved .NET scanning is now generally available — Snyk Updates - https://updates.snyk.io/improved-net-scanning-is-now-generally-available/
- How Snyk's VS Extension Scans .NET Solutions — Safeguard - https://safeguard.sh/resources/blog/snyk-vs-extension-dotnet

---

## 5. Dependency Security Best Practices

### Definitions

**Core Definition**
Dependency security best practices are empirically grounded guidelines for controlling what enters a .NET build, verifying that it is what was intended, continuously assessing it for known vulnerabilities, and remediating findings before they reach production.

**Technical Definition**
Best practices span **discovery** (audit direct and transitive packages), **source control** (package source mapping, central package management, cleared feeds), **reproducibility** (lock files and `--locked-mode`), **provenance** (signed packages, trusted publishing, SBOMs), **continuous assessment** (NuGet Audit in CI, scheduled `dotnet list package --vulnerable` reports, Dependabot or Snyk), **remediation discipline** (treat critical/high warnings as errors, track accepted advisories with expiry dates), and **build hygiene** (scoped API keys, trusted publishing instead of long-lived secrets).

**Beginner-Friendly Explanation**
Dependency security best practices are "the habits that keep your package supply chain trustworthy." They include: audit everything (not just what you reference directly), pin package names to the right feeds so nobody can substitute a package, lock versions so builds are reproducible, prefer signed packages, run scanning in CI so new CVEs fail the build, and keep your own publishing credentials short-lived. None of these alone is enough; together they make supply-chain attacks expensive and detectable.

---

### Purposes

- To make the dependency graph visible and auditable.
- To prevent dependency confusion and typosquatting.
- To make restores deterministic and tamper-evident.
- To detect known vulnerabilities continuously, not periodically.
- To remediate high-severity findings before they ship.
- To protect the organisation's own publishing identity.
- To produce evidence for compliance and audit.

---

### Guidelines and Structure

#### Audit Everything

```
✅ Set NuGetAuditMode to all; include transitive packages.
✅ Run dotnet list package --vulnerable --include-transitive in a scheduled CI job.
✅ Pair build-time warnings with an on-demand report for evidence.
```

#### Pin What You Resolve

```
✅ Use packageSourceMapping so internal names resolve only from private feeds.
✅ Commit a shared nuget.config with <clear /> at the root.
✅ Use central package management (Directory.Packages.props) for solution-wide pinning.
```

#### Make Restores Reproducible

```
✅ Enable RestorePackagesWithLockFile and commit packages.lock.json.
✅ Enforce dotnet restore --locked-mode in CI.
```

#### Verify Provenance

```
✅ Prefer signed packages and feeds that enforce signing.
✅ Generate a CycloneDX SBOM per shipped package.
✅ Use NuGet Trusted Publishing instead of long-lived API keys.
```

#### Assess Continuously

```
✅ Enable NuGet Audit and treat NU1903/NU1904 as errors in CI.
✅ Enable Dependabot alerts, security updates, and grouped version updates.
✅ Use Snyk or equivalent SCA for precise graphs and private feeds.
```

#### Remediate Deliberately

```
✅ Fix critical and high findings before shipping.
✅ Record accepted advisories with a reason and an expiry date so they are revisited.
✅ Do not suppress audit warnings without a documented, time-bounded risk acceptance.
```

#### Protect Your Own Publishing Identity

```
✅ Use short-lived credentials or Trusted Publishing (OIDC).
✅ Scope API keys to the narrowest package and permission set.
✅ Delete exposed keys immediately; never commit them or log them.
```

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complete CI Dependency Security Gate

```xml
<!-- Directory.Build.props -->
<Project>
  <PropertyGroup>
    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditMode>all</NuGetAuditMode>
    <NuGetAuditLevel>low</NuGetAuditLevel>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
  </PropertyGroup>
</Project>
```

```yaml
# CI step
- run: dotnet restore --locked-mode
- run: dotnet list package --vulnerable --include-transitive --format json > vulnerable.json
- uses: actions/upload-artifact@v4
  with:
    name: vulnerability-report
    path: vulnerable.json
```

**Expected Output (conceptual)**

```
Restore fails if the lock file drifts or a vulnerable package is resolved.
The vulnerability report is stored as a CI artifact for evidence.
```

**Why this output occurs**

- The lock file enforces reproducibility; NuGet Audit fails the build on vulnerabilities; the report provides an auditable record.

---

#### Example 2: Package Source Mapping + Lock File

```xml
<!-- nuget.config -->
<packageSourceMapping>
  <packageSource key="nuget.org">
    <package pattern="*" />
  </packageSource>
  <packageSource key="internal-feed">
    <package pattern="Contoso.*" />
  </packageSource>
</packageSourceMapping>
```

```bash
dotnet restore --locked-mode
```

**Expected Output (conceptual)**

```
Contoso.Billing resolves only from internal-feed.
Restore fails if packages.lock.json and the project disagree.
```

**Why this output occurs**

- Source mapping defeats dependency confusion; locked mode turns silent substitution into a loud failure.

---

### Real-World Cases

- **Enterprise .NET supply chains** — Private feed as the only allowed source, signed packages, central package management, and SBOM per release.
- **Open-source maintainers** — Dependabot alerts and security updates, plus Trusted Publishing to reduce credential risk.
- **Regulated industries** — Scheduled `dotnet list package --vulnerable` reports stored as compliance evidence.

---

### References

- .NET Security Best Practices (2026 Guide) — Safeguard - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- NuGet Supply Chain Security Guide (.NET, 2026) — Safeguard - https://safeguard.sh/resources/blog/nuget-supply-chain-security
- NuGet Audit: Stop Shipping Vulnerable .NET Packages — Coherent Solutions - https://www.coherentsolutions.com/insights/how-to-stop-shipping-solutions-with-vulnerable-dot-net-packages
- Package Source Mapping — Microsoft Learn - https://learn.microsoft.com/en-us/nuget/consume-packages/package-source-mapping
- Strengthening NuGet Supply Chain Security: Reducing API Key Lifetime — .NET Blog - https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/
- Securing the .NET Supply Chain: Best Practices for Managing NuGet Dependencies — Mirko Peters - https://www.m365.fm/community/u/mirko-p/blog/securing-the-net-supply-chain-best-practices-for-managing-nuget-dependencies/
- Dependabot Configuration & Management — GitHub Awesome Copilot - https://github.com/github/awesome-copilot/blob/main/skills/dependabot/SKILL.md
- Snyk CLI support for .NET — Snyk User Docs - https://docs.snyk.io/supported-languages/supported-languages-list/.net/snyk-cli-for-.net

---

## Supplementary: OWASP Dependency-Check

### Definitions

**Core Definition**
OWASP Dependency-Check is a free, open-source software composition analysis tool that identifies project dependencies and checks them against known vulnerability databases, including support for .NET assemblies and NuGet manifests.

**Technical Definition**
Dependency-Check analyses `.csproj` and `.vbproj` files via its MSBuild analyzer, `.dll` and `.exe` assemblies via its Assembly analyzer (which uses GrokAssembly and requires the .NET 8 runtime), `packages.config` via its NuGetConf analyzer, and `.nuspec` files via its Nuspec analyzer. It produces reports in HTML, JSON, XML, CSV, and SARIF formats. It is commonly used in air-gapped or on-premises environments where SaaS scanning is not permitted, and in CI pipelines alongside NuGet Audit and Dependabot.

**Beginner-Friendly Explanation**
OWASP Dependency-Check is "the free, run-it-yourself dependency scanner." It looks at your project files and compiled assemblies, matches them against vulnerability databases, and produces a report. It is useful when you cannot use a cloud service, or when you want a second scanner alongside the built-in .NET tooling.

---

### Purposes

- To provide a free, standalone SCA scanner for .NET and other ecosystems.
- To scan compiled assemblies when source manifests are not available.
- To run in air-gapped environments without SaaS dependencies.
- To produce SARIF output for integration with code-scanning dashboards.
- To complement NuGet Audit and Dependabot with an independent data source.

---

### Syntax Rules and Structure

#### General Syntax

```bash
# Install and run via the CLI (conceptual)
dependency-check --project MyApp --scan ./src --format HTML --out ./reports
```

**Component breakdown**

- `--project` — Names the report.
- `--scan` — Points at the directory containing project files or assemblies.
- `--format` — `HTML`, `JSON`, `XML`, `CSV`, or `SARIF`.
- `--out` — Output directory.

#### Supported .NET Inputs

| Analyzer | File Types |
|---|---|
| **MSBuild** | `*.csproj`, `*.vbproj` (including `Directory.Build.props` and `Directory.Packages.props`) |
| **Assembly** | `*.exe`, `*.dll` (requires .NET 8 runtime) |
| **NugetConf** | `packages.config` |
| **Nuspec** | `*.nuspec` |

*Source: OWASP Dependency-Check file type analyzers*.

#### Syntax Rules

- Run it alongside NuGet Audit; independent scanners may have different advisory coverage.
- Use SARIF output for integration with GitHub Code Scanning or Azure DevOps.
- For assembly scanning, ensure the .NET 8 runtime is available.
- Treat the report as one input to remediation, not as a complete risk assessment.

#### Constraints and Limitations

- It does not assess reachability; a vulnerable package is reported regardless of whether the vulnerable function is called.
- Assembly scanning depends on GrokAssembly and the .NET 8 runtime.
- Advisory coverage may lag the GitHub Advisory Database used by NuGet Audit.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Scanning a Solution

```bash
dependency-check --project MySolution --scan ./src --format SARIF --out ./reports
```

**Expected Output (conceptual)**

```
Report generated: ./reports/dependency-check-report.sarif
Findings: 3 vulnerable dependencies (1 high, 2 moderate)
```

**Why this output occurs**

- Dependency-Check parses the project files and assemblies, matches them against advisories, and emits a SARIF report.

---

### Real-World Cases

- **Air-gapped government environments** — No SaaS dependency scanning permitted; Dependency-Check runs locally.
- **Defence-in-depth scanning** — A second scanner alongside NuGet Audit catches advisories that one database may miss.
- **Compiled-artifact scanning** — When source manifests are unavailable, assembly scanning still identifies package versions.

---

### References

- File Type Analyzers — OWASP Dependency-Check - https://dependency-check.github.io/DependencyCheck/analyzers/
- .NET reachability with dosai — OWASP dep-scan - https://github.com/owasp-dep-scan/dep-scan/blob/main/documentation/docs/languages/dotnet-reachability.mdx

---

## Summary Table: Dependency Security Controls

| Control | What It Answers | Primary Tool / Mechanism |
|---|---|---|
| **NuGet Audit** | Are any resolved packages known-vulnerable? | `NuGetAudit`, `NuGetAuditMode`, NU1901–NU1904 |
| **`dotnet list package --vulnerable`** | What is the on-demand vulnerable inventory? | CLI report with `--include-transitive` and `--format json` |
| **Package source mapping** | Where is each package allowed to come from? | `packageSourceMapping` in `nuget.config` |
| **Lock files** | Is the restore reproducible? | `RestorePackagesWithLockFile` + `--locked-mode` |
| **Dependabot** | Has a CVE been published, and can it be fixed automatically? | Alerts + security updates + version updates |
| **Snyk** | What does a precise, SDK-resolved graph reveal? | CLI or SCM integration |
| **OWASP Dependency-Check** | What does an independent scanner find? | MSBuild, Assembly, NuGetConf, Nuspec analyzers |
| **SBOM** | What exactly shipped? | CycloneDX per package |
| **Trusted Publishing** | How are our own packages published safely? | OIDC-based short-lived credentials |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`NuGetAuditMode` default** | `direct` in .NET 8; `all` in .NET 9+ (transitive auditing) |
| **`dotnet list package --vulnerable` without `--include-transitive`** | Omits transitive packages; many advisories live in transitive dependencies |
| **Package source mapping** | Applies to restore/install/update, not to `dotnet list package` metadata lookups |
| **Long-lived NuGet API keys** | New keys limited to 30 days from August 17, 2026; all older keys expire November 1, 2026 |
| **`BinaryFormatter`** | Removed in .NET 9; a supply-chain vulnerability in its own right |
| **Dependabot without grouping** | Can produce excessive PR noise; use `groups` |
| **Snyk on Visual Studio Build Tools** | Not supported |
| **Snyk Windows-specific frameworks** | WPF/WCF only for .NET 10+ |
| **OWASP Dependency-Check assembly scanning** | Requires the .NET 8 runtime |

---

## References (Consolidated)

- NuGet Audit: Stop Shipping Vulnerable .NET Packages — Coherent Solutions - https://www.coherentsolutions.com/insights/how-to-stop-shipping-solutions-with-vulnerable-dot-net-packages
- .NET Security Best Practices (2026 Guide) — Safeguard - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- NuGet Supply Chain Security Guide (.NET, 2026) — Safeguard - https://safeguard.sh/resources/blog/nuget-supply-chain-security
- Package Source Mapping — Microsoft Learn - https://learn.microsoft.com/en-us/nuget/consume-packages/package-source-mapping
- SDK updates in .NET 9 Preview 6 (NuGetAudit transitive) — dotnet/core - https://github.com/dotnet/core/blob/main/release-notes/9.0/preview/preview6/sdk.md
- Strengthening NuGet Supply Chain Security: Reducing API Key Lifetime — .NET Blog - https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/
- Securing the .NET Supply Chain: Best Practices for Managing NuGet Dependencies — Mirko Peters - https://www.m365.fm/community/u/mirko-p/blog/securing-the-net-supply-chain-best-practices-for-managing-nuget-dependencies/
- dotnet list package vulnerable or NuGet Audit: which should I use? — Safeguard - https://safeguard.sh/resources/blog/dotnet-list-package-vulnerable-vs-nuget-audit-which-to-use
- Machine-readable output for dotnet list package — NuGet Home - https://github.com/nuget/home/blob/main/accepted/2020/MachineReadableOutputDotNetListPackage.md
- Upgrade app dependencies — Microsoft Learn - https://github.com/microsoftdocs/learn/blob/main/learn-pr/language/dotnet-dependencies/includes/5-exercise-dependency-management.md
- Dependabot Configuration & Management — GitHub Awesome Copilot - https://github.com/github/awesome-copilot/blob/main/skills/dependabot/SKILL.md
- Setting Up Dependency Scanning on GitHub — Safeguard - https://safeguard.sh/resources/blog/setting-up-dependency-scanning-github
- SCM integrations for .NET — Snyk User Docs - https://docs.snyk.io/supported-languages/supported-languages-list/.net/scm-integrations-for-.net
- CLI support for .NET — Snyk User Docs - https://docs.snyk.io/supported-languages/supported-languages-list/.net/snyk-cli-for-.net
- Improved .NET scanning is now generally available — Snyk Updates - https://updates.snyk.io/improved-net-scanning-is-now-generally-available/
- How Snyk's VS Extension Scans .NET Solutions — Safeguard - https://safeguard.sh/resources/blog/snyk-vs-extension-dotnet
- File Type Analyzers — OWASP Dependency-Check - https://dependency-check.github.io/DependencyCheck/analyzers/
- .NET reachability with dosai — OWASP dep-scan - https://github.com/owasp-dep-scan/dep-scan/blob/main/documentation/docs/languages/dotnet-reachability.mdx
- Pragmatic.Design supply-chain security — GitHub - https://github.com/pragmatic-design/Pragmatic.Design/blob/main/docs/security/supply-chain.md
- Minimum package publish age — Microsoft Learn - https://learn.microsoft.com/en-us/nuget/consume-packages/minimum-package-publish-age