# C# Minimal APIs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Minimal APIs** are a lightweight, low-ceremony way to build HTTP APIs in ASP.NET Core using top-level statements and lambda expressions, without controllers, inheritance, or attribute-based routing. They map HTTP verbs directly to route handlers in `Program.cs`, minimising framework overhead and boilerplate while preserving full access to the ASP.NET Core feature set.

**Technical Definition**
Minimal APIs are built on the ASP.NET Core **endpoint routing** infrastructure. Route handlers are registered via `Map*` extension methods (`MapGet`, `MapPost`, `MapPut`, `MapDelete`, etc.) on `WebApplication`. Parameters are populated by **parameter binding**, which automatically maps request data from route values, query strings, headers, request bodies, and dependency injection services. Return values are converted to HTTP responses by **result execution**; handlers can return `string`, arbitrary objects (JSON-serialised), `IResult`, or `TypedResults`. Minimal APIs support **route constraints**, **OpenAPI metadata**, **endpoint filters**, and **`[AsParameters]`** for grouping parameters.

**Beginner-Friendly Explanation**
Minimal APIs are the **“no-frills” way to build a web API in C#**. Instead of creating a controller class and decorating it with attributes, you write a single line like `app.MapGet("/hello", () => "Hello World");`. That is it—no class, no `[HttpGet]`, no `ControllerBase`. ASP.NET Core handles the rest: it listens for the request, runs your lambda, and sends the result back. For small APIs, microservices, and prototypes, Minimal APIs are faster to write and easier to read than traditional MVC controllers.

---

### Key Characteristics

- **Low ceremony** — No controllers, no inheritance, no attribute routing required.
- **Lambda-based handlers** — Route handlers are lambdas or local functions.
- **Automatic parameter binding** — Route values, query strings, headers, bodies, and DI services are bound automatically.
- **`Map*` extension methods** — `MapGet`, `MapPost`, `MapPut`, `MapDelete`, `MapPatch`, `MapMethods`.
- **Rich return types** — Strings, JSON objects, `IResult`, `TypedResults`, `Task<T>`, `ValueTask<T>`.
- **Endpoint filters** — Cross-cutting concerns via `IEndpointFilter`.
- **OpenAPI metadata** — Built-in support for endpoint descriptions.
- **Route constraints** — `{id:int}`, `{name:alpha}`, `{price:decimal}`, etc.

---

### Prerequisites

- Basic understanding of **ASP.NET Core fundamentals** (`Program.cs`, `WebApplicationBuilder`).
- Familiarity with **HTTP** (verbs, status codes, headers, bodies).
- Knowledge of **lambda expressions** and **dependency injection**.
- Awareness of **`IResult`** and JSON serialisation (`System.Text.Json`).

---

### Related Programming Areas

- **REST API development** — Minimal APIs are the fastest way to build RESTful endpoints in .NET.
- **Microservices** — Lightweight footprint makes them ideal for containerised services.
- **Prototyping** — Quick iteration without controller scaffolding.
- **OpenAPI/Swagger** — Built-in metadata support for API documentation.
- **Endpoint filters** — Cross-cutting validation, logging, and authorization.

---

### Core Concepts / Features

1. Minimal APIs (Overarching Concept)
2. `MapGet()`
3. `MapPost()`
4. `MapPut()`
5. `MapDelete()`
6. Route Parameters
7. Query Parameters
8. Request Body
9. Response
10. Minimal API Best Practices

---

## 1. Minimal APIs (Overarching Concept)

### Definitions

**Core Definition**
Minimal APIs are a streamlined ASP.NET Core programming model that maps HTTP requests directly to route handlers using minimal code.

**Technical Definition**
Minimal APIs are built on **endpoint routing**. `WebApplication` implements both `IEndpointRouteBuilder` and `IApplicationBuilder`. `Map*` methods register endpoints with the routing middleware. Parameter binding converts request data into strongly typed handler parameters. Return values are processed by result execution: strings are written as `text/plain`, objects are JSON-serialised, and `IResult` objects control the response explicitly.

**Beginner-Friendly Explanation**
Minimal APIs let you write **web endpoints as one-liners**. You call `app.MapGet("/", () => "Hello");` and the framework handles everything else.

---

### Purposes

- To **reduce boilerplate** for simple APIs and microservices.
- To **improve startup performance** (fewer types, less reflection).
- To make API code **easier to read and maintain** for small surfaces.
- To **leverage top-level statements** and modern C# features.
- To **support OpenAPI** and endpoint filters without controllers.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

**Component breakdown**

- `WebApplication.CreateBuilder(args)` — Creates the builder.
- `builder.Build()` — Builds the app.
- `app.MapGet(pattern, handler)` — Registers a GET endpoint.
- `app.Run()` — Starts the server (Kestrel by default).

#### Syntax Rules

- Handlers can be lambdas, local functions, or methods.
- Handler parameters are bound automatically from request data.
- Return values can be `string`, `T` (JSON), `IResult`, `Task<T>`, or `ValueTask<T>`.
- Endpoints are registered after `builder.Build()`.

#### Constraints and Limitations

- Minimal APIs are optimised for small surfaces; large APIs may benefit from controllers.
- Some advanced MVC features (model validation filters, `ApiController` behaviours) are not available by default.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Minimal API

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

**Expected Output (HTTP response)**
```
Hello, World!
```

**Why this output occurs**
`MapGet` registers a handler for `GET /`. Kestrel listens and returns the string directly with `Content-Type: text/plain`.

---

### Real-World Cases

- **Health-check endpoints** — `/health` returning `"OK"`.
- **Webhooks** — Small endpoints that receive and acknowledge events.
- **Microservices** — Lightweight APIs with minimal dependencies.

---

### References

- Minimal APIs quick reference — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Minimal APIs overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/overview

---

## 2. `MapGet()`

### Definitions

**Core Definition**
`MapGet()` registers an endpoint that handles HTTP `GET` requests for a specified route pattern.

**Technical Definition**
`MapGet(IEndpointRouteBuilder, string pattern, RequestDelegate handler)` adds a route endpoint to the endpoint data source. The handler is invoked when an incoming `GET` request matches the pattern. Parameters are bound from route values, query strings, and DI services (not from the request body by default, because `GET` requests are not expected to have a body).

**Beginner-Friendly Explanation**
`MapGet` means **“when someone asks for this URL with GET, run this code.”**

---

### Purposes

- To **read data** without modifying server state.
- To expose **query endpoints** (list, search, fetch by ID).
- To provide **health checks** and **metadata** endpoints.
- To serve as the default verb for browser navigation and REST reads.

---

### Syntax Rules and Structure

```csharp
app.MapGet("/resource", () => "Hello");
app.MapGet("/resource/{id}", (int id) => $"Item {id}");
app.MapGet("/search", (string q) => $"Searching for {q}");
```

**Component breakdown**

- `pattern` — The route template.
- `handler` — The lambda or method that produces the response.

#### Syntax Rules

- `GET` handlers do not bind the request body implicitly.
- Route and query parameters are bound automatically.
- The return value is written to the response.

#### Constraints and Limitations

- Avoid side effects in `GET` handlers (REST semantics).
- Large result sets should be paginated.

---

### Annotated Code Examples and Expected Outputs

#### Example: `MapGet` with Route and Query Parameters

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products/{id:int}", (int id) => $"Product {id}");
app.MapGet("/products", (string? category) =>
    category is null ? "All products" : $"Products in {category}");

app.Run();
```

**Expected Output**

- `GET /products/42` → `Product 42`
- `GET /products?category=books` → `Products in books`
- `GET /products` → `All products`

**Why this output occurs**
Route values bind to `id`; query strings bind to `category`. The `int` constraint ensures `id` is numeric.

---

### Real-World Cases

- **REST reads** — `GET /api/orders/{id}`.
- **Search** — `GET /api/products?query=...`.
- **Health checks** — `GET /health`.

---

### References

- `MapGet` — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 3. `MapPost()`

### Definitions

**Core Definition**
`MapPost()` registers an endpoint that handles HTTP `POST` requests, typically used to create new resources.

**Technical Definition**
`MapPost` adds a route endpoint for the `POST` verb. Parameters of complex types are bound from the **request body as JSON** by default; simple types are bound from route or query strings. The handler usually returns `201 Created` with a `Location` header.

**Beginner-Friendly Explanation**
`MapPost` means **“when someone sends data to this URL with POST, create something.”**

---

### Purposes

- To **create resources** (REST `POST`).
- To **accept data** from clients (JSON payloads).
- To **trigger actions** that are not idempotent.
- To return **`201 Created`** with the new resource’s location.

---

### Syntax Rules and Structure

```csharp
app.MapPost("/resource", (CreateDto dto) => { /* create */ return Results.Created(...); });
```

**Component breakdown**

- Complex parameter — Bound from JSON body.
- Simple parameter — Bound from route/query.
- Use `[FromBody]` explicitly if binding is ambiguous.

#### Syntax Rules

- `POST` binds complex types from the body by default.
- `GET`, `HEAD`, `OPTIONS`, `DELETE` do **not** bind from the body implicitly; use `[FromBody]` or read `HttpRequest` manually.
- Return `TypedResults.Created` or `Results.Created` for 201 responses.

#### Constraints and Limitations

- Malformed JSON results in `400 Bad Request`.
- Non-nullable body parameters are required; use nullable types for optional bodies.

---

### Annotated Code Examples and Expected Outputs

#### Example: `MapPost` with JSON Body

```csharp
public record CreateProductDto(string Name, decimal Price);

app.MapPost("/products", (CreateProductDto dto) =>
{
    var id = Random.Shared.Next(1, 1000);
    return TypedResults.Created($"/products/{id}", new { id, dto.Name, dto.Price });
});
```

**Expected Output (201 Created)**
```
Location: /products/123
{ "id": 123, "name": "Widget", "price": 9.99 }
```

**Why this output occurs**
The JSON body is deserialised into `CreateProductDto`; `TypedResults.Created` sets the status and `Location` header.

---

### Real-World Cases

- **Create order** — `POST /api/orders`.
- **Register user** — `POST /api/users`.
- **Submit form** — `POST /api/feedback`.

---

### References

- `MapPost` — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 4. `MapPut()`

### Definitions

**Core Definition**
`MapPut()` registers an endpoint that handles HTTP `PUT` requests, typically used to update or replace an existing resource.

**Technical Definition**
`MapPut` adds a route endpoint for the `PUT` verb. Like `POST`, complex types are bound from the request body as JSON by default. The handler usually returns `204 No Content` on success or `404 Not Found` if the resource does not exist.

**Beginner-Friendly Explanation**
`MapPut` means **“replace this resource with the data I am sending.”**

---

### Purposes

- To **update or replace** existing resources.
- To support **idempotent** updates (calling `PUT` multiple times has the same effect).
- To return `204 No Content` or `200 OK` with the updated resource.

---

### Syntax Rules and Structure

```csharp
app.MapPut("/resource/{id}", (int id, UpdateDto dto) => { /* update */ });
```

**Component breakdown**

- `id` — Route parameter.
- `dto` — Body parameter (JSON).
- Return `TypedResults.NoContent()` or `TypedResults.Ok(...)`.

#### Syntax Rules

- `PUT` binds complex types from the body by default.
- Route parameters are bound from the URL.
- Use `[FromBody]` explicitly if needed.

#### Constraints and Limitations

- If the resource does not exist, return `404 Not Found`.
- `PUT` should be idempotent.

---

### Annotated Code Examples and Expected Outputs

#### Example: `MapPut` with `AsParameters`

```csharp
public record UpdateProductRequest(int Id, UpdateProductDto Dto);

app.MapPut("/products/{id}", ([AsParameters] UpdateProductRequest request) =>
{
    // update logic
    return TypedResults.NoContent();
});
```

**Expected Output**
```
204 No Content
```

**Why this output occurs**
`[AsParameters]` groups route and body parameters into a single record, simplifying the handler signature.

---

### Real-World Cases

- **Update profile** — `PUT /api/users/{id}`.
- **Replace document** — `PUT /api/documents/{id}`.
- **Inventory update** — `PUT /api/inventory/{sku}`.

---

### References

- `MapPut` — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 5. `MapDelete()`

### Definitions

**Core Definition**
`MapDelete()` registers an endpoint that handles HTTP `DELETE` requests, typically used to remove a resource.

**Technical Definition**
`MapDelete` adds a route endpoint for the `DELETE` verb. Like `GET`, `DELETE` does **not** bind the request body implicitly; parameters come from route values, query strings, or DI. The handler usually returns `204 No Content` or `404 Not Found`.

**Beginner-Friendly Explanation**
`MapDelete` means **“remove this resource.”**

---

### Purposes

- To **delete resources**.
- To return `204 No Content` on success.
- To support idempotent deletion (deleting a non-existent resource may return `404` or `204`, depending on design).

---

### Syntax Rules and Structure

```csharp
app.MapDelete("/resource/{id}", (int id) => { /* delete */ return TypedResults.NoContent(); });
```

**Component breakdown**

- `id` — Route parameter.
- Return `TypedResults.NoContent()` or `TypedResults.NotFound()`.

#### Syntax Rules

- `DELETE` does not bind the body implicitly.
- Use route parameters or query strings.
- `[FromBody]` can be used explicitly if a body is required (uncommon).

#### Constraints and Limitations

- `DELETE` is expected to be idempotent.
- Return appropriate status codes (`204`, `404`, `409` if conflict).

---

### Annotated Code Examples and Expected Outputs

#### Example: `MapDelete` with `AsParameters`

```csharp
app.MapDelete("/products/{id}", ([AsParameters] TodoItemRequest request) =>
{
    if (await request.Db.Products.FindAsync(request.Id) is Product p)
    {
        request.Db.Products.Remove(p);
        await request.Db.SaveChangesAsync();
        return TypedResults.NoContent();
    }
    return TypedResults.NotFound();
});
```

**Expected Output**
```
204 No Content (if found) or 404 Not Found
```

**Why this output occurs**
`[AsParameters]` combines the route `id` and the injected `DbContext` into a single request object.

---

### Real-World Cases

- **Delete user** — `DELETE /api/users/{id}`.
- **Remove item from cart** — `DELETE /api/carts/{cartId}/items/{itemId}`.
- **Cancel subscription** — `DELETE /api/subscriptions/{id}`.

---

### References

- `MapDelete` — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 6. Route Parameters

### Definitions

**Core Definition**
Route parameters are segments of the URL path that are captured and bound to handler parameters.

**Technical Definition**
Route parameters are defined in the route template using `{name}` or `{name:constraint}`. Constraints (`int`, `alpha`, `datetime`, `decimal`, `guid`, `minlength(n)`, `regex(...)`, etc.) validate the segment before the handler runs. If validation fails, the framework returns `404 Not Found` without invoking the handler. Parameters are bound by name or by type.

**Beginner-Friendly Explanation**
Route parameters are the **variable parts of the URL**. `GET /products/42` captures `42` as the `id` parameter.

---

### Purposes

- To **identify resources** by ID or slug.
- To **validate input** via constraints.
- To keep URLs **clean and RESTful**.
- To bind values directly to handler parameters.

---

### Syntax Rules and Structure

```csharp
app.MapGet("/products/{id:int}", (int id) => $"Product {id}");
app.MapGet("/users/{username:alpha}", (string username) => $"User {username}");
app.MapGet("/orders/{date:datetime}", (DateTime date) => $"Orders on {date}");
```

**Component breakdown**

- `{id:int}` — Route parameter with an `int` constraint.
- `{username:alpha}` — Alphabetic constraint.
- `{date:datetime}` — DateTime constraint.

#### Syntax Rules

- Constraints are optional but recommended.
- A failed constraint returns `404`, not `400`.
- Parameters bind by name; type must be convertible from string.

#### Constraints and Limitations

- Cannot bind complex types from route parameters directly (use `TryParse` for custom types).
- Route parameters are required unless made optional with `?` or default values.

---

### Annotated Code Examples and Expected Outputs

#### Example: Route Constraints

```csharp
app.MapGet("/products/{id:int}", (int id) => $"Product {id}");
app.MapGet("/products/{name:alpha}", (string name) => $"Product {name}");
```

**Expected Output**

- `GET /products/42` → `Product 42`
- `GET /products/widget` → `Product widget`
- `GET /products/abc123` → `404 Not Found` (fails `int` and `alpha`)

**Why this output occurs**
Constraints filter requests; only matching requests reach the handler.

---

### Real-World Cases

- **REST IDs** — `GET /api/orders/{id:guid}`.
- **Slugs** — `GET /blog/{slug}`.
- **Dates** — `GET /reports/{year:int}/{month:int}`.

---

### References

- Route parameters — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Route constraints — ASP.NET Core routing — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints

---

## 7. Query Parameters

### Definitions

**Core Definition**
Query parameters are key-value pairs appended to the URL after `?` that are bound to handler parameters.

**Technical Definition**
Query string values are bound automatically to handler parameters whose names match the query keys (case-insensitive). Optional parameters can be nullable or have default values. If a required query parameter is missing, the framework throws `BadHttpRequestException` (resulting in `400 Bad Request`).

**Beginner-Friendly Explanation**
Query parameters are the **`?key=value` part of the URL**. `GET /products?category=books` binds `"books"` to a `category` parameter.

---

### Purposes

- To **filter, sort, and paginate** results.
- To pass **optional parameters** without changing the route.
- To support **search queries**.
- To avoid overloading route templates.

---

### Syntax Rules and Structure

```csharp
app.MapGet("/products", (string? category, int page = 1) =>
    $"Category: {category ?? "all"}, Page: {page}");
```

**Component breakdown**

- `string? category` — Optional query parameter.
- `int page = 1` — Default value.

#### Syntax Rules

- Query parameters bind by name.
- Nullable types and default values make parameters optional.
- Required query parameters cause `400` if missing.
- `[FromQuery]` can be used explicitly.

#### Constraints and Limitations

- Complex objects can be bound from query strings using `[AsParameters]` or custom `TryParse`.
- Query strings have length limits (browser/server dependent).

---

### Annotated Code Examples and Expected Outputs

#### Example: Optional Query Parameters

```csharp
app.MapGet("/products", (int? pageNumber) =>
    $"Requesting page {pageNumber ?? 1}");
```

**Expected Output**

- `GET /products?pageNumber=3` → `Requesting page 3`
- `GET /products` → `Requesting page 1`
- `GET /products?pageNumber=abc` → `400 Bad Request` (binding failure)

**Why this output occurs**
`int?` makes the parameter optional; invalid values fail binding.

---

### Real-World Cases

- **Pagination** — `?page=2&pageSize=20`.
- **Search** — `?q=dotnet`.
- **Sorting** — `?sort=price&order=desc`.

---

### References

- Query parameters — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Parameter binding in Minimal API apps — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/parameter-binding

---

## 8. Request Body

### Definitions

**Core Definition**
The request body is the payload sent by the client (typically JSON) that is bound to complex handler parameters.

**Technical Definition**
For `POST`, `PUT`, and `PATCH`, complex types are bound from the JSON body by default using `System.Text.Json`. For `GET`, `DELETE`, `HEAD`, and `OPTIONS`, body binding is **not implicit**; use `[FromBody]` or read `HttpRequest` manually. A nullable complex parameter makes the body optional; a non-nullable one makes it required.

**Beginner-Friendly Explanation**
The request body is **the data the client sends**, usually as JSON. Minimal APIs deserialise it into your C# object automatically for `POST` and `PUT`.

---

### Purposes

- To **accept structured data** from clients.
- To create or update resources.
- To support **complex payloads** (nested objects, arrays).
- To use `System.Text.Json` for high-performance serialisation.

---

### Syntax Rules and Structure

```csharp
app.MapPost("/products", (ProductDto dto) => { /* dto from body */ });
app.MapGet("/search", ([FromBody] SearchCriteria criteria) => { /* explicit */ });
```

**Component breakdown**

- Complex parameter — Bound from JSON body for `POST`/`PUT`.
- `[FromBody]` — Forces body binding for other verbs.
- Nullable parameter — Optional body.

#### Syntax Rules

- Body binding uses `System.Text.Json` by default.
- Malformed JSON → `400 Bad Request`.
- Missing required body → `400 Bad Request` or `400`/`415` depending on configuration.
- Use `HttpRequest` for streaming or custom deserialisation.

#### Constraints and Limitations

- `GET`/`DELETE` do not bind the body implicitly.
- Large bodies should be streamed or limited.
- Custom `TryParse`/`BindAsync` can bind custom types.

---

### Annotated Code Examples and Expected Outputs

#### Example: Optional Body with Nullable Parameter

```csharp
app.MapPost("/register", (Person? person) =>
{
    if (person is null) return TypedResults.BadRequest("No data");
    return TypedResults.Ok(person.Name);
});
```

**Expected Output**

- Empty body → `400 Bad Request` (if non-nullable) or handled as `null` (if nullable).
- Valid JSON `{ "Name": "Samson" }` → `200 OK` with `"Samson"`.

**Why this output occurs**
A nullable `Person?` allows the handler to run with `null` when no body is sent; a non-nullable parameter would fail binding.

---

### Real-World Cases

- **Create/update resources** — JSON payloads for `POST`/`PUT`.
- **Webhooks** — Receiving event payloads.
- **File uploads** — `IFormFile` (form binding, not JSON).

---

### References

- Request body binding — Minimal APIs quick reference — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Parameter binding in Minimal API apps — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/parameter-binding

---

## 9. Response

### Definitions

**Core Definition**
The response is the HTTP output produced by a route handler, including the status code, headers, and body.

**Technical Definition**
Minimal API handlers can return:
- **`string`** — Written as `text/plain`.
- **`T` (any other type)** — JSON-serialised as `application/json`.
- **`IResult`** — Fully controls the response; `Results` and `TypedResults` provide helpers.
- **`Task<T>` / `ValueTask<T>`** — Asynchronous equivalents.
`TypedResults` is preferred over `Results` because it returns strongly typed objects, improving readability, testability, and OpenAPI metadata.

**Beginner-Friendly Explanation**
The response is **what the API sends back**. You can return a plain string, an object (converted to JSON automatically), or an `IResult` for full control (status codes, headers, redirects, files, etc.).

---

### Purposes

- To return **data** (JSON or text).
- To set **status codes** (`200`, `201`, `204`, `400`, `404`, etc.).
- To add **headers** and **cookies**.
- To support **streaming**, **files**, and **redirects**.
- To provide **OpenAPI metadata** via typed results.

---

### Syntax Rules and Structure

```csharp
// String
app.MapGet("/hello", () => "Hello");

// JSON
app.MapGet("/json", () => new { Message = "Hello" });

// TypedResults
app.MapGet("/typed", () => TypedResults.Ok(new Message { Text = "Hello" }));

// Results
app.MapGet("/result", () => Results.NotFound());

// Custom status
app.MapGet("/teapot", () => Results.StatusCode(418));
```

**Component breakdown**

- `string` → `text/plain`.
- `T` → `application/json`.
- `TypedResults` → Strongly typed `IResult` implementations.
- `Results` → `IResult` helpers.

#### Syntax Rules

- `TypedResults` is preferred for OpenAPI and unit testing.
- `IResult` gives full control over status, headers, and body.
- `null` returns are handled as `404` or empty depending on return type.

#### Constraints and Limitations

- Returning a raw object serialises with default `System.Text.Json` options.
- `TypedResults` requires .NET 7+.

---

### Annotated Code Examples and Expected Outputs

#### Example: `TypedResults` with Status Codes

```csharp
app.MapGet("/products/{id:int}", (int id) =>
    id > 0
        ? TypedResults.Ok(new { Id = id, Name = "Widget" })
        : TypedResults.BadRequest("Invalid ID"));
```

**Expected Output**

- `GET /products/42` → `200 OK` with JSON `{"id":42,"name":"Widget"}`
- `GET /products/0` → `400 Bad Request` with `"Invalid ID"`

**Why this output occurs**
`TypedResults.Ok` and `TypedResults.BadRequest` produce strongly typed results that control the status code and body.

---

### Real-World Cases

- **REST responses** — `200`, `201`, `204`, `400`, `404`, `500`.
- **File downloads** — `Results.File`.
- **Redirects** — `Results.Redirect`.
- **Problem details** — `Results.Problem` for standardized error responses.

---

### References

- Create responses in Minimal API applications — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses
- `TypedResults` vs. `Results` — Minimal APIs — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses#typedresults-vs-results

---

## 10. Minimal API Best Practices

### Definitions

**Core Definition**
Minimal API best practices are the guidelines for building secure, maintainable, and well-documented Minimal APIs.

**Technical Definition**
Best practices encompass **preferring `TypedResults` over `Results`** for OpenAPI and testability; using **`[AsParameters]`** to group related parameters; applying **route constraints** for validation; **validating inputs** with `TryParse` or `BindAsync`; **organising endpoints** with `MapGroup`; **using endpoint filters** for cross-cutting concerns; and **avoiding anti-patterns** such as using `ConfigureServices` instead of `builder.Services`.

**Beginner-Friendly Explanation**
The best practices are: **use `TypedResults`**, **group parameters with `[AsParameters]`**, **use route constraints**, **validate inputs**, **organise endpoints**, and **use filters for shared logic**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Prefer TypedResults
return TypedResults.Ok(dto);   // not Results.Ok(dto)

// 2. Group parameters with AsParameters
app.MapPost("/todoitems", ([AsParameters] CreateTodoItemRequest request) => { ... });

// 3. Use route constraints
app.MapGet("/products/{id:int}", (int id) => ...);

// 4. Validate inputs with TryParse/BindAsync for custom types
app.MapGet("/map", (Point point) => ...);

// 5. Organise endpoints with MapGroup
var group = app.MapGroup("/api/products");
group.MapGet("/", GetAll);
group.MapGet("/{id}", GetById);

// 6. Use endpoint filters for cross-cutting concerns
app.MapGet("/secure", () => "secret").AddEndpointFilter<AuthFilter>();

// 7. Use builder.Services, not ConfigureServices
builder.Services.AddSingleton<IService, Service>();
```

**Component breakdown**

- **`TypedResults`** — Strong typing, OpenAPI, testability.
- **`[AsParameters]`** — Fewer parameters, reusable request objects.
- **Route constraints** — Early validation, `404` on failure.
- **Custom binding** — `TryParse`/`BindAsync` for custom types.
- **`MapGroup`** — Route prefixes and shared configuration.
- **Endpoint filters** — Reusable cross-cutting logic.
- **`builder.Services`** — The recommended DI registration pattern.

#### Rules and Constraints

- **Do** prefer `TypedResults` over `Results`.
- **Do** use `[AsParameters]` for handlers with many parameters.
- **Do** apply route constraints for type safety.
- **Do** organise endpoints with `MapGroup`.
- **Do** use endpoint filters for validation and logging.
- **Do not** use `ConfigureServices`; use `builder.Services`.
- **Do not** ignore input validation; use binding or filters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Structured Minimal API

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IProductService, ProductService>();

var app = builder.Build();

var products = app.MapGroup("/api/products")
    .WithTags("Products");

products.MapGet("/", (IProductService svc) =>
    TypedResults.Ok(svc.GetAll()));

products.MapGet("/{id:int}", (int id, IProductService svc) =>
    svc.Get(id) is { } p
        ? TypedResults.Ok(p)
        : TypedResults.NotFound());

products.MapPost("/", ([AsParameters] CreateProductRequest req, IProductService svc) =>
{
    var created = svc.Create(req.Dto);
    return TypedResults.Created($"/api/products/{created.Id}", created);
});

app.Run();
```

**Expected Output (conceptual)**
```
// Well-organised endpoints with TypedResults, AsParameters, MapGroup, and DI.
```

**Why this structure is preferred**
It groups related endpoints, uses strong types for responses, and keeps handler signatures clean.

---

### Real-World Cases

- **API grouping** — `/api/v1/products`, `/api/v1/orders`.
- **OpenAPI documentation** — `TypedResults` provides automatic response metadata.
- **Validation filters** — Shared validation logic across endpoints.

---

### References

- Minimal APIs quick reference — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- ASP0012: Prefer `builder.Services` over `ConfigureServices` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/asp0012
- Parameter binding in Minimal API apps — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/parameter-binding
- Create responses in Minimal API applications — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `TypedResults` | Introduced in **.NET 7**; preferred over `Results`. |
| `[AsParameters]` | Introduced in **.NET 7**; groups parameters into a single object. |
| `MapGroup` | Introduced in **.NET 7**; organises endpoints with shared prefixes. |
| Endpoint filters (`IEndpointFilter`) | Introduced in **.NET 7**; cross-cutting concerns. |
| Body binding for `GET`/`DELETE` | **Not implicit**; use `[FromBody]` or read `HttpRequest`. |
| `ConfigureServices` | **Not recommended** in Minimal APIs; use `builder.Services` (ASP0012). |
| `Results` vs. `TypedResults` | `TypedResults` is **strongly recommended** for OpenAPI and testing. |

---

## References (Consolidated)

- Minimal APIs quick reference — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Parameter binding in Minimal API apps — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/parameter-binding
- Create responses in Minimal API applications — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses
- `TypedResults` vs. `Results` — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses#typedresults-vs-results
- ASP0012: Prefer `builder.Services` over `ConfigureServices` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/asp0012
- Route constraints — ASP.NET Core routing — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- Minimal API design documents — .NET 8 — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/minimal-apis
- Binding request data — GitHub — https://github.com
- Matching requests — GitHub — https://github.com

---