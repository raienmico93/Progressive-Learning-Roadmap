# ASP.NET Core Integration Testing: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the integration testing ecosystem built around **`Microsoft.AspNetCore.Mvc.Testing`**. Integration tests verify that multiple components — middleware, routing, model binding, filters, DI, and the database — work together correctly. They sit between unit tests (fast, isolated) and end‑to‑end tests (slow, browser‑driven) on the **Test Pyramid**. A critical distinction: **`WebApplicationFactory<T>`** is the high‑level API you use; **`TestServer`** is the in‑memory server it bootstraps internally. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Integration testing is the practice of verifying that multiple components of an application work together correctly by exercising the full HTTP request pipeline — middleware, routing, model binding, filters, dependency injection, and database access — without deploying to a real server. |
| **Technical Definition** | ASP.NET Core integration tests use **`WebApplicationFactory<TEntryPoint>`** to bootstrap the real application in‑process with a **`TestServer`**. `CreateClient()` returns an `HttpClient` that sends requests through the complete middleware pipeline to the actual endpoints. The factory can be customized to replace services (e.g., swap the database for an in‑memory provider or a **Testcontainer**), seed test data, and mock authentication. Assertions verify HTTP status codes, content types, response bodies, and database side effects. |
| **Beginner‑Friendly Explanation** | Integration testing is the **“test the whole plumbing”** check. Instead of testing one method in isolation (unit test), you send a real HTTP request to your app — through routing, middleware, controllers, and JSON serialization — and check the response. It catches the bugs that unit tests miss: a missing `[Authorize]`, a misconfigured route, a broken DI registration, or a serialization issue that only appears when the full pipeline runs. |

### Key Characteristics

- **Full‑pipeline verification** – Requests traverse middleware, routing, filters, and controllers.
- **In‑process hosting** – `TestServer` runs the app without a network port, keeping tests fast.
- **Real HTTP semantics** – `HttpClient` sends and receives real HTTP messages; status codes, headers, and bodies are verified.
- **Configurable dependencies** – `WebApplicationFactory` can replace services (database, authentication) per test scenario.
- **Slower than unit tests, faster than E2E** – Typically seconds per test vs. milliseconds (unit) or minutes (E2E).
- **Database‑aware** – Can use in‑memory providers, SQLite in‑memory, or **Testcontainers** for production‑like behaviour.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- `Microsoft.AspNetCore.Mvc.Testing` package in the test project.
- A test framework (**xUnit** recommended).
- For database testing: **EF Core InMemory**, **SQLite in‑memory**, or **Testcontainers** (Docker).
- Familiarity with **xUnit** (`IClassFixture`, `IAsyncLifetime`) and **dependency injection**.

### Related Programming Areas

- **Unit testing** – Tests isolated units; integration tests verify wiring.
- **End‑to‑end testing** – Browser automation (Playwright) for full‑stack flows.
- **Dependency injection** – The factory replaces services to isolate tests.
- **Middleware pipeline** – Integration tests exercise the real pipeline end‑to‑end.
- **Database testing** – Testcontainers, Respawn, and in‑memory providers.

### Core Concepts / Features

1. **Integration testing** – The discipline, the pipeline, and the Test Pyramid placement.
2. **`WebApplicationFactory<T>`** – Bootstrapping the app in‑process and customizing it.
3. **`TestServer`** – The in‑memory server that hosts the application.
4. **Database testing** – In‑memory providers, SQLite, Testcontainers, and Respawn.
5. **API testing** – Verifying status codes, content types, JSON bodies, auth, and validation.
6. **Integration testing best practices** – Isolation, test data, authentication mocking, and CI integration.

---

## 1. Integration Testing (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Integration testing verifies that multiple components of an application work together correctly by exercising the full request pipeline end‑to‑end. |
| **Technical Definition** | Integration tests in ASP.NET Core use `WebApplicationFactory<TEntryPoint>` to create a `TestServer` that hosts the application in‑memory. An `HttpClient` sends requests through the complete middleware pipeline — routing, filters, model binding, controllers, and serialization — and the test asserts on the HTTP response. The factory can replace registered services (database, authentication) to isolate the test scenario. |
| **Beginner‑Friendly Explanation** | Integration testing is the **“send a real request and check the real response”** test. It is not about testing one method in isolation (that is a unit test) — it is about testing that the whole machine works together: the route matches, the middleware runs, the controller executes, the database is queried, and the JSON comes back correctly. |

### Purposes

- **To** verify **end‑to‑end HTTP behaviour** — request → middleware → controller → response.
- **To** catch **wiring bugs** — DI configuration, routing, filters, serialization, and middleware order.
- **To** test **authentication and authorization** in the real pipeline (401 vs. 403).
- **To** validate **database integration** — EF Core queries, migrations, and constraints.
- **To** provide **confidence before deployment** that the application works as assembled, not just as individual pieces.

### Syntax Rules and Structure

#### The Integration Test Pipeline

```text
Test Method → HttpClient → TestServer → Middleware Pipeline → Routing
→ Controller/Endpoint → Services → Database → Response → Assertions
```

**Component breakdown:**

| Component | Responsibility |
|-----------|----------------|
| **Test method** | Defines the scenario and assertions. |
| **`HttpClient`** | Sends HTTP requests to the `TestServer`. |
| **`TestServer`** | Hosts the application in‑memory. |
| **Middleware pipeline** | The real ASP.NET Core pipeline (auth, CORS, etc.). |
| **Endpoint** | The controller action or Minimal API handler. |
| **Services / Database** | The real or replaced dependencies. |

**Syntax rules:**

- **Integration tests must use `WebApplicationFactory<Program>`** (or a custom subclass) —`Program` is the entry point in .NET 6+ top‑level statements.
- **`CreateClient()`** returns an `HttpClient` configured to talk to the `TestServer`.
- **Test classes should use `IClassFixture<WebApplicationFactory<Program>>`** to share the factory across tests in the class.

**Constraints and limitations:**

- **Integration tests are slower** than unit tests —limit them to critical infrastructure scenarios, not every permutation of data.
- **In‑memory databases are not production** —use Testcontainers for realistic database behaviour.
- **Authentication requires configuration** —use a test authentication handler or inject a fake `ClaimsPrincipal`.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class BasicTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public BasicTests(WebApplicationFactory<Program> factory)
        => _client = factory.CreateClient();

    [Theory]
    [InlineData("/")]
    [InlineData("/api/products")]
    public async Task Get_EndpointsReturnSuccessAndCorrectContentType(string url)
    {
        // Act
        var response = await _client.GetAsync(url);

        // Assert
        response.EnsureSuccessStatusCode();   // 200–299
        Assert.Equal("application/json",
            response.Content.Headers.ContentType?.MediaType);
    }
}
```

**Step‑by‑step explanation:**

1. `IClassFixture<WebApplicationFactory<Program>>` shares the factory across all tests in the class.
2. `CreateClient()` returns an `HttpClient` connected to the in‑process `TestServer`.
3. Each `[InlineData]` row sends a GET request to a different URL.
4. `EnsureSuccessStatusCode()` asserts a 2xx status; the content type is verified.

**Expected Output:** Both endpoints return `200 OK` with `application/json` content type. If the route were misconfigured or the serialization broken, the test would fail.

**Why this output?** The request traverses the real middleware pipeline and endpoint; the response reflects the actual application behaviour.

### Real‑World Cases

- **API endpoint verification:** Status codes, content types, headers.
- **Authentication/authorization:** 401 vs. 403 with real policies.
- **Middleware behaviour:** CORS, exception handling, rate limiting.
- **Database integration:** Repository queries against a test database.

### References

- Integration tests in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- ASP.NET Core Integration Testing (GitHub) – https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/csharp/testing/integration.md

---

## 2. `WebApplicationFactory<T>` (Bootstrapping and Customization)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `WebApplicationFactory<TEntryPoint>` is the primary class for bootstrapping an ASP.NET Core application in‑process for integration testing. |
| **Technical Definition** | `WebApplicationFactory<TEntryPoint>` (in `Microsoft.AspNetCore.Mvc.Testing`) creates a **`TestServer`** for the application and provides `CreateClient()` to obtain an `HttpClient` connected to it. `TEntryPoint` is the entry point class of the SUT —typically `Program` (or the legacy `Startup`). A **custom factory** subclasses `WebApplicationFactory<T>` and overrides `ConfigureWebHost(IWebHostBuilder)` to replace services, set the environment, and seed test data. |
| **Beginner‑Friendly Explanation** | `WebApplicationFactory` is the **“launcher”** for integration tests. You give it your application’s entry point (`Program`), and it starts the app in memory so you can send HTTP requests to it. If you need to replace the database or mock authentication, you create a subclass and override `ConfigureWebHost` —that is where you swap out real services for test doubles. |

### Purposes

- **To** **bootstrap the real application** in‑process without network overhead.
- **To** **provide an `HttpClient`** connected to the `TestServer`.
- **To** **customize the host** —replace services, set the environment, seed data.
- **To** **share the factory** across tests via `IClassFixture`.
- **To** **isolate test scenarios** by swapping dependencies (database, auth) per factory.

### Syntax Rules and Structure

#### General Syntax (Default Factory)

```csharp
public class BasicTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public BasicTests(WebApplicationFactory<Program> factory)
        => _client = factory.CreateClient();
}
```

#### General Syntax (Custom Factory)

```csharp
public class CustomWebApplicationFactory<TProgram>
    : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove the real DbContext registration
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor is not null) services.Remove(descriptor);

            // Add an in-memory database for testing
            services.AddDbContext<AppDbContext>(options =>
                options.UseInMemoryDatabase("TestDb"));

            // Seed test data
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Database.EnsureCreated();
            SeedTestData(db);
        });

        builder.UseEnvironment("Testing");
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: WebApplicationFactory<TProgram>` | Subclass the factory. |
| `ConfigureWebHost(IWebHostBuilder)` | Override to customize the host. |
| `builder.ConfigureServices(...)` | Replace or add services. |
| `services.Remove(descriptor)` | Remove the production `DbContext` registration. |
| `builder.UseEnvironment("Testing")` | Set the environment to `Testing`. |

**Syntax rules:**

- **`TEntryPoint` must be accessible** —typically `public partial class Program { }` in `Program.cs` (top‑level statements).
- **`ConfigureWebHost` is called after the app’s own `ConfigureServices`** —you can override registrations.
- **Use `IClassFixture<CustomWebApplicationFactory<T>>`** to share the customized factory.

**Constraints and limitations:**

- **`ConfigureServices` order matters** —the test callback runs after the app’s registration, so `Remove` works for the `DbContext`.
- **Avoid `BuildServiceProvider` in production** —it is acceptable in test factory setup but should not be called repeatedly.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class CustomWebApplicationFactory<TProgram>
    : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor is not null) services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseInMemoryDatabase("TestDb"));

            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Database.EnsureCreated();
            SeedTestData(db);
        });
    }

    private void SeedTestData(AppDbContext db)
    {
        db.Users.Add(new User { Id = 1, Name = "Alice" });
        db.Users.Add(new User { Id = 2, Name = "Bob" });
        db.SaveChanges();
    }
}

public class UsersApiTests : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public UsersApiTests(CustomWebApplicationFactory<Program> factory)
        => _client = factory.CreateClient();

    [Fact]
    public async Task GetUsers_ReturnsSeededUsers()
    {
        var response = await _client.GetAsync("/api/users");
        var users = await response.Content.ReadFromJsonAsync<List<UserDto>>();

        Assert.Equal(2, users!.Count);
        Assert.Contains(users, u => u.Name == "Alice");
    }
}
```

**Step‑by‑step explanation:**

1. The custom factory replaces the production `DbContext` with an in‑memory database.
2. Test data is seeded before any test runs.
3. The test class shares the factory via `IClassFixture`.
4. `GetUsers` sends a request; the response contains the seeded users.

**Expected Output:** The endpoint returns the two seeded users; the test asserts on the count and content.

**Why this output?** The factory swapped the database for an in‑memory provider and seeded known data, making the test deterministic.

### Real‑World Cases

- **In‑memory database tests:** Fast, isolated, no Docker required.
- **Authentication mocking:** Replace the authentication handler with a test handler.
- **Environment‑specific configuration:** Set `Testing` to load test‑specific `appsettings.Testing.json`.

### References

- Basic tests with the default `WebApplicationFactory` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#basic-tests-with-the-default-webapplicationfactory
- Customizing `WebApplicationFactory` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#customize-webapplicationfactory

---

## 3. `TestServer` (The In‑Memory Server)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `TestServer` is an in‑memory ASP.NET Core server that hosts the application for the duration of a test, eliminating the need for a network port. |
| **Technical Definition** | `TestServer` (in `Microsoft.AspNetCore.TestHost`) hosts an ASP.NET Core application using the real `IWebHost`/`IHost` pipeline but without binding to a TCP port. `WebApplicationFactory` creates and manages the `TestServer` automatically. `TestServer.CreateClient()` returns an `HttpClient` that dispatches requests directly into the server’s middleware pipeline. The server is disposed when the factory is disposed. |
| **Beginner‑Friendly Explanation** | `TestServer` is the **“fake web server”** that runs your app in memory. It is the engine inside `WebApplicationFactory`. You rarely interact with it directly —the factory creates it, gives you an `HttpClient`, and disposes it when the tests finish. It makes integration tests fast because there is no real network stack involved. |

### Purposes

- **To** **host the application in memory** without binding a TCP port.
- **To** **dispatch HTTP requests** directly into the middleware pipeline.
- **To** **share the server** across tests via the factory fixture.
- **To** **dispose the server** automatically when tests complete.
- **To** **support middleware testing** in isolation (e.g., testing a custom middleware component).

### Syntax Rules and Structure

#### General Syntax (Implicit via Factory)

```csharp
var factory = new WebApplicationFactory<Program>();
var client = factory.CreateClient();   // TestServer is created internally
```

#### General Syntax (Explicit `TestServer` for Middleware Testing)

```csharp
var host = await new HostBuilder()
    .ConfigureWebHost(webBuilder =>
    {
        webBuilder.UseTestServer()
            .Configure(app => app.UseMiddleware<MyMiddleware>());
    })
    .StartAsync();

var client = host.GetTestClient();
var response = await client.GetAsync("/");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseTestServer()` | Configures the host to use `TestServer`. |
| `GetTestClient()` | Returns an `HttpClient` connected to the server. |
| `WebApplicationFactory` | Creates a `TestServer` automatically. |

**Syntax rules:**

- **`TestServer` is created by `WebApplicationFactory`** —you do not need to instantiate it directly for typical integration tests.
- **`TestServer` uses the real middleware pipeline** —it is not a mock.
- **The server is disposed** when the factory (or host) is disposed.

**Constraints and limitations:**

- **`TestServer` does not support real network protocols** —WebSockets, HTTP/2, and streaming may behave differently than Kestrel.
- **Some middleware may behave differently** —e.g., HTTPS redirection, forwarded headers.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class MiddlewareTests
{
    [Fact]
    public async Task CustomMiddleware_AddsHeader()
    {
        using var host = await new HostBuilder()
            .ConfigureWebHost(webBuilder =>
            {
                webBuilder.UseTestServer()
                    .Configure(app => app.UseMiddleware<HeaderMiddleware>());
            })
            .StartAsync();

        var client = host.GetTestClient();
        var response = await client.GetAsync("/");

        Assert.True(response.Headers.Contains("X-Custom"));
    }
}
```

**Expected Output:** The response contains the `X-Custom` header added by `HeaderMiddleware`.

**Why this output?** The `TestServer` hosts only the middleware under test, isolating it from the full application.

### Real‑World Cases

- **Middleware isolation tests:** Test a custom middleware without booting the full app.
- **Lightweight integration tests:** When you need the pipeline but not the full application.

### References

- `TestServer` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.testhost.testserver
- Test middleware – https://learn.microsoft.com/en-us/aspnet/core/test/middleware

---

## 4. Database Testing (In‑Memory, SQLite, Testcontainers, Respawn)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Database testing in integration tests verifies that the application’s data access layer works correctly against a database — either an in‑memory substitute or a real database engine. |
| **Technical Definition** | Integration tests can use: **(1) EF Core InMemory** —a fast, non‑relational provider that does not enforce constraints; **(2) SQLite in‑memory** —a real relational engine that supports most SQL; **(3) Testcontainers** —Docker containers running the production database (SQL Server, PostgreSQL, etc.); **(4) Respawn** —a utility that resets the database to a clean checkpoint between tests. The choice depends on the fidelity required vs. the speed desired. |
| **Beginner‑Friendly Explanation** | Database testing is **“does the data layer actually work?”** You have three options: a fake in‑memory database (fast, but not realistic), SQLite (a real database but not the same as production), or Testcontainers (a real database in Docker —the most realistic but slowest). Respawn helps by wiping the database between tests so each test starts clean. |

### Purposes

- **To** verify **EF Core queries, migrations, and constraints** work correctly.
- **To** catch **database‑specific bugs** (collation, SQL dialect, index behaviour).
- **To** ensure **test isolation** —each test starts from a known database state.
- **To** balance **speed and fidelity** —in‑memory for fast tests, Testcontainers for realistic tests.
- **To** validate **seed data and migrations** as part of the application.

### Syntax Rules and Structure

#### Option 1: EF Core InMemory (Fast, Not Realistic)

```csharp
services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));
```

**When to use:** Simple CRUD tests where relational fidelity is not critical.

#### Option 2: SQLite InMemory (Relational, Fast)

```csharp
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open();
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(connection));
```

**When to use:** Tests that need relational behaviour (joins, constraints) without Docker.

#### Option 3: Testcontainers (Realistic, Docker)

```csharp
public class DatabaseFixture : IAsyncLifetime
{
    private readonly MsSqlContainer _container = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    public string ConnectionString => _container.GetConnectionString();

    public Task InitializeAsync() => _container.StartAsync();
    public Task DisposeAsync() => _container.DisposeAsync().AsTask();
}
```

**When to use:** Production‑grade fidelity —SQL dialect, collation, constraints.

#### Option 4: Respawn (Database Cleanup)

```csharp
var respawner = await Respawner.CreateAsync(connectionString, new RespawnerOptions
{
    TablesToIgnore = new[] { "__EFMigrationsHistory" }
});
await respawner.ResetAsync(connectionString);
```

**When to use:** Reset the database between tests without dropping and recreating it.

**Syntax rules:**

- **InMemory** is the fastest but least realistic —it does not enforce foreign keys or unique constraints.
- **SQLite in‑memory** is more realistic but still not the production database.
- **Testcontainers** is the most realistic but requires Docker and is slower.
- **Respawn** should be used with a real database to clean state between tests.

**Constraints and limitations:**

- **InMemory is not recommended for testing** —Microsoft explicitly discourages its use beyond simple demos.
- **Testcontainers requires Docker** —CI environments must support Docker.
- **SQLite has type limitations** (`DateTimeOffset`, `decimal`) —use value converters if needed.

### Annotated Complete Step‑by‑Step Code Example (Testcontainers)

```csharp
public class IntegrationTestWebAppFactory : WebApplicationFactory<Program>, IAsyncLifetime
{
    private readonly PostgreSqlContainer _dbContainer = new PostgreSqlBuilder()
        .WithImage("postgres:15-alpine")
        .WithDatabase("testdb")
        .Build();

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor is not null) services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseNpgsql(_dbContainer.GetConnectionString()));
        });
    }

    public async Task InitializeAsync() => await _dbContainer.StartAsync();
    public new async Task DisposeAsync() => await _dbContainer.DisposeAsync();
}

public class ProductsApiTests : IClassFixture<IntegrationTestWebAppFactory>
{
    private readonly HttpClient _client;
    public ProductsApiTests(IntegrationTestWebAppFactory factory)
        => _client = factory.CreateClient();

    [Fact]
    public async Task CreateProduct_PersistsToDatabase()
    {
        var response = await _client.PostAsJsonAsync("/api/products",
            new { name = "Laptop", price = 999.99m });

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

**Expected Output:** The POST creates a product in a real PostgreSQL container; the test verifies `201 Created`.

**Why this output?** Testcontainers spins up a real PostgreSQL instance; the application connects to it exactly as it would in production.

### Real-World Cases

- **SQL Server/PostgreSQL fidelity:** Testcontainers for production‑like database behaviour.
- **Fast CI pipelines:** SQLite in‑memory for relational tests without Docker overhead.
- **Database resets:** Respawn between tests to ensure isolation.

### References

- Testing an ASP.NET Core web app with Testcontainers – https://docs.docker.com/guides/testcontainers-dotnet-aspnet-core/
- Do you use Testcontainers for integration testing? (SSW Rules) – https://www.ssw.com.au/rules/use-testcontainers
- Respawn (GitHub) – https://github.com/jbogard/Respawn
- EF Core InMemory provider – https://learn.microsoft.com/en-us/ef/core/providers/in-memory/

---

## 5. API Testing (Verifying HTTP Behaviour)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API testing in integration tests verifies that HTTP endpoints return the correct status codes, headers, and response bodies, and that authentication and validation behave as expected. |
| **Technical Definition** | API integration tests use `HttpClient` to send requests to the `TestServer` and assert on the `HttpResponseMessage` —status code, content type, headers, and deserialized body. Authentication can be tested by configuring a test authentication handler (`TestAuthHandler`) that bypasses real credential validation but exercises the authorization pipeline. |
| **Beginner‑Friendly Explanation** | API testing is **“does the endpoint return the right thing?”** You send a `GET /api/products/1` and check that it returns `200 OK` with the correct JSON. You send a request without authentication and check that it returns `401`. You send invalid data and check that it returns `400` with validation errors. |

### Purposes

- **To** verify **status codes** —200, 201, 204, 400, 401, 403, 404, 409, 422.
- **To** verify **content types** —`application/json`, `application/xml`.
- **To** verify **response bodies** —deserialize JSON and assert on properties.
- **To** test **authentication and authorization** —401 vs. 403 in the real pipeline.
- **To** test **validation** —`400 Bad Request` with `ProblemDetails` or `ValidationProblemDetails`.

### Syntax Rules and Structure

#### General Syntax (GET and Assert)

```csharp
[Fact]
public async Task GetProduct_ExistingId_Returns200WithJson()
{
    var response = await _client.GetAsync("/api/products/1");

    Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    Assert.Equal("application/json",
        response.Content.Headers.ContentType?.MediaType);

    var product = await response.Content.ReadFromJsonAsync<ProductDto>();
    Assert.NotNull(product);
    Assert.Equal("Laptop", product.Name);
}
```

#### General Syntax (POST and Assert Created)

```csharp
[Fact]
public async Task CreateProduct_ValidInput_Returns201WithLocation()
{
    var response = await _client.PostAsJsonAsync("/api/products",
        new { name = "Mouse", price = 25.00m });

    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    Assert.NotNull(response.Headers.Location);
}
```

#### General Syntax (Authentication Mocking)

```csharp
builder.ConfigureTestServices(services =>
{
    services.AddAuthentication("TestScheme")
        .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>(
            "TestScheme", _ => { });
});

// TestAuthHandler returns a ClaimsPrincipal with configurable claims
public class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var claims = new[] { new Claim(ClaimTypes.Name, "TestUser") };
        var identity = new ClaimsIdentity(claims, "TestScheme");
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, "TestScheme");
        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

**Syntax rules:**

- **`HttpClient` sends real HTTP requests** —status codes and headers are real.
- **`ReadFromJsonAsync<T>()`** deserializes the response body.
- **Authentication mocking** replaces the real authentication handler with a test handler that returns a known user.
- **Redirects should be controlled** when testing authentication —use `WebApplicationFactoryClientOptions` to disable auto‑redirect.

**Constraints and limitations:**

- **`HttpClient` follows redirects by default** —disable with `AllowAutoRedirect = false` when testing 401 → login redirects.
- **Test authentication handlers bypass real credentials** —they test authorization, not authentication mechanics.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class ProductsApiTests : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public ProductsApiTests(CustomWebApplicationFactory<Program> factory)
        => _client = factory.CreateClient(new WebApplicationFactoryClientOptions
        {
            AllowAutoRedirect = false   // Control redirects for auth tests
        });

    [Fact]
    public async Task GetProducts_Returns200WithJson()
    {
        var response = await _client.GetAsync("/api/products");
        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
        var products = await response.Content.ReadFromJsonAsync<List<ProductDto>>();
        Assert.NotEmpty(products!);
    }

    [Fact]
    public async Task CreateProduct_InvalidInput_Returns400()
    {
        var response = await _client.PostAsJsonAsync("/api/products",
            new { name = "", price = -1m });   // invalid
        Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
    }

    [Fact]
    public async Task SecureEndpoint_Unauthenticated_Returns401()
    {
        var response = await _client.GetAsync("/api/secure");
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }
}
```

**Expected Output:** The GET returns 200 with JSON; the invalid POST returns 400; the unauthenticated request to a secure endpoint returns 401.

**Why this output?** Each test exercises a different aspect of the HTTP contract —status codes, validation, and authentication.

### Real-World Cases

- **API contracts:** Verify status codes, content types, and JSON shape.
- **Authentication/authorization:** 401 vs. 403 in the real pipeline.
- **Validation:** `400 Bad Request` with `ProblemDetails`.
- **CRUD flows:** Create → read → update → delete through HTTP.

### References

- Integration tests in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Mock authentication in integration tests – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#mock-authentication
- Test Minimal API apps – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/test-min-api

---

## 6. Integration Testing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Integration testing best practices are guidelines for writing tests that are reliable, isolated, maintainable, and genuinely verify the application’s wiring. |
| **Technical Definition** | Best practices span **database isolation** (Respawn, unique database names, transactions), **test data discipline** (seed only what the test needs), **authentication mocking** (`TestAuthHandler`), **environment configuration** (`Testing`), **factory sharing** (`IClassFixture`), **redirect control** for auth tests, and **CI integration** (`dotnet test` with coverage). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for integration testing—they help you avoid flaky tests, tests that depend on shared state, and test suites that are so slow or unreliable that nobody trusts them. |

### Purposes

- **To** ensure **test isolation** —each test starts from a known state.
- **To** keep tests **fast** —share containers and factories, avoid unnecessary work.
- **To** **avoid flakiness** —control redirects, seed data deliberately, avoid shared mutable state.
- **To** **mock authentication** without bypassing the authorization pipeline.
- **To** **integrate with CI** —run `dotnet test` on every commit.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Database isolation** | Use **Respawn** or unique database names per test; reset state between tests. | Share a dirty database across tests —order‑dependent tests fail. |
| **Test data** | Seed only the records the test needs. | Seed a large shared fixture that makes tests hard to understand. |
| **Factory sharing** | Use `IClassFixture<CustomWebApplicationFactory<Program>>` to share the factory. | Re‑bootstrap the app for every test method —slow. |
| **Authentication** | Use a **`TestAuthHandler`** to mock authentication and test authorization. | Hit the real identity provider in tests —slow and fragile. |
| **Redirects** | Set `AllowAutoRedirect = false` when testing 401/403 and redirects. | Let `HttpClient` follow redirects blindly —auth assertions become meaningless. |
| **Environment** | Set `builder.UseEnvironment("Testing")` to load test‑specific configuration. | Run tests in `Production` —they may hit real services. |
| **Database choice** | Use **Testcontainers** for production fidelity; SQLite for relational speed. | Use EF Core InMemory for tests that depend on relational constraints. |
| **Antiforgery** | Handle antiforgery tokens for form POST tests. | Ignore antiforgery —form posts fail with 400. |
| **Coverage** | Run `dotnet test` with coverage in CI. | Skip integration tests because “they’re slow” —they catch the most expensive bugs. |
| **Scope** | Test critical paths —auth, CRUD, middleware, serialization. | Test every permutation —unit tests should cover logic variations. |

### Annotated Code Example (Best‑Practice Setup)

```csharp
// Factory with Testcontainers + Respawn + TestAuthHandler
public class IntegrationTestWebAppFactory : WebApplicationFactory<Program>, IAsyncLifetime
{
    private readonly PostgreSqlContainer _dbContainer = new PostgreSqlBuilder()
        .WithImage("postgres:15-alpine").Build();

    private Respawner _respawner = null!;

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureTestServices(services =>
        {
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
            if (descriptor is not null) services.Remove(descriptor);

            services.AddDbContext<AppDbContext>(options =>
                options.UseNpgsql(_dbContainer.GetConnectionString()));

            // Mock authentication
            services.AddAuthentication("TestScheme")
                .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>(
                    "TestScheme", _ => { });
        });
        builder.UseEnvironment("Testing");
    }

    public async Task InitializeAsync()
    {
        await _dbContainer.StartAsync();
        // ... apply migrations, create Respawner checkpoint ...
    }

    public new async Task DisposeAsync() => await _dbContainer.DisposeAsync();
}

// Test class
public class OrdersApiTests : IClassFixture<IntegrationTestWebAppFactory>
{
    private readonly HttpClient _client;
    public OrdersApiTests(IntegrationTestWebAppFactory factory)
        => _client = factory.CreateClient(new WebApplicationFactoryClientOptions
        {
            AllowAutoRedirect = false
        });

    [Fact]
    public async Task CreateOrder_Authenticated_Returns201()
    {
        var response = await _client.PostAsJsonAsync("/api/orders",
            new { customerId = 1, total = 99.99m });

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

**Why this follows best practices:**

- **Testcontainers** provides a real PostgreSQL database for production fidelity.
- **`TestAuthHandler`** mocks authentication without bypassing authorization.
- **`AllowAutoRedirect = false`** ensures 401/403 assertions are meaningful.
- **`IClassFixture`** shares the factory across tests, avoiding re‑bootstrap.
- **`UseEnvironment("Testing")`** loads test‑specific configuration.
- **Respawn** (not shown in full) resets the database between tests for isolation.

### Real-World Cases

- **Production‑grade CI pipelines:** Testcontainers + Respawn + factory sharing.
- **Fast local development:** SQLite in‑memory for quick feedback; Testcontainers for pre‑merge validation.
- **Authentication‑heavy APIs:** TestAuthHandler to test policies without a real IdP.

### References

- Integration tests in ASP.NET Core (best practices) – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Do you use Testcontainers for integration testing? – https://www.ssw.com.au/rules/use-testcontainers
- Respawn (GitHub) – https://github.com/jbogard/Respawn
- Testing, Performance, And Operations (GitHub) – https://github.com/midudev/autoskills/blob/2dbab03b41988327e5c714d3fb233ebbce915e8c/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md

---

## Summary Table: Integration Testing at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Integration testing** | Verify full HTTP pipeline | `WebApplicationFactory<Program>`, `HttpClient` | API endpoints, middleware, DI, auth |
| **`WebApplicationFactory<T>`** | Bootstrap the app in‑process | `CreateClient()`, `ConfigureWebHost` | Every integration test |
| **`TestServer`** | In‑memory server | Created by the factory; `UseTestServer` | Middleware isolation, lightweight pipeline tests |
| **Database testing** | Verify data access | InMemory, SQLite, Testcontainers, Respawn | EF Core queries, migrations, constraints |
| **API testing** | Verify HTTP contract | Status codes, content types, JSON, auth | Every endpoint |
| **Best practices** | Reliable, isolated tests | Respawn, TestAuthHandler, `IClassFixture`, `AllowAutoRedirect = false` | Every integration test suite |

---

## References (Full List)

- Integration tests in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests
- Basic tests with the default `WebApplicationFactory` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#basic-tests-with-the-default-webapplicationfactory
- Customize `WebApplicationFactory` – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#customize-webapplicationfactory
- Mock authentication in integration tests – https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests#mock-authentication
- `TestServer` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.testhost.testserver
- Test middleware – https://learn.microsoft.com/en-us/aspnet/core/test/middleware
- Test Minimal API apps – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/test-min-api
- Testing an ASP.NET Core web app with Testcontainers – https://docs.docker.com/guides/testcontainers-dotnet-aspnet-core/
- Do you use Testcontainers for integration testing? (SSW Rules) – https://www.ssw.com.au/rules/use-testcontainers
- Respawn (GitHub) – https://github.com/jbogard/Respawn
- EF Core InMemory provider – https://learn.microsoft.com/en-us/ef/core/providers/in-memory/
- ASP.NET Core Integration Testing (GitHub) – https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/csharp/testing/integration.md
- How to Set Up Integration Testing in ASP.NET Core (OneUptime) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-set-up-integration-testing-aspnet-core/README.md
- dotnet-integration-testing SKILL.md (GitHub) – https://github.com/ronnythedev/dotnet-clean-architecture-skills/blob/main/skills/22-dotnet-integration-testing/SKILL.md
- Testing, Performance, And Operations (GitHub) – https://github.com/midudev/autoskills/blob/2dbab03b41988327e5c714d3fb233ebbce915e8c/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- Guided: Testing ASP.NET Core 10 Web APIs (Pluralsight) – https://www.pluralsight.com/labs/codeLabs/guided-testing-aspnet-core-10-web-apis