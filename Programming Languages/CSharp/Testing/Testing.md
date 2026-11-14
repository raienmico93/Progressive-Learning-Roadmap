# Testing Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Software testing is the systematic process of evaluating a software system to determine whether it satisfies specified requirements, behaves correctly under expected and unexpected conditions, and is free of defects that would compromise its quality or reliability.

**Technical Definition**
Software testing is a verification and validation activity that exercises a system or its components under controlled conditions, observes the results, and compares them against expected outcomes. In modern .NET development, testing spans multiple levels — unit, integration, end-to-end, and functional — each targeting different risk profiles. Testing is supported by test frameworks (xUnit, NUnit, MSTest, TUnit), assertion libraries (FluentAssertions, Shouldly), mocking libraries (Moq, NSubstitute, FakeItEasy), and specialized tools for mutation testing (Stryker.NET), BDD (Reqnroll), and browser automation (Playwright, Selenium) .

**Beginner-Friendly Explanation**
Testing is "checking that your code does what it is supposed to do." You write small programs called *tests* that call your code with known inputs and check that the outputs are correct. If the code breaks later, the tests fail, and you know something is wrong. Testing gives you confidence to change code without fear.

---

### Key Characteristics

- **Layered** — Testing is not one activity but a spectrum from fast unit tests to slow end-to-end tests .
- **Repeatable** — Good tests produce the same result every time they run .
- **Automated** — Tests run without human intervention and report pass/fail programmatically .
- **Fast feedback** — Unit tests complete in milliseconds, enabling rapid iteration .
- **Regression protection** — Tests catch defects introduced by changes to existing code .
- **Executable documentation** — Well-named tests describe expected behavior .
- **Design pressure** — Testability drives loose coupling and good design .
- **Risk-focused** — Different test types target different classes of risk (logic, integration, user flows) .

---

### Prerequisites

- Working knowledge of C# classes, interfaces, methods, and properties.
- Basic understanding of dependency injection and the SOLID principles (especially Dependency Inversion).
- Familiarity with the .NET CLI (`dotnet test`, `dotnet build`).
- A test framework package (xUnit, NUnit, MSTest, or TUnit) .
- Basic understanding of async/await for testing asynchronous code.
- Awareness of what a database, HTTP API, and browser are, to understand integration and E2E tests.

---

### Related Programming Areas

- **Test-driven development (TDD)** — Writing tests before production code .
- **Behavior-driven development (BDD)** — Specifying behavior in business-readable language .
- **Continuous integration (CI)** — Tests run automatically on every commit .
- **Mutation testing** — Evaluating the quality of the test suite itself .
- **Code coverage** — Measuring which code is exercised by tests .
- **Testcontainers** — Spinning up real dependencies (databases, brokers) in Docker for integration tests .
- **Browser automation** — Playwright and Selenium for end-to-end web testing .

---

### Core Concepts / Features

1. Testing (overarching discipline)
2. Test types
   - Unit tests
   - Integration tests
   - End-to-end tests
   - Functional tests
3. Test pyramid
4. Test-driven development (TDD)
5. Behavior-driven development (BDD)
6. Test coverage
7. Test isolation
8. Test doubles
   - Mocks
   - Stubs
   - Spies
   - Fakes
9. Testing best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Testing (Overarching Discipline)

### Definitions

**Core Definition**
Testing is the practice of executing software with the intent of finding defects and gaining confidence in its correctness, reliability, and quality.

**Technical Definition**
Testing encompasses a set of activities: test planning, test design, test execution, result analysis, and defect reporting. In automated .NET development, testing is implemented through test frameworks that discover, execute, and report on test methods. Tests are classified by scope (unit, integration, end-to-end), by purpose (functional, performance, security), and by technique (black-box, white-box, mutation). The modern testing ecosystem includes xUnit, NUnit, MSTest, and TUnit for test execution; Moq, NSubstitute, and FakeItEasy for test doubles; Stryker.NET for mutation testing; and Playwright and Selenium for browser automation .

**Beginner-Friendly Explanation**
Testing is "making sure your code works before your users find out it doesn't." You write tests that exercise your code and check the results. When you change code later, the tests tell you if you broke something.

---

### Purposes

- To verify that code meets its requirements.
- To detect regressions when code changes.
- To provide executable documentation of expected behavior.
- To enable safe refactoring.
- To drive good design through testability.
- To reduce the cost of defects by catching them early.
- To increase confidence in releases.
- To support continuous delivery.

---

### Syntax Rules and Structure

#### General Syntax (Test Project Setup)

```xml
<!-- Test project .csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
    <PackageReference Include="xunit" Version="2.7.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.7" />
  </ItemGroup>
</Project>
```

```bash
# Run all tests
dotnet test

# Run tests in a specific project
dotnet test tests/MyProject.Tests/MyProject.Tests.csproj

# Filter tests by name
dotnet test --filter "FullyQualifiedName~CalculatorTests"

# Collect coverage
dotnet test --collect:"XPlat Code Coverage"
```

**Component breakdown**

- `Microsoft.NET.Test.Sdk` — The test platform SDK required by all test frameworks.
- `xunit` — The test framework package.
- `xunit.runner.visualstudio` — Adapter for Visual Studio and `dotnet test`.
- `dotnet test` — Discovers and executes tests.

#### Syntax Rules

- Test projects must reference the test SDK and a test framework adapter.
- Test methods must be discoverable by the framework (e.g., `[Fact]` in xUnit, `[Test]` in NUnit, `[TestMethod]` in MSTest) .
- Tests should be in a separate project from production code.
- Tests should not depend on execution order.

#### Constraints and Limitations

- Tests add maintenance overhead; poorly written tests become a liability .
- Tests cannot prove the absence of defects, only their presence.
- Test execution time grows with suite size; balance speed and coverage.
- Test environments may differ from production, limiting fidelity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Minimal Test Class (xUnit)

```csharp
using Xunit;

public class Calculator
{
    public int Add(int a, int b) => a + b;
}

public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        var result = calculator.Add(2, 3);

        // Assert
        Assert.Equal(5, result);
    }
}
```

**Line-by-line comments**

- `Calculator` — The production class under test.
- `[Fact]` — Marks the method as a test with no parameters.
- `Arrange` — Creates the object under test.
- `Act` — Calls the method being tested.
- `Assert.Equal` — Verifies the expected result.

**Expected Output**

```
Test Run Successful.
Total tests: 1
     Passed: 1
```

**Why this output occurs**

- The test method is discovered by the xUnit adapter.
- `Add(2, 3)` returns `5`, matching the assertion.

---

### Real-World Cases

- **API services** — Unit and integration tests validate endpoints and business logic.
- **Desktop applications** — Tests validate view models and services.
- **Libraries** — Comprehensive unit tests ensure backward compatibility.
- **CI/CD pipelines** — Tests gate deployments and prevent regressions.

---

### References

- .NET testing documentation — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/
- xUnit.net documentation - https://xunit.net/
- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices

---

## 2. Test Types

### Definitions

**Core Definition**
Test types are categories of testing activities distinguished by their scope, purpose, and the level of the system they exercise.

**Technical Definition**
The principal test types in modern .NET development are unit tests (isolated component verification), integration tests (component interaction verification), end-to-end tests (full-system user-flow verification), and functional tests (requirement verification from the user's perspective). These types differ in speed, cost, fidelity, and the class of defects they detect .

**Beginner-Friendly Explanation**
Different tests check different things. Unit tests check one small piece. Integration tests check that pieces work together. End-to-end tests check the whole system from the user's point of view. Functional tests check that features do what the requirements say.

---

### Purposes

- To match test scope to the risk being addressed.
- To balance speed, cost, and confidence.
- To detect defects at the earliest possible stage.
- To provide appropriate feedback for different development activities.
- To avoid over-reliance on slow, brittle tests.
- To structure a balanced test portfolio.

---

### Sub-Feature: Unit Tests

#### Definitions

**Core Definition**
A unit test exercises a single, isolated unit of code — typically a method or class — to verify that it behaves as expected.

**Technical Definition**
Unit tests are fast, deterministic, and isolated from external dependencies (databases, file systems, networks, clocks). They use test doubles to replace collaborators and assert on return values, state changes, or thrown exceptions. In .NET, unit tests are written with xUnit, NUnit, MSTest, or TUnit and follow the Arrange-Act-Assert pattern .

**Beginner-Friendly Explanation**
A unit test checks one small piece of code in isolation. It is fast and runs in milliseconds.

#### Purposes

- To verify the correctness of individual methods and classes.
- To catch logic errors early.
- To enable safe refactoring.
- To document the expected behavior of a unit.
- To drive design toward loose coupling and testability.
- To provide fast feedback during development.

#### Syntax Rules and Structure

```csharp
[Fact]
public void MethodName_Scenario_ExpectedBehavior()
{
    // Arrange
    var sut = new SystemUnderTest();

    // Act
    var result = sut.Method(input);

    // Assert
    Assert.Equal(expected, result);
}
```

**Component breakdown**

- `[Fact]` — xUnit attribute for a parameterless test.
- `Arrange` — Setup of the system under test and dependencies.
- `Act` — Invocation of the method being tested.
- `Assert` — Verification of the expected outcome.

**Syntax Rules**

- One logical assertion per test (prefer a single Act).
- Tests must be isolated: no shared mutable state .
- Tests must be deterministic: no reliance on time, randomness, or external services.
- Test names should describe the scenario and expected behavior .

**Constraints and Limitations**

- Unit tests cannot catch integration defects (e.g., SQL errors, serialization mismatches).
- Over-mocking can produce tests that pass even when the real system is broken.
- Unit tests do not verify user-facing behavior.

#### Annotated Code Example

```csharp
public class StringCalculator
{
    public int Add(string numbers)
    {
        if (string.IsNullOrEmpty(numbers)) return 0;
        return numbers.Split(',').Select(int.Parse).Sum();
    }
}

public class StringCalculatorTests
{
    [Fact]
    public void Add_EmptyString_ReturnsZero()
    {
        var calc = new StringCalculator();
        var result = calc.Add("");
        Assert.Equal(0, result);
    }

    [Theory]
    [InlineData("1", 1)]
    [InlineData("1,2", 3)]
    [InlineData("10,20,30", 60)]
    public void Add_CommaSeparatedNumbers_ReturnsSum(string input, int expected)
    {
        var calc = new StringCalculator();
        var result = calc.Add(input);
        Assert.Equal(expected, result);
    }
}
```

**Expected Output**

```
Passed: Add_EmptyString_ReturnsZero
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "1", expected: 1)
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "1,2", expected: 3)
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "10,20,30", expected: 60)
```

**Why this output occurs**

- `[Theory]` with `[InlineData]` runs the same test with different inputs .
- Each data row is a separate test case.

#### Real-World Cases

- **Business logic** — Pricing rules, validation, calculations.
- **Domain models** — Invariants and state transitions.
- **Utility methods** — String manipulation, date formatting.

#### References

- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit Theory and data-driven tests - https://xunit.net/docs/getting-started/netcore/cmdline

---

### Sub-Feature: Integration Tests

#### Definitions

**Core Definition**
An integration test verifies that two or more components work together correctly, including their interactions with infrastructure such as databases, message brokers, and HTTP services.

**Technical Definition**
Integration tests exercise the boundaries between components — the database, the file system, the HTTP pipeline, or external services. In .NET, integration tests for ASP.NET Core use `WebApplicationFactory<T>` to bootstrap the application in memory, and Testcontainers to spin up real dependencies in Docker . They are slower than unit tests but catch interface and wiring defects that unit tests cannot.

**Beginner-Friendly Explanation**
Integration tests check that different parts of your system work together — for example, that your repository actually saves to a database, or that your API endpoint returns the right HTTP status.

#### Purposes

- To verify database queries and mappings.
- To verify HTTP pipeline configuration (routing, middleware, filters).
- To verify serialization and deserialization.
- To catch configuration and wiring errors.
- To test against real infrastructure (databases, brokers).
- To increase confidence in deployment.

#### Syntax Rules and Structure

```csharp
public class ApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ApiIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetWeather_ReturnsSuccess()
    {
        var response = await _client.GetAsync("/weatherforecast");
        response.EnsureSuccessStatusCode();
    }
}
```

**Component breakdown**

- `IClassFixture<WebApplicationFactory<Program>>` — Shares the test server across tests in the class.
- `CreateClient()` — Returns an `HttpClient` configured to call the in-memory server.
- `EnsureSuccessStatusCode()` — Asserts a 2xx response.

**Syntax Rules**

- Use `WebApplicationFactory<T>` for in-memory ASP.NET Core integration tests.
- Use Testcontainers for real databases, brokers, and other infrastructure.
- Clean up database state between tests (Respawn or transactions) .
- Integration tests belong in a separate project from unit tests.

**Constraints and Limitations**

- Slower than unit tests; should be used selectively.
- Testcontainers requires Docker.
- In-memory databases (EF Core InMemory) do not behave like relational databases.
- External services should be mocked or replaced with test doubles.

#### Annotated Code Example

```csharp
public class BlogApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public BlogApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace the real database with a test database
                services.RemoveAll<DbContextOptions<BloggingContext>>();
                services.AddDbContext<BloggingContext>(options =>
                    options.UseSqlite("DataSource=:memory:"));
            });
        }).CreateClient();
    }

    [Fact]
    public async Task GetBlogs_ReturnsOkWithBlogs()
    {
        var response = await _client.GetAsync("/api/blogs");
        response.EnsureSuccessStatusCode();
        var blogs = await response.Content.ReadFromJsonAsync<List<BlogDto>>();
        Assert.NotNull(blogs);
    }
}
```

**Expected Output**

```
Passed: GetBlogs_ReturnsOkWithBlogs
```

**Why this output occurs**

- The in-memory server boots the real ASP.NET Core pipeline.
- The database is replaced with SQLite in-memory.
- The endpoint returns a JSON array, which is deserialized.

#### Real-World Cases

- **API endpoints** — Verify routing, model binding, and status codes.
- **Database repositories** — Verify SQL and ORM mappings.
- **Message brokers** — Verify publish/consume round trips.
- **Authentication** — Verify token validation and authorization policies.

#### References

- Integration testing in ASP.NET Core - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Testcontainers for .NET - https://dotnet.testcontainers.org/
- Respawn (database cleanup) - https://github.com/jbogard/Respawn

---

### Sub-Feature: End-to-End Tests

#### Definitions

**Core Definition**
An end-to-end (E2E) test exercises the entire system — from the user interface through the backend to the database and back — simulating a real user journey.

**Technical Definition**
E2E tests drive the application through its external interfaces (browser, HTTP API, mobile UI) and verify observable outcomes. In .NET web applications, Playwright for .NET and Selenium are the primary browser automation tools. E2E tests are slow, brittle, and expensive, so they should be reserved for critical user journeys .

**Beginner-Friendly Explanation**
E2E tests simulate a real user: they open the browser, click buttons, fill forms, and check that the right page appears. They test the whole system at once.

#### Purposes

- To verify critical user journeys.
- To catch integration defects across the full stack.
- To validate deployment configuration.
- To provide confidence in releases.
- To test cross-browser and cross-device behavior.
- To catch regressions that unit and integration tests miss.

#### Syntax Rules and Structure

```csharp
using Microsoft.Playwright;
using Xunit;

public class LoginE2ETests : IAsyncLifetime
{
    private IPlaywright _playwright = null!;
    private IBrowser _browser = null!;
    private IPage _page = null!;

    public async Task InitializeAsync()
    {
        _playwright = await Playwright.CreateAsync();
        _browser = await _playwright.Chromium.LaunchAsync(new() { Headless = true });
        _page = await _browser.NewPageAsync();
    }

    [Fact]
    public async Task Login_ValidCredentials_RedirectsToDashboard()
    {
        await _page.GotoAsync("https://localhost:5001/login");
        await _page.FillAsync("#username", "alice");
        await _page.FillAsync("#password", "secret");
        await _page.ClickAsync("button[type=submit]");
        await _page.WaitForURLAsync("**/dashboard");
        Assert.Contains("Dashboard", await _page.TitleAsync());
    }

    public async Task DisposeAsync()
    {
        await _browser.DisposeAsync();
        _playwright.Dispose();
    }
}
```

**Expected Output**

```
Passed: Login_ValidCredentials_RedirectsToDashboard
```

**Why this output occurs**

- Playwright launches Chromium, navigates to the login page, fills credentials, submits the form, and waits for the dashboard URL.
- The assertion confirms the page title.

#### Syntax Rules

- Each test should run in a new browser context for isolation .
- Use Playwright's auto-waiting and web-first assertions; avoid hard-coded delays .
- Keep E2E suites small (5–10% of total tests) .
- Run E2E tests against a deployed environment or a full local stack.

#### Constraints and Limitations

- Slow (seconds to minutes per test).
- Brittle due to UI changes, timing, and environment differences.
- Expensive to maintain.
- Debugging failures is harder than unit tests.

#### Real-World Cases

- **E-commerce** — Add to cart, checkout, payment.
- **SaaS onboarding** — Sign up, verify email, first login.
- **Banking** — Transfer funds, view statements.

#### References

- Playwright for .NET documentation - https://playwright.dev/dotnet/
- Selenium documentation - https://www.selenium.dev/documentation/
- End-to-end testing best practices - https://playwright.dev/dotnet/docs/best-practices

---

### Sub-Feature: Functional Tests

#### Definitions

**Core Definition**
A functional test verifies that a feature or system behaves according to its specified requirements from the user's perspective.

**Technical Definition**
Functional testing is black-box testing: it validates inputs and expected outputs against requirements without knowledge of internal implementation. It can be performed at the unit, integration, or system level. Functional tests answer the question "Does the software do what the requirements say?" rather than "Do the components integrate correctly?" .

**Beginner-Friendly Explanation**
Functional tests check that features work the way the requirements say they should. They focus on what the user puts in and what they get back.

#### Purposes

- To verify that features meet business requirements.
- To validate acceptance criteria.
- To provide a user-centric view of correctness.
- To complement structural tests (unit, integration).
- To support acceptance testing and sign-off.
- To catch missing or misunderstood requirements.

#### Syntax Rules and Structure

Functional tests can be written at any level, from unit-style acceptance tests to full-system scenarios:

```csharp
[Fact]
public void Transfer_Funds_Succeeds_When_BalanceIsSufficient()
{
    // Arrange (user perspective)
    var account = new Account(balance: 100m);
    var target = new Account(balance: 0m);

    // Act
    account.Transfer(target, 50m);

    // Assert
    Assert.Equal(50m, account.Balance);
    Assert.Equal(50m, target.Balance);
}
```

**Component breakdown**

- The test expresses a requirement ("transfer succeeds when balance is sufficient").
- The assertion verifies the user-visible outcome (balances).

**Syntax Rules**

- Functional tests should be derived from requirements or acceptance criteria.
- They should be independent of implementation details.
- They can be automated at any level (unit, integration, E2E).

#### Constraints and Limitations

- Functional tests do not verify non-functional requirements (performance, security).
- They may duplicate coverage if not carefully designed.
- They require clear, testable requirements.

#### Real-World Cases

- **Acceptance testing** — Verifying user stories before release.
- **Regression testing** — Confirming that features still work after changes.
- **Contract testing** — Verifying that APIs honor their contracts.

#### References

- Functional vs. integration testing — Tricentis - https://www.tricentis.com/learn/functional-testing-vs-integration-testing
- ISTQB glossary — functional testing - https://glossary.istqb.org/

---

## 3. Test Pyramid

### Definitions

**Core Definition**
The test pyramid is a model that recommends distributing automated tests across layers: many fast unit tests at the base, fewer integration tests in the middle, and very few end-to-end tests at the top.

**Technical Definition**
The test pyramid was introduced by Mike Cohn in *Succeeding with Agile* (2009) and popularized by Martin Fowler. It expresses a trade-off between test speed, cost, and confidence. The base layer (unit tests) is fast, cheap, and numerous; the middle layer (service/integration tests) is slower and fewer; the top layer (UI/E2E tests) is slowest, most brittle, and least numerous. A common target distribution is approximately 70% unit, 20% integration, and 10% E2E, though exact ratios vary by project .

**Beginner-Friendly Explanation**
The test pyramid says: write lots of small, fast tests; write some medium tests; and write only a few big, slow tests. This gives you fast feedback and good coverage without a slow, brittle test suite.

---

### Purposes

- To balance test speed, cost, and confidence.
- To avoid over-reliance on slow, brittle UI tests.
- To provide fast feedback during development.
- To allocate testing effort efficiently.
- To guide the design of a healthy test portfolio.
- To prevent the "ice cream cone" anti-pattern (many manual/E2E tests, few unit tests).

---

### Syntax Rules and Structure

#### General Structure

```
        /\
       /  \        E2E tests (5–10%)
      /----\       Critical user journeys only
     /      \
    /--------\     Integration tests (15–25%)
   /          \    Service layer, DI, database, HTTP
  /------------\
 /--------------\  Unit tests (65–80%)
/________________\ Fast, isolated, numerous
```

**Component breakdown**

- **Base** — Unit tests: milliseconds per test, no infrastructure.
- **Middle** — Integration/service tests: seconds per test, real interactions.
- **Top** — E2E/UI tests: tens of seconds per test, full system.

#### Syntax Rules

- Write many unit tests, fewer integration tests, and few E2E tests .
- Do not invert the pyramid (the "ice cream cone" anti-pattern).
- Adjust ratios based on system complexity and risk.
- Frontend-heavy applications may use a "testing trophy" (more integration tests) .

#### Constraints and Limitations

- The pyramid is a heuristic, not a law.
- Microservices and distributed systems may require more integration tests.
- UI-heavy applications may benefit from a different shape.
- The pyramid does not prescribe specific tools or frameworks.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Distributing Tests Across Layers

```
Project structure:
src/
  MyApp/                    (production code)
tests/
  MyApp.UnitTests/          (hundreds of tests, milliseconds)
  MyApp.IntegrationTests/   (dozens of tests, seconds)
  MyApp.E2ETests/           (a handful of tests, tens of seconds)
```

**Expected Output (conceptual)**

```
Unit tests:        500 passed (total: 1.2s)
Integration tests:  40 passed (total: 12s)
E2E tests:           5 passed (total: 45s)
```

**Why this output occurs**

- Unit tests dominate in count and are fastest.
- Integration tests are fewer and slower.
- E2E tests are fewest and slowest.

---

### Real-World Cases

- **Web APIs** — Heavy unit tests on business logic, moderate integration tests on endpoints, few E2E tests on critical flows.
- **Microservices** — More integration and contract tests to verify service boundaries.
- **Mobile apps** — Unit tests on view models, integration tests on APIs, E2E tests on key journeys.

---

### References

- The Practical Test Pyramid — Martin Fowler - https://martinfowler.com/articles/practical-test-pyramid.html
- Test pyramid — Mike Cohn (Succeeding with Agile, 2009) - https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid
- Testing pyramid — CircleCI - https://circleci.com/blog/testing-pyramid/

---

## 4. Test-Driven Development (TDD)

### Definitions

**Core Definition**
Test-driven development is a software development process in which tests are written before the production code, following a short Red-Green-Refactor cycle.

**Technical Definition**
TDD follows three steps: **Red** — write a failing test for the next small piece of functionality; **Green** — write the minimal production code to make the test pass; **Refactor** — improve the code while keeping all tests green. TDD produces a comprehensive, fast, and reliable test suite and drives emergent design. In .NET, TDD is commonly practiced with xUnit or NUnit .

**Beginner-Friendly Explanation**
TDD means "write the test first, then write the code that makes it pass." It feels backwards at first, but it ensures that every piece of code has a test and that the design stays simple.

---

### Purposes

- To ensure every piece of code has a test.
- To drive simple, testable design.
- To provide immediate feedback on correctness.
- To prevent over-engineering (write only what is needed to pass).
- To create a regression safety net.
- To improve confidence during refactoring.
- To document behavior through tests.

---

### Syntax Rules and Structure

#### The TDD Cycle

```
RED → Write a failing test
GREEN → Write minimal code to pass
REFACTOR → Improve while keeping tests green
Repeat
```

#### Syntax Rules (xUnit Example)

```csharp
// RED: Write a failing test
[Fact]
public void CreateAsync_ValidInput_ReturnsPatient()
{
    var service = new PatientService();
    var result = service.Create(new CreatePatientDto { Name = "Alice" });
    Assert.NotNull(result);
}

// GREEN: Write minimal code
public class PatientService
{
    public PatientDto Create(CreatePatientDto input)
    {
        return new PatientDto { Name = input.Name };
    }
}

// REFACTOR: Improve without breaking tests
public class PatientService
{
    private readonly IPatientRepository _repository;
    public PatientService(IPatientRepository repository) => _repository = repository;

    public PatientDto Create(CreatePatientDto input)
    {
        var patient = new Patient(input.Name);
        _repository.Add(patient);
        return PatientDto.From(patient);
    }
}
```

**Component breakdown**

- **Red** — The test fails because the production code does not exist or is incomplete.
- **Green** — The simplest implementation passes the test.
- **Refactor** — The code is restructured while tests remain green.

#### Syntax Rules

- Write only enough production code to pass the current test.
- Refactor in small steps, running tests after each change.
- Keep the cycle short (minutes, not hours).
- Use the FIRST principles for tests .

#### Constraints and Limitations

- TDD requires discipline and practice.
- It can be awkward for exploratory or UI-heavy work.
- Over-testing trivial code can slow development.
- TDD does not replace integration or E2E testing.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Full TDD Cycle

```csharp
// Step 1: RED — write a failing test
public class FizzBuzzTests
{
    [Fact]
    public void Get_1_Returns_1()
    {
        Assert.Equal("1", FizzBuzz.Get(1));
    }
}

// Step 2: GREEN — minimal code
public static class FizzBuzz
{
    public static string Get(int n) => n.ToString();
}

// Step 3: RED — next test
[Fact]
public void Get_3_Returns_Fizz()
{
    Assert.Equal("Fizz", FizzBuzz.Get(3));
}

// Step 4: GREEN — add logic
public static string Get(int n)
{
    if (n % 3 == 0) return "Fizz";
    return n.ToString();
}

// Step 5: Refactor — extract rules, add tests for 5 and 15
```

**Expected Output**

```
All FizzBuzz tests pass after each Green step.
```

**Why this output occurs**

- Each test drives a new piece of behavior.
- The implementation grows incrementally.

---

### Real-World Cases

- **Domain logic** — TDD is highly effective for business rules.
- **API development** — Test-first design clarifies contracts.
- **Legacy modernization** — Characterization tests before refactoring.

---

### References

- TDD with xUnit in .NET (community) - https://github.com/thapaliyabikendra/ai-artifacts/blob/main/.claude/knowledge/implementations/dotnet/xunit-tdd.md
- Test-driven development — Microsoft - https://learn.microsoft.com/en-us/visualstudio/test/quick-start-test-driven-development-with-test-explorer
- TDD — Martin Fowler - https://martinfowler.com/bliki/TestDrivenDevelopment.html

---

## 5. Behavior-Driven Development (BDD)

### Definitions

**Core Definition**
Behavior-driven development is an agile practice that specifies system behavior in business-readable language and turns those specifications into automated tests.

**Technical Definition**
BDD emerged from TDD and emphasizes collaboration between developers, testers, and business stakeholders. Scenarios are written in Gherkin syntax (`Given`/`When`/`Then`) and bound to step definitions in code. In .NET, Reqnroll (the successor to SpecFlow) is the primary BDD framework, supporting xUnit, NUnit, MSTest, and TUnit as underlying runners .

**Beginner-Friendly Explanation**
BDD is "TDD with plain-English tests." You write scenarios like "Given a customer with $100, When they withdraw $50, Then the balance is $50," and those scenarios become automated tests.

---

### Purposes

- To bridge the communication gap between technical and business stakeholders.
- To specify behavior in unambiguous, executable language.
- To drive development from acceptance criteria.
- To provide living documentation.
- To reduce misunderstandings about requirements.
- To align testing with business value.

---

### Syntax Rules and Structure

#### Gherkin Syntax

```gherkin
Feature: Account withdrawals
  As a customer
  I want to withdraw cash
  So that I can pay for goods

  Scenario: Sufficient funds
    Given an account with balance 100
    When I withdraw 50
    Then the balance should be 50

  Scenario: Insufficient funds
    Given an account with balance 30
    When I withdraw 50
    Then the withdrawal should fail
    And the balance should remain 30
```

#### C# Step Definitions (Reqnroll)

```csharp
[Binding]
public class WithdrawalSteps
{
    private Account _account = null!;

    [Given("an account with balance {int}")]
    public void GivenAccountWithBalance(int balance)
    {
        _account = new Account(balance);
    }

    [When("I withdraw {int}")]
    public void WhenIWithdraw(int amount)
    {
        _account.Withdraw(amount);
    }

    [Then("the balance should be {int}")]
    public void ThenBalanceShouldBe(int expected)
    {
        Assert.Equal(expected, _account.Balance);
    }
}
```

**Component breakdown**

- `Feature` — A high-level capability.
- `Scenario` — A concrete example of behavior.
- `Given` — Precondition.
- `When` — Action.
- `Then` — Expected outcome.
- `[Binding]` — Marks the step definition class.
- `[Given]`, `[When]`, `[Then]` — Map Gherkin steps to C# methods.

#### Syntax Rules

- Scenarios should be written in business language, not technical jargon.
- Each scenario should test one behavior.
- Step definitions should be reusable and focused.
- Reqnroll is the recommended BDD framework for new .NET projects; SpecFlow is end-of-life .

#### Constraints and Limitations

- BDD requires collaboration and discipline.
- Poorly written scenarios become brittle and hard to maintain.
- BDD is not a replacement for unit testing.
- Tooling and maintenance overhead can be significant.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complete Reqnroll Scenario

```gherkin
Feature: Calculator
  Scenario: Add two numbers
    Given the first number is 2
    And the second number is 3
    When I add them
    Then the result should be 5
```

```csharp
[Binding]
public class CalculatorSteps
{
    private int _first, _second, _result;

    [Given("the first number is {int}")]
    public void GivenFirstNumber(int n) => _first = n;

    [Given("the second number is {int}")]
    public void GivenSecondNumber(int n) => _second = n;

    [When("I add them")]
    public void WhenIAdd() => _result = _first + _second;

    [Then("the result should be {int}")]
    public void ThenResultShouldBe(int expected) => Assert.Equal(expected, _result);
}
```

**Expected Output**

```
Scenario: Add two numbers ... passed
```

**Why this output occurs**

- Reqnroll parses the Gherkin and invokes the matching step definitions.
- The assertions verify the expected result.

---

### Real-World Cases

- **Financial systems** — Withdrawal, deposit, and transfer rules.
- **E-commerce** — Cart, checkout, and discount rules.
- **Healthcare** — Appointment scheduling and eligibility.

---

### References

- Reqnroll documentation - https://docs.reqnroll.net/
- BDD — Martin Fowler - https://martinfowler.com/bliki/BehaviorDrivenDevelopment.html
- SpecFlow (EOL; use Reqnroll) - https://specflow.org/

---

## 6. Test Coverage

### Definitions

**Core Definition**
Test coverage is a metric that measures the proportion of code exercised by a test suite.

**Technical Definition**
Coverage is typically reported at the line, branch, and method levels. Line coverage measures the percentage of executable lines executed; branch coverage measures the percentage of decision branches taken. Coverage is collected by tools such as Coverlet and reported through CI pipelines. High coverage is a necessary but not sufficient condition for high test quality; mutation testing complements coverage by assessing whether tests actually detect faults .

**Beginner-Friendly Explanation**
Coverage tells you "how much of your code was run by your tests." If coverage is 80%, then 80% of your code was executed. But coverage does not tell you whether the tests actually checked the right things.

---

### Purposes

- To identify untested code paths.
- To provide an objective metric for CI quality gates.
- To guide test-writing priorities.
- To highlight dead or unreachable code.
- To complement mutation testing.
- To track testing progress over time.

---

### Syntax Rules and Structure

#### Coverage Collection

```bash
# Collect coverage with Coverlet
dotnet test --collect:"XPlat Code Coverage"

# Generate a report (using ReportGenerator)
reportgenerator -reports:coverage.cobertura.xml -targetdir:coverage-report
```

```xml
<!-- Coverlet MSBuild integration -->
<PackageReference Include="coverlet.collector" Version="6.0.0" />
```

**Component breakdown**

- `--collect:"XPlat Code Coverage"` — Enables cross-platform coverage collection.
- `coverage.cobertura.xml` — The coverage data file.
- `reportgenerator` — Converts coverage data into HTML reports.

#### Syntax Rules

- Coverage should be collected in CI, not only locally.
- Set realistic thresholds (e.g., 70–80% for line coverage) .
- Track coverage trends over time.
- Use branch coverage alongside line coverage.

#### Constraints and Limitations

- Coverage measures execution, not verification .
- 100% coverage does not guarantee correct tests.
- Coverage of generated code, DTOs, and glue code can inflate numbers.
- Over-emphasis on coverage can lead to gaming the metric.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Coverage Report

```bash
dotnet test --collect:"XPlat Code Coverage"
```

**Expected Output (conceptual)**

```
Coverage Results:
  Line coverage:   85%
  Branch coverage: 72%
  Method coverage: 90%
```

**Why this output occurs**

- 85% of lines were executed by tests.
- 72% of branches were taken.
- 90% of methods were called.

---

### Real-World Cases

- **CI quality gates** — Fail builds if coverage drops below a threshold.
- **Legacy code** — Identify untested areas before refactoring.
- **Compliance** — Demonstrate test coverage to auditors.

---

### References

- Coverlet documentation - https://github.com/coverlet-coverage/coverlet
- ReportGenerator - https://github.com/danielpalme/ReportGenerator
- Unit test coverage — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage

---

## 7. Test Isolation

### Definitions

**Core Definition**
Test isolation is the property that each test runs independently, without relying on or affecting other tests.

**Technical Definition**
Isolated tests have no shared mutable state, no dependency on execution order, and no reliance on external resources that may change between runs. Isolation is achieved through fresh setup per test, test doubles for collaborators, and cleanup of external state (databases, files, environment variables). xUnit enforces isolation by creating a new instance of the test class for each test method .

**Beginner-Friendly Explanation**
Isolation means "tests don't interfere with each other." If Test A passes and Test B fails, it should be because of Test B's code, not because Test A changed something.

---

### Purposes

- To make failures attributable to the correct test.
- To allow tests to run in any order.
- To enable parallel execution.
- To prevent flaky tests caused by shared state.
- To make tests reproducible.
- To simplify debugging.

---

### Syntax Rules and Structure

#### Isolation Techniques

```csharp
// Good: fresh instance per test (xUnit default)
public class OrderTests
{
    private readonly OrderService _service;

    public OrderTests()
    {
        _service = new OrderService(new InMemoryOrderRepository());
    }

    [Fact]
    public void CreateOrder_ValidItems_Succeeds()
    {
        // uses its own _service
    }
}
```

```csharp
// Good: no shared static state
[Fact]
public void Test1_CreatePatient()
{
    var patient = new Patient { Name = "John" };
}

[Fact]
public void Test2_UsePatient()
{
    // does NOT depend on Test1
}
```

**Component breakdown**

- Constructor — Runs before each test, creating fresh state.
- `IDisposable` — Cleans up after each test.
- No static fields — Avoid shared state.

#### Syntax Rules

- Each test should set up its own data.
- Use test doubles to avoid shared infrastructure.
- Clean up external state (database, files) after each test.
- Do not rely on test execution order.
- Avoid static mutable fields in test classes .

#### Constraints and Limitations

- Isolation adds setup overhead.
- Some scenarios (e.g., database schema) are expensive to reset per test.
- Testcontainers and Respawn mitigate the cost for integration tests .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Isolated vs. Non-Isolated Tests

```csharp
// Non-isolated: shared static state
public class BadTests
{
    private static Patient _sharedPatient;

    [Fact]
    public void Test1_Create() { _sharedPatient = new Patient { Name = "John" }; }

    [Fact]
    public void Test2_Use() { Assert.Equal("John", _sharedPatient.Name); } // depends on Test1
}

// Isolated: fresh state per test
public class GoodTests
{
    [Fact]
    public void Create_ReturnsPatient()
    {
        var patient = new Patient { Name = "John" };
        Assert.Equal("John", patient.Name);
    }

    [Fact]
    public void Use_SeededPatient()
    {
        var patient = new Patient { Name = "Jane" };
        Assert.Equal("Jane", patient.Name);
    }
}
```

**Expected Output**

```
GoodTests: both tests pass independently.
BadTests: Test2 may fail if run alone.
```

**Why this output occurs**

- The bad tests share static state, creating an order dependency.
- The good tests create their own data.

---

### Real-World Cases

- **Parallel test execution** — Isolated tests can run concurrently.
- **CI pipelines** — Isolated tests produce consistent results across runs.
- **Testcontainers** — Fresh database containers per test class.

---

### References

- Unit testing best practices — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit isolation — https://xunit.net/docs/shared-context
- Respawn — https://github.com/jbogard/Respawn

---

## 8. Test Doubles

### Definitions

**Core Definition**
A test double is a replaceable object that stands in for a real dependency during testing.

**Technical Definition**
Test doubles are classified into five types: **Dummies** (passed but never used), **Stubs** (provide canned answers), **Spies** (record calls for later verification), **Mocks** (set expectations and verify interactions), and **Fakes** (working but simplified implementations). In .NET, Moq, NSubstitute, and FakeItEasy are the primary mocking libraries .

**Beginner-Friendly Explanation**
Test doubles are "stunt doubles" for your dependencies. If your code depends on a database, you use a fake database so the test runs fast and does not need a real database.

---

### Purposes

- To isolate the unit under test from external dependencies.
- To make tests fast and deterministic.
- To simulate error conditions and edge cases.
- To verify interactions between objects.
- To replace slow or unavailable infrastructure.
- To enable testing of code that depends on non-deterministic systems (time, randomness).

---

### Sub-Feature: Mocks

#### Definitions

**Core Definition**
A mock is a test double that verifies interactions — it asserts that specific methods were called with specific arguments.

**Technical Definition**
Mocks are configured with expectations (`Setup` in Moq, `Received` in NSubstitute) and verified after the Act phase. A mock fails the test if the expected interaction did not occur or occurred with the wrong arguments .

**Beginner-Friendly Explanation**
A mock checks that your code talked to the right collaborator in the right way.

#### Purposes

- To verify that a dependency was called.
- To verify call arguments and order.
- To ensure side effects occurred.
- To test command-style interactions.

#### Syntax Rules and Structure

```csharp
// Moq
var mockRepo = new Mock<IPatientRepository>();
mockRepo.Setup(r => r.Add(It.IsAny<Patient>()));
// ... Act
mockRepo.Verify(r => r.Add(It.IsAny<Patient>()), Times.Once);

// NSubstitute
var repo = Substitute.For<IPatientRepository>();
// ... Act
repo.Received(1).Add(Arg.Any<Patient>());
```

**Component breakdown**

- `Setup` / `Returns` — Configure stub behavior (optional for pure mocks).
- `Verify` / `Received` — Assert the interaction.
- `Times.Once` / `Received(1)` — Specify the expected call count.

#### Constraints and Limitations

- Over-verification couples tests to implementation details.
- Mocks are not suitable for verifying state; use stubs or fakes.
- Mocking frameworks differ in syntax and performance .

#### Annotated Code Example

```csharp
[Fact]
public void CreatePatient_SavesToRepository()
{
    var repo = Substitute.For<IPatientRepository>();
    var service = new PatientService(repo);

    service.Create(new CreatePatientDto { Name = "Alice" });

    repo.Received(1).Add(Arg.Is<Patient>(p => p.Name == "Alice"));
}
```

**Expected Output**

```
Passed: CreatePatient_SavesToRepository
```

**Why this output occurs**

- The mock verifies that `Add` was called once with a patient named Alice.

#### Real-World Cases

- **Command handlers** — Verify that commands invoke repositories and services.
- **Event publishing** — Verify that events were raised.
- **Audit logging** — Verify that logging calls occurred.

#### References

- NSubstitute documentation - https://nsubstitute.github.io/
- Moq documentation - https://github.com/moq/moq
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology

---

### Sub-Feature: Stubs

#### Definitions

**Core Definition**
A stub is a test double that provides canned answers to calls made during a test, without verifying interactions.

**Technical Definition**
Stubs are configured to return specific values or throw specific exceptions when called. They replace real dependencies that would otherwise return unpredictable or slow results. Stubs do not cause tests to fail based on interaction; they only supply data .

**Beginner-Friendly Explanation**
A stub is a "fake answer machine." When your code asks it a question, it gives a predetermined answer.

#### Purposes

- To supply data to the unit under test.
- To simulate error conditions.
- To avoid slow or unavailable dependencies.
- To make tests deterministic.

#### Syntax Rules and Structure

```csharp
// Moq
var stub = new Mock<IPatientRepository>();
stub.Setup(r => r.GetById(1)).Returns(new Patient { Name = "Alice" });

// NSubstitute
var stub = Substitute.For<IPatientRepository>();
stub.GetById(1).Returns(new Patient { Name = "Alice" });
```

**Component breakdown**

- `Setup(...).Returns(...)` — Configures the stub.
- No `Verify` call — stubs are not verified.

#### Constraints and Limitations

- Stubs do not verify behavior.
- Over-stubbing can hide integration issues.
- Stubs should be used for queries, not commands.

#### Annotated Code Example

```csharp
[Fact]
public void GetPatient_ReturnsPatientFromRepository()
{
    var stub = Substitute.For<IPatientRepository>();
    stub.GetById(1).Returns(new Patient { Id = 1, Name = "Alice" });
    var service = new PatientService(stub);

    var result = service.Get(1);

    Assert.Equal("Alice", result.Name);
}
```

**Expected Output**

```
Passed: GetPatient_ReturnsPatientFromRepository
```

**Why this output occurs**

- The stub returns the configured patient.
- The assertion verifies the result.

#### Real-World Cases

- **Repository queries** — Return fixed data for service tests.
- **API clients** — Return canned JSON responses.
- **Time providers** — Return a fixed date/time.

#### References

- NSubstitute documentation - https://nsubstitute.github.io/
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology

---

### Sub-Feature: Spies

#### Definitions

**Core Definition**
A spy is a test double that records interactions for later inspection without failing the test automatically.

**Technical Definition**
Spies wrap real objects or interfaces and record calls, arguments, and return values. Unlike mocks, spies do not have built-in expectations; the test inspects the recorded calls manually. In .NET, NSubstitute's `Received()` and Moq's `Verify()` blur the line between spies and mocks, but the conceptual distinction remains .

**Beginner-Friendly Explanation**
A spy is a "recording device." It watches what your code does and lets you check the recording later.

#### Purposes

- To inspect interactions after the fact.
- To debug unexpected behavior.
- To verify call order or arguments without upfront expectations.
- To observe side effects.

#### Syntax Rules and Structure

```csharp
// NSubstitute (spy-like)
var spy = Substitute.For<ILogger>();
// ... Act
spy.Received().Log(Arg.Any<string>());
```

**Component breakdown**

- `Received()` — Asserts that a call occurred.
- The assertion happens after the Act phase.

#### Constraints and Limitations

- Spies are conceptually similar to mocks in most .NET libraries.
- Overuse leads to brittle tests.

#### Annotated Code Example

```csharp
[Fact]
public void ProcessOrder_LogsProcessingMessage()
{
    var spy = Substitute.For<ILogger<OrderService>>();
    var service = new OrderService(spy);

    service.Process(new Order());

    spy.Received(1).Log(
        LogLevel.Information,
        Arg.Any<EventId>(),
        Arg.Is<object>(o => o.ToString()!.Contains("Processing")),
        Arg.Any<Exception>(),
        Arg.Any<Func<object, Exception?, string>>());
}
```

**Expected Output**

```
Passed: ProcessOrder_LogsProcessingMessage
```

**Why this output occurs**

- The spy records the logging call.
- The assertion verifies the call occurred with the expected message.

#### Real-World Cases

- **Logging verification** — Confirm that important events were logged.
- **Audit trails** — Verify that audit entries were created.

#### References

- NSubstitute documentation - https://nsubstitute.github.io/
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology

---

### Sub-Feature: Fakes

#### Definitions

**Core Definition**
A fake is a test double with a working but simplified implementation of a dependency.

**Technical Definition**
Fakes contain real logic but take shortcuts that make them unsuitable for production. Examples include in-memory databases, in-memory repositories, and fake HTTP clients. Fakes are preferred over mocks when state verification is more natural than interaction verification .

**Beginner-Friendly Explanation**
A fake is a "simplified working version" of a dependency. For example, an in-memory database that works like a real database but stores everything in memory.

#### Purposes

- To replace slow or complex dependencies with simpler implementations.
- To enable state-based testing.
- To test against realistic behavior without real infrastructure.
- To reduce test setup complexity.

#### Syntax Rules and Structure

```csharp
public class InMemoryPatientRepository : IPatientRepository
{
    private readonly List<Patient> _patients = new();

    public void Add(Patient patient) => _patients.Add(patient);
    public Patient? GetById(int id) => _patients.FirstOrDefault(p => p.Id == id);
    public IEnumerable<Patient> GetAll() => _patients;
}
```

**Component breakdown**

- Implements the same interface as the real dependency.
- Stores data in memory.
- Provides working behavior for tests.

#### Constraints and Limitations

- Fakes may diverge from the real implementation.
- They require maintenance as the interface evolves.
- They are not suitable for testing database-specific behavior (SQL, constraints).

#### Annotated Code Example

```csharp
[Fact]
public void CreatePatient_ThenGet_ReturnsSamePatient()
{
    var repo = new InMemoryPatientRepository();
    var service = new PatientService(repo);

    service.Create(new CreatePatientDto { Name = "Alice" });
    var patient = service.Get(1);

    Assert.Equal("Alice", patient.Name);
}
```

**Expected Output**

```
Passed: CreatePatient_ThenGet_ReturnsSamePatient
```

**Why this output occurs**

- The fake repository stores and retrieves the patient.
- The assertion verifies the round trip.

#### Real-World Cases

- **In-memory databases** — SQLite in-memory for integration-style tests.
- **In-memory message buses** — For event-driven tests.
- **Fake HTTP clients** — For service client tests.

#### References

- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology
- NSubstitute documentation - https://nsubstitute.github.io/

---

## 9. Testing Best Practices

### Definitions

**Core Definition**
Testing best practices are guidelines for writing tests that are reliable, maintainable, fast, and effective at catching defects.

**Technical Definition**
Best practices encompass the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), the Arrange-Act-Assert structure, descriptive naming, appropriate use of test doubles, layering tests according to the test pyramid, and integrating mutation testing for suite quality .

**Beginner-Friendly Explanation**
Best practices are "the habits that make tests helpful instead of painful." Good tests are fast, independent, and easy to understand. Bad tests are slow, flaky, and hard to maintain.

---

### Purposes

- To keep tests fast and reliable.
- To make failures easy to diagnose.
- To reduce test maintenance cost.
- To ensure tests actually verify behavior.
- To support continuous integration.
- To build a test suite that developers trust.

---

### Guidelines and Structure

#### FIRST Principles

```
F — Fast: Tests run in milliseconds.
I — Isolated: No shared state; no execution-order dependency.
R — Repeatable: Same result every time, on any machine.
S — Self-validating: Pass/fail without manual inspection.
T — Timely: Written at the right time (before or alongside code).
```

#### Arrange-Act-Assert (AAA)

```csharp
[Fact]
public void Add_TwoNumbers_ReturnsSum()
{
    // Arrange
    var calculator = new Calculator();

    // Act
    var result = calculator.Add(2, 3);

    // Assert
    Assert.Equal(5, result);
}
```

- **Arrange** — Set up the system under test and its dependencies.
- **Act** — Execute the method being tested.
- **Assert** — Verify the expected outcome.

#### Naming Conventions

```csharp
// Good: Method_Scenario_ExpectedBehavior
[Fact]
public void Add_EmptyString_ReturnsZero() { ... }

// Bad: vague name
[Fact]
public void Test1() { ... }
```

#### Test Doubles

- Use stubs for queries (return data).
- Use mocks for commands (verify interactions).
- Prefer fakes for stateful dependencies.
- Avoid over-mocking; test behavior, not implementation.

#### Test Pyramid

- Many unit tests, fewer integration tests, very few E2E tests.
- Run unit tests on every commit; integration tests on every PR; E2E tests on merge/release.

#### Coverage and Mutation Testing

- Track line and branch coverage.
- Use Stryker.NET to verify that tests actually detect faults.
- Do not chase 100% coverage blindly.

#### CI/CD Integration

```bash
dotnet test --logger "trx;LogFileName=results.trx" --collect:"XPlat Code Coverage"
```

- Run tests automatically on every push.
- Publish test results and coverage reports.
- Fail the build on test failures or coverage regressions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Well-Structured Test Suite

```csharp
public class OrderServiceTests
{
    private readonly IOrderRepository _repo;
    private readonly OrderService _sut;

    public OrderServiceTests()
    {
        _repo = Substitute.For<IOrderRepository>();
        _sut = new OrderService(_repo);
    }

    [Fact]
    public void PlaceOrder_ValidOrder_SavesAndReturnsId()
    {
        // Arrange
        var order = new Order { Total = 100m };
        _repo.Save(Arg.Any<Order>()).Returns(42);

        // Act
        var id = _sut.PlaceOrder(order);

        // Assert
        Assert.Equal(42, id);
        _repo.Received(1).Save(order);
    }

    [Fact]
    public void PlaceOrder_ZeroTotal_ThrowsArgumentException()
    {
        var order = new Order { Total = 0m };

        Assert.Throws<ArgumentException>(() => _sut.PlaceOrder(order));
        _repo.DidNotReceive().Save(Arg.Any<Order>());
    }
}
```

**Expected Output**

```
Passed: PlaceOrder_ValidOrder_SavesAndReturnsId
Passed: PlaceOrder_ZeroTotal_ThrowsArgumentException
```

**Why this output occurs**

- The tests are isolated (fresh substitutes per test).
- The mocks verify the expected interactions.
- The exception test verifies the guard clause.

---

### Real-World Cases

- **Enterprise applications** — Layered test suites with CI gates.
- **Open-source libraries** — Comprehensive unit tests and mutation testing.
- **Microservices** — Contract tests and integration tests.

---

### References

- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- Arrange-Act-Assert — Martin Fowler - https://martinfowler.com/bliki/GivenWhenThen.html
- Stryker.NET mutation testing - https://stryker-mutator.io/docs/stryker-net/introduction/
- NSubstitute documentation - https://nsubstitute.github.io/
- Testcontainers for .NET - https://dotnet.testcontainers.org/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| SpecFlow | End-of-life; Reqnroll is the community-maintained successor . |
| EF Core InMemory provider | Not a relational database; unsuitable for integration tests that rely on relational semantics. |
| `[SetUp]` / `[TearDown]` (xUnit) | Not available; use constructors and `IDisposable` . |
| Coverage as a quality metric | Necessary but not sufficient; pair with mutation testing . |
| Stryker.NET mutation score | Configurable thresholds; common defaults are High 80%, Low 60%, Break 50% . |
| Playwright hard-coded waits | Avoid; use auto-waiting and web-first assertions . |
| Testcontainers | Requires Docker; not suitable for environments without container support. |
| MSTest | Historically weaker CI integration; improved in .NET Core era . |
| xUnit parallel execution | Default for test collections; requires isolation . |
| Static mutable state in tests | Causes order dependencies and flaky tests . |

---

## References (Consolidated)

- .NET testing documentation — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/
- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Integration testing in ASP.NET Core - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- xUnit.net documentation - https://xunit.net/
- NUnit documentation - https://docs.nunit.org/
- MSTest documentation - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest
- TUnit documentation - https://github.com/thomhurst/TUnit
- The Practical Test Pyramid — Martin Fowler - https://martinfowler.com/articles/practical-test-pyramid.html
- Test pyramid — Mike Cohn (Succeeding with Agile, 2009) - https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid
- TDD — Martin Fowler - https://martinfowler.com/bliki/TestDrivenDevelopment.html
- TDD with xUnit in .NET (community) - https://github.com/thapaliyabikendra/ai-artifacts/blob/main/.claude/knowledge/implementations/dotnet/xunit-tdd.md
- BDD — Martin Fowler - https://martinfowler.com/bliki/BehaviorDrivenDevelopment.html
- Reqnroll documentation - https://docs.reqnroll.net/
- SpecFlow (EOL) - https://specflow.org/
- Coverlet documentation - https://github.com/coverlet-coverage/coverlet
- ReportGenerator - https://github.com/danielpalme/ReportGenerator
- Stryker.NET mutation testing - https://stryker-mutator.io/docs/stryker-net/introduction/
- Mutation testing in C# — Xebia - https://xebia.dev/blog/mutation-testing-in-c/
- NSubstitute documentation - https://nsubstitute.github.io/
- Moq documentation - https://github.com/moq/moq
- FakeItEasy documentation - https://fakeiteasy.github.io/
- Testcontainers for .NET - https://dotnet.testcontainers.org/
- Respawn — https://github.com/jbogard/Respawn
- Playwright for .NET - https://playwright.dev/dotnet/
- Selenium documentation - https://www.selenium.dev/documentation/
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- Arrange-Act-Assert — IEEE study - https://ieeexplore.ieee.org/document/10494733
- xUnit Theory and data-driven tests - https://xunit.net/docs/getting-started/netcore/cmdline
- BrowserStack — NUnit vs xUnit vs MSTest - https://www.browserstack.com/guide/nunit-vs-xunit-vs-mstest
- BrowserStack — Best C# Testing Frameworks 2026 - https://www.browserstack.com/guide/c-sharp-testing-frameworks
- SSW.Rules — Testing tools - https://www.ssw.com.au/rules/testing-tools