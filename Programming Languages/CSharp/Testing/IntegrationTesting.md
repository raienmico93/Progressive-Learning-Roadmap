# Integration Testing in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Integration testing is the practice of verifying that multiple software components — such as an application's HTTP pipeline, its business logic, and its data store — work together correctly as an assembled whole.

**Technical Definition**
Integration testing in .NET, particularly for ASP.NET Core applications, is the process of exercising a running application's components in combination. It validates the wiring between the web host, dependency injection container, middleware pipeline, routing, model binding, filters, business services, and the database. Unlike unit tests, which isolate a single unit, integration tests verify the interactions between these layers, often using a real or realistic database. The `Microsoft.AspNetCore.Mvc.Testing` package provides `WebApplicationFactory<T>`, which bootstraps the application in memory to provide an `HttpClient` for end-to-end HTTP testing without network deployment .

**Beginner-Friendly Explanation**
Integration testing checks that the different parts of your application "talk" to each other correctly. If a unit test is like checking a single Lego brick, an integration test is like snapping a few bricks together to see if they form the right shape — for example, checking that an HTTP request reaches your controller, calls your business logic, saves to the database, and returns the correct response.

---

### Key Characteristics

- **Full Pipeline Coverage** — Exercises routing, model binding, validation, filters, and middleware in a single test .
- **Realistic Wiring** — Uses the application's actual dependency injection (DI) container and configuration .
- **In-Memory Hosting** — `WebApplicationFactory` creates a `TestServer` that hosts the app in memory, eliminating network deployment .
- **Database Fidelity** — Can be configured to use in-memory databases, SQLite, or real database instances via Testcontainers for higher fidelity .
- **Slower than Unit Tests** — Because they involve more moving parts, integration tests take longer to execute than isolated unit tests.
- **High Value** — A single integration test can cover multiple layers and catch wiring errors that unit tests miss .

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and the `Program.cs` entry point (minimal hosting model).
- Understanding of dependency injection and the `IServiceCollection` configuration.
- Familiarity with a unit testing framework (xUnit, NUnit, or MSTest).
- The `Microsoft.AspNetCore.Mvc.Testing` NuGet package.
- Basic knowledge of HTTP, JSON serialization, and database contexts (EF Core).

---

### Related Programming Areas

- **Unit Testing** — For isolated business logic verification.
- **End-to-End (E2E) Testing** — For browser-driven user flows; integration testing is a subset focused on the server side.
- **Test-Driven Development (TDD)** — Integration tests can be written first to define the contract.
- **API Testing** — A specialized form of integration testing focused on HTTP endpoints.

---

### Core Concepts / Features

1. Integration Testing (Overarching Discipline)
2. `WebApplicationFactory<T>`
3. `TestServer`
4. Database Testing
5. API Testing
6. Integration Testing Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Integration Testing (Overarching Discipline)

### Definitions

**Core Definition**
Integration testing is the level of software testing that verifies the interfaces and interactions between integrated components or systems.

**Technical Definition**
In the .NET ecosystem, integration testing focuses on the request/response pipeline of an ASP.NET Core application. It uses an in-memory test host (`TestServer`) created by `WebApplicationFactory<TEntryPoint>` to process `HttpClient` requests through the application's real middleware, routing, and dependency injection infrastructure . This allows developers to validate the application's behavior as a cohesive unit, including authentication, authorization, model validation, serialization, and persistence, without deploying to a real server.

**Beginner-Friendly Explanation**
Integration testing is "testing the team, not just the players." You test how your controller, your service, and your database work together to handle a real request.

---

### Purposes

- To verify the request pipeline, including middleware, routing, and filters.
- To confirm that dependency injection is configured correctly.
- To validate that the database integration works as expected.
- To test authentication and authorization logic through real HTTP requests .
- To catch serialization and deserialization issues.
- To provide higher confidence than unit tests alone before deployment .

---

### Syntax Rules and Structure

#### General Project Setup

```
MySolution/
  src/
    MyApp/                  (ASP.NET Core project)
  tests/
    MyApp.IntegrationTests/
      MyApp.IntegrationTests.csproj
      BasicTests.cs
```

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="8.0.0" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
    <PackageReference Include="xunit" Version="2.7.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.7" />
  </ItemGroup>
</Project>
```

**Component breakdown**

- `Microsoft.AspNetCore.Mvc.Testing` — Provides `WebApplicationFactory<T>` and the in-memory test infrastructure.
- The test project must reference the production project.

#### Syntax Rules

- The test class typically implements `IClassFixture<WebApplicationFactory<TEntryPoint>>` to share the factory across tests .
- `CreateClient()` returns an `HttpClient` configured to send requests to the in-memory server .
- `TEntryPoint` is usually the `Program` class (or `Startup` in older templates).

#### Constraints and Limitations

- Integration tests are slower than unit tests.
- They may require a database or other infrastructure.
- In-memory hosting does not replicate every aspect of a real network environment (e.g., TLS termination, reverse proxies).

---

### Annotated Code Example

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Xunit;

public class BasicTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;

    public BasicTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }

    [Theory]
    [InlineData("/")]
    [InlineData("/Index")]
    [InlineData("/About")]
    public async Task Get_EndpointsReturnSuccessAndCorrectContentType(string url)
    {
        // Arrange
        var client = _factory.CreateClient();

        // Act
        var response = await client.GetAsync(url);

        // Assert
        response.EnsureSuccessStatusCode(); // Status Code 200-299
        Assert.Equal("text/html; charset=utf-8",
            response.Content.Headers.ContentType.ToString());
    }
}
```

**Expected Output**

```
Passed: Get_EndpointsReturnSuccessAndCorrectContentType(url: "/")
Passed: Get_EndpointsReturnSuccessAndCorrectContentType(url: "/Index")
Passed: Get_EndpointsReturnSuccessAndCorrectContentType(url: "/About")
```

**Why this output occurs**

- `WebApplicationFactory<Program>` bootstraps the real application in memory.
- `CreateClient()` provides an `HttpClient` that sends requests through the actual pipeline.
- The test verifies both the status code and the content type for multiple endpoints.

---

### Real-World Cases

- **API health checks** — Verifying that key endpoints return 200 OK.
- **CRUD operations** — Testing that an API can create, read, update, and delete entities via HTTP.
- **Authentication flows** — Verifying that protected endpoints require authentication and return 401/403 appropriately.

---

### References

- Integration tests in ASP.NET Core — Microsoft - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Testing, Performance, And Operations (community guide) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md

---

## 2. `WebApplicationFactory<T>`

### Definitions

**Core Definition**
`WebApplicationFactory<TEntryPoint>` is a class in the `Microsoft.AspNetCore.Mvc.Testing` namespace that bootstraps an ASP.NET Core application in memory for integration testing.

**Technical Definition**
`WebApplicationFactory<TEntryPoint>` creates a `TestServer` instance that hosts the application specified by `TEntryPoint`. It configures the host, infers the content root path, and provides `CreateClient()` to obtain an `HttpClient` that sends requests directly to the in-memory server. It can be customized by inheriting from it and overriding `ConfigureWebHost` to replace services, configure middleware, or change configuration . It is the recommended and standard way to perform integration testing in ASP.NET Core .

**Beginner-Friendly Explanation**
`WebApplicationFactory` is the "launcher" for your integration tests. You give it your application's entry point (`Program`), and it starts your app in a special in-memory mode. Then you can use a normal `HttpClient` to send requests and check the responses.

---

### Purposes

- To bootstrap the SUT (System Under Test) in memory without network deployment.
- To provide an `HttpClient` for making HTTP requests to the in-memory server.
- To allow customization of the web host configuration for testing.
- To enable sharing of the test server across multiple tests in a class or collection .
- To simplify the setup of integration tests by handling content root inference and host configuration .

---

### Syntax Rules and Structure

#### General Syntax

```csharp
public class CustomWebApplicationFactory<TProgram> : WebApplicationFactory<TProgram>
    where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Replace services, configure test database, etc.
        });
    }
}
```

```csharp
// Using the factory
public class Tests : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public Tests(CustomWebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
}
```

**Component breakdown**

- `TProgram` — The entry point class (usually `Program`).
- `ConfigureWebHost` — Override to customize the `IWebHostBuilder` for tests.
- `builder.ConfigureServices` — Replace or add services for testing.
- `CreateClient()` — Returns an `HttpClient` for the in-memory server.

#### Syntax Rules

- The factory can be used directly (`WebApplicationFactory<Program>`) or customized via inheritance.
- `ConfigureWebHost` is the primary extension point for test configuration.
- `ConfigureServices` inside `ConfigureWebHost` runs after the app's own service registration .
- The factory should be shared via `IClassFixture` or `ICollectionFixture` to avoid recreating the server per test .

#### Constraints and Limitations

- `WebApplicationFactory` exposes only a portion of `TestServer`'s functionality; for advanced scenarios (e.g., WebSockets), you may need to use `TestServer` directly .
- The content root path is inferred; if it fails, you can annotate the test assembly with `WebApplicationFactoryContentRootAttribute` .
- Service replacement must be done carefully to avoid breaking dependencies.

---

### Annotated Code Example

```csharp
public class CustomWebApplicationFactory<TProgram> : WebApplicationFactory<TProgram>
    where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor != null) services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseInMemoryDatabase("TestDb"));
        });
    }
}

public class OrderApiTests : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public OrderApiTests(CustomWebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetOrders_ReturnsOk()
    {
        var response = await _client.GetAsync("/api/orders");
        response.EnsureSuccessStatusCode();
    }
}
```

**Expected Output**

```
Passed: GetOrders_ReturnsOk
```

**Why this output occurs**

- The custom factory replaces the production database with an in-memory database.
- The `HttpClient` sends a request through the real pipeline to the `/api/orders` endpoint.
- The endpoint executes and returns a successful response.

---

### Real-World Cases

- **Replacing databases** — Swap a production database for an in-memory or SQLite test database.
- **Mocking external services** — Replace an HTTP client or email sender with a test double.
- **Configuring authentication** — Set up a test authentication handler for protected endpoints.

---

### References

- Integration tests in ASP.NET Core (customization) — Microsoft - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Customizing WebApplicationFactory (community examples) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md

---

## 3. `TestServer`

### Definitions

**Core Definition**
`TestServer` is the in-memory server infrastructure provided by `Microsoft.AspNetCore.TestHost` that executes an ASP.NET Core application without a real network listener.

**Technical Definition**
`TestServer` is the low-level component that `WebApplicationFactory` wraps. It implements an `IServer` and processes `HttpClient` requests directly in memory. `WebApplicationFactory<T>` creates a `TestServer` and exposes it through the `Server` property. While `WebApplicationFactory` is the recommended high-level API, `TestServer` can be used directly for scenarios that require lower-level control, such as WebSocket testing or advanced request building .

**Beginner-Friendly Explanation**
`TestServer` is the "engine" that runs your app in memory. `WebApplicationFactory` is the convenient wrapper around it. Most of the time you use `WebApplicationFactory`, but `TestServer` is there if you need more control.

---

### Purposes

- To execute ASP.NET Core applications without network deployment.
- To provide the underlying infrastructure for `WebApplicationFactory`.
- To enable low-level testing scenarios (e.g., WebSockets, custom request builders) .
- To process HTTP requests entirely in memory for speed and isolation.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Direct use of TestServer (rare; WebApplicationFactory is preferred)
var builder = new WebHostBuilder()
    .UseStartup<Startup>();

var server = new TestServer(builder);
var client = server.CreateClient();
var response = await client.GetAsync("/");
```

**Component breakdown**

- `TestServer` — The in-memory server instance.
- `CreateClient()` — Returns an `HttpClient` connected to the server.
- `Server` property on `WebApplicationFactory` — Provides access to the underlying `TestServer` .

#### Syntax Rules

- `TestServer` is typically used through `WebApplicationFactory`, not directly.
- The `Server` property of `WebApplicationFactory` exposes the `TestServer` instance.
- Requests are processed through the full middleware pipeline.

#### Constraints and Limitations

- `TestServer` lacks the automatic content root inference and configuration that `WebApplicationFactory` provides .
- Using `TestServer` directly is an infrastructure concern; `WebApplicationFactory` is the recommended high-level API .

---

### Annotated Code Example

```csharp
public class TestServerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly TestServer _server;

    public TestServerTests(WebApplicationFactory<Program> factory)
    {
        _server = factory.Server; // Access the underlying TestServer
    }

    [Fact]
    public void TestServer_IsConfigured()
    {
        Assert.NotNull(_server);
        Assert.NotNull(_server.Services); // Access the DI container
    }
}
```

**Expected Output**

```
Passed: TestServer_IsConfigured
```

**Why this output occurs**

- The `WebApplicationFactory` creates and exposes a `TestServer`.
- The test verifies that the server and its services are available.

---

### Real-World Cases

- **WebSocket testing** — `TestServer` provides `CreateWebSocketClient`, which `WebApplicationFactory` does not expose .
- **Advanced request building** — Using `RequestBuilder` for fine-grained control over HTTP requests.

---

### References

- TestServer (Microsoft.AspNetCore.TestHost) - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.testhost.testserver
- Clearing up confusion between WebApplicationFactory and TestServer — GitHub Issue - https://github.com/dotnet/aspnetcore/issues/60492

---

## 4. Database Testing

### Definitions

**Core Definition**
Database testing in integration tests is the practice of verifying an application's data access logic against a real or realistic database engine.

**Technical Definition**
Database integration testing involves configuring the `DbContext` to use a test database — such as SQLite in-memory, an in-memory EF Core provider, or a real database in a Docker container via Testcontainers — and then exercising repositories and services to verify queries, mappings, transactions, and constraints. The choice of provider involves a trade-off between speed and fidelity: SQLite in-memory is the recommended default for most repository behavior, while real database containers (SQL Server, PostgreSQL) provide the highest fidelity for provider-specific SQL and migrations .

**Beginner-Friendly Explanation**
Database testing means "checking that your code saves and retrieves data correctly from a database." You can use a lightweight in-memory database for speed, or spin up a real database in Docker for maximum realism.

---

### Purposes

- To verify EF Core query translation and SQL generation.
- To validate relationships, foreign keys, and constraints.
- To test migrations against a real database engine.
- To ensure that transactions and concurrency behave correctly.
- To catch provider-specific SQL issues that in-memory providers miss .

---

### Syntax Rules and Structure

#### Provider Selection Guide

| Provider | Best For | Avoid For |
|---|---|---|
| **EF Core InMemory** | Very small tests (change tracking, simple interceptors) | Relational semantics, query translation, SQL features  |
| **SQLite in-memory** | Default for repository and relational behavior tests | Provider-specific SQL Server/PostgreSQL behavior  |
| **SQL Server / PostgreSQL in Docker** | Final confidence for bulk operations, migrations, provider-specific SQL  | Everyday fast feedback loops |

#### General Setup (SQLite In-Memory)

```csharp
var connection = new SqliteConnection("Data Source=:memory:");
await connection.OpenAsync();

var services = new ServiceCollection();
services.AddDbContext<AppDbContext>(options => options.UseSqlite(connection));

await using var provider = services.BuildServiceProvider();
await using var scope = provider.CreateAsyncScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
await db.Database.EnsureCreatedAsync();
```

**Component breakdown**

- `SqliteConnection("Data Source=:memory:")` — Creates an in-memory SQLite database.
- `UseSqlite(connection)` — Configures EF Core to use the open connection.
- `EnsureCreatedAsync()` — Creates the schema (or use `Migrate()` for migrations).

#### General Setup (Testcontainers)

```csharp
var postgres = new PostgreSqlBuilder().Build();
await postgres.StartAsync();

var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseNpgsql(postgres.GetConnectionString())
    .Options;
```

**Component breakdown**

- `PostgreSqlBuilder` — Testcontainers builder for PostgreSQL.
- `StartAsync()` — Starts the Docker container.
- `GetConnectionString()` — Retrieves the dynamic connection string .

#### Syntax Rules

- Use `IAsyncLifetime` for async setup and teardown of containers .
- Use a `Wait` strategy to ensure the database is ready before running tests .
- Clean data between tests using transactions, table truncation, or fresh containers .
- Apply migrations to the test database to ensure schema fidelity .

#### Constraints and Limitations

- In-memory databases do not enforce relational constraints the same way as real databases.
- SQLite and SQL Server/PostgreSQL have different SQL dialects and behaviors .
- Testcontainers requires Docker to be installed and running.
- Container startup adds significant time to test execution.

---

### Annotated Code Example

```csharp
public class DatabaseFixture : IAsyncLifetime
{
    private PostgreSqlContainer _postgres;
    public AppDbContext Db { get; private set; }

    public async Task InitializeAsync()
    {
        _postgres = new PostgreSqlBuilder().Build();
        await _postgres.StartAsync();

        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseNpgsql(_postgres.GetConnectionString())
            .Options;

        Db = new AppDbContext(options);
        await Db.Database.MigrateAsync();
    }

    public async Task DisposeAsync()
    {
        await Db.DisposeAsync();
        await _postgres.DisposeAsync();
    }
}

public class OrderRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public OrderRepositoryTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public async Task AddOrder_SavesToDatabase()
    {
        var repo = new OrderRepository(_fixture.Db);
        await repo.AddAsync(new Order { Total = 100m });
        var count = await _fixture.Db.Orders.CountAsync();
        Assert.Equal(1, count);
    }
}
```

**Expected Output**

```
Passed: AddOrder_SavesToDatabase
```

**Why this output occurs**

- The PostgreSQL container is started, and migrations are applied.
- The repository saves an order.
- The test queries the database to verify persistence.

---

### Real-World Cases

- **Repository tests** — Verify that EF Core queries return correct data.
- **Migration testing** — Ensure that migrations apply cleanly to a real database.
- **Bulk operations** — Validate performance and correctness of `ExecuteUpdate` and `ExecuteDelete`.

---

### References

- Testing EF Core applications — Microsoft - https://learn.microsoft.com/en-us/ef/core/testing/
- Pragmatic EF Core testing guide (community) - https://github.com/pragmatic-design/Pragmatic.Design/blob/main/Pragmatic.Persistence/docs/efcore/07-testing-generated-persistence.md
- Testcontainers for .NET — Docker - https://docs-labs.docker.com/guides/testcontainers-dotnet/
- Testcontainers database patterns (community) - https://github.com/Aaronontheweb/dotnet-skills/blob/main/skills/testcontainers/database-patterns.md

---

## 5. API Testing

### Definitions

**Core Definition**
API testing is a form of integration testing that verifies the behavior of HTTP endpoints, including routing, model binding, validation, authentication, and response formatting.

**Technical Definition**
API integration testing uses `WebApplicationFactory<Program>` to send real HTTP requests through the application's pipeline to API controllers or minimal API endpoints. It asserts on status codes, response headers, and response body content. This approach validates the complete request/response cycle, including serialization, filters, middleware, and error handling, without bypassing the framework . It is considered the highest-value test pattern because a single test covers routing, binding, validation, business logic, and persistence .

**Beginner-Friendly Explanation**
API testing is "checking that your web endpoints work correctly." You send a GET, POST, PUT, or DELETE request to your API and check that it returns the right status code and the right JSON.

---

### Purposes

- To validate routing and endpoint configuration.
- To verify model binding and validation attributes.
- To test authentication and authorization policies through real HTTP requests .
- To confirm correct serialization of responses (JSON, XML).
- To ensure that error responses (400, 404, 500) are handled correctly.
- To catch middleware and filter misconfigurations.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
var client = factory.CreateClient();

// GET
var response = await client.GetAsync("/api/orders/1");
response.EnsureSuccessStatusCode();
var order = await response.Content.ReadFromJsonAsync<OrderDto>();

// POST
var newOrder = new CreateOrderDto { Total = 100m };
var postResponse = await client.PostAsJsonAsync("/api/orders", newOrder);
postResponse.EnsureSuccessStatusCode();

// PUT / DELETE
await client.PutAsJsonAsync("/api/orders/1", updatedOrder);
await client.DeleteAsync("/api/orders/1");
```

**Component breakdown**

- `CreateClient()` — Returns an `HttpClient` for the in-memory server.
- `GetAsync`, `PostAsJsonAsync`, `PutAsJsonAsync`, `DeleteAsync` — Send HTTP requests.
- `ReadFromJsonAsync<T>()` — Deserializes the response body.

#### Syntax Rules

- Use `EnsureSuccessStatusCode()` or assert on `response.StatusCode` to verify success.
- Use `ReadFromJsonAsync<T>()` to deserialize JSON responses.
- Configure `WebApplicationFactoryClientOptions` to control redirects and cookies .
- Test both success and failure paths (e.g., 400, 404, 401).

#### Constraints and Limitations

- In-memory hosting does not test network-level concerns (TLS, proxies).
- Antiforgery tokens and cookies may require special handling .
- Authentication may need a test authentication handler.

---

### Annotated Code Example

```csharp
public class OrdersApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public OrdersApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task CreateOrder_ValidRequest_ReturnsCreated()
    {
        var dto = new CreateOrderDto { ProductId = 1, Quantity = 2 };

        var response = await _client.PostAsJsonAsync("/api/orders", dto);

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        var created = await response.Content.ReadFromJsonAsync<OrderDto>();
        Assert.NotNull(created);
        Assert.True(created.Id > 0);
    }

    [Fact]
    public async Task GetOrder_NotFound_Returns404()
    {
        var response = await _client.GetAsync("/api/orders/9999");
        Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
    }
}
```

**Expected Output**

```
Passed: CreateOrder_ValidRequest_ReturnsCreated
Passed: GetOrder_NotFound_Returns404
```

**Why this output occurs**

- The POST request is processed through the full pipeline, including model binding and validation.
- The API returns 201 Created with the created order.
- The GET request for a non-existent order returns 404 Not Found.

---

### Real-World Cases

- **REST API validation** — Testing CRUD endpoints.
- **Authentication testing** — Verifying that JWT tokens or cookies grant/deny access.
- **Error handling** — Ensuring that validation errors return 400 with a problem details body.

---

### References

- Integration tests in ASP.NET Core — Microsoft - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- API Integration Testing (community example) - https://github.com/abpframework/abp/blob/dev/docs/en/Testing/API-Integration-Testing.md

---

## 6. Integration Testing Best Practices

### Definitions

**Core Definition**
Integration testing best practices are guidelines for writing tests that are reliable, maintainable, fast enough to run frequently, and that provide high confidence in the integrated behavior of the application.

**Technical Definition**
Best practices encompass using layered testing (unit tests for logic, integration tests for wiring, and E2E tests for user flows) , sharing the `WebApplicationFactory` via fixtures, choosing the appropriate database provider for the test's purpose, ensuring test isolation through cleanup or fresh containers, controlling the environment, and verifying both happy and error paths. They also include avoiding over-testing permutations and focusing on high-value integration points .

**Beginner-Friendly Explanation**
Best practices are "how to write integration tests that help without slowing you down." You share expensive setup, use the right database for the job, clean up between tests, and don't test every possible combination.

---

### Purposes

- To keep integration tests fast enough to run on every commit.
- To ensure tests are reliable and not flaky.
- To maximize confidence per test.
- To reduce maintenance cost.
- To complement unit tests effectively.

---

### Guidelines and Structure

#### Use Layered Testing

```
Unit Tests          → Pure business logic, no I/O
Integration Tests   → Request pipeline, DI, database, auth, framework wiring 
E2E Tests           → Full user flows (Playwright, Selenium)
```

- Do not replace unit tests with integration tests; they serve different purposes.
- Integration tests are the "sweet spot" for API correctness .

#### Share the Factory, Not the State

```csharp
public class Tests : IClassFixture<WebApplicationFactory<Program>>
{
    // Shared factory, fresh HttpClient per test
}
```

- Use `IClassFixture<WebApplicationFactory<T>>` to create the server once per test class .
- Create a new `HttpClient` for each test to avoid cookie/header leakage.

#### Choose the Right Database Provider

- **Default:** SQLite in-memory for relational behavior .
- **High fidelity:** Testcontainers (SQL Server, PostgreSQL) for provider-specific SQL and migrations .
- **Avoid:** EF Core InMemory for anything requiring relational semantics .

#### Ensure Test Isolation

- Clean the database between tests (transactions, truncation, or fresh containers) .
- Use `IAsyncLifetime` for async setup/teardown .
- Use random ports for Testcontainers .

#### Control the Environment

- Set `ASPNETCORE_ENVIRONMENT` to `Testing` or a custom value .
- Disable shadow copying if tests rely on file paths .

#### Test Both Success and Failure Paths

- Verify 200 OK for valid requests.
- Verify 400 Bad Request for validation errors.
- Verify 401/403 for unauthorized requests.
- Verify 404 for missing resources.

#### Avoid Over-Testing

- Do not write integration tests for every permutation of data .
- Focus on high-value integration points: controllers, middleware, database queries.
- Unit tests should cover exhaustive input permutations.

#### Use Test Doubles Appropriately

- Replace external services (email, payment gateways) with test doubles when needed .
- Use real databases for data-access tests, not mocks.

---

### Annotated Code Example

```csharp
public class OrdersApiTests : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly CustomWebApplicationFactory<Program> _factory;

    public OrdersApiTests(CustomWebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }

    [Fact]
    public async Task GetOrders_ReturnsOkAndJson()
    {
        // Arrange: fresh client per test
        var client = _factory.CreateClient();

        // Act
        var response = await client.GetAsync("/api/orders");

        // Assert: status and content type
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json", response.Content.Headers.ContentType.MediaType);
    }

    [Fact]
    public async Task CreateOrder_InvalidRequest_ReturnsBadRequest()
    {
        var client = _factory.CreateClient();
        var invalidDto = new CreateOrderDto { Quantity = -1 }; // invalid

        var response = await client.PostAsJsonAsync("/api/orders", invalidDto);

        Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
    }
}
```

**Expected Output**

```
Passed: GetOrders_ReturnsOkAndJson
Passed: CreateOrder_InvalidRequest_ReturnsBadRequest
```

**Why this output occurs**

- The shared factory bootstraps the application once.
- Each test creates a fresh `HttpClient` to avoid state leakage.
- Both success and failure paths are verified.

---

### Real-World Cases

- **CI pipelines** — Integration tests run after unit tests and before E2E tests.
- **API contract verification** — Ensuring that endpoints match the OpenAPI specification.
- **Database migration safety** — Running integration tests against a containerized database to validate migrations.

---

### References

- Integration tests in ASP.NET Core — Microsoft - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Testing, Performance, And Operations (community guide) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- Testcontainers best practices (community) - https://github.com/Aaronontheweb/dotnet-skills/blob/main/skills/testcontainers/database-patterns.md
- Integration test isolation (community) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md

---

## Framework and Tooling Summary

| Tool / Package | Purpose | When to Use |
|---|---|---|
| `Microsoft.AspNetCore.Mvc.Testing` | `WebApplicationFactory<T>` | Standard integration tests for ASP.NET Core |
| `Microsoft.AspNetCore.TestHost` | `TestServer` | Low-level scenarios (WebSockets, custom request builders) |
| `Microsoft.EntityFrameworkCore.Sqlite` | SQLite in-memory database | Default for repository/relational behavior tests  |
| `Testcontainers.PostgreSql` / `Testcontainers.MsSql` | Real database in Docker | High-fidelity tests for provider-specific SQL and migrations  |
| `xunit` | Test runner | Running integration tests |
| `Respawn` (community) | Database cleanup | Resetting database state between tests  |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| EF Core InMemory provider | Not a relational database; avoid for relational semantics and query translation tests  |
| `WebApplicationFactory` using `TestServer` directly with `ConfigureClient` | Configuration may not be reflected; use with caution  |
| `TestServer` as the primary API | `WebApplicationFactory` is the recommended high-level API; `TestServer` is an infrastructure concept  |
| `CreateWebSocketClient` | Only available on `TestServer`, not `WebApplicationFactory`  |
| `EnsureCreated()` with migrations | Incompatible; choose one approach |
| In-memory hosting limitations | Does not test TLS, reverse proxies, or network-level concerns |
| Testcontainers startup time | Containers add seconds to test execution; use strategically  |

---

## References (Consolidated)

- Integration tests in ASP.NET Core — Microsoft - https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Clearing up confusion when using WebApplicationFactory and TestServer — GitHub Issue - https://github.com/dotnet/aspnetcore/issues/60492
- Testing EF Core applications — Microsoft - https://learn.microsoft.com/en-us/ef/core/testing/
- Pragmatic EF Core testing guide (community) - https://github.com/pragmatic-design/Pragmatic.Design/blob/main/Pragmatic.Persistence/docs/efcore/07-testing-generated-persistence.md
- Testcontainers for .NET — Docker - https://docs-labs.docker.com/guides/testcontainers-dotnet/
- Testcontainers database patterns (community) - https://github.com/Aaronontheweb/dotnet-skills/blob/main/skills/testcontainers/database-patterns.md
- Testing, Performance, And Operations (community guide) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- TestServer (Microsoft.AspNetCore.TestHost) - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.testhost.testserver
- Integration test isolation best practices (community) - https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- API Integration Testing (community) - https://github.com/abpframework/abp/blob/dev/docs/en/Testing/API-Integration-Testing.md