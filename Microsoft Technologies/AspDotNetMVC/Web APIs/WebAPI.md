# ASP.NET Core Web API Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core Web API** (.NET 6–10). Web APIs are HTTP-based services that expose data and operations to clients (SPAs, mobile apps, other services). ASP.NET Core implements APIs through **API controllers** (`[ApiController]` + `ControllerBase`) and **Minimal APIs** (`MapGet`, `MapPost`, etc.). This cheat sheet focuses on the **controller-based** model, which remains fully supported and is the recommended choice for larger APIs with rich routing, filters, and OpenAPI integration. Minimal APIs are noted where they differ. **.NET 9** introduced built-in OpenAPI document generation (`Microsoft.AspNetCore.OpenApi`), replacing Swashbuckle as the default in templates. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Web API is an application programming interface exposed over HTTP that allows client applications to interact with a service using standard HTTP methods and data formats (typically JSON). |
| **Technical Definition** | In ASP.NET Core, a Web API is built on the same routing, model binding, and middleware pipeline as MVC, but with controller actions that return **data** (JSON/XML) instead of HTML views. `[ApiController]` enables API-specific conventions: automatic model-state validation (400 responses), binding-source inference, and `ProblemDetails` error responses. Controllers derive from `ControllerBase` (not `Controller`) because they do not render views. |
| **Beginner-Friendly Explanation** | A Web API is the **"backend for other apps"**. Instead of returning web pages for humans to view, it returns data (usually JSON) for programs to consume. A mobile app, a JavaScript frontend, or another service sends HTTP requests to your API, and your API responds with the requested data or performs the requested operation. Think of it as a **menu of operations** that other software can order from. |

### Key Characteristics

- **HTTP-native** – Uses standard HTTP verbs (GET, POST, PUT, PATCH, DELETE), status codes, and headers.
- **Stateless** – Each request contains all the information needed to process it; no server-side session is required (by default).
- **Data-oriented** – Returns structured data (JSON by default) rather than HTML.
- **Resource-based** – RESTful APIs model the domain as resources identified by URLs.
- **Contract-driven** – Consumers depend on the shape of the request/response payloads and status codes.
- **Versionable** – APIs evolve over time and must manage breaking changes deliberately.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **HTTP** (methods, status codes, headers).
- Familiarity with **C#**, **controllers**, **routing**, and **dependency injection**.
- Basic understanding of **JSON** and **model binding**.

### Related Programming Areas

- **REST** – The architectural style most commonly used for Web APIs.
- **MVC** – Web APIs share infrastructure (routing, model binding, filters) with MVC.
- **Authentication and authorization** – JWT, OAuth, and policy-based security for APIs.
- **OpenAPI** – Machine-readable API descriptions used for documentation and client generation.
- **Minimal APIs** – A lighter alternative to controllers for small services.

### Core Concepts / Features

1. **Web APIs** – The concept, request/response model, and ASP.NET Core implementation.
2. **REST** – The architectural style and its constraints.
3. **RESTful design** – Resource modeling, HTTP verb semantics, status codes, and HATEOAS.
4. **API controllers** – `[ApiController]`, `ControllerBase`, routing, model binding, and `ProblemDetails`.
5. **Web API best practices** – Versioning, pagination, security, error handling, and documentation.

---

## 1. Web APIs (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Web API is an HTTP-based interface that exposes data and operations to client applications in a machine-readable format. |
| **Technical Definition** | ASP.NET Core Web APIs are built using controllers that derive from `ControllerBase` and are decorated with `[ApiController]`. They use attribute routing (`[Route]`, `[HttpGet]`, etc.), return `IActionResult` or typed results (`ActionResult<T>`), and rely on `System.Text.Json` for serialization by default. The framework short-circuits invalid model state with a 400 response and converts unhandled errors into `ProblemDetails` responses when configured. |
| **Beginner-Friendly Explanation** | A Web API is the **"data waiter"** for your application. When a client asks for "the order with ID 42," the API fetches it from the database and returns it as JSON. When the client says "create a new order," the API validates the data and saves it. The client never sees HTML—just structured data it can use however it wants. |

### Purposes

- **To** expose application functionality to **multiple client types** (web, mobile, desktop, other services).
- **To** decouple the **backend** from the **frontend**, allowing independent evolution and deployment.
- **To** enable **integration** between systems (microservices, third-party integrations, automation).
- **To** provide a **stable, versioned contract** for data exchange.
- **To** leverage **standard HTTP infrastructure** (caching, authentication, compression, proxies).

### Syntax Rules and Structure

#### General Syntax (API Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(new[] { "Laptop", "Mouse" });

    [HttpGet("{id:int}")]
    public IActionResult GetById(int id) => Ok($"Product {id}");

    [HttpPost]
    public IActionResult Create([FromBody] ProductDto dto) => CreatedAtAction(
        nameof(GetById), new { id = 42 }, dto);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[ApiController]` | Enables API conventions (automatic 400s, binding inference, ProblemDetails). |
| `: ControllerBase` | Base class for API controllers (no view support). |
| `[Route("api/[controller]")]` | Sets the base route using the controller name. |
| `[HttpGet]`, `[HttpPost]` | HTTP verb attributes with optional route templates. |
| `IActionResult` / `ActionResult<T>` | Return type for HTTP responses. |

#### General Syntax (Minimal API Equivalent)

```csharp
var app = builder.Build();

app.MapGet("/api/products", () => new[] { "Laptop", "Mouse" });
app.MapGet("/api/products/{id:int}", (int id) => $"Product {id}");
app.MapPost("/api/products", (ProductDto dto) =>
    Results.Created($"/api/products/42", dto));
```

**Syntax rules:**

- **API controllers must derive from `ControllerBase`** —deriving from `Controller` adds view support that APIs do not need.
- **`[ApiController]` is recommended** —it enables automatic model validation and binding inference, reducing boilerplate.
- **Return types should be `IActionResult` or `ActionResult<T>`** —this allows status codes, headers, and content negotiation.
- **Use `[FromBody]`, `[FromQuery]`, `[FromRoute]`** to specify binding sources explicitly when inference is ambiguous.

**Constraints and limitations:**

- **APIs should not return views** —`ControllerBase` does not support `View()`; use `Controller` only for MVC.
- **Automatic 400 responses** —`[ApiController]` short-circuits invalid model state before the action executes; ensure error shapes are acceptable to clients.
- **No session by default** —APIs are stateless; use tokens for authentication rather than cookies where appropriate.

### Annotated Complete Step-by-Step Code Example

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();          // Registers API controller services
builder.Services.AddEndpointsApiExplorer(); // Enables endpoint metadata for OpenAPI
builder.Services.AddOpenApi();              // .NET 9+: built-in OpenAPI document generation

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();                       // Serve the OpenAPI document
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();

// ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private static readonly List<Product> _products = new()
    {
        new(1, "Laptop", 999.99m),
        new(2, "Mouse", 25.00m)
    };

    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_products);

    [HttpGet("{id:int}")]
    public ActionResult<Product> GetById(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        return product is null ? NotFound() : Ok(product);
    }

    [HttpPost]
    public ActionResult<Product> Create([FromBody] Product product)
    {
        product.Id = _products.Max(p => p.Id) + 1;
        _products.Add(product);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}

public record Product(int Id, string Name, decimal Price);
```

**Step-by-step explanation:**

1. `AddControllers()` registers API controller services (no views).
2. `AddOpenApi()` registers the built-in OpenAPI document generator (.NET 9+).
3. `MapControllers()` maps attribute-routed controllers.
4. `ProductsController` uses `[ApiController]` and `ControllerBase` for API conventions.
5. `GetAll` returns `200 OK` with the product list.
6. `GetById` returns `404 NotFound` when the product does not exist, `200 OK` otherwise.
7. `Create` returns `201 Created` with a `Location` header pointing to the new resource.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products` | `200 OK` with JSON array of products |
| `GET /api/products/1` | `200 OK` with the product JSON |
| `GET /api/products/99` | `404 Not Found` |
| `POST /api/products` with JSON body | `201 Created` with `Location: /api/products/3` |

**Why this output?** The API returns data as JSON (default formatter), uses appropriate HTTP status codes, and provides resource location on creation.

### Real-World Cases

- **Mobile app backends:** iOS/Android apps consume JSON APIs.
- **SPA backends:** React/Angular/Vue frontends call APIs for data.
- **Third-party integrations:** Partner systems consume your API for data exchange.
- **Microservices:** Services communicate via internal APIs.

### References

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- Tutorial: Create a web API with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api

---

## 2. REST (The Architectural Style)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | REST (Representational State Transfer) is an architectural style for distributed systems that relies on stateless communication, a uniform interface, and resource-based addressing over HTTP. |
| **Technical Definition** | REST defines six architectural constraints: **client-server**, **statelessness**, **cacheability**, **layered system**, **code on demand** (optional), and the **uniform interface** (identification of resources, manipulation through representations, self-descriptive messages, and hypermedia as the engine of application state). REST is **not a protocol or standard**—it is a set of design constraints. HTTP is the most common implementation substrate. |
| **Beginner-Friendly Explanation** | REST is a **style of designing web APIs** that aligns with how HTTP was meant to work. Instead of inventing custom operations ("getUserById," "createUser"), you model your domain as **resources** (like `users` and `orders`) and use HTTP verbs to act on them (`GET` to read, `POST` to create, `PUT` to replace, `DELETE` to remove). REST APIs are stateless, cacheable, and easy to understand because they follow familiar HTTP patterns. |

### Purposes

- **To** provide a **standard, predictable** way to design APIs that leverage HTTP semantics.
- **To** enable **scalability** through statelessness and cacheability.
- **To** promote **loose coupling** between clients and servers via a uniform interface.
- **To** support **intermediaries** (proxies, CDNs, gateways) through layered architecture.
- **To** make APIs **easy to understand and consume** for developers familiar with HTTP.

### Syntax Rules and Structure

#### The Six REST Constraints

| Constraint | Description |
|------------|-------------|
| **Client-server** | UI concerns are separated from data storage concerns. |
| **Stateless** | Each request contains all information needed; the server stores no client session state. |
| **Cacheable** | Responses must indicate whether they can be cached (via HTTP headers). |
| **Layered system** | Intermediaries (proxies, gateways) can be inserted without clients knowing. |
| **Code on demand** (optional) | Servers can send executable code (e.g., JavaScript) to clients. |
| **Uniform interface** | Resource identification via URLs, manipulation via representations, self-descriptive messages, HATEOAS. |

**Syntax rules:**

- **REST is not a protocol** —it is a design style; a RESTful API adheres to the constraints, while an HTTP API may not.
- **Statelessness is required** —the server should not rely on session state between requests.
- **Resources are identified by URLs** —nouns, not verbs (`/users/42`, not `/getUser?id=42`).
- **Representations** —resources are transferred as representations (JSON, XML) and manipulated through those representations.

**Constraints and limitations:**

- **REST is a spectrum** —few APIs implement all constraints (especially HATEOAS); the term "RESTful" is often used loosely for resource-oriented HTTP APIs.
- **Statelessness has costs** —every request must authenticate and carry context; tokens and caching are essential.
- **Not ideal for all scenarios** —RPC-style operations (e.g., `/sendEmail`) may fit poorly with pure REST.

### Annotated Complete Step-by-Step Code Example

```csharp
// Resource-oriented API design
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // GET /api/orders — read the collection
    [HttpGet]
    public ActionResult<IEnumerable<Order>> GetAll() => Ok(_orders);

    // GET /api/orders/42 — read a specific resource
    [HttpGet("{id:int}")]
    public ActionResult<Order> GetById(int id) => ...;

    // POST /api/orders — create a new resource
    [HttpPost]
    public ActionResult<Order> Create([FromBody] Order order) => ...;

    // PUT /api/orders/42 — replace a resource
    [HttpPut("{id:int}")]
    public IActionResult Update(int id, [FromBody] Order order) => ...;

    // DELETE /api/orders/42 — delete a resource
    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id) => ...;
}
```

**Expected Output:** Each endpoint operates on the `orders` resource identified by a URL, using the appropriate HTTP verb.

**Why this output?** The API follows REST's uniform interface: resources are identified by URLs, and operations are expressed through standard HTTP verbs rather than custom method names.

### Real-World Cases

- **Public APIs:** GitHub, Stripe, and Twilio expose RESTful APIs.
- **Internal microservices:** Resource-oriented APIs for service-to-service communication.
- **Mobile backends:** REST APIs consumed by mobile apps.

### References

- Architectural Styles and the Design of Network-based Software Architectures (Fielding's dissertation) – https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
- REST (Wikipedia) – https://en.wikipedia.org/wiki/Representational_state_transfer

---

## 3. RESTful Design (Resource Modeling and HTTP Semantics)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | RESTful design is the practice of modeling an API's domain as resources addressed by URLs, manipulated through standard HTTP methods, and returning semantically correct status codes. |
| **Technical Definition** | RESTful design involves **resource identification** (nouns in URLs), **HTTP method semantics** (GET is safe/idempotent, PUT is idempotent, POST is neither, DELETE is idempotent), **status code correctness** (200, 201, 204, 400, 401, 403, 404, 409, 422, 500), **content negotiation** (Accept/Content-Type), **HATEOAS** (hypermedia links), and **pagination/filtering conventions** for collections. |
| **Beginner-Friendly Explanation** | RESTful design is the **"how to build a clean REST API"** guide. It says: use nouns in URLs (`/products`, not `/getProducts`), use the right HTTP verb for each action (`GET` to read, `POST` to create, `PUT` to replace, `DELETE` to remove), return the right status code (`201` when creating, `404` when not found), and include links so clients can discover related resources. |

### Purposes

- **To** provide a **predictable, self-documenting** API surface.
- **To** leverage **HTTP semantics** for caching, idempotency, and safety guarantees.
- **To** enable **correct client behaviour** through proper status codes and headers.
- **To** support **discoverability** through HATEOAS (optional but recommended for maturity).
- **To** make APIs **easier to learn, document, and maintain**.

### Syntax Rules and Structure

#### Resource Naming

| Pattern | Example | Notes |
|---------|---------|-------|
| **Plural nouns** | `/api/products` | Collections are plural. |
| **Specific resource** | `/api/products/42` | Identifier in the path. |
| **Sub-resources** | `/api/customers/42/orders` | Nested relationships. |
| **Query filtering** | `/api/products?category=electronics` | Filters on collections. |

#### HTTP Method Semantics

| Method | Purpose | Safe? | Idempotent? | Typical Status Codes |
|--------|---------|-------|-------------|----------------------|
| **GET** | Read a resource or collection | Yes | Yes | 200, 404 |
| **POST** | Create a resource | No | No | 201, 400, 409 |
| **PUT** | Replace a resource | No | Yes | 200, 204, 404 |
| **PATCH** | Partially update a resource | No | No (by default) | 200, 204, 404 |
| **DELETE** | Remove a resource | No | Yes | 204, 404 |

#### Status Code Selection

| Status Code | Meaning | When to Use |
|-------------|---------|-------------|
| **200 OK** | Success with response body | GET, PUT, PATCH returning data. |
| **201 Created** | Resource created | POST with `Location` header. |
| **204 No Content** | Success without body | PUT, PATCH, DELETE with no response body. |
| **400 Bad Request** | Malformed request | Invalid JSON or parameters. |
| **401 Unauthorized** | Authentication required | Missing or invalid credentials. |
| **403 Forbidden** | Authenticated but not allowed | Authorization failure. |
| **404 Not Found** | Resource does not exist | GET, PUT, DELETE on missing resource. |
| **409 Conflict** | Conflict with current state | Duplicate creation, concurrency conflict. |
| **422 Unprocessable Entity** | Validation failure | Semantically invalid but well-formed request. |
| **500 Internal Server Error** | Server failure | Unhandled exceptions. |

**Syntax rules:**

- **Use nouns, not verbs, in URLs** —`/api/products`, not `/api/getProducts`.
- **Use the correct HTTP verb** —do not use GET for state-changing operations.
- **Return the most specific status code** —`201 Created` on creation, `204 No Content` on successful delete without body.
- **Use `Location` header on `201 Created`** —point to the new resource's URL.
- **Support `Accept` and `Content-Type`** —respect content negotiation.

**Constraints and limitations:**

- **HATEOAS is rarely fully implemented** —many "RESTful" APIs are resource-oriented HTTP APIs that do not include hypermedia links.
- **PATCH is complex** —JSON Patch (`application/json-patch+json`) and JSON Merge Patch (`application/merge-patch+json`) are standard formats but add complexity.
- **Idempotency requires discipline** —POST and PATCH are not inherently idempotent; use idempotency keys for retry safety.

### Annotated Complete Step-by-Step Code Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    // GET /api/customers — collection
    [HttpGet]
    public ActionResult<IEnumerable<CustomerDto>> GetAll() => Ok(_repo.GetAll());

    // GET /api/customers/42 — single resource
    [HttpGet("{id:int}")]
    public ActionResult<CustomerDto> GetById(int id)
    {
        var customer = _repo.Find(id);
        return customer is null ? NotFound() : Ok(customer);
    }

    // GET /api/customers/42/orders — sub-resource collection
    [HttpGet("{id:int}/orders")]
    public ActionResult<IEnumerable<OrderDto>> GetOrders(int id)
    {
        var customer = _repo.Find(id);
        return customer is null ? NotFound() : Ok(_repo.GetOrders(id));
    }

    // POST /api/customers — create
    [HttpPost]
    public ActionResult<CustomerDto> Create([FromBody] CreateCustomerDto dto)
    {
        var created = _repo.Create(dto);
        return CreatedAtAction(nameof(GetById),
            new { id = created.Id }, created);
    }

    // PUT /api/customers/42 — full replace
    [HttpPut("{id:int}")]
    public IActionResult Update(int id, [FromBody] UpdateCustomerDto dto)
    {
        if (!_repo.Exists(id)) return NotFound();
        _repo.Update(id, dto);
        return NoContent();
    }

    // DELETE /api/customers/42 — delete
    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id)
    {
        if (!_repo.Exists(id)) return NotFound();
        _repo.Delete(id);
        return NoContent();
    }
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/customers` | `200 OK` with array |
| `GET /api/customers/42` | `200 OK` or `404 Not Found` |
| `GET /api/customers/42/orders` | `200 OK` or `404 Not Found` |
| `POST /api/customers` | `201 Created` + `Location` |
| `PUT /api/customers/42` | `204 No Content` or `404` |
| `DELETE /api/customers/42` | `204 No Content` or `404` |

**Why this output?** Each endpoint uses the correct HTTP method, returns semantically correct status codes, and follows resource-oriented URL conventions.

### Real-World Cases

- **E-commerce APIs:** `/api/products`, `/api/orders`, `/api/customers/{id}/orders`.
- **Content APIs:** `/api/articles`, `/api/articles/{id}/comments`.
- **SaaS APIs:** `/api/tenants/{tenantId}/users`, `/api/subscriptions`.

### References

- RESTful API design best practices – https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- HTTP request methods – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- HTTP response status codes – https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

---

## 4. API Controllers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An API controller is a class that derives from `ControllerBase`, is decorated with `[ApiController]`, and exposes actions that return data (JSON) rather than HTML views. |
| **Technical Definition** | API controllers use attribute routing (`[Route]`, `[HttpGet]`, etc.) and return `IActionResult` or `ActionResult<T>`. `[ApiController]` enables three key behaviours: **automatic model-state validation** (returns 400 with `ProblemDetails` when `ModelState` is invalid), **binding-source inference** (`[FromBody]` for complex types, `[FromRoute]` for route parameters, `[FromQuery]` for simple types), and **ProblemDetails responses** for error status codes. |
| **Beginner-Friendly Explanation** | An API controller is the **class that handles API requests**. It is like a regular MVC controller, but it returns data instead of pages. The `[ApiController]` attribute turns on helpful defaults—like automatically returning `400 Bad Request` when the request body is invalid—so you write less boilerplate code. |

### Purposes

- **To** handle HTTP requests and return data responses.
- **To** leverage API-specific conventions (automatic 400s, binding inference) via `[ApiController]`.
- **To** provide a familiar, testable structure for larger APIs.
- **To** integrate with filters, model binding, and OpenAPI.
- **To** support content negotiation and status-code control via `IActionResult`.

### Syntax Rules and Structure

#### General Syntax

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_repo.GetAll());

    [HttpGet("{id:int}")]
    public ActionResult<Product> GetById(int id) => ...;

    [HttpPost]
    public ActionResult<Product> Create([FromBody] ProductDto dto) => ...;
}
```

**Component breakdown:**

| Attribute / Component | Description |
|-----------------------|-------------|
| `[ApiController]` | Enables API conventions. |
| `ControllerBase` | Base class without view support. |
| `[Route("api/[controller]")]` | Base route using controller name. |
| `[HttpGet]`, `[HttpPost]`, etc. | HTTP method and route. |
| `ActionResult<T>` | Return type combining data and status codes. |
| `[FromBody]`, `[FromQuery]`, `[FromRoute]` | Binding source attributes. |

#### `[ApiController]` Conventions

| Convention | Behaviour |
|------------|-----------|
| **Automatic 400** | Invalid `ModelState` returns `400 Bad Request` with `ProblemDetails`. |
| **Binding inference** | Complex types → `[FromBody]`; route parameters → `[FromRoute]`; simple types → `[FromQuery]`. |
| **ProblemDetails** | Error status codes include a `ProblemDetails` body. |
| **Attribute routing required** | Conventional routing is not supported for API controllers. |

**Syntax rules:**

- **API controllers must use attribute routing** —`[Route]` or HTTP verb attributes.
- **Return `ActionResult<T>`** —allows returning either `T` (implicit `200 OK`) or an `IActionResult` (`NotFound()`, `BadRequest()`, etc.).
- **Use `[FromBody]` for complex types** —`[ApiController]` infers it, but explicit is clearer.
- **Use `[FromQuery]` for complex query parameters** in GET requests.

**Constraints and limitations:**

- **No view support** —`ControllerBase` does not have `View()`, `PartialView()`, etc.
- **Automatic 400 may not fit all clients** —some APIs prefer a custom error shape; configure `ApiBehaviorOptions` to customize.
- **Complex types cannot be bound from query strings** without `[FromQuery]` and a custom model binder.

### Annotated Complete Step-by-Step Code Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repo;

    public ProductsController(IProductRepository repo) => _repo = repo;

    [HttpGet]
    public async Task<ActionResult<IReadOnlyList<ProductDto>>> GetAll(
        [FromQuery] string? category, [FromQuery] int page = 1)
    {
        var products = await _repo.ListAsync(category, page);
        return Ok(products);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _repo.FindAsync(id);
        return product is null ? NotFound() : Ok(product);
    }

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Create([FromBody] CreateProductDto dto)
    {
        var created = await _repo.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById),
            new { id = created.Id }, created);
    }

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        if (!await _repo.ExistsAsync(id)) return NotFound();
        await _repo.DeleteAsync(id);
        return NoContent();
    }
}
```

**Step-by-step explanation:**

1. `[ApiController]` enables automatic 400 responses and binding inference.
2. `[Route("api/[controller]")]` sets the base path to `/api/products`.
3. `GetAll` uses `[FromQuery]` for optional filters and pagination.
4. `GetById` returns `404 NotFound` when the product does not exist.
5. `Create` returns `201 Created` with a `Location` header.
6. `Delete` returns `204 No Content` on success.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/products?category=electronics&page=2` | `200 OK` with filtered products |
| `GET /api/products/42` | `200 OK` or `404 Not Found` |
| `POST /api/products` (valid JSON) | `201 Created` + `Location` |
| `POST /api/products` (invalid JSON) | `400 Bad Request` with `ProblemDetails` |
| `DELETE /api/products/42` | `204 No Content` or `404 Not Found` |

**Why this output?** `[ApiController]` handles invalid model state automatically; the actions use status codes that match the operation outcome; the `Location` header guides clients to the new resource.

### Real-World Cases

- **CRUD APIs:** Controllers for products, orders, users.
- **Search APIs:** `[FromQuery]` parameters for filtering, sorting, and pagination.
- **File upload APIs:** `[FromForm]` with `IFormFile`.
- **Bulk operations:** `[HttpPost]` with a collection in the body.

### References

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- `[ApiController]` attribute – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- `ControllerBase` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase

---

## 5. Web API Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Web API best practices are guidelines for designing, implementing, and operating APIs that are secure, consistent, performant, and maintainable. |
| **Technical Definition** | Best practices span **versioning** (URL, header, or media type), **pagination and filtering** (offset/limit, cursor-based), **error handling** (`ProblemDetails`, consistent error shapes), **security** (HTTPS, authentication, authorization, rate limiting), **documentation** (OpenAPI), **content negotiation**, **caching** (`ETag`, `Cache-Control`), and **idempotency** (idempotency keys for POST). |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for building APIs—they help you avoid breaking changes, security holes, confusing error messages, and APIs that are impossible to consume reliably. |

### Purposes

- **To** ensure **backward compatibility** through deliberate versioning.
- **To** protect the API from **abuse and attacks** (rate limiting, HTTPS, authentication).
- **To** provide **consistent, actionable error responses**.
- **To** enable **efficient data retrieval** through pagination and filtering.
- **To** make the API **discoverable and usable** through OpenAPI documentation.
- **To** support **reliable retries** through idempotency.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Versioning** | Use URL path (`/api/v1/products`) or header versioning; document the policy. | Introduce breaking changes without versioning. |
| **Pagination** | Use `?page=1&pageSize=20` or cursor-based pagination; return total count in metadata. | Return unbounded collections. |
| **Filtering/sorting** | Use query parameters (`?category=x&sort=name`); document supported fields. | Accept arbitrary field names without validation. |
| **Error handling** | Return `ProblemDetails` with a stable error code. | Return raw exception messages or HTML error pages. |
| **Security** | Enforce HTTPS; use JWT/OAuth; apply rate limiting. | Expose APIs over HTTP or without authentication. |
| **Content negotiation** | Respect `Accept` and `Content-Type`; default to JSON. | Hard-code JSON without considering clients. |
| **Caching** | Use `ETag` and `Cache-Control` for cacheable responses. | Disable caching for static data. |
| **Idempotency** | Support `Idempotency-Key` for POST where retries are expected. | Assume POST is safe to retry without idempotency. |
| **Documentation** | Generate OpenAPI; keep examples current. | Leave the API undocumented. |
| **Rate limiting** | Apply `AddRateLimiter` (.NET 7+). | Allow unbounded request rates. |
| **Health checks** | Expose `/health` for load balancers and orchestrators. | Leave health checks unmonitored. |
| **Testing** | Use `WebApplicationFactory` for integration tests. | Test APIs only through manual calls. |

### Annotated Code Example (Best-Practice API Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddProblemDetails();             // Consistent ProblemDetails responses
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
    });
});
builder.Services.AddOpenApi();                    // .NET 9+

var app = builder.Build();

app.UseExceptionHandler();                        // ProblemDetails for unhandled errors
app.UseStatusCodePages();                         // ProblemDetails for error status codes
app.UseHttpsRedirection();
app.UseRateLimiter();
app.UseAuthorization();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.MapControllers().RequireRateLimiting("fixed");
app.MapHealthChecks("/health");

app.Run();
```

**Why this follows best practices:**

- **`AddProblemDetails()`** ensures consistent error responses across the API.
- **`UseExceptionHandler()` and `UseStatusCodePages()`** produce `ProblemDetails` for unhandled exceptions and error status codes.
- **`AddRateLimiter()`** protects the API from abuse.
- **`UseHttpsRedirection()`** enforces HTTPS.
- **`AddOpenApi()`** generates API documentation.
- **`MapHealthChecks("/health")`** provides a health endpoint for orchestration.
- **`RequireRateLimiting("fixed")`** applies the rate limit policy to API controllers.

### Real-World Cases

- **Public APIs:** Versioning, OpenAPI documentation, rate limiting, and `ProblemDetails` errors.
- **Mobile backends:** Pagination, idempotency keys, and aggressive caching.
- **Microservices:** Health checks, consistent error shapes, and lightweight versioning.
- **Enterprise APIs:** OAuth/OpenID Connect, HTTPS, and strict rate limiting.

### References

- ASP.NET Core web API best practices – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- API design best practices – https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- OpenAPI specification – https://spec.openapis.org/oas/latest.html

---

## Summary Table: Web API Fundamentals at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Web APIs** | Expose data and operations to clients | `[ApiController]`, `ControllerBase`, `AddControllers()` | Every API backend |
| **REST** | Architectural style for HTTP APIs | Stateless, resource-oriented, uniform interface | Designing API architecture |
| **RESTful design** | Resource modeling and HTTP semantics | Nouns in URLs, correct verbs and status codes | Every RESTful API |
| **API controllers** | Handle API requests and return data | `ControllerBase`, `[ApiController]`, `ActionResult<T>` | Controller-based APIs |
| **Best practices** | Secure, consistent, maintainable APIs | Versioning, pagination, `ProblemDetails`, HTTPS, OpenAPI, rate limiting | Every production API |

---

## References (Full List)

- Build web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- Tutorial: Create a web API with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api
- `[ApiController]` attribute – https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute
- `ControllerBase` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase
- API design best practices – https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- HTTP request methods – https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- HTTP response status codes – https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- REST (Wikipedia) – https://en.wikipedia.org/wiki/Representational_state_transfer
- Fielding's dissertation on REST – https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
- OpenAPI specification – https://spec.openapis.org/oas/latest.html
- ASP.NET Core rate limiting middleware – https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit
- ASP.NET Core health checks – https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks