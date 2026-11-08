# ASP.NET Core Minimal APIs: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core Minimal APIs** (.NET 6–10). Minimal APIs were introduced in .NET 6 as a lightweight alternative to controller-based APIs. They have matured significantly: **.NET 7** added route groups, filters, and `TypedResults`; **.NET 8** added form binding, `[FromKeyedServices]`, and AOT support; **.NET 9** added built-in OpenAPI; **.NET 10** added validation and OpenAPI 3.1. Minimal APIs are **not a replacement for controllers**—they are a complementary model best suited for small, focused services, microservices, and high-performance endpoints. Controllers remain the recommended choice for large APIs with rich filter pipelines, versioning, and complex model binding. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Minimal APIs are a lightweight way to build HTTP APIs in ASP.NET Core with minimal ceremony—routes are defined directly in `Program.cs` using `MapGet`, `MapPost`, `MapPut`, `MapDelete`, and related methods, without controllers or `[ApiController]`. |
| **Technical Definition** | Minimal APIs map HTTP verbs and route templates to **request delegates** (`Func<...>`) via `IEndpointRouteBuilder` extension methods (`MapGet`, `MapPost`, etc.). Parameters are resolved from route/query/body/DI/headers via **inferred binding**, results are returned as `IResult` or typed results (`TypedResults`), and middleware/filters can be applied per endpoint or per route group. The pipeline is endpoint-routing based—no MVC API explorer or `[ApiController]` conventions are involved. |
| **Beginner-Friendly Explanation** | Minimal APIs are the **"no-ceremony" way to build APIs**. Instead of creating a controller class with attributes, you write a single line: `app.MapGet("/products/{id}", (int id) => ...)`. The framework figures out that `id` comes from the URL, runs your lambda, and serializes the result to JSON. It is fast to write, fast at runtime, and perfect for small services—but for large APIs with dozens of endpoints, complex filters, and versioning, controllers are usually easier to maintain. |

### Key Characteristics

- **Lambda-first** – Endpoints are defined inline in `Program.cs` or extension methods.
- **Inferred binding** – Parameters are resolved from route, query, body, DI, or headers based on type and name.
- **`IResult` returns** – `Results.Ok(...)`, `TypedResults.Ok<T>(...)`, or raw objects (implicitly `200 OK`).
- **Route groups** – `.MapGroup("/api/v1")` organizes and applies shared configuration.
- **Filters** – Endpoint filters (`.AddEndpointFilter(...)`) provide cross-cutting logic.
- **Performance** – Lower overhead than MVC controllers (no filters pipeline, no model binder service lookups).
- **AOT-friendly** – .NET 8+ supports Native AOT with source-generated JSON.

### Prerequisites

- .NET SDK (6.0+ recommended; 8+ for advanced features), ASP.NET Core project.
- Working knowledge of **HTTP**, **routing**, and **dependency injection**.
- Familiarity with **C# lambdas** and **async/await**.
- Basic understanding of **JSON serialization** and **API design**.

### Related Programming Areas

- **Web API fundamentals** – The broader HTTP API architecture.
- **API controllers** – The alternative, controller-based model.
- **Routing** – Endpoint routing and route templates.
- **Dependency injection** – Service resolution in endpoint handlers.
- **OpenAPI** – Documenting minimal API endpoints.
- **Filters** – Endpoint filters for cross-cutting concerns.

### Core Concepts / Features

1. **Minimal APIs** – The overall model, request delegates, and binding.
2. **`MapGet()`** – Define GET endpoints.
3. **`MapPost()`** – Define POST endpoints.
4. **`MapPut()`** – Define PUT endpoints.
5. **`MapDelete()`** – Define DELETE endpoints.
6. **Minimal API best practices** – Organization, validation, filtering, and security.

---

## 1. Minimal APIs (The Model)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Minimal APIs are a low-ceremony approach to building HTTP APIs where endpoints are defined as route mappings to request delegates in code. |
| **Technical Definition** | Minimal APIs use `IEndpointRouteBuilder` extension methods (`MapGet`, `MapPost`, etc.) to register **endpoint delegates**. Parameters are resolved by **parameter binding inference**: route parameters by name, query parameters for simple types, body for complex types (`[FromBody]` inferred), DI services for registered types, and `HttpContext`/`HttpRequest`/`HttpResponse`/`CancellationToken` directly. Results can be returned as `IResult` (via `Results` or `TypedResults`), raw objects (serialized as JSON with `200 OK`), or `Task<T>`. Route groups (`.MapGroup(...)`) apply common prefixes and conventions. |
| **Beginner-Friendly Explanation** | Minimal APIs are **"one-liner endpoints"**. Instead of a controller class with methods and attributes, you write `app.MapGet("/hello", () => "Hello")` and you have an endpoint. The framework looks at your lambda's parameters and figures out where to get the values—from the URL, the query string, the JSON body, or the DI container. It serializes whatever you return to JSON. |

### Purposes

- **To** build small APIs **quickly** with minimal code and configuration.
- **To** support **high-performance** endpoints with lower overhead than controllers.
- **To** enable **Native AOT** and trimming scenarios (with source-generated JSON).
- **To** provide a **simple mental model** for microservices and focused APIs.
- **To** allow **fine-grained endpoint configuration** without controller conventions.

### Syntax Rules and Structure

#### General Syntax (Basic Endpoint)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `app.MapGet` | Maps a GET request to a delegate. |
| `"/"` | The route template. |
| `() => "Hello World!"` | The request delegate; returns a string (implicitly `200 OK`, `text/plain`). |

#### Request Delegate Signature Patterns

```csharp
// No parameters
app.MapGet("/ping", () => "pong");

// Route parameter
app.MapGet("/products/{id:int}", (int id) => $"Product {id}");

// DI service
app.MapGet("/products", (IProductService service) => service.GetAll());

// Body + return typed result
app.MapPost("/products", (Product product) => TypedResults.Created($"/products/{product.Id}", product));

// HttpContext access
app.MapGet("/info", (HttpContext context) => context.Request.Headers.UserAgent.ToString());
```

**Syntax rules:**

- **Simple types** (`int`, `string`, `Guid`, `bool`, `DateTime`) are bound from **route parameters** by name, or from **query string** if not in the route.
- **Complex types** are bound from the **JSON body** by default in POST/PUT/PATCH.
- **Registered services** are resolved from **DI**.
- **`HttpContext`**, `HttpRequest`, `HttpResponse`, `ClaimsPrincipal`, and `CancellationToken` are bound directly.
- **Return values** are serialized as JSON with `200 OK` unless a specific `IResult` or `TypedResults` is returned.

**Constraints and limitations:**

- **No built-in filter pipeline like MVC** —use endpoint filters for cross-cutting concerns.
- **No `[ApiController]` conventions** —automatic 400 responses and binding inference do not apply; you must return `TypedResults.ValidationProblem(...)` explicitly or use `AddValidation()` (.NET 10).
- **OpenAPI metadata is opt-in** —use `.WithName()`, `.WithTags()`, `.WithSummary()`, `.Produces<T>()` for documentation.
- **Large APIs become unwieldy in `Program.cs`** —use extension methods, route groups, or move to controllers when the API grows.

### Annotated Complete Step-by-Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddSingleton<IProductService, InMemoryProductService>();

var app = builder.Build();

// Basic endpoint
app.MapGet("/", () => "Products API v1");

// Route parameter with constraint
app.MapGet("/products/{id:int}", (int id, IProductService service) =>
    service.Find(id) is { } product
        ? Results.Ok(product)
        : Results.NotFound());

// Collection
app.MapGet("/products", (IProductService service) =>
    Results.Ok(service.GetAll()));

app.Run();

public interface IProductService
{
    IEnumerable<Product> GetAll();
    Product? Find(int id);
}

public record Product(int Id, string Name, decimal Price);
```

**Step-by-step explanation:**

1. `IProductService` is registered as a singleton in DI.
2. `MapGet("/")` returns a plain string.
3. `MapGet("/products/{id:int}")` binds `id` from the route and `service` from DI; returns `Results.Ok` or `Results.NotFound`.
4. `MapGet("/products")` returns the collection.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /` | `200 OK`, `text/plain`, "Products API v1" |
| `GET /products/1` | `200 OK` with product JSON, or `404 Not Found` |
| `GET /products` | `200 OK` with JSON array |

**Why this output?** Minimal APIs infer binding sources from parameter types and names; return values are serialized as JSON unless `IResult` is used.

### Real-World Cases

- **Microservices:** Small, focused services with a handful of endpoints.
- **Health checks and internal tools:** `/health`, `/metrics`, `/status` endpoints.
- **High-performance APIs:** AOT-compiled, minimal-overhead endpoints.
- **Prototypes:** Rapid API development before committing to controllers.

### References

- Minimal APIs overview (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/overview
- Minimal APIs quick reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 2. `MapGet()` (Retrieving Resources)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `MapGet()` registers an endpoint that handles HTTP GET requests for a specified route template. |
| **Technical Definition** | `EndpointRouteBuilderExtensions.MapGet(IEndpointRouteBuilder, string, RequestDelegate)` maps GET requests to a delegate. The delegate can accept route/query parameters, DI services, `HttpContext`, and `CancellationToken`. GET endpoints should be **safe** (no side effects) and **idempotent** (repeated requests produce the same result) per HTTP semantics. `MapGet` returns `IEndpointConventionBuilder` for further configuration (`.WithName()`, `.RequireAuthorization()`, etc.). |
| **Beginner-Friendly Explanation** | `MapGet()` is the **"read"** endpoint. When a client asks `GET /products/42`, the framework runs the delegate you attached to that route and sends back the result. GET requests should not change data—they just retrieve it. |

### Purposes

- **To** retrieve **single resources** (`GET /products/{id}`).
- **To** retrieve **collections** (`GET /products`) with optional filtering and pagination.
- **To** expose **read-only** endpoints that are safe and cacheable.
- **To** support **query-string parameters** for filtering and pagination.
- **To** serve **discovery/status** endpoints (`/health`, `/version`).

### Syntax Rules and Structure

#### General Syntax (Basic GET)

```csharp
app.MapGet("/products", () => TypedResults.Ok(new[] { "Laptop", "Mouse" }));
```

#### General Syntax (Route + Query Parameters)

```csharp
app.MapGet("/products/{category}",
    (string category, string? sort, int page = 1) =>
        TypedResults.Ok($"Category: {category}, Sort: {sort}, Page: {page}"));
```

**Component breakdown:**

| Parameter | Source | Notes |
|-----------|--------|-------|
| `category` | **Route** (`{category}`) | Matched by name. |
| `sort` | **Query** (`?sort=name`) | Simple type not in route → query. |
| `page` | **Query** (`?page=2`) | Default value applied if absent. |

#### General Syntax (Asynchronous GET)

```csharp
app.MapGet("/products/{id:int}", async (int id, IProductService service, CancellationToken ct) =>
{
    var product = await service.FindAsync(id, ct);
    return product is null ? Results.NotFound() : Results.Ok(product);
});
```

**Syntax rules:**

- **Route parameters** are matched by name; use constraints (`{id:int}`) to validate.
- **Query parameters** are inferred for simple types not present in the route; use `[FromQuery]` to be explicit.
- **GET endpoints should be safe and idempotent** —avoid side effects.
- **Return `IResult`** (`Results.*` or `TypedResults.*`) for explicit status codes.
- **Async lambdas** are supported and recommended for I/O-bound work.

**Constraints and limitations:**

- **Complex types cannot be bound from query strings** by default—use `[AsParameters]` or a parameter object.
- **GET with a request body is discouraged** —HTTP semantics allow it, but many clients/servers do not support it.
- **No automatic pagination** —implement it yourself or use a helper library.

### Annotated Complete Step-by-Step Code Example

```csharp
// Single resource
app.MapGet("/products/{id:int}", async (int id, IProductService service, CancellationToken ct) =>
{
    var product = await service.FindAsync(id, ct);
    return product is null
        ? Results.NotFound()
        : Results.Ok(product);
})
.WithName("GetProduct")
.Produces<Product>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound);

// Collection with pagination
app.MapGet("/products", async (
    IProductService service,
    int page = 1,
    int pageSize = 20,
    CancellationToken ct = default) =>
{
    var (items, total) = await service.ListAsync(page, pageSize, ct);
    return Results.Ok(new { items, total, page, pageSize });
})
.WithName("ListProducts");
```

**Step-by-step explanation:**

1. `MapGet("/products/{id:int}")` matches `GET /products/42`; `id` is bound from the route, `service` from DI, `ct` from the request aborted token.
2. `Results.NotFound()` and `Results.Ok(product)` return explicit status codes.
3. `.WithName("GetProduct")` names the endpoint for URL generation and OpenAPI.
4. `.Produces<Product>(200)` and `.Produces(404)` document the response types.
5. The collection endpoint binds `page` and `pageSize` from the query string.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /products/1` | `200 OK` with product JSON, or `404 Not Found` |
| `GET /products?page=2&pageSize=10` | `200 OK` with paginated JSON |

**Why this output?** Minimal API binding resolves route and query parameters by name; `Results.*` produces the explicit status codes.

### Real-World Cases

- **Product catalogue:** `GET /products` and `GET /products/{id}`.
- **Search endpoints:** `GET /search?q=laptop&page=2`.
- **Health/status:** `GET /health`, `GET /version`.

### References

- `MapGet` (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapget
- HTTP semantics: safe methods (RFC 9110) – https://www.rfc-editor.org/rfc/rfc9110#name-safe-methods

---

## 3. `MapPost()` (Creating Resources)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `MapPost()` registers an endpoint that handles HTTP POST requests, typically for creating new resources or submitting data. |
| **Technical Definition** | `MapPost(IEndpointRouteBuilder, string, RequestDelegate)` maps POST requests. Complex parameters are bound from the **JSON body** by default; use `[FromBody]`, `[FromForm]`, or `[FromServices]` to override. POST endpoints are **not safe or idempotent**—repeated submissions may create duplicate resources. Successful creation should return `201 Created` with a `Location` header pointing to the new resource (`TypedResults.Created`). |
| **Beginner-Friendly Explanation** | `MapPost()` is the **"create"** endpoint. When a client sends `POST /products` with a JSON body, the framework deserializes the body into your parameter, runs your delegate, and sends back the result—usually `201 Created` with a `Location` header so the client knows where to find the new product. |

### Purposes

- **To** create **new resources** (`POST /products`).
- **To** submit **commands or actions** that have side effects.
- **To** accept **complex request bodies** (JSON, form data, files).
- **To** return **`201 Created` with a `Location` header** for created resources.
- **To** support **file uploads** and form submissions.

### Syntax Rules and Structure

#### General Syntax (Create with Body)

```csharp
app.MapPost("/products", (CreateProductDto input, IProductService service) =>
{
    var product = service.Create(input);
    return TypedResults.Created($"/products/{product.Id}", product);
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CreateProductDto input` | Complex type → inferred `[FromBody]`. |
| `IProductService service` | Registered service → resolved from DI. |
| `TypedResults.Created(...)` | Returns `201 Created` with `Location` header. |

#### General Syntax (Explicit Body Binding)

```csharp
app.MapPost("/products", ([FromBody] ProductDto input) => ...);
```

**Syntax rules:**

- **Complex types** are bound from the JSON body by default in POST.
- **`[FromBody]`** is optional but explicit; use it when inference is ambiguous.
- **Return `TypedResults.Created`** to include the `Location` header; use `Results.Created` for the untyped variant.
- **Body size limits** are enforced by the server; use `[RequestSizeLimit]` or middleware for custom limits.

**Constraints and limitations:**

- **No automatic 400 responses** like `[ApiController]` —validate manually and return `TypedResults.ValidationProblem(...)` or use `AddValidation()` (.NET 10).
- **One body parameter per endpoint** —multiple complex types are not supported from the body.
- **Mass assignment risk** —bind DTOs, never domain entities.

### Annotated Complete Step-by-Step Code Example

```csharp
app.MapPost("/products", async (
    CreateProductDto input,
    IProductService service,
    IValidator<CreateProductDto> validator,
    CancellationToken ct) =>
{
    var validation = await validator.ValidateAsync(input, ct);
    if (!validation.IsValid)
        return Results.ValidationProblem(validation.ToDictionary());

    var product = await service.CreateAsync(input, ct);
    return TypedResults.Created($"/products/{product.Id}", product);
})
.WithName("CreateProduct")
.Accepts<CreateProductDto>("application/json")
.Produces<Product>(StatusCodes.Status201Created)
.ProducesValidationProblem();
```

**Step-by-step explanation:**

1. `input` is bound from the JSON body.
2. `service` and `validator` are resolved from DI.
3. Validation failures return `400 Bad Request` with a `ValidationProblemDetails` body.
4. `TypedResults.Created` returns `201 Created` with the `Location` header.
5. `.Accepts`, `.Produces`, and `.ProducesValidationProblem()` document the endpoint for OpenAPI.

**Expected Output:**

| Request | Response |
|---------|----------|
| `POST /products` with valid JSON | `201 Created` with `Location: /products/42` |
| `POST /products` with invalid JSON | `400 Bad Request` with `ValidationProblemDetails` |

**Why this output?** Complex types are bound from the body; `TypedResults.Created` returns the creation status with location; validation failures return a structured error.

### Real-World Cases

- **CRUD create endpoints:** `POST /products`, `POST /orders`.
- **Command endpoints:** `POST /orders/{id}/cancel`, `POST /payments`.
- **File uploads:** `POST /files` with `IFormFile` binding.

### References

- `MapPost` (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mappost
- `TypedResults.Created` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.typedresults.created

---

## 4. `MapPut()` (Replacing Resources)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `MapPut()` registers an endpoint that handles HTTP PUT requests, typically for replacing an entire resource with new data. |
| **Technical Definition** | `MapPut(IEndpointRouteBuilder, string, RequestDelegate)` maps PUT requests. PUT is **idempotent** —repeating the same request has the same effect as the first. The body typically contains the complete replacement representation. Success is usually `200 OK` (with body) or `204 No Content` (without body); `201 Created` is appropriate if the PUT creates the resource at the client-specified URI. |
| **Beginner-Friendly Explanation** | `MapPut()` is the **"replace"** endpoint. When a client sends `PUT /products/42` with the full product JSON, the framework updates product 42 to match the new data. Unlike PATCH (partial update), PUT replaces the entire resource. It is idempotent—sending the same PUT twice has the same effect. |

### Purposes

- **To** **replace** an existing resource entirely (`PUT /products/42`).
- **To** provide **idempotent** update operations (safe to retry).
- **To** support **client-controlled resource identity** (client specifies the ID in the URL).
- **To** return **`204 No Content`** or **`200 OK`** with the updated representation.

### Syntax Rules and Structure

```csharp
app.MapPut("/products/{id:int}", async (
    int id,
    UpdateProductDto input,
    IProductService service,
    CancellationToken ct) =>
{
    var existing = await service.FindAsync(id, ct);
    if (existing is null) return Results.NotFound();

    await service.UpdateAsync(id, input, ct);
    return Results.NoContent();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `id` | Route parameter. |
| `input` | Complex type → bound from body. |
| `Results.NotFound()` | Returned if the resource does not exist. |
| `Results.NoContent()` | Returns `204 No Content` on success. |

**Syntax rules:**

- **PUT is idempotent** —design the endpoint so repeated calls produce the same state.
- **Return `404 NotFound`** if the resource does not exist (or `201 Created` if PUT is allowed to create).
- **Return `204 No Content`** when the body is not needed; **`200 OK`** when returning the updated resource.

**Constraints and limitations:**

- **No automatic concurrency handling** —use ETags or `[Timestamp]`/`RowVersion` fields manually.
- **Mass assignment risk** —always bind DTOs.
- **PUT replaces the entire resource** —missing fields in the body may be interpreted as null/removal; document clearly.

### Annotated Complete Step-by-Step Code Example

```csharp
app.MapPut("/products/{id:int}", async (
    int id,
    UpdateProductDto input,
    IProductService service,
    CancellationToken ct) =>
{
    var existing = await service.FindAsync(id, ct);
    if (existing is null)
        return Results.NotFound();

    await service.UpdateAsync(existing, input, ct);
    return TypedResults.NoContent();
})
.WithName("UpdateProduct")
.Accepts<UpdateProductDto>("application/json")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound);
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `PUT /products/42` (exists) | `204 No Content` |
| `PUT /products/99` (not found) | `404 Not Found` |

**Why this output?** PUT replaces the full resource; `204 No Content` indicates success without a body; `404` indicates the resource does not exist.

### Real-World Cases

- **CRUD update:** `PUT /products/42` with the full replacement.
- **Idempotent retries:** Clients can safely retry PUT requests after network failures.

### References

- `MapPut` (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapput

---

## 5. `MapDelete()` (Removing Resources)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `MapDelete()` registers an endpoint that handles HTTP DELETE requests, typically for removing resources. |
| **Technical Definition** | `MapDelete(IEndpointRouteBuilder, string, RequestDelegate)` maps DELETE requests. DELETE is **idempotent** —repeated deletes of the same resource produce the same result (the resource is gone). Success returns `204 No Content` (no body) or `200 OK` (with body). `404 Not Found` is appropriate if the resource does not exist, though some APIs return `204` for idempotency. |
| **Beginner-Friendly Explanation** | `MapDelete()` is the **"remove"** endpoint. When a client sends `DELETE /products/42`, the framework runs your delegate to delete product 42. It returns `204 No Content` if successful. If the product does not exist, it can return `404`. Because DELETE is idempotent, calling it twice does not cause errors—the second call simply confirms the resource is gone. |

### Purposes

- **To** remove an existing resource (`DELETE /products/42`).
- **To** provide **idempotent** deletion (safe to retry).
- **To** return **`204 No Content`** on successful deletion.
- **To** support **soft delete** patterns (marking as deleted rather than physical removal).

### Syntax Rules and Structure

```csharp
app.MapDelete("/products/{id:int}", async (
    int id,
    IProductService service,
    CancellationToken ct) =>
{
    var deleted = await service.DeleteAsync(id, ct);
    return deleted ? Results.NoContent() : Results.NotFound();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `id` | Route parameter. |
| `service.DeleteAsync(...)` | Deletes the resource; returns `true` if deleted, `false` if not found. |
| `Results.NoContent()` | Returns `204 No Content` on success. |

**Syntax rules:**

- **DELETE is idempotent** —design for safe retries.
- **Return `204 No Content`** on success; **`404 Not Found`** if the resource does not exist (or `204` for idempotency).
- **Soft delete** (marking as deleted) is a common alternative to physical deletion—document the behaviour.

**Constraints and limitations:**

- **No cascade confirmation** —deleting a resource may delete related data (depends on the database configuration).
- **Authorization must be enforced** —deletion is a high-risk operation; require role/policy.
- **Audit logging** is recommended for compliance.

### Annotated Complete Step-by-Step Code Example

```csharp
app.MapDelete("/products/{id:int}", async (
    int id,
    IProductService service,
    ILogger<Program> logger,
    CancellationToken ct) =>
{
    var deleted = await service.DeleteAsync(id, ct);
    if (!deleted) return Results.NotFound();

    logger.LogInformation("Product {Id} deleted", id);
    return Results.NoContent();
})
.WithName("DeleteProduct")
.RequireAuthorization("AdminOnly")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound);
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `DELETE /products/42` (exists, authorized) | `204 No Content` |
| `DELETE /products/42` (exists, unauthorized) | `403 Forbidden` or `401` |
| `DELETE /products/99` (not found) | `404 Not Found` |

**Why this output?** The endpoint returns `204` on success, `404` if the resource is missing, and authorization enforces the `AdminOnly` policy before the handler runs.

### Real-World Cases

- **CRUD delete:** `DELETE /products/42`.
- **Soft delete:** mark as deleted and return `204`.
- **Cascading deletes:** delete a customer and their orders (handled at the database or service layer).

### References

- `MapDelete` (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapdelete

---

## 6. Minimal API Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Minimal API best practices are guidelines for organizing, securing, documenting, and validating minimal API endpoints so they remain maintainable as the API grows. |
| **Technical Definition** | Best practices span **organization** (route groups, extension methods), **validation** (`AddValidation` in .NET 10, manual validation, or FluentValidation), **error handling** (`ProblemDetails`, `UseExceptionHandler`), **security** (`RequireAuthorization`, fallback policies), **OpenAPI metadata** (`.WithName`, `.WithSummary`, `.Produces`), **typed results** (`TypedResults`), **DTO binding** (mass-assignment protection), **rate limiting**, and **when to switch to controllers**. |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for minimal APIs—they help you avoid a 1000-line `Program.cs`, missed validation, undocumented endpoints, and the temptation to force minimal APIs into scenarios where controllers are better. |

### Purposes

- **To** keep `Program.cs` **organized** as the API grows.
- **To** **validate** input consistently.
- **To** **document** endpoints in OpenAPI.
- **To** **secure** endpoints with authentication, authorization, and rate limiting.
- **To** use **typed results** for better OpenAPI and compile-time checks.
- **To** **know when to use controllers** instead.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Organization** | Use `.MapGroup("/api/v1")` and extension methods (`app.MapProductEndpoints()`). | Put hundreds of endpoints in `Program.cs`. |
| **Validation** | Use `AddValidation()` (.NET 10+), FluentValidation, or manual `TypedResults.ValidationProblem`. | Skip validation—minimal APIs don't auto-validate. |
| **Results** | Use `TypedResults.Ok<T>`, `TypedResults.Created`, `TypedResults.NotFound`. | Use `Results.*` (untyped) when `TypedResults` is available. |
| **OpenAPI** | Use `.WithName()`, `.WithSummary()`, `.WithTags()`, `.Produces<T>()`. | Leave endpoints undocumented. |
| **Binding** | Bind **DTOs**, not domain entities. | Bind EF entities—mass-assignment risk. |
| **Security** | Apply `.RequireAuthorization()`, `.AllowAnonymous()`, and rate limiting. | Leave endpoints unprotected by default. |
| **Error handling** | Use `UseExceptionHandler()` and `ProblemDetails`. | Let unhandled exceptions return 500s with stack traces. |
| **Filters** | Use `.AddEndpointFilter<T>()` for cross-cutting concerns. | Duplicate validation/logging in every endpoint. |
| **Async** | Use `async`/`await` and propagate `CancellationToken`. | Use synchronous I/O. |
| **When to switch** | Move to controllers when the API has many endpoints, complex filters, or versioning. | Force minimal APIs to serve a large, complex API. |

### Annotated Code Example (Best-Practice Organization)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IProductService, InMemoryProductService>();
builder.Services.AddProblemDetails();
builder.Services.AddOpenApi();

var app = builder.Build();

app.UseExceptionHandler();
app.UseStatusCodePages();
app.MapProductEndpoints();     // Extension method organization
app.Run();

// ProductEndpoints.cs
public static class ProductEndpoints
{
    public static IEndpointRouteBuilder MapProductEndpoints(this IEndpointRouteBuilder app)
    {
        var group = app.MapGroup("/api/products")
            .WithTags("Products")
            .RequireAuthorization();

        group.MapGet("/", async (IProductService service, CancellationToken ct) =>
            TypedResults.Ok(await service.ListAsync(ct)))
            .WithName("ListProducts")
            .Produces<IReadOnlyList<Product>>(StatusCodes.Status200OK);

        group.MapGet("/{id:int}", async (int id, IProductService service, CancellationToken ct) =>
            await service.FindAsync(id, ct) is { } product
                ? TypedResults.Ok(product)
                : TypedResults.NotFound())
            .WithName("GetProduct")
            .Produces<Product>(StatusCodes.Status200OK)
            .Produces(StatusCodes.Status404NotFound);

        group.MapPost("/", async (CreateProductDto input, IProductService service, CancellationToken ct) =>
        {
            var product = await service.CreateAsync(input, ct);
            return TypedResults.Created($"/api/products/{product.Id}", product);
        })
        .WithName("CreateProduct")
        .Produces<Product>(StatusCodes.Status201Created);

        return app;
    }
}
```

**Why this follows best practices:**

- **Route group** (`MapGroup`) applies the prefix, tags, and authorization policy once.
- **Extension method** keeps `Program.cs` clean.
- **`TypedResults`** provides compile-time checks and better OpenAPI.
- **`.WithName`** and **`.Produces`** document the endpoints.
- **`CancellationToken`** is propagated for cancellation.
- **DTO binding** prevents mass assignment.
- **`RequireAuthorization`** enforces security; individual endpoints can opt out with `.AllowAnonymous()`.
- **`UseExceptionHandler`** + **`AddProblemDetails`** provide consistent errors.

### Real-World Cases

- **Microservices:** Focused API organized with route groups and extension methods.
- **Prototypes that grow:** Switch to controllers when the API reaches dozens of endpoints.
- **High-performance APIs:** `TypedResults` + AOT + minimal middleware.

### References

- Minimal APIs quick reference (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Endpoint filters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/min-api-filters
- `TypedResults` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.typedresults
- Route groups – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers#route-groups

---

## Summary Table: Minimal API Methods at a Glance

| Method | HTTP Verb | Purpose | Safe? | Idempotent? | Success Status |
|--------|-----------|---------|-------|-------------|----------------|
| **`MapGet`** | GET | Read resources | ✅ Yes | ✅ Yes | 200 (or 404) |
| **`MapPost`** | POST | Create resources | ❌ No | ❌ No | 201 Created + Location |
| **`MapPut`** | PUT | Replace resources | ❌ No | ✅ Yes | 200 or 204 |
| **`MapDelete`** | DELETE | Remove resources | ❌ No | ✅ Yes | 204 or 200 |
| **`MapPatch`** | PATCH | Partial update | ❌ No | ❌ No (by default) | 200 or 204 |

---

## References (Full List)

- Minimal APIs overview (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/overview
- Minimal APIs quick reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- `MapGet` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapget
- `MapPost` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mappost
- `MapPut` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapput
- `MapDelete` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis#mapdelete
- Endpoint filters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/min-api-filters
- Route groups – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers#route-groups
- `TypedResults` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.typedresults
- `TypedResults.Created` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.typedresults.created
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- HTTP semantics (RFC 9110) – https://www.rfc-editor.org/rfc/rfc9110
- Minimal APIs .NET 10 validation (`AddValidation`) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/validation