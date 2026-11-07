# Action Parameters in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). The **binding source attributes** (`[FromQuery]`, `[FromRoute]`, `[FromBody]`, `[FromForm]`, `[FromHeader]`, `[FromServices]`) are stable across versions. In **API controllers** decorated with `[ApiController]`, binding source inference automatically applies many of these attributes based on parameter types and names, reducing the need for explicit annotations. In **MVC view controllers** (no `[ApiController]`), model binding defaults to **form values, route values, and query strings** in a defined order. `[FromBody]` requires a formatter (System.Text.Json by default) and is not used for MVC form submissions. `[FromServices]` enables method-level dependency injection but is generally discouraged in favour of constructor injection (see best practices).

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action parameters are the **inputs** declared on an action method that the framework populates from the incoming HTTP request (route, query string, body, form, headers, or DI container) before the action executes. |
| **Technical Definition** | Action parameters are bound by the **model binding** subsystem (for route, query, form, and header sources) or by **input formatters** (for the request body). The framework determines each parameter's **binding source** through explicit attributes (`[FromQuery]`, `[FromRoute]`, `[FromBody]`, `[FromForm]`, `[FromHeader]`, `[FromServices]`) or, in `[ApiController]` controllers, through **inference rules** based on parameter type and name. Bound values undergo type conversion, validation (via `ModelState`), and are then passed to the action. |
| **Beginner-Friendly Explanation** | Action parameters are the **blanks your action fills in**. When a request arrives, the framework looks at the URL (`/products/5`), the query string (`?page=2`), the request body (JSON), form data, or headers, and fills in your action's parameters accordingly. You declare `int id`, and the framework finds `5` in the URL and passes it in. |

### Key Characteristics

- **Declarative** – You declare parameters on the action; the framework finds the values.
- **Multiple sources** – Data can come from route, query, body, form, headers, or DI.
- **Type conversion** – Strings from the request are converted to the parameter's type (int, Guid, DateTime, POCO, etc.).
- **Validation-aware** – Binding failures and validation errors populate `ModelState`, which can be checked in the action or handled automatically by `[ApiController]`.
- **Inference in APIs** – `[ApiController]` infers binding sources, reducing boilerplate.
- **Explicit override** – Attributes override inference when you need control.
- **Complex types** – POCOs can be bound from the body (JSON) or from form/query values (property-by-property).

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllers()` or `AddControllersWithViews()`).
- Basic understanding of **controllers**, **action methods**, and **HTTP requests**.
- Familiarity with **C# types** and **JSON** (for body binding).

### Related Programming Areas

- **Model Binding** – The subsystem that populates parameters from route/query/form/header.
- **Input Formatters** – Read the request body (JSON, XML) and deserialize to POCOs.
- **Model Validation** – Validates bound values via data annotations; populates `ModelState`.
- **Dependency Injection** – `[FromServices]` resolves parameters from the DI container.
- **API Controllers** – `[ApiController]` applies binding-source inference.
- **Filters** – Action filters can inspect/replace bound arguments.

### Core Concepts / Features

1. **Action Parameters (General)** – What they are and how the framework populates them.
2. **Model Binding** – The engine that maps request data to parameters.
3. **`[FromQuery]`** – Bind from the query string.
4. **`[FromRoute]`** – Bind from route values.
5. **`[FromBody]`** – Bind from the request body (JSON/XML).
6. **`[FromForm]`** – Bind from form data (`application/x-www-form-urlencoded` or `multipart/form-data`).
7. **`[FromHeader]`** – Bind from HTTP request headers.
8. **`[FromServices]`** – Bind from the DI container.
9. **Action Parameter Best Practices** – Guidelines for clear, safe, and maintainable parameter design.

Each concept is elaborated with Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Action Parameters (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action parameters are the **declared inputs** on a controller action method, populated automatically from the HTTP request before the action body executes. |
| **Technical Definition** | Action parameters are processed by the **action invoker** during the model binding stage. Each parameter is associated with a **binding source** (`BindingSource`). Simple types (int, string, DateTime, etc.) are bound from route/query/form/header by default; complex types are bound from the body in APIs or from form/query in MVC. After binding and validation, the parameters are passed to the action method. Binding failures are recorded in `ModelState`. |
| **Beginner-Friendly Explanation** | Action parameters are the **values your action needs to do its job**. If you're building a product detail page, you need a product ID. You declare `public IActionResult Details(int id)`, and the framework extracts `5` from `/products/5` and passes it as `id`. You don't parse the URL yourself—the framework does it for you. |

### Purposes (each begins with “To”)

- **To** provide action methods with the data they need without manual request parsing.
- **To** automatically convert request strings into strongly typed values (int, Guid, DateTime, POCOs).
- **To** validate incoming data and record errors in `ModelState` for consistent error handling.
- **To** keep actions clean by separating HTTP plumbing from business logic.
- **To** support multiple data sources (route, query, body, form, header, DI) with explicit attributes when needed.

### Syntax Rules and Structure

#### General Syntax

```csharp
public IActionResult ActionName(
    int id,                              // Bound from route/query by convention
    [FromQuery] int page,                // Explicit query binding
    [FromBody] CreateProductDto dto,     // Body binding (JSON)
    [FromHeader(Name = "X-Tenant")] string tenant, // Header binding
    [FromServices] IProductService svc)  // DI binding
{
    // ...
}
```

#### Component Breakdown

| Parameter style | Source | Notes |
|-----------------|--------|-------|
| Simple type without attribute | Route → Query → Form (MVC); Route → Query (API inference) | Depends on controller type. |
| Complex type without attribute (MVC) | Form → Route → Query (property-by-property) | Model binding aggregates values from multiple sources. |
| Complex type without attribute (API) | Body (JSON) — inferred | `[ApiController]` infers body for complex types. |
| `[FromQuery]`, `[FromRoute]`, `[FromForm]`, `[FromHeader]` | Explicit source | Overrides inference. |
| `[FromBody]` | Request body via input formatter | Requires `Content-Type: application/json` (or XML). |
| `[FromServices]` | DI container | Method-level DI. |

#### Syntax Rules

- **At most one `[FromBody]` parameter per action** – the request body can be read only once.
- **Simple types** are bound from route/query/form by default in MVC.
- **Complex types** are bound from the body by inference in `[ApiController]`; in MVC, they are bound from form/route/query values.
- **`[FromRoute]` requires the value to be present in the route**; otherwise binding fails (or defaults apply for nullable types).
- **Binding source attributes can be combined with model validation attributes** (`[Required]`, `[Range]`) on the parameter or the DTO's properties.

#### Constraints and Limitations

- **`[FromBody]` cannot be used for MVC form posts** – use `[FromForm]` instead.
- **Re-reading the body** – Once `[FromBody]` reads the request body, it cannot be read again in the same request.
- **Property-by-property binding** – Complex types bound from form/query require all needed properties to be present; missing values become defaults.
- **`[FromServices]` is discouraged** for dependencies used across actions; use constructor injection instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mixed Binding Sources

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{customerId:int}/orders")]
    public ActionResult<IEnumerable<OrderDto>> GetOrders(
        int customerId,                                      // From route
        [FromQuery] int page = 1,                            // From query
        [FromQuery] int pageSize = 20,                       // From query
        [FromHeader(Name = "X-Tenant-Id")] string tenantId)  // From header
    {
        return Ok(new { customerId, page, pageSize, tenantId });
    }
}
```

**Step-by-step explanation:**

1. `customerId` is bound from the `{customerId:int}` route segment.
2. `page` and `pageSize` are bound from `?page=2&pageSize=10`.
3. `tenantId` is bound from the `X-Tenant-Id` header.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/Orders/5/orders?page=2&pageSize=10` with header `X-Tenant-Id: acme` | `{ "customerId": 5, "page": 2, "pageSize": 10, "tenantId": "acme" }` |
| Missing `X-Tenant-Id` header | Depends on nullability; for non-nullable `string`, binding failure yields a 400 in `[ApiController]`. |

**Why this output?** Each attribute directs the binder to the correct source; missing required header values produce validation errors under `[ApiController]`.

### Real-World Cases

- **REST APIs** – Route IDs, query filters, JSON bodies, tenant headers, and injected services.
- **MVC pages** – Route IDs, query filters, form posts, and constructor-injected services.
- **File uploads** – `IFormFile` parameters bound from form data.

### References

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Binding source attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding#binding-source-parameter-attributes

---

## 2. Model Binding

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model binding is the process of **converting HTTP request data into .NET objects** (action parameters and their properties) using value providers, metadata, and type converters. |
| **Technical Definition** | Model binding in ASP.NET Core consists of: (1) **Value providers** that read data from route, query string, form, headers, and body; (2) **Model metadata** describing the target type (simple/complex, bindable, validation attributes); (3) **Type converters** (`TypeConverter`, `IModelBinder`) that parse strings into target types; (4) **Validation** that records errors in `ModelState`. Binding source attributes and `[ApiController]` inference determine which value provider is used for each parameter. |
| **Beginner-Friendly Explanation** | Model binding is the **automatic data-fetching assistant** for your action. It reads the URL, the query string, the form fields, the headers, and the JSON body, and matches those values to the parameters of your action. You don't have to write code to parse `id` out of `/products/5`—model binding does it and hands you an `int id`. |

### Purposes (each begins with “To”)

- **To** eliminate manual request parsing (`Request.Query["page"]`, `Request.Form["name"]`, JSON deserialization).
- **To** convert strings from the request into strongly typed values with consistent rules.
- **To** apply validation and record errors in a central place (`ModelState`).
- **To** support complex types (POCOs) whose properties come from multiple sources.
- **To** honour binding source attributes and inference so that behaviour is explicit and predictable.

### Syntax Rules and Structure

#### General Binding Flow (Conceptual)

```
For each parameter:
    1. Determine binding source (attribute → inference → default)
    2. Read raw value(s) from the value provider (route, query, form, header, body)
    3. Convert to target type (TypeConverter / formatter / nested binding)
    4. Validate (data annotations; ModelState entries on failure)
    5. Pass to action (or short-circuit if [ApiController] and invalid)
```

#### Component Breakdown

| Component | Role |
|-----------|------|
| **Value providers** | Read raw data: `RouteValueProvider`, `QueryStringValueProvider`, `FormValueProvider`, `HeaderValueProvider`, `BodyModelBinder` (JSON/XML). |
| **Model metadata** | Describes the parameter type; drives simple vs. complex binding and validation. |
| **Type converters / binders** | Convert strings to types (`int`, `DateTime`, `Guid`, enums, custom types). |
| **ModelState** | Collects binding and validation errors. |

#### Syntax Rules

- **Simple types** (primitives, `string`, `DateTime`, `Guid`, `decimal`) use `TypeConverter` for conversion.
- **Complex types** in MVC are bound property-by-property using value providers; in APIs, the body is used (via `BodyModelBinder`).
- **Binding order in MVC** (default): Form → Route → Query. In APIs with `[ApiController]`, inference overrides this per parameter.
- **Culture** – Binding uses the current culture by default; use `InvariantCulture` for route/header if needed via `[Bind]` or customisation.

#### Constraints and Limitations

- **Body can be read only once** – do not combine `[FromBody]` with form binding in the same action.
- **Complex-type binding from query requires full property set** – missing properties become defaults; partial updates are typically done with dedicated DTOs or PATCH.
- **Binding failures are not exceptions** – they are recorded in `ModelState`; the action must check (MVC) or `[ApiController]` auto-returns 400.
- **Culture sensitivity** – numeric and date binding depend on `CultureInfo.CurrentCulture` unless configured otherwise.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple-Type Binding in MVC

```csharp
public IActionResult Search(string q, int page = 1)
{
    return Content($"q={q}, page={page}");
}
```

**Step-by-step explanation:**

1. `q` is bound from the query string (`?q=routing`).
2. `page` is bound from the query string or defaults to `1`.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /Home/Search?q=routing&page=2` | `q=routing, page=2` |
| `GET /Home/Search?q=routing` | `q=routing, page=1` |

**Why this output?** Simple types use the query string provider; `int` conversion uses the current culture.

---

#### Example 2: Complex-Type Binding in MVC (Property-by-Property)

```csharp
public IActionResult Create(ProductFormModel model)
{
    if (!ModelState.IsValid) return View(model);
    _service.Create(model);
    return RedirectToAction(nameof(Index));
}
```

```csharp
public class ProductFormModel
{
    [Required] public string Name { get; set; }
    [Range(0.01, 10000)] public decimal Price { get; set; }
}
```

**Step-by-step explanation:**

1. The form model is bound from form fields (`Name`, `Price`).
2. Validation attributes run after binding.
3. `ModelState.IsValid` is checked; the view is re-rendered with errors if invalid.

**Expected Output:** Valid form → product created; invalid → view with validation messages.

**Why this output?** Complex types in MVC are bound property-by-property from value providers (form, route, query), then validated.

---

#### Example 3: Complex-Type Binding in API (`[FromBody]` Inference)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto) // Inferred [FromBody]
    {
        var product = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}
```

**Expected Output:** The JSON body is deserialized into `CreateProductDto`; invalid bodies yield a 400.

**Why this output?** With `[ApiController]`, complex-type parameters are inferred as `[FromBody]`, and the input formatter (System.Text.Json) deserializes the JSON.

### Real-World Cases

- **Search forms** – Query-bound models with multiple optional filters.
- **CRUD APIs** – JSON bodies deserialized into DTOs.
- **HTML forms** – Form-bound view models with validation.

### References

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Customize model binding – https://learn.microsoft.com/en-us/aspnet/core/mvc/advanced/custom-model-binding
- Model validation – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 3. `[FromQuery]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromQuery]` binds a parameter from the **query string** (`?key=value&other=…`). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromQueryAttribute` derives from `BindingSourceAttribute` with `BindingSource = BindingSource.Query`. It instructs the model binder to use the `QueryStringValueProvider`. For arrays and collections, the same key can appear multiple times (`?ids=1&ids=2`) or be comma-separated (`?ids=1,2`) depending on configuration. |
| **Beginner-Friendly Explanation** | `[FromQuery]` tells the framework: “Get this value from the part of the URL after the `?`.” For example, in `/products?page=2&pageSize=10`, `page=2` and `pageSize=10` are query parameters. |

### Purposes (each begins with “To”)

- **To** bind values from the query string explicitly, even when inference might choose a different source.
- **To** bind collections from repeated query keys (`?tag=a&tag=b`) or comma-separated values.
- **To** support filtering, paging, sorting, and search parameters in API endpoints.
- **To** keep query-string semantics explicit and self-documenting in action signatures.

### Syntax Rules and Structure

#### General Syntax

```csharp
[FromQuery] int page
[FromQuery(Name = "q")] string searchTerm
[FromQuery] int[] ids
[FromQuery] List<string> tags
[FromQuery] ProductFilter filter   // complex type bound property-by-property
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `Name` | Overrides the key looked up in the query string. | Useful for short names or keyword clashes. |
| Arrays/collections | Repeated keys or comma-separated values (depending on configuration). | In APIs with `[ApiController]`, arrays are common. |
| Complex types | Bound property-by-property from query keys. | Prefer a dedicated `Filter` DTO for clarity. |

#### Syntax Rules

- **Complex types** bound from query are bound **property-by-property**; each property must appear as a query key.
- **Missing query values** for non-nullable parameters produce binding errors.
- **Case-insensitive** key matching in ASP.NET Core (`page` matches `Page`).
- **No default value** – use C# default parameter values (`int page = 1`) for optional query values.

#### Constraints and Limitations

- **URL length limits** – large query strings can exceed server/proxy limits.
- **No nesting** – nested objects in query strings require custom binding or flat DTOs.
- **Sensitive data in URLs** – query strings are logged and visible in browsers; never put secrets there.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Paging

```csharp
[HttpGet]
public ActionResult<IEnumerable<ProductDto>> GetAll(
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 20)
{
    return Ok(_service.GetPage(page, pageSize));
}
```

**Expected Output:**

| Request | Effect |
|---------|--------|
| `GET /api/products?page=2&pageSize=10` | Returns page 2 (10 items) |
| `GET /api/products` | Defaults to page 1, size 20 |

**Why this output?** `[FromQuery]` binds from the query string; C# default values provide fallbacks.

---

#### Example 2: Repeated Query Keys

```csharp
[HttpGet("search")]
public ActionResult Search([FromQuery] string[] tag)
{
    return Ok(tag);
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products/search?tag=new&tag=sale` | `["new", "sale"]` |

**Why this output?** Repeated keys are collected into the array; in `[ApiController]` this is supported by default; customising comma-separated behaviour requires `QueryStringValueProviderFactory` options.

---

#### Example 3: Renamed Query Key

```csharp
[HttpGet]
public ActionResult Search([FromQuery(Name = "q")] string searchTerm)
{
    return Ok(searchTerm);
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products?q=laptop` | `"laptop"` |

**Why this output?** `Name = "q"` maps the `q` query key to `searchTerm`.

### Real-World Cases

- **Pagination** – `?page=2&pageSize=50`.
- **Filtering** – `?status=active&category=electronics`.
- **Search** – `?q=routing&sort=relevance`.
- **Multi-select filters** – `?tag=a&tag=b`.

### References

- `FromQueryAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromqueryattribute
- Binding source attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding#binding-source-parameter-attributes

---

## 4. `[FromRoute]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromRoute]` binds a parameter from a **route value** captured by the route template (e.g., `{id}` in `products/{id}`). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromRouteAttribute` derives from `BindingSourceAttribute` with `BindingSource = BindingSource.Path`. It instructs the binder to use the `RouteValueProvider`. Values are populated by the routing middleware before model binding. In `[ApiController]`, route parameters are typically inferred when the parameter name matches a route token. |
| **Beginner-Friendly Explanation** | `[FromRoute]` tells the framework: “Get this value from the **URL path** itself.” In `products/{id}`, the `{id}` is a route parameter, so `/products/5` gives you `id = 5`. |

### Purposes (each begins with “To”)

- **To** bind resource identifiers captured in the URL path (e.g., `{id}`, `{slug}`, `{customerId}`).
- **To** explicitly declare route-based binding when inference is ambiguous.
- **To** support hierarchical URLs with multiple route values (`/customers/{customerId}/orders/{orderId}`).
- **To** leverage route constraints (`{id:int}`, `{id:guid}`) to ensure clean values reach the action.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpGet("{id:int}")]
public IActionResult GetById([FromRoute] int id) { ... }
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `[FromRoute]` | Route value provider | Requires the key to exist in the route template. |
| `[FromRoute(Name = "x")]` | Custom route key | Maps the `x` route token to the parameter. |

#### Syntax Rules

- **Route token name must match** the parameter name or `Name` argument; otherwise binding fails.
- **Route constraints** (`{id:int}`) ensure only valid formats match; the action receives clean values.
- **Multiple route values** are supported; e.g., `[FromRoute] int customerId, [FromRoute] int orderId`.
- **In `[ApiController]`** , parameters matching route tokens are inferred as route-bound.

#### Constraints and Limitations

- **Route values are URL-encoded** – special characters may need normalisation (e.g., slugs).
- **Don't expose sensitive values in routes** – routes are logged; avoid secrets.
- **Route ordering** – route constraints and template specificity determine which action matches.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Single Route Parameter

```csharp
[HttpGet("{id:int}")]
public IActionResult GetById([FromRoute] int id)
{
    return Ok(id);
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products/5` | `5` |
| `GET /api/products/abc` | 404 (route constraint fails) |

**Why this output?** The `int` constraint ensures only integers reach the action; `[FromRoute]` binds the captured value.

---

#### Example 2: Multiple Route Values

```csharp
[HttpGet("customers/{customerId:int}/orders/{orderId:int}")]
public IActionResult GetOrder(
    [FromRoute] int customerId,
    [FromRoute] int orderId)
{
    return Ok(new { customerId, orderId });
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products/customers/5/orders/10` | `{ "customerId": 5, "orderId": 10 }` |

**Why this output?** Each route token is captured by the route template and bound to the corresponding parameter.

---

#### Example 3: Slug Route Parameter

```csharp
[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost([FromRoute] string slug) => Ok(slug);
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /blog/my-first-post` | `"my-first-post"` |
| `GET /blog/My-Post` | 404 (regex constraint fails) |

**Why this output?** The regex constraint enforces URL-safe slug format; `[FromRoute]` binds the captured slug.

### Real-World Cases

- **Resource IDs** – `products/{id:int}`, `users/{id:guid}`.
- **Hierarchical resources** – `/customers/{customerId}/orders/{orderId}`.
- **Slugs for SEO** – `/blog/{slug}`.

### References

- `FromRouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters

---

## 5. `[FromBody]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromBody]` binds a parameter from the **request body** using an **input formatter** (typically JSON via System.Text.Json). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromBodyAttribute` binds the parameter using the `BodyModelBinder`, which selects an input formatter based on `Content-Type` (e.g., `application/json`). The formatter deserializes the body into the parameter type. Binding failures are added to `ModelState` (and yield 400 in `[ApiController]`). Only **one** `[FromBody]` parameter is permitted per action because the body stream can be read once. |
| **Beginner-Friendly Explanation** | `[FromBody]` tells the framework: “Read the **JSON payload** sent by the client and turn it into a C# object.” When a POST request sends `{ "name": "Laptop", "price": 999 }`, `[FromBody]` deserializes it into a `ProductDto` and passes it to your action. |

### Purposes (each begins with “To”)

- **To** deserialize JSON (or XML) request bodies into strongly typed DTOs.
- **To** support REST API POST/PUT/PATCH operations with structured payloads.
- **To** apply JSON-specific validation (e.g., `[Required]`, `[Range]` on DTO properties) via `ModelState`.
- **To** integrate with `[ApiController]` for automatic binding source inference on complex types.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpPost]
public IActionResult Create([FromBody] CreateProductDto dto) { ... }
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `[FromBody]` | Request body | Requires `Content-Type: application/json` (or XML). |
| Complex type | Input formatter | System.Text.Json by default. |
| Only one per action | Stream can be read once. | |

#### Syntax Rules

- **`Content-Type` must match the formatter**; otherwise a 415 Unsupported Media Type results.
- **Only one `[FromBody]` parameter per action**.
- **`[ApiController]` infers `[FromBody]`** for complex types, so the attribute is often optional in API controllers.
- **Non-JSON bodies** – register XML or other formatters or use `[Consumes]` for content negotiation.

#### Constraints and Limitations

- **Cannot combine with `[FromForm]`** in the same action – the body cannot be read twice.
- **No partial body reads** – malformed JSON produces 400 and the action is not executed.
- **OpenAPI/Swagger** – requires proper DTO types for schema generation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Explicit `[FromBody]` (MVC Controller)

```csharp
[HttpPost]
public IActionResult Create([FromBody] CreateProductDto dto)
{
    if (!ModelState.IsValid) return BadRequest(ModelState);
    var product = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `POST /products` with valid JSON | 201 Created with product |
| `POST /products` with invalid JSON | 400 Bad Request with validation details |

**Why this output?** `[FromBody]` deserializes the JSON; validation attributes on the DTO populate `ModelState`.

---

#### Example 2: Inferred `[FromBody]` in `[ApiController]`

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto) // Inferred [FromBody]
    {
        var product = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}
```

**Expected Output:** The same as Example 1, with automatic 400 on invalid models.

**Why this output?** `[ApiController]` infers `[FromBody]` for complex types, so no explicit attribute is needed.

---

#### Example 3: JSON Patch (`JsonPatchDocument<T>`)

```csharp
[HttpPatch("{id:int}")]
public IActionResult Patch(int id, [FromBody] JsonPatchDocument<ProductDto> patch)
{
    var dto = _service.GetById(id);
    if (dto == null) return NotFound();
    patch.ApplyTo(dto, ModelState);
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    _service.Update(id, dto);
    return NoContent();
}
```

**Expected Output:** Applies a JSON Patch document to the resource; invalid patches return 400.

**Why this output?** `[FromBody]` binds the `JsonPatchDocument` (application/json-patch+json) and `ApplyTo` mutates the DTO.

### Real-World Cases

- **REST resource creation** – `POST /api/products` with JSON body.
- **Updates** – `PUT /api/products/{id}` with JSON body.
- **JSON Patch** – `PATCH` with `application/json-patch+json`.

### References

- `FromBodyAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute
- Input formatters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- `[ApiController]` binding source inference – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#binding-source-parameter-inference

---

## 6. `[FromForm]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromForm]` binds a parameter from **form data** submitted with `application/x-www-form-urlencoded` or `multipart/form-data` content types. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromFormAttribute` derives from `BindingSourceAttribute` with `BindingSource = BindingSource.Form`. It uses the `FormValueProvider`, which reads `Request.Form`. For file uploads, parameters of type `IFormFile` or `IFormFileCollection` are populated from the multipart form data. Binding only succeeds if the request uses a form content type. |
| **Beginner-Friendly Explanation** | `[FromForm]` tells the framework: “Read the **HTML form fields** the user submitted.” When a user fills out a form and hits Submit, the browser sends the fields as form data. `[FromForm]` binds those values (and uploaded files) to your action parameters. |

### Purposes (each begins with “To”)

- **To** bind HTML form submissions (text fields, dropdowns, checkboxes) into strongly typed models.
- **To** handle file uploads via `IFormFile` / `IFormFileCollection`.
- **To** support MVC view pages that post forms (`<form method="post">`).
- **To** differentiate form-based posts from JSON body posts in the same API.
- **To** apply validation attributes to form models for server-side validation.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpPost]
public IActionResult Create([FromForm] ProductFormModel model) { ... }

[HttpPost("upload")]
public IActionResult Upload([FromForm] IFormFile file, [FromForm] string description) { ... }
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `[FromForm]` | Form value provider | Requires `Content-Type: multipart/form-data` or `application/x-www-form-urlencoded`. |
| `IFormFile` | Multipart form data | Bound by the `FormFileValueProvider`. |
| `IFormFileCollection` | Multiple files | Bound from all files in the multipart request. |

#### Syntax Rules

- **Use `multipart/form-data` for file uploads**; `application/x-www-form-urlencoded` cannot send files.
- **`[FromForm]` and `[FromBody]` cannot be combined** in the same action.
- **`[ApiController]` does not infer `[FromForm]`** – you must annotate explicitly.
- **Complex form models** are bound property-by-property.

#### Constraints and Limitations

- **Request body can be read only once** – don't combine with `[FromBody]`.
- **File size limits** – default request body limits apply; configure `KestrelServerOptions.Limits.MaxRequestBodySize` and multipart limits.
- **Antiforgery tokens** – browser-facing forms should include and validate antiforgery tokens.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Form Post (MVC)

```csharp
[HttpPost]
public IActionResult Create([FromForm] ProductFormModel model)
{
    if (!ModelState.IsValid) return View(model);
    _service.Create(model);
    return RedirectToAction(nameof(Index));
}
```

```csharp
public class ProductFormModel
{
    [Required] public string Name { get; set; }
    [Range(0.01, 10000)] public decimal Price { get; set; }
}
```

```cshtml
<form method="post" asp-action="Create">
    <input asp-for="Name" />
    <span asp-validation-for="Name"></span>
    <input asp-for="Price" />
    <span asp-validation-for="Price"></span>
    <button type="submit">Save</button>
</form>
```

**Expected Output:** Valid form → redirect to Index; invalid → re-render form with messages.

**Why this output?** `[FromForm]` binds form fields to the model; validation populates `ModelState`.

---

#### Example 2: File Upload

```csharp
[HttpPost("upload")]
public async Task<IActionResult> Upload(
    [FromForm] IFormFile file,
    [FromForm] string description)
{
    if (file == null || file.Length == 0) return BadRequest("File required");
    var path = Path.Combine("uploads", Path.GetFileName(file.FileName));
    await using var stream = System.IO.File.Create(path);
    await file.CopyToAsync(stream);
    return Ok(new { description, file.Length });
}
```

**Expected Output:** `POST /upload` with `multipart/form-data` containing `file` and `description` saves the file and returns details.

**Why this output?** `IFormFile` binds the uploaded file from multipart form data; `description` binds from the same form.

---

#### Example 3: Multiple Files

```csharp
[HttpPost("bulk-upload")]
public async Task<IActionResult> BulkUpload([FromForm] IFormFileCollection files)
{
    foreach (var file in files)
    {
        await using var stream = System.IO.File.Create(Path.Combine("uploads", file.FileName));
        await file.CopyToAsync(stream);
    }
    return Ok(files.Count);
}
```

**Expected Output:** `POST /bulk-upload` with multiple files returns the count of uploaded files.

**Why this output?** `IFormFileCollection` binds all uploaded files from the multipart request.

### Real-World Cases

- **Contact forms** – Name, email, message fields.
- **File uploads** – Avatars, documents, CSV imports.
- **Admin panels** – Form-based CRUD with validation.
- **Newsletter signup** – Simple `email` form field.

### References

- `FromFormAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute
- File uploads in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads
- Request size limits – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options

---

## 7. `[FromHeader]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromHeader]` binds a parameter from an **HTTP request header**. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromHeaderAttribute` derives from `BindingSourceAttribute` with `BindingSource = BindingSource.Header`. It uses the `HeaderValueProvider` (if registered) to read `Request.Headers`. In API controllers, headers are not inferred; explicit annotation (or `[FromHeader(Name = "…")]`) is required. For missing headers, binding fails for non-nullable parameters and produces 400 under `[ApiController]`. |
| **Beginner-Friendly Explanation** | `[FromHeader]` tells the framework: “Get this value from the request **headers**.” Headers carry metadata like `Authorization`, `X-Tenant-Id`, `Accept-Language`, and `If-Match`. `[FromHeader]` lets you expose a header directly as a parameter. |

### Purposes (each begins with “To”)

- **To** bind metadata (tenant ID, correlation ID, API version, locale) carried in headers.
- **To** support versioning or multi-tenancy via custom headers.
- **To** allow conditional requests (e.g., `If-Match`, `If-None-Match`) to be handled explicitly.
- **To** provide a cleaner alternative to reading `Request.Headers["…"]` inside the action.

### Syntax Rules and Structure

#### General Syntax

```csharp
[FromHeader(Name = "X-Tenant-Id")] string tenantId
[FromHeader] string acceptLanguage
[FromHeader] Guid correlationId
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `Name` | Header name | Case-insensitive match; hyphens allowed. |
| Collection types | Multiple header values | Bound if the header appears multiple times. |

#### Syntax Rules

- **Header name is required** if it doesn't match the parameter name (e.g., hyphens).
- **Missing header** for non-nullable parameter → binding failure (400 in `[ApiController]`).
- **Case-insensitive** header matching.
- **No inference** – `[FromHeader]` must be explicit, even in `[ApiController]`.

#### Constraints and Limitations

- **Header size limits** – Kestrel limits total header size; huge headers can be rejected.
- **Custom headers may be stripped by proxies** – verify behavior in deployment.
- **Don't put secrets in non-secure channels** – headers are visible to intermediaries (use HTTPS).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tenant ID Header

```csharp
[HttpGet]
public IActionResult Get(
    [FromHeader(Name = "X-Tenant-Id")] string tenantId)
{
    return Ok(new { tenantId });
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/data` with `X-Tenant-Id: acme` | `{ "tenantId": "acme" }` |
| Missing header | 400 (in `[ApiController]`) |

**Why this output?** `[FromHeader]` reads the header from the request; missing non-nullable headers produce binding errors.

---

#### Example 2: Correlation ID

```csharp
[HttpGet]
public IActionResult Get(
    [FromHeader(Name = "X-Correlation-Id")] Guid? correlationId)
{
    return Ok(new { correlationId = correlationId ?? Guid.NewGuid() });
}
```

**Expected Output:** Returns the client-provided GUID or a new one if absent.

**Why this output?** Nullable `Guid?` allows the header to be optional.

---

#### Example 3: `If-None-Match` (ETag)

```csharp
[HttpGet("{id:int}")]
public IActionResult Get(
    int id,
    [FromHeader(Name = "If-None-Match")] string? ifNoneMatch)
{
    var etag = _service.GetETag(id);
    if (ifNoneMatch == etag) return StatusCode(304);
    Response.Headers.ETag = etag;
    return Ok(_service.Get(id));
}
```

**Expected Output:** Returns 304 Not Modified when the ETag matches; otherwise, 200 with the resource.

**Why this output?** The header parameter enables conditional request handling directly in the action.

### Real-World Cases

- **Multi-tenant APIs** – `X-Tenant-Id`.
- **Distributed tracing** – `X-Correlation-Id` for log correlation.
- **Conditional GET** – `If-None-Match` with ETags.
- **Localisation** – `Accept-Language` to select culture.

### References

- `FromHeaderAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromheaderattribute
- Header-based versioning (ASP.NET API Versioning) – https://github.com/dotnet/aspnet-api-versioning

---

## 8. `[FromServices]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromServices]` binds a parameter from the **dependency injection (DI) container** at action invocation time. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FromServicesAttribute` derives from `BindingSourceAttribute` with `BindingSource = BindingSource.Services`. The action invoker resolves the parameter type from `HttpContext.RequestServices`. This enables **method-level** dependency injection without requiring a constructor. It is functionally similar to constructor injection for that one action; Microsoft recommends constructor injection for shared dependencies. |
| **Beginner-Friendly Explanation** | `[FromServices]` tells the framework: “Get this object from the **service container** and pass it in.” If an action needs a service (like a repository or email sender) only for that one action, you can add a `[FromServices]` parameter instead of adding a constructor dependency. |

### Purposes (each begins with “To”)

- **To** inject a service **only** where needed, avoiding controller-wide dependencies that aren't used by most actions.
- **To** enable lightweight controllers that resolve dependencies per-action.
- **To** support scenarios where the service lifetime is **transient or scoped** and shouldn't be held by the controller.
- **To** simplify refactoring when only one action requires a specific service.

### Syntax Rules and Structure

#### General Syntax

```csharp
public IActionResult Action([FromServices] IEmailSender emailSender)
{
    emailSender.Send(...);
    return Ok();
}
```

#### Component Breakdown

| Parameter | Source | Notes |
|-----------|--------|-------|
| `[FromServices]` | DI container | Resolved via `HttpContext.RequestServices`. |
| Service type | Registered in `Program.cs` | Must be registered; otherwise, runtime exception. |
| Lifetime | Scoped / transient / singleton | Same rules as constructor injection. |

#### Syntax Rules

- **The type must be registered** in DI; otherwise, the action will throw `InvalidOperationException`.
- **No inference** – `[FromServices]` must be explicit.
- **Not a substitute for constructor injection** for dependencies used across many actions.

#### Constraints and Limitations

- **Harder to unit-test** – constructor injection is more transparent; `[FromServices]` dependencies must be supplied through `RequestServices` in tests.
- **Hides dependencies** – action signatures become noisy when many `[FromServices]` parameters are used.
- **Microsoft guidance** – prefer constructor injection; use `[FromServices]` only when a service is truly action-specific.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Action-Specific Service

```csharp
public class ReportsController : Controller
{
    private readonly IReportService _reports;
    public ReportsController(IReportService reports) => _reports = reports;

    public IActionResult Index() => View();

    [HttpPost("export")]
    public async Task<IActionResult> Export([FromServices] ICsvExporter csvExporter)
    {
        var stream = await csvExporter.ExportAsync(_reports.GetAll());
        return File(stream, "text/csv", "reports.csv");
    }
}
```

**Expected Output:** `POST /Reports/Export` returns a CSV file.

**Why this output?** `ICsvExporter` is only needed for the export action, so `[FromServices]` avoids adding it to the constructor.

---

#### Example 2: Scoped Service Per Action

```csharp
[HttpPost]
public IActionResult Checkout(
    CheckoutDto dto,
    [FromServices] IPaymentGateway gateway,
    [FromServices] ILogger<CheckoutController> logger)
{
    logger.LogInformation("Checkout started for {UserId}", dto.UserId);
    var result = gateway.Charge(dto.Amount);
    return result.Success ? Ok(result) : BadRequest(result.Error);
}
```

**Expected Output:** Returns 200 with result or 400 with error.

**Why this output?** Services are resolved per-action; logging and payment gateway are injected only where needed.

### Real-World Cases

- **Reporting endpoints** – CSV/PDF exporters needed only for a few actions.
- **Admin-only actions** – specialized services used by single endpoints.
- **Conditional heavy services** – expensive services resolved only when the action is invoked.

### References

- `FromServicesAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

---

## 9. Action Parameter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action parameter best practices are guidelines for declaring and binding parameters in ways that are **clear, safe, testable, and semantically correct**, leveraging explicit sources and inference appropriately. |
| **Technical Definition** | Best practices include: preferring explicit binding source attributes when inference is unclear, using DTOs for complex inputs, avoiding `[FromBody]`+`[FromForm]` combinations, favouring constructor injection over `[FromServices]` for shared dependencies, validating with data annotations, choosing `ActionResult<T>` for typed API signatures, and avoiding `[FromHeader]` for large/secret data. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep parameter binding predictable and clean. For example: use a DTO for form or JSON payloads, put IDs in routes, use `[FromQuery]` for filters and paging, don't put secrets in headers or URLs, and prefer constructor injection over method-level injection unless the dependency is action-specific. |

### Purposes (each begins with “To”)

- **To** make parameter binding **explicit and predictable** across the codebase.
- **To** reduce ambiguity and accidental binding errors (especially between MVC and API controllers).
- **To** keep controllers testable by favouring constructor injection for shared dependencies.
- **To** ensure security (no secrets in URLs, headers, or logs).
- **To** align with REST/HTTP semantics (IDs in routes, filters in query, payloads in body).
- **To** provide accurate OpenAPI documentation for API parameters.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Prefer explicit source attributes when inference is unclear** | Avoids subtle bugs between MVC and API behaviour. | `[FromQuery] int page` in API actions |
| **Use DTOs for complex inputs** | Encapsulates validation and shape. | `CreateProductDto` for POST bodies |
| **Only one `[FromBody]` per action** | Body stream is single-read. | Combine multiple DTO needs into one wrapper or separate endpoints |
| **Don't mix `[FromBody]` and `[FromForm]`** | Conflicting content types and single-read stream. | Separate upload endpoints for JSON vs. form |
| **Put IDs in routes, filters in query, payloads in body** | Follows HTTP semantics. | `/api/products/{id}` + `?page=2` + JSON body for POST |
| **Prefer constructor injection over `[FromServices]`** | Clearer dependencies, easier tests. | `HomeController(IProductService svc)` |
| **Use `[FromServices]` only for action-specific dependencies** | Avoids noisy constructors for rarely-used services. | CSV exporter used only by one endpoint |
| **Validate input via data annotations on DTOs** | Consistent with `ModelState` and Problem Details. | `[Required]`, `[Range]` on properties |
| **Avoid secrets in query strings/headers** | URLs and headers may be logged or exposed. | Send secrets in the body over HTTPS |
| **Prefer `[FromHeader]` with `Name` for hyphenated headers** | Parameter names cannot contain hyphens. | `[FromHeader(Name = "X-Tenant-Id")] string tenantId` |
| **Bound the size of collections** | Prevent DoS via huge payloads or arrays. | `[FromQuery] int[] ids` with `.Take(50)` or validation |
| **Leverage `[ApiController]` for automatic 400s** | Consistent error shape via Problem Details. | Annotate API controllers with `[ApiController]` |
| **Document parameters with XML comments / `[FromQuery(Name = "...")]`** | Improves API docs and client generation. | `<param name="page">...` with `[FromQuery]` |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Well-Designed API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(IProductService service, ILogger<ProductsController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet]
    public ActionResult<IEnumerable<ProductDto>> GetAll(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20,
        [FromQuery] string? search = null)
    {
        pageSize = Math.Clamp(pageSize, 1, 100);
        return Ok(_service.GetPage(page, pageSize, search));
    }

    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById([FromRoute] int id)
    {
        var product = _service.GetById(id);
        return product is null ? NotFound() : product;
    }

    [HttpPost]
    public ActionResult<ProductDto> Create([FromBody] CreateProductDto dto)
    {
        var product = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    [HttpPost("import")]
    public async Task<IActionResult> Import(
        [FromForm] IFormFile csv,
        [FromHeader(Name = "X-Tenant-Id")] string tenantId,
        [FromServices] ICsvImporter importer)
    {
        using var stream = csv.OpenReadStream();
        var count = await importer.ImportAsync(stream, tenantId);
        return Ok(new { imported = count });
    }
}
```

**Step-by-step explanation:**

1. Constructor injection supplies shared services (`_service`, `_logger`).
2. Query parameters are explicitly bound with `[FromQuery]`; `pageSize` is clamped for safety.
3. Route parameter `id` is explicitly bound; constraints ensure integer IDs.
4. Body parameter `dto` uses `[FromBody]` explicitly (though inference would apply).
5. Header `X-Tenant-Id` is explicitly bound; `[FromServices]` resolves the importer for that one action.

**Expected Output:**

| Endpoint | Behaviour |
|----------|-----------|
| `GET /api/products?page=2&pageSize=1000` | pageSize clamped to 100 |
| `GET /api/products/abc` | 404 (constraint fails) |
| `POST /api/products` with valid JSON | 201 Created |
| `POST /api/products/import` with CSV and tenant header | 200 with import count |

**Why this output?** Explicit binding sources make behaviour unambiguous; clamping and constraints prevent abuse; constructor + action-specific DI is balanced.

---

#### Example 2: MVC Form with Validation and Antiforgery

```csharp
public class AccountController : Controller
{
    private readonly IAccountService _accounts;
    public AccountController(IAccountService accounts) => _accounts = accounts;

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Register([FromForm] RegisterViewModel model)
    {
        if (!ModelState.IsValid) return View(model);
        _accounts.Register(model);
        return RedirectToAction(nameof(Index), "Home");
    }
}
```

**Expected Output:** Valid form → redirect; invalid → re-render with messages.

**Why this output?** `[FromForm]` binds form fields; `[ValidateAntiForgeryToken]` protects against CSRF; validation is checked in the action.

---

#### Example 3: Action-Specific Dependency via `[FromServices]`

```csharp
public IActionResult SendDigest([FromServices] IDigestService digest)
{
    digest.QueueAll();
    return Accepted();
}
```

**Expected Output:** `202 Accepted`.

**Why this output?** `[FromServices]` avoids adding `IDigestService` to the controller constructor when only one action uses it.

### Real-World Cases

- **REST APIs** – Explicit `[FromQuery]`, `[FromRoute]`, `[FromBody]` for clarity and docs.
- **Multi-tenant SaaS** – `[FromHeader]` tenant ID with middleware-based fallback.
- **Admin imports** – `[FromForm]` + `[FromServices]` for file importers.
- **Public forms** – `[FromForm]` with antiforgery and validation.

### References

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Binding source parameter inference – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#binding-source-parameter-inference
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- File uploads in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads
- Antiforgery in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery

---

## Summary Table: Binding Sources at a Glance

| Attribute | Source | Typical Use | Inference (`[ApiController]`) |
|-----------|--------|-------------|-------------------------------|
| *(no attribute)* | Depends on type/controller | Simple types from route/query; complex types from form (MVC) or body (API) | Yes |
| **`[FromQuery]`** | Query string | Paging, filtering, search | No |
| **`[FromRoute]`** | URL path | Resource IDs | Yes (matching route token) |
| **`[FromBody]`** | JSON/XML body | POST/PUT payloads | Yes (complex types) |
| **`[FromForm]`** | Form data | HTML forms, file uploads | No |
| **`[FromHeader]`** | HTTP headers | Tenant, correlation, locale | No |
| **`[FromServices]`** | DI container | Action-specific services | No |

---

## References (Full List)

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Binding source parameter attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding#binding-source-parameter-attributes
- Create web APIs with ASP.NET Core (binding source inference) – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#binding-source-parameter-inference
- Customize model binding – https://learn.microsoft.com/en-us/aspnet/core/mvc/advanced/custom-model-binding
- Model validation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Input formatters in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- File uploads in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Antiforgery in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Kestrel request limits – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- `FromQueryAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromqueryattribute
- `FromRouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute
- `FromBodyAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute
- `FromFormAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute
- `FromHeaderAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromheaderattribute
- `FromServicesAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute
- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors