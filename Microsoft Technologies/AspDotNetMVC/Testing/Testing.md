# ASP.NET Core Testing Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the testing ecosystem around it: **xUnit** (the recommended framework), **WebApplicationFactory** for integration tests, **Playwright** for end-to-end tests, and the **Test Pyramid** as the guiding strategy. Testing is not an afterthought — it is a first-class engineering discipline. A service without tests is a service that cannot be safely refactored, and a test suite without a strategy (all unit tests, or all E2E tests) is either too slow or too shallow. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Software testing is the systematic process of verifying that an application behaves as expected, preventing regressions, documenting behaviour, and enabling confident change. |
| **Technical Definition** | Testing in ASP.NET Core spans **unit tests** (isolated class/method verification with mocked dependencies), **integration tests** (in-process HTTP verification using `WebApplicationFactory<TEntryPoint>` and `TestServer`), and **end-to-end tests** (browser automation with Playwright). The **Test Pyramid** guides the distribution: many fast unit tests, fewer integration tests, and very few E2E tests. **Test-Driven Development (TDD)** inverts the workflow: write a failing test (Red), write minimal code to pass (Green), then refactor while keeping tests green. |
| **Beginner-Friendly Explanation** | Testing is the **safety net for your code**. Unit tests check small pieces in isolation — like testing that a calculator adds correctly. Integration tests check that pieces work together — like testing that an API endpoint returns the right JSON. End-to-end tests check the whole system from the user's perspective — like a robot clicking through your app in a browser. The test pyramid says: write lots of fast unit tests, some integration tests, and only a few slow E2E tests. TDD says: write the test *before* the code, so the test drives the design. |

### Key Characteristics

- **Fast feedback** – Unit tests run in milliseconds; integration tests in seconds; E2E tests in minutes.
- **Isolation** – Unit tests do not touch databases, file systems, or networks; integration tests use test doubles or in-memory databases.
- **Repeatability** – A good test produces the same result every time, regardless of order or environment.
- **Self-checking** – Tests automatically report pass/fail; no manual inspection.
- **Living documentation** – Well-named tests describe what the system does.
- **Regression protection** – Tests catch unintended changes before they reach production.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- A test framework: **xUnit** (recommended), NUnit, or MSTest.
- A mocking library: **NSubstitute** or Moq.
- `Microsoft.AspNetCore.Mvc.Testing` package for integration tests.
- `Microsoft.Playwright` package (and browser binaries) for E2E tests.

### Related Programming Areas

- **Dependency injection** – Enables testability by allowing mock injection.
- **Middleware pipeline** – Integration tests exercise the full pipeline.
- **API controllers / Minimal APIs** – The endpoints under test.
- **Testcontainers** – Spins up real databases in Docker for integration tests.
- **CI/CD** – Tests run automatically on every commit.

### Core Concepts / Features

1. **Testing** – The discipline, the benefits, and the tooling.
2. **Test types** – Unit tests, integration tests, and end-to-end tests.
3. **Test pyramid** – The distribution strategy and its rationale.
4. **Test-driven development** – Red-Green-Refactor and its benefits.
5. **Testing best practices** – Naming, isolation, coverage, and CI integration.

---

## 1. Testing (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Testing is the process of executing software to verify that it meets requirements, behaves correctly, and continues to do so as the codebase evolves. |
| **Technical Definition** | Automated testing in .NET uses test frameworks (xUnit, NUnit, MSTest) to define tests as methods with attributes (`[Fact]`, `[Theory]`). The test runner discovers and executes tests, reporting pass/fail. Tests follow the **Arrange-Act-Assert (AAA)** pattern: set up the scenario, execute the behaviour, verify the outcome. Assertions (`Assert.Equal`, `Assert.True`, `Assert.Throws`) define expected outcomes. |
| **Beginner-Friendly Explanation** | Testing is the **“does it actually work?”** check. Instead of manually clicking through your app every time you change code, you write automated tests that do it for you. They run in seconds and tell you immediately if you broke something. Tests also serve as documentation — they show how the code is supposed to be used. |

### Purposes

- **To** **prevent regressions** — catch bugs before they reach production.
- **To** **document behaviour** — tests describe what the code does and how to use it.
- **To** **enable confident refactoring** — change code with the safety net of tests.
- **To** **improve design** — code that is hard to test is often poorly designed.
- **To** **reduce manual testing cost** — automated tests replace repetitive manual checks.
- **To** **support CI/CD** — tests gate deployments and catch issues early.

### Syntax Rules and Structure

#### General Syntax (xUnit Test)

```csharp
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

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[Fact]` | Marks a parameterless test method. |
| `[Theory]` + `[InlineData]` | Parameterized tests. |
| `Arrange` | Set up the system under test (SUT). |
| `Act` | Execute the behaviour. |
| `Assert` | Verify the outcome. |

#### Test Project Setup

```xml
<!-- Test.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
    <PackageReference Include="xunit" Version="2.*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.*" />
  </ItemGroup>
</Project>
```

**Syntax rules:**

- **Test projects should be separate from production projects** —never ship test code.
- **Use `dotnet test`** to run the full suite.
- **Each test should be independent** —no shared mutable state between tests.

**Constraints and limitations:**

- **Code coverage is a metric, not a goal** —high coverage with weak assertions is worthless.
- **Tests that depend on order or shared state are flaky** —avoid static state and shared fixtures where possible.

### Annotated Complete Step-by-Step Code Example

```csharp
// Production code
public class TemperatureConverter
{
    public double CelsiusToFahrenheit(double celsius) => (celsius * 9 / 5) + 32;
}

// Test code
public class TemperatureConverterTests
{
    [Theory]
    [InlineData(0, 32)]
    [InlineData(100, 212)]
    [InlineData(-40, -40)]
    public void CelsiusToFahrenheit_ReturnsExpected(double celsius, double expected)
    {
        var converter = new TemperatureConverter();
        var actual = converter.CelsiusToFahrenheit(celsius);
        Assert.Equal(expected, actual, precision: 5);
    }
}
```

**Step-by-step explanation:**

1. The SUT (`TemperatureConverter`) has a single method.
2. `[Theory]` with `[InlineData]` runs the test three times with different inputs.
3. Each run follows Arrange (create converter), Act (call method), Assert (compare with expected).
4. `precision: 5` handles floating-point rounding.

**Expected Output:** All three test cases pass; if the formula were wrong, the failing case would identify the exact input.

**Why this output?** The parameterized test covers multiple scenarios without duplicating code; the precise assertion avoids false failures from floating-point arithmetic.

### Real-World Cases

- **Business logic:** Pricing rules, tax calculations, discount application.
- **Validation:** Input validation rules, format checks.
- **Mapping:** Entity-to-DTO mapping correctness.

### References

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit documentation – https://xunit.net/

---

## 2. Test Types

### 2.1 Unit Tests

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A unit test verifies the behaviour of a single, isolated unit of code (usually a method or class) without touching external dependencies. |
| **Technical Definition** | Unit tests instantiate the **System Under Test (SUT)** directly, replacing its dependencies with **test doubles** (mocks, stubs, fakes) via constructor injection or a mocking library (NSubstitute, Moq). They do not use the ASP.NET Core pipeline, databases, file systems, or network. They run in **milliseconds** and are the foundation of the test pyramid. |
| **Beginner-Friendly Explanation** | A unit test is the **“test one small thing”** test. If you have a method that calculates a discount, the unit test passes in fake inputs and checks the output — without needing a database, a web server, or a browser. It is fast, isolated, and tells you exactly what broke. |

#### Purposes

- **To** verify **business logic** in isolation.
- **To** catch **regressions** in individual methods or classes.
- **To** **document** the expected behaviour of a unit.
- **To** enable **fast feedback** during development.
- **To** drive **testable design** (dependency injection, single responsibility).

#### Syntax Rules and Structure

```csharp
public class OrderServiceTests
{
    [Fact]
    public async Task CalculateTotal_AppliesDiscount_WhenCustomerIsPremium()
    {
        // Arrange
        var repo = Substitute.For<IOrderRepository>();
        repo.GetItemsAsync(Arg.Any<int>(), Arg.Any<CancellationToken>())
            .Returns(new[] { new OrderItem(100m) });

        var service = new OrderService(repo);

        // Act
        var total = await service.CalculateTotalAsync(1, isPremium: true, CancellationToken.None);

        // Assert
        Assert.Equal(90m, total);   // 10% discount
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Substitute.For<T>()` | Creates a mock (NSubstitute). |
| `.Returns(...)` | Configures the mock's behaviour. |
| `Arg.Any<T>()` | Matches any argument value. |
| `Assert.Equal` | Verifies the expected outcome. |

**Syntax rules:**

- **Unit tests must not touch infrastructure** —no `DbContext`, no `HttpClient`, no file system.
- **Use interfaces for dependencies** so they can be mocked.
- **One behaviour per test** —avoid asserting multiple unrelated things.
- **Name tests with `Method_Scenario_ExpectedOutcome`** for clarity.

**Constraints and limitations:**

- **Mocking too much** can make tests brittle and coupled to implementation details.
- **Unit tests cannot verify wiring** —whether DI is configured correctly, whether middleware runs, whether the database schema matches. That is what integration tests are for.

#### Annotated Complete Step-by-Step Code Example

```csharp
public class PriceCalculatorTests
{
    [Theory]
    [InlineData(100, 0, 100)]
    [InlineData(100, 10, 90)]
    [InlineData(100, 100, 0)]
    public void ApplyDiscount_ReturnsCorrectAmount(
        decimal price, decimal discountPercent, decimal expected)
    {
        var calculator = new PriceCalculator();
        var result = calculator.ApplyDiscount(price, discountPercent);
        Assert.Equal(expected, result);
    }

    [Fact]
    public void ApplyDiscount_NegativePercent_Throws()
    {
        var calculator = new PriceCalculator();
        Assert.Throws<ArgumentOutOfRangeException>(
            () => calculator.ApplyDiscount(100, -1));
    }
}
```

**Expected Output:** Valid discounts return the correct amount; negative percentages throw `ArgumentOutOfRangeException`.

**Why this output?** The theory covers multiple valid cases; the fact verifies the guard clause.

#### Real-World Cases

- **Domain logic:** Discount rules, pricing, tax.
- **Validation:** Email format, password strength.
- **Mapping:** DTO ↔ entity conversions.

#### References

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit test controller logic in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/testing

---

### 2.2 Integration Tests

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An integration test verifies that multiple components of the application work together correctly — typically by exercising the full HTTP pipeline, including routing, middleware, model binding, and serialization. |
| **Technical Definition** | ASP.NET Core integration tests use **`WebApplicationFactory<TEntryPoint>`** to boot the real application in-process with a **`TestServer`**. `CreateClient()` returns an `HttpClient` that sends requests through the full middleware pipeline without network overhead. The `WebApplicationFactory` can be customized to replace services (e.g., swap the database for an in-memory provider or Testcontainer). Tests follow the same Arrange-Act-Assert pattern but operate on HTTP responses. |
| **Beginner-Friendly Explanation** | An integration test is the **“test the whole plumbing”** test. It sends an HTTP request to your app — through routing, middleware, controllers, and JSON serialization — and checks the response. It catches bugs that unit tests miss: a missing `[Authorize]`, a misconfigured route, a serialization issue, or a broken DI registration. |

#### Purposes

- **To** verify **end-to-end HTTP behaviour** —request → middleware → controller → response.
- **To** catch **wiring bugs** —DI configuration, routing, filters, serialization.
- **To** test **authentication and authorization** in the real pipeline.
- **To** validate **database integration** (with in-memory or Testcontainers).
- **To** ensure **middleware order** is correct.

#### Syntax Rules and Structure

```csharp
public class ProductsApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductsApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_ReturnsSuccessAndJson()
    {
        var response = await _client.GetAsync("/api/products");
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json",
            response.Content.Headers.ContentType?.MediaType);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `WebApplicationFactory<Program>` | Boots the real app in-process. |
| `IClassFixture<T>` | Shares the factory across tests in the class. |
| `CreateClient()` | Returns an `HttpClient` that talks to the `TestServer`. |
| `EnsureSuccessStatusCode()` | Asserts 2xx status. |

**Syntax rules:**

- **Use `WebApplicationFactory<Program>`** —`Program` is the entry point (top-level statements are supported in .NET 6+).
- **Custom factories** inherit from `WebApplicationFactory<T>` and override `ConfigureWebHost` to replace services.
- **Separate unit and integration tests into different projects** —they have different dependencies and run at different speeds.
- **Use `IClassFixture`** to share the factory and avoid re-bootstrapping per test.

**Constraints and limitations:**

- **Integration tests are slower** than unit tests —limit them to critical infrastructure scenarios.
- **The in-memory database is not the same as production** —use Testcontainers for realistic database behaviour.
- **Authentication requires configuration** —use a test authentication handler or inject a fake `ClaimsPrincipal`.

#### Annotated Complete Step-by-Step Code Example

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Replace the real DbContext with an in-memory database
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor is not null) services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseInMemoryDatabase("TestDb"));
        });
    }
}

public class OrdersApiTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;

    public OrdersApiTests(CustomWebApplicationFactory factory)
        => _client = factory.CreateClient();

    [Fact]
    public async Task CreateOrder_Returns201WithLocation()
    {
        var response = await _client.PostAsJsonAsync("/api/orders",
            new { customerId = 1, total = 99.99m });

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        Assert.NotNull(response.Headers.Location);
    }
}
```

**Step-by-step explanation:**

1. `CustomWebApplicationFactory` overrides `ConfigureWebHost` to swap the database.
2. `CreateClient()` returns an `HttpClient` connected to the in-process `TestServer`.
3. `PostAsJsonAsync` sends a JSON request through the full pipeline.
4. The test asserts `201 Created` and the presence of a `Location` header.

**Expected Output:** The POST returns `201 Created` with a `Location` header; the response body contains the created order.

**Why this output?** The request traverses routing, model binding, the controller, and JSON serialization — verifying the entire HTTP pipeline.

#### Real-World Cases

- **API endpoint verification:** Status codes, content types, headers.
- **Authentication/authorization:** `401` vs. `403` with real policies.
- **Middleware behaviour:** CORS, exception handling, rate limiting.
- **Database integration:** Repository queries against a test database.

#### References

- Integration tests in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- `WebApplicationFactory<TEntryPoint>` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#basic-tests-with-the-default-webapplicationfactory

---

### 2.3 End-to-End Tests

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An end-to-end (E2E) test verifies the entire application from the user's perspective — including the browser, front-end, back-end, and database — simulating real user interactions. |
| **Technical Definition** | E2E tests use browser automation tools like **Playwright for .NET** to launch a real browser, navigate to the application, interact with UI elements (click, type, select), and assert on the visible result. They exercise the full stack: client-side JavaScript, HTTP requests, server-side processing, and persistence. They are **slow**, **fragile**, and **expensive** —hence the small proportion in the test pyramid. |
| **Beginner-Friendly Explanation** | An E2E test is the **“robot user”** test. A real browser opens your app, clicks buttons, fills forms, and checks that the right page appears. It catches integration issues that unit and integration tests miss — broken JavaScript, CSS that hides buttons, redirects that fail. But it is slow and brittle, so you write only a few for the most critical user journeys. |

#### Purposes

- **To** verify **critical user journeys** —login, checkout, registration.
- **To** catch **full-stack bugs** —front-end/back-end mismatches, redirect loops.
- **To** validate **browser behaviour** —JavaScript, cookies, redirects.
- **To** provide **confidence** before releases.
- **To** test **SPA + API** integration in a real browser.

#### Syntax Rules and Structure

```csharp
using Microsoft.Playwright;
using Microsoft.Playwright.Xunit;

public class CheckoutFlowTests : PageTest
{
    [Fact]
    public async Task User_Can_Add_To_Cart_And_Checkout()
    {
        await Page.GotoAsync("https://localhost:5001");
        await Page.ClickAsync("text=Products");
        await Page.ClickAsync("text=Laptop");
        await Page.ClickAsync("button:has-text('Add to Cart')");
        await Page.ClickAsync("text=Checkout");
        await Expect(Page).ToHaveURLAsync(new Regex("/checkout"));
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `PageTest` | Playwright's base class for page-based tests. |
| `Page.GotoAsync(...)` | Navigates to a URL. |
| `Page.ClickAsync(...)` | Clicks an element by selector. |
| `Expect(Page).ToHaveURLAsync(...)` | Asserts the current URL. |

**Syntax rules:**

- **E2E tests run against a running application** —start the app before the tests (e.g., `dotnet run` in CI).
- **Use page object models** to keep tests maintainable —don't hard-code selectors in every test.
- **Run E2E tests in CI** with headless browsers (Playwright supports headless mode).
- **Limit E2E tests to critical paths** —not every feature needs one.

**Constraints and limitations:**

- **Slow** —each test launches a browser and navigates pages; a suite can take minutes.
- **Fragile** —UI changes, timing issues, and network flakiness cause false failures.
- **Expensive** —browser automation requires more infrastructure and maintenance than unit tests.

#### Annotated Complete Step-by-Step Code Example

```csharp
public class LoginFlowTests : PageTest
{
    [Fact]
    public async Task ValidCredentials_RedirectToDashboard()
    {
        await Page.GotoAsync("https://localhost:5001/login");
        await Page.FillAsync("#email", "alice@example.com");
        await Page.FillAsync("#password", "P@ssw0rd!");
        await Page.ClickAsync("button[type=submit]");

        await Expect(Page).ToHaveURLAsync(new Regex("/dashboard"));
        await Expect(Page.Locator("h1")).ToHaveTextAsync("Dashboard");
    }

    [Fact]
    public async Task InvalidCredentials_ShowError()
    {
        await Page.GotoAsync("https://localhost:5001/login");
        await Page.FillAsync("#email", "alice@example.com");
        await Page.FillAsync("#password", "wrong");
        await Page.ClickAsync("button[type=submit]");

        await Expect(Page.Locator(".validation-summary-errors"))
            .ToBeVisibleAsync();
    }
}
```

**Expected Output:** Valid credentials redirect to `/dashboard` with the “Dashboard” heading; invalid credentials show a validation error.

**Why this output?** Playwright drives a real browser through the login form, exercising the full stack including client-side validation, server-side authentication, and redirects.

#### Real-World Cases

- **Checkout flows:** Add to cart → payment → confirmation.
- **Login/registration:** Credential validation and redirects.
- **Critical CRUD:** Create → read → update → delete through the UI.

#### References

- Integration tests in ASP.NET Core (E2E guidance) – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Playwright for .NET – https://playwright.dev/dotnet/

---

## 3. Test Pyramid

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Test Pyramid is a metaphor for structuring automated tests: many fast unit tests at the base, fewer integration tests in the middle, and very few slow E2E tests at the top. |
| **Technical Definition** | Popularized by **Mike Cohn** and expanded by **Martin Fowler**, the test pyramid guides test distribution to balance **speed**, **cost**, and **confidence**. Unit tests are cheap, fast, and isolate failures; integration tests are slower but verify wiring; E2E tests are slowest but provide the highest confidence for critical journeys. A common target is **70% unit, 20% integration, 10% E2E**, though the exact ratio depends on the project. |
| **Beginner-Friendly Explanation** | The test pyramid is the **“don't put all your tests at the top”** rule. If you write mostly E2E tests, your suite becomes slow and flaky. If you write only unit tests, you miss integration bugs. The pyramid says: **lots of small, fast tests** at the bottom; **some medium tests** in the middle; **a few big tests** at the top for the most important paths. |

### Purposes

- **To** **balance speed and confidence** —fast unit tests catch most bugs; E2E tests catch the critical paths.
- **To** **reduce test suite runtime** —unit tests run in milliseconds; E2E tests in minutes.
- **To** **isolate failures** —unit tests pinpoint the broken method; E2E failures require investigation.
- **To** **guide investment** —write more tests where they are cheapest and most effective.
- **To** **avoid the “ice cream cone” anti-pattern** —many slow E2E tests and few unit tests.

### Syntax Rules and Structure

#### The Pyramid Shape

```text
        /\
       /  \      E2E (5–10%)
      /----\     — slow, brittle, high confidence
     /      \
    /--------\   Integration (15–25%)
   /          \  — moderate speed, verifies wiring
  /------------\
 /              \ Unit (70–80%)
/----------------\ — fast, isolated, cheap
```

**Component breakdown:**

| Layer | Proportion | Speed | What It Verifies |
|-------|------------|-------|------------------|
| **Unit** | 70–80% | Milliseconds | Individual methods/classes in isolation. |
| **Integration** | 15–25% | Seconds | HTTP pipeline, DI, database, middleware. |
| **E2E** | 5–10% | Minutes | Full user journeys in a real browser. |

**Syntax rules:**

- **Start with unit tests** —they are the fastest and cheapest.
- **Add integration tests for wiring** —especially endpoints, auth, and middleware.
- **Reserve E2E tests for critical paths** —login, checkout, payment.
- **Do not invert the pyramid** —a suite of mostly E2E tests is slow and fragile.

**Constraints and limitations:**

- **The exact percentages are not dogma** —Martin Fowler warns against treating 70/20/10 as a target; the goal is a balanced portfolio, not a ratio.
- **Some systems need more integration tests** —e.g., API-heavy services with little business logic.

### Annotated Code Example (Test Distribution in Practice)

```csharp
// Unit test (fast, isolated)
[Fact]
public void ApplyDiscount_10Percent_Returns90() { ... }

// Integration test (HTTP pipeline)
[Fact]
public async Task GetProducts_ReturnsJson() { ... }

// E2E test (critical journey)
[Fact]
public async Task Checkout_CompleteFlow_Succeeds() { ... }
```

**Expected Output:** The unit test runs in milliseconds, the integration test in ~100ms, and the E2E test in seconds. The suite as a whole provides fast feedback for most changes and high confidence for critical paths.

**Why this output?** Each layer serves a different purpose; together they form a balanced portfolio.

### Real-World Cases

- **Web APIs:** Heavy unit tests for business logic; integration tests for endpoints; few E2E tests for critical flows.
- **SPAs:** Unit tests for components; integration tests for API calls; E2E for login and checkout.
- **Microservices:** Unit tests per service; integration tests for service boundaries; E2E for cross-service workflows.

### References

- The Practical Test Pyramid (Martin Fowler) – https://martinfowler.com/articles/practical-test-pyramid.html
- Test Pyramid origins (Mike Cohn) – https://martinfowler.com/bliki/TestPyramid.html
- Testing Pyramid: Complete Guide – https://testsigma.com/blog/testing-pyramid/

---

## 4. Test-Driven Development (TDD)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Test-Driven Development is a software development approach where tests are written **before** the production code, following the **Red-Green-Refactor** cycle. |
| **Technical Definition** | TDD follows three steps: **Red** —write a failing test that defines the desired behaviour; **Green** —write the minimal production code to make the test pass; **Refactor** —improve the code while keeping all tests green. In .NET, TDD is typically practiced with **xUnit**, **NSubstitute**, and **`dotnet test`**. The cycle is short —often minutes —and repeats for each small increment of functionality. |
| **Beginner-Friendly Explanation** | TDD is the **“test first, code second”** approach. Instead of writing code and then testing it, you write a test that says “this method should return 5 when given 2 and 3.” The test fails (Red) because the method doesn't exist. You write the simplest code to make it pass (Green). Then you clean up the code (Refactor) while the test keeps you safe. It feels backwards at first, but it produces cleaner, more testable designs. |

### Purposes

- **To** **drive design** —writing tests first forces you to think about interfaces and usage before implementation.
- **To** **ensure test coverage** —every piece of code is written because a test demanded it.
- **To** **produce maintainable code** —the refactor step keeps the code clean.
- **To** **reduce debugging time** —failures point to the most recent change.
- **To** **document requirements** —each test is a specification of behaviour.

### Syntax Rules and Structure

#### The Red-Green-Refactor Cycle

```text
RED   → Write a failing test (the feature does not exist yet)
  ↓
GREEN → Write the minimal code to make the test pass
  ↓
REFACTOR → Improve the code while keeping tests green
  ↓
(repeat)
```

**Component breakdown:**

| Phase | Action | Outcome |
|-------|--------|---------|
| **Red** | Write a test for the next small behaviour. | Test fails —the code is missing or wrong. |
| **Green** | Write just enough code to pass. | Test passes —the behaviour exists. |
| **Refactor** | Clean up duplication, improve names, simplify. | Tests still pass —behaviour unchanged. |

#### General Syntax (TDD in .NET with xUnit)

```csharp
// Step 1: Red — write a failing test
[Fact]
public void AddItem_WhenCalled_IncreasesItemCount()
{
    var cart = new Cart();
    var item = new Item { Id = 1, Price = 10m };
    cart.AddItem(item);
    Assert.Equal(1, cart.ItemCount);
}
// The test fails — Cart and Item do not exist.

// Step 2: Green — write minimal code to pass
public class Cart
{
    private readonly List<Item> _items = new();
    public int ItemCount => _items.Count;
    public void AddItem(Item item) => _items.Add(item);
}

// Step 3: Refactor — improve design while tests stay green
public class Cart
{
    private readonly List<Item> _items = new();
    public IReadOnlyList<Item> Items => _items;
    public int ItemCount => _items.Count;
    public void AddItem(Item item)
    {
        ArgumentNullException.ThrowIfNull(item);
        _items.Add(item);
    }
}
```

**Syntax rules:**

- **Write the test first** —do not write production code until a test demands it.
- **Keep the Green step minimal** —do not add features that no test requires.
- **Refactor only when all tests pass** —never refactor on Red.
- **Run the test suite frequently** —after every Green and Refactor step.

**Constraints and limitations:**

- **TDD is not a silver bullet** —it works best for well-understood, incremental features.
- **Over-mocking** in TDD can produce tests that verify implementation details rather than behaviour.
- **TDD requires discipline** —skipping the Refactor step leads to messy code despite good coverage.

### Annotated Complete Step-by-Step Code Example

```csharp
// Feature: A password validator that requires at least 8 characters.

// RED: Write the first failing test
[Fact]
public void IsValid_ShortPassword_ReturnsFalse()
{
    var validator = new PasswordValidator();
    Assert.False(validator.IsValid("abc"));
}

// GREEN: Minimal implementation
public class PasswordValidator
{
    public bool IsValid(string password) => password.Length >= 8;
}

// RED: Next test — exactly 8 characters is valid
[Fact]
public void IsValid_EightCharacters_ReturnsTrue()
{
    var validator = new PasswordValidator();
    Assert.True(validator.IsValid("abcdefgh"));
}

// GREEN: The current implementation already passes — no code change needed.

// RED: Next test — null throws
[Fact]
public void IsValid_Null_Throws()
{
    var validator = new PasswordValidator();
    Assert.Throws<ArgumentNullException>(() => validator.IsValid(null!));
}

// GREEN: Add guard clause
public bool IsValid(string password)
{
    ArgumentNullException.ThrowIfNull(password);
    return password.Length >= 8;
}

// REFACTOR: The code is already clean — tests remain green.
```

**Expected Output:** The test suite grows one behaviour at a time; each new test fails first, then passes with minimal code. The final `IsValid` method handles all tested cases.

**Why this output?** TDD builds the implementation incrementally, with each test driving one small piece of behaviour.

### Real-World Cases

- **New feature development:** Build features incrementally with TDD.
- **Bug fixes:** Write a failing test that reproduces the bug, then fix it (Green).
- **Refactoring legacy code:** Add characterization tests first, then refactor safely.

### References

- .NET Test-Driven Development (Compile-N-Run) – https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/5-net-test-driven-development.mdx
- TDD with xUnit in .NET – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/skills/tdd/SKILL.md
- xUnit TDD cycle – https://xunit.net/docs/getting-started/netcore/cmdline

---

## 5. Testing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Testing best practices are guidelines for writing tests that are fast, reliable, maintainable, and genuinely useful — not just a coverage metric. |
| **Technical Definition** | Best practices span **naming** (`Method_Scenario_ExpectedOutcome`), **isolation** (no shared mutable state, no test ordering dependencies), **the AAA pattern**, **one assertion per test** (or one logical assertion), **avoiding infrastructure in unit tests**, **using `IClassFixture` for shared integration setup**, **separating unit and integration test projects**, and **running tests in CI**. |
| **Beginner-Friendly Explanation** | Best practices are the **“do's and don'ts”** for testing —they help you avoid flaky tests, tests that pass but don't actually verify anything, and test suites that are so slow nobody runs them. |

### Purposes

- **To** keep tests **readable** —a failing test should immediately tell you what broke.
- **To** keep tests **fast** —slow tests discourage running them.
- **To** keep tests **reliable** —flaky tests erode trust.
- **To** keep tests **maintainable** —tests that are harder to change than the code are a liability.
- **To** ensure **coverage** is meaningful, not just high.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Naming** | Use `Method_Scenario_ExpectedOutcome` (e.g., `Add_NegativeNumber_Throws`). | Name tests `Test1`, `Test2`. |
| **Structure** | Follow Arrange-Act-Assert. | Mix setup, execution, and assertions randomly. |
| **Assertions** | One logical assertion per test. | Assert five unrelated things in one test. |
| **Isolation** | Each test sets up its own state. | Share mutable state between tests. |
| **Speed** | Unit tests in milliseconds; integration in seconds; E2E only for critical paths. | Put database calls in unit tests. |
| **Mocking** | Mock interfaces for external dependencies. | Mock everything —including the SUT itself. |
| **Integration tests** | Use `WebApplicationFactory` + `IClassFixture` for shared setup. | Re-bootstrap the app for every test method. |
| **Database** | Use in-memory or Testcontainers for integration tests. | Hit the production database in tests. |
| **Projects** | Separate `MyApp.Tests.Unit` and `MyApp.Tests.Integration`. | Mix all tests in one project with conflicting dependencies. |
| **CI** | Run `dotnet test` on every commit; fail the build on test failure. | Skip tests in CI because "they're slow." |

### Annotated Code Example (Best-Practice Test Suite)

```csharp
// Unit test — fast, isolated, clear name
public class DiscountCalculatorTests
{
    [Theory]
    [InlineData(100, 0, 100)]
    [InlineData(100, 10, 90)]
    public void ApplyDiscount_ValidPercent_ReturnsDiscountedPrice(
        decimal price, decimal percent, decimal expected)
    {
        var sut = new DiscountCalculator();
        var result = sut.ApplyDiscount(price, percent);
        Assert.Equal(expected, result);
    }
}

// Integration test — shared factory, real HTTP pipeline
public class ProductsApiTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;

    public ProductsApiTests(CustomWebApplicationFactory factory)
        => _client = factory.CreateClient();

    [Fact]
    public async Task GetProducts_Returns200WithJson()
    {
        var response = await _client.GetAsync("/api/products");
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json",
            response.Content.Headers.ContentType?.MediaType);
    }
}
```

**Why this follows best practices:**

- **Clear names** describe the method, scenario, and expected outcome.
- **Theory** parameterizes multiple cases without duplication.
- **Integration test uses `IClassFixture`** to share the factory.
- **Unit and integration tests are separate** —different projects, different speeds.
- **Assertions are focused** —each test verifies one logical outcome.

### Real-World Cases

- **CI pipelines:** Tests run on every PR; unit tests provide fast feedback.
- **Refactoring legacy code:** Characterization tests protect existing behaviour.
- **API contracts:** Integration tests verify the HTTP contract before deployment.

### References

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Integration tests in ASP.NET Core (best practices) – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- The Practical Test Pyramid (Martin Fowler) – https://martinfowler.com/articles/practical-test-pyramid.html

---

## Summary Table: Testing Fundamentals at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Testing** | Verify behaviour, prevent regressions | xUnit `[Fact]`, `[Theory]`, `Assert` | Every production codebase |
| **Unit tests** | Test single units in isolation | SUT + mocks (NSubstitute, Moq) | Business logic, validation, mapping |
| **Integration tests** | Test HTTP pipeline + wiring | `WebApplicationFactory<Program>`, `HttpClient` | API endpoints, auth, middleware |
| **End-to-end tests** | Test full user journeys | Playwright for .NET | Critical paths (login, checkout) |
| **Test pyramid** | Balance speed and confidence | 70% unit, 20% integration, 10% E2E (approximate) | Test strategy planning |
| **TDD** | Test-first development | Red-Green-Refactor with xUnit | Incremental feature development |
| **Best practices** | Fast, reliable, maintainable tests | AAA, clear names, isolation, CI | Every test suite |

---

## References (Full List)

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit test controller logic in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/testing
- Integration tests in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- `WebApplicationFactory<TEntryPoint>` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#basic-tests-with-the-default-webapplicationfactory
- xUnit documentation – https://xunit.net/
- Playwright for .NET – https://playwright.dev/dotnet/
- The Practical Test Pyramid (Martin Fowler) – https://martinfowler.com/articles/practical-test-pyramid.html
- Test Pyramid origins (Mike Cohn) – https://martinfowler.com/bliki/TestPyramid.html
- .NET Test-Driven Development (Compile-N-Run) – https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/5-net-test-driven-development.mdx
- TDD with xUnit in .NET – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/skills/tdd/SKILL.md
- Test Strategy Guide (distribution percentages) – https://github.com/…/test-strategy.md
- Testing Pyramid: Complete Guide – https://testsigma.com/blog/testing-pyramid/