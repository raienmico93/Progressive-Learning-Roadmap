# Test Automation in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Test automation is the practice of executing software tests automatically, without human intervention, as part of a repeatable and reliable process that provides fast feedback on code quality.

**Technical Definition**
Test automation in .NET encompasses the orchestration of test execution through CI/CD pipelines, the configuration of parallel test execution for performance, the collection and publication of test results and code coverage metrics, and the enforcement of quality gates that prevent regressions from reaching production. It is built on the foundation of test frameworks (xUnit, NUnit, MSTest), runners (`dotnet test`, VSTest), coverage tools (Coverlet), reporting tools (TRX, JUnit XML, ReportGenerator), and pipeline platforms (GitHub Actions, Azure DevOps). Automation transforms testing from a manual, ad-hoc activity into a continuous, measurable, and enforceable engineering discipline .

**Beginner-Friendly Explanation**
Test automation is "letting the computer run your tests for you, every time you change code." Instead of manually running tests before a release, you set up a pipeline that builds your code, runs all the tests, checks how much code was covered, and reports the results — automatically. If something breaks, the pipeline fails and tells you before the bug reaches users.

---

### Key Characteristics

- **Continuous feedback** — Tests run on every push or pull request, catching regressions early.
- **Pipeline as code** — CI/CD workflows are defined in version-controlled YAML files.
- **Parallel execution** — Tests run concurrently to reduce feedback time.
- **Standardized reporting** — Results are published in machine-readable formats (TRX, JUnit XML) and human-readable dashboards.
- **Coverage tracking** — Code coverage is collected, reported, and used as a quality gate.
- **Fail-fast** — The pipeline stops on the first failure, preventing wasted compute.
- **Build once, deploy many** — Artifacts are built once and promoted through environments.
- **Never skip tests** — Tests gate the pipeline; no deployment without passing tests .

---

### Prerequisites

- A .NET solution with unit, integration, and possibly E2E test projects.
- Familiarity with `dotnet test`, `dotnet build`, and `dotnet restore`.
- A CI/CD platform account (GitHub Actions, Azure DevOps, GitLab CI, etc.).
- Basic understanding of YAML syntax for pipeline definitions.
- The Coverlet collector package for code coverage.
- Test reporting tools (`trx2junit`, ReportGenerator) as needed.

---

### Related Programming Areas

- **Unit testing** — The foundation of the automated test suite.
- **Integration testing** — Higher-fidelity tests that run in the pipeline after unit tests.
- **End-to-end testing** — Browser-driven tests that run against deployed environments.
- **Code coverage** — Measuring test effectiveness.
- **Mutation testing** — Evaluating the quality of the test suite.
- **DevOps** — The cultural and technical practices that make automation effective.

---

### Core Concepts / Features

1. CI Integration
2. Test Pipelines
3. Parallel Testing
4. Test Reporting
5. Code Coverage
6. Testing Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. CI Integration

### Definitions

**Core Definition**
CI (Continuous Integration) integration is the practice of automatically building and testing code changes in a shared repository on every commit or pull request.

**Technical Definition**
CI integration involves configuring a CI platform (GitHub Actions, Azure DevOps, GitLab CI, Jenkins) to execute a defined sequence of steps — restore, build, format check, test, coverage collection, and artifact publication — whenever code is pushed or a pull request is opened. The CI pipeline acts as a quality gate: if any step fails, the change is blocked from merging. Best practices include caching dependencies, running tests with `--no-build` after a build step, and never skipping tests .

**Beginner-Friendly Explanation**
CI integration is "the robot that checks your code every time you push." It builds your project, runs your tests, and tells you if anything is broken — before your teammates even review your code.

---

### Purposes

- To catch integration issues and regressions early.
- To enforce consistent quality standards across the team.
- To provide fast feedback on every code change.
- To automate repetitive build and test steps.
- To gate merges on passing tests.
- To produce versioned, deployable artifacts.

---

### Syntax Rules and Structure

#### General CI Workflow Stages

```
1. Checkout code
2. Setup .NET SDK
3. Restore NuGet packages
4. Build (Release configuration)
5. Format check (optional)
6. Run tests (with TRX logger and coverage)
7. Publish test results
8. Upload coverage reports
9. (Optional) Publish artifacts / deploy
```

#### GitHub Actions Example

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: '8.0.x'
  DOTNET_NOLOGO: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true

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
        run: dotnet build --no-restore --configuration Release

      - name: Test
        run: >
          dotnet test --no-build --configuration Release
          --logger "trx;LogFileName=test-results.trx"
          --results-directory TestResults
          --collect:"XPlat Code Coverage"

      - name: Publish test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: TestResults/*.trx

      - name: Publish coverage
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: '**/coverage.cobertura.xml'
```

**Component breakdown**

- `actions/checkout` — Checks out the repository.
- `actions/setup-dotnet` — Installs the specified .NET SDK version.
- `dotnet restore` — Restores NuGet dependencies (can be cached).
- `dotnet build --no-restore` — Builds without redundant restore.
- `dotnet test --no-build` — Runs tests against the built artifacts.
- `--logger "trx;..."` — Produces a TRX results file.
- `--collect:"XPlat Code Coverage"` — Enables Coverlet coverage collection.
- `actions/upload-artifact` — Uploads results and coverage for later inspection.

#### Syntax Rules

- Use `--no-build` in the test step to avoid rebuilding; build once and test the artifact.
- Use `if: always()` on publish steps so results are uploaded even when tests fail.
- Cache NuGet packages (`actions/cache`) to speed up restore .
- Pin SDK versions and tool versions for reproducibility.
- Never skip tests; they must gate the pipeline .

#### Constraints and Limitations

- CI minutes may be limited on free tiers; parallelization and caching help.
- In-memory databases may not reflect production behavior; integration tests with Testcontainers add time.
- Secrets must be managed securely (e.g., GitHub Secrets, Azure Key Vault).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: GitHub Actions with Testcontainers

```yaml
services:
  postgres:
    image: postgres:16
    env:
      POSTGRES_DB: testdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - 5432:5432
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5

steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-dotnet@v4
    with:
      dotnet-version: '8.0.x'
  - run: dotnet restore
  - run: dotnet build --no-restore -c Release
  - run: dotnet test --no-build -c Release --collect:"XPlat Code Coverage"
    env:
      ConnectionStrings__Default: "Host=localhost;Database=testdb;Username=postgres;Password=postgres"
```

**Expected Output (conceptual)**

```
Build succeeded.
Test Run Successful. Total tests: 42, Passed: 42.
Coverage collected.
```

**Why this output occurs**

- The PostgreSQL service container is healthy before tests run.
- The connection string environment variable is injected into the test process.
- Integration tests connect to the real database.

---

### Real-World Cases

- **Open-source projects** — GitHub Actions runs tests on every PR.
- **Enterprise .NET** — Azure DevOps pipelines gate merges on test results.
- **Microservices** — Each service has its own CI pipeline with unit and integration tests.

---

### References

- .NET CI/CD with GitHub Actions — Microsoft - https://learn.microsoft.com/en-us/dotnet/devops/github-actions-overview
- CI/CD for .NET — Microsoft - https://learn.microsoft.com/en-us/dotnet/devops/
- .NET CI/CD patterns (community) - https://skillhub.360.com/skills/codewithmukesh/dotnet-claude-kit/ci-cd

---

## 2. Test Pipelines

### Definitions

**Core Definition**
A test pipeline is the ordered sequence of test stages — unit, integration, and end-to-end — that execute within a CI/CD workflow, each stage building on the previous one.

**Technical Definition**
A test pipeline implements the test pyramid in automation: fast unit tests run first on every build; slower integration tests (often with Testcontainers) run after unit tests pass; and E2E tests run against deployed environments on merge or release. Each stage has its own duration target, blocking behavior, and reporting requirements. Pipelines are defined as code (YAML) and versioned alongside the application .

**Beginner-Friendly Explanation**
A test pipeline is "the assembly line of testing." First, the fast unit tests run. If they pass, the integration tests run. If those pass, the end-to-end tests run. Each stage catches different kinds of problems, and the slowest stages run last so you get fast feedback.

---

### Purposes

- To provide layered feedback at the appropriate speed and cost.
- To fail fast on cheap tests before running expensive tests.
- To separate unit, integration, and E2E concerns.
- To control resource usage and pipeline duration.
- To align testing effort with the test pyramid.
- To enable selective test execution (e.g., unit tests on every push, E2E on merge).

---

### Syntax Rules and Structure

#### Test Stage Strategy

| Stage | Tests | When to Run | Duration Target | Blocks |
|---|---|---|---|---|
| **Build** | Unit / in-memory | Every push | Seconds | All later stages |
| **Test** | Integration (Testcontainers) | After build passes | Minutes | Merge/PR gate |
| **E2E** | Playwright / Selenium | After staging deployment | < 20 minutes | Production promotion |

*Sources: testing standards and pipeline guidance* .

#### General Pipeline Syntax (GitHub Actions)

```yaml
jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
      - run: dotnet test --filter "Category=Unit" --logger trx

  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
      - run: dotnet test --filter "Category=Integration" --logger trx

  e2e-tests:
    needs: integration-tests
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy to staging and run Playwright"
```

**Component breakdown**

- `needs` — Declares stage dependencies; later stages wait for earlier ones.
- `--filter "Category=Unit"` — Runs only tests with the matching trait.
- `services` — Spins up real infrastructure for integration tests.

#### Syntax Rules

- Unit tests run on every push; integration tests on every PR; E2E on merge or release.
- Use `needs` (GitHub Actions) or `dependsOn` (Azure DevOps) to enforce ordering.
- Publish results from each stage for visibility.
- Do not run E2E tests on every commit; they are too slow and brittle.

#### Constraints and Limitations

- Pipeline complexity grows with the number of stages.
- Integration and E2E stages consume more CI minutes.
- Flaky E2E tests can block releases; quarantine and fix them promptly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multi-Stage Pipeline

```yaml
jobs:
  build:
    steps:
      - run: dotnet build -c Release

  unit:
    needs: build
    steps:
      - run: dotnet test --no-build -c Release --filter "Category=Unit"

  integration:
    needs: unit
    services:
      postgres: { image: postgres:16 }
    steps:
      - run: dotnet test --no-build -c Release --filter "Category=Integration"

  e2e:
    needs: integration
    steps:
      - run: echo "Run Playwright against staging"
```

**Expected Output (conceptual)**

```
build: success
unit: 250 passed
integration: 18 passed
e2e: 5 passed
```

**Why this output occurs**

- Each stage depends on the previous one.
- Unit tests run first and are the fastest.
- Integration and E2E tests run only after earlier stages pass.

---

### Real-World Cases

- **Monorepos** — Per-project pipelines with shared templates.
- **Microservices** — Each service has its own layered pipeline.
- **Mobile apps** — Unit tests on every commit; UI tests on nightly builds.

---

### References

- Testing standards — xUnit / CI gates (community) - https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/main/.claude/rules/testing.md
- CI/CD pipelines for .NET (community) - https://skillhub.360.com/skills/codewithmukesh/dotnet-claude-kit/ci-cd
- Automated testing in Azure Pipelines (community) - https://cloudwebschool.com/automated-testing-azure-pipelines

---

## 3. Parallel Testing

### Definitions

**Core Definition**
Parallel testing is the execution of multiple tests or test assemblies concurrently to reduce total test execution time.

**Technical Definition**
Parallel testing in .NET operates at three levels: solution level (test projects run in parallel), project level (framework-specific settings control test parallelism within an assembly), and multi-targeted projects (target frameworks run in parallel in .NET 9+). xUnit runs test collections in parallel by default; NUnit requires explicit opt-in via `[Parallelizable]`; MSTest has configurable parallelization. Parallelism must be managed carefully to avoid shared-state conflicts .

**Beginner-Friendly Explanation**
Parallel testing is "running many tests at the same time instead of one by one." If you have 100 tests that each take 1 second, running them sequentially takes 100 seconds; running 10 in parallel takes about 10 seconds. But tests that share a database or a file must not run at the same time, so you need to control parallelism.

---

### Purposes

- To reduce total test execution time.
- To provide faster feedback in CI pipelines.
- To utilize multi-core CI agents efficiently.
- To scale test suites as they grow.
- To separate fast unit tests (parallel) from slow integration tests (sequential) .

---

### Syntax Rules and Structure

#### Solution-Level Control

```bash
# Limit parallel test assemblies
dotnet test -p:MaxCpuCount=1

# Disable multi-TFM parallelism
dotnet test -p:TestTfmsInParallel=false
```

#### xUnit Parallelism

```json
// xunit.runner.json
{
  "$schema": "https://xunit.net/schema/current/xunit.runner.schema.json",
  "parallelizeAssembly": false,
  "parallelizeTestCollections": true,
  "maxParallelThreads": 4
}
```

```csharp
// Disable parallelization for a collection
[CollectionDefinition("Database", DisableParallelization = true)]
public class DatabaseCollection { }
```

**Component breakdown**

- `parallelizeTestCollections` — Runs test collections in parallel (default `true`).
- `parallelizeAssembly` — Runs test assemblies in parallel (default `true`).
- `maxParallelThreads` — Limits concurrency; `0` uses processor count, `-1` is unlimited .
- `DisableParallelization` — Opts a collection out of parallel execution.

#### NUnit Parallelism

```csharp
// Assembly-level opt-in
[assembly: Parallelizable(ParallelScope.Fixtures)]
[assembly: LevelOfParallelism(4)]

// Fixture-level
[Parallelizable(ParallelScope.All)]
public class OrderTests { ... }

// Exclude a specific test
[NonParallelizable]
public void SharedResourceTest() { ... }
```

**Component breakdown**

- `ParallelScope.Self` — The test itself runs in parallel with other tests.
- `ParallelScope.Children` — Child tests run in parallel with each other.
- `ParallelScope.Fixtures` — Fixtures run in parallel with each other.
- `LevelOfParallelism` — Sets the number of worker threads (default: processor count, minimum 2) .

#### Syntax Rules

- xUnit parallelizes collections, not individual tests within a collection.
- NUnit requires explicit `[Parallelizable]`; nothing runs in parallel by default.
- Use `[Collection]` (xUnit) or `[NonParallelizable]` (NUnit) to protect shared resources.
- Integration tests with shared databases should run sequentially; unit tests can run in parallel .

#### Constraints and Limitations

- Shared mutable state (static fields, singletons, shared fixtures) causes intermittent failures.
- Parallelism increases memory and CPU usage.
- Test frameworks differ in default behavior and configuration APIs.
- Debugging parallel failures is harder than sequential failures.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: xUnit Collection-Based Control

```csharp
[CollectionDefinition("Database", DisableParallelization = true)]
public class DatabaseCollection { }

[Collection("Database")]
public class UserRepositoryTests { ... }

[Collection("Database")]
public class OrderRepositoryTests { ... }

// These collections run in parallel with each other
public class UnitTests1 { ... }
public class UnitTests2 { ... }
```

**Expected Output**

```
UnitTests1 and UnitTests2 run in parallel.
UserRepositoryTests and OrderRepositoryTests run sequentially (same collection).
```

**Why this output occurs**

- Tests in the same collection run sequentially to protect the shared database.
- Different collections and uncollected tests run in parallel.

---

#### Example 2: NUnit Parallelizable

```csharp
[assembly: Parallelizable(ParallelScope.Fixtures)]
[assembly: LevelOfParallelism(4)]

[Parallelizable(ParallelScope.All)]
public class CalculatorTests
{
    [Test] public void Add_Works() { ... }
    [Test] public void Subtract_Works() { ... }
}
```

**Expected Output**

```
CalculatorTests fixtures run in parallel; tests within the fixture run in parallel.
```

**Why this output occurs**

- `ParallelScope.Fixtures` enables fixture-level parallelism.
- `ParallelScope.All` enables test-level parallelism within the fixture.
- `LevelOfParallelism(4)` limits to four worker threads.

---

### Real-World Cases

- **Large test suites** — Parallel unit tests reduce CI time from minutes to seconds.
- **Multi-project solutions** — Test projects run in parallel at the solution level.
- **Integration tests** — Sequential execution for shared databases; parallel for isolated containers.

---

### References

- Parallel test execution control (community) - https://github.com/nikiforovall/claude-code-rules/blob/main/plugins/handbook-dotnet/skills/dotnet-test/references/parallel-execution.md
- Running tests in parallel — NUnit - https://docs.nunit.org/articles/nunit/writing-tests/parallel-tests.html
- xUnit MaxParallelThreads API - https://api.xunit.net/
- xUnit parallelization (community) - https://github.com/nikiforovall/claude-code-rules/blob/main/plugins/handbook-dotnet/skills/dotnet-test/references/parallel-execution.md

---

## 4. Test Reporting

### Definitions

**Core Definition**
Test reporting is the collection, formatting, and publication of test execution results so that humans and machines can understand which tests passed, failed, or were skipped.

**Technical Definition**
`dotnet test` produces results in TRX (Visual Studio Test Results) format by default when the `--logger "trx"` option is used. Many CI platforms (GitLab, CircleCI, Harness) expect JUnit XML format, requiring conversion via `trx2junit` or native JUnit loggers. Reports include test names, outcomes, durations, error messages, and stack traces. Reporting integrates with CI dashboards, pull request annotations, and artifact storage .

**Beginner-Friendly Explanation**
Test reporting is "the scoreboard for your tests." It tells you which tests passed, which failed, how long they took, and why they failed. In CI, reports are published so the whole team can see the results.

---

### Purposes

- To provide visibility into test outcomes.
- To enable CI platforms to annotate pull requests with failures.
- To track test duration and flakiness over time.
- To produce artifacts for auditing and debugging.
- To support machine-readable formats for dashboards.
- To fail the pipeline when tests fail.

---

### Syntax Rules and Structure

#### TRX Logger (Native)

```bash
dotnet test --logger "trx;LogFileName=test-results.trx" --results-directory TestResults
```

#### JUnit Logger (Native, if supported)

```bash
dotnet test --logger "junit;LogFilePath=test-results.xml"
```

#### TRX to JUnit Conversion

```bash
dotnet tool install -g trx2junit
trx2junit TestResults/*.trx
```

#### GitHub Actions Upload

```yaml
- name: Publish test results
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-results
    path: TestResults/*.trx
```

**Component breakdown**

- `--logger "trx;..."` — Generates TRX output.
- `--logger "junit;..."` — Generates JUnit XML output (if the logger is available).
- `trx2junit` — Converts TRX to JUnit XML for platforms that require it.
- `upload-artifact` — Stores results as pipeline artifacts.

#### Syntax Rules

- Always publish results with `if: always()` so failures are visible.
- Use TRX for Visual Studio and Azure DevOps; use JUnit XML for GitLab, CircleCI, and Harness.
- Convert TRX to JUnit if the CI platform does not natively parse TRX .
- Store results as artifacts for later inspection.

#### Constraints and Limitations

- TRX is not JUnit XML; conversion is required for some platforms .
- Large result files may slow artifact uploads.
- Test names and stack traces may expose sensitive information; sanitize if necessary.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multi-Format Logging

```bash
dotnet test \
  --logger "trx;LogFileName=results.trx" \
  --logger "junit;LogFilePath=results.xml" \
  --results-directory TestResults
```

**Expected Output (conceptual)**

```
TestResults/results.trx
TestResults/results.xml (JUnit XML)
```

**Why this output occurs**

- Two loggers are configured; each produces its own file format.
- The results directory contains both formats.

---

### Real-World Cases

- **GitLab CI** — Requires JUnit XML; convert TRX with `trx2junit` .
- **Azure DevOps** — Natively parses TRX with the `PublishTestResults` task.
- **Harness CI** — Accepts both TRX and JUnit XML; TRX is automatically converted .

---

### References

- TRX to JUnit conversion (community) - https://github.com/thomhurst/TUnit/pull/5019
- trx2junit NuGet package - https://www.nuget.org/packages/trx2junit
- Test reporting formats (community) - https://github.com/thomhurst/TUnit/pull/5019

---

## 5. Code Coverage

### Definitions

**Core Definition**
Code coverage is a metric that measures the proportion of production code executed by the test suite.

**Technical Definition**
Code coverage is collected by Coverlet, the default coverage collector for .NET, which produces Cobertura-format XML reports. `dotnet test --collect:"XPlat Code Coverage"` gathers coverage data during test execution. ReportGenerator converts Cobertura XML into human-readable HTML reports with line, branch, and method coverage. Coverage is typically used as a CI quality gate with a threshold (e.g., 80% line coverage) .

**Beginner-Friendly Explanation**
Code coverage tells you "how much of your code was actually run by your tests." If coverage is 80%, then 80% of your code was executed. But coverage does not tell you whether the tests checked the right things — it only tells you what was executed.

---

### Purposes

- To identify untested code paths.
- To provide an objective metric for CI quality gates.
- To guide test-writing priorities.
- To highlight dead or unreachable code.
- To track testing progress over time.
- To complement mutation testing for suite quality.

---

### Syntax Rules and Structure

#### Collecting Coverage

```bash
dotnet test --collect:"XPlat Code Coverage"
```

This produces `coverage.cobertura.xml` in the `TestResults` directory .

#### Generating HTML Reports

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool

reportgenerator \
  -reports:"TestResults/**/coverage.cobertura.xml" \
  -targetdir:"CoverageReport" \
  -reporttypes:Html
```

#### CI Integration (GitHub Actions)

```yaml
- name: Test with coverage
  run: dotnet test --collect:"XPlat Code Coverage" --results-directory TestResults

- name: Generate coverage report
  run: |
    dotnet tool install -g dotnet-reportgenerator-globaltool
    reportgenerator -reports:"TestResults/**/coverage.cobertura.xml" -targetdir:"CoverageReport" -reporttypes:Html

- name: Upload coverage
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: CoverageReport/
```

**Component breakdown**

- `--collect:"XPlat Code Coverage"` — Enables Coverlet collection.
- `coverage.cobertura.xml` — The coverage data file.
- `reportgenerator` — Converts Cobertura XML to HTML, Cobertura, SonarQube, and other formats.
- `-reporttypes:Html` — Generates an interactive HTML report.

#### Syntax Rules

- Add `coverlet.collector` to the test project (included by default in new xUnit/MSTest templates).
- Use `--results-directory` to control where coverage files are written.
- Set coverage thresholds in CI to fail builds that drop below the target (e.g., 80% line coverage) .
- Focus coverage on domain logic, validation, auth, and failure paths — not glue code .

#### Constraints and Limitations

- Coverage measures execution, not verification; 100% coverage does not guarantee correct tests.
- Generated code, DTOs, and migrations can inflate coverage numbers; exclude them.
- Branch coverage is more meaningful than line coverage but requires additional tooling.
- Coverage collection slows down test execution slightly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Coverage Collection and Reporting

```bash
dotnet test --collect:"XPlat Code Coverage" --results-directory TestResults

reportgenerator \
  -reports:"TestResults/**/coverage.cobertura.xml" \
  -targetdir:"CoverageReport" \
  -reporttypes:"Html;lcov"
```

**Expected Output (conceptual)**

```
Coverage Report:
  Line coverage:   85%
  Branch coverage: 72%
  Method coverage: 90%
```

**Why this output occurs**

- Coverlet executes the tests and records which lines were hit.
- ReportGenerator parses the Cobertura XML and computes the percentages.

---

### Real-World Cases

- **CI quality gates** — Fail builds if coverage drops below 80%.
- **Legacy code** — Identify untested areas before refactoring.
- **Compliance** — Demonstrate coverage to auditors.

---

### References

- Use code coverage for unit testing — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage
- Coverlet documentation - https://github.com/coverlet-coverage/coverlet
- ReportGenerator - https://github.com/danielpalme/ReportGenerator
- Coverage collection guide (community) - https://github.com/afonsoft/skills/blob/main/skills/quality-test-implementation/references/coverage-dotnet.md

---

## 6. Testing Best Practices

### Definitions

**Core Definition**
Testing best practices for automation are guidelines for building and maintaining a CI/CD test suite that is fast, reliable, and provides meaningful feedback.

**Technical Definition**
Best practices encompass the test pyramid (many unit tests, fewer integration tests, few E2E tests), fail-fast pipelines, build-once-deploy-many, never skipping tests, caching dependencies, publishing results consistently, setting realistic coverage thresholds, isolating tests for parallel execution, and treating pipeline configuration as code .

**Beginner-Friendly Explanation**
Best practices are "the habits that keep your automated tests helpful instead of slow and flaky." They include running fast tests first, not skipping tests to save time, and making sure your tests do not interfere with each other.

---

### Purposes

- To keep feedback fast and reliable.
- To maximize confidence per CI minute.
- To reduce flakiness and maintenance cost.
- To ensure tests gate deployments effectively.
- To align automation with the test pyramid.
- To make the pipeline a trusted source of truth.

---

### Guidelines and Structure

#### Follow the Test Pyramid

```
        /\
       /E2E\       (few, slow, critical journeys)
      /------\
     /Integration\ (moderate, real infrastructure)
    /------------\
   /   Unit Tests \ (many, fast, isolated)
  /----------------\
```

- Unit tests: hundreds/thousands, milliseconds each.
- Integration tests: dozens/hundreds, seconds each.
- E2E tests: a handful, tens of seconds each.

#### Fail Fast

- Run unit tests first; they are the fastest and catch the most common defects.
- Run integration tests only after unit tests pass.
- Run E2E tests last.

#### Build Once, Deploy Many

```yaml
- name: Build
  run: dotnet build -c Release
# Promote the same artifact through environments
```

- Build the artifact once; do not rebuild for each environment.
- This ensures the exact bits tested are the bits deployed.

#### Never Skip Tests

```yaml
# ✅ Tests gate the pipeline
- run: dotnet test --no-build -c Release

# ❌ Do not bypass tests
# - run: dotnet deploy --skip-tests
```

- Tests must be a required gate for merge and deployment .

#### Cache Dependencies

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
```

- NuGet restore is a common bottleneck; cache packages to speed up pipelines.

#### Publish Results Consistently

- Use `if: always()` to upload results even on failure.
- Use standard formats (TRX, JUnit XML) so dashboards can parse them.

#### Set Realistic Coverage Thresholds

- Target 80%+ line coverage for domain logic .
- Exclude generated code and migrations.
- Do not chase 100% coverage; focus on critical paths.

#### Isolate Tests for Parallelism

- Use `[Collection]` (xUnit) or `[NonParallelizable]` (NUnit) for shared resources.
- Run unit tests in parallel; run integration tests sequentially if they share a database .
- Use Testcontainers with unique ports/databases per test class .

#### Treat Pipeline as Code

- Store YAML pipelines in the repository.
- No manual click-ops in the CI UI.
- Version and review pipeline changes like application code.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complete CI Pipeline Following Best Practices

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - uses: actions/cache@v4
        with:
          path: ~/.nuget/packages
          key: nuget-${{ hashFiles('**/*.csproj') }}
      - run: dotnet restore
      - run: dotnet build --no-restore -c Release
      - run: dotnet format --verify-no-changes --no-restore
      - run: >
          dotnet test --no-build -c Release
          --logger "trx;LogFileName=test-results.trx"
          --collect:"XPlat Code Coverage"
          --results-directory TestResults
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: TestResults/
```

**Expected Output (conceptual)**

```
Restore: cached
Build: success
Format: clean
Tests: 250 passed, coverage 85%
Artifacts: uploaded
```

**Why this output occurs**

- Dependencies are cached, saving restore time.
- Build runs once; tests run against the built artifact.
- Results and coverage are published even if tests fail.

---

### Real-World Cases

- **Enterprise CI** — Azure DevOps pipelines with multi-stage gates.
- **Open-source** — GitHub Actions with matrix builds and coverage badges.
- **Microservices** — Per-service pipelines with contract and integration tests.

---

### References

- .NET CI/CD best practices — Microsoft - https://learn.microsoft.com/en-us/dotnet/devops/
- CI/CD principles (community) - https://skillhub.360.com/skills/codewithmukesh/dotnet-claude-kit/ci-cd
- Testing standards and coverage targets (community) - https://github.com/affaan-m/ecc/blob/main/rules/csharp/testing.md
- xUnit parallel execution best practices (community) - https://github.com/nikiforovall/claude-code-rules/blob/main/plugins/handbook-dotnet/skills/dotnet-test/references/parallel-execution.md

---

## Framework and Tooling Summary

| Concern | Tool / Configuration | Notes |
|---|---|---|
| **CI platform** | GitHub Actions, Azure DevOps, GitLab CI | YAML pipelines as code |
| **Test runner** | `dotnet test` | Supports `--no-build`, `--filter`, `--logger`, `--collect` |
| **Coverage** | Coverlet (`coverlet.collector`) | `--collect:"XPlat Code Coverage"` |
| **Coverage reporting** | ReportGenerator | Converts Cobertura XML to HTML, lcov, SonarQube |
| **Test reporting** | TRX logger (native), JUnit logger, `trx2junit` | TRX for Azure DevOps; JUnit for GitLab/CircleCI |
| **Parallelism (xUnit)** | `xunit.runner.json`, `[Collection]` | Collections run in parallel by default |
| **Parallelism (NUnit)** | `[Parallelizable]`, `[LevelOfParallelism]` | Opt-in; no parallelism by default |
| **Containerized databases** | Testcontainers | Real PostgreSQL, SQL Server, etc. |
| **Pipeline patterns** | Build once, deploy many; fail fast; never skip tests | Community and Microsoft guidance |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| TRX as universal format | TRX is Microsoft-specific; GitLab and some CI platforms require JUnit XML conversion |
| EF Core InMemory provider | Not a relational database; unsuitable for integration tests requiring relational semantics |
| xUnit `[SetUp]` / `[TearDown]` | Not available; use constructors and `IDisposable` |
| xUnit parallelization on by default | Tests sharing resources must use `[Collection]` or `DisableParallelization` |
| NUnit parallelism | Opt-in via `[Parallelizable]`; nothing runs in parallel by default |
| `MaxParallelThreads = -1` | Unlimited threads; can cause resource exhaustion |
| `MaxParallelThreads = 0` | Uses processor count (xUnit default algorithm may be processor count × 2) |
| Coverage thresholds as absolute gates | 100% coverage does not guarantee correctness; pair with mutation testing |
| `trx2junit` conversion | Required for platforms that do not natively parse TRX |
| `--no-build` with `dotnet test` | Requires a prior `dotnet build`; avoids redundant compilation |

---

## References (Consolidated)

- .NET CI/CD with GitHub Actions — Microsoft - https://learn.microsoft.com/en-us/dotnet/devops/github-actions-overview
- CI/CD for .NET — Microsoft - https://learn.microsoft.com/en-us/dotnet/devops/
- .NET CI/CD patterns (community) - https://skillhub.360.com/skills/codewithmukesh/dotnet-claude-kit/ci-cd
- Parallel test execution control (community) - https://github.com/nikiforovall/claude-code-rules/blob/main/plugins/handbook-dotnet/skills/dotnet-test/references/parallel-execution.md
- Running tests in parallel — NUnit - https://docs.nunit.org/articles/nunit/writing-tests/parallel-tests.html
- xUnit MaxParallelThreads API - https://api.xunit.net/
- Use code coverage for unit testing — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage
- Coverlet documentation - https://github.com/coverlet-coverage/coverlet
- ReportGenerator - https://github.com/danielpalme/ReportGenerator
- Coverage collection guide (community) - https://github.com/afonsoft/skills/blob/main/skills/quality-test-implementation/references/coverage-dotnet.md
- TRX to JUnit conversion (community) - https://github.com/thomhurst/TUnit/pull/5019
- trx2junit NuGet package - https://www.nuget.org/packages/trx2junit
- Testing standards — xUnit / CI gates (community) - https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/main/.claude/rules/testing.md
- Automated testing in Azure Pipelines (community) - https://cloudwebschool.com/automated-testing-azure-pipelines
- C# testing standards (community) - https://github.com/affaan-m/ecc/blob/main/rules/csharp/testing.md
- xUnit parallel execution best practices (community) - https://github.com/nikiforovall/claude-code-rules/blob/main/plugins/handbook-dotnet/skills/dotnet-test/references/parallel-execution.md