# Unit Testing in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Unit testing is the practice of verifying the behavior of individual, isolated units of code — typically methods or classes — by executing them with controlled inputs and asserting that the observed outcomes match the expected outcomes.

**Technical Definition**
A unit test is an automated, deterministic, and isolated test that exercises a single unit of production code (a method, a class, or a small cluster of tightly related classes) without touching external infrastructure such as databases, file systems, networks, or clocks. In .NET, unit tests are written using test frameworks — xUnit, NUnit, or MSTest — that provide test discovery attributes, lifecycle hooks, assertion APIs, and runners integrated with `dotnet test` and Visual Studio. Unit tests follow the Arrange-Act-Assert (AAA) structure and the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely).

**Beginner-Friendly Explanation**
A unit test is a small program that checks one piece of your code. You give the code some input, you call it, and you check that the output is what you expected. If the code breaks later, the test fails and tells you something is wrong. Unit tests are fast — they run in milliseconds — because they do not need a database or a browser.

---

### Key Characteristics

- **Fast** — Unit tests run in milliseconds, enabling rapid feedback.
- **Isolated** — Each test runs independently, with no shared mutable state and no dependency on execution order.
- **Deterministic** — The same input always produces the same result; no reliance on time, randomness, or external services.
- **Repeatable** — Tests produce consistent results on any machine, in any environment.
- **Self-validating** — Tests report pass or fail automatically, without manual inspection.
- **Framework-driven** — xUnit, NUnit, and MSTest provide discovery, execution, and assertions.
- **Design pressure** — Testability encourages loose coupling and dependency injection.
- **Regression protection** — Unit tests catch defects introduced by changes to existing code.

---

### Prerequisites

- Working knowledge of C# classes, interfaces, methods, and properties.
- Understanding of dependency injection and the Dependency Inversion Principle.
- Familiarity with the .NET CLI (`dotnet test`, `dotnet build`).
- A test framework package (xUnit, NUnit, or MSTest).
- Basic understanding of `async`/`await` for testing asynchronous code.
- Awareness of test doubles (stubs, mocks, fakes) for isolating dependencies.

---

### Related Programming Areas

- **Test-driven development (TDD)** — Writing tests before production code.
- **Behavior-driven development (BDD)** — Specifying behavior in business-readable language.
- **Integration testing** — Verifying component interactions.
- **Test doubles and mocking** — Moq, NSubstitute, FakeItEasy.
- **Mutation testing** — Stryker.NET for evaluating test suite quality.
- **Code coverage** — Coverlet and ReportGenerator.
- **Continuous integration** — Automated test execution on every commit.

---

### Core Concepts / Features

1. Unit testing (overarching discipline)
2. xUnit
3. NUnit
4. MSTest
5. Test attributes
   - `[Fact]`
   - `[Theory]`
   - `[Test]`
   - `[TestCase]`
6. Assertions
7. Test fixtures
8. Test organization
9. Unit testing best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Unit Testing (Overarching Discipline)

### Definitions

**Core Definition**
Unit testing is the systematic practice of verifying individual units of software in isolation to ensure they behave as specified.

**Technical Definition**
Unit testing is a verification technique in which a unit — the smallest testable piece of software — is exercised in isolation from its dependencies. In .NET, unit tests are compiled into test assemblies, discovered by test adapters, and executed by runners such as VSTest or `dotnet test`. The tests assert on return values, state changes, or thrown exceptions. Unit testing is distinguished from integration testing (which exercises real dependencies) and end-to-end testing (which exercises the full system) by its speed, isolation, and granularity.

**Beginner-Friendly Explanation**
Unit testing is "checking each Lego brick before you build the castle." You test one small piece at a time, in isolation, so that when something breaks you know exactly which brick is faulty.

---

### Purposes

- To verify that individual methods and classes behave correctly.
- To catch logic errors at the earliest possible stage.
- To provide a regression safety net for refactoring.
- To serve as executable documentation of expected behavior.
- To drive testable, loosely coupled design.
- To reduce debugging time and defect escape rate.
- To enable fast feedback during development.

---

### Syntax Rules and Structure

#### General Project Structure

```
MySolution/
  src/
    MyApp/                     (production code)
  tests/
    MyApp.Tests/               (test project)
      MyApp.Tests.csproj
      CalculatorTests.cs
```

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
dotnet test
dotnet test --filter "FullyQualifiedName~CalculatorTests"
dotnet test --collect:"XPlat Code Coverage"
```

**Component breakdown**

- `src/` — Production code; no test framework references.
- `tests/` — One or more test projects; references the production project.
- `Microsoft.NET.Test.Sdk` — Required test platform SDK.
- `xunit` / `NUnit` / `MSTest.TestFramework` — The test framework.
- `xunit.runner.visualstudio` / `NUnit3TestAdapter` / `MSTest.TestAdapter` — Test adapter for discovery.
- `dotnet test` — Discovers and executes tests.

#### Syntax Rules

- Test projects must reference the test SDK and a framework adapter.
- Test methods must be discoverable by the framework (e.g., `[Fact]`, `[Test]`, `[TestMethod]`).
- Tests should be isolated: no shared mutable state, no execution-order dependency.
- Tests should be deterministic: no reliance on time, randomness, or external services.
- Test names should describe the scenario and expected behavior.

#### Constraints and Limitations

- Unit tests cannot catch integration defects (SQL errors, serialization mismatches).
- Over-mocking can produce tests that pass even when the real system is broken.
- Unit tests do not verify user-facing behavior.
- Tests add maintenance overhead; poorly written tests become a liability.

---

### Annotated Code Example

```csharp
using Xunit;

public class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Divide(int a, int b) =>
        b == 0 ? throw new DivideByZeroException() : a / b;
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

    [Fact]
    public void Divide_ByZero_ThrowsDivideByZeroException()
    {
        var calculator = new Calculator();

        Assert.Throws<DivideByZeroException>(() => calculator.Divide(10, 0));
    }
}
```

**Expected Output**

```
Passed: Add_TwoPositiveNumbers_ReturnsSum
Passed: Divide_ByZero_ThrowsDivideByZeroException
```

**Why this output occurs**

- The first test verifies the return value of `Add`.
- The second test verifies that the expected exception type is thrown.
- Both tests are isolated and deterministic.

---

### Real-World Cases

- **Business logic** — Pricing rules, tax calculation, validation.
- **Domain models** — Invariants and state transitions.
- **Utility methods** — String parsing, date formatting, math helpers.
- **API handlers** — Controller logic with mocked services.

---

### References

- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit testing in .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/
- xUnit.net documentation - https://xunit.net/

---

## 2. xUnit

### Definitions

**Core Definition**
xUnit is a free, open-source, community-focused unit testing framework for .NET that emphasizes simplicity, extensibility, and isolation.

**Technical Definition**
xUnit.net was written by the original creator of NUnit v2 and is designed to encourage TDD best practices. It provides `[Fact]` for parameterless tests, `[Theory]` with `[InlineData]`, `[MemberData]`, and `[ClassData]` for data-driven tests, a rich `Assert` class, fixture interfaces (`IClassFixture<T>`, `ICollectionFixture<T>`), and built-in parallelization. A new instance of the test class is created for every test method, enforcing isolation.

**Beginner-Friendly Explanation**
xUnit is a modern testing framework. You mark test methods with `[Fact]` (a single fact) or `[Theory]` (a theory with data). xUnit creates a fresh object for each test, so tests do not accidentally share state.

---

### Purposes

- To provide a simple, convention-based testing framework.
- To enforce test isolation by creating a new test class instance per test.
- To support data-driven testing with `[Theory]`.
- To enable parallel test execution by default.
- To offer extensibility through custom facts, theories, and fixtures.
- To integrate with `dotnet test` and Visual Studio.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        var calculator = new Calculator();
        var result = calculator.Add(2, 3);
        Assert.Equal(5, result);
    }

    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(10, 20, 30)]
    public void Add_DataDriven_ReturnsSum(int a, int b, int expected)
    {
        var calculator = new Calculator();
        Assert.Equal(expected, calculator.Add(a, b));
    }
}
```

**Component breakdown**

- `[Fact]` — Marks a parameterless test method.
- `[Theory]` — Marks a data-driven test method.
- `[InlineData(...)]` — Supplies inline arguments for each theory case.
- `[MemberData]` — References a static property or method returning test data.
- `[ClassData]` — References a class implementing `IEnumerable<object[]>`.
- `Assert.Equal` — Asserts equality.

#### Syntax Rules

- Test classes are public and non-static.
- Test methods are public, return `void` or `Task`, and have no parameters for `[Fact]`.
- `[Theory]` methods must have parameters matching the data rows.
- xUnit creates a new instance of the test class for each test method.
- Setup is done in the constructor; teardown via `IDisposable` or `IAsyncLifetime`.
- xUnit has no `[SetUp]`/`[TearDown]` attributes; use constructors and `Dispose`.

#### Constraints and Limitations

- xUnit is opinionated and restrictive compared with NUnit.
- No built-in `[SetUp]`/`[TearDown]`; use constructor and `IDisposable`.
- Parallelization is on by default; tests must be isolated.
- Fixture scope is controlled by `IClassFixture` and `ICollectionFixture`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Fact and Theory

```csharp
using Xunit;

public class StringCalculatorTests
{
    [Fact]
    public void Add_EmptyString_ReturnsZero()
    {
        var calc = new StringCalculator();
        Assert.Equal(0, calc.Add(""));
    }

    [Theory]
    [InlineData("1", 1)]
    [InlineData("1,2", 3)]
    [InlineData("10,20,30", 60)]
    public void Add_CommaSeparatedNumbers_ReturnsSum(string input, int expected)
    {
        var calc = new StringCalculator();
        Assert.Equal(expected, calc.Add(input));
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

- `[Fact]` runs once; `[Theory]` runs once per `[InlineData]` row.
- Each data row is reported as a separate test case.

---

#### Example 2: MemberData for Complex Data

```csharp
public class DiscountTests
{
    public static TheoryData<decimal, decimal> DiscountData => new()
    {
        { 100m, 0.10m },
        { 200m, 0.20m },
        { 500m, 0.25m }
    };

    [Theory]
    [MemberData(nameof(DiscountData))]
    public void ApplyDiscount_ReturnsReducedPrice(decimal price, decimal rate)
    {
        var discount = new DiscountService();
        var result = discount.Apply(price, rate);
        Assert.Equal(price * (1 - rate), result);
    }
}
```

**Expected Output**

```
Passed: ApplyDiscount_ReturnsReducedPrice(price: 100, rate: 0.10)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 200, rate: 0.20)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 500, rate: 0.25)
```

**Why this output occurs**

- `TheoryData<T1, T2>` supplies strongly typed data rows.
- `[MemberData]` references the static property.

---

### Real-World Cases

- **Open-source libraries** — xUnit is the dominant framework in the .NET OSS ecosystem.
- **ASP.NET Core** — The default test framework in `dotnet new xunit` templates.
- **Data-driven tests** — Pricing tiers, validation rules, and parsing logic.

---

### References

- xUnit.net official documentation - https://xunit.net/
- xUnit.net getting started - https://xunit.net/docs/getting-started/netcore/cmdline
- xUnit Theory and data-driven tests - https://xunit.net/docs/getting-started/netcore/cmdline
- Unit testing best practices (xUnit examples) — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices

---

## 3. NUnit

### Definitions

**Core Definition**
NUnit is a mature, open-source unit testing framework for .NET, originally ported from JUnit, offering a rich attribute set, constraint-based assertions, and extensive parameterization.

**Technical Definition**
NUnit provides `[Test]` for simple tests, `[TestCase]` for inline parameterized tests, `[TestCaseSource]` for external data, `[TestFixture]` for grouping, `[SetUp]`/`[TearDown]` for per-test lifecycle, `[OneTimeSetUp]`/`[OneTimeTearDown]` for fixture lifecycle, and the constraint model (`Assert.That(actual, Is.EqualTo(expected))`). NUnit is known for flexibility and a large ecosystem of attributes.

**Beginner-Friendly Explanation**
NUnit is a classic testing framework. You mark tests with `[Test]` and use `[TestCase]` to run the same test with different data. It has a very flexible assertion syntax and many configuration options.

---

### Purposes

- To provide a flexible, feature-rich testing framework.
- To support inline and external data-driven tests.
- To offer per-test and per-fixture lifecycle hooks.
- To support categorization and filtering of tests.
- To provide a constraint-based assertion model.
- To integrate with Visual Studio and `dotnet test`.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using NUnit.Framework;

[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp]
    public void SetUp() => _calculator = new Calculator();

    [Test]
    public void Add_TwoNumbers_ReturnsSum()
    {
        Assert.That(_calculator.Add(2, 3), Is.EqualTo(5));
    }

    [TestCase(1, 2, 3)]
    [TestCase(10, 20, 30)]
    public void Add_DataDriven_ReturnsSum(int a, int b, int expected)
    {
        Assert.That(_calculator.Add(a, b), Is.EqualTo(expected));
    }
}
```

**Component breakdown**

- `[TestFixture]` — Marks the test class (optional in modern NUnit).
- `[SetUp]` — Runs before each test.
- `[TearDown]` — Runs after each test.
- `[Test]` — Marks a simple test method.
- `[TestCase(...)]` — Supplies inline arguments for a parameterized test.
- `Assert.That(actual, Is.EqualTo(expected))` — Constraint-based assertion.

#### Syntax Rules

- `[TestCase]` can be applied multiple times to create multiple cases.
- `[TestCaseSource]` references a static member returning test data.
- `[Values]` and `[Range]` supply parameter values.
- `[ExpectedResult]` on `[TestCase]` verifies the return value automatically.
- Named parameters on `[TestCase]` set metadata (`Ignore`, `Category`, `Author`, `Description`).

#### Constraints and Limitations

- NUnit is more flexible but less opinionated than xUnit.
- Parallel execution is configurable but not on by default in the same way as xUnit.
- Some attribute names differ from MSTest and xUnit, complicating migration.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: TestCase with ExpectedResult

```csharp
[TestCase(12, 3, ExpectedResult = 4)]
[TestCase(12, 2, ExpectedResult = 6)]
[TestCase(12, 4, ExpectedResult = 3)]
public int DivideTest(int n, int d) => n / d;
```

**Expected Output**

```
Passed: DivideTest(12, 3)
Passed: DivideTest(12, 2)
Passed: DivideTest(12, 4)
```

**Why this output occurs**

- NUnit compares the method’s return value to `ExpectedResult`.
- No explicit assertion is needed in the body.

---

#### Example 2: SetUp and TearDown

```csharp
[TestFixture]
public class DatabaseTests
{
    private TestDatabase _db;

    [OneTimeSetUp]
    public void OneTimeSetUp() => _db = TestDatabase.Create();

    [SetUp]
    public void SetUp() => _db.Clear();

    [Test]
    public void Insert_AddsRow() { _db.Insert("a"); Assert.That(_db.Count, Is.EqualTo(1)); }

    [OneTimeTearDown]
    public void OneTimeTearDown() => _db.Dispose();
}
```

**Expected Output**

```
Passed: Insert_AddsRow
```

**Why this output occurs**

- `[OneTimeSetUp]` runs once before all tests.
- `[SetUp]` clears state before each test.
- `[OneTimeTearDown]` cleans up after all tests.

---

### Real-World Cases

- **Legacy .NET Framework projects** — NUnit has deep historical adoption.
- **Complex parameterization** — `[TestCaseSource]` and `[Values]` for combinatorial data.
- **Categorized test suites** — `[Category]` for filtering in CI.

---

### References

- NUnit official documentation - https://docs.nunit.org/
- NUnit TestCase attribute - https://docs.nunit.org/articles/nunit/writing-tests/attributes/testcase.html
- NUnit vs xUnit vs MSTest — BrowserStack - https://www.browserstack.com/guide/nunit-vs-xunit-vs-mstest

---

## 4. MSTest

### Definitions

**Core Definition**
MSTest is Microsoft’s built-in unit testing framework for .NET, integrated with Visual Studio and the .NET SDK, providing `[TestClass]`, `[TestMethod]`, and a rich assertion API.

**Technical Definition**
MSTest is part of the `Microsoft.VisualStudio.TestTools.UnitTesting` namespace. Test classes are marked with `[TestClass]`, and test methods with `[TestMethod]`. MSTest supports lifecycle attributes (`[TestInitialize]`, `[TestCleanup]`, `[ClassInitialize]`, `[ClassCleanup]`), data-driven testing (`[DataRow]`, `[DynamicData]`), and categorization (`[TestCategory]`). In MSTest v3.8+, `Assert.Throws<T>` and `Assert.ThrowsExactly<T>` are preferred over the legacy `Assert.ThrowsException<T>`.

**Beginner-Friendly Explanation**
MSTest is Microsoft’s testing framework. It comes with Visual Studio and uses `[TestClass]` and `[TestMethod]`. It integrates well with the Microsoft toolchain.

---

### Purposes

- To provide first-party testing support in Visual Studio.
- To integrate seamlessly with Azure DevOps and Microsoft tooling.
- To support data-driven tests with `[DataRow]` and `[DynamicData]`.
- To offer lifecycle hooks for setup and cleanup.
- To provide a familiar API for teams already in the Microsoft ecosystem.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class CalculatorTests
{
    private Calculator _calculator;

    [TestInitialize]
    public void Setup() => _calculator = new Calculator();

    [TestMethod]
    public void Add_TwoNumbers_ReturnsSum()
    {
        Assert.AreEqual(5, _calculator.Add(2, 3));
    }

    [DataTestMethod]
    [DataRow(1, 2, 3)]
    [DataRow(10, 20, 30)]
    public void Add_DataDriven_ReturnsSum(int a, int b, int expected)
    {
        Assert.AreEqual(expected, _calculator.Add(a, b));
    }
}
```

**Component breakdown**

- `[TestClass]` — Marks the test class.
- `[TestMethod]` — Marks a test method.
- `[TestInitialize]` — Runs before each test.
- `[TestCleanup]` — Runs after each test.
- `[DataTestMethod]` + `[DataRow(...)]` — Data-driven test.
- `Assert.AreEqual` — Asserts equality.

#### Syntax Rules

- Test classes must have `[TestClass]`.
- Test methods must have `[TestMethod]` (or `[DataTestMethod]`).
- Test methods must be `public`, non-static, return `void` or `Task`, and have no parameters (unless data-driven).
- Do not use `async void` for test methods; use `async Task`.
- In MSTest 3.8+, prefer `Assert.Throws<T>` / `Assert.ThrowsExactly<T>` over `Assert.ThrowsException<T>`.

#### Constraints and Limitations

- MSTest is historically less flexible than NUnit and xUnit.
- Data-driven attributes differ from other frameworks (`[DataRow]` vs. `[InlineData]`).
- Parallelization configuration is less automatic than xUnit.
- Naming conventions are more verbose (`[TestClass]`, `[TestMethod]`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: TestMethod and TestInitialize

```csharp
[TestClass]
public class OrderServiceTests
{
    private OrderService _service;

    [TestInitialize]
    public void Initialize() => _service = new OrderService();

    [TestMethod]
    public void CreateOrder_ValidItems_ReturnsOrderId()
    {
        var id = _service.CreateOrder(new[] { new OrderItem { Price = 10m } });
        Assert.IsTrue(id > 0);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void CreateOrder_EmptyItems_Throws()
    {
        _service.CreateOrder(Array.Empty<OrderItem>());
    }
}
```

**Expected Output**

```
Passed: CreateOrder_ValidItems_ReturnsOrderId
Passed: CreateOrder_EmptyItems_Throws
```

**Why this output occurs**

- `[TestInitialize]` creates a fresh service before each test.
- `[ExpectedException]` asserts that the exception is thrown (legacy approach; `Assert.Throws` is preferred in v3.8+).

---

#### Example 2: DataTestMethod with DataRow

```csharp
[TestClass]
public class DiscountTests
{
    [DataTestMethod]
    [DataRow(100.0, 0.1, 90.0)]
    [DataRow(200.0, 0.2, 160.0)]
    public void ApplyDiscount_ReturnsReducedPrice(double price, double rate, double expected)
    {
        var discount = new DiscountService();
        Assert.AreEqual(expected, discount.Apply((decimal)price, (decimal)rate), 0.001);
    }
}
```

**Expected Output**

```
Passed: ApplyDiscount_ReturnsReducedPrice (100, 0.1, 90)
Passed: ApplyDiscount_ReturnsReducedPrice (200, 0.2, 160)
```

**Why this output occurs**

- Each `[DataRow]` produces a separate test case.
- `Assert.AreEqual` with a delta tolerates floating-point imprecision.

---

### Real-World Cases

- **Enterprise .NET Framework projects** — MSTest is deeply integrated with Visual Studio.
- **Azure DevOps pipelines** — Native support for MSTest results and coverage.
- **Legacy test suites** — Migration to xUnit is common but MSTest remains supported.

---

### References

- Write tests with MSTest — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-writing-tests
- MSTest assertions — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-assertions
- NUnit vs xUnit vs MSTest — BrowserStack - https://www.browserstack.com/guide/nunit-vs-xunit-vs-mstest

---

## 5. Test Attributes

### Definitions

**Core Definition**
Test attributes are declarative markers that identify test classes, test methods, data sources, and lifecycle hooks to the test framework.

**Technical Definition**
Each framework defines its own attribute set. xUnit uses `[Fact]` and `[Theory]`; NUnit uses `[Test]` and `[TestCase]`; MSTest uses `[TestClass]` and `[TestMethod]`. Attributes control discovery, parameterization, skipping, categorization, and lifecycle. The same conceptual attribute may have different names and capabilities across frameworks.

**Beginner-Friendly Explanation**
Attributes are labels you put on classes and methods to tell the test framework "this is a test" or "this method supplies data."

---

### Sub-Feature: `[Fact]` (xUnit)

#### Definitions

**Core Definition**
`[Fact]` marks a parameterless method as a single, unconditional test in xUnit.

**Technical Definition**
`FactAttribute` is the fundamental xUnit test attribute. It discovers and executes the method once. It supports properties such as `Skip` (to skip the test), `DisplayName`, and `Timeout`. It cannot accept parameters; for parameterized tests, use `[Theory]`.

**Beginner-Friendly Explanation**
`[Fact]` means "this method is one test that always runs the same way."

#### Purposes

- To mark a method as a test.
- To provide a single, deterministic test case.
- To support skipping with a reason.
- To integrate with xUnit discovery.

#### Syntax Rules

```csharp
[Fact]
public void MethodName_Scenario_ExpectedBehavior()
{
    // Arrange-Act-Assert
}

[Fact(Skip = "Not implemented yet")]
public void PendingTest() { }
```

#### Constraints

- The method must be public, non-static, parameterless, and return `void` or `Task`.
- Cannot supply parameters; use `[Theory]` instead.

#### References

- xUnit FactAttribute - https://xunit.net/docs/getting-started/netcore/cmdline

---

### Sub-Feature: `[Theory]` (xUnit)

#### Definitions

**Core Definition**
`[Theory]` marks a parameterized test method whose data rows are supplied by `[InlineData]`, `[MemberData]`, or `[ClassData]`.

**Technical Definition**
`TheoryAttribute` discovers the method and generates one test case per data row. The method parameters must match the data row types. `[InlineData]` supplies compile-time constants; `[MemberData]` references a static property or method; `[ClassData]` references a class implementing `IEnumerable<object[]>`.

**Beginner-Friendly Explanation**
`[Theory]` means "this method is a theory that is tested with several sets of data."

#### Purposes

- To reduce duplication across similar test cases.
- To improve coverage with multiple inputs.
- To separate test logic from test data.
- To support complex data sources.

#### Syntax Rules

```csharp
[Theory]
[InlineData(1, 2, 3)]
[InlineData(10, 20, 30)]
public void Add_DataDriven_ReturnsSum(int a, int b, int expected) { ... }

[Theory]
[MemberData(nameof(TestData))]
public void Test_WithMemberData(int value) { ... }

[Theory]
[ClassData(typeof(TestDataGenerator))]
public void Test_WithClassData(int value) { ... }
```

**Component breakdown**

- `[Theory]` — Enables data-driven execution.
- `[InlineData]` — Inline arguments.
- `[MemberData]` — Static member returning `IEnumerable<object[]>`.
- `[ClassData]` — Type implementing `IEnumerable<object[]>`.

#### Constraints

- Parameter types must match the data row types.
- `[InlineData]` arguments must be compile-time constants.
- Each data row is reported as a separate test case.

#### References

- xUnit Theory and data-driven tests - https://xunit.net/docs/getting-started/netcore/cmdline

---

### Sub-Feature: `[Test]` (NUnit)

#### Definitions

**Core Definition**
`[Test]` marks a method as a test in NUnit.

**Technical Definition**
`TestAttribute` discovers a parameterless test method. For parameterized tests, use `[TestCase]` or `[TestCaseSource]`. `[Test]` can be combined with `[Category]`, `[Ignore]`, and `[Description]`.

**Beginner-Friendly Explanation**
`[Test]` means "this method is a test."

#### Purposes

- To mark a method as a test.
- To support simple, parameterless tests.
- To integrate with NUnit lifecycle attributes.

#### Syntax Rules

```csharp
[Test]
public void MethodName_Scenario_ExpectedBehavior() { ... }

[Test, Ignore("Reason")]
public void SkippedTest() { }
```

#### Constraints

- The method must be public, non-static, and parameterless.
- Use `[TestCase]` for parameterized tests.

#### References

- NUnit Test attribute - https://docs.nunit.org/articles/nunit/writing-tests/attributes/test.html

---

### Sub-Feature: `[TestCase]` (NUnit)

#### Definitions

**Core Definition**
`[TestCase]` marks a parameterized method as a test and supplies inline arguments for each case.

**Technical Definition**
`TestCaseAttribute` serves the dual purpose of marking the method as a test and providing arguments. It can be applied multiple times to create multiple cases. `ExpectedResult` verifies the return value automatically. Named parameters control metadata (`Ignore`, `Category`, `Author`, `Description`, `TestName`, `Explicit`).

**Beginner-Friendly Explanation**
`[TestCase]` means "run this test with these specific arguments."

#### Purposes

- To inline test data without separate data sources.
- To create multiple cases from one method.
- To verify return values with `ExpectedResult`.
- To attach per-case metadata.

#### Syntax Rules

```csharp
[TestCase(12, 3, 4)]
[TestCase(12, 2, 6)]
public void DivideTest(int n, int d, int q) => Assert.That(n / d, Is.EqualTo(q));

[TestCase(12, 3, ExpectedResult = 4)]
public int DivideTestWithResult(int n, int d) => n / d;
```

#### Constraints

- Arguments must be compile-time constants (with conversion attempts).
- Once any parameter has explicit data, all parameters need a data source.
- `ExpectedResult` requires the method to return a compatible type.

#### References

- NUnit TestCase attribute - https://docs.nunit.org/articles/nunit/writing-tests/attributes/testcase.html

---

## 6. Assertions

### Definitions

**Core Definition**
Assertions are statements that verify the expected outcome of a test and fail the test if the outcome does not match.

**Technical Definition**
Assertions compare actual values to expected values, check for null, verify exception types, validate collections, and check boolean conditions. xUnit uses the static `Assert` class with methods like `Equal`, `NotEqual`, `True`, `False`, `Null`, `NotNull`, `Contains`, `Throws`, `ThrowsAsync`, `InRange`, `Matches`, and `Same`. NUnit uses the constraint model (`Assert.That(actual, Is.EqualTo(expected))`) plus `Assert.Throws`. MSTest uses `Assert.AreEqual`, `Assert.IsTrue`, `Assert.ThrowsException` (legacy), and in v3.8+ `Assert.Throws` / `Assert.ThrowsExactly`.

**Beginner-Friendly Explanation**
An assertion is "checking the answer." You compare what your code returned to what you expected, and the test fails if they do not match.

---

### Purposes

- To verify return values and state.
- To confirm that exceptions are thrown.
- To validate collections and object properties.
- To fail fast with a clear message.
- To provide diagnostic output on failure.

---

### Syntax Rules and Structure

#### General Syntax (xUnit)

```csharp
Assert.Equal(expected, actual);
Assert.NotEqual(notExpected, actual);
Assert.True(condition);
Assert.False(condition);
Assert.Null(obj);
Assert.NotNull(obj);
Assert.Contains(expectedSubstring, actualString);
Assert.DoesNotContain(unexpected, collection);
Assert.InRange(actual, low, high);
Assert.Matches(pattern, actual);

var ex = Assert.Throws<InvalidOperationException>(() => Method());
Assert.Equal("expected message", ex.Message);

await Assert.ThrowsAsync<InvalidOperationException>(async () => await MethodAsync());
```

**Component breakdown**

- `Assert.Equal` — Value equality.
- `Assert.True` / `False` — Boolean checks.
- `Assert.Null` / `NotNull` — Null checks.
- `Assert.Contains` — String or collection containment.
- `Assert.Throws<T>` — Synchronous exception assertion.
- `Assert.ThrowsAsync<T>` — Asynchronous exception assertion.

#### Syntax Rules (NUnit)

```csharp
Assert.That(actual, Is.EqualTo(expected));
Assert.That(actual, Is.Not.Null);
Assert.That(collection, Has.Count.EqualTo(3));
Assert.That(() => Method(), Throws.TypeOf<InvalidOperationException>());
Assert.That(ex.Message, Is.EqualTo("expected"));
```

#### Syntax Rules (MSTest)

```csharp
Assert.AreEqual(expected, actual);
Assert.AreNotEqual(notExpected, actual);
Assert.IsTrue(condition);
Assert.IsFalse(condition);
Assert.IsNull(obj);
Assert.IsNotNull(obj);
Assert.ThrowsException<InvalidOperationException>(() => Method()); // legacy
Assert.Throws<InvalidOperationException>(() => Method()); // v3.8+
```

#### Constraints and Limitations

- One logical assertion per test is recommended (but multiple related assertions are acceptable).
- Over-asserting couples tests to implementation details.
- `Assert.AreEqual` on objects uses reference equality unless overridden.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Exception Assertion in xUnit

```csharp
[Fact]
public void Divide_ByZero_ThrowsWithMessage()
{
    var calculator = new Calculator();

    var ex = Assert.Throws<DivideByZeroException>(() => calculator.Divide(10, 0));

    Assert.Equal("Attempted to divide by zero.", ex.Message);
}
```

**Expected Output**

```
Passed: Divide_ByZero_ThrowsWithMessage
```

**Why this output occurs**

- `Assert.Throws` captures the exception.
- The message assertion verifies the exception details.

---

#### Example 2: Async Exception Assertion in xUnit

```csharp
[Fact]
public async Task GetAsync_NotFound_ThrowsNotFoundException()
{
    var service = new UserService(new FakeRepository());

    var ex = await Assert.ThrowsAsync<NotFoundException>(
        () => service.GetAsync(99));

    Assert.Equal("User 99 not found", ex.Message);
}
```

**Expected Output**

```
Passed: GetAsync_NotFound_ThrowsNotFoundException
```

**Why this output occurs**

- `Assert.ThrowsAsync` awaits the task and captures the exception.
- The test method must be `async Task`.

---

### Real-World Cases

- **API tests** — Assert status codes and response bodies.
- **Domain tests** — Assert invariants and exception contracts.
- **Collection tests** — Assert ordering, counts, and contents.

---

### References

- xUnit Assert class API - https://api.xunit.net/
- NUnit assertion model - https://docs.nunit.org/articles/nunit/writing-tests/assertions/assertions.html
- MSTest assertions — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-assertions
- xUnit ThrowsAsync rule - https://xunit.net/xunit.analyzers/rules/xUnit2014

---

## 7. Test Fixtures

### Definitions

**Core Definition**
A test fixture is a shared context — setup and teardown logic and shared resources — that multiple tests use.

**Technical Definition**
Fixtures control the lifecycle and sharing of expensive resources (database connections, HTTP servers, configuration). In xUnit, `IClassFixture<T>` shares a fixture instance across all tests in a single test class; `ICollectionFixture<T>` shares a fixture across all test classes in a collection, declared on a collection definition class. xUnit v3 also supports `AssemblyFixtureAttribute` for assembly-wide sharing. NUnit uses `[OneTimeSetUp]`/`[OneTimeTearDown]` on the fixture class. MSTest uses `[ClassInitialize]`/`[ClassCleanup]` (static) for class-level sharing.

**Beginner-Friendly Explanation**
A fixture is "shared setup." If many tests need the same database or web server, you create it once in a fixture instead of recreating it for every test.

---

### Purposes

- To share expensive resources across tests.
- To reduce setup time.
- To centralize configuration and cleanup.
- To control the scope of sharing (class, collection, assembly).
- To manage lifecycle correctly (create once, dispose once).

---

### Syntax Rules and Structure

#### xUnit — `IClassFixture<T>`

```csharp
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture()
    {
        // create database once
    }

    public void Dispose()
    {
        // dispose database
    }
}

public class RepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public RepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void Query_ReturnsData() { ... }
}
```

**Component breakdown**

- `IClassFixture<T>` — Implemented by the test class; the fixture is created once per test class.
- Constructor injection — The fixture is passed to the test class constructor.

#### xUnit — `ICollectionFixture<T>`

```csharp
[CollectionDefinition("Database")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("Database")]
public class RepositoryTests
{
    public RepositoryTests(DatabaseFixture fixture) { ... }
}
```

**Component breakdown**

- `[CollectionDefinition]` — Declares a collection and its fixture.
- `[Collection("Database")]` — Marks test classes as members of the collection.
- The fixture is created once per collection and shared across all member classes.

#### NUnit — `[OneTimeSetUp]` / `[OneTimeTearDown]`

```csharp
[TestFixture]
public class DatabaseTests
{
    private Database _db;

    [OneTimeSetUp]
    public void OneTimeSetUp() => _db = Database.Create();

    [OneTimeTearDown]
    public void OneTimeTearDown() => _db.Dispose();
}
```

#### MSTest — `[ClassInitialize]` / `[ClassCleanup]`

```csharp
[TestClass]
public class DatabaseTests
{
    private static Database _db;

    [ClassInitialize]
    public static void ClassInitialize(TestContext context) => _db = Database.Create();

    [ClassCleanup]
    public static void ClassCleanup() => _db.Dispose();
}
```

#### Syntax Rules

- xUnit fixtures are created once per scope (class or collection) and disposed when the scope ends.
- Collection fixtures must be declared on a collection definition class, not on a test class.
- Fixture constructors should be lightweight or perform shared setup.
- Fixtures must be thread-safe if tests run in parallel.

#### Constraints and Limitations

- Shared fixtures can introduce test coupling if state is not reset.
- xUnit creates a new test class instance per test, but the fixture is shared.
- MSTest `[ClassInitialize]` and `[ClassCleanup]` must be static.
- NUnit `[OneTimeSetUp]` runs once per fixture.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: xUnit Class Fixture

```csharp
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture()
    {
        Connection = new SqlConnection("...");
        Connection.Open();
    }

    public SqlConnection Connection { get; }

    public void Dispose() => Connection.Dispose();
}

public class PatientRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public PatientRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void GetById_ReturnsPatient()
    {
        var repo = new PatientRepository(_fixture.Connection);
        var patient = repo.GetById(1);
        Assert.NotNull(patient);
    }
}
```

**Expected Output**

```
Passed: GetById_ReturnsPatient
```

**Why this output occurs**

- The fixture creates the connection once for the test class.
- The test uses the shared connection.

---

### Real-World Cases

- **Database integration tests** — Shared connection or container.
- **Web API integration tests** — `WebApplicationFactory` shared across tests.
- **Configuration loading** — Shared configuration object.

---

### References

- xUnit shared context (fixtures) - https://xunit.net/docs/shared-context
- xUnit fixture analyzer xUnit1041 - https://xunit.net/xunit.analyzers/rules/xUnit1041
- NUnit lifecycle attributes - https://docs.nunit.org/articles/nunit/writing-tests/attributes/setup.html
- MSTest lifecycle attributes — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-writing-tests

---

## 8. Test Organization

### Definitions

**Core Definition**
Test organization is the structuring of test code into projects, namespaces, classes, and categories so that tests are discoverable, maintainable, and runnable in meaningful subsets.

**Technical Definition**
Test organization encompasses project layout (`src/` vs. `tests/`), naming conventions, class grouping, categorization (`[Trait]` in xUnit, `[Category]` in NUnit, `[TestCategory]` in MSTest), collection definitions for shared fixtures, and parallelization configuration. Good organization mirrors the production code structure and makes it easy to run targeted subsets in CI.

**Beginner-Friendly Explanation**
Test organization is "keeping your tests tidy." You put tests in separate projects, name them clearly, and group them so you can run just the tests you need.

---

### Purposes

- To make tests easy to find and maintain.
- To mirror production code structure.
- To enable selective test execution (by category, class, or project).
- To control parallelization and shared resources.
- To support CI pipelines with targeted test stages.

---

### Syntax Rules and Structure

#### Project Layout

```
src/
  MyApp.Domain/
  MyApp.Infrastructure/
  MyApp.Api/
tests/
  MyApp.Domain.Tests/
  MyApp.Infrastructure.Tests/
  MyApp.Api.IntegrationTests/
```

- One test project per production project (or per layer).
- Unit and integration tests in separate projects.
- Test project names end with `.Tests` or `.IntegrationTests`.

#### Categorization (xUnit — `[Trait]`)

```csharp
[Trait("Category", "Unit")]
public class CalculatorTests { ... }

[Trait("Category", "Integration")]
public class DatabaseTests { ... }
```

#### Categorization (NUnit — `[Category]`)

```csharp
[Category("Unit")]
public class CalculatorTests { ... }
```

#### Categorization (MSTest — `[TestCategory]`)

```csharp
[TestClass]
[TestCategory("Unit")]
public class CalculatorTests { ... }
```

#### Collections and Parallelization (xUnit)

```csharp
[CollectionDefinition("Database")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("Database")]
public class RepositoryTests { ... }
```

- Tests in the same collection run sequentially; different collections run in parallel.

#### Syntax Rules

- Test classes should be grouped by the unit under test.
- Use categories to distinguish unit, integration, and E2E tests.
- Use collections to control parallelization for shared resources.
- Avoid mixing unrelated tests in the same class.

#### Constraints and Limitations

- Over-categorization adds ceremony.
- Collection fixtures limit parallelization within the collection.
- Project proliferation can slow builds.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Organized Test Project

```
tests/
  MyApp.Domain.Tests/
    Customers/
      CustomerTests.cs
    Orders/
      OrderTests.cs
  MyApp.Api.IntegrationTests/
    Customers/
      CustomerEndpointTests.cs
```

```csharp
// CustomerTests.cs
[Trait("Category", "Unit")]
public class CustomerTests
{
    [Fact]
    public void Create_ValidName_Succeeds() { ... }
}
```

**Expected Output (conceptual)**

```
dotnet test --filter "Category=Unit"       # runs domain unit tests
dotnet test --filter "Category=Integration" # runs API integration tests
```

**Why this output occurs**

- Traits allow filtering by category.
- Project structure mirrors production code.

---

### Real-World Cases

- **Large solutions** — Layer-specific test projects.
- **CI pipelines** — Unit tests on every commit; integration tests on PRs.
- **Microservices** — Per-service test projects.

---

### References

- xUnit traits and collections - https://xunit.net/docs/shared-context
- NUnit categories - https://docs.nunit.org/articles/nunit/writing-tests/attributes/category.html
- MSTest test categories — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-writing-tests
- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices

---

## 9. Unit Testing Best Practices

### Definitions

**Core Definition**
Unit testing best practices are guidelines for writing tests that are readable, reliable, fast, and effective at catching defects.

**Technical Definition**
Best practices include the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), the Arrange-Act-Assert structure, descriptive naming (`MethodName_Scenario_ExpectedBehavior`), one logical assertion per test, avoidance of logic in tests, use of test doubles appropriately, and integration into CI pipelines.

**Beginner-Friendly Explanation**
Best practices are "the habits that make tests helpful instead of painful." Good tests are fast, independent, and easy to understand.

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
I — Isolated: No shared state; no order dependency.
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
// Good: MethodName_Scenario_ExpectedBehavior
public void Add_EmptyString_ReturnsZero()

// Bad: vague
public void Test1()
```

#### Test Doubles

- Use stubs for queries (return data).
- Use mocks for commands (verify interactions).
- Prefer fakes for stateful dependencies.
- Avoid over-mocking; test behavior, not implementation.

#### One Logical Assertion

- Prefer a single logical assertion per test.
- Multiple related assertions are acceptable if they verify the same behavior.

#### Avoid Logic in Tests

- Do not use `if`, loops, or complex calculations in tests.
- Tests should be simple and linear.

#### Test Independence

- Each test should set up its own data.
- No static mutable state.
- No reliance on execution order.

#### CI Integration

```bash
dotnet test --logger "trx;LogFileName=results.trx" --collect:"XPlat Code Coverage"
```

- Run tests on every push.
- Publish results and coverage.
- Fail the build on test failures.

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

- Tests are isolated; fresh substitutes per test.
- Mocks verify interactions.
- Exception test verifies the guard clause.

---

### Real-World Cases

- **Enterprise applications** — Layered test suites with CI gates.
- **Open-source libraries** — Comprehensive unit tests and mutation testing.
- **Microservices** — Contract tests and integration tests.

---

### References

- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit Test Patterns (naming, AAA) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/19-c-sharp-best-prac-sharptic-sharpes/7-c-sharp-testing-strategies.mdx
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- NUnit vs xUnit vs MSTest — BrowserStack - https://www.browserstack.com/guide/nunit-vs-xunit-vs-mstest

---

## Framework Comparison Summary

| Feature | xUnit | NUnit | MSTest |
|---|---|---|---|
| Test method attribute | `[Fact]` / `[Theory]` | `[Test]` / `[TestCase]` | `[TestMethod]` / `[DataTestMethod]` |
| Test class attribute | None required | `[TestFixture]` (optional) | `[TestClass]` (required) |
| Inline data | `[InlineData]` | `[TestCase]` | `[DataRow]` |
| External data | `[MemberData]` / `[ClassData]` | `[TestCaseSource]` | `[DynamicData]` |
| Setup / Teardown | Constructor / `IDisposable` | `[SetUp]` / `[TearDown]` | `[TestInitialize]` / `[TestCleanup]` |
| One-time setup | `IClassFixture` / `ICollectionFixture` | `[OneTimeSetUp]` / `[OneTimeTearDown]` | `[ClassInitialize]` / `[ClassCleanup]` |
| Exception assertion | `Assert.Throws<T>` / `ThrowsAsync` | `Assert.That(() => ..., Throws.TypeOf<T>())` | `Assert.Throws<T>` (v3.8+) / `ThrowsException` |
| Categorization | `[Trait("Category", "...")]` | `[Category("...")]` | `[TestCategory("...")]` |
| Parallelization | On by default (collections) | Configurable | Configurable |
| Isolation model | New class instance per test | Per-fixture instance | Per-test instance |

*Sources: BrowserStack comparison and framework documentation*.

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `[ExpectedException]` (MSTest) | Legacy; prefer `Assert.Throws<T>` / `Assert.ThrowsExactly<T>` in MSTest 3.8+ |
| `Assert.ThrowsException<T>` (MSTest) | Legacy; `Assert.Throws<T>` preferred in v3.8+ |
| `async void` test methods | Unsafe; use `async Task` (MSTest explicitly warns against `async void`) |
| xUnit `[SetUp]` / `[TearDown]` | Not available; use constructor and `IDisposable` |
| xUnit collection fixtures on test classes | Must be declared on collection definition classes, not test classes |
| `Assert.AreEqual` on objects (MSTest) | Uses reference equality unless overridden |
| NUnit `[TestCase]` with mixed data sources | Once any parameter has explicit data, all parameters need a data source |
| xUnit v3 assembly fixtures | `AssemblyFixtureAttribute` is v3-only |
| MSTest `DiscoverInternals` | Assembly-level attribute for discovering internal tests |
| xUnit parallelization | On by default; tests sharing resources must use `[Collection]` |

---

## References (Consolidated)

- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- Unit testing in .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/
- xUnit.net documentation - https://xunit.net/
- xUnit shared context (fixtures) - https://xunit.net/docs/shared-context
- xUnit fixture analyzer xUnit1041 - https://xunit.net/xunit.analyzers/rules/xUnit1041
- xUnit ThrowsAsync rule xUnit2014 - https://xunit.net/xunit.analyzers/rules/xUnit2014
- NUnit documentation - https://docs.nunit.org/
- NUnit TestCase attribute - https://docs.nunit.org/articles/nunit/writing-tests/attributes/testcase.html
- NUnit Test attribute - https://docs.nunit.org/articles/nunit/writing-tests/attributes/test.html
- NUnit categories - https://docs.nunit.org/articles/nunit/writing-tests/attributes/category.html
- NUnit lifecycle attributes - https://docs.nunit.org/articles/nunit/writing-tests/attributes/setup.html
- MSTest — Write tests - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-writing-tests
- MSTest assertions — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-assertions
- NUnit vs xUnit vs MSTest — BrowserStack - https://www.browserstack.com/guide/nunit-vs-xunit-vs-mstest
- xUnit Test Patterns (naming, AAA) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/19-c-sharp-best-prac-sharptic-sharpes/7-c-sharp-testing-strategies.mdx
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing