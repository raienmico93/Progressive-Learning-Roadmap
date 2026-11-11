# C# Web API — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A **Web API** in C# is an HTTP-based service, typically built with ASP.NET Core, that exposes application functionality over the web using RESTful principles. It accepts HTTP requests, processes them, and returns structured responses (usually JSON) that can be consumed by any client—browsers, mobile apps, SPAs, or other services.

**Technical Definition:** ASP.NET Core Web API is a framework for building HTTP services using controllers derived from `ControllerBase` or Minimal APIs. Controllers are activated and disposed on a per-request basis . The framework supports **REST (Representational State Transfer)**, an architectural style that uses HTTP verbs (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) to operate on resources identified by URLs. Web API controllers typically derive from `ControllerBase` (not `Controller`, which adds view support for web pages) . The `[ApiController]` attribute enables opinionated API behaviours such as automatic model validation and binding source inference . Routing is configured via `[Route]` and HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.) . API versioning is supported through libraries such as `Asp.Versioning.Mvc`, and documentation is generated via **OpenAPI** (formerly Swagger) using tools like **Swashbuckle** or **NSwag** .

**Beginner-Friendly Explanation:** A Web API is **a way to let other software talk to your C# application over the internet**. Instead of building a website that humans click on, you build endpoints that other programs call. For example, a mobile app might call `GET /api/products` to fetch a product list, or `POST /api/orders` to create an order. You write controller classes with methods that handle these requests and return JSON. The framework handles the plumbing—routing, serialization, validation, and documentation—so you can focus on business logic.

---

### Key Characteristics

- **HTTP-centric** — Every operation is an HTTP request; verbs carry semantic meaning (`GET` reads, `POST` creates, `PUT` replaces, `PATCH` updates partially, `DELETE` removes) .
- **Resource-oriented** — URLs identify resources (`/api/products/42`), not actions.
- **Stateless** — Each request contains all the information needed; the server does not retain client state between requests.
- **JSON-first** — Responses are typically JSON (`application/json`), though XML and other formats are supported.
- **Controller-based or Minimal APIs** — Two programming models; controllers offer structure, Minimal APIs offer brevity .
- **Attribute-driven** — `[ApiController]`, `[Route]`, `[HttpGet]`, etc., configure routing, binding, and behaviour .
- **Versionable** — APIs can evolve without breaking existing clients via versioning strategies .
- **Documentable** — OpenAPI/Swagger generates interactive documentation automatically .

---

### Prerequisites

- Solid understanding of **C#** (classes, interfaces, async/await, generics, attributes).
- Familiarity with **ASP.NET Core fundamentals** (`Program.cs`, dependency injection, middleware).
- Knowledge of **HTTP** (verbs, status codes, headers, request/response bodies).
- Awareness of **JSON serialization** (`System.Text.Json`).
- Basic understanding of **REST principles** (resources, statelessness, uniform interface).

---

### Related Programming Areas

- **REST API design** — Resource modelling, idempotency, and HTTP semantics .
- **OpenAPI/Swagger** — Contract-first API documentation and client generation .
- **API versioning** — Evolving APIs without breaking clients .
- **Authentication and authorization** — JWT, OAuth, API keys.
- **Minimal APIs** — A lower-ceremony alternative for HTTP APIs .

---

### Core Concepts / Features

1. Web API (Overarching Concept)
2. REST APIs
3. `[ApiController]`
4. `[Route]`
5. `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]`
6. API Versioning
7. API Documentation (Swagger / OpenAPI)
8. Web API Best Practices

---

## 1. Web API (Overarching Concept)

### Definitions

**Core Definition:** A Web API is an HTTP service built with ASP.NET Core that exposes application logic to external clients via standard HTTP methods.

**Technical Definition:** ASP.NET Core supports creating web APIs using controllers or Minimal APIs. Controller-based APIs consist of one or more classes deriving from `ControllerBase`, activated per request . The framework provides routing, model binding, validation, content negotiation, and result execution. Web API controllers should derive from `ControllerBase` rather than `Controller`; `Controller` derives from `ControllerBase` and adds view support for web pages .

**Beginner-Friendly Explanation:** A Web API is **a web service that returns data, not web pages**. Other programs call its URLs and get JSON back.

---

### Purposes

- To expose business logic to **multiple clients** (mobile, web, desktop) via a single HTTP interface.
- To enable **service-to-service communication** in microservices architectures.
- To provide a **platform-agnostic** contract (any language that can make HTTP calls can consume it).
- To leverage **HTTP semantics** for caching, idempotency, and status reporting .

---

### Syntax Rules and Structure

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_products);

    [HttpGet("{id:int}")]
    public ActionResult<Product> GetById(int id) =>
        _products.FirstOrDefault(p => p.Id == id) is { } p ? Ok(p) : NotFound();
}
```

**Component breakdown**

- `[ApiController]` — Enables API-specific behaviours .
- `[Route("api/[controller]")]` — Sets the route prefix .
- `ControllerBase` — Base class providing HTTP helpers (`Ok`, `NotFound`, `BadRequest`, `CreatedAtAction`) .
- `ActionResult<T>` — Return type supporting both success and failure results.

#### Syntax Rules

- Web API controllers derive from `ControllerBase`.
- `[ApiController]` is applied at the controller level.
- Routes are defined via `[Route]` and HTTP verb attributes.
- Actions return `ActionResult<T>`, `IActionResult`, or `T`.

#### Constraints and Limitations

- Controllers are per-request; do not store request-scoped state in fields.
- API controllers should not use view-related helpers (`View`, `PartialView`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Complete CRUD Web API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ReservationsController : ControllerBase
{
    private static readonly List<Reservation> _store = new();

    [HttpGet]
    public ActionResult<IEnumerable<Reservation>> List([FromQuery] string? filter) =>
        Ok(_store.Where(r => filter is null || r.Status == filter));

    [HttpGet("{id:int}")]
    public ActionResult<Reservation> Get(int id) =>
        _store.FirstOrDefault(r => r.Id == id) is { } r ? Ok(r) : NotFound();

    [HttpPost]
    public ActionResult<Reservation> Create([FromBody] CreateReservation request)
    {
        var reservation = new Reservation { Id = _store.Count + 1, Status = "open" };
        _store.Add(reservation);
        return CreatedAtAction(nameof(Get), new { id = reservation.Id }, reservation);
    }

    [HttpPut("{id:int}")]
    public IActionResult Put(int id, [FromBody] UpdateReservation request) =>
        _store.Any(r => r.Id == id) ? NoContent() : NotFound();

    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id) =>
        _store.RemoveAll(r => r.Id == id) > 0 ? NoContent() : NotFound();
}
```

**Expected Output (conceptual)**
```
GET    /api/Reservations           → 200 OK with JSON array
GET    /api/Reservations/1         → 200 OK or 404 Not Found
POST   /api/Reservations           → 201 Created with Location header
PUT    /api/Reservations/1         → 204 No Content or 404
DELETE /api/Reservations/1         → 204 No Content or 404
```

**Why this output occurs:** Each action is mapped to a URL and HTTP verb; `ControllerBase` helpers produce the correct status codes and payloads .

---

### Real-World Cases

- **E-commerce APIs** — Product catalogue, cart, orders.
- **Mobile backends** — JSON APIs consumed by iOS/Android apps.
- **Microservices** — Lightweight HTTP services communicating over the network.
- **Partner integrations** — Public APIs documented with OpenAPI.

---

### References

- Create web APIs with ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/71c331669b8a6c1acfea6db3420ad6399ca018dc/aspnetcore/tutorials/first-web-api.md
- Create web APIs with ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/web-api/

---

## 2. REST APIs

### Definitions

**Core Definition:** REST (Representational State Transfer) is an architectural style for building APIs that uses HTTP verbs to operate on resources identified by URLs.

**Technical Definition:** RESTful APIs preserve HTTP method semantics and implement idempotency. `GET` and `HEAD` return resource data and response headers without a body; they must not create, update, or delete resources. `GET`, `HEAD`, `PUT`, and `DELETE` are idempotent—repeating an identical request has the same intended effect as performing it once. `POST` and `PATCH` are not inherently idempotent . REST APIs should validate request data, control field access with DTOs, and limit side effects of `POST` actions to the created resource and directly related resources .

**Beginner-Friendly Explanation:** REST is **a set of rules for building APIs that feel natural**. Use `GET` to read, `POST` to create, `PUT` to replace, `PATCH` to update, and `DELETE` to remove. URLs should name things (resources), not actions.

---

### Purposes

- To provide a **uniform, predictable** interface across services.
- To enable **caching** and **idempotent retries** via HTTP semantics .
- To **decouple** clients from server implementation details.
- To support **stateless** scaling across multiple servers.

---

### Syntax Rules and Structure (REST Semantics)

| Verb | Semantics | Idempotent? | Safe? |
|---|---|---|---|
| `GET` | Read resource | ✔️ | ✔️ |
| `HEAD` | Read headers only | ✔️ | ✔️ |
| `POST` | Create resource | ❌ | ❌ |
| `PUT` | Replace resource | ✔️ | ❌ |
| `PATCH` | Partial update | ❌ (document) | ❌ |
| `DELETE` | Remove resource | ✔️ | ❌ |

**Component breakdown**

- **Idempotent** — Repeating the request has the same effect as one call .
- **Safe** — Does not change server state.
- **Status codes** — `200`, `201`, `204`, `400`, `404`, `409`, etc.

#### Syntax Rules

- URLs identify resources (`/api/orders/42`), not actions.
- Use nouns, not verbs, in route paths.
- Return appropriate status codes for each outcome.

#### Constraints and Limitations

- `POST` is not idempotent; duplicate submissions can create duplicates.
- `PATCH` idempotency depends on the operation (setting a value vs. incrementing).

---

### Annotated Code Examples and Expected Outputs

#### Example: Idempotent `DELETE`

```csharp
[HttpDelete("{id:int}")]
public IActionResult Delete(int id)
{
    var removed = _store.RemoveAll(r => r.Id == id);
    return removed > 0 ? NoContent() : NotFound();
}
```

**Expected Output**

- First `DELETE /api/items/1` → `204 No Content`.
- Second `DELETE /api/items/1` → `404 Not Found`.

**Why this output occurs:** The operation is idempotent: the state (resource removed) is the same after both calls, even though status codes differ .

---

### Real-World Cases

- **Public APIs** — Stripe, GitHub, and Microsoft Graph follow REST principles.
- **Microservices** — RESTful contracts between services.
- **Caching proxies** — `GET` responses cacheable; `PUT`/`DELETE` idempotent for retries.

---

### References

- Best practices for RESTful web API implementation — Microsoft Learn — https://learn.microsoft.com/en-sg/azure/architecture/best-practices/api-implementation
- RESTful Web API design — Microsoft Learn — https://learn.microsoft.com/api-design

---

## 3. `[ApiController]`

### Definitions

**Core Definition:** `[ApiController]` is an attribute applied to controller classes that enables a set of opinionated, API-specific behaviours.

**Technical Definition:** The `[ApiController]` attribute can be applied to a controller class to enable the following fixed behaviours: **automatic HTTP 400 responses** for model validation errors, **binding source parameter inference** (e.g., `[FromBody]` for complex types), **multipart/form-data request inference**, and **problem details for error status codes** . It is typically combined with `[Route]` on the controller .

**Beginner-Friendly Explanation:** `[ApiController]` is a **switch that turns on smart API defaults**—automatic validation, better error responses, and smarter parameter binding.

---

### Purposes

- To enable **automatic model validation** (400 responses) without manual `ModelState` checks .
- To infer **binding sources** for parameters (`[FromBody]`, `[FromRoute]`, `[FromQuery]`) .
- To return **standardised problem details** for errors.
- To reduce boilerplate in API controllers.

---

### Syntax Rules and Structure

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<Product> Create(CreateProductDto dto) => ...;
}
```

**Component breakdown**

- `[ApiController]` — Enables API behaviours.
- `[Route]` — Defines the route prefix.
- Complex parameters automatically infer `[FromBody]` .

#### Syntax Rules

- Apply `[ApiController]` at the controller level.
- Combine with `[Route]` for explicit routing.
- Complex types bind from the body by default; simple types from route/query.

#### Constraints and Limitations

- Automatic 400 responses may not match custom error contracts; suppress or customise via `ApiBehaviorOptions` .
- Binding source inference can be overridden with explicit attributes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Automatic 400 Validation

```csharp
public class CreateProductDto
{
    [Required] public string Name { get; set; } = "";
    [Range(0.01, 1_000_000)] public decimal Price { get; set; }
}

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<CreateProductDto> Create(CreateProductDto dto) =>
        CreatedAtAction(nameof(Get), new { id = 1 }, dto);

    [HttpGet("{id:int}")]
    public ActionResult Get(int id) => Ok();
}
```

**Expected Output**

- Invalid POST (missing `Name`) → `400 Bad Request` with problem details.
- Valid POST → `201 Created`.

**Why this output occurs:** `[ApiController]` automatically returns 400 when `ModelState` is invalid .

---

### Real-World Cases

- **All API controllers** — `[ApiController]` is the default in Web API templates .
- **Custom base controllers** — Apply `[ApiController]` once on a base class for consistency .

---

### References

- Create web APIs with ASP.NET Core — `[ApiController]` attribute — https://learn.microsoft.com/en-us/aspnet/core/web-api/

---

## 4. `[Route]`

### Definitions

**Core Definition:** `[Route]` specifies the URL pattern for a controller or action.

**Technical Definition:** The `[Route]` attribute defines the URL template for a controller or action. When applied at the controller level, it sets a prefix for all actions in that controller . The `[controller]` token is replaced with the controller name (minus the `Controller` suffix) . Attribute routing takes precedence over conventional routing.

**Beginner-Friendly Explanation:** `[Route]` tells ASP.NET Core **what URL leads to this controller or action**.

---

### Purposes

- To define **RESTful URL patterns** for resources .
- To support **token replacement** (`[controller]`, `[action]`) for maintainable routes .
- To enable **nested or custom routes** beyond conventions .

---

### Syntax Rules and Structure

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase { ... }

[Route("api/users/{userId}/reservations")]
public class UserReservationsController : ControllerBase { ... }
```

**Component breakdown**

- `[Route("api/[controller]")]` — Prefixes all actions with `/api/Products` .
- `[controller]` — Replaced with the controller name .
- `{userId}` — Route parameter.

#### Syntax Rules

- `[Route]` can be applied at controller or action level.
- Controller-level routes combine with action-level routes.
- Token replacement keeps routes in sync with class names .

#### Constraints and Limitations

- Concrete routes (`[Route("api/reservations")]`) avoid accidental route changes on rename .
- Conflicting routes cause ambiguous match exceptions.

---

### Annotated Code Examples and Expected Outputs

#### Example: Token-Based Route

```csharp
[ApiController]
[Route("api/[controller]")]
public class ReservationsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(id);
}
```

**Expected Output**

- `GET /api/Reservations/42` → `200 OK` with `42`.

**Why this output occurs:** `[controller]` resolves to `Reservations`; `{id:int}` binds the route parameter .

---

### Real-World Cases

- **API prefixes** — `/api/v1/products`.
- **Nested resources** — `/api/users/{userId}/orders` .
- **Token-based routes** — `[controller]` for automatic rename support .

---

### References

- Routing in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- ASP.NET Core Web API — Routing — https://raw.githubusercontent.com/oskardudycz/WebApiWith.NET/main/README.md

---

## 5. HTTP Verb Attributes

### Definitions

**Core Definition:** HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]`) map controller actions to specific HTTP methods.

**Technical Definition:** ASP.NET Core provides specific attributes representing HTTP methods: `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, `[HttpHead]`, `[HttpPatch]`. These attributes match requests based on both the URL route and the HTTP method . They are typically combined with `[Route]` at the controller level to form complete routes .

**Beginner-Friendly Explanation:** HTTP verb attributes tell ASP.NET Core **which HTTP method this action responds to**.

---

### Purposes

- To **map actions to HTTP verbs** semantically .
- To enable **multiple actions on the same URL** with different verbs.
- To enforce **RESTful semantics** (`GET` reads, `POST` creates, etc.) .

---

### Syntax Rules and Structure

```csharp
[HttpGet]                    // GET /api/products
[HttpGet("{id:int}")]        // GET /api/products/42
[HttpPost]                   // POST /api/products
[HttpPut("{id:int}")]        // PUT /api/products/42
[HttpPatch("{id:int}")]      // PATCH /api/products/42
[HttpDelete("{id:int}")]     // DELETE /api/products/42
```

**Component breakdown**

- `[HttpGet]` — Handles `GET`.
- `[HttpPost]` — Handles `POST`.
- `[HttpPut]` — Handles `PUT`.
- `[HttpPatch]` — Handles `PATCH`.
- `[HttpDelete]` — Handles `DELETE`.

#### Syntax Rules

- Verb attributes can include route templates.
- Multiple verb attributes can be applied to one action (e.g., `[HttpGet]` and `[HttpHead]`).
- Verb attributes override conventional routing.

#### Constraints and Limitations

- `GET` and `DELETE` do not bind the request body implicitly.
- `POST` and `PUT` bind complex types from the body by default.

---

### Annotated Code Examples and Expected Outputs

#### Example: Full CRUD with Verb Attributes

```csharp
[ApiController]
[Route("api/[controller]")]
public class ReservationsController : ControllerBase
{
    [HttpGet] public IActionResult List() => Ok();
    [HttpGet("{id:int}")] public IActionResult Get(int id) => Ok(id);
    [HttpPost] public IActionResult Create([FromBody] CreateReservation request) => CreatedAtAction(nameof(Get), new { id = 1 }, request);
    [HttpPut("{id:int}")] public IActionResult Put(int id, [FromBody] UpdateReservation request) => NoContent();
    [HttpDelete("{id:int}")] public IActionResult Delete(int id) => NoContent();
}
```

**Expected Output (mapping)**

| Request | Action |
|---|---|
| `GET /api/Reservations` | `List` |
| `GET /api/Reservations/1` | `Get` |
| `POST /api/Reservations` | `Create` |
| `PUT /api/Reservations/1` | `Put` |
| `DELETE /api/Reservations/1` | `Delete` |

**Why this output occurs:** Each verb attribute matches the corresponding HTTP method and route .

---

### Real-World Cases

- **CRUD APIs** — Standard verb-to-action mapping .
- **Partial updates** — `PATCH` for field-level changes.
- **File uploads** — `POST` with `multipart/form-data`.

---

### References

- ASP.NET Core Web API — HTTP verb attributes — https://raw.githubusercontent.com/oskardudycz/WebApiWith.NET/main/README.md
- HTTP request methods — Microsoft Learn — https://learn.microsoft.com/api-design#http-request-methods

---

## 6. API Versioning

### Definitions

**Core Definition:** API versioning is the practice of exposing multiple versions of an API simultaneously so clients can upgrade on their own schedule without breaking.

**Technical Definition:** The ASP.NET API Versioning library (`Asp.Versioning.Mvc`) provides support for versioning controller-based APIs. Services should respond with `api-supported-versions` and `api-deprecated-versions` headers to advertise available and deprecated versions . Version reporting is disabled by default and can be enabled globally via `ApiVersioningOptions.ReportApiVersions` or per-service via `[ReportApiVersions]` . Common strategies include **URL path versioning** (`/api/v1/products`), **query string versioning** (`?api-version=1.0`), and **header versioning** .

**Beginner-Friendly Explanation:** API versioning means **keeping old versions of your API alive while you add new ones**. Clients using `v1` keep working while new clients use `v2`.

---

### Purposes

- To **evolve APIs** without breaking existing clients .
- To **advertise supported and deprecated versions** to clients .
- To **manage deprecation** with clear timelines .

---

### Syntax Rules and Structure

```csharp
// Controller with API version
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase { ... }

[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase { ... }

// Enable version reporting
builder.Services.AddApiVersioning(options =>
{
    options.ReportApiVersions = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
});
```

**Component breakdown**

- `[ApiVersion("1.0")]` — Declares the version.
- `{version:apiVersion}` — Route parameter for the version.
- `ReportApiVersions` — Enables `api-supported-versions` headers .

#### Syntax Rules

- Apply `[ApiVersion]` to controllers.
- Use `{version:apiVersion}` in the route template for URL versioning.
- Enable `ReportApiVersions` to advertise versions .

#### Constraints and Limitations

- Versioning adds routing complexity.
- Deprecated versions must eventually be removed .

---

### Annotated Code Examples and Expected Outputs

#### Example: Version Discovery via `OPTIONS`

```csharp
[HttpOptions]
public IActionResult Options()
{
    Response.Headers.Allow = "GET, POST, OPTIONS";
    return Ok();
}
```

**Expected Output (response headers)**
```
allow: GET, POST, OPTIONS
api-supported-versions: 1.0, 2.0, 3.0
```

**Why this output occurs:** The `OPTIONS` handler returns the allowed methods and, when version reporting is enabled, the supported versions .

---

### Real-World Cases

- **Public APIs** — Stripe uses date-based versions; GitHub uses URL versions.
- **Internal services** — Gradual rollout of breaking changes.
- **Deprecation** — Advertise deprecated versions with sunset timelines .

---

### References

- Version Discovery — ASP.NET API Versioning — https://dotnet.github.io/aspnet-api-versioning/aspnet-core/version-discovery.html
- Versioning a RESTful web API — Microsoft Learn — https://learn.microsoft.com/azure/architecture/best-practices/api-design#versioning-a-restful-web-api

---

## 7. API Documentation (Swagger / OpenAPI)

### Definitions

**Core Definition:** OpenAPI (formerly Swagger) is a language-agnostic specification for describing REST APIs; Swagger is the tooling ecosystem that generates interactive documentation from OpenAPI documents.

**Technical Definition:** OpenAPI documentation defines a standard, language-agnostic interface description for web APIs, allowing both humans and computers to discover and understand the capabilities of a service without access to source code . **Swashbuckle** is the most common tool for generating OpenAPI documents and serving Swagger UI in ASP.NET Core applications . The `Microsoft.AspNetCore.OpenApi` package provides built-in support for generating OpenAPI documents in Minimal APIs and controller-based APIs . Microsoft recommends enabling OpenAPI UIs only in development .

**Beginner-Friendly Explanation:** OpenAPI/Swagger is **automatic, interactive documentation** for your API. It lists every endpoint, its parameters, and its responses—and lets you try requests from the browser.

---

### Purposes

- To **document the API contract** for consumers .
- To **generate client SDKs** from the specification.
- To **test APIs interactively** via Swagger UI .
- To **keep docs in sync** with code through automatic generation .

---

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

**Component breakdown**

- `AddEndpointsApiExplorer()` — Discovers endpoints.
- `AddSwaggerGen()` — Generates the OpenAPI document.
- `UseSwagger()` — Serves the JSON document.
- `UseSwaggerUI()` — Serves the interactive UI.

#### Syntax Rules

- Swagger UI should be enabled only in development .
- Use XML comments and `[ProducesResponseType]` to enrich the document .
- .NET 9+ uses `Microsoft.AspNetCore.OpenApi` for built-in document generation .

#### Constraints and Limitations

- Generated documentation is only as accurate as the code annotations.
- Exposing Swagger UI in production is a security risk .

---

### Annotated Code Examples and Expected Outputs

#### Example: Swagger UI in Development

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1"));
}
```

**Expected Output**
```
Browser: /swagger → interactive UI listing all endpoints
```

**Why this output occurs:** Swashbuckle generates the OpenAPI document and serves the UI .

---

### Real-World Cases

- **Public APIs** — Stripe, Microsoft Graph expose OpenAPI documents.
- **Client generation** — NSwag or OpenAPI Generator produce typed clients.
- **API gateways** — Azure API Management imports OpenAPI documents .

---

### References

- Sử dụng Swashbuckle để tạo tài liệu OpenAPI — Microsoft Learn — https://learn.microsoft.com/vi-vn/training/modules/integrate-openapi-enabled-web-api-with-apim-and-visual-studio/2-activate-openapi-to-web-api-using-swashbuckle
- OpenAPI with ASP.NET Core in .NET 9 and 10 — GitHub — https://github.com

---

## 8. Web API Best Practices

### Definitions

**Core Definition:** Web API best practices are the guidelines for building secure, performant, and maintainable HTTP services.

**Technical Definition:** Best practices derive from Microsoft's RESTful API implementation guidance: **preserve HTTP method semantics and implement idempotency** ; **validate request data and control field access with DTOs** ; **limit side effects of POST actions** ; **avoid chatty operations**; **enable version reporting** ; **document with OpenAPI**; and **enable Swagger UI only in development** .

**Beginner-Friendly Explanation:** The best practices are: **use the right verb**, **don't expose your database entities**, **validate everything**, **keep POST side effects focused**, **version your API**, and **document it**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Use DTOs, not entities
public ActionResult<ProductDto> Get(int id) =>
    _service.Get(id) is { } p ? Ok(p.ToDto()) : NotFound();

// 2. Validate input
[HttpPost]
public ActionResult Create([FromBody] CreateProductDto dto) =>
    ModelState.IsValid ? CreatedAtAction(...) : BadRequest(ModelState);

// 3. Idempotent DELETE
[HttpDelete("{id:int}")]
public IActionResult Delete(int id) =>
    _store.RemoveAll(r => r.Id == id) > 0 ? NoContent() : NotFound();

// 4. Version reporting
builder.Services.AddApiVersioning(o => o.ReportApiVersions = true);

// 5. Swagger only in development
if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); }
```

**Component breakdown**

- **DTOs** — Prevent over-posting and leaking internal fields .
- **Validation** — `ModelState` + `[ApiController]` automatic 400.
- **Idempotency** — `DELETE`, `PUT` safe to retry .
- **Versioning** — Advertise versions via headers .
- **Documentation** — OpenAPI, development-only UI .

#### Rules and Constraints

- **Do** preserve HTTP semantics and idempotency .
- **Do** use DTOs for input and output .
- **Do** validate all request data .
- **Do** enable version reporting .
- **Do not** bind entities directly to request bodies .
- **Do not** expose Swagger UI in production .

---

### Annotated Code Examples and Expected Outputs

#### Example: DTO-Validated, Idempotent Endpoint

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public ActionResult<ProductDto> Create([FromBody] CreateProductDto dto)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        var product = _service.Create(dto);
        return CreatedAtAction(nameof(Get), new { id = product.Id, version = "1.0" }, product.ToDto());
    }

    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id) =>
        _service.Delete(id) ? NoContent() : NotFound();
}
```

**Expected Output**

- Invalid POST → `400 Bad Request` with validation details.
- Valid POST → `201 Created` with `Location` header.
- First DELETE → `204 No Content`; second → `404 Not Found`.

**Why this output occurs:** It combines DTOs, automatic validation, idempotent deletion, and versioned routing .

---

### Real-World Cases

- **Public APIs** — Stripe, GitHub, Microsoft Graph.
- **Microservices** — Versioned, documented contracts.
- **Mobile backends** — DTOs shield internal models from clients.

---

### References

- Best practices for RESTful web API implementation — Microsoft Learn — https://learn.microsoft.com/en-sg/azure/architecture/best-practices/api-implementation
- Create web APIs with ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/web-api/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Controller` base class for APIs | **Discouraged**; derive from `ControllerBase` for Web APIs . |
| `[ApiController]` | **Recommended**; enables automatic 400 responses and binding inference . |
| `[Route("api/[controller]")]` | **Token-based**; concrete routes avoid rename side effects . |
| `GET`/`DELETE` body binding | **Not implicit**; use `[FromBody]` or read `HttpRequest` manually. |
| `POST`/`PATCH` idempotency | **Not inherent**; document and implement artificial idempotency if needed . |
| Swagger UI in production | **Security risk**; enable only in development . |
| `Microsoft.AspNetCore.OpenApi` | **.NET 9+**; built-in OpenAPI support . |

---

## References (Consolidated)

- Create web APIs with ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/71c331669b8a6c1acfea6db3420ad6399ca018dc/aspnetcore/tutorials/first-web-api.md
- Create web APIs with ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/web-api/
- ASP.NET Core Web API — Routing and HTTP verbs — https://raw.githubusercontent.com/oskardudycz/WebApiWith.NET/main/README.md
- Best practices for RESTful web API implementation — Microsoft Learn — https://learn.microsoft.com/en-sg/azure/architecture/best-practices/api-implementation
- Version Discovery — ASP.NET API Versioning — https://dotnet.github.io/aspnet-api-versioning/aspnet-core/version-discovery.html
- Sử dụng Swashbuckle để tạo tài liệu OpenAPI — Microsoft Learn — https://learn.microsoft.com/vi-vn/training/modules/integrate-openapi-enabled-web-api-with-apim-and-visual-studio/2-activate-openapi-to-web-api-using-swashbuckle
- OpenAPI with ASP.NET Core in .NET 9 and 10 — GitHub — https://github.com
- Routing in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Versioning a RESTful web API — Microsoft Learn — https://learn.microsoft.com/azure/architecture/best-practices/api-design#versioning-a-restful-web-api

---