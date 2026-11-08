# ASP.NET Core API Controllers: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **controller‑based Web API** model. API controllers derive from `ControllerBase` and are decorated with `[ApiController]`. The attribute was introduced in ASP.NET Core 2.1 and has been refined since—behaviours such as binding‑source inference, automatic 400 responses, and `ProblemDetails` are stable. **Minimal APIs** are an alternative for small services; controllers remain the recommended choice for larger APIs with rich routing, filters, versioning, and OpenAPI integration. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An API controller is an ASP.NET Core class that handles HTTP requests and returns data (typically JSON) rather than HTML views, using attribute routing and API‑specific conventions. |
| **Technical Definition** | An API controller is a public class that derives from `ControllerBase`, is decorated with `[ApiController]`, and uses attribute routing (`[Route]`, `[HttpGet]`, etc.) to map actions to URLs. `[ApiController]` enables **binding‑source inference**, **automatic model‑state validation** (returning `400 Bad Request` with `ProblemDetails` when `ModelState` is invalid), **`ProblemDetails` responses** for error status codes, and **attribute‑routing enforcement**. Actions typically return `IActionResult` or `ActionResult<T>`. |
| **Beginner‑Friendly Explanation** | An API controller is the **class that answers API requests**. It is like a regular MVC controller, but it returns **data** (JSON) instead of web pages. The `[ApiController]` attribute turns on helpful defaults—automatic validation, smarter parameter binding, and standard error responses—so you write less boilerplate. The `[Route]` attribute tells ASP.NET Core which URL the controller (and its actions) answer. |

### Key Characteristics

- **`ControllerBase`‑derived** – No view support; optimized for data responses.
- **`[ApiController]` convention** – Automatic 400s, binding inference, and `ProblemDetails`.
- **Attribute‑routed** – Conventional routing is not supported for API controllers.
- **Return‑type flexibility** – `IActionResult`, `ActionResult<T>`, `T`, or typed results.
- **DTO‑friendly** – Designed to bind and return data transfer objects rather than domain entities.
- **OpenAPI‑aware** – Automatically included in OpenAPI documents.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **HTTP**, **routing**, and **model binding**.
- Familiarity with **dependency injection** and **C# attributes**.
- Basic understanding of **REST principles** (resources, HTTP verbs, status codes).

### Related Programming Areas

- **Web API fundamentals** – The broader HTTP API architecture.
- **RESTful design** – Resource modeling and HTTP semantics.
- **Model binding** – Populating action parameters from the request.
- **Validation** – `[ApiController]` automatic 400 responses.
- **OpenAPI** – API documentation generated from controllers.
- **Filters** – Cross‑cutting concerns applied to API controllers.

### Core Concepts / Features

1. **API controllers** – The class concept, structure, and lifecycle.
2. **`[ApiController]`** – The attribute and its conventions.
3. **`[Route]`** – Attribute routing for controllers and actions.
4. **API controller best practices** – Design, security, and maintainability guidelines.

---

## 1. API Controllers (The Class)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An API controller is a class that handles HTTP requests for an API and returns data responses. |
| **Technical Definition** | An API controller is a public class in an ASP.NET Core project that (1) derives from `ControllerBase`, (2) is decorated with `[ApiController]`, (3) uses attribute routing, and (4) exposes action methods that map HTTP verbs and routes to return values. Controllers are discovered by `AddControllers()` and mapped by `MapControllers()`. They are instantiated per request by the `DefaultControllerActivator` (or from DI when `AddControllersAsServices()` is called), and their constructor dependencies are resolved from the DI container. |
| **Beginner‑Friendly Explanation** | An API controller is the **class that handles a family of API requests**. For example, a `ProductsController` handles everything under `/api/products`—listing, fetching, creating, updating, and deleting products. Each public method (an **action**) handles one specific request. The class is instantiated fresh for each request, with its dependencies supplied by the DI container. |

### Purposes

- **To** group related API endpoints in a cohesive class with shared dependencies and routing.
- **To** provide a **familiar, testable structure** for larger APIs (vs. minimal API delegates).
- **To** leverage **filters, model binding, validation, and OpenAPI** in a unified pipeline.
- **To** support **dependency injection** for repositories, services, and loggers.
- **To** enable **fine‑grained routing** through attribute routing at the controller and action levels.

### Syntax Rules and Structure

#### General Syntax (Controller Skeleton)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;

    public ProductsController(IProductService service) => _service = service;

    [HttpGet]
    public async Task<ActionResult<IReadOnlyList<ProductDto>>> GetAll()
        => Ok(await _service.ListAsync());

    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
        => await _service.FindAsync(id) is { } product
            ? Ok(product)
            : NotFound();
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[ApiController]` | Enables API conventions. |
| `[Route("api/[controller]")]` | Base route using the controller name (`Products`). |
| `: ControllerBase` | Base class without view support. |
| Constructor | Receives dependencies via DI. |
| Action methods | Public methods decorated with HTTP verb attributes. |

#### Controller Lifecycle

```text
Request → Routing selects controller/action → Controller instantiated (per request)
        → Constructor dependencies resolved → Filters run → Action executes
        → Result executed → Response written → Controller disposed
```

**Syntax rules:**

- **API controllers must derive from `ControllerBase`** (not `Controller`).
- **Controllers are per‑request** – do not store request‑specific state in fields that persist across requests.
- **Controller discovery** requires `AddControllers()` in `Program.cs`; routing requires `MapControllers()`.
- **Constructor injection is preferred** – action injection (`[FromServices]`) is available for one‑off needs.

**Constraints and limitations:**

- **No view support** – `ControllerBase` lacks `View()`, `PartialView()`, etc.
- **Controller instances are not registered in DI by default** – call `AddControllersAsServices()` if you need controllers resolved from DI (e.g., for decorators or interceptors).
- **Actions must be public** – non‑public methods are not discovered as actions.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddScoped<IProductService, ProductService>();
var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();

// ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductService service,
        ILogger<ProductsController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet]
    public async Task<ActionResult<IReadOnlyList<ProductDto>>> GetAll()
    {
        _logger.LogInformation("Listing all products");
        var products = await _service.ListAsync();
        return Ok(products);
    }
}
```

**Step‑by‑step explanation:**

1. `AddControllers()` registers API controller services.
2. `AddScoped<IProductService, ProductService>()` registers the service the controller depends on.
3. `MapControllers()` maps attribute‑routed controllers.
4. `ProductsController` derives from `ControllerBase`, is decorated with `[ApiController]`, and receives its dependencies via constructor injection.
5. `GetAll` returns `200 OK` with the product list.

**Expected Output:** A request to `GET /api/products` returns `200 OK` with a JSON array of products.

**Why this output?** The controller is instantiated per request, dependencies are resolved, the action executes, and the result is serialized as JSON.

### Real‑World Cases

- **CRUD APIs:** `ProductsController`, `OrdersController`, `CustomersController`.
- **Search APIs:** `SearchController` with `[FromQuery]` filters.
- **Admin APIs:** `AdminUsersController` with policy‑protected actions.
- **Reporting APIs:** `ReportsController` with complex queries and DTO projections.

### References

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- `ControllerBase` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase

---

## 2. `[ApiController]` (The Attribute)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ApiController]` is an attribute that enables API‑specific conventions on a controller class, reducing boilerplate and standardizing behaviour. |
| **Technical Definition** | `ApiControllerAttribute` implements `IApiBehaviorMetadata` and is consumed by the `ApiBehaviorApplicationModelProvider`. It enables: (1) **binding‑source inference** (complex types → `[FromBody]`, route parameters → `[FromRoute]`, simple types → `[FromQuery]`, `IFormFile` → `[FromForm]`), (2) **automatic 400 responses** when `ModelState` is invalid (via `ModelStateInvalidFilter`), (3) **`ProblemDetails` responses** for error status codes, and (4) **attribute‑routing requirement** (controllers using `[ApiController]` cannot use conventional routing). |
| **Beginner‑Friendly Explanation** | `[ApiController]` is the **"make this an API"** sticker. It turns on a set of sensible defaults: parameters are bound from the right place automatically, invalid request bodies produce a clean `400` response, and errors are returned as `ProblemDetails` JSON. Without it, you would have to write all that behaviour yourself. |

### Purposes

- **To** enable **API‑specific conventions** that reduce boilerplate and improve consistency.
- **To** infer **binding sources** automatically, so `[FromBody]` and `[FromQuery]` are often unnecessary.
- **To** short‑circuit invalid requests with **automatic `400 Bad Request`** responses containing `ProblemDetails`.
- **To** require **attribute routing**, ensuring API controllers have explicit routes.
- **To** produce **consistent error responses** across the API.

### Syntax Rules and Structure

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public ActionResult<OrderDto> Create(OrderInputModel input)
    {
        // input is inferred as [FromBody]
        // invalid ModelState → automatic 400
        return Created("", new OrderDto());
    }
}
```

**Component breakdown:**

| Convention | Behaviour |
|------------|-----------|
| **Binding inference** | `OrderInputModel` (complex) → `[FromBody]`; `int id` in route → `[FromRoute]`; `string? filter` in action → `[FromQuery]`. |
| **Automatic 400** | `ModelState.IsValid == false` → `400 Bad Request` with `ValidationProblemDetails`. |
| **Attribute routing required** | Controllers using `[ApiController]` must use attribute routing. |
| **ProblemDetails** | Error status codes include a `ProblemDetails` body by default. |

#### Overriding Defaults (ApiBehaviorOptions)

```csharp
builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.SuppressModelStateInvalidFilter = true;   // Disable automatic 400
    options.InvalidModelStateResponseFactory = context =>
    {
        var errors = context.ModelState
            .Where(kv => kv.Value!.Errors.Count > 0)
            .ToDictionary(
                kv => kv.Key,
                kv => kv.Value!.Errors.Select(e => e.ErrorMessage).ToArray());
        return new BadRequestObjectResult(new { errors });
    };
});
```

**Syntax rules:**

- **Automatic 400 can be suppressed** via `SuppressModelStateInvalidFilter` when custom error shapes are required.
- **The default error response** is `ValidationProblemDetails`, a `ProblemDetails` subclass with an `errors` dictionary.
- **Binding inference can be overridden** with explicit `[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]` attributes.
- **`[ApiController]` is inherited** – applying it to a base controller makes all derived controllers API controllers.

**Constraints and limitations:**

- **Automatic 400 responses can be surprising** for clients expecting a different error shape; configure `InvalidModelStateResponseFactory` if needed.
- **Binding inference may choose unexpectedly** – when a complex type is bound to the body, query parameters won't populate it; use explicit attributes to disambiguate.
- **Minimal APIs use different conventions** – `[ApiController]` applies only to controller‑based APIs.

### Annotated Complete Step‑by‑Step Code Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<CustomerDto>> Create(
        CreateCustomerDto input,               // → [FromBody] inferred
        [FromHeader(Name = "X-Tenant")] string tenant, // → header
        CancellationToken ct)                  // → request aborted token
    {
        // Automatic 400 runs BEFORE this action when ModelState is invalid.
        var created = await _service.CreateAsync(tenant, input, ct);
        return CreatedAtAction(nameof(GetById),
            new { id = created.Id }, created);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<CustomerDto>> GetById(int id) =>
        await _service.FindAsync(id) is { } customer
            ? Ok(customer)
            : NotFound();
}

// DTO with validation attributes
public record CreateCustomerDto(
    [property: Required, StringLength(100)] string Name,
    [property: EmailAddress] string Email);
```

**Step‑by‑step explanation:**

1. `[ApiController]` enables binding inference: `input` is bound from the body, `tenant` from the header, `ct` from the request aborted token.
2. Validation attributes on `CreateCustomerDto` are evaluated during model binding.
3. If `ModelState` is invalid, the `ModelStateInvalidFilter` short‑circuits with `400 Bad Request`—the action never runs.
4. If valid, the service is called and a `201 Created` response is returned.

**Expected Output:** A valid POST returns `201 Created` with the new customer. An invalid POST (missing name, malformed email) returns `400 Bad Request` with `ValidationProblemDetails` listing the errors.

**Why this output?** `[ApiController]` wires up the automatic 400 path and infers binding sources, eliminating boilerplate from the controller.

### Real‑World Cases

- **Public APIs:** Consistent validation errors via `ValidationProblemDetails`.
- **Microservice APIs:** Reduced boilerplate for binding and validation.
- **APIs with custom error formats:** Override `InvalidModelStateResponseFactory` while retaining the rest of `[ApiController]`.

### References

- `[ApiController]` attribute – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- `ApiBehaviorOptions` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions

---

## 3. `[Route]` (Attribute Routing)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Route]` is an attribute that defines the URL template for a controller or an action in attribute‑based routing. |
| **Technical Definition** | `RouteAttribute` implements `IRouteTemplateProvider` and applies a route template to the controller class or action. Templates can include **literal segments**, **parameters** (`{id}`), **defaults** (`{id=1}`), **constraints** (`{id:int}`), and **optional markers** (`{id?}`). The `[controller]` and `[action]` tokens are replaced with the controller and action names. `[Route]` is often combined with HTTP verb attributes (`[HttpGet]`, `[HttpPost]`). |
| **Beginner‑Friendly Explanation** | `[Route]` is the **"URL pattern"** sticker. It tells ASP.NET Core which URLs this controller (or action) responds to. `[Route("api/[controller]")]` on a `ProductsController` means the base path is `/api/products`. Adding `[HttpGet("{id:int}")]` to an action means `GET /api/products/42` is handled by that action. |

### Purposes

- **To** define **explicit, predictable URLs** for API endpoints.
- **To** support **fine‑grained routing** per action, rather than convention‑based controller/action mapping.
- **To** enable **RESTful URL structures** with resource identifiers (`/api/products/42`).
- **To** apply **route constraints** (`int`, `guid`, `regex`) to validate URL segments.
- **To** support **route token substitution** (`[controller]`, `[action]`) for convention‑based attribute routing.

### Syntax Rules and Structure

#### General Syntax (Controller‑Level Route)

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase { ... }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `api/` | Literal prefix. |
| `[controller]` | Token replaced with controller name (`Products`). |

#### General Syntax (Action‑Level Route)

```csharp
[HttpGet("{id:int}")]
public ActionResult<Product> GetById(int id) => ...;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[HttpGet]` | HTTP verb; combined with the controller route. |
| `{id:int}` | Route parameter with `int` constraint. |

#### Combined Route Templates

| Controller Route | Action Route | Effective URL |
|------------------|--------------|---------------|
| `api/[controller]` | `[HttpGet]` | `GET /api/products` |
| `api/[controller]` | `[HttpGet("{id:int}")]` | `GET /api/products/42` |
| `api/[controller]` | `[HttpPost]` | `POST /api/products` |
| `api/[controller]` | `[HttpPut("{id:int}")]` | `PUT /api/products/42` |
| `api/[controller]` | `[HttpDelete("{id:int}")]` | `DELETE /api/products/42` |

#### Route Constraints

```csharp
[HttpGet("{id:int}")]                    // Integer only
[HttpGet("{slug:regex(^[a-z0-9-]+$)}")]  // Regex
[HttpGet("{date:datetime}")]             // DateTime
[HttpGet("{id:guid}")]                   // GUID
```

**Syntax rules:**

- **Controller‑level `[Route]`** applies to all actions unless overridden.
- **Action‑level routes** combine with the controller route (concatenated, not overridden).
- **`[controller]`** is replaced by the controller name without the `Controller` suffix.
- **`[action]`** is replaced by the action method name.
- **Multiple `[Route]` attributes** on one action create multiple URLs for the same action.

**Constraints and limitations:**

- **Controllers with `[ApiController]` must use attribute routing** —conventional routing is not supported.
- **Ambiguous routes** cause runtime errors; use distinct route templates or constraints to disambiguate.
- **Route parameters and action parameters must match** —the `id` in `{id:int}` must correspond to an `id` parameter (or be bound via `[FromRoute]`).

### Annotated Complete Step‑by‑Step Code Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet]                          // GET  /api/orders
    public ActionResult<IEnumerable<OrderDto>> GetAll() => Ok(_orders);

    [HttpGet("{id:int}")]              // GET  /api/orders/42
    public ActionResult<OrderDto> GetById(int id) =>
        _orders.FirstOrDefault(o => o.Id == id) is { } order
            ? Ok(order)
            : NotFound();

    [HttpGet("{id:int}/items")]        // GET  /api/orders/42/items
    public ActionResult<IEnumerable<OrderItemDto>> GetItems(int id) =>
        Ok(_orders.FirstOrDefault(o => o.Id == id)?.Items ?? []);

    [HttpPost]                         // POST /api/orders
    public ActionResult<OrderDto> Create([FromBody] CreateOrderDto dto)
    {
        var created = _service.Create(dto);
        return CreatedAtAction(nameof(GetById),
            new { id = created.Id }, created);
    }

    [HttpDelete("{id:int}")]           // DELETE /api/orders/42
    public IActionResult Delete(int id)
    {
        if (!_service.Exists(id)) return NotFound();
        _service.Delete(id);
        return NoContent();
    }
}
```

**Step‑by‑step explanation:**

1. `[Route("api/[controller]")]` sets the base path to `/api/orders`.
2. `[HttpGet]` maps `GET /api/orders`.
3. `[HttpGet("{id:int}")]` maps `GET /api/orders/42`; the `int` constraint rejects non‑numeric IDs.
4. `[HttpGet("{id:int}/items")]` maps `GET /api/orders/42/items`.
5. `[HttpPost]` maps `POST /api/orders`.
6. `[HttpDelete("{id:int}")]` maps `DELETE /api/orders/42`.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/orders` | `200 OK` with array |
| `GET /api/orders/42` | `200 OK` or `404 Not Found` |
| `GET /api/orders/42/items` | `200 OK` with items |
| `POST /api/orders` | `201 Created` + `Location` |
| `DELETE /api/orders/42` | `204 No Content` or `404 Not Found` |
| `GET /api/orders/abc` | `404 Not Found` (constraint fails) |

**Why this output?** The controller‑level `[Route]` and action‑level HTTP verb attributes combine to define the URL space; constraints reject invalid segments before the action runs.

### Real‑World Cases

- **RESTful resource APIs:** `/api/products`, `/api/products/{id}`, `/api/products/{id}/reviews`.
- **Versioned APIs:** `[Route("api/v{version:apiVersion}/[controller]")]` with API versioning.
- **Nested resources:** `/api/customers/{customerId}/orders/{orderId}`.
- **SEO‑friendly slugs:** `[HttpGet("articles/{slug}")]` for content APIs.

### References

- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- `RouteAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints

---

## 4. API Controller Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API controller best practices are guidelines for designing and implementing controllers that are secure, consistent, maintainable, and aligned with RESTful API design. |
| **Technical Definition** | Best practices span **thin controllers** (delegating business logic to services), **DTO‑based binding** (never bind domain entities), **correct status codes** (`201`, `204`, `404`, `409`, `422`), **validation via `[ApiController]`**, **versioning**, **async‑all‑the‑way**, **`CancellationToken` propagation**, **`Location` headers on `201`**, **`ProblemDetails` errors**, and **pagination conventions** for collections. |
| **Beginner‑Friendly Explanation** | Best practices are the **"do's and don'ts"** for API controllers—they help you avoid bloated controllers, security holes from binding entities directly, incorrect status codes, and APIs that are hard to consume or evolve. |

### Purposes

- **To** keep controllers **thin** and business logic in services.
- **To** prevent **over‑posting** by binding DTOs instead of domain entities.
- **To** return **correct status codes** and **resource locations**.
- **To** provide **consistent error responses** via `ProblemDetails`.
- **To** support **scalable, cancellable** request processing.
- **To** enable **safe evolution** through versioning and additive changes.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Controller size** | Keep actions thin—delegate to services. | Put business logic or data access in actions. |
| **Binding targets** | Bind **DTOs/ViewModels** with validation attributes. | Bind domain entities (over‑posting, accidental property exposure). |
| **Status codes** | `201 Created` + `Location` on POST; `204 No Content` on PUT/PATCH/DELETE without body; `404` for missing resources; `409` for conflicts. | Return `200 OK` for everything or leak 500s for validation failures. |
| **Validation** | Rely on `[ApiController]` automatic 400s; customize `InvalidModelStateResponseFactory` if needed. | Ignore `ModelState`; duplicate validation logic in actions. |
| **Async** | Use `async`/`await` and propagate `CancellationToken`. | Use synchronous data access or ignore `HttpContext.RequestAborted`. |
| **Route design** | Use plural nouns (`/api/products`) and consistent verbs. | Mix RPC‑style verbs (`/api/getProducts`) with REST. |
| **Action signatures** | Use `ActionResult<T>` to combine data and status codes. | Use `object` or untyped returns. |
| **Versioning** | Version from day one (URL, header, or media type). | Break existing consumers without a version change. |
| **Error shape** | Use `ProblemDetails` for standardized errors. | Return raw exception messages or HTML. |
| **Authentication** | Use `[Authorize]`; keep authentication configuration in `Program.cs`. | Roll custom auth inside actions. |
| **Logging** | Inject `ILogger<T>` and log at the controller or service boundary. | Use `Console.WriteLine` or log sensitive data. |
| **Testing** | Use `WebApplicationFactory` for integration tests; mock services for unit tests. | Test controllers only through manual calls. |

### Annotated Code Example (Best‑Practice Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductService service,
        ILogger<ProductsController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet]
    [AllowAnonymous]
    public async Task<ActionResult<PagedResult<ProductDto>>> List(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20,
        CancellationToken ct = default)
    {
        var result = await _service.ListAsync(page, pageSize, ct);
        return Ok(result);
    }

    [HttpGet("{id:int}")]
    [AllowAnonymous]
    public async Task<ActionResult<ProductDto>> GetById(
        int id, CancellationToken ct)
        => await _service.FindAsync(id, ct) is { } product
            ? Ok(product)
            : NotFound();

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Create(
        CreateProductDto input, CancellationToken ct)
    {
        var created = await _service.CreateAsync(input, ct);
        _logger.LogInformation("Product {Id} created", created.Id);
        return CreatedAtAction(nameof(GetById),
            new { id = created.Id }, created);
    }

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id, CancellationToken ct)
    {
        if (!await _service.ExistsAsync(id, ct)) return NotFound();
        await _service.DeleteAsync(id, ct);
        return NoContent();
    }
}

public record CreateProductDto(
    [property: Required, StringLength(100)] string Name,
    [property: Range(0.01, 1_000_000)] decimal Price);
```

**Why this follows best practices:**

- **Thin controller** – business logic is in `IProductService`.
- **DTO binding** – `CreateProductDto` with validation attributes; no domain entity exposure.
- **Correct status codes** – `201 Created` with `Location` on create; `204 No Content` on delete; `404` on missing.
- **`ActionResult<T>`** – combines data and status codes.
- **Async + `CancellationToken`** – scalable and cancellable.
- **`[ApiController]`** – automatic validation errors.
- **Authorization** – `[Authorize]` on the controller; `[AllowAnonymous]` on read endpoints.
- **Logging** – injected logger, logs meaningful events without sensitive data.

### Real‑World Cases

- **Public APIs:** DTO‑based inputs, `ProblemDetails` errors, versioning from day one.
- **Admin APIs:** `[Authorize]` with policy requirements, audit logging, strict status codes.
- **Mobile backends:** Pagination, `CancellationToken`, and additive versioning.
- **Microservice APIs:** Thin controllers, service delegation, health checks, and consistent error shapes.

### References

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- API design best practices – https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- `[ApiController]` conventions – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- `ProblemDetails` for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457

---

## Summary Table: API Controllers at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **API controllers** | Handle HTTP requests and return data | `ControllerBase`, `[ApiController]`, `ActionResult<T>` | Controller‑based APIs |
| **`[ApiController]`** | Enable API conventions | Binding inference, automatic 400, `ProblemDetails` | Every API controller |
| **`[Route]`** | Define URL templates | `[Route("api/[controller]")]`, `{id:int}` | Attribute‑routed APIs |
| **Best practices** | Secure, consistent, maintainable controllers | Thin actions, DTO binding, correct status codes, async, versioning | Every production API |

---

## References (Full List)

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- `[ApiController]` attribute – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- `ControllerBase` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase
- `ApiBehaviorOptions` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- `RouteAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- API design best practices – https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- Tutorial: Create a web API with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api