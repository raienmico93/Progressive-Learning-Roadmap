# ASP.NET Core Test Automation: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **test automation** discipline — the practice of running automated tests in a CI/CD pipeline, reporting their results, measuring coverage, and maintaining suite health. It builds on the testing concepts (unit, integration, E2E) from earlier cheat sheets; here the focus is on **operationalising** those tests. Key ecosystem shifts: **Microsoft.Testing.Platform (MTP)** is replacing VSTest as the default runner in .NET 10; **Coverlet** is the standard coverage collector; **ReportGenerator** converts coverage data into human-readable reports; **Testcontainers** is the default for integration database tests; **Playwright** is the default for E2E. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Test automation is the practice of integrating automated tests into the software delivery pipeline so that they run consistently on every change, results are visible, and failures block progression — providing fast, reliable feedback that manual testing cannot match. |
| **Technical Definition** | Test automation encompasses **CI integration** (`dotnet test` executed by GitHub Actions or Azure Pipelines on every commit/PR), **test pipelines** (staged execution of unit → integration → E2E tests), **test reporting** (TRX/JUnit output published to the CI interface), **code coverage** (Coverlet collection + ReportGenerator reporting), and **suite health** (flaky test quarantine, retries, and quality gates). The tooling stack is `Microsoft.NET.Test.Sdk` + `xunit`/`NUnit`/`MSTest` + `Coverlet` + `ReportGenerator`, orchestrated by `dotnet test` inside CI YAML. |
| **Beginner-Friendly Explanation** | Test automation is the **“tests that run themselves, every time”** discipline. You write unit, integration, and E2E tests, and then you wire them into CI so that every pull request automatically runs them. If something breaks, the PR is blocked and the failure is visible in the CI interface. Coverage reports show what percentage of your code the tests exercise, and quarantine keeps flaky tests from poisoning the signal. Without automation, tests are just files that nobody runs. |

### Key Characteristics

- **Shift-left** – Catch problems as early as possible; a bug caught by a unit test in CI costs seconds, the same bug in production costs hours.
- **Staged execution** – Fast tests first (unit), then slower tests (integration), then slowest (E2E) — so feedback arrives quickly and slow tests do not block every commit.
- **Visible results** – Test results are published to the CI interface (GitHub Actions / Azure DevOps), not buried in logs.
- **Quality gates** – `dotnet format`, `dotnet build -warnaserror`, `dotnet test`, and vulnerability checks must pass before merge.
- **Coverage tracking** – Coverlet collects coverage; ReportGenerator produces HTML and Cobertura reports; thresholds can gate the pipeline.
- **Flaky test quarantine** – Known-flaky tests are excluded from regular CI and monitored separately, so they do not block every PR.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project with a test project.
- A CI platform: **GitHub Actions** or **Azure Pipelines**.
- `Microsoft.NET.Test.Sdk`, a test framework (`xunit`, NUnit, or MSTest), and `Coverlet.collector` in the test project.
- `dotnet-reportgenerator-globaltool` (optional, for readable coverage reports).
- For integration tests: **Testcontainers** (Docker) or SQLite in-memory.
- For E2E tests: **Playwright** or Selenium, with browser binaries installed in CI.

### Related Programming Areas

- **Unit testing** – The fast feedback layer; runs on every commit.
- **Integration testing** – Runs on PR merge or staging deployment.
- **E2E testing** – Runs after staging deployment, before production promotion.
- **Code coverage** – Measures how much code the tests exercise.
- **CI/CD pipelines** – The orchestration layer that runs everything.
- **Flaky test management** – Quarantine, retries, and suite maintenance.

### Core Concepts / Features

1. **CI integration** – Running `dotnet test` in GitHub Actions and Azure Pipelines.
2. **Test pipelines** – Staged test execution (unit → integration → E2E) with quality gates.
3. **Test reporting** – TRX, JUnit, and GitHub Actions/Azure DevOps test result publishing.
4. **Code coverage** – Coverlet collection and ReportGenerator reporting.
5. **Test automation best practices** – Shift-left, quarantine, quality gates, and suite maintenance.

---

## 1. CI Integration (Running Tests Automatically)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | CI integration is the practice of wiring automated tests into a continuous integration pipeline so that they run automatically on every commit or pull request, and their results are visible to the team. |
| **Technical Definition** | CI integration for .NET involves adding a `dotnet test` step (or `DotNetCoreCLI@2` task in Azure Pipelines) to the pipeline YAML, configuring it to collect coverage (`--collect:"XPlat Code Coverage"`), log results (`--logger "trx"`), and fail the build when tests fail (`failTaskOnFailedTests: true` or equivalent). The pipeline must restore, build, and test in sequence, with `--no-build` on the test step to avoid rebuilding. |
| **Beginner-Friendly Explanation** | CI integration is **“connecting the tests to the robot that runs them”** . You write a YAML file that tells GitHub Actions or Azure Pipelines: “When someone opens a pull request, restore the code, build it, and run `dotnet test`. If any test fails, block the merge.” The result is that no broken code reaches main without someone seeing the failure. |

### Purposes

- **To** run tests **automatically** on every commit and pull request, eliminating reliance on developers remembering to run them locally.
- **To** **block merges** when tests fail, preventing regressions from reaching the main branch.
- **To** provide **visible, actionable feedback** through the CI interface (test results, failure logs, coverage reports).
- **To** **shift-left** — catch problems in seconds during CI rather than hours in production.
- **To** enforce **consistent quality gates** —formatting, build warnings, tests, and vulnerability checks — on every change.

### Syntax Rules and Structure

#### GitHub Actions (`.github/workflows/ci.yml`)

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: '10.0.x'

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore --configuration Release -warnaserror

      - name: Test
        run: dotnet test --no-build --configuration Release --verbosity normal
          --collect:"XPlat Code Coverage"
          --logger "trx;LogFileName=test-results.trx"
          --results-directory ./artifacts/test-results

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: ./artifacts/test-results
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `actions/checkout@v4` | Checks out the repository. |
| `actions/setup-dotnet@v4` | Installs the .NET SDK. |
| `dotnet restore` | Restores NuGet packages. |
| `dotnet build --no-restore` | Builds the solution. |
| `dotnet test --no-build` | Runs tests without rebuilding. |
| `--collect:"XPlat Code Coverage"` | Enables Coverlet coverage collection. |
| `--logger "trx"` | Emits a TRX test result file. |

#### Azure Pipelines (`azure-pipelines.yml`)

```yaml
trigger:
  branches:
    include: [main, develop]

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetSdkVersion: '10.x'

stages:
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: BuildJob
        steps:
          - task: UseDotNet@2
            inputs:
              version: $(dotnetSdkVersion)

          - task: DotNetCoreCLI@2
            displayName: 'Restore packages'
            inputs:
              command: 'restore'

          - task: DotNetCoreCLI@2
            displayName: 'Build'
            inputs:
              command: 'build'
              arguments: '--no-restore --configuration $(buildConfiguration)'

          - task: DotNetCoreCLI@2
            displayName: 'Run unit tests'
            inputs:
              command: 'test'
              projects: '**/*UnitTests.csproj'
              arguments: >-
                --no-build --configuration $(buildConfiguration)
                --logger "trx;LogFileName=unit-test-results.trx"
                --collect:"XPlat Code Coverage"
                --results-directory $(Agent.TempDirectory)

          - task: PublishTestResults@2
            displayName: 'Publish unit test results'
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '$(Agent.TempDirectory)/**/*.trx'
              failTaskOnFailedTests: true
```

**Syntax rules:**

- **`dotnet test --no-build`** —build once, test without rebuilding to save time.
- **`-warnaserror`** on build treats warnings as errors, catching analyzer issues early.
- **`--collect:"XPlat Code Coverage"`** enables Coverlet coverage.
- **`--logger "trx"`** emits TRX results for CI publishing.
- **`failTaskOnFailedTests: true`** (Azure) or the implicit failure of `dotnet test` (GitHub Actions) blocks the pipeline on test failure.

**Constraints and limitations:**

- **`dotnet test` without a filter runs all tests** —including slow E2E and flaky quarantined tests. Use filters to stage execution.
- **Test results must be published** —running tests without publishing results means failures are invisible in the CI interface.
- **Windows-specific tests require a Windows runner** —`net8.0-windows` targets and WinForms/WPF tests need `windows-latest`.

### Annotated Complete Step-by-Step Code Example

```yaml
# GitHub Actions — complete build-and-test workflow
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Restore
        run: dotnet restore

      - name: Build with warnings as errors
        run: dotnet build --no-restore -c Release -warnaserror

      - name: Unit tests
        run: >
          dotnet test --no-build -c Release
          --filter "Category=Unit"
          --collect:"XPlat Code Coverage"
          --logger "trx;LogFileName=unit.trx"
          --results-directory ./TestResults

      - name: Publish results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: ./TestResults
```

**Step-by-step explanation:**

1. The workflow triggers on every push and pull request.
2. The SDK is installed, packages restored, and the solution built with warnings as errors.
3. Only unit tests (filtered by `Category=Unit`) run —fast feedback.
4. Coverage is collected and results are emitted as TRX.
5. Artifacts are uploaded even if tests fail, so failures can be investigated.

**Expected Output:** If any unit test fails, the workflow fails and the PR is blocked; test results and coverage artifacts are available for download.

**Why this output?** The pipeline enforces the quality gate: no merge until tests pass.

### Real-World Cases

- **Pull request gates:** Unit tests on every PR; integration tests on merge to main; E2E on staging deployment.
- **Multi-OS matrices:** Run tests on Windows, Linux, and macOS for cross-platform libraries.
- **Nightly runs:** Run the full suite (including E2E and quarantined tests) on a schedule to catch regressions that fast CI misses.

### References

- Build, test, and deploy .NET Core projects (Azure Pipelines) – https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/dotnet-core
- GitHub Actions CI/CD Pipeline for ASP.NET Core – https://www.toolliyo.com/blogs/github-actions-cicd-pipeline-for-aspnet-core-real-yaml-walkthrough-mpyb2z8m
- ci-cd-and-automation Agent Skill – https://skillsmp.com/creators/peterblazejewicz/claude-plugins/plugins-dotnet-skills-skills-ci-cd-and-automation

---

## 2. Test Pipelines (Staged Execution)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A test pipeline is the staged arrangement of test types —unit, integration, and E2E —so that fast, cheap tests run first and slow, expensive tests run later, providing quick feedback without blocking on slow suites. |
| **Technical Definition** | A test pipeline uses **test filters** (NUnit `Category`, xUnit `Trait`, MSTest `TestCategory`) or separate test projects to partition the suite. Unit tests run on every commit (`< 2 minutes`), integration tests on PR merge (`< 10 minutes`), and E2E tests after staging deployment (`< 20 minutes`). Each stage publishes its own results and coverage, and failure at any stage blocks progression to the next. |
| **Beginner-Friendly Explanation** | A test pipeline is the **“fast lane first”** rule. Unit tests are fast (milliseconds), so they run on every commit. Integration tests are slower (seconds), so they run when code is merged to main. E2E tests are slowest (minutes), so they run only after staging deployment. This way, developers get quick feedback on obvious bugs without waiting for the full browser suite. |

### Purposes

- **To** provide **fast feedback** —unit tests catch most bugs in under two minutes.
- **To** **avoid blocking** every commit on slow E2E tests.
- **To** **balance speed and confidence** —fast tests cover logic; slow tests cover integration.
- **To** **isolate failures** —a unit test failure points to logic; an E2E failure points to a full-stack integration issue.
- **To** **optimise CI cost** —slow tests consume more CI minutes; stage them carefully.

### Syntax Rules and Structure

#### Test Type Stages (from Azure Pipelines guidance)

| Test type | When to run | Duration target | Blocks |
|-----------|-------------|-----------------|--------|
| **Unit tests** | Every commit, every PR | `< 2 minutes` | Build stage |
| **Integration tests** | Every PR merge to main | `< 10 minutes` | Deploy to Dev stage |
| **End-to-end tests** | After staging deployment | `< 20 minutes` | Staging → Prod promotion |
| **Load tests** | Scheduled or manual | Variable | Production gating (optional) |

#### General Syntax (Filtering Tests by Category)

```bash
# Run only unit tests (NUnit)
dotnet test --filter "TestCategory=Unit"

# Run only integration tests (xUnit)
dotnet test --filter "Category=Integration"

# Exclude quarantined tests
dotnet test --filter-not-trait "quarantined=true"
```

**Component breakdown:**

| Filter | Framework | Purpose |
|--------|-----------|---------|
| `TestCategory=Unit` | NUnit | Selects tests marked `[Category("Unit")]`. |
| `Category=Integration` | xUnit | Selects tests marked `[Trait("Category", "Integration")]`. |
| `quarantined=true` | xUnit | Excludes flaky tests marked with a quarantine trait. |

**Syntax rules:**

- **Tag tests with categories** (`[Category]`, `[Trait]`, `[TestCategory]`) so the pipeline can filter them.
- **Run unit tests on every commit** —they are the fast feedback layer.
- **Run integration tests on merge to main** —they verify the HTTP pipeline and database.
- **Run E2E tests after staging deployment** —they verify the full user journey in a browser.

**Constraints and limitations:**

- **Running all tests on every commit is slow** —a suite with E2E tests can take 20+ minutes, frustrating developers.
- **Integration tests against a real database require infrastructure** —Testcontainers or a provisioned database.
- **E2E tests against a shared staging environment can conflict** —use unique test data and isolation.

### Annotated Complete Step-by-Step Code Example

```yaml
# Azure Pipelines — staged test execution
stages:
  - stage: UnitTests
    jobs:
      - job: RunUnitTests
        steps:
          - script: dotnet test --filter "TestCategory=Unit" --collect:"XPlat Code Coverage"
            displayName: 'Unit tests (fast feedback)'

  - stage: IntegrationTests
    dependsOn: UnitTests
    condition: succeeded()
    jobs:
      - job: RunIntegrationTests
        steps:
          - script: dotnet test --filter "Category=Integration"
            displayName: 'Integration tests (PR merge)'

  - stage: E2ETests
    dependsOn: IntegrationTests
    condition: succeeded()
    jobs:
      - job: RunE2ETests
        steps:
          - script: dotnet test --filter "Category=E2E"
            displayName: 'E2E tests (staging deployment)'
```

**Step-by-step explanation:**

1. Unit tests run first and must pass before integration tests start.
2. Integration tests run only if unit tests succeed.
3. E2E tests run only if integration tests succeed.

**Expected Output:** A failure at any stage blocks subsequent stages, preventing slow tests from running unnecessarily when fast tests already found a problem.

**Why this output?** Staged execution saves time and CI cost by failing fast.

### Real-World Cases

- **Web APIs:** Unit tests on every PR; integration tests on merge; E2E on staging.
- **SPAs:** Unit tests for components; integration tests for API calls; E2E for login and checkout.
- **Microservices:** Unit tests per service; integration tests for service boundaries; E2E for cross-service workflows.

### References

- Automated Testing in Azure Pipelines (staged execution) – https://cloudwebschool.com/docs/azure/devops-and-cicd/automated-testing-in-azure-pipelines/
- Testing Standards — C# / xUnit (unit/in-memory vs TestContainers/E2E) – https://raw.githubusercontent.com/…/testing-standards.md

---

## 3. Test Reporting (Making Results Visible)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Test reporting is the practice of publishing test results —pass/fail counts, failure messages, durations, and attachments —to the CI interface so that failures are visible and actionable. |
| **Technical Definition** | Test reporting involves emitting **TRX** (Visual Studio Test Results) or **JUnit XML** files from `dotnet test` (`--logger "trx"` or a JUnit logger package) and publishing them to the CI platform via `PublishTestResults@2` (Azure Pipelines) or an artifact upload + test reporter action (GitHub Actions). **Microsoft.Testing.Platform** (MTP) in .NET 10 adds native `--report-gh` and `--report-azdo` flags for direct CI integration. |
| **Beginner-Friendly Explanation** | Test reporting is **“making the test results visible in the CI dashboard”** . When a test fails, you should not have to dig through 500 lines of console output —the CI interface should show a red X next to the failing test, with the failure message and stack trace. TRX and JUnit are the two standard formats that CI platforms understand. |

### Purposes

- **To** make **failures visible** —the CI interface shows exactly which test failed and why.
- **To** **track test history** —pass/fail trends over time, flaky test detection.
- **To** **publish coverage** —coverage reports alongside test results.
- **To** **attach artefacts** —screenshots, logs, and traces from failed E2E tests.
- **To** **integrate with the PR** —failures appear as PR checks, blocking merge.

### Syntax Rules and Structure

#### Emitting TRX (xUnit / NUnit / MSTest)

```bash
dotnet test --logger "trx;LogFileName=test-results.trx" \
            --results-directory ./TestResults
```

#### Emitting JUnit (via logger package)

```xml
<PackageReference Include="JunitXml.TestLogger" Version="*" />
```

```bash
dotnet test --logger "junit;LogFilePath=test-results.xml"
```

#### Azure Pipelines (PublishTestResults@2)

```yaml
- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'VSTest'
    testResultsFiles: '$(Agent.TempDirectory)/**/*.trx'
    mergeTestResults: true
    failTaskOnFailedTests: true
    testRunTitle: 'Unit Tests - $(Build.BuildNumber)'
```

#### GitHub Actions (Artifact Upload + Test Reporter)

```yaml
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-results
    path: ./TestResults
```

**Syntax rules:**

- **Always publish results with `if: always()`** —results are most valuable when tests fail.
- **Use `failTaskOnFailedTests: true`** in Azure Pipelines to block the pipeline on failure.
- **TRX is the native .NET format**; JUnit is more portable across CI platforms.
- **MTP (`.NET 10`) adds `--report-gh` and `--report-azdo`** for direct CI integration without manual publishing.

**Constraints and limitations:**

- **GitHub Actions does not have a built-in TRX viewer** —use an action like `dorny/test-reporter` or `EnricoMi/publish-unit-test-result-action` to render results.
- **TRX files can be large** —publish them as artifacts rather than inlining them in logs.

### Annotated Complete Step-by-Step Code Example

```yaml
# Azure Pipelines — full test reporting
- task: DotNetCoreCLI@2
  displayName: 'Run tests with TRX and coverage'
  inputs:
    command: 'test'
    arguments: >
      --configuration Release --no-build
      --logger "trx;LogFileName=test-results.trx"
      --collect:"XPlat Code Coverage"
      --results-directory $(Agent.TempDirectory)

- task: PublishTestResults@2
  displayName: 'Publish test results'
  condition: succeededOrFailed()
  inputs:
    testResultsFormat: 'VSTest'
    testResultsFiles: '$(Agent.TempDirectory)/**/*.trx'
    failTaskOnFailedTests: true
    testRunTitle: 'Tests - $(Build.BuildNumber)'
```

**Expected Output:** Test results appear in the Azure Pipelines “Tests” tab; the pipeline fails if any test fails.

**Why this output?** `PublishTestResults@2` parses the TRX files and surfaces them in the CI interface; `failTaskOnFailedTests` enforces the gate.

### Real-World Cases

- **PR checks:** Test results appear as required status checks; merge is blocked on failure.
- **Flaky test detection:** Tracking test durations and pass rates over time reveals flaky tests.
- **E2E failure debugging:** Screenshots and traces attached as artefacts to the test result.

### References

- Reporting test results (TRX, JUnit, GitHubActionsTestLogger) – https://github.com/…/reporting-test-results.md
- Microsoft.Testing.Platform CI Reports (`--report-gh`, `--report-azdo`) – https://en.ittrip.xyz/
- TestResults library (TRX/JUnit serialisation) – https://github.com/demaconsulting/TestResults

---

## 4. Code Coverage (Measuring What Tests Exercise)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Code coverage is the metric that measures the proportion of application code executed by the test suite —lines, branches, or methods. |
| **Technical Definition** | In .NET, coverage is collected by **Coverlet** (`coverlet.collector` NuGet package, `--collect:"XPlat Code Coverage"` on `dotnet test`), which produces **Cobertura** XML. **ReportGenerator** (`dotnet-reportgenerator-globaltool`) converts Cobertura output into HTML reports, Markdown summaries, and CI-friendly formats. Coverage can be enforced as a **quality gate** (e.g., fail if line coverage drops below 60%). |
| **Beginner-Friendly Explanation** | Code coverage is the **“what percentage of my code did the tests touch?”** metric. If you have 1,000 lines of code and the tests execute 700 of them, line coverage is 70%. It is not a measure of test quality —a test that asserts nothing can still cover code —but it helps identify untested areas. Coverlet collects the data; ReportGenerator makes it readable; the CI pipeline can fail if coverage drops too low. |

### Purposes

- **To** **identify untested code** —areas with low coverage are candidates for new tests.
- **To** **track coverage trends** —coverage should not decrease as the codebase grows.
- **To** **enforce a minimum threshold** —prevent coverage from silently eroding.
- **To** **visualise coverage** —HTML reports show exactly which lines are covered.
- **To** **exclude generated code and migrations** —`[ExcludeFromCodeCoverage]` keeps the metric meaningful.

### Syntax Rules and Structure

#### General Syntax (Collect Coverage)

```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura
```

Or using the collector:

```bash
dotnet test --collect:"XPlat Code Coverage"
```

#### General Syntax (Generate Report)

```bash
reportgenerator \
  "-reports:**/*.cobertura.xml" \
  "-targetdir:./CoverageReport" \
  "-reporttypes:HtmlInline_AzurePipelines;Cobertura"
```

#### General Syntax (Azure Pipelines)

```yaml
- task: reportgenerator@5
  inputs:
    reports: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    targetdir: '$(Build.SourcesDirectory)/CoverageReport'
    reporttypes: 'HtmlInline_AzurePipelines;Cobertura'

- task: PublishCodeCoverageResults@2
  inputs:
    summaryFileLocation: '$(Build.SourcesDirectory)/CoverageReport/Cobertura.xml'
    reportDirectory: '$(Build.SourcesDirectory)/CoverageReport'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `coverlet.collector` | NuGet package that collects coverage during `dotnet test`. |
| `--collect:"XPlat Code Coverage"` | Enables coverage collection. |
| `coverage.cobertura.xml` | The Cobertura-format coverage output. |
| `reportgenerator` | Converts Cobertura to HTML, Markdown, etc. |
| `PublishCodeCoverageResults@2` | Azure Pipelines task that surfaces coverage in the UI. |

**Syntax rules:**

- **Use `--collect:"XPlat Code Coverage"`** —it is the simplest way to collect coverage with `dotnet test`.
- **Use ReportGenerator** —raw Cobertura XML is not human-readable.
- **Publish coverage with `if: always()`** —coverage is valuable even when tests fail.
- **Set a coverage threshold** —start modest (e.g., 60%) and increase over time.

**Constraints and limitations:**

- **Coverage is not quality** —100% coverage with weak assertions proves nothing.
- **Line coverage vs branch coverage** —branch coverage is more meaningful but slower to collect.
- **Exclude generated code** —`[ExcludeFromCodeCoverage]` on migrations, generated DTOs, and `Program.cs` (if appropriate) keeps the metric focused.

### Annotated Complete Step-by-Step Code Example

```yaml
# GitHub Actions — coverage collection and reporting
- name: Test with coverage
  run: >
    dotnet test --no-build -c Release
    --collect:"XPlat Code Coverage"
    --results-directory ./TestResults

- name: Install ReportGenerator
  run: dotnet tool install --global dotnet-reportgenerator-globaltool

- name: Generate coverage report
  run: >
    reportgenerator
    "-reports:./TestResults/**/coverage.cobertura.xml"
    "-targetdir:./CoverageReport"
    "-reporttypes:Html;MarkdownSummaryGithub"

- name: Upload coverage report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: ./CoverageReport
```

**Expected Output:** An HTML coverage report and a Markdown summary are generated and uploaded as artefacts.

**Why this output?** Coverlet collects the data; ReportGenerator converts it to readable formats; the artefact makes it accessible after the CI run.

### Real-World Cases

- **Coverage gates:** Fail the pipeline if line coverage drops below a threshold.
- **Coverage trends:** Track coverage over time to ensure it does not erode.
- **Untested area identification:** Use the HTML report to find uncovered branches.

### References

- Code Coverage (Coverlet + ReportGenerator) – https://github.com/microsoft/fluentui-blazor/blob/…/testing-coverage.md
- Add code coverage reporting (CI integration) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/commit/…
- Test code coverage - CLI – https://github.com/…/test-code-coverage.md

---

## 5. Test Automation Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Test automation best practices are guidelines for building and operating a CI‑integrated test suite that is fast, reliable, and trustworthy —not a suite that is slow, flaky, and ignored. |
| **Technical Definition** | Best practices span **shift-left quality gates** (format → build → unit → integration → E2E), **staged execution** (fast tests first), **flaky test quarantine** (`[QuarantinedTest]` trait, separate CI schedule), **coverage tracking with thresholds**, **test result publishing** (TRX/JUnit to the CI interface), **artifact capture** (coverage reports, E2E screenshots), and **suite maintenance** (removing redundant tests, fixing flaky tests, keeping categories consistent). |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for test automation —they help you avoid the three classic CI disasters: **flaky tests** that block every PR, **invisible failures** that nobody sees, and **slow suites** that developers bypass. The antidotes are quarantine, visible reporting, and staged execution. |

### Purposes

- **To** keep CI **green** —flaky tests do not block every PR; real failures are visible.
- **To** keep feedback **fast** —unit tests on every commit; slow tests staged.
- **To** keep coverage **meaningful** —tracked, reported, and gated.
- **To** keep the suite **maintainable** —redundant tests removed; categories consistent.
- **To** enforce **quality gates** —no merge without passing format, build, and tests.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Shift left** | Run format check → build (warnings as errors) → unit → integration → E2E in that order. | Run E2E tests before unit tests —slow failures block fast feedback. |
| **Staged execution** | Unit tests `< 2 min` on every commit; integration `< 10 min` on merge; E2E `< 20 min` after staging. | Run the full suite (including E2E) on every commit. |
| **Flaky tests** | Mark with `[QuarantinedTest("reason")]`; exclude from regular CI (`--filter-not-trait "quarantined=true"`); run in a separate scheduled CI. | Ignore flaky tests; retry them endlessly in regular CI. |
| **Reporting** | Publish TRX/JUnit results to the CI interface with `if: always()`. | Run tests without publishing results —failures are invisible. |
| **Coverage** | Collect with Coverlet; report with ReportGenerator; enforce a modest threshold. | Chase 100% coverage with meaningless assertions. |
| **Artifacts** | Upload coverage reports, E2E screenshots, and traces on failure. | Discard artifacts —debugging E2E failures without them is painful. |
| **Quality gates** | `dotnet format --verify-no-changes`, `dotnet build -warnaserror`, `dotnet list package --vulnerable`. | Merge code that builds with warnings or has known vulnerable dependencies. |
| **Suite maintenance** | Remove redundant tests; fix flaky tests promptly; keep category tags consistent. | Let the suite grow without pruning —it becomes slow and brittle. |
| **MTP awareness** | Use `Microsoft.Testing.Platform` (`--report-gh`, `--report-azdo`) in .NET 10+ for native CI reporting. | Rely on legacy VSTest adapters without checking MTP compatibility. |

### Annotated Code Example (Best-Practice Quarantine Setup)

```csharp
// Flaky test marked for quarantine
[Fact]
[QuarantinedTest("https://github.com/org/repo/issues/123")]
public async Task FlakyE2ETest()
{
    // Known to fail intermittently due to timing
}
```

```yaml
# Regular CI — excludes quarantined tests
- name: Test (regular)
  run: dotnet test --filter-not-trait "quarantined=true"

# Quarantine CI — runs quarantined tests every 6 hours
on:
  schedule:
    - cron: '0 */6 * * *'
jobs:
  quarantine:
    steps:
      - run: dotnet test --filter-trait "quarantined=true"
```

**Why this follows best practices:**

- **Quarantine trait** isolates flaky tests so they do not block every PR.
- **Regular CI excludes them** —developers get reliable feedback.
- **Scheduled quarantine CI** monitors them —they are not forgotten.
- **The reason URL** links to the tracking issue, ensuring accountability.

### Real-World Cases

- **Large monorepos:** Quarantine keeps the main CI green while flaky tests are fixed on a separate schedule.
- **E2E-heavy suites:** Staged execution ensures E2E tests do not block unit feedback.
- **Coverage-gated libraries:** A minimum coverage threshold prevents erosion.

### References

- Quarantined Tests (Microsoft Aspire) – https://github.com/microsoft/aspire/blob/…/quarantined-tests.md
- Flaky tests in MSTest: every pattern that breaks CI – https://staging.mergify.com/learn/flaky-tests/mstest
- ci-cd-and-automation Agent Skill (quality gates) – https://skillsmp.com/creators/peterblazejewicz/claude-plugins/plugins-dotnet-skills-skills-ci-cd-and-automation

---

## Summary Table: Test Automation at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **CI integration** | Run tests automatically on every change | GitHub Actions `dotnet test`, Azure `DotNetCoreCLI@2` | Every project with automated tests |
| **Test pipelines** | Staged execution (unit → integration → E2E) | Test filters (`--filter`), categories, separate projects | Any suite with slow tests |
| **Test reporting** | Make failures visible in CI | TRX (`--logger "trx"`), JUnit, `PublishTestResults@2` | Every CI pipeline |
| **Code coverage** | Measure code exercised by tests | Coverlet, ReportGenerator, `--collect:"XPlat Code Coverage"` | When tracking test effectiveness |
| **Best practices** | Reliable, fast, trustworthy suite | Shift-left, quarantine, quality gates, artifact capture | Every production CI suite |

---

## References (Full List)

- Build, test, and deploy .NET Core projects (Azure Pipelines) – https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/dotnet-core
- Automated Testing in Azure Pipelines (staged execution, reporting) – https://cloudwebschool.com/docs/azure/devops-and-cicd/automated-testing-in-azure-pipelines/
- GitHub Actions CI/CD Pipeline for ASP.NET Core – https://www.toolliyo.com/blogs/github-actions-cicd-pipeline-for-aspnet-core-real-yaml-walkthrough-mpyb2z8m
- ci-cd-and-automation Agent Skill (quality gates) – https://skillsmp.com/creators/peterblazejewicz/claude-plugins/plugins-dotnet-skills-skills-ci-cd-and-automation
- dotnet-cicd-patterns (GitHub Actions + Azure DevOps examples) – https://lobehub.com/skills/onesmartguy-next-level-real-estate-dotnet-cicd-patterns
- Reporting test results (TRX, JUnit, GitHubActionsTestLogger) – https://github.com/…/reporting-test-results.md
- Microsoft.Testing.Platform CI Reports (`--report-gh`, `--report-azdo`) – https://en.ittrip.xyz/
- TestResults library (TRX/JUnit serialisation) – https://github.com/demaconsulting/TestResults
- Code Coverage (Coverlet + ReportGenerator) – https://github.com/microsoft/fluentui-blazor/blob/…/testing-coverage.md
- Add code coverage reporting (CI integration) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/commit/…
- Test code coverage - CLI – https://github.com/…/test-code-coverage.md
- Quarantined Tests (Microsoft Aspire) – https://github.com/microsoft/aspire/blob/…/quarantined-tests.md
- Flaky tests in MSTest: every pattern that breaks CI – https://staging.mergify.com/learn/flaky-tests/mstest
- Testing Standards — C# / xUnit (unit/in-memory vs TestContainers/E2E) – https://raw.githubusercontent.com/…/testing-standards.md
- Testing (FastEndpoints CI commands and filters) – https://github.com/fastendpoints/fastendpoints/blob/…/testing.md