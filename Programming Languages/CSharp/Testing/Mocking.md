# Mocking in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Mocking is the practice of replacing real dependencies with controlled substitute objects during testing so that the system under test can be exercised in isolation, and interactions with those dependencies can be verified.

**Technical Definition**
Mocking is a form of test double usage in which a substitute object is created to stand in for a real collaborator. Mocking frameworks — Moq, NSubstitute, and FakeItEasy — generate these substitutes at runtime (typically via dynamic proxies for interfaces and virtual members). The substitute can be configured to return specific values, throw exceptions, execute callbacks, and record invocations. After the system under test (SUT) executes, the test can verify that the expected interactions occurred with the expected arguments. Mocking is distinguished from stubbing (which supplies data without verification) and faking (which provides a working simplified implementation).

**Beginner-Friendly Explanation**
Mocking is “using a stunt double for your dependencies.” If your code needs a database, you do not connect to a real database in a unit test — you create a fake database object that behaves the way you want, and you check that your code talked to it correctly. Mocking frameworks make creating these stunt doubles easy.

---

### Key Characteristics

- **Isolation** — The SUT is tested without real databases, networks, file systems, or clocks.
- **Interaction verification** — Tests can assert that specific methods were called with specific arguments.
- **Behavior configuration** — Substitutes can return values, throw exceptions, and invoke callbacks.
- **Determinism** — Tests produce the same result every run because dependencies are controlled.
- **Speed** — No I/O means tests run in milliseconds.
- **Design pressure** — Mocking encourages dependency injection and interface-based design.
- **Framework-driven** — Moq, NSubstitute, and FakeItEasy provide the APIs; the concepts are shared.
- **Risk of over-mocking** — Excessive mocking couples tests to implementation details and can hide real defects.

---

### Prerequisites

- Working knowledge of C# interfaces, classes, virtual methods, and delegates/lambdas.
- Understanding of dependency injection and the Dependency Inversion Principle.
- Familiarity with a unit testing framework (xUnit, NUnit, or MSTest).
- Basic understanding of the Arrange-Act-Assert pattern.
- Awareness of the distinction between stubs, mocks, spies, and fakes.

---

### Related Programming Areas

- **Unit testing** — Mocking is most commonly used in unit tests.
- **Test-driven development (TDD)** — Mocking supports test-first design of collaborators.
- **Behavior-driven development (BDD)** — Verification aligns with behavior specifications.
- **Integration testing** — Mocking is used less; real dependencies are preferred.
- **Dependency injection** — Mocking depends on seams created by DI.
- **SOLID principles** — Interface segregation and dependency inversion make mocking natural.

---

### Core Concepts / Features

1. Mocking (overarching discipline)
2. Moq
3. NSubstitute
4. FakeItEasy
5. Mock objects
6. Mock methods
7. Mock expectations
8. Mocking best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with “To”), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Mocking (Overarching Discipline)

### Definitions

**Core Definition**
Mocking is the creation and use of substitute objects that stand in for real dependencies during testing, enabling isolation and interaction verification.

**Technical Definition**
Mocking is a technique within the test-double family. It involves creating a substitute for a dependency, configuring its behavior (optional), injecting it into the SUT, executing the SUT, and verifying interactions. Mocking frameworks generate substitutes for interfaces and virtual/abstract members; they cannot mock sealed classes, static members, or non-virtual members without additional tooling (e.g., JustMock, TypeMock). The three dominant .NET frameworks are Moq (Setup/Verify), NSubstitute (Substitute.For / Received), and FakeItEasy (A.Fake / A.CallTo / MustHaveHappened) .

**Beginner-Friendly Explanation**
Mocking is “faking a dependency so you can test your code alone.” You create a fake object, tell it how to behave, give it to your code, and then check that your code used it correctly.

---

### Purposes

- To isolate the SUT from external dependencies.
- To make tests fast and deterministic.
- To verify that the SUT interacts with its collaborators correctly.
- To simulate error conditions and edge cases.
- To enable testing of code that depends on non-deterministic systems (time, randomness, networks).
- To support test-driven development by allowing collaborators to be designed from the consumer’s perspective.

---

### Syntax Rules and Structure

#### General Mocking Workflow

```
1. Create a substitute for the dependency.
2. Configure behavior (optional): return values, exceptions, callbacks.
3. Inject the substitute into the SUT.
4. Execute the SUT (Act).
5. Verify interactions (optional): call counts, arguments.
```

#### Syntax Rules

- Mock interfaces or virtual/abstract members; sealed classes require special tooling.
- Prefer constructor injection so substitutes can be supplied.
- Configure only what the test needs; avoid over-specification.
- Verify only interactions that are part of the SUT’s contract.
- Use argument matchers deliberately; avoid `It.IsAny` when a specific value is meaningful .

#### Constraints and Limitations

- Cannot mock sealed classes, static members, or non-virtual members with free frameworks.
- Over-mocking couples tests to implementation details.
- Strict mocks can make tests brittle.
- Mocking does not replace integration testing; critical flows need real-dependency tests .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Mocking an Interface with Moq

```csharp
using Moq;
using Xunit;

public interface IEmailSender
{
    void Send(string to, string subject, string body);
}

public class OrderService
{
    private readonly IEmailSender _email;
    public OrderService(IEmailSender email) => _email = email;

    public void PlaceOrder(string customerEmail)
    {
        // business logic
        _email.Send(customerEmail, "Order confirmation", "Your order is placed.");
    }
}

public class OrderServiceTests
{
    [Fact]
    public void PlaceOrder_SendsConfirmationEmail()
    {
        // Arrange
        var mockEmail = new Mock<IEmailSender>();
        var service = new OrderService(mockEmail.Object);

        // Act
        service.PlaceOrder("alice@example.com");

        // Assert
        mockEmail.Verify(
            e => e.Send("alice@example.com", "Order confirmation", It.IsAny<string>()),
            Times.Once);
    }
}
```

**Line-by-line comments**

- `IEmailSender` — The dependency interface.
- `Mock<IEmailSender>` — Creates a mock substitute.
- `mockEmail.Object` — The substitute instance injected into the SUT.
- `Verify` — Asserts that `Send` was called once with the expected recipient and subject.

**Expected Output**

```
Passed: PlaceOrder_SendsConfirmationEmail
```

**Why this output occurs**

- The mock records the call to `Send`.
- `Verify` checks that the interaction occurred exactly once with the specified arguments.

---

### Real-World Cases

- **Service layer tests** — Mock repositories, email senders, and loggers.
- **API handler tests** — Mock command/query handlers and external clients.
- **Domain tests** — Mock time providers and random number generators.
- **Legacy code** — Introduce seams through interfaces and mock them.

---

### References

- Moq documentation (GitHub) - https://github.com/devlooped/moq
- NSubstitute documentation - https://nsubstitute.github.io/
- FakeItEasy documentation - https://fakeiteasy.github.io/
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology

---

## 2. Moq

### Definitions

**Core Definition**
Moq is a popular mocking framework for .NET that uses a `Mock<T>` container and lambda-based `Setup`/`Verify` syntax to create and configure substitutes.

**Technical Definition**
Moq is the most widely used .NET mocking library by download count . It creates mocks via `new Mock<T>()` and exposes the substitute through `.Object`. Behavior is configured with `.Setup(x => x.Method(...)).Returns(...)`; verification uses `.Verify(x => x.Method(...), Times.Once)`. Moq supports argument matchers (`It.IsAny<T>`, `It.Is<T>(predicate)`), async returns (`ReturnsAsync`), sequences (`SetupSequence`), callbacks, protected members, and strict/loose behavior. Moq is open-source under the BSD-3 license; the 2023 SponsorLink controversy prompted some teams to evaluate alternatives, but the feature was reverted .

**Beginner-Friendly Explanation**
Moq is “the classic mocking framework.” You create a `Mock<IService>`, tell it what to return with `Setup`, and check calls with `Verify`. It has a large community and abundant examples.

---

### Purposes

- To provide a mature, feature-rich mocking framework.
- To support lambda-based setup and verification.
- To offer argument matchers and predicate-based matching.
- To handle async methods, properties, and events.
- To support strict and loose mock behavior.
- To integrate with all major .NET test frameworks.

---

### Syntax Rules and Structure

#### General Syntax

```
// Create
var mock = new Mock<IService>();

// Setup return value
mock.Setup(x => x.GetValue()).Returns(42);

// Setup with argument matcher
mock.Setup(x => x.GetById(It.IsAny<int>()))
    .Returns<int>(id => new Entity { Id = id });

// Async
mock.Setup(x => x.GetAsync(1)).ReturnsAsync(entity);

// Throw
mock.Setup(x => x.Save(null)).Throws<ArgumentNullException>();

// Callback
Entity captured = null;
mock.Setup(x => x.Save(It.IsAny<Entity>()))
    .Callback<Entity>(e => captured = e);

// Verify
mock.Verify(x => x.Save(It.IsAny<Entity>()), Times.Once);

// Strict / loose
var strict = new Mock<IService>(MockBehavior.Strict);
var loose = new Mock<IService>(MockBehavior.Loose); // default
```

**Component breakdown**

- `Mock<T>` — Container that generates and holds the substitute.
- `.Object` — The substitute instance.
- `.Setup(...)` — Configures behavior.
- `.Returns(...)` / `.Throws(...)` — Specifies outcomes.
- `.Callback(...)` — Captures arguments or executes side effects.
- `.Verify(...)` — Asserts interaction.
- `Times.Once` / `Times.Exactly(n)` / `Times.Never` — Call-count expectations.
- `MockBehavior.Strict` — Unconfigured calls throw; `Loose` — unconfigured calls return default values .

#### Syntax Rules

- The mocked type must be an interface or a class with virtual/abstract members.
- `Setup` expressions must use argument matchers or specific values.
- `Returns` must match the method’s return type.
- `Verify` must be called after the Act phase.
- `It.IsAny<T>()` should be used sparingly; prefer specific values when meaningful.

#### Constraints and Limitations

- Cannot mock sealed classes, static members, or non-virtual members.
- Strict mocks make tests brittle and should be used selectively .
- `It.IsAny` in `Verify` without a corresponding `Setup` can behave unexpectedly .
- Moq’s `VerifyAll()` verifies all setups, which can over-constrain tests.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Setup, Returns, and Verify

```csharp
var mockRepo = new Mock<IRepository>();
mockRepo.Setup(r => r.GetById(1)).Returns(new Order { Id = 1, Total = 99.99m });

var service = new OrderService(mockRepo.Object);
var order = service.GetOrder(1);

Assert.Equal(99.99m, order.Total);
mockRepo.Verify(r => r.GetById(1), Times.Once);
```

**Expected Output**

```
Passed: GetOrder_ReturnsOrderFromRepository
```

**Why this output occurs**

- `Setup` configures `GetById(1)` to return the order.
- `Verify` confirms the method was called once with `1`.

---

#### Example 2: Async with ReturnsAsync and Callback

```csharp
var mockClient = new Mock<IApiClient>();
string capturedUrl = null;

mockClient.Setup(c => c.GetAsync(It.IsAny<string>()))
    .Callback<string>(url => capturedUrl = url)
    .ReturnsAsync(new HttpResponseMessage(HttpStatusCode.OK));

var service = new ApiService(mockClient.Object);
await service.FetchAsync("/users");

Assert.Equal("/users", capturedUrl);
mockClient.Verify(c => c.GetAsync("/users"), Times.Once);
```

**Expected Output**

```
Passed: FetchAsync_CallsApiWithUrl
```

**Why this output occurs**

- `Callback` captures the URL argument.
- `ReturnsAsync` supplies a successful HTTP response.
- `Verify` confirms the call.

---

### Real-World Cases

- **Repository tests** — Mock data access interfaces.
- **HTTP client tests** — Mock `HttpClient` handlers or API client interfaces.
- **Logger tests** — Verify logging calls.
- **Command handler tests** — Verify that commands were dispatched.

---

### References

- Moq GitHub repository - https://github.com/devlooped/moq
- Common Moq Patterns (community) - https://github.com/dezverev/cursorSkillTemplate/blob/main/.cursor/skills/write-unit-tests/moq-patterns.md
- Moq strict vs loose behavior - https://github.com/keghub/unit-testing-csharp/blob/master/docs/moq/mock-customization.md

---

## 3. NSubstitute

### Definitions

**Core Definition**
NSubstitute is a mocking framework for .NET that emphasizes readability through a “substitute” metaphor: the substitute object itself is both the fake and the configuration target.

**Technical Definition**
NSubstitute creates substitutes with `Substitute.For<T>()`. Behavior is configured by calling methods directly on the substitute and chaining `.Returns(...)`. Verification uses `.Received()` and `.DidNotReceive()` with argument matchers (`Arg.Any<T>()`, `Arg.Is<T>(predicate)`). NSubstitute is lenient by default — unconfigured calls return default values and do not throw. It does not support strict mocks, a deliberate design choice aligned with AAA-style testing . NSubstitute is open-source under the BSD-3 license.

**Beginner-Friendly Explanation**
NSubstitute is “the readable mocking framework.” You call `Substitute.For<IService>()`, and the returned object is your fake. You say `service.Get(1).Returns(value)` and `service.Received().Get(1)` — it reads almost like English.

---

### Purposes

- To provide a readable, natural-language mocking API.
- To eliminate the mock container/wrapper abstraction.
- To support argument matchers and predicate matching.
- To handle async methods and events.
- To integrate with xUnit, NUnit, and MSTest.
- To offer a lenient-by-default philosophy that reduces brittleness.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Create
var calculator = Substitute.For<ICalculator>();

// Configure return
calculator.Add(1, 2).Returns(3);

// Configure with matcher
calculator.Get(Arg.Any<int>()).Returns(42);

// Async
service.GetAsync(1).Returns(Task.FromResult(data));

// Verify received
calculator.Received().Add(1, Arg.Any<int>());
calculator.Received(1).Add(1, 2);
calculator.DidNotReceive().Add(2, 2);

// Clear received calls
calculator.ClearReceivedCalls();

// Raise event
calculator.PoweringUp += Raise.Event();
```

**Component breakdown**

- `Substitute.For<T>()` — Creates the substitute.
- `.Returns(...)` — Configures the return value.
- `.Received()` / `.DidNotReceive()` — Asserts interaction.
- `Arg.Any<T>()` / `Arg.Is<T>(predicate)` — Argument matchers.
- `ClearReceivedCalls()` — Resets recorded invocations without clearing setups .

#### Syntax Rules

- The mocked type must be an interface or a class with virtual/overridable members .
- `.Returns(...)` must be chained to a method call expression.
- `Received()` must be followed by the method call to verify.
- Argument matchers can be used in both `Returns` and `Received` contexts.

#### Constraints and Limitations

- No strict mocks; unconfigured calls succeed silently .
- Cannot mock non-virtual members of concrete classes.
- `Received()` without a count checks “at least once”; use `Received(n)` for exact counts.
- Overuse of `Arg.Any` reduces test specificity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Substitute, Returns, and Received

```csharp
var calculator = Substitute.For<ICalculator>();
calculator.Add(1, 2).Returns(3);

Assert.Equal(3, calculator.Add(1, 2));

calculator.Received().Add(1, Arg.Any<int>());
calculator.DidNotReceive().Add(2, 2);
```

**Expected Output**

```
Passed: Add_ReturnsConfiguredValue
Passed: Add_ReceivedExpectedCall
```

**Why this output occurs**

- `.Returns(3)` configures the substitute.
- `Received()` verifies the call happened; `DidNotReceive()` verifies it did not.

---

#### Example 2: Async and Event Raising

```csharp
var service = Substitute.For<IUserService>();
service.GetAsync(1).Returns(Task.FromResult(new User { Id = 1, Name = "Alice" }));

var user = await service.GetAsync(1);
Assert.Equal("Alice", user.Name);

await service.Received(1).GetAsync(1);

bool poweredUp = false;
service.PoweredUp += () => poweredUp = true;
service.PoweredUp += Raise.Event();
Assert.True(poweredUp);
```

**Expected Output**

```
Passed: GetAsync_ReturnsUser
Passed: PoweredUp_EventRaised
```

**Why this output occurs**

- `.Returns(Task.FromResult(...))` configures the async method.
- `Raise.Event()` triggers the event on the substitute.

---

### Real-World Cases

- **Service layer tests** — Mock repositories and API clients.
- **Async code** — Configure `Task`-returning methods.
- **Event-driven code** — Raise events to test handlers.
- **Legacy modernization** — Introduce interfaces and substitute them.

---

### References

- NSubstitute official documentation - https://nsubstitute.github.io/
- NSubstitute argument matchers - https://nsubstitute.github.io/help/argument-matchers/
- NSubstitute clearing received calls - https://nsubstitute.github.io/help/clear-received-calls/
- Moq to NSubstitute migration cheat sheet - https://github.com/thomhurst/NSubstitute.MoqToNSubstitute

---

## 4. FakeItEasy

### Definitions

**Core Definition**
FakeItEasy is a mocking framework for .NET that emphasizes natural-language syntax through a static `A` entry point, using `A.Fake<T>()`, `A.CallTo(...)`, and `MustHaveHappened()` for creation, configuration, and verification.

**Technical Definition**
FakeItEasy creates fakes with `A.Fake<T>()`. Behavior is configured with `A.CallTo(() => fake.Method(...)).Returns(...)`. Verification uses the same `A.CallTo(...).MustHaveHappened()` syntax, so there is one shape for both setup and assertion . FakeItEasy supports argument constraints (`A<T>.Ignored`, `A<T>._`, `That.Matches(...)`), ordered assertions (`Then`), strict fakes, and async methods. It is open-source under the MIT license with no licensing controversies .

**Beginner-Friendly Explanation**
FakeItEasy is “the fluent, English-like mocking framework.” You say `A.Fake<IShop>()`, configure with `A.CallTo(() => shop.GetCandy()).Returns(lollipop)`, and verify with `A.CallTo(() => shop.BuyCandy(lollipop)).MustHaveHappened()`. The same syntax is used for setup and checking.

---

### Purposes

- To provide a fluent, natural-language mocking API.
- To unify setup and verification syntax.
- To support strict fakes and ordered assertions.
- To offer readable argument constraints.
- To integrate with all major test frameworks.
- To remain fully open-source under a permissive license.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Create
var shop = A.Fake<ICandyShop>();

// Configure return
A.CallTo(() => shop.GetTopSellingCandy()).Returns(lollipop);

// Argument constraints
A.CallTo(() => shop.BuyCandy(A<ICandy>.Ignored)).MustHaveHappened();
A.CallTo(() => shop.BuyCandy(A<ICandy>._)).MustHaveHappened();
A.CallTo(() => shop.BuyCandy(A<ICandy>.That.Matches(c => c.IsSweet))).MustHaveHappened();

// Verify
A.CallTo(() => shop.BuyCandy(lollipop)).MustHaveHappened();

// Ordered assertions
A.CallTo(() => unitOfWorkFactory.BeginWork()).MustHaveHappened()
    .Then(A.CallTo(() => unitOfWork.Commit()).MustHaveHappened());

// Strict fake
var strictShop = A.Fake<ICandyShop>(options => options.Strict());
```

**Component breakdown**

- `A.Fake<T>()` — Creates the fake.
- `A.CallTo(() => ...)` — Specifies the call.
- `.Returns(...)` — Configures the return value.
- `.MustHaveHappened()` — Verifies the call occurred.
- `A<T>.Ignored` / `A<T>._` — Ignores an argument.
- `That.Matches(...)` — Predicate-based constraint.
- `.Then(...)` — Ordered assertion.

#### Syntax Rules

- Argument constraints must appear inside the `A.CallTo` lambda, not stored in variables .
- `MustHaveHappened()` with no arguments checks “at least once”; use `MustHaveHappenedOnceExactly()` for exact counts.
- Strict fakes throw on unconfigured calls.
- The same `A.CallTo` shape is used for setup and verification.

#### Constraints and Limitations

- Cannot mock sealed classes or static members without additional tooling.
- Strict fakes can be brittle; use selectively.
- `A<T>.Ignored` and `That` must be inside `A.CallTo`; they cannot be assigned to variables .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Fake, Configure, Verify

```csharp
var lollipop = A.Fake<ICandy>();
var shop = A.Fake<ICandyShop>();

A.CallTo(() => shop.GetTopSellingCandy()).Returns(lollipop);

var developer = new SweetTooth();
developer.BuyTastiestCandy(shop);

A.CallTo(() => shop.BuyCandy(lollipop)).MustHaveHappened();
```

**Expected Output**

```
Passed: BuyTastiestCandy_should_buy_top_selling_candy_from_shop
```

**Why this output occurs**

- `GetTopSellingCandy` returns the fake lollipop.
- `BuyCandy` is called with that lollipop.
- `MustHaveHappened` verifies the interaction.

---

#### Example 2: Argument Constraints and Ordered Assertions

```csharp
var foo = A.Fake<IFoo>();

foo.Bar("hello", 7);

A.CallTo(() => foo.Bar("hello", A<int>.Ignored)).MustHaveHappened();
A.CallTo(() => foo.Bar("hello", 7)).MustHaveHappened();

// Ordered
A.CallTo(() => foo.Bar("first", 1)).MustHaveHappened()
    .Then(A.CallTo(() => foo.Bar("second", 2)).MustHaveHappened());
```

**Expected Output**

```
Passed: Bar_IgnoredArgument_Matches
Passed: Bar_OrderedCalls_Verified
```

**Why this output occurs**

- `A<int>.Ignored` matches any integer for the second argument.
- `.Then(...)` verifies the order of calls.

---

### Real-World Cases

- **Fluent test suites** — Readable setup and verification.
- **Verification-heavy scenarios** — Strict fakes and ordered assertions.
- **Teams valuing natural language** — BDD-style readability.
- **Open-source projects** — MIT license with no restrictions.

---

### References

- FakeItEasy official documentation - https://fakeiteasy.github.io/
- FakeItEasy quickstart - https://fakeiteasy.github.io/docs/8.0.1/quickstart/
- FakeItEasy argument constraints - https://fakeiteasy.github.io/docs/8.0.1/argument-constraints/
- FakeItEasy ordered assertions - https://fakeiteasy.github.io/docs/8.0.1/ordered-assertions/

---

## 5. Mock Objects

### Definitions

**Core Definition**
A mock object is a substitute object that stands in for a real collaborator and records interactions so that the test can verify the SUT’s behavior.

**Technical Definition**
In the test-double taxonomy, mock objects are pre-programmed with expectations and are used to verify that the SUT calls them in the expected way. They differ from stubs (which supply data but are not verified) and fakes (which have working implementations). Mocking frameworks blur these categories — Moq and NSubstitute can act as both stubs and mocks depending on whether verification is performed. The key distinction is intent: a mock is used when the interaction itself is the behavior under test .

**Beginner-Friendly Explanation**
A mock object is “a fake collaborator that remembers what happened.” You give it to your code, your code uses it, and then you ask the mock, “Did they call you? With what arguments?” The mock tells you.

---

### Purposes

- To verify that the SUT communicates correctly with collaborators.
- To isolate the SUT from real dependencies.
- To simulate complex or unavailable services.
- To test command-style interactions (side effects).
- To enforce contracts between components.

---

### Syntax Rules and Structure

#### Mock Object Lifecycle

```
1. Create the mock.
2. Configure expected behavior (optional).
3. Inject into the SUT.
4. Execute the SUT.
5. Verify interactions.
```

#### Syntax Rules (Framework-Agnostic)

- Mocks should be created per test (or per test class for shared fixtures).
- Verification should focus on interactions that matter to the SUT’s contract.
- Mocks should not be used to verify trivial or incidental calls.

#### Constraints and Limitations

- Mocks test interactions, not state; prefer state-based tests when possible.
- Over-mocking couples tests to implementation details.
- Mocks cannot verify real integration behavior.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Mock Object in Context

```csharp
public interface IPaymentGateway
{
    bool Charge(string cardNumber, decimal amount);
}

public class CheckoutService
{
    private readonly IPaymentGateway _gateway;
    public CheckoutService(IPaymentGateway gateway) => _gateway = gateway;

    public bool Checkout(string card, decimal amount)
    {
        if (amount <= 0) throw new ArgumentException();
        return _gateway.Charge(card, amount);
    }
}

[Fact]
public void Checkout_ValidAmount_ChargesGateway()
{
    var gateway = Substitute.For<IPaymentGateway>();
    gateway.Charge(Arg.Any<string>(), Arg.Any<decimal>()).Returns(true);

    var service = new CheckoutService(gateway);
    service.Checkout("4111", 99.99m);

    gateway.Received(1).Charge("4111", 99.99m);
}
```

**Expected Output**

```
Passed: Checkout_ValidAmount_ChargesGateway
```

**Why this output occurs**

- The mock gateway records the charge.
- `Received(1)` verifies the exact interaction.

---

### Real-World Cases

- **Payment processing** — Verify charges, refunds, and void calls.
- **Email and notifications** — Verify that messages were sent.
- **Audit logging** — Verify that audit entries were created.
- **Message publishing** — Verify that events were published.

---

### References

- Mock object pattern — Martin Fowler - https://martinfowler.com/articles/mocksArentStubs.html
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology

---

## 6. Mock Methods

### Definitions

**Core Definition**
Mock methods are the individual method calls on a substitute that are configured, captured, or verified during a test.

**Technical Definition**
Mock methods are the expressions passed to setup and verification APIs (`Setup(x => x.Method(...))` in Moq, `substitute.Method(...).Returns(...)` in NSubstitute, `A.CallTo(() => fake.Method(...))` in FakeItEasy). Each framework provides mechanisms to configure return values, throw exceptions, execute callbacks, and verify call counts for specific method signatures. Argument matchers allow the test to match methods by pattern rather than exact values.

**Beginner-Friendly Explanation**
Mock methods are “the methods you tell the fake to watch.” You say “when someone calls `Save`, return `true`” or “check that `Send` was called once.”

---

### Purposes

- To configure the behavior of individual dependency methods.
- To verify that specific methods were called.
- To capture arguments for later assertions.
- To simulate exceptions and edge cases.
- To test async methods and properties.

---

### Syntax Rules and Structure

#### Moq

```csharp
mock.Setup(x => x.GetValue()).Returns(42);
mock.Setup(x => x.GetById(It.IsAny<int>())).Returns<int>(id => new Entity { Id = id });
mock.Setup(x => x.SaveAsync(It.IsAny<Entity>())).ThrowsAsync(new ValidationException());
mock.Verify(x => x.Save(It.IsAny<Entity>()), Times.Once);
```

#### NSubstitute

```csharp
sub.GetValue().Returns(42);
sub.GetById(Arg.Any<int>()).Returns(42);
sub.SaveAsync(Arg.Any<Entity>()).Returns(Task.FromException(new ValidationException()));
sub.Received(1).Save(Arg.Any<Entity>());
```

#### FakeItEasy

```csharp
A.CallTo(() => fake.GetValue()).Returns(42);
A.CallTo(() => fake.GetById(A<int>.Ignored)).Returns(42);
A.CallTo(() => fake.SaveAsync(A<Entity>.Ignored)).ThrowsAsync(new ValidationException());
A.CallTo(() => fake.Save(A<Entity>.Ignored)).MustHaveHappenedOnceExactly();
```

**Component breakdown**

- Setup expression — Identifies the method and arguments.
- `.Returns(...)` / `.Throws(...)` — Specifies behavior.
- Verification expression — Identifies the method and arguments to check.
- Count assertion — `Times.Once`, `Received(1)`, `MustHaveHappenedOnceExactly()`.

#### Syntax Rules

- The method expression must match the method signature.
- Async methods require framework-specific async APIs (`ReturnsAsync`, `ThrowsAsync`, `Task.FromResult`).
- Properties are configured with property-specific APIs (`SetupGet`, `.Property.Returns()`, `A.CallTo`).
- Verification must occur after the Act phase.

#### Constraints and Limitations

- Non-virtual methods on concrete classes cannot be mocked.
- Extension methods cannot be mocked directly (they are static).
- Overloaded methods may require explicit type arguments.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Configuring and Verifying a Method Across Frameworks

```csharp
// Moq
var moq = new Mock<IRepo>();
moq.Setup(r => r.Save(It.IsAny<Entity>())).Returns(true);
moq.Verify(r => r.Save(It.IsAny<Entity>()), Times.Once);

// NSubstitute
var sub = Substitute.For<IRepo>();
sub.Save(Arg.Any<Entity>()).Returns(true);
sub.Received(1).Save(Arg.Any<Entity>());

// FakeItEasy
var fake = A.Fake<IRepo>();
A.CallTo(() => fake.Save(A<Entity>.Ignored)).Returns(true);
A.CallTo(() => fake.Save(A<Entity>.Ignored)).MustHaveHappenedOnceExactly();
```

**Expected Output**

```
All three frameworks pass the equivalent verification.
```

**Why this output occurs**

- Each framework records the call to `Save`.
- Each verification confirms one invocation with any `Entity` argument.

---

### Real-World Cases

- **Repository save** — Verify that `Save` was called with the expected entity.
- **Async API calls** — Configure `GetAsync` to return a task.
- **Exception paths** — Configure a method to throw and assert the SUT handles it.
- **Property access** — Verify that a property getter was called.

---

### References

- Moq common patterns (methods, async, callbacks) - https://github.com/dezverev/cursorSkillTemplate/blob/main/.cursor/skills/write-unit-tests/moq-patterns.md
- NSubstitute argument matchers - https://nsubstitute.github.io/help/argument-matchers/
- FakeItEasy argument constraints - https://fakeiteasy.github.io/docs/8.0.1/argument-constraints/

---

## 7. Mock Expectations

### Definitions

**Core Definition**
A mock expectation is an assertion that a specific interaction with a mock object did (or did not) occur during the test, optionally with a specific call count and argument pattern.

**Technical Definition**
Mock expectations are expressed through verification APIs: Moq’s `Verify` with `Times`, NSubstitute’s `Received`/`DidNotReceive`, and FakeItEasy’s `MustHaveHappened`. Expectations can be exact (`Times.Once`, `Received(1)`, `MustHaveHappenedOnceExactly()`), range-based (`Times.Between`, `Received(1, 3)`), or negative (`Times.Never`, `DidNotReceive`, `MustNotHaveHappened`). Argument matchers refine expectations to specific argument patterns. Strict mocks turn every unconfigured call into a failure, effectively making all interactions expectations .

**Beginner-Friendly Explanation**
A mock expectation is “checking that the fake was used the right way.” You say “this method should have been called exactly once with these arguments” or “this method should never have been called.”

---

### Purposes

- To verify that the SUT interacts with collaborators as specified.
- To catch missing or extra calls.
- To enforce call counts and ordering.
- To ensure arguments are passed correctly.
- To detect regressions in interaction contracts.

---

### Syntax Rules and Structure

#### Call Counts

| Intent | Moq | NSubstitute | FakeItEasy |
|---|---|---|---|
| Exactly once | `Times.Once` | `Received(1)` | `MustHaveHappenedOnceExactly()` |
| Never | `Times.Never` | `DidNotReceive()` | `MustNotHaveHappened()` |
| At least once | `Times.AtLeastOnce` | `Received()` | `MustHaveHappened()` |
| Exactly n | `Times.Exactly(n)` | `Received(n)` | `MustHaveHappened(n, n, false)` |
| Between | `Times.Between(min, max, Range.Inclusive)` | `Received(min, max)` | `MustHaveHappened(min, max, false)` |

#### Argument Expectations

- Moq: `It.IsAny<T>()`, `It.Is<T>(predicate)`, `It.IsRegex(...)`.
- NSubstitute: `Arg.Any<T>()`, `Arg.Is<T>(predicate)`.
- FakeItEasy: `A<T>.Ignored`, `A<T>._`, `A<T>.That.Matches(predicate)`.

#### Syntax Rules

- Expectations should be placed after the Act phase.
- Use the loosest expectation that still captures the contract (prefer `AtLeastOnce` over `Once` unless exact count matters).
- Avoid `VerifyAll()` / verifying every setup; verify only meaningful interactions.
- Strict mocks should be used sparingly; they make every call an expectation .

#### Constraints and Limitations

- Over-specifying expectations makes tests brittle.
- Verifying implementation details rather than behavior couples tests to internals.
- Expectations on `Arg.Any` / `It.IsAny` can hide argument bugs .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Call Count Expectations

```csharp
// Moq
mock.Verify(x => x.Save(It.IsAny<Entity>()), Times.Once);
mock.Verify(x => x.Log(It.IsAny<string>()), Times.Exactly(3));
mock.Verify(x => x.Delete(It.IsAny<int>()), Times.Never);

// NSubstitute
sub.Received(1).Save(Arg.Any<Entity>());
sub.Received(3).Log(Arg.Any<string>());
sub.DidNotReceive().Delete(Arg.Any<int>());

// FakeItEasy
A.CallTo(() => fake.Save(A<Entity>.Ignored)).MustHaveHappenedOnceExactly();
A.CallTo(() => fake.Log(A<string>.Ignored)).MustHaveHappened(3, 3, false);
A.CallTo(() => fake.Delete(A<int>.Ignored)).MustNotHaveHappened();
```

**Expected Output**

```
All frameworks pass if the call counts match.
```

**Why this output occurs**

- Each framework counts invocations and compares to the expectation.

---

#### Example 2: Argument Expectations

```csharp
// Moq
mock.Verify(x => x.Save(It.Is<Entity>(e => e.Id == 1)));

// NSubstitute
sub.Received().Save(Arg.Is<Entity>(e => e.Id == 1));

// FakeItEasy
A.CallTo(() => fake.Save(A<Entity>.That.Matches(e => e.Id == 1))).MustHaveHappened();
```

**Expected Output**

```
Passed if an entity with Id == 1 was saved.
```

**Why this output occurs**

- Predicate matchers refine the expectation to specific arguments.

---

### Real-World Cases

- **Command handlers** — Verify that exactly one repository save occurred.
- **Event publishing** — Verify that the correct event was published once.
- **Caching** — Verify that the cache was checked before the database.
- **Idempotency** — Verify that an operation was not repeated.

---

### References

- Moq verification (Times, It matchers) - https://github.com/devlooped/moq
- NSubstitute received calls - https://nsubstitute.github.io/help/received-calls/
- FakeItEasy assertion - https://fakeiteasy.github.io/docs/8.0.1/assertion/
- Testing anti-patterns (over-specification) - https://github.com/opensmartroute/testing-anti-patterns

---

## 8. Mocking Best Practices

### Definitions

**Core Definition**
Mocking best practices are guidelines for using mock objects in a way that produces reliable, maintainable, and meaningful tests without over-coupling to implementation details.

**Technical Definition**
Best practices encompass mocking only boundaries (external dependencies), preferring state verification over interaction verification where possible, using the loosest verification that captures the contract, avoiding mocking of third-party types directly, keeping mocks per-test, and ensuring that critical flows are also covered by integration tests. They also include framework-specific guidance: prefer NSubstitute for readability, Moq for feature richness, and FakeItEasy for strict verification and natural-language syntax .

**Beginner-Friendly Explanation**
Best practices are “the habits that keep mocking helpful instead of harmful.” Mock the things you cannot control (databases, APIs, time), verify only what matters, and do not mock everything just because you can.

---

### Purposes

- To keep tests reliable and meaningful.
- To avoid over-coupling to implementation details.
- To ensure that mocks do not hide real integration problems.
- To make tests readable and maintainable.
- To choose the right framework for the team’s style.
- To balance unit isolation with integration confidence.

---

### Guidelines and Structure

#### Mock Boundaries, Not Internals

```
✅ Mock: database repositories, HTTP clients, email senders, clocks, file systems.
❌ Avoid mocking: your own domain classes, value objects, or pure functions.
```

- Mock only at architectural boundaries .
- Use real implementations for pure logic and in-memory fakes for simple state.

#### Prefer State Verification Over Interaction Verification

```csharp
// ✅ Prefer: assert on the result/state
var result = service.Calculate(input);
Assert.Equal(expected, result);

// ⚠️ Use sparingly: verify interactions
mockRepo.Verify(r => r.Save(It.IsAny<Entity>()), Times.Once);
```

- Use interaction verification for commands and side effects.
- Use state verification for queries and calculations.

#### Use the Loosest Expectation That Captures the Contract

```csharp
// ✅ Prefer: at least once
mock.Verify(x => x.Save(It.IsAny<Entity>()), Times.AtLeastOnce);

// ❌ Avoid unless exact count matters
mock.Verify(x => x.Save(It.IsAny<Entity>()), Times.Once);
```

- Exact counts make tests brittle.
- Verify the minimum behavior required by the contract.

#### Avoid Over-Mocking

- Do not mock to “be safe”; it hides real behavior .
- Do not test mock behavior; test the SUT’s behavior.
- Do not add test-only methods to production classes .

#### Keep Mocks Per Test (or Per Class with a Fixture)

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
}
```

- Fresh substitutes per test prevent state leakage.
- Use `IClassFixture<T>` for shared setup only when necessary.

#### Choose the Right Framework for Your Team

| Framework | Best For |
|---|---|
| **Moq** | Feature richness, large community, existing codebases  |
| **NSubstitute** | Readability, natural-language syntax, lenient-by-default  |
| **FakeItEasy** | Strict verification, unified setup/verify syntax, MIT license  |

- Consider team familiarity and codebase consistency.
- Avoid mixing frameworks within the same project.

#### Complement with Integration Tests

- Unit tests with mocks verify isolated logic.
- Integration tests verify real database, HTTP, and messaging behavior.
- One integration test per module validates the real flow .

#### Avoid Mocking Things You Do Not Own

- Mock interfaces you control, not third-party concrete classes.
- Wrap third-party dependencies behind your own interface.
- Use adapter patterns to create seams.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Boundary Mocking with State Verification

```csharp
public class PriceCalculator
{
    private readonly IDiscountProvider _discounts;
    public PriceCalculator(IDiscountProvider discounts) => _discounts = discounts;

    public decimal Calculate(decimal price)
    {
        var rate = _discounts.GetRateFor(price);
        return price * (1 - rate);
    }
}

[Fact]
public void Calculate_AppliesDiscount()
{
    var discounts = Substitute.For<IDiscountProvider>();
    discounts.GetRateFor(100m).Returns(0.1m);

    var calculator = new PriceCalculator(discounts);
    var result = calculator.Calculate(100m);

    Assert.Equal(90m, result);
    // No interaction verification needed — state is asserted
}
```

**Expected Output**

```
Passed: Calculate_AppliesDiscount
```

**Why this output occurs**

- The boundary (discount provider) is mocked.
- The SUT’s output is verified by state, not by verifying the mock call.

---

#### Example 2: Interaction Verification for a Command

```csharp
[Fact]
public void PlaceOrder_ValidOrder_SavesOnce()
{
    var repo = Substitute.For<IOrderRepository>();
    var service = new OrderService(repo);

    service.PlaceOrder(new Order { Total = 100m });

    repo.Received(1).Save(Arg.Is<Order>(o => o.Total == 100m));
}
```

**Expected Output**

```
Passed: PlaceOrder_ValidOrder_SavesOnce
```

**Why this output occurs**

- The command (saving) is verified because the side effect is the behavior under test.
- The argument predicate ensures the correct entity was saved.

---

### Real-World Cases

- **API services** — Mock external HTTP clients, verify repository saves.
- **Background workers** — Mock message queues, verify publish calls.
- **Domain services** — Mock time and randomness, assert on calculated results.
- **Legacy code** — Introduce seams and mock boundaries gradually.

---

### References

- Mocking best practices — Microsoft (Azure SDK testing) - https://learn.microsoft.com/en-us/dotnet/azure/sdk/unit-testing-mocking
- When to use mocking — Typemock - https://www.typemock.com/when-to-use-mocking-in-unit-testing/
- Testing anti-patterns — OpenSmartRoute - https://opensmartroute.ai/testing-anti-patterns/
- Moq vs NSubstitute vs FakeItEasy comparison 2026 - https://www.pistack.xyz/posts/2026-07-04-csharp-mocking-frameworks-moq-nsubstitute-fakeiteasy-justmock/
- Moq vs NSubstitute vs FakeItEasy in .NET 2026 - https://codingdroplets.com/moq-vs-nsubstitute-vs-fakeiteasy-in-net-which-mocking-framework-should-your-team-use-in-2026

---

## Framework Comparison Summary

| Feature | Moq | NSubstitute | FakeItEasy |
|---|---|---|---|
| Creation | `new Mock<T>()` | `Substitute.For<T>()` | `A.Fake<T>()` |
| Setup | `.Setup(x => x.Method()).Returns(...)` | `.Method().Returns(...)` | `A.CallTo(() => x.Method()).Returns(...)` |
| Verification | `.Verify(x => x.Method(), Times.Once)` | `.Received(1).Method()` | `A.CallTo(() => x.Method()).MustHaveHappened()` |
| Negative verification | `Times.Never` | `DidNotReceive()` | `MustNotHaveHappened()` |
| Argument matchers | `It.IsAny<T>()`, `It.Is<T>(pred)` | `Arg.Any<T>()`, `Arg.Is<T>(pred)` | `A<T>.Ignored`, `A<T>.That.Matches(pred)` |
| Async support | `ReturnsAsync()`, `ThrowsAsync()` | `.Returns(Task.FromResult(...))` | `.Returns(...)`, `.ThrowsAsync(...)` |
| Strict mode | `MockBehavior.Strict` | Not supported (lenient only) | `options.Strict()` |
| Clear calls | `mock.Reset()` | `ClearReceivedCalls()` | `Fake.ClearRecordedCalls()` |
| License | BSD-3 | BSD-3 | MIT |
| Best for | Feature richness, large community | Readability, natural syntax | Strict verification, unified syntax |

*Sources: framework documentation and comparison articles* .

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Moq SponsorLink | 2023 controversy; feature reverted, but trust erosion prompted some migrations  |
| Moq `VerifyAll()` | Verifies all setups; can over-constrain tests. |
| NSubstitute strict mocks | Not supported by design; lenient-by-default . |
| FakeItEasy `A<T>.Ignored` outside `A.CallTo` | Invalid; constraints must be inside the lambda . |
| Mocking sealed classes / static members | Not supported by Moq, NSubstitute, or FakeItEasy free versions; requires JustMock or TypeMock . |
| `It.IsAny` in `Verify` without `Setup` | Can behave unexpectedly; configure the setup first . |
| Over-mocking | Hides real integration problems; complement with integration tests . |
| Testing mock behavior | Anti-pattern; test the SUT, not the mock . |

---

## References (Consolidated)

- Moq GitHub repository - https://github.com/devlooped/moq
- Moq documentation (community patterns) - https://github.com/dezverev/cursorSkillTemplate/blob/main/.cursor/skills/write-unit-tests/moq-patterns.md
- Moq strict vs loose behavior - https://github.com/keghub/unit-testing-csharp/blob/master/docs/moq/mock-customization.md
- NSubstitute official documentation - https://nsubstitute.github.io/
- NSubstitute argument matchers - https://nsubstitute.github.io/help/argument-matchers/
- NSubstitute clearing received calls - https://nsubstitute.github.io/help/clear-received-calls/
- FakeItEasy official documentation - https://fakeiteasy.github.io/
- FakeItEasy quickstart - https://fakeiteasy.github.io/docs/8.0.1/quickstart/
- FakeItEasy argument constraints - https://fakeiteasy.github.io/docs/8.0.1/argument-constraints/
- FakeItEasy ordered assertions - https://fakeiteasy.github.io/docs/8.0.1/ordered-assertions/
- Moq vs NSubstitute vs FakeItEasy comparison 2026 - https://www.pistack.xyz/posts/2026-07-04-csharp-mocking-frameworks-moq-nsubstitute-fakeiteasy-justmock/
- Moq vs NSubstitute vs FakeItEasy in .NET 2026 - https://codingdroplets.com/moq-vs-nsubstitute-vs-fakeiteasy-in-net-which-mocking-framework-should-your-team-use-in-2026
- Mocking best practices — Microsoft (Azure SDK testing) - https://learn.microsoft.com/en-us/dotnet/azure/sdk/unit-testing-mocking
- When to use mocking — Typemock - https://www.typemock.com/when-to-use-mocking-in-unit-testing/
- Testing anti-patterns — OpenSmartRoute - https://opensmartroute.ai/testing-anti-patterns/
- Test doubles — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#terminology
- Mock object pattern — Martin Fowler - https://martinfowler.com/articles/mocksArentStubs.html