# xUnit — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
xUnit is a free, open-source, community-focused unit testing framework for .NET that emphasizes simplicity, extensibility, and test isolation through a convention-based design.

**Technical Definition**
xUnit.net was written by the original creator of NUnit v2 and is designed to encourage Test-Driven Development (TDD) best practices. It provides `[Fact]` for parameterless tests, `[Theory]` with `[InlineData]`, `[MemberData]`, and `[ClassData]` for data-driven tests, a rich `Assert` class, fixture interfaces (`IClassFixture<T>`, `ICollectionFixture<T>`), and built-in parallelization. A new instance of the test class is created for every test method, enforcing isolation without attributes. xUnit v3 (current major version) targets .NET 8+ and supports the Microsoft Testing Platform and Native AOT.

**Beginner-Friendly Explanation**
xUnit is a modern testing framework. You mark test methods with `[Fact]` (a single fact) or `[Theory]` (a theory tested with data). xUnit creates a fresh object for each test, so tests do not accidentally share state. Instead of `[SetUp]` and `[TearDown]` attributes, you use the class constructor and `IDisposable` — this is xUnit's "convention over configuration" philosophy.

---

### Key Characteristics

- **Convention over configuration** — Constructor acts as setup; `IDisposable` acts as teardown. No `[SetUp]`/`[TearDown]` attributes.
- **Test isolation by default** — A new test class instance is created for every test method.
- **Parameterized testing** — `[Theory]` with `[InlineData]`, `[MemberData]`, or `[ClassData]` for data-driven tests.
- **Parallel by default** — xUnit runs test collections in parallel; use `[Collection]` to control parallelism when tests share resources.
- **Rich assertion API** — `Assert.Equal`, `Assert.Throws`, `Assert.Collection`, and more.
- **Fixture system** — `IClassFixture<T>` for sharing within a class; `ICollectionFixture<T>` for sharing across classes.
- **Arrange-Act-Assert** — The recommended structure for every test.
- **Descriptive naming** — `MethodName_Scenario_ExpectedBehavior`.

---

### Prerequisites

- Working knowledge of C# classes, interfaces, methods, and properties.
- Understanding of dependency injection and the Dependency Inversion Principle.
- Familiarity with the .NET CLI (`dotnet test`, `dotnet build`).
- The xUnit NuGet packages (`xunit`, `xunit.runner.visualstudio`).
- Basic understanding of `async`/`await` for testing asynchronous code.
- Awareness of test doubles (stubs, mocks, fakes) for isolating dependencies.

---

### Related Programming Areas

- **Test-driven development (TDD)** — Writing tests before production code.
- **Behavior-driven development (BDD)** — Specifying behavior in business-readable language.
- **Integration testing** — Verifying component interactions with `WebApplicationFactory`.
- **Test doubles and mocking** — Moq, NSubstitute, FakeItEasy.
- **Mutation testing** — Stryker.NET for evaluating test suite quality.
- **Code coverage** — Coverlet and ReportGenerator.
- **Continuous integration** — Automated test execution on every commit.

---

### Core Concepts / Features

1. xUnit (overarching framework)
2. `[Fact]`
3. `[Theory]`
4. `[InlineData]`
5. `[MemberData]`
6. `[ClassData]`
7. Fixtures (shared context)
8. `IClassFixture<T>`
9. `ICollectionFixture<T>`
10. xUnit best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. xUnit (Overarching Framework)

### Definitions

**Core Definition**
xUnit is a testing framework that provides attributes for discovering and executing tests, an assertion library for verifying outcomes, and a fixture system for sharing setup across tests.

**Technical Definition**
xUnit.net is distributed as NuGet packages (`xunit`, `xunit.runner.visualstudio`, `xunit.v3` for v3) and integrates with `dotnet test` and Visual Studio. Test classes are public, non-static classes; test methods are public, return `void` or `Task`, and are marked with `[Fact]` or `[Theory]`. xUnit creates a new instance of the test class for each test method, which means instance fields are never shared between tests. Setup logic goes in the constructor; teardown logic goes in `IDisposable.Dispose` or `IAsyncLifetime` for async cleanup. xUnit v3 targets .NET 8+ and uses the Microsoft Testing Platform.

**Beginner-Friendly Explanation**
xUnit is the "test runner" plus the "test toolkit." It finds your `[Fact]` and `[Theory]` methods, runs them, and reports pass or fail. It also gives you `Assert` methods to check results and fixtures to share expensive setup.

---

### Purposes

- To provide a simple, convention-based testing framework for .NET.
- To enforce test isolation by creating a new test class instance per test.
- To support data-driven testing with `[Theory]`.
- To enable parallel test execution by default.
- To offer extensibility through custom facts, theories, and fixtures.
- To integrate with `dotnet test` and Visual Studio.

---

### Syntax Rules and Structure

#### Project Setup

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsPackable>false</IsPackable>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
    <PackageReference Include="xunit" Version="2.7.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.7" />
  </ItemGroup>
  <ItemGroup>
    <Using Include="Xunit" />
  </ItemGroup>
</Project>
```

```bash
dotnet test
dotnet test --filter "FullyQualifiedName~CalculatorTests"
dotnet test --collect:"XPlat Code Coverage"
```

**Component breakdown**

- `Microsoft.NET.Test.Sdk` — Required test platform SDK.
- `xunit` — The test framework package.
- `xunit.runner.visualstudio` — Adapter for Visual Studio and `dotnet test`.
- `<Using Include="Xunit" />` — Global using for the `Xunit` namespace.
- `dotnet test` — Discovers and executes tests.

#### Syntax Rules

- Test classes are public and non-static.
- Test methods are public, return `void` or `Task`, and have no parameters for `[Fact]`.
- xUnit creates a new instance of the test class for each test method.
- Setup is done in the constructor; teardown via `IDisposable` or `IAsyncLifetime`.
- There are no `[SetUp]`/`[TearDown]` attributes; use constructors and `Dispose`.

#### Constraints and Limitations

- xUnit is opinionated and restrictive compared with NUnit.
- No built-in `[SetUp]`/`[TearDown]`; use constructor and `IDisposable`.
- Parallelization is on by default; tests must be isolated.
- Fixture scope is controlled by `IClassFixture` and `ICollectionFixture`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Test Class

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
    private readonly Calculator _calculator;

    public CalculatorTests()
    {
        // Constructor acts as setup — runs before each test
        _calculator = new Calculator();
    }

    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange (already done in constructor)

        // Act
        var result = _calculator.Add(2, 3);

        // Assert
        Assert.Equal(5, result);
    }

    [Fact]
    public void Divide_ByZero_ThrowsDivideByZeroException()
    {
        Assert.Throws<DivideByZeroException>(() => _calculator.Divide(10, 0));
    }
}
```

**Line-by-line comments**

- `Calculator` — Production class under test.
- `CalculatorTests` — Test class; public, non-static.
- Constructor — Runs before each test; creates a fresh `Calculator` instance.
- `[Fact]` — Marks a parameterless test method.
- `Assert.Equal` — Verifies the return value.
- `Assert.Throws<T>` — Verifies the exception type.

**Expected Output**

```
Passed: Add_TwoPositiveNumbers_ReturnsSum
Passed: Divide_ByZero_ThrowsDivideByZeroException
```

**Why this output occurs**

- Each test gets a fresh `Calculator` because xUnit creates a new test class instance per test.
- `Add(2, 3)` returns `5`, matching the assertion.
- `Divide(10, 0)` throws `DivideByZeroException`, which is asserted.

---

### Real-World Cases

- **Open-source libraries** — xUnit is the dominant framework in the .NET OSS ecosystem.
- **ASP.NET Core** — The default test framework in `dotnet new xunit` templates.
- **Data-driven tests** — Pricing tiers, validation rules, and parsing logic.

---

### References

- xUnit.net official documentation - https://xunit.net/
- Getting Started with xUnit.net v3 - https://xunit.net/docs/getting-started/v3/getting-started
- Unit testing best practices — .NET (xUnit examples) - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices

---

## 2. `[Fact]`

### Definitions

**Core Definition**
`[Fact]` marks a parameterless method as a single, unconditional test in xUnit.

**Technical Definition**
`FactAttribute` is the fundamental xUnit test attribute. It discovers and executes the method exactly once. It supports properties such as `Skip` (to skip the test), `DisplayName`, and `Timeout`. It cannot accept parameters; for parameterized tests, use `[Theory]`. A `[Fact]` test behaves as a test with no test data passed in as a parameter.

**Beginner-Friendly Explanation**
`[Fact]` means "this method is one test that always runs the same way." It has no inputs — you set up everything inside the method itself.

---

### Purposes

- To mark a method as a test.
- To provide a single, deterministic test case.
- To support skipping with a reason.
- To integrate with xUnit discovery.
- To serve as the simplest form of automated test.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
[Fact]
public void MethodName_Scenario_ExpectedBehavior()
{
    // Arrange-Act-Assert
}

[Fact(Skip = "Not implemented yet")]
public void PendingTest() { }

[Fact(DisplayName = "Addition of two positive numbers")]
public void Add_TwoNumbers_ReturnsSum() { }
```

**Component breakdown**

- `[Fact]` — The attribute that marks the method.
- `Skip = "reason"` — Skips the test and reports the reason.
- `DisplayName = "..."` — Overrides the test name in reports.

#### Syntax Rules

- The method must be public, non-static, parameterless, and return `void` or `Task`.
- The test class must be public and non-static.
- The method can be `async Task` for asynchronous tests; `async void` is unsafe.

#### Constraints and Limitations

- Cannot supply parameters; use `[Theory]` for data-driven tests.
- Cannot have overloads with different parameters in the same class.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Simple Fact

```csharp
[Fact]
public void Add_EmptyString_ReturnsZero()
{
    var calculator = new StringCalculator();
    var result = calculator.Add("");
    Assert.Equal(0, result);
}
```

**Expected Output**

```
Passed: Add_EmptyString_ReturnsZero
```

**Why this output occurs**

- The test runs once, calls `Add("")`, and asserts the result is `0`.

---

#### Example 2: Skipped Fact

```csharp
[Fact(Skip = "Feature not implemented")]
public void Export_ToPdf_ReturnsFile()
{
    // ...
}
```

**Expected Output**

```
Skipped: Export_ToPdf_ReturnsFile (Feature not implemented)
```

**Why this output occurs**

- The `Skip` property prevents execution and reports the reason.

---

### Real-World Cases

- **Simple unit tests** — Methods with no varying inputs.
- **Regression tests** — One scenario per test.
- **Guard clause tests** — Verifying exceptions for invalid inputs.

---

### References

- xUnit FactAttribute API - https://xunit.net/docs/getting-started/v3/getting-started

---

## 3. `[Theory]`

### Definitions

**Core Definition**
`[Theory]` marks a parameterized test method whose data rows are supplied by `[InlineData]`, `[MemberData]`, or `[ClassData]`.

**Technical Definition**
`TheoryAttribute` discovers the method and generates one test case per data row. The method parameters must match the data row types. A `[Theory]` method must have at least one data attribute; otherwise the analyzer emits xUnit1003 ("Theory methods must have test data"). Each data row is reported as a separate test case in the test runner.

**Beginner-Friendly Explanation**
`[Theory]` means "this method is a theory that is tested with several sets of data." Instead of writing five nearly identical `[Fact]` methods, you write one `[Theory]` and supply five data rows.

---

### Purposes

- To reduce duplication across similar test cases.
- To improve coverage with multiple inputs.
- To separate test logic from test data.
- To support complex data sources.
- To report each data row as a distinct test case.

---

### Syntax Rules and Structure

#### General Syntax

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
- `[MemberData]` — Static member returning test data.
- `[ClassData]` — Type implementing `IEnumerable<object[]>`.

#### Syntax Rules

- The method must have parameters; parameter types must match data row types.
- At least one data attribute is required (xUnit1003).
- Each data row is reported as a separate test case.
- The method can be `async Task` for asynchronous tests.

#### Constraints and Limitations

- `[InlineData]` arguments must be compile-time constants.
- `[MemberData]` and `[ClassData]` argument counts must match method parameters (xUnit1037, xUnit1038).
- Data rows are immutable; the test cannot modify them.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Theory with InlineData

```csharp
[Theory]
[InlineData("1", 1)]
[InlineData("1,2", 3)]
[InlineData("10,20,30", 60)]
public void Add_CommaSeparatedNumbers_ReturnsSum(string input, int expected)
{
    var calc = new StringCalculator();
    Assert.Equal(expected, calc.Add(input));
}
```

**Expected Output**

```
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "1", expected: 1)
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "1,2", expected: 3)
Passed: Add_CommaSeparatedNumbers_ReturnsSum(input: "10,20,30", expected: 60)
```

**Why this output occurs**

- Each `[InlineData]` row produces a separate test case.
- The method runs three times with different inputs.

---

### Real-World Cases

- **Pricing tiers** — Different discount rates for different price ranges.
- **Validation rules** — Valid and invalid inputs for a validator.
- **Parsing logic** — Multiple string formats producing the same result.

---

### References

- xUnit Theory and data-driven tests - https://xunit.net/docs/getting-started/v3/getting-started
- xUnit1003 — Theory methods must have test data - https://xunit.net/xunit.analyzers/rules/xUnit1003

---

## 4. `[InlineData]`

### Definitions

**Core Definition**
`[InlineData]` supplies inline, compile-time constant arguments for a `[Theory]` test method.

**Technical Definition**
`InlineDataAttribute` is applied multiple times to a `[Theory]` method, each application providing one data row. The arguments must be compile-time constants (or `typeof` expressions, or arrays of constants). The attribute can also be combined with `[MemberData]` or `[ClassData]` on the same method, though this is uncommon.

**Beginner-Friendly Explanation**
`[InlineData]` means "run this theory with these specific values written right here in the code."

---

### Purposes

- To provide simple, readable test data inline.
- To avoid separate data source classes for trivial cases.
- To make test cases immediately visible in the source code.
- To support quick iteration on test scenarios.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
[Theory]
[InlineData(1, 2, 3)]
[InlineData(10, 20, 30)]
public void Add_DataDriven_ReturnsSum(int a, int b, int expected) { ... }

[Theory]
[InlineData("valid@email.com", true)]
[InlineData("invalid-email", false)]
public void IsValid_Email_ReturnsExpected(string email, bool expected) { ... }
```

**Component breakdown**

- `[InlineData(...)]` — One row of arguments.
- Multiple attributes — Multiple test cases.
- Argument types — Must match the method parameter types.

#### Syntax Rules

- Arguments must be compile-time constants (literals, `const` values, `typeof`, `nameof`).
- Cannot pass `null` directly in some older versions; use `[InlineData(null)]` with nullable types.
- Arrays can be passed as `new[] { ... }` in C# 12+; otherwise use `[MemberData]`.

#### Constraints and Limitations

- Cannot supply complex objects or runtime-computed data.
- Cannot share data across test classes (use `[MemberData]` or `[ClassData]`).
- Large data sets make the source file unwieldy.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiple InlineData Rows

```csharp
[Theory]
[InlineData(1, 2, 3)]
[InlineData(-1, 1, 0)]
[InlineData(0, 0, 0)]
public void Add_Integers_ReturnsSum(int a, int b, int expected)
{
    var calculator = new Calculator();
    Assert.Equal(expected, calculator.Add(a, b));
}
```

**Expected Output**

```
Passed: Add_Integers_ReturnsSum(a: 1, b: 2, expected: 3)
Passed: Add_Integers_ReturnsSum(a: -1, b: 1, expected: 0)
Passed: Add_Integers_ReturnsSum(a: 0, b: 0, expected: 0)
```

**Why this output occurs**

- Each attribute supplies one row; the method runs three times.

---

### Real-World Cases

- **Basic arithmetic tests** — Small, fixed sets of inputs.
- **Boolean logic tests** — True/false combinations.
- **String validation tests** — A few representative inputs.

---

### References

- xUnit InlineData attribute - https://xunit.net/docs/getting-started/v3/getting-started
- xUnit Theory data attributes - https://xunit.net/xunit.analyzers/rules/xUnit1003

---

## 5. `[MemberData]`

### Definitions

**Core Definition**
`[MemberData]` supplies test data from a static property, field, or method that returns `IEnumerable<object[]>` or `TheoryData<T...>`.

**Technical Definition**
`MemberDataAttribute` references a static member by name. The member can return `IEnumerable<object[]>`, `IEnumerable<TheoryDataRow>`, or a strongly typed `TheoryData<T1, T2, ...>` (recommended for type safety and IntelliSense). The member can be on the test class or on another class (using `MemberType`). Member data is evaluated at runtime, allowing complex objects, computed values, and external data.

**Beginner-Friendly Explanation**
`[MemberData]` means "get the test data from this static property or method." It is more flexible than `[InlineData]` because the data can be computed at runtime and can include complex objects.

---

### Purposes

- To supply complex or runtime-computed test data.
- To share test data across multiple test methods or classes.
- To keep large data sets out of the test method attributes.
- To support strongly typed data with `TheoryData<T>`.
- To generate combinatorial test cases programmatically.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Using object[]
public static IEnumerable<object[]> TestData => new[]
{
    new object[] { 1, 2, 3 },
    new object[] { 10, 20, 30 }
};

[Theory]
[MemberData(nameof(TestData))]
public void Add_DataDriven_ReturnsSum(int a, int b, int expected) { ... }
```

```csharp
// Using TheoryData<T> (recommended)
public static TheoryData<int, int, int> TestData => new()
{
    { 1, 2, 3 },
    { 10, 20, 30 }
};

[Theory]
[MemberData(nameof(TestData))]
public void Add_DataDriven_ReturnsSum(int a, int b, int expected) { ... }
```

**Component breakdown**

- `nameof(TestData)` — References the static member.
- `TheoryData<T1, T2, ...>` — Strongly typed data container.
- `IEnumerable<object[]>` — Loosely typed alternative.
- `MemberType` — Optional; specifies the class containing the member.

#### Syntax Rules

- The member must be `public static`.
- The member must return `IEnumerable<object[]>`, `IEnumerable<TheoryDataRow>`, or `TheoryData<T...>`.
- The number of arguments must match the test method parameters (xUnit1037, xUnit1038).
- The member can be a property or a method; methods are invoked when the theory runs.

#### Constraints and Limitations

- Data is evaluated at runtime, so test discovery may be slower.
- Complex objects must be serializable for some test runners (xUnit1045).
- `MemberData` members are `static`, so they cannot access instance state.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: MemberData with TheoryData

```csharp
public class DiscountTests
{
    public static TheoryData<decimal, decimal, decimal> DiscountData => new()
    {
        { 100m, 0.10m, 90m },
        { 200m, 0.20m, 160m },
        { 500m, 0.25m, 375m }
    };

    [Theory]
    [MemberData(nameof(DiscountData))]
    public void ApplyDiscount_ReturnsReducedPrice(decimal price, decimal rate, decimal expected)
    {
        var discount = new DiscountService();
        Assert.Equal(expected, discount.Apply(price, rate));
    }
}
```

**Expected Output**

```
Passed: ApplyDiscount_ReturnsReducedPrice(price: 100, rate: 0.10, expected: 90)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 200, rate: 0.20, expected: 160)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 500, rate: 0.25, expected: 375)
```

**Why this output occurs**

- `TheoryData<decimal, decimal, decimal>` supplies strongly typed rows.
- `[MemberData]` references the static property.
- Each row produces a separate test case.

---

#### Example 2: MemberData from Another Class

```csharp
public static class TestDataSources
{
    public static TheoryData<string, bool> EmailValidationData => new()
    {
        { "valid@email.com", true },
        { "invalid-email", false },
        { "another@domain.org", true }
    };
}

public class EmailValidatorTests
{
    [Theory]
    [MemberData(nameof(TestDataSources.EmailValidationData), MemberType = typeof(TestDataSources))]
    public void IsValid_Email_ReturnsExpected(string email, bool expected)
    {
        var validator = new EmailValidator();
        Assert.Equal(expected, validator.IsValid(email));
    }
}
```

**Expected Output**

```
Passed: IsValid_Email_ReturnsExpected(email: "valid@email.com", expected: True)
Passed: IsValid_Email_ReturnsExpected(email: "invalid-email", expected: False)
Passed: IsValid_Email_ReturnsExpected(email: "another@domain.org", expected: True)
```

**Why this output occurs**

- `MemberType` specifies the class containing the data member.
- The data is shared across test classes.

---

### Real-World Cases

- **Pricing engines** — Complex tiered pricing data.
- **Validation rules** — Large sets of valid and invalid inputs.
- **Shared test data** — Common data used across multiple test classes.

---

### References

- xUnit MemberData attribute - https://xunit.net/docs/getting-started/v3/getting-started
- xUnit1037 — Fewer arguments than parameters - https://xunit.net/xunit.analyzers/rules/xUnit1037
- xUnit1038 — More arguments than parameters - https://xunit.net/xunit.analyzers/rules/xUnit1038
- xUnit1045 — Serializable data - https://xunit.net/xunit.analyzers/rules/xUnit1045

---

## 6. `[ClassData]`

### Definitions

**Core Definition**
`[ClassData]` supplies test data from a class that implements `IEnumerable<object[]>` or `IEnumerable<TheoryDataRow>`.

**Technical Definition**
`ClassDataAttribute` takes a `Type` argument. The specified class must have a public parameterless constructor and implement `IEnumerable<object[]>`, `IEnumerable<TheoryDataRow>`, or `IEnumerable<TheoryData<T...>>`. The class is instantiated and enumerated when the theory runs. `ClassData` is useful for sharing data generators across test classes and separating data generation from test methods.

**Beginner-Friendly Explanation**
`[ClassData]` means "get the test data from this separate class." It is similar to `[MemberData]`, but the data lives in its own class that implements `IEnumerable`.

---

### Purposes

- To share data generators across multiple test classes.
- To separate data generation from test logic.
- To support complex, stateful data generation.
- To keep test classes focused on assertions.
- To enable reuse of data sources across projects.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
public class CalculatorTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 1, 2, 3 };
        yield return new object[] { 10, 20, 30 };
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

[Theory]
[ClassData(typeof(CalculatorTestData))]
public void Add_DataDriven_ReturnsSum(int a, int b, int expected) { ... }
```

**Component breakdown**

- `[ClassData(typeof(...))]` — Specifies the data class.
- The class must implement `IEnumerable<object[]>` (or `IEnumerable<TheoryDataRow>`).
- The class must have a public parameterless constructor.

#### Syntax Rules

- The class must be public and have a parameterless constructor.
- The class must implement `IEnumerable<object[]>` or `IEnumerable<TheoryDataRow>`.
- The data is enumerated when the theory runs.
- The argument count must match the method parameters (xUnit1037, xUnit1038).

#### Constraints and Limitations

- The data class cannot easily access test-specific state.
- Data is instantiated per theory execution; sharing mutable state is unsafe.
- More verbose than `[InlineData]` or `[MemberData]` for simple cases.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: ClassData with object[]

```csharp
public class CalculatorTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 1, 2, 3 };
        yield return new object[] { -1, 1, 0 };
        yield return new object[] { 0, 0, 0 };
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

public class CalculatorTests
{
    [Theory]
    [ClassData(typeof(CalculatorTestData))]
    public void Add_Integers_ReturnsSum(int a, int b, int expected)
    {
        var calculator = new Calculator();
        Assert.Equal(expected, calculator.Add(a, b));
    }
}
```

**Expected Output**

```
Passed: Add_Integers_ReturnsSum(a: 1, b: 2, expected: 3)
Passed: Add_Integers_ReturnsSum(a: -1, b: 1, expected: 0)
Passed: Add_Integers_ReturnsSum(a: 0, b: 0, expected: 0)
```

**Why this output occurs**

- The data class enumerates three rows.
- Each row produces a separate test case.

---

#### Example 2: ClassData with TheoryData

```csharp
public class DiscountTestData : TheoryData<decimal, decimal, decimal>
{
    public DiscountTestData()
    {
        Add(100m, 0.10m, 90m);
        Add(200m, 0.20m, 160m);
        Add(500m, 0.25m, 375m);
    }
}

[Theory]
[ClassData(typeof(DiscountTestData))]
public void ApplyDiscount_ReturnsReducedPrice(decimal price, decimal rate, decimal expected)
{
    var discount = new DiscountService();
    Assert.Equal(expected, discount.Apply(price, rate));
}
```

**Expected Output**

```
Passed: ApplyDiscount_ReturnsReducedPrice(price: 100, rate: 0.10, expected: 90)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 200, rate: 0.20, expected: 160)
Passed: ApplyDiscount_ReturnsReducedPrice(price: 500, rate: 0.25, expected: 375)
```

**Why this output occurs**

- `TheoryData<decimal, decimal, decimal>` provides strong typing.
- The data class is instantiated and enumerated.

---

### Real-World Cases

- **Shared data sources** — Reusable test data across multiple test classes.
- **Complex data generation** — Combinations of values generated programmatically.
- **Cross-project data** — Data classes shared via a common test library.

---

### References

- xUnit ClassData attribute - https://xunit.net/docs/getting-started/v3/getting-started
- xUnit Theory data attributes - https://xunit.net/xunit.analyzers/rules/xUnit1003
- xUnit1037 / xUnit1038 — Argument count rules - https://xunit.net/xunit.analyzers/rules/xUnit1037

---

## 7. Fixtures (Shared Context)

### Definitions

**Core Definition**
A fixture is a shared context — setup and teardown logic and shared resources — that multiple tests use to avoid expensive re-initialization.

**Technical Definition**
xUnit provides several mechanisms for sharing context: constructor/`Dispose` for per-test isolation (the default), `IClassFixture<T>` for sharing across all tests in a single test class, `ICollectionFixture<T>` for sharing across multiple test classes in a collection, and assembly fixtures (xUnit v3) for sharing across an entire test assembly. Fixtures are created once per scope and disposed when the scope ends. Shared fixtures must be thread-safe if tests run in parallel.

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

#### Fixture Types

| Fixture Type | Scope | How to Use |
|---|---|---|
| Constructor / `IDisposable` | Per test | Default; new instance per test |
| `IClassFixture<T>` | Per test class | Implement on test class; inject via constructor |
| `ICollectionFixture<T>` | Per collection | Define on collection definition class; apply `[Collection]` to test classes |
| Assembly fixture (v3) | Per assembly | `[assembly: AssemblyFixture(typeof(T))]` |

#### Syntax Rules

- Fixture classes should implement `IDisposable` or `IAsyncDisposable` for cleanup.
- `IClassFixture<T>` is implemented by the test class itself.
- `ICollectionFixture<T>` is declared on a collection definition class, not on a test class.
- Collection definition classes must have `[CollectionDefinition("name")]`.
- Test classes in a collection share the collection fixture and run sequentially within the collection.

#### Constraints and Limitations

- Shared fixtures can introduce test coupling if state is not reset.
- xUnit creates a new test class instance per test, but the fixture is shared.
- `ICollectionFixture` limits parallelization within the collection.
- Fixtures should not rely on test-specific state.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Per-Test Isolation (Constructor/Dispose)

```csharp
public class OrderServiceTests : IDisposable
{
    private readonly OrderService _service;

    public OrderServiceTests()
    {
        _service = new OrderService(new InMemoryOrderRepository());
    }

    public void Dispose()
    {
        // cleanup after each test
    }

    [Fact]
    public void CreateOrder_ValidItems_Succeeds()
    {
        // uses its own _service
    }
}
```

**Expected Output**

```
Passed: CreateOrder_ValidItems_Succeeds
```

**Why this output occurs**

- The constructor runs before each test; `Dispose` runs after each test.
- Each test gets a fresh service.

---

### Real-World Cases

- **Database integration tests** — Shared connection or container.
- **Web API integration tests** — `WebApplicationFactory` shared across tests.
- **Configuration loading** — Shared configuration object.

---

### References

- Sharing Context between Tests — xUnit.net - https://xunit.net/docs/shared-context
- xUnit1041 — Fixture sources - https://xunit.net/xunit.analyzers/rules/xUnit1041

---

## 8. `IClassFixture<T>`

### Definitions

**Core Definition**
`IClassFixture<T>` shares a single fixture instance across all tests in a single test class.

**Technical Definition**
`IClassFixture<T>` is an interface implemented by the test class. The fixture type `T` is instantiated once when the test class is first used and disposed when the test class's tests complete. The fixture instance is injected into the test class constructor. `IClassFixture<T>` can decorate the test class or the collection definition class. It is suitable for expensive resources that are needed by all tests in one class but not by other classes.

**Beginner-Friendly Explanation**
`IClassFixture<T>` means "create this fixture once and share it with every test in this class."

---

### Purposes

- To share setup across all tests in one class.
- To avoid recreating expensive resources per test.
- To provide a clean, typed way to inject shared dependencies.
- To support per-class lifecycle management.
- To reduce test execution time for class-scoped resources.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture() { /* create database */ }
    public void Dispose() { /* dispose database */ }
}

public class RepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public RepositoryTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public void Query_ReturnsData() { ... }
}
```

**Component breakdown**

- `IClassFixture<DatabaseFixture>` — Implemented by the test class.
- Constructor injection — The fixture is passed to the test class constructor.
- `IDisposable` — Optional; for cleanup.

#### Syntax Rules

- The fixture class must be public and have a parameterless constructor (or a constructor that xUnit can resolve).
- The test class must implement `IClassFixture<T>`.
- The fixture is created once per test class and disposed after all tests in the class run.
- The fixture instance is shared across all test methods in the class.

#### Constraints and Limitations

- The fixture is not shared across different test classes; use `ICollectionFixture<T>` for that.
- The fixture is not thread-safe by default; ensure safety if tests run in parallel.
- The fixture cannot access test-specific state.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: IClassFixture with DatabaseFixture

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
- The fixture is disposed after all tests in the class complete.

---

### Real-World Cases

- **Database repository tests** — Shared connection per test class.
- **API controller tests** — Shared `WebApplicationFactory` per class.
- **Configuration tests** — Shared configuration object per class.

---

### References

- Sharing Context between Tests — xUnit.net - https://xunit.net/docs/shared-context
- xUnit1041 — Fixture sources - https://xunit.net/xunit.analyzers/rules/xUnit1041

---

## 9. `ICollectionFixture<T>`

### Definitions

**Core Definition**
`ICollectionFixture<T>` shares a single fixture instance across all test classes in a test collection.

**Technical Definition**
`ICollectionFixture<T>` is declared on a collection definition class marked with `[CollectionDefinition("name")]`. Test classes that belong to the collection are marked with `[Collection("name")]`. The fixture is created once when the collection is first used and disposed when the collection's tests complete. All test classes in the collection share the same fixture instance, and tests within the collection run sequentially (not in parallel) to avoid conflicts.

**Beginner-Friendly Explanation**
`ICollectionFixture<T>` means "create this fixture once and share it with every test class in this collection."

---

### Purposes

- To share setup across multiple test classes.
- To avoid recreating expensive resources per class.
- To group related tests that share dependencies.
- To control parallelization (tests in a collection run sequentially).
- To centralize lifecycle management for shared resources.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
[CollectionDefinition("Database")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("Database")]
public class PatientRepositoryTests
{
    private readonly DatabaseFixture _fixture;

    public PatientRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void GetById_ReturnsPatient() { ... }
}

[Collection("Database")]
public class OrderRepositoryTests
{
    private readonly DatabaseFixture _fixture;

    public OrderRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void GetById_ReturnsOrder() { ... }
}
```

**Component breakdown**

- `[CollectionDefinition("name")]` — Declares the collection and its fixture.
- `ICollectionFixture<T>` — Specifies the fixture type.
- `[Collection("name")]` — Marks test classes as members of the collection.
- The fixture is injected into each test class constructor.

#### Syntax Rules

- The collection definition class must have `[CollectionDefinition("name")]`.
- Test classes must have `[Collection("name")]`.
- The fixture is created once per collection and disposed after all tests in the collection run.
- Tests within a collection run sequentially; different collections run in parallel.

#### Constraints and Limitations

- Collection fixtures cannot be used on the test class directly; they must be on a separate collection definition class (xUnit1059).
- Within a collection, tests run sequentially, reducing parallelism.
- The fixture is shared across classes, so state must be managed carefully.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: ICollectionFixture with DatabaseFixture

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

[CollectionDefinition("Database")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("Database")]
public class PatientRepositoryTests
{
    private readonly DatabaseFixture _fixture;

    public PatientRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void GetById_ReturnsPatient()
    {
        var repo = new PatientRepository(_fixture.Connection);
        Assert.NotNull(repo.GetById(1));
    }
}

[Collection("Database")]
public class OrderRepositoryTests
{
    private readonly DatabaseFixture _fixture;

    public OrderRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public void GetById_ReturnsOrder()
    {
        var repo = new OrderRepository(_fixture.Connection);
        Assert.NotNull(repo.GetById(1));
    }
}
```

**Expected Output**

```
Passed: GetById_ReturnsPatient
Passed: GetById_ReturnsOrder
```

**Why this output occurs**

- The `DatabaseFixture` is created once for the entire collection.
- Both test classes share the same connection.
- Tests in the collection run sequentially.

---

### Real-World Cases

- **Integration tests** — Shared database across multiple repository test classes.
- **API tests** — Shared `WebApplicationFactory` across controller test classes.
- **Configuration tests** — Shared configuration across related test classes.

---

### References

- Sharing Context between Tests — xUnit.net - https://xunit.net/docs/shared-context
- xUnit1041 — Fixture sources - https://xunit.net/xunit.analyzers/rules/xUnit1041
- xUnit1059 — Collection fixture on test class - https://xunit.net/xunit.analyzers/rules/xUnit1059

---

## 10. xUnit Best Practices

### Definitions

**Core Definition**
xUnit best practices are guidelines for writing tests that are fast, isolated, deterministic, readable, and maintainable within the xUnit framework.

**Technical Definition**
Best practices include the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), the Arrange-Act-Assert structure, descriptive naming (`MethodName_Scenario_ExpectedBehavior`), one logical assertion per test, avoidance of logic in tests, appropriate use of fixtures, and integration into CI pipelines. xUnit-specific practices include using constructors for setup, `IDisposable` for teardown, `[Theory]` for data-driven tests, and `[Collection]` to control parallelization.

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

#### Fixture Usage

- Use constructor/`Dispose` for per-test isolation (default).
- Use `IClassFixture<T>` for shared context within a class.
- Use `ICollectionFixture<T>` for shared context across classes.
- Use collection fixtures to group tests with shared dependencies.

#### Parallelization

- xUnit runs test collections in parallel by default.
- Use `[Collection]` to serialize tests that share resources.
- Ensure fixtures are thread-safe if tests run in parallel.

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

- Tests are isolated; fresh substitutes per test via constructor.
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
- xUnit shared context documentation - https://xunit.net/docs/shared-context
- xUnit analyzers rules - https://xunit.net/xunit.analyzers/rules/
- xUnit Test Patterns (naming, AAA) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/19-c-sharp-best-prac-sharptic-sharpes/7-c-sharp-testing-strategies.mdx
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `[SetUp]` / `[TearDown]` | Not available in xUnit; use constructor and `IDisposable` |
| `async void` test methods | Unsafe; use `async Task` |
| `ICollectionFixture` on test class | Invalid; must be on collection definition class (xUnit1059) |
| `[InlineData]` with `null` | Use nullable types; behavior varies by version |
| `MemberData`/`ClassData` argument count mismatch | Analyzers xUnit1037 and xUnit1038 catch this at compile time |
| Non-serializable theory data | xUnit1045 requires serializable data for test discovery |
| xUnit v3 `TheoryDataRow` | v3-only; use `TheoryData<T>` for v2/v3 compatibility |
| Assembly fixtures | xUnit v3 only; `AssemblyFixtureAttribute` |
| Parallel execution | On by default; tests sharing resources must use `[Collection]` |
| `EnsureCreated` with migrations | Incompatible; choose one approach |

---

## References (Consolidated)

- xUnit.net official documentation - https://xunit.net/
- Getting Started with xUnit.net v3 - https://xunit.net/docs/getting-started/v3/getting-started
- Sharing Context between Tests — xUnit.net - https://xunit.net/docs/shared-context
- xUnit analyzers rules - https://xunit.net/xunit.analyzers/rules/
- xUnit1003 — Theory methods must have test data - https://xunit.net/xunit.analyzers/rules/xUnit1003
- xUnit1037 — Fewer arguments than parameters - https://xunit.net/xunit.analyzers/rules/xUnit1037
- xUnit1038 — More arguments than parameters - https://xunit.net/xunit.analyzers/rules/xUnit1038
- xUnit1041 — Fixture sources - https://xunit.net/xunit.analyzers/rules/xUnit1041
- xUnit1045 — Serializable data - https://xunit.net/xunit.analyzers/rules/xUnit1045
- xUnit1059 — Collection fixture on test class - https://xunit.net/xunit.analyzers/rules/xUnit1059
- Unit testing best practices — .NET - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- xUnit Test Patterns (naming, AAA) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/19-c-sharp-best-prac-sharptic-sharpes/7-c-sharp-testing-strategies.mdx
- FIRST principles - https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- xUnit Theory Data examples - https://andrewlock.net/creating-parameterised-tests-in-xunit-with-inlinedata-classdata-and-memberdata/