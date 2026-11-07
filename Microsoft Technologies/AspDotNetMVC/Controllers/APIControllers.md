# API Controllers in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). The `[ApiController]` attribute and its opinionated behaviours have been stable since ASP.NET Core 2.1. `ControllerBase` is the recommended base class for web API controllers; `Controller` adds view support and is intended for MVC pages. `[Route]` attribute routing is a **requirement** when `[ApiController]` is applied. In ASP.NET Core 10 and later, known API endpoints return 401/403 instead of redirecting to login pages when using cookie authentication, further distinguishing API behaviour from MVC.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API controllers are controller classes that handle HTTP requests and return **data** (JSON, XML, etc.) rather than HTML views, serving as the foundation for RESTful web APIs in ASP.NET Core. |
| **Technical Definition** | An API controller derives from `ControllerBase` and is typically decorated with `[ApiController]` and `[Route]`. The `[ApiController]` attribute enables opinionated, API-specific behaviours: **attribute routing requirement**, **automatic HTTP 400 responses** for invalid `ModelState`, **binding source parameter inference**, **multipart/form-data inference** for `IFormFile`, and **ProblemDetails** for error status codes. `[Route]` defines the URL template at the controller or action level, supporting token replacement (`[controller]`, `[action]`, `[area]`). Controllers are activated and disposed **per request** and should remain thin by delegating business logic to services. |
| **Beginner-Friendly Explanation** | API controllers are the **data endpoints** of your application. Unlike traditional MVC controllers that return HTML pages for browsers, API controllers return structured data (usually JSON) for clients like mobile apps, SPAs, or other services. You mark them with `[ApiController]` to get helpful automatic behaviours (like returning 400 errors for bad input), and you use `[Route]` to define the URLs they respond to. |

### Key Characteristics

- **`ControllerBase` base class** – Lightweight, HTTP-focused; no view support.
- **`[ApiController]` opinionated behaviours** – Attribute routing required, automatic 400s, binding source inference, ProblemDetails.
- **Attribute routing mandatory** – Conventional routes defined by `UseEndpoints`, `UseMvc`, or `UseMvcWithDefaultRoute` are **inaccessible** for `[ApiController]`-decorated controllers.
- **Binding source inference** – Parameters are automatically bound from body, route, query, etc., based on type and name.
- **Automatic ProblemDetails** – Error responses (400+) are transformed into RFC 7807 ProblemDetails.
- **Thin by design** – Business logic belongs in services; controllers handle HTTP concerns only.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllers()`).
- **`app.MapControllers()`** called in `Program.cs` to discover attribute-routed controllers.
- Basic understanding of **HTTP verbs**, **routing**, and **JSON serialization**.

### Related Programming Areas

- **Routing** – Attribute routing is the primary routing strategy for APIs.
- **Model Binding** – `[ApiController]` infers binding sources; explicit attributes override.
- **Model Validation** – Data annotations trigger automatic 400 responses.
- **ProblemDetails** – Standard error response format for APIs.
- **Minimal APIs** – An alternative to controllers for simple APIs; controllers remain preferred for larger, organised APIs.
- **OpenAPI/Swagger** – `[ApiController]` and `[ProducesResponseType]` improve API documentation.

### Core Concepts / Features

1. **API Controllers (General)** – The role, base class, and lifecycle.
2. **`[ApiController]`** – The attribute that enables opinionated API behaviours.
3. **`[Route]`** – Attribute routing for defining API URL templates.
4. **API Controller Best Practices** – Guidelines for thin, secure, and well-documented APIs.

---

## 1. API Controllers (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API controllers are classes derived from `ControllerBase` that handle HTTP requests and return data responses (JSON, XML, files) for web API clients. |
| **Technical Definition** | A controller-based web API consists of one or more controller classes deriving from `ControllerBase`. Controllers are activated and disposed **per request**. `ControllerBase` provides HTTP-oriented helpers (`Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`) and properties (`User`, `Request`, `Response`) without view-rendering capabilities. API controllers typically use **attribute routing** and return `IActionResult`, `ActionResult<T>`, or POCOs that are serialized by output formatters. |
| **Beginner-Friendly Explanation** | API controllers are the **data-serving counterparts** of MVC controllers. When a mobile app or JavaScript client calls `/api/products/5`, the API controller fetches the product and returns it as JSON. Unlike MVC controllers, API controllers don't render HTML pages—they just send data. They're lighter and focused purely on HTTP data exchange. |

### Purposes (each begins with “To”)

- **To** expose application functionality as HTTP endpoints that return structured data (JSON, XML) for clients.
- **To** provide a **lightweight** base class (`ControllerBase`) without the overhead of view rendering.
- **To** leverage `[ApiController]` opinionated behaviours for consistent API semantics (automatic 400s, ProblemDetails).
- **To** support RESTful resource modelling with HTTP verbs and attribute routing.
- **To** enable **DTO-based** responses that decouple the API surface from domain models.

### Syntax Rules and Structure

#### General Syntax

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById(int id) { ... }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ControllerBase` | Base class for APIs; provides HTTP helpers, no view support. |
| `[ApiController]` | Enables opinionated API behaviours (see Section 2). |
| `[Route]` | Defines the URL template (see Section 3). |
| `ActionResult<T>` | Typed return allowing multiple status codes. |

#### Syntax Rules

- **Derive from `ControllerBase`** for APIs; use `Controller` only if the same controller must also serve views.
- **Apply `[ApiController]`** to enable API-specific behaviours.
- **Apply `[Route]`** — attribute routing is a requirement for `[ApiController]`.
- **Controllers are per-request** — do not store request state in fields.

#### Constraints and Limitations

- **`Controller` should not be used for pure APIs** — it adds view support that APIs don't need.
- **Conventional routing is inaccessible** when `[ApiController]` is applied.
- **`ControllerBase` lacks `View()`** — attempting to return a view will not compile.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal API Controller

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;

    [HttpGet]
    public ActionResult<IEnumerable<ProductDto>> GetAll()
        => Ok(_service.GetAll());

    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetById(id);
        if (product == null) return NotFound();
        return product;
    }
}
```

**Step-by-step explanation:**

1. `ControllerBase` provides `Ok()` and `NotFound()` without view support.
2. `[ApiController]` enables automatic model validation and binding inference.
3. `[Route("api/[controller]")]` sets the base URL to `api/Products`.
4. `[HttpGet]` and `[HttpGet("{id:int}")]` map GET requests to the respective actions.

**Expected Output:**

| Method | URL | Result |
|--------|-----|--------|
| GET | `/api/Products` | 200 OK with JSON array |
| GET | `/api/Products/5` | 200 OK with JSON object (if found) |
| GET | `/api/Products/999` | 404 Not Found |

**Why this output?** `ControllerBase` helpers produce the correct status codes; `[ApiController]` ensures invalid models are rejected with 400.

---

#### Example 2: API Controller with POST and `CreatedAtAction`

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var created = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** `201 Created`, `Location: /api/Products/42`, body = created product JSON.

**Why this output?** `CreatedAtAction` generates the `Location` header using the named action and route values.

---

#### Example 3: Same Controller Serving Views and APIs (Not Recommended for Pure APIs)

```csharp
public class HybridController : Controller  // Derives from Controller, not ControllerBase
{
    public IActionResult Index() => View();          // MVC page
    [HttpGet("api/data")]
    public IActionResult GetData() => Ok(new { });   // API endpoint
}
```

**Expected Output:** `/Hybrid/Index` renders a view; `/api/data` returns JSON.

**Why this output?** `Controller` inherits from `ControllerBase`, so both view and API capabilities are available. However, Microsoft recommends deriving from `ControllerBase` for pure APIs to avoid unnecessary overhead.

### Real-World Cases

- **REST APIs** – `ProductsController`, `OrdersController`, `CustomersController` returning JSON.
- **Microservices** – lightweight controllers with `[ApiController]` and attribute routing.
- **Mobile backends** – API controllers serving JSON to iOS/Android clients.

### References

- Create web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- `ControllerBase` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase

---

## 2. `[ApiController]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ApiController]` is an attribute applied to a controller class that enables **opinionated, API-specific behaviours**, including attribute routing requirement, automatic HTTP 400 responses, binding source inference, and ProblemDetails for error statuses. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ApiControllerAttribute` triggers a set of conventions and filters: (1) **Attribute routing requirement** – actions are inaccessible via conventional routes. (2) **Automatic 400 responses** – `ModelStateInvalidFilter` short-circuits invalid models with `ValidationProblemDetails`. (3) **Binding source inference** – parameters are bound from body, route, query, etc., based on rules (complex types → body; route-matching names → route; others → query). (4) **Multipart/form-data inference** – `IFormFile` parameters infer `multipart/form-data`. (5) **ProblemDetails transformation** – error status codes (400+) are converted to RFC 7807 ProblemDetails. The attribute can be applied to individual controllers, a base class, or the entire assembly. |
| **Beginner-Friendly Explanation** | `[ApiController]` is a **magic label** you put on API controllers to get helpful automatic behaviours. It tells the framework: “Treat this as a proper API—require attribute routing, automatically return 400 for bad input, figure out where parameters should come from, and return standard error formats.” Without it, you'd have to write all that plumbing yourself. |

### Purposes (each begins with “To”)

- **To** enforce **attribute routing** as the routing strategy for the controller.
- **To** automatically return **HTTP 400** with `ValidationProblemDetails` when `ModelState` is invalid, eliminating manual checks.
- **To** **infer binding sources** for action parameters (body, route, query, form) without explicit attributes.
- **To** transform error status codes (400+) into standard **ProblemDetails** responses (RFC 7807).
- **To** enable **multipart/form-data inference** for `IFormFile` parameters.
- **To** provide consistent API semantics across all controllers in the application.

### Syntax Rules and Structure

#### General Syntax (Controller Level)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase { ... }
```

#### General Syntax (Base Class)

```csharp
[ApiController]
public abstract class MyApiControllerBase : ControllerBase { }

[Route("api/[controller]")]
public class ProductsController : MyApiControllerBase { ... }
```

#### General Syntax (Assembly Level)

```csharp
[assembly: ApiController]
```

#### Component Breakdown

| Application level | Effect |
|-------------------|--------|
| Controller | Behaviours apply to that controller only. |
| Base class | Behaviours apply to all derived controllers. |
| Assembly | Behaviours apply to **all** controllers in the assembly; **no opt-out**. |

#### Syntax Rules

- **Attribute routing is required** — conventional routes are ignored.
- **Automatic 400s** — `if (!ModelState.IsValid)` is unnecessary in actions.
- **Binding inference** — explicit attributes (`[FromQuery]`, `[FromBody]`, etc.) override inference.
- **ProblemDetails** — error results (400+) are transformed automatically.

#### Constraints and Limitations

- **No opt-out at assembly level** — applying `[ApiController]` to the assembly affects all controllers.
- **Inference can be disabled** — via `SuppressInferBindingSourcesForParameters = true` in `ApiBehaviorOptions`.
- **Automatic 400s can be disabled** — via `SuppressModelStateInvalidFilter = true`.
- **Binding inference exceptions** — complex built-in types like `IFormCollection` and `CancellationToken` are ignored; simple types (string, int) are not inferred as `[FromBody]`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Automatic 400 Response

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        // No need for: if (!ModelState.IsValid) return BadRequest(ModelState);
        var created = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

**Step-by-step explanation:**

1. `[ApiController]` enables `ModelStateInvalidFilter`.
2. If `dto` fails validation (e.g., `[Required]` on a property), the filter automatically returns 400 with `ValidationProblemDetails`.
3. The action body is never executed for invalid models.

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Valid DTO | 201 Created |
| Invalid DTO (missing required field) | 400 Bad Request with `ValidationProblemDetails` |

**Why this output?** The automatic filter short-circuits invalid requests, ensuring consistent error responses.

---

#### Example 2: Binding Source Inference

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{customerId:int}/orders")]
    public ActionResult<IEnumerable<OrderDto>> GetOrders(
        int customerId,                          // Inferred [FromRoute]
        [FromQuery] int page = 1,                // Explicit [FromQuery]
        [FromQuery] int pageSize = 20,           // Explicit [FromQuery]
        [FromHeader(Name = "X-Tenant-Id")] string tenantId)  // Explicit [FromHeader]
    {
        return Ok(new { customerId, page, pageSize, tenantId });
    }
}
```

**Expected Output:**

| Request | Result |
|---------|--------|
| `GET /api/Orders/5/orders?page=2&pageSize=10` with header `X-Tenant-Id: acme` | `{ "customerId": 5, "page": 2, "pageSize": 10, "tenantId": "acme" }` |

**Why this output?** `[ApiController]` infers `customerId` as route-bound because it matches the route token; explicit attributes override inference for the others.

---

#### Example 3: Assembly-Level `[ApiController]`

```csharp
// Program.cs or AssemblyInfo.cs
[assembly: ApiController]

// All controllers in the assembly now have API behaviours
public class ProductsController : ControllerBase { ... }
public class OrdersController : ControllerBase { ... }
```

**Expected Output:** All controllers in the assembly require attribute routing, return automatic 400s, and use binding inference.

**Why this output?** Assembly-level application applies `[ApiController]` behaviours globally; no per-controller opt-out is possible.

### Real-World Cases

- **REST APIs** – consistent 400 responses and binding inference across all controllers.
- **Microservices** – assembly-level `[ApiController]` ensures uniform API behaviour.
- **Public APIs** – ProblemDetails responses simplify client error handling.

### References

- `ApiControllerAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apicontrollerattribute
- Create web APIs with ASP.NET Core: ApiController attribute – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- Binding source parameter inference – https://learn.microsoft.com/en-us/aspnet/core/web-api/#binding-source-parameter-inference

---

## 3. `[Route]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Route]` is an attribute that defines the **URL template** for a controller or action, enabling attribute routing. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.RouteAttribute` implements `IRouteTemplateProvider` and specifies a route template string, optional `Order` (for precedence), and optional `Name` (for URL generation). When applied to a controller, the template serves as a **prefix** for all actions; when applied to an action, it **combines** with the controller template (unless it starts with `/` or `~/`, which makes it absolute). **Token replacement** replaces `[controller]`, `[action]`, and `[area]` with the corresponding names. Route templates support parameters, optional markers, default values, and constraints. |
| **Beginner-Friendly Explanation** | `[Route]` is the **address label** for your API endpoints. On a controller, it sets the base URL (like `api/products`). On an action, it adds the specific path (like `{id}`). Together they tell the framework: “This code handles URLs that look like this.” You can use shortcuts like `[controller]` to automatically use the controller’s name. |

### Purposes (each begins with “To”)

- **To** define the URL template for a controller or action in attribute routing.
- **To** set a **shared prefix** for all actions in a controller (controller-level `[Route]`).
- **To** define **action-specific paths** (action-level `[Route]`).
- **To** support **token replacement** (`[controller]`, `[action]`, `[area]`) for convention-based templates.
- **To** allow **absolute templates** (starting with `/` or `~/`) that override the controller prefix.
- **To** integrate with HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.) for method-specific routing.

### Syntax Rules and Structure

#### General Syntax (Controller Level)

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase { ... }
```

#### General Syntax (Action Level)

```csharp
[HttpGet("{id:int}")]
public ActionResult<ProductDto> GetById(int id) { ... }
```

#### General Syntax (Absolute Override)

```csharp
[HttpGet("~/api/special")]
public IActionResult Special() { ... }
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `[controller]` | Replaced with the controller name (without `Controller` suffix). |
| `[action]` | Replaced with the action method name. |
| `[area]` | Replaced with the area name. |
| `/` or `~/` prefix | Makes the template **absolute**, ignoring the controller prefix. |
| `{id:int}` | Route parameter with an `int` constraint. |

#### Syntax Rules

- **Attribute routing is required** for `[ApiController]` controllers.
- **Controller-level `[Route]`** applies a prefix to all actions.
- **Action-level templates combine** with the controller prefix unless they start with `/` or `~/`.
- **Token replacement** occurs as the **final step** of building attribute routes.
- **Reserved names** (`action`, `area`, `controller`, `handler`, `page`) cannot be used as route parameters.

#### Constraints and Limitations

- **Don't use `[FromRoute]` when values might contain `%2f`** (`/`) — `%2f` won't be unescaped; use `[FromQuery]` instead.
- **Conventional routes are ignored** for `[ApiController]` controllers.
- **Route template precedence** determines which action matches when multiple templates are candidates.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Controller-Level Route with Token Replacement

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                 // GET api/Products
    public ActionResult GetAll() => Ok();

    [HttpGet("{id:int}")]    // GET api/Products/5
    public ActionResult GetById(int id) => Ok(id);
}
```

**Step-by-step explanation:**

1. `[Route("api/[controller]")]` sets the base template to `api/Products`.
2. `[HttpGet]` inherits the base template → `GET api/Products`.
3. `[HttpGet("{id:int}")]` appends `{id:int}` → `GET api/Products/5`.

**Expected Output:**

| Request | Result |
|---------|--------|
| `GET /api/Products` | 200 OK |
| `GET /api/Products/5` | 200 OK with ID |
| `GET /api/Products/abc` | 404 (constraint fails) |

**Why this output?** The `int` constraint rejects non-integer IDs; token replacement produces the correct URL segments.

---

#### Example 2: Action-Level Route with Absolute Override

```csharp
[ApiController]
[Route("api/[controller]")]
public class ReportsController : ControllerBase
{
    [HttpGet("daily")]                  // GET api/Reports/daily
    public IActionResult Daily() => Ok();

    [HttpGet("~/reports/summary")]      // GET reports/summary (absolute)
    public IActionResult Summary() => Ok();
}
```

**Expected Output:**

| Request | Result |
|---------|--------|
| `GET /api/Reports/daily` | 200 OK |
| `GET /reports/summary` | 200 OK |
| `GET /api/Reports/reports/summary` | 404 (not combined) |

**Why this output?** The `~/` prefix makes the action template absolute, bypassing the controller prefix.

---

#### Example 3: Route with Multiple Tokens

```csharp
[ApiController]
[Route("[controller]/[action]")]
public class BooksController : ControllerBase
{
    public IActionResult List() => Ok();          // GET Books/List
    public IActionResult Details(int id) => Ok(); // GET Books/Details?id=5
}
```

**Expected Output:** `GET /Books/List` and `GET /Books/Details?id=5` are routed to the respective actions.

**Why this output?** Token replacement converts `[controller]` to `Books` and `[action]` to `List` or `Details`.

### Real-World Cases

- **REST APIs** – `[Route("api/[controller]")]` for resource-based URLs.
- **Versioned APIs** – `[Route("api/v{version:apiVersion}/[controller]")]` for versioning.
- **Custom URL structures** – absolute templates for non-resource endpoints.

### References

- `RouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- Routing to controller actions: Attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing
- Token replacement in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing

---

## 4. API Controller Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API controller best practices are guidelines for writing controllers that are **thin, secure, well-documented, and semantically correct** in their HTTP behaviour. |
| **Technical Definition** | Best practices include: deriving from `ControllerBase` (not `Controller`), applying `[ApiController]`, keeping controllers thin by delegating business logic to services, using DTOs to decouple the API surface from domain models, using `ActionResult<T>` for typed responses, leveraging `[ProducesResponseType]` for OpenAPI documentation, using `[Consumes]`/`[Produces]` for content negotiation, handling validation with automatic 400s (or `ValidationProblem`), and avoiding over-posting. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep APIs clean and reliable. For example: use `ControllerBase` instead of `Controller`, keep controllers thin (business logic goes in services), use DTOs instead of exposing database models, and let `[ApiController]` handle validation automatically. These practices make your API easier to secure, test, and document. |

### Purposes (each begins with “To”)

- **To** keep controllers **thin** by delegating business logic to services, improving testability and maintainability.
- **To** use **DTOs** (Data Transfer Objects) to prevent over-posting and decouple the API from internal models.
- **To** leverage `[ApiController]` features (automatic 400s, binding inference, ProblemDetails) for consistency.
- **To** provide **accurate OpenAPI documentation** via `[ProducesResponseType]`, `[Consumes]`, and `[Produces]`.
- **To** return semantically correct **HTTP status codes** (`CreatedAtAction`, `NoContent`, `NotFound`).
- **To** avoid **conventional routing** (inaccessible with `[ApiController]`) and embrace **attribute routing**.
- **To** handle **errors consistently** via ProblemDetails and `ValidationProblem`.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Derive from `ControllerBase`** | Avoid view overhead; APIs don't render HTML. | `public class ProductsController : ControllerBase` |
| **Apply `[ApiController]`** | Automatic 400s, binding inference, ProblemDetails. | `[ApiController]` on the controller |
| **Keep controllers thin** | Business logic belongs in services. | `_service.Create(dto); return CreatedAtAction(...)` |
| **Use DTOs, not domain models** | Prevents over-posting and decouples API from DB. | `CreateProductDto` instead of `Product` |
| **Use `ActionResult<T>`** | Typed responses with multiple status codes. | `public ActionResult<ProductDto> GetById(int id)` |
| **Document responses with `[ProducesResponseType]`** | Accurate OpenAPI schemas. | `[ProducesResponseType(StatusCodes.Status201Created)]` |
| **Use `CreatedAtAction`/`CreatedAtRoute`** | Provides `Location` header for created resources. | `return CreatedAtAction(nameof(GetById), new { id }, dto);` |
| **Use `NoContent` for updates/deletes** | Matches REST semantics. | `return NoContent();` |
| **Use `ValidationProblem` for validation errors** | Standard Problem Details (RFC 7807). | `if (!ModelState.IsValid) return ValidationProblem(ModelState);` |
| **Avoid `[FromRoute]` for values with `%2f`** | `%2f` won't be unescaped; use `[FromQuery]`. | Use `[FromQuery]` for slashes. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Well-Designed REST API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;

    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public ActionResult<IEnumerable<ProductDto>> GetAll()
        => Ok(_service.GetAll());

    [HttpGet("{id:int}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetById(id);
        if (product == null) return NotFound();
        return product;
    }

    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        var created = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Update(int id, ProductDto dto)
    {
        if (!_service.Exists(id)) return NotFound();
        _service.Update(id, dto);
        return NoContent();
    }

    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Delete(int id)
    {
        if (!_service.Exists(id)) return NotFound();
        _service.Delete(id);
        return NoContent();
    }
}
```

**Step-by-step explanation:**

1. `ControllerBase` provides HTTP helpers without view support.
2. `[ApiController]` enables automatic 400s, binding inference, and ProblemDetails.
3. `[Route("api/[controller]")]` sets the base URL to `api/Products`.
4. `[ProducesResponseType]` documents possible status codes for OpenAPI.
5. Actions return `ActionResult<T>` for typed responses.

**Expected Output:**

| Endpoint | Result |
|----------|--------|
| GET `/api/Products` | 200 OK JSON array |
| GET `/api/Products/5` | 200 OK JSON or 404 |
| POST `/api/Products` | 201 Created with Location or 400 |
| PUT `/api/Products/5` | 204 or 404 |
| DELETE `/api/Products/5` | 204 or 404 |

**Why this output?** Each action returns the semantically correct result; `[ApiController]` handles validation and binding automatically.

---

#### Example 2: Using DTOs to Prevent Over-Posting

```csharp
// ❌ BAD: Using domain model directly
[HttpPost]
public IActionResult Create(Product product) { ... }  // Over-posting risk

// ✅ GOOD: Using DTO
[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var created = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** Only the properties in `CreateProductDto` are bound; sensitive fields on `Product` (e.g., `IsAdmin`) cannot be over-posted.

**Why this output?** DTOs limit the input surface, preventing clients from binding to properties they shouldn't access.

---

#### Example 3: Explicit Binding Source with Warning

```csharp
[HttpGet("search")]
public ActionResult Search([FromQuery] string term)
{
    // ✅ Good: [FromQuery] for values that may contain slashes
    return Ok(_service.Search(term));
}

// ❌ Avoid: [FromRoute] for values with %2f
[HttpGet("search/{term}")]
public ActionResult SearchRoute([FromRoute] string term) { ... }  // %2f issues
```

**Expected Output:** `GET /api/products/search?term=a/b` works correctly with `[FromQuery]`; `%2f` in route values would cause issues.

**Why this output?** `%2f` won't be unescaped to `/` in route values; `[FromQuery]` avoids this limitation.

### Real-World Cases

- **Public REST APIs** – consistent status codes, ProblemDetails, and OpenAPI documentation.
- **Multi-tenant SaaS** – DTOs prevent over-posting; `[ProducesResponseType]` documents tenant-specific responses.
- **Microservices** – thin controllers with `[ApiController]` and attribute routing for clear API contracts.

### References

- Create web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- How should I structure controllers and DTOs in an ASP.NET Core Web API? – https://learn.microsoft.com/en-us/answers/questions/5595029/how-should-i-structure-controllers-and-dtos-in-an
- `ApiControllerAttribute` best practices – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute

---

## Summary Table: API Controllers at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **API Controllers** | Handle HTTP requests and return data | Derive from `ControllerBase`; thin and focused | All web API endpoints |
| **`[ApiController]`** | Enable opinionated API behaviours | Attribute routing required; automatic 400s; binding inference | All API controllers |
| **`[Route]`** | Define URL templates | Controller prefix + action template; token replacement; absolute overrides | All attribute-routed APIs |
| **Best Practices** | Thin, secure, well-documented APIs | DTOs, `ActionResult<T>`, `[ProducesResponseType]`, `CreatedAtAction` | All production APIs |

---

## References (Full List)

- Create web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- `ApiControllerAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apicontrollerattribute
- `ControllerBase` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase
- `RouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- Binding source parameter inference – https://learn.microsoft.com/en-us/aspnet/core/web-api/#binding-source-parameter-inference
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors
- How should I structure controllers and DTOs in an ASP.NET Core Web API? – https://learn.microsoft.com/en-us/answers/questions/5595029/how-should-i-structure-controllers-and-dtos-in-an
- `ApiBehaviorOptions` (disable inference/automatic 400s) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions
- `ProducesResponseTypeAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute
- ASP.NET Core 10 API endpoint authentication behavior – https://learn.microsoft.com/en-us/aspnet/core/web-api/#1