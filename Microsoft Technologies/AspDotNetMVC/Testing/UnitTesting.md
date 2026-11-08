# ASP.NET Core Unit Testing: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **unit testing in ASP.NET Core** (.NET 6–10). Three test frameworks dominate the .NET ecosystem: **xUnit** (recommended by Microsoft and used by the ASP.NET Core team itself), **NUnit** (mature, feature-rich), and **MSTest** (Microsoft's original, now v3+ with many improvements). Two mocking libraries dominate: **Moq** (widely used, but note the **SponsorLink controversy in 2023** that drove many teams to alternatives) and **NSubstitute** (cleaner syntax, no sponsorship issues). Unit tests target isolated behavior — they never touch databases, HTTP, file systems, or real clocks. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Unit testing is the practice of verifying the behavior of a single, isolated unit of code — usually a method or class — by controlling its inputs and asserting on its outputs, without touching external infrastructure. |
| **Technical Definition** | A unit test instantiates the **System Under Test (SUT)** directly, replaces its dependencies with **test doubles** (mocks, stubs, fakes) via constructor injection or a mocking library, executes the method under test, and verifies the outcome with **assertions**. Unit tests run in milliseconds, have no I/O, and are deterministic. They follow the **Arrange-Act-Assert (AAA)** pattern. Test frameworks (xUnit, NUnit, MSTest) provide attributes for discovery and assertion libraries; mocking libraries (Moq, NSubstitute) provide test doubles. |
| **Beginner-Friendly Explanation** | A unit test is the **"check one small thing"** test. If you have a `DiscountCalculator.ApplyDiscount` method, the unit test passes fake inputs and checks the output — without a database, a web server, or a real clock. It runs in milliseconds and tells you exactly what broke when it fails. Unit tests are the foundation of the test pyramid because they are the fastest and cheapest way to catch bugs. |

### Key Characteristics

- **Isolated** – No database, network, file system, or real time.
- **Fast** – Runs in milliseconds; thousands can run in seconds.
- **Deterministic** – Same result every time, regardless of environment or order.
- **Self-checking** – Automatically reports pass/fail.
- **Focused** – Tests one behavior per method.
- **Named descriptively** – `Method_Scenario_ExpectedOutcome` tells you what broke.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- A test framework: **xUnit**, **NUnit**, or **MSTest**.
- A mocking library: **Moq** or **NSubstitute**.
- `Microsoft.NET.Test.Sdk` and `xunit.runner.visualstudio` (or equivalent) packages.
- `dotnet test` CLI or a test runner (Visual Studio, Rider, VS Code).

### Related Programming Areas

- **Dependency injection** – Enables testability by allowing mock injection.
- **Test pyramid** – Unit tests form the base; integration and E2E tests sit above.
- **TDD** – Red-Green-Refactor drives unit test development.
- **SOLID principles** – Testable code is usually well-designed code.
- **Testcontainers** – Not for unit tests —integration tests use these.

### Core Concepts / Features

1. **Unit testing** – The discipline and the AAA pattern.
2. **xUnit** – The recommended framework, attributes, and conventions.
3. **NUnit** – The mature alternative with richer attribute model.
4. **MSTest** – Microsoft's original framework, modernized in v3+.
5. **Test attributes** – `[Fact]`, `[Theory]`, `[InlineData]` (and equivalents).
6. **Assertions** – Verifying outcomes with framework-specific assertions.
7. **Mocking** – Test doubles for dependencies.
8. **Moq** – The popular mocking library (with licensing caveat).
9. **NSubstitute** – The cleaner, sponsor-free alternative.
10. **Unit testing best practices** – Naming, isolation, and maintainability.

---

## 1. Unit Testing (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Unit testing is the practice of verifying the behavior of an isolated unit of code by controlling its inputs and asserting on its outputs. |
| **Technical Definition** | A unit test creates the SUT directly, replaces its dependencies with **test doubles** (mocks, stubs, fakes), invokes the method under test, and verifies the result. It follows the **Arrange-Act-Assert** pattern: set up the scenario, execute the behavior, verify the outcome. Tests are discovered by the framework via attributes (`[Fact]`, `[Test]`, `[TestMethod]`) and executed by `dotnet test`. |
| **Beginner-Friendly Explanation** | Unit testing is the **"does this method work?"** check. You give the method fake inputs and check the output — no database, no web server, no browser. If the method is wrong, the test fails and tells you exactly what broke. It is the fastest, cheapest, and most precise way to catch bugs. |

### Purposes

- **To** verify **business logic** in isolation — pricing, validation, mapping.
- **To** catch **regressions** — when a change breaks existing behavior.
- **To** **document** the expected behavior of each unit.
- **To** enable **fast feedback** — tests run in milliseconds.
- **To** drive **testable design** — difficult-to-test code is usually poorly designed.
- **To** serve as **living documentation** — readable tests explain what the code does.

### Syntax Rules and Structure

#### The AAA Pattern

```csharp
[Fact]
public void ApplyDiscount_10Percent_ReturnsDiscountedPrice()
{
    // Arrange — set up the SUT and dependencies
    var calculator = new DiscountCalculator();

    // Act — execute the behaviour
    var result = calculator.ApplyDiscount(100m, 10m);

    // Assert — verify the outcome
    Assert.Equal(90m, result);
}
```

**Component breakdown:**

| Phase | Purpose | Typical code |
|-------|---------|--------------|
| **Arrange** | Create the SUT, configure mocks, prepare inputs. | `var sut = new Service(mock.Object);` |
| **Act** | Call the method under test. | `var result = sut.DoWork(input);` |
| **Assert** | Verify the outcome. | `Assert.Equal(expected, result);` |

#### Naming Convention

```text
MethodName_Scenario_ExpectedOutcome
```

**Examples:**

- `ApplyDiscount_10Percent_Returns90`
- `FindById_UnknownId_ReturnsNull`
- `Create_NullInput_ThrowsArgumentNullException`

**Syntax rules:**

- **One behavior per test** —do not assert five unrelated things.
- **Tests should be independent** —no shared mutable state.
- **Tests should be deterministic** —no real clocks, no random values without seeding.
- **Test names should describe behavior** —the failure should tell you what broke.

**Constraints and limitations:**

- **Unit tests cannot verify wiring** —DI configuration, middleware order, and database schema are integration-test concerns.
- **Unit tests cannot verify serialization** —JSON shape, content negotiation are integration concerns.
- **High coverage with weak assertions is worthless** —a test that asserts `not null` proves almost nothing.

### Annotated Complete Step-by-Step Code Example

```csharp
// Production code
public class PasswordValidator
{
    public bool IsValid(string? password)
    {
        if (password is null) return false;
        return password.Length >= 8
            && password.Any(char.IsUpper)
            && password.Any(char.IsDigit);
    }
}

// Test code (xUnit)
public class PasswordValidatorTests
{
    private readonly PasswordValidator _sut = new();

    [Theory]
    [InlineData("Abcdefg1", true)]        // 8+ chars, upper, digit
    [InlineData("Abcdefg", false)]        // no digit
    [InlineData("abcdefg1", false)]       // no uppercase
    [InlineData("Abc1", false)]           // too short
    public void IsValid_VariousPasswords_ReturnsExpected(string password, bool expected)
    {
        Assert.Equal(expected, _sut.IsValid(password));
    }

    [Fact]
    public void IsValid_Null_ReturnsFalse()
        => Assert.False(_sut.IsValid(null));
}
```

**Step-by-step explanation:**

1. The SUT is instantiated once per test class (no dependencies).
2. The `[Theory]` covers multiple cases without duplicating code.
3. Each row exercises one validation rule failure/success.
4. `[Fact]` verifies the null guard.

**Expected Output:** All tests pass; if the uppercase rule were removed, the failing case (`"abcdefg1"` → expected `false` but got `true`) would identify the exact input.

**Why this output?** The theory isolates each rule; the failure message identifies the exact input that broke it.

### Real-World Cases

- **Pricing engines:** Discounts, tax, totals.
- **Validation:** Password rules, email format, age limits.
- **Mapping:** DTO ↔ entity conversions.
- **Domain services:** Order state transitions, inventory checks.

### References

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit test controller logic in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/testing

---

## 2. xUnit

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | xUnit is a free, open-source, community-focused unit testing framework for .NET, and the recommended framework for ASP.NET Core projects. |
| **Technical Definition** | xUnit.net uses **facts** (always-run tests) and **theories** (parameterized tests) as its primary test declaration model. It creates a **new instance of the test class for each test method** —ensuring test isolation. It supports **fixtures** (`IClassFixture`, `ICollectionFixture`) for shared setup, **custom attributes** for extensibility, and **async test methods** natively (`async Task`). The ASP.NET Core team uses xUnit for its own tests. |
| **Beginner-Friendly Explanation** | xUnit is the **"modern default"** testing framework. The .NET team uses it, most tutorials use it, and it is designed for clean, isolated tests. It has fewer attributes than NUnit and MSTest —just `[Fact]` for "always run this" and `[Theory]` for "run this with multiple inputs" —but that simplicity is a feature. |

### Purposes

- **To** provide **modern, isolated testing** —new class instance per test.
- **To** support **parameterized tests** via `[Theory]` and `[InlineData]`.
- **To** enable **shared setup** via fixtures.
- **To** support **async tests** natively.
- **To** serve as the **standard framework** for ASP.NET Core projects.

### Syntax Rules and Structure

#### Installation

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Microsoft.NET.Test.Sdk
```

#### General Syntax (Facts and Theories)

```csharp
public class CalculatorTests
{
    [Fact]
    public void Add_2And3_Returns5()
    {
        Assert.Equal(5, new Calculator().Add(2, 3));
    }

    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
    {
        Assert.Equal(expected, new Calculator().Add(a, b));
    }
}
```

**Component breakdown:**

| Attribute | Purpose |
|-----------|---------|
| `[Fact]` | A test with no parameters. |
| `[Theory]` | A parameterized test. |
| `[InlineData(...)]` | Supplied parameters for a theory. |
| `[MemberData]` / `[ClassData]` | Alternative data sources. |

#### Fixtures

```csharp
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture() { /* setup */ }
    public void Dispose() { /* cleanup */ }
}

public class MyTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    public MyTests(DatabaseFixture fixture) => _fixture = fixture;
}
```

**Syntax rules:**

- **Each test gets a fresh instance** of the test class —no shared instance state between tests.
- **`IClassFixture<T>`** shares one fixture per test class.
- **`ICollectionFixture<T>`** shares one fixture across multiple test classes.
- **`async Task`** is the recommended test method signature for async tests.

**Constraints and limitations:**

- **No `[SetUp]`/`[TearDown]` attributes** —use the constructor for setup and `IDisposable.Dispose()` for cleanup.
- **`IClassFixture`** should only be used for expensive, immutable setup (e.g., a test database).
- **Test parallelization** —xUnit runs test classes in parallel by default; ensure fixtures and shared state are thread-safe.

### Annotated Complete Step-by-Step Code Example

```csharp
public class OrderServiceTests
{
    private readonly OrderService _sut;
    private readonly IOrderRepository _repo = Substitute.For<IOrderRepository>();

    public OrderServiceTests()
    {
        _sut = new OrderService(_repo);
    }

    [Fact]
    public async Task GetAsync_ExistingId_ReturnsOrder()
    {
        var order = new Order(1, "Alice", 99.99m);
        _repo.FindAsync(1, Arg.Any<CancellationToken>()).Returns(order);

        var result = await _sut.GetAsync(1, CancellationToken.None);

        Assert.NotNull(result);
        Assert.Equal(1, result!.Id);
    }

    [Theory]
    [InlineData(0)]
    [InlineData(-1)]
    public async Task GetAsync_InvalidId_Throws(int id)
    {
        await Assert.ThrowsAsync<ArgumentOutOfRangeException>(
            () => _sut.GetAsync(id, CancellationToken.None));
    }
}
```

**Expected Output:** Existing IDs return the order; invalid IDs throw `ArgumentOutOfRangeException`.

**Why this output?** xUnit creates a fresh `OrderServiceTests` instance per test, ensuring isolation; the mocked repository is configured per test.

### Real-World Cases

- **Business logic:** The primary target.
- **ASP.NET Core services:** With constructor-injected dependencies.
- **Parameterized validation rules:** Via `[Theory]`.

### References

- xUnit.net getting started – https://xunit.net/docs/getting-started/netcore/cmdline
- xUnit vs NUnit vs MSTest (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test

---

## 3. NUnit

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | NUnit is a mature, feature-rich, open-source unit testing framework for .NET —one of the oldest and most widely used. |
| **Technical Definition** | NUnit uses `[Test]` for test methods, `[TestCase]` for parameterized tests, and `[SetUp]`/`[TearDown]` for per-test setup and cleanup. It supports **`[OneTimeSetUp]`/`[OneTimeTearDown]`** for class-level setup, **`[TestFixture]`** for grouping, and a rich set of attributes for categorizing, ignoring, and sequencing tests. NUnit creates one test class instance per fixture by default (not per test, unless `[FixtureLifeCycle(LifeCycle.InstancePerTestCase)]` is set). |
| **Beginner-Friendly Explanation** | NUnit is the **"mature veteran"** testing framework. It has been around since the early .NET days and offers a rich attribute model —more options than xUnit, which some teams find helpful and others find overwhelming. It supports `[SetUp]`/`[TearDown]` methods, test categories, and fine-grained lifecycle control. |

### Purposes

- **To** provide a **rich attribute model** for test organization.
- **To** support **`[SetUp]`/`[TearDown]`** for classic setup/cleanup patterns.
- **To** enable **test categorization** via `[Category]`.
- **To** support **`[TestCase]`** parameterized tests.
- **To** offer **fine-grained lifecycle control** for shared state.

### Syntax Rules and Structure

#### Installation

```bash
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package Microsoft.NET.Test.Sdk
```

#### General Syntax (Tests and TestCases)

```csharp
[TestFixture]
public class CalculatorTests
{
    private Calculator _sut;

    [SetUp]
    public void SetUp() => _sut = new Calculator();

    [Test]
    public void Add_2And3_Returns5()
    {
        Assert.That(_sut.Add(2, 3), Is.EqualTo(5));
    }

    [TestCase(2, 3, 5)]
    [TestCase(-1, 1, 0)]
    public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
    {
        Assert.That(_sut.Add(a, b), Is.EqualTo(expected));
    }
}
```

**Component breakdown:**

| Attribute | Purpose |
|-----------|---------|
| `[TestFixture]` | Marks a test class. |
| `[Test]` | A test method. |
| `[TestCase(...)]` | Parameterized test inputs. |
| `[SetUp]` | Runs before each test. |
| `[TearDown]` | Runs after each test. |
| `[OneTimeSetUp]` / `[OneTimeTearDown]` | Runs once per fixture. |

#### Assertion Style

```csharp
Assert.That(result, Is.EqualTo(5));
Assert.That(result, Is.GreaterThan(0));
Assert.That(() => _sut.DoWork(null!), Throws.ArgumentNullException);
```

**Syntax rules:**

- **NUnit creates one instance per fixture by default** —use `[FixtureLifeCycle(LifeCycle.InstancePerTestCase)]` for per-test instances.
- **`[SetUp]`/`[TearDown]`** are the classic setup/cleanup hooks.
- **Assert.That** is the modern constraint-based assertion style.

**Constraints and limitations:**

- **Default fixture lifecycle shares instance state** —if tests mutate fields, they can interfere with each other. Use `[FixtureLifeCycle(LifeCycle.InstancePerTestCase)]` to isolate.
- **More attributes to learn** —`[Test]`, `[TestCase]`, `[TestCaseSource]`, `[SetUp]`, `[TearDown]`, `[OneTimeSetUp]`, `[OneTimeTearDown]`, `[Category]`, `[Ignore]`, etc.

### Annotated Complete Step-by-Step Code Example

```csharp
[TestFixture]
[FixtureLifeCycle(LifeCycle.InstancePerTestCase)]   // isolate instance state
public class OrderServiceTests
{
    private OrderService _sut;
    private IOrderRepository _repo;

    [SetUp]
    public void SetUp()
    {
        _repo = Substitute.For<IOrderRepository>();
        _sut = new OrderService(_repo);
    }

    [Test]
    public async Task GetAsync_ExistingId_ReturnsOrder()
    {
        _repo.FindAsync(1, Arg.Any<CancellationToken>())
            .Returns(new Order(1, "Alice", 99.99m));

        var result = await _sut.GetAsync(1, CancellationToken.None);

        Assert.That(result, Is.Not.Null);
        Assert.That(result.Id, Is.EqualTo(1));
    }

    [TestCase(0)]
    [TestCase(-1)]
    public void GetAsync_InvalidId_Throws(int id)
    {
        Assert.ThrowsAsync<ArgumentOutOfRangeException>(
            () => _sut.GetAsync(id, CancellationToken.None));
    }
}
```

**Expected Output:** The same behavior as the xUnit example —existing IDs return orders; invalid IDs throw.

**Why this output?** `[SetUp]` runs before each test; `[FixtureLifeCycle(LifeCycle.InstancePerTestCase)]` ensures a fresh instance per test.

### Real-World Cases

- **Legacy test suites:** Migrated from MSTest or older frameworks.
- **Teams that prefer `[SetUp]`/`[TearDown]`** over constructors.
- **Tests needing categories** for selective execution.

### References

- NUnit documentation – https://docs.nunit.org/
- NUnit vs xUnit vs MSTest comparison – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test

---

## 4. MSTest

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | MSTest is Microsoft's original unit testing framework for .NET, modernized in v3+ with improvements in performance, extensibility, and attribute model. |
| **Technical Definition** | MSTest uses `[TestMethod]` for test methods, `[TestClass]` for test classes, and `[DataRow]` for parameterized tests. It supports `[TestInitialize]`/`[TestCleanup]` and `[ClassInitialize]`/`[ClassCleanup]` for setup and teardown. MSTest v3+ adds **`[DataTestMethod]`** (like `[Theory]`), **assembly-level fixtures** (`[AssemblyInitialize]`), and **parallelization** support. It is fully supported and shipped with Visual Studio. |
| **Beginner-Friendly Explanation** | MSTest is the **"original Microsoft"** test framework. It comes with Visual Studio, has a familiar attribute model, and works well for teams already using it. MSTest v3+ has closed many of the gaps with xUnit and NUnit —it now has parameterized tests, parallelization, and better fixtures. |

### Purposes

- **To** provide a **Visual Studio-native** testing experience.
- **To** support **classic setup/cleanup patterns** (`[TestInitialize]`, `[TestCleanup]`).
- **To** offer **parameterized tests** via `[DataRow]` and `[DataTestMethod]`.
- **To** support **assembly-level fixtures** via `[AssemblyInitialize]`.
- **To** serve teams already standardized on MSTest.

### Syntax Rules and Structure

#### Installation

```bash
dotnet add package MSTest.TestFramework
dotnet add package MSTest.TestAdapter
dotnet add package Microsoft.NET.Test.Sdk
```

#### General Syntax

```csharp
[TestClass]
public class CalculatorTests
{
    private Calculator _sut;

    [TestInitialize]
    public void Initialize() => _sut = new Calculator();

    [TestMethod]
    public void Add_2And3_Returns5()
    {
        Assert.AreEqual(5, _sut.Add(2, 3));
    }

    [DataTestMethod]
    [DataRow(2, 3, 5)]
    [DataRow(-1, 1, 0)]
    public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
    {
        Assert.AreEqual(expected, _sut.Add(a, b));
    }
}
```

**Component breakdown:**

| Attribute | Purpose |
|-----------|---------|
| `[TestClass]` | Marks a test class. |
| `[TestMethod]` | A test method. |
| `[DataTestMethod]` + `[DataRow]` | Parameterized test. |
| `[TestInitialize]` | Runs before each test. |
| `[TestCleanup]` | Runs after each test. |
| `[ClassInitialize]` / `[ClassCleanup]` | Runs once per class. |

**Syntax rules:**

- **`[TestClass]` and `[TestMethod]`** are required.
- **`[DataTestMethod]`** enables parameterized tests; `[DataRow]` supplies the data.
- **`[TestInitialize]`** runs before each test; **`[ClassInitialize]`** runs once per class.

**Constraints and limitations:**

- **`[DataTestMethod]` was introduced in MSTest v2** —older versions lack it.
- **`Assert.AreEqual`** (classic style) has been supplemented by `Assert.AreEqual<T>` and `Assert.That` in newer versions.

### Annotated Complete Step-by-Step Code Example

```csharp
[TestClass]
public class OrderServiceTests
{
    private OrderService _sut;
    private IOrderRepository _repo;

    [TestInitialize]
    public void Initialize()
    {
        _repo = Substitute.For<IOrderRepository>();
        _sut = new OrderService(_repo);
    }

    [TestMethod]
    public async Task GetAsync_ExistingId_ReturnsOrder()
    {
        _repo.FindAsync(1, Arg.Any<CancellationToken>())
            .Returns(new Order(1, "Alice", 99.99m));

        var result = await _sut.GetAsync(1, CancellationToken.None);

        Assert.IsNotNull(result);
        Assert.AreEqual(1, result.Id);
    }
}
```

**Expected Output:** The test passes when the mock returns an order and the service returns it unchanged.

**Why this output?** `[TestInitialize]` sets up the SUT and mock before each test; assertions verify the result.

### Real-World Cases

- **Visual Studio-heavy teams:** MSTest integrates tightly with VS Test Explorer.
- **Legacy test suites:** Migrating to v3+ preserves existing tests with minimal changes.
- **Enterprise applications:** Where MSTest is the standardized choice.

### References

- MSTest documentation – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest
- MSTest vs xUnit vs NUnit comparison – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test

---

## 5. Test Attributes

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Test attributes are .NET attributes that mark methods as tests or supply data to parameterized tests, enabling the test framework to discover and execute them. |
| **Technical Definition** | In xUnit, `[Fact]` marks a parameterless test, `[Theory]` marks a parameterized test, and `[InlineData]`, `[MemberData]`, `[ClassData]` supply the parameters. In NUnit, `[Test]` and `[TestCase]` serve equivalent roles; in MSTest, `[TestMethod]` and `[DataTestMethod]`/`[DataRow]`. The framework's test runner discovers methods with these attributes and executes them, reporting results. |
| **Beginner-Friendly Explanation** | Test attributes are the **"flags"** that tell the framework "this is a test" or "run this test with these inputs." `[Fact]` means "always run this once." `[Theory]` means "run this with each set of data." `[InlineData]` provides one set of inputs. Without attributes, the test methods would just be ordinary methods that never run. |

### Purposes

- **To** **mark methods as tests** for discovery by the runner.
- **To** **parameterize tests** with multiple inputs (`[Theory]`/`[InlineData]`).
- **To** **share data** across tests via `[MemberData]` or `[ClassData]`.
- **To** **control execution** (skip, categorize, sequence).
- **To** **provide metadata** for reporting.

### Syntax Rules and Structure

#### xUnit `[Fact]`

```csharp
[Fact]
public void Add_2And3_Returns5()
{
    Assert.Equal(5, new Calculator().Add(2, 3));
}
```

**When to use:** A single scenario with no parameters, or when you need a one-off test.

#### xUnit `[Theory]` + `[InlineData]`

```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(-1, 1, 0)]
[InlineData(0, 0, 0)]
public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
{
    Assert.Equal(expected, new Calculator().Add(a, b));
}
```

**When to use:** Multiple inputs for the same logic —the most common pattern for data-driven tests.

#### xUnit `[Theory]` + `[MemberData]`

```csharp
public static IEnumerable<object[]> AddCases =>
    new[]
    {
        new object[] { 2, 3, 5 },
        new object[] { -1, 1, 0 }
    };

[Theory]
[MemberData(nameof(AddCases))]
public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
{
    Assert.Equal(expected, new Calculator().Add(a, b));
}
```

**When to use:** Data comes from a property, method, or complex object —cannot be expressed as `[InlineData]`.

#### xUnit `[Theory]` + `[ClassData]`

```csharp
public class AddTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 2, 3, 5 };
        yield return new object[] { -1, 1, 0 };
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

[Theory]
[ClassData(typeof(AddTestData))]
public void Add_VariousInputs_ReturnsSum(int a, int b, int expected)
{
    Assert.Equal(expected, new Calculator().Add(a, b));
}
```

**When to use:** Reusable test data sets shared across test classes.

#### Equivalents in NUnit and MSTest

| Concept | xUnit | NUnit | MSTest |
|---------|-------|-------|--------|
| Single test | `[Fact]` | `[Test]` | `[TestMethod]` |
| Parameterized test | `[Theory]` | `[TestCase]` | `[DataTestMethod]` |
| Inline data | `[InlineData]` | `[TestCase(...)]` | `[DataRow(...)]` |
| Member data | `[MemberData]` | `[TestCaseSource]` | `[DynamicData]` |
| Class data | `[ClassData]` | `[TestCaseSource]` | `[DynamicData]` |

**Syntax rules:**

- **`[Fact]`** —no parameters, always runs once.
- **`[Theory]`** —parameters must be supplied by `[InlineData]`, `[MemberData]`, or `[ClassData]`.
- **`[InlineData]`** —arguments must be compile-time constants (strings, numbers, enums, `null`).
- **`[MemberData]`** —references a static property or method returning `IEnumerable<object[]>`.

**Constraints and limitations:**

- **`[InlineData]`** cannot reference objects created at runtime —use `[MemberData]` for complex inputs.
- **`[MemberData]`** must be a static property/field/method.
- **Parameter count must match** the test method signature —mismatches cause runtime errors, not compile errors.

### Annotated Complete Step-by-Step Code Example

```csharp
public class EmailValidatorTests
{
    [Theory]
    [InlineData("alice@example.com", true)]
    [InlineData("bob+tag@sub.example.co.uk", true)]
    [InlineData("alice@", false)]
    [InlineData("", false)]
    [InlineData(null, false)]
    public void IsValid_VariousInputs_ReturnsExpected(string? email, bool expected)
    {
        Assert.Equal(expected, EmailValidator.IsValid(email));
    }

    public static IEnumerable<object[]> GeneratedEmails
    {
        get
        {
            for (var i = 0; i < 10; i++)
                yield return new object[] { $"user{i}@example.com", true };
        }
    }

    [Theory]
    [MemberData(nameof(GeneratedEmails))]
    public void IsValid_GeneratedEmails_ReturnsTrue(string email, bool expected)
    {
        Assert.Equal(expected, EmailValidator.IsValid(email));
    }
}
```

**Expected Output:** Each `[InlineData]` runs as a separate test case; the `[MemberData]` generates ten more cases. Failures identify the exact input.

**Why this output?** The test runner treats each data row as an individual test, giving clear failure messages.

### Real-World Cases

- **Validation rules:** Emails, passwords, phone numbers —many inputs, one logic path.
- **Calculations:** Discount tiers, tax brackets, date math.
- **Parsing:** Various string formats to typed values.

### References

- xUnit theories – https://xunit.net/docs/getting-started/netcore/cmdline
- `[Theory]` and `[InlineData]` – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test

---

## 6. Assertions

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Assertions are the verification step in a test —they check that the actual outcome matches the expected outcome, and fail the test if it does not. |
| **Technical Definition** | Each test framework provides an assertion API: xUnit uses `Assert.Equal`, `Assert.True`, `Assert.Throws`, `Assert.Null`, `Assert.Contains`, etc.; NUnit uses the constraint-based `Assert.That(actual, Is.EqualTo(expected))`; MSTest uses `Assert.AreEqual`, `Assert.IsTrue`, `Assert.IsNull`, etc. Modern .NET also supports **FluentAssertions** (a popular third-party library) for a more readable, expressive style, though its licensing changed in 2025. |
| **Beginner-Friendly Explanation** | Assertions are the **"check"** in "arrange, act, assert." After running the method, you assert that the result is what you expected. If the assertion fails, the test fails and tells you the expected vs. actual values. Good assertions are specific —they check exact values, not just "not null." |

### Purposes

- **To** **verify outcomes** —the actual result matches the expected result.
- **To** **provide clear failure messages** —expected vs. actual values.
- **To** **express intent** —readable assertions document what the test checks.
- **To** **verify exceptions** —assert that a method throws a specific exception.
- **To** **verify collections** —assert that a sequence contains specific elements.

### Syntax Rules and Structure

#### xUnit Assertions

| Assertion | Purpose |
|-----------|---------|
| `Assert.Equal(expected, actual)` | Equality. |
| `Assert.NotEqual(expected, actual)` | Inequality. |
| `Assert.True(condition)` | Boolean true. |
| `Assert.False(condition)` | Boolean false. |
| `Assert.Null(value)` / `Assert.NotNull(value)` | Nullability. |
| `Assert.Throws<TException>(() => ...)` | Exception thrown (sync). |
| `Assert.ThrowsAsync<TException>(() => ...)` | Exception thrown (async). |
| `Assert.Contains(expected, collection)` | Collection membership. |
| `Assert.Empty(collection)` / `Assert.NotEmpty(collection)` | Collection emptiness. |

#### NUnit Assertions (Constraint Model)

```csharp
Assert.That(actual, Is.EqualTo(5));
Assert.That(actual, Is.Not.Null);
Assert.That(collection, Does.Contain("x"));
Assert.That(() => sut.DoWork(null!), Throws.ArgumentNullException);
```

#### MSTest Assertions

```csharp
Assert.AreEqual(5, actual);
Assert.IsTrue(condition);
Assert.IsNull(value);
Assert.ThrowsException<ArgumentNullException>(() => sut.DoWork(null!));
```

**Syntax rules:**

- **Use the most specific assertion** —`Assert.Equal` over `Assert.True(a == b)`.
- **Assert on exact values** —avoid `Assert.NotNull` when you can assert the value.
- **Assert one logical outcome per test** —multiple assertions are acceptable if they verify the same behavior.

**Constraints and limitations:**

- **`Assert.True(a == b)`** produces unhelpful failure messages —use `Assert.Equal`.
- **FluentAssertions licensing** changed in 2025 —Xceed took over commercial licensing; check before adopting.
- **Assertions in loops** —use `Assert.Collection` or collect results to assert on.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ShoppingCartTests
{
    [Fact]
    public void AddItem_IncreasesCountAndTotal()
    {
        var cart = new ShoppingCart();
        cart.AddItem(new Item(Id: 1, Price: 19.99m));
        cart.AddItem(new Item(Id: 2, Price: 5.00m));

        Assert.Equal(2, cart.ItemCount);
        Assert.Equal(24.99m, cart.Total);
        Assert.Contains(cart.Items, i => i.Id == 1);
    }

    [Fact]
    public void AddItem_Null_Throws()
    {
        var cart = new ShoppingCart();
        Assert.Throws<ArgumentNullException>(() => cart.AddItem(null!));
    }

    [Fact]
    public async Task CheckoutAsync_EmptyCart_Throws()
    {
        var cart = new ShoppingCart();
        await Assert.ThrowsAsync<InvalidOperationException>(
            () => cart.CheckoutAsync(CancellationToken.None));
    }
}
```

**Expected Output:** All three tests pass; each verifies a specific behavior.

**Why this output?** Each assertion is specific and produces a clear failure message if the behavior regresses.

### Real-World Cases

- **State verification:** Item count, total, contents.
- **Exception verification:** Null guards, invalid state.
- **Async behavior:** `ThrowsAsync`, `ThrowsException`.

### References

- xUnit assertions – https://xunit.net/docs/getting-started/netcore/cmdline
- NUnit constraints – https://docs.nunit.org/articles/nunit/writing-tests/constraints/Constraints.html
- MSTest assertions – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest

---

## 7. Mocking (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Mocking is the practice of creating fake implementations of a class's dependencies so that the class can be tested in isolation, with full control over the behavior of those dependencies. |
| **Technical Definition** | A **test double** stands in for a real dependency. The major types are **stubs** (return canned values), **mocks** (verify that specific calls were made), and **fakes** (working but simplified implementations, e.g., an in-memory repository). In .NET, mocking libraries (Moq, NSubstitute, FakeItEasy) generate test doubles for interfaces and virtual members at runtime. The SUT receives the double via constructor injection, letting the test control and observe its interactions. |
| **Beginner-Friendly Explanation** | Mocking is the **"stand-in actor"** technique. If your service depends on a database repository, you don't want to actually hit the database in a unit test. Instead, you create a fake repository that you fully control —you decide what it returns, and you can check whether your service called it correctly. Mocking lets you test your code in isolation, without the real dependencies. |

### Purposes

- **To** **isolate the SUT** —remove external dependencies (databases, APIs, clocks).
- **To** **control dependency behavior** —return specific values, throw exceptions.
- **To** **verify interactions** —did the service call the repository with the right parameters?
- **To** **test edge cases** —simulate failure conditions that are hard to trigger with real dependencies.
- **To** **make tests fast and deterministic** —no network, no database, no real time.

### Syntax Rules and Structure

#### The Test Double Spectrum

| Type | Behavior | Example |
|------|----------|---------|
| **Stub** | Returns canned values | `repo.Find(1)` returns a fixed order. |
| **Mock** | Verifies interactions | `repo.Received().Save(order)`. |
| **Fake** | Working simplified implementation | In-memory repository. |

#### General Syntax (NSubstitute)

```csharp
var repo = Substitute.For<IOrderRepository>();
repo.FindAsync(1, Arg.Any<CancellationToken>())
    .Returns(new Order(1, "Alice", 99.99m));

var sut = new OrderService(repo);
```

#### General Syntax (Moq)

```csharp
var repo = new Mock<IOrderRepository>();
repo.Setup(r => r.FindAsync(1, It.IsAny<CancellationToken>()))
    .ReturnsAsync(new Order(1, "Alice", 99.99m));

var sut = new OrderService(repo.Object);
```

**Syntax rules:**

- **Mock interfaces, not concrete classes** —interfaces are easiest to substitute.
- **Configure only what the test needs** —unconfigured calls return default values.
- **Verify interactions sparingly** —verify behavior, not implementation.
- **Use `Arg.Any<T>()`/`It.IsAny<T>()`** for arguments whose exact value the test doesn't care about.

**Constraints and limitations:**

- **Over-mocking** couples tests to implementation —if you refactor the service to call the repository differently, the test breaks even if behavior is unchanged.
- **Mocks don't prove integration** —they prove the SUT calls the dependency as expected; integration tests prove the dependency works.
- **Concrete class mocking** requires `virtual` methods (both libraries) or interfaces.

### Annotated Complete Step-by-Step Code Example

```csharp
public class OrderServiceTests
{
    [Fact]
    public async Task PlaceOrder_ValidOrder_SavesAndReturnsOrder()
    {
        // Arrange
        var repo = Substitute.For<IOrderRepository>();
        var sut = new OrderService(repo);
        var order = new Order(1, "Alice", 99.99m);

        // Act
        var result = await sut.PlaceOrderAsync(order, CancellationToken.None);

        // Assert — verify interaction
        await repo.Received(1).SaveAsync(
            Arg.Is<Order>(o => o.Id == 1),
            Arg.Any<CancellationToken>());
        Assert.Equal(order, result);
    }
}
```

**Expected Output:** The test passes if the service calls `SaveAsync` exactly once with the expected order and returns the same order.

**Why this output?** `Substitute.For` creates the mock; `Received(1)` verifies the call count; `Arg.Is` verifies the argument.

### Real-World Cases

- **Repository dependencies:** Stub the repository, verify the service logic.
- **External APIs:** Stub the HTTP client, simulate responses and failures.
- **Time:** Stub `IClock`, control "now."
- **Unit of Work:** Verify `SaveChangesAsync` was called.

### References

- Moq vs NSubstitute comparison – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Test doubles (Martin Fowler) – https://martinfowler.com/bliki/TestDouble.html

---

## 8. Moq

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Moq is a popular, mature mocking library for .NET that uses `Mock<T>` objects and lambda-based setup expressions to create and configure test doubles. |
| **Technical Definition** | Moq creates `Mock<T>` instances for interfaces and virtual members. Configuration uses `.Setup(x => x.Method(...))` with `.Returns(...)`/`.Throws(...)`; verification uses `.Verify(x => x.Method(...), Times.Once)`. It supports **strict** vs. **loose** mocks, **callback** configuration, **argument matchers** (`It.IsAny<T>`, `It.Is<T>`), and **property** setups. **Licensing note:** In 2023, Moq's maintainer added **SponsorLink** to the build, which prompted widespread concerns; many teams migrated to NSubstitute or FakeItEasy. Later releases removed SponsorLink, but the controversy led to reduced adoption. |
| **Beginner-Friendly Explanation** | Moq is the **"classic mocking library"**. You write `new Mock<IRepository>()`, configure it with `.Setup(...)`, pass `mock.Object` to the SUT, and verify calls with `.Verify(...)`. It is powerful and widely documented, but a 2023 licensing controversy drove some teams to alternatives. Check the current version and licensing status before adopting. |

### Purposes

- **To** create **test doubles** for interfaces and virtual members.
- **To** configure **return values** and **exceptions**.
- **To** **verify interactions** —call counts and arguments.
- **To** support **strict** mocks (fail on unexpected calls).
- **To** provide a **fluent, lambda-based API**.

### Syntax Rules and Structure

#### General Syntax (Setup and Verify)

```csharp
var mock = new Mock<IOrderRepository>();

// Configure a return value
mock.Setup(r => r.FindAsync(1, It.IsAny<CancellationToken>()))
    .ReturnsAsync(new Order(1, "Alice", 99.99m));

// Use the mock
var sut = new OrderService(mock.Object);
await sut.GetAsync(1, CancellationToken.None);

// Verify the interaction
mock.Verify(
    r => r.FindAsync(1, It.IsAny<CancellationToken>()),
    Times.Once);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `new Mock<T>()` | Creates the mock. |
| `.Setup(...)` | Configures behavior. |
| `.Returns(...)` / `.Throws(...)` | Specifies the outcome. |
| `It.IsAny<T>()` / `It.Is<T>(...)` | Argument matchers. |
| `.Verify(..., Times.Once)` | Verifies interaction. |

#### Strict vs. Loose Mocks

```csharp
// Loose (default) — unconfigured calls return default values
var loose = new Mock<IRepository>();

// Strict — unconfigured calls throw
var strict = new Mock<IRepository>(MockBehavior.Strict);
```

**Syntax rules:**

- **Loose mocks** (default) return default values for unconfigured calls.
- **Strict mocks** throw `MockException` for unconfigured calls —use sparingly; they make tests brittle.
- **`mock.Object`** is the instance passed to the SUT.

**Constraints and limitations:**

- **SponsorLink controversy** —check the version and licensing status; prefer alternatives if uncomfortable.
- **Over-mocking** —`.Verify` on every call couples tests to implementation.
- **Concrete classes** require `virtual` methods.

### Annotated Complete Step-by-Step Code Example

```csharp
public class OrderServiceTests
{
    [Fact]
    public async Task PlaceOrder_SavesAndReturnsOrder()
    {
        // Arrange
        var mock = new Mock<IOrderRepository>();
        mock.Setup(r => r.SaveAsync(
                It.IsAny<Order>(),
                It.IsAny<CancellationToken>()))
            .Returns(Task.CompletedTask);

        var sut = new OrderService(mock.Object);
        var order = new Order(1, "Alice", 99.99m);

        // Act
        var result = await sut.PlaceOrderAsync(order, CancellationToken.None);

        // Assert
        mock.Verify(
            r => r.SaveAsync(
                It.Is<Order>(o => o.Id == 1),
                It.IsAny<CancellationToken>()),
            Times.Once);
        Assert.Equal(order, result);
    }
}
```

**Expected Output:** The test passes if the service saves the order once and returns it.

**Why this output?** `Setup` configures the repository to accept the save; `Verify` checks the call; the assertion verifies the result.

### Real-World Cases

- **Teams already using Moq:** Migrating has cost; Moq remains fully functional.
- **Simple interface mocking:** Repositories, external services.
- **Interaction verification:** Confirming a service calls its dependency.

### References

- Moq documentation – https://github.com/devlooped/moq
- Moq SponsorLink controversy – https://github.com/devlooped/moq/wiki/FAQ

---

## 9. NSubstitute

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | NSubstitute is a friendly, easy-to-use mocking library for .NET that uses `Substitute.For<T>()` and natural C# syntax for configuration and verification. |
| **Technical Definition** | NSubstitute creates substitutes via `Substitute.For<T>()`. Configuration uses the **"setup by calling"** style: `sub.Method(args).Returns(value)`. Argument matching uses `Arg.Any<T>()`, `Arg.Is<T>(...)`. Verification uses `Received(count)`: `sub.Received(1).SaveAsync(...)`. It has **no strict/loose distinction** (substitutes return default values unless configured) and **no licensing controversy**. Its API is often described as more natural and readable than Moq. |
| **Beginner-Friendly Explanation** | NSubstitute is the **"friendlier mocking library"**. You write `Substitute.For<IRepository>()`, configure with `sub.Get(1).Returns(value)`, and verify with `sub.Received(1).Save(...)`. It reads like plain English, has no licensing issues, and is fully open-source. Many teams switched to NSubstitute after the Moq controversy. |

### Purposes

- **To** create **test doubles** with a clean, readable API.
- **To** configure **return values** and **exceptions**.
- **To** **verify interactions** with `Received`.
- **To** avoid **licensing concerns** —fully open-source, no SponsorLink.
- **To** provide **natural C# syntax** for mocking.

### Syntax Rules and Structure

#### General Syntax (Configuration and Verification)

```csharp
var repo = Substitute.For<IOrderRepository>();

// Configure
repo.FindAsync(1, Arg.Any<CancellationToken>())
    .Returns(new Order(1, "Alice", 99.99m));

// Use
var sut = new OrderService(repo);
await sut.GetAsync(1, CancellationToken.None);

// Verify
await repo.Received(1).FindAsync(
    1, Arg.Any<CancellationToken>());
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Substitute.For<T>()` | Creates the substitute. |
| `.Returns(value)` | Configures the return value. |
| `.Throws(exception)` | Configures an exception. |
| `Arg.Any<T>()` / `Arg.Is<T>(...)` | Argument matchers. |
| `Received(count)` | Verifies the call count. |

#### Did Not Receive

```csharp
await repo.DidNotReceive().SaveAsync(
    Arg.Any<Order>(), Arg.Any<CancellationToken>());
```

**Syntax rules:**

- **No strict mode** —unconfigured calls return default values.
- **`Received(n)`** verifies the call was made exactly `n` times.
- **`DidNotReceive()`** verifies the call was never made.
- **`.Returns(...)`** on `Task<T>` returns a completed task.

**Constraints and limitations:**

- **No strict mode** —unlike Moq, NSubstitute does not fail on unexpected calls.
- **Argument matchers must be used consistently** —mixing `Arg.Any<T>()` and literal values in the same call works, but be careful.
- **Concrete classes** require `virtual` members.

### Annotated Complete Step-by-Step Code Example

```csharp
public class OrderServiceTests
{
    [Fact]
    public async Task PlaceOrder_SavesAndReturnsOrder()
    {
        // Arrange
        var repo = Substitute.For<IOrderRepository>();
        var sut = new OrderService(repo);
        var order = new Order(1, "Alice", 99.99m);

        // Act
        var result = await sut.PlaceOrderAsync(order, CancellationToken.None);

        // Assert
        await repo.Received(1).SaveAsync(
            Arg.Is<Order>(o => o.Id == 1),
            Arg.Any<CancellationToken>());
        Assert.Equal(order, result);
    }

    [Fact]
    public async Task PlaceOrder_NullOrder_ThrowsWithoutSaving()
    {
        var repo = Substitute.For<IOrderRepository>();
        var sut = new OrderService(repo);

        await Assert.ThrowsAsync<ArgumentNullException>(
            () => sut.PlaceOrderAsync(null!, CancellationToken.None));

        await repo.DidNotReceive().SaveAsync(
            Arg.Any<Order>(), Arg.Any<CancellationToken>());
    }
}
```

**Expected Output:** Valid orders are saved once and returned; null orders throw without saving.

**Why this output?** `Received(1)` confirms the save; `DidNotReceive` confirms the guard clause prevented the call.

### Real-World Cases

- **Teams migrating from Moq:** Cleaner syntax, no licensing concerns.
- **New projects:** Recommended by many .NET developers for its readability.
- **Interaction-heavy tests:** `Received` and `DidNotReceive` are expressive.

### References

- NSubstitute documentation – https://nsubstitute.github.io/
- NSubstitute vs Moq comparison – https://nsubstitute.github.io/help/comparison-to-moq/

---

## 10. Unit Testing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Unit testing best practices are guidelines for writing tests that are fast, isolated, readable, and maintainable. |
| **Technical Definition** | Best practices span **naming** (Method_Scenario_ExpectedOutcome), **structure** (AAA), **isolation** (no shared state, no infrastructure), **assertion specificity** (exact values, not just non-null), **mocking discipline** (mock dependencies, not the SUT), **test doubles** (prefer fakes/stubs over mocks when possible), **async discipline** (async tests for async code), **test data** (builders, object mothers), and **CI integration** (run `dotnet test` on every commit). |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for unit testing—they help you avoid flaky tests, tests that pass but don't actually verify anything, and test suites that are so slow or brittle that nobody maintains them. |

### Purposes

- **To** keep tests **readable** —a failing test should immediately tell you what broke.
- **To** keep tests **fast** —unit tests should run in milliseconds.
- **To** keep tests **reliable** —no flakiness from shared state or time.
- **To** keep tests **maintainable** —tests that are harder to change than the code are a liability.
- **To** ensure **coverage is meaningful** —assert on specific outcomes.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Naming** | `Method_Scenario_ExpectedOutcome`. | `Test1`, `Test2`. |
| **Structure** | Arrange-Act-Assert. | Mix setup and assertions. |
| **Isolation** | No shared mutable state; no static state. | Depend on test execution order. |
| **Infrastructure** | Mock repositories, HTTP, clocks. | Touch the database, file system, or network. |
| **Assertions** | Assert exact values, exception types, collection contents. | Assert only `not null` —it proves almost nothing. |
| **Mocking** | Mock external dependencies; verify interactions sparingly. | Mock everything; verify every call. |
| **Test doubles** | Prefer **fakes** for complex dependencies (in-memory repository). | Use mocks for everything —it makes tests brittle. |
| **Async** | Use `async Task` test methods and `ThrowsAsync`. | Use `.Result` or `.Wait()` in tests. |
| **Test data** | Use object mothers/builders for readable setup. | Duplicate 20 lines of setup in every test. |
| **Frameworks** | Pick **one** framework per project. | Mix xUnit, NUnit, and MSTest in one test project. |
| **CI** | Run `dotnet test` on every PR. | Skip tests because "they're slow" —that's a signal to fix them. |

### Annotated Code Example (Best-Practice Test)

```csharp
public class DiscountCalculatorTests
{
    private readonly DiscountCalculator _sut = new();

    [Theory]
    [InlineData(100, 0, 100)]
    [InlineData(100, 10, 90)]
    [InlineData(100, 100, 0)]
    public void ApplyDiscount_ValidPercent_ReturnsDiscountedPrice(
        decimal price, decimal percent, decimal expected)
    {
        var result = _sut.ApplyDiscount(price, percent);
        Assert.Equal(expected, result);
    }

    [Fact]
    public void ApplyDiscount_NegativePercent_Throws()
    {
        Assert.Throws<ArgumentOutOfRangeException>(
            () => _sut.ApplyDiscount(100, -1));
    }

    [Fact]
    public void ApplyDiscount_PercentAbove100_Throws()
    {
        Assert.Throws<ArgumentOutOfRangeException>(
            () => _sut.ApplyDiscount(100, 101));
    }
}
```

**Why this follows best practices:**

- **Clear names** —`ApplyDiscount_ValidPercent_ReturnsDiscountedPrice`.
- **Theory** covers multiple valid cases.
- **Facts** cover guard clauses individually —one behavior per test.
- **Specific assertions** —`Assert.Equal(expected, result)`, `Assert.Throws<T>`.
- **No infrastructure** —pure in-memory logic.
- **No shared state** —new `DiscountCalculator` per class instance (xUnit creates new instance per test).

### Real-World Cases

- **PR gates:** Unit tests provide fast feedback on every commit.
- **Refactoring safety:** Tests protect existing behavior during restructuring.
- **Bug fixes:** Write a failing test that reproduces the bug, then fix it.

### References

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit vs NUnit vs MSTest – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test
- NSubstitute best practices – https://nsubstitute.github.io/help/
- Moq documentation – https://github.com/devlooped/moq
- FluentAssertions licensing (2025) – https://xceed.com/

---

## Summary Table: Unit Testing Tools at a Glance

| Tool | Category | Key Strength | When to Choose |
|------|----------|--------------|----------------|
| **xUnit** | Test framework | Modern, isolated, recommended for ASP.NET Core | New ASP.NET Core projects |
| **NUnit** | Test framework | Rich attribute model, `[SetUp]`/`[TearDown]` | Teams preferring classic lifecycle patterns |
| **MSTest** | Test framework | Visual Studio-native, v3+ modernized | Visual Studio-heavy teams |
| **`[Fact]`** | xUnit attribute | One-off test | Single scenario |
| **`[Theory]`** | xUnit attribute | Parameterized test | Multiple inputs, one logic path |
| **`[InlineData]`** | xUnit attribute | Inline parameters | Compile-time constant inputs |
| **Moq** | Mocking library | Mature, widely documented | Teams already using it (check licensing) |
| **NSubstitute** | Mocking library | Cleaner syntax, no licensing issues | New projects, teams migrating from Moq |
| **Best practices** | Guidelines | Fast, reliable, maintainable tests | Every test suite |

---

## References (Full List)

- Unit testing best practices for .NET – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit test controller logic in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/testing
- xUnit documentation – https://xunit.net/docs/getting-started/netcore/cmdline
- NUnit documentation – https://docs.nunit.org/
- NUnit constraints – https://docs.nunit.org/articles/nunit/writing-tests/constraints/Constraints.html
- MSTest documentation – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest
- xUnit vs NUnit vs MSTest – https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test
- Moq documentation – https://github.com/devlooped/moq
- Moq SponsorLink FAQ – https://github.com/devlooped/moq/wiki/FAQ
- NSubstitute documentation – https://nsubstitute.github.io/
- NSubstitute vs Moq – https://nsubstitute.github.io/help/comparison-to-moq/
- Test doubles (Martin Fowler) – https://martinfowler.com/bliki/TestDouble.html
- FluentAssertions licensing (2025) – https://xceed.com/
- The Practical Test Pyramid (Martin Fowler) – https://martinfowler.com/articles/practical-test-pyramid.html