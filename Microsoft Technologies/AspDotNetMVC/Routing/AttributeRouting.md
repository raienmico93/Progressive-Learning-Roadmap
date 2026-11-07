# Attribute Routing in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10) and the **endpoint routing** system that became the default in ASP.NET Core 3.0. Attribute routing is the **recommended and idiomatic** approach for **REST APIs** and is fully supported alongside conventional routing for controllers and views. Legacy routing (`System.Web.Routing`, pre‑3.0) is mentioned only where behaviour differs significantly. `[Route]`, `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, and `[HttpDelete]` are the primary attributes covered. All route attributes implement `IRouteTemplateProvider` and are discovered at application startup.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Attribute routing is a routing strategy in ASP.NET Core MVC where URL patterns are defined **directly on controller classes and action methods** using C# attributes such as `[Route]`, `[HttpGet]`, and `[HttpPost]`. |
| **Technical Definition** | Attribute routing uses attributes that implement `IRouteTemplateProvider` to build `RouteEndpoint` instances at application startup. Each attribute supplies a route template, an optional order, and an optional name. The routing middleware matches incoming requests against these endpoints, applying inline constraints and precedence rules. Actions decorated with route attributes are **attribute‑routed** and cannot be reached through conventional routes; conversely, conventionally routed actions cannot be reached through attribute routes. |
| **Beginner‑Friendly Explanation** | Attribute routing is like writing the **address label directly on each package** instead of keeping a master address book. You put `[Route("api/products")]` on a controller and `[HttpGet("{id}")]` on an action, and the framework immediately knows: “A GET request to `/api/products/5` should run this action.” The route lives right next to the code it belongs to, making it easy to see what URL triggers what code. |

### Key Characteristics

- **Co‑located configuration** – Routes are defined on the controller or action itself, not in a central `Program.cs` file.
- **HTTP verb‑aware** – `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]` constrain matching to specific HTTP methods.
- **Token replacement** – `[controller]` and `[action]` tokens are replaced with the controller and action names at runtime, enabling convention‑based prefixes.
- **Fine‑grained control** – Optional parameters, default values, and constraints can be applied per action, making it easy to require an `id` for one action and make it optional for another.
- **Multiple routes per action** – An action can have multiple route attributes, supporting scenarios such as multiple HTTP verbs or backward‑compatible URLs.
- **Precedence‑based matching** – Endpoint routing computes route precedence from template specificity; literal segments outrank parameters, and constrained parameters outrank unconstrained ones.

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or 10.0 recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`builder.Services.AddControllers()` or `AddControllersWithViews()`).
- `app.MapControllers()` called to discover attribute‑routed controllers.
- Basic understanding of **controllers**, **actions**, and **HTTP verbs**.
- Familiarity with **route template syntax** (parameters, optional markers, constraints).

### Related Programming Areas

- **Conventional Routing** – The alternative routing strategy, preferred for controllers serving HTML views.
- **REST API Design** – Attribute routing is the standard for modelling resources and HTTP verbs.
- **URL Generation** – `IUrlHelper`, `LinkGenerator`, and Tag Helpers use attribute route metadata to produce links.
- **Model Binding** – Route values captured by attribute routes are bound to action parameters.
- **API Versioning** – Attribute routes with version segments (`api/v{version:apiVersion}/...`) are common in versioned APIs.

### Core Concepts / Features

1. **Attribute Routing** – The overarching strategy and its configuration.
2. **`[Route]`** – The foundational attribute for defining route templates at the controller or action level.
3. **`[HttpGet]`** – Constrains matching to HTTP GET requests and optionally supplies a route template.
4. **`[HttpPost]`** – Constrains matching to HTTP POST requests.
5. **`[HttpPut]`** – Constrains matching to HTTP PUT requests.
6. **`[HttpPatch]`** – Constrains matching to HTTP PATCH requests.
7. **`[HttpDelete]`** – Constrains matching to HTTP DELETE requests.
8. **Route Parameters** – Named placeholders that capture URL values.
9. **Route Constraints** – Validation rules applied inline to route parameters.
10. **Attribute Routing Best Practices** – Guidelines for maintainable, predictable, and secure route design.

Each concept is elaborated with the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real‑World Cases, and References.

---

## 1. Attribute Routing

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Attribute routing is a declarative routing approach where URL patterns are attached to controllers and actions via attributes, replacing or supplementing centrally defined conventional routes. |
| **Technical Definition** | At startup, the MVC application model scans controller classes and action methods for attributes implementing `IRouteTemplateProvider` (including `RouteAttribute` and the HTTP verb attributes). These attributes are converted into `AttributeRouteModel` instances and combined to produce `RouteEndpoint` entries. The routing middleware then matches requests against these endpoints using the same precedence and constraint evaluation logic as conventional routing. |
| **Beginner‑Friendly Explanation** | Attribute routing is like **labelling each room in a building with its own address** instead of keeping a single directory at the front desk. You write `[Route("api/books")]` above a controller and `[HttpGet("{id}")]` above an action, and the framework builds a direct map from URLs to code. This makes it very clear which URL calls which piece of logic. |

### Purposes (each begins with “To”)

- **To** define routes directly on the controller or action, keeping the URL definition physically close to the code it executes.
- **To** support fine‑grained per‑action control over route templates, optional parameters, and constraints.
- **To** model REST APIs naturally by combining resource‑oriented templates with HTTP verb attributes.
- **To** enable multiple routes to reach the same action when backward compatibility or multiple verbs are required.
- **To** leverage token replacement (`[controller]`, `[action]`) for convention‑based URL prefixes without repeating literal names.

### Syntax Rules and Structure

#### General Syntax

```csharp
[Route("api/[controller]")]          // Controller-level route prefix
[ApiController]
public class BooksController : ControllerBase
{
    [HttpGet]                         // GET api/Books
    public IActionResult GetAll() { ... }

    [HttpGet("{id:int}")]            // GET api/Books/5
    public IActionResult GetById(int id) { ... }

    [HttpPost]                        // POST api/Books
    public IActionResult Create(...) { ... }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[Route("api/[controller]")]` | Controller‑level template. `[controller]` is replaced with the controller name without the “Controller” suffix (`Books`). |
| `[ApiController]` | Enables API‑specific behaviours (automatic model validation, binding source inference). |
| `[HttpGet]` | Constrains the action to HTTP GET; the route template is inherited from the controller. |
| `[HttpGet("{id:int}")]` | Combines the controller prefix with `{id:int}` → `api/Books/{id:int}`. |

#### Syntax Rules

- **Controller‑level `[Route]`** applies a prefix to all actions in the controller. Action‑level templates are appended to this prefix.
- **Token replacement** is performed as the final step of building attribute routes. `[controller]` and `[action]` are replaced with the controller and action names.
- **Absolute vs relative templates** – A template starting with `/` or `~/` overrides the controller prefix. Relative templates are appended.
- **Multiple route attributes** on a single action create multiple endpoints for that action. Each combines independently with the controller template.
- **Attribute‑routed actions are exclusive** – Placing a route attribute on a controller or action makes it attribute‑routed; it cannot be reached through conventional routes, and conventionally routed actions cannot be reached through attribute routes.

#### Constraints and Limitations

- **Explicit templates required** – Unlike conventional routing, attribute routing does not infer templates from controller/action names unless token replacement is used. Every attribute‑routed action needs at least one route template (directly or via the controller prefix).
- **Route order** – Use `Order` on the attribute (or via a custom `IRouteTemplateProvider`) to control precedence when templates could be ambiguous. Overusing `Order` can mask design problems.
- **Multiple routes can cause ambiguity** – While an action can have multiple routes, keeping the URL space simple and well defined is recommended. Use multiple routes only where needed (e.g., supporting legacy clients).

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic Attribute‑Routed API Controller

`Controllers/BooksController.cs`

```csharp 
using Microsoft.AspNetCore.Mvc;

[Route("api/[controller]")]          // → api/Books
[ApiController]
public class BooksController : ControllerBase
{
    [HttpGet]                         // GET api/Books
    public IActionResult GetAll() => Ok(new[] { "Book1", "Book2" });

    [HttpGet("{id:int}")]            // GET api/Books/5
    public IActionResult GetById(int id) => Ok($"Book {id}");

    [HttpPost]                        // POST api/Books
    public IActionResult Create([FromBody] string title) => Created("", title);
}
```

`Program.cs`
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers(); // Discover attribute-routed controllers
app.Run();
```

**Step‑by‑step explanation:**

1. `[Route("api/[controller]")]` sets the controller prefix to `api/Books` after token replacement.
2. `[HttpGet]` on `GetAll` inherits the controller prefix and matches `GET api/Books`.
3. `[HttpGet("{id:int}")]` on `GetById` appends `{id:int}` to the prefix, matching `GET api/Books/5` but rejecting non‑integer IDs.
4. `[HttpPost]` on `Create` matches `POST api/Books`.
5. `app.MapControllers()` registers all attribute‑routed controllers.

**Expected Output:**

| HTTP Method | URL | Action Invoked |
|-------------|-----|----------------|
| GET | `/api/Books` | `GetAll` |
| GET | `/api/Books/7` | `GetById(7)` |
| GET | `/api/Books/abc` | **404** (constraint fails) |
| POST | `/api/Books` | `Create` |

**Why this output?** The `int` constraint on `{id:int}` rejects non‑integer values before the action is selected. Token replacement converts `[controller]` to `Books`, producing clean, resource‑oriented URLs.

---

#### Example 2: Multiple Routes on a Single Action

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("list")]
    [HttpGet("all")]
    public IActionResult GetAll() => Ok("All products");
}
```

**Step‑by‑step explanation:**

- Two `[HttpGet]` attributes are placed on the same action.
- Each attribute creates a separate endpoint: `GET api/Products/list` and `GET api/Products/all`.
- Both endpoints invoke the same `GetAll` action.

**Expected Output:**

| URL | Action |
|-----|--------|
| `/api/Products/list` | `GetAll` |
| `/api/Products/all` | `GetAll` |

**Why this output?** Multiple route attributes allow one action to respond to multiple URL patterns. This is useful for backward compatibility or providing aliases, but should be used sparingly to keep the URL space well defined.

---

#### Example 3: Mixing Attribute Routing with Conventional Routing

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapControllers(); // Attribute-routed controllers

// HomeController (conventionally routed – no [Route])
public class HomeController : Controller
{
    public IActionResult Index() => Content("Home");
}

// ApiBooksController (attribute-routed – has [Route])
[Route("api/books")]
[ApiController]
public class ApiBooksController : ControllerBase
{
    [HttpGet] // GET api/books
    public IActionResult GetAll() => Ok("API Books");
}
```

**Step‑by‑step explanation:**

1. The conventional default route handles `HomeController` because it has no route attributes.
2. `ApiBooksController` has `[Route("api/books")]`, making it attribute‑routed. It is matched by `MapControllers()` and **not** by the conventional route.
3. This separation follows the recommended pattern: conventional routing for HTML‑serving controllers, attribute routing for REST APIs.

**Expected Output:**

| URL | Matched By | Action |
|-----|------------|--------|
| `/` | Conventional `default` | `HomeController.Index` |
| `/api/books` | Attribute route | `ApiBooksController.GetAll` |

**Why this output?** Attribute‑routed controllers are excluded from conventional routing, and conventionally routed controllers are excluded from attribute routing. The two strategies coexist in the same application but apply to different controllers.

### Real‑World Cases

- **RESTful product API:** `[Route("api/[controller]")]` on `ProductsController` with `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]` actions provides a complete CRUD API with minimal configuration.
- **Versioned API:** `[Route("api/v{version:apiVersion}/[controller]")]` allows multiple API versions to coexist, with each version’s controllers defining their own templates.
- **Hybrid MVC + API application:** Conventional routes handle Razor view controllers, while attribute‑routed controllers expose JSON APIs under `/api/...`.

### References

- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis
- `IRouteTemplateProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routing.iroutetemplateprovider

---

## 2. `[Route]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Route]` is the foundational attribute for attribute routing, used to attach a route template to a controller class or an action method. |
| **Technical Definition** | `RouteAttribute` implements `IRouteTemplateProvider`. It supplies a `Template` string, an optional `Order` (int?), and an optional `Name` (string). At startup, the MVC application model collects `[Route]` attributes, performs token replacement, and combines controller‑level and action‑level templates into `AttributeRouteModel` entries used by the routing middleware. |
| **Beginner‑Friendly Explanation** | `[Route]` is the **address sticker** you put on a controller or action. On a controller, it sets a common prefix (like `api/products`). On an action, it adds the specific path (like `{id}`). Together they tell the framework: “This code handles URLs that look like this.” |

### Purposes (each begins with “To”)

- **To** define a URL template directly on a controller or action without central configuration.
- **To** establish a shared URL prefix for all actions in a controller.
- **To** support token replacement (`[controller]`, `[action]`) so that templates stay in sync with class and method names.
- **To** allow optional `Order` and `Name` metadata for precedence control and URL generation.
- **To** serve as the basis for HTTP verb attributes, which inherit from `HttpMethodAttribute` and ultimately implement `IRouteTemplateProvider`.

### Syntax Rules and Structure

#### General Syntax

```csharp
[Route("template")]                          // On a controller or action
[Route("template", Order = 1)]               // With precedence order
[Route("template", Name = "routeName")]      // With a route name for URL generation
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Template` | The route template string (e.g., `"api/[controller]"`, `"{id:int}"`). |
| `Order` | Optional integer controlling precedence when multiple routes could match. Lower values are matched first. |
| `Name` | Optional unique name used by `IUrlHelper.RouteUrl` and `LinkGenerator`. |

#### Syntax Rules

- **Controller‑level `[Route]`** applies to all actions in the controller. Action‑level templates are **appended** to the controller prefix unless they start with `/` or `~/` (absolute override).
- **Token replacement** occurs as the **last step** of building attribute routes: `[controller]` → controller name without “Controller”, `[action]` → action method name.
- **Escape token delimiters** – To use literal `[` or `]`, repeat the character (`[[` or `]]`).
- **Multiple `[Route]` attributes** can be placed on the same action to create multiple endpoints.

#### Constraints and Limitations

- **No template inference** – Unlike conventional routing, `[Route]` does not infer a template from the controller or action name unless tokens are used. An attribute‑routed action must have at least one template.
- **Controller‑level `[Route]` excludes conventional routing** – Any route attribute on the controller makes **all** actions in that controller attribute‑routed.
- **Order usage** – Overusing `Order` can make routing harder to reason about. Rely on template precedence first; use `Order` only for genuine ambiguities.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Controller‑Level and Action‑Level Templates

```csharp
[Route("api/[controller]")]          // → api/Customers
[ApiController]
public class CustomersController : ControllerBase
{
    [Route("")]                       // → api/Customers (empty action template)
    [HttpGet]
    public IActionResult GetAll() => Ok();

    [Route("{id:int}")]              // → api/Customers/{id:int}
    [HttpGet]
    public IActionResult GetById(int id) => Ok(id);
}
```

**Step‑by‑step explanation:**

1. `[Route("api/[controller]")]` sets the controller prefix to `api/Customers`.
2. `[Route("")]` on `GetAll` produces the full template `api/Customers`.
3. `[Route("{id:int}")]` on `GetById` produces `api/Customers/{id:int}`.

**Expected Output:**

| URL | Action |
|-----|--------|
| `GET /api/Customers` | `GetAll` |
| `GET /api/Customers/5` | `GetById(5)` |

**Why this output?** The controller prefix is combined with each action template. An empty action template (`""`) uses the controller prefix as‑is.

---

#### Example 2: Token Replacement in Action Templates

```csharp
[Route("[controller]/[action]")]     // → Books/GetAll, Books/GetById, ...
public class BooksController : Controller
{
    public IActionResult GetAll() => Content("All books");
    public IActionResult GetById(int id) => Content($"Book {id}");
}
```

**Step‑by‑step explanation:**

- `[controller]` is replaced with `Books`.
- `[action]` is replaced with the action method name (`GetAll`, `GetById`).
- The resulting templates are `Books/GetAll` and `Books/GetById` (with `{id?}` implicitly available through model binding).

**Expected Output:**

| URL | Action |
|-----|--------|
| `/Books/GetAll` | `GetAll` |
| `/Books/GetById/3` | `GetById(3)` |

**Why this output?** Token replacement converts the generic template into specific URLs at startup. This is useful when you want a consistent `controller/action` structure without repeating literal names.

---

#### Example 3: Absolute Template Override

```csharp
[Route("api/[controller]")]
[ApiController]
public class ReportsController : ControllerBase
{
    [HttpGet("~/reports/summary")]   // Absolute override – ignores controller prefix
    public IActionResult Summary() => Ok("Summary");
}
```

**Step‑by‑step explanation:**

- The action template starts with `~/`, making it **absolute**. It overrides the controller prefix.
- The resulting route is `reports/summary`, **not** `api/Reports/reports/summary`.

**Expected Output:**

| URL | Action |
|-----|--------|
| `GET /reports/summary` | `Summary` |
| `GET /api/Reports/reports/summary` | **404** |

**Why this output?** The `~/` prefix signals that the template is absolute and should not be combined with the controller‑level template.

### Real‑World Cases

- **Resource‑oriented API:** `[Route("api/[controller]")]` provides a consistent base for all CRUD endpoints.
- **Admin area prefix:** `[Route("admin/[controller]")]` groups administrative endpoints under `/admin/...` without affecting public APIs.
- **Custom legacy URL support:** `[Route("old-products")]` as an absolute template on an action allows legacy clients to continue using an old URL while the new API uses `api/products`.

### References

- `RouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- Token replacement in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing

---

## 3. `[HttpGet]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HttpGet]` is an attribute that constrains an action to respond only to HTTP GET requests and optionally supplies a route template. |
| **Technical Definition** | `HttpGetAttribute` derives from `HttpMethodAttribute`, which implements `IActionConstraint` and `IRouteTemplateProvider`. It sets the HTTP method constraint to `GET` and, if a template is provided, contributes that template to the action’s attribute route. During matching, only GET requests are considered for the endpoint. |
| **Beginner‑Friendly Explanation** | `[HttpGet]` is like putting a sign on a door that says **“GET requests only.”** When a browser or API client asks to read data (using an HTTP GET), this action handles it. If someone tries to POST to the same URL, this action is skipped. |

### Purposes (each begins with “To”)

- **To** restrict an action to HTTP GET requests, enforcing RESTful semantics (GET = safe, read‑only retrieval).
- **To** optionally supply a route template that combines with the controller prefix.
- **To** enable the routing middleware to reject non‑GET requests before the action executes.
- **To** work seamlessly with `[ApiController]` to produce 405 Method Not Allowed responses when the URL matches but the verb does not.
- **To** allow multiple `[HttpGet]` attributes on one action for aliases or backward compatibility.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpGet]                              // Inherits controller prefix; matches GET to the prefix
[HttpGet("template")]                  // Appends template to controller prefix
[HttpGet("template", Order = 1)]       // With precedence order
[HttpGet("template", Name = "routeName")] // With route name
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `template` (optional) | Route template appended to the controller‑level template. |
| `Order` (optional) | Precedence order. |
| `Name` (optional) | Route name for URL generation. |

#### Syntax Rules

- **Without a template**, `[HttpGet]` matches the controller prefix exactly (if no other action template is supplied).
- **With a template**, the template is appended to the controller prefix unless it starts with `/` or `~/` (absolute).
- **HTTP method constraint** – Only GET requests are matched. Other verbs are rejected (typically 405 if another action matches the URL with a different verb).
- **Multiple verb attributes** – An action can have both `[HttpGet]` and `[HttpPost]` to respond to multiple verbs on the same URL.

#### Constraints and Limitations

- **Idempotency assumption** – GET should not modify server state. Using `[HttpGet]` on an action that mutates data violates HTTP semantics and can cause problems with caching, prefetching, and crawlers.
- **No automatic template from action name** – `[HttpGet]` alone does not create a template from the action name; it relies on the controller prefix or an explicit template.
- **Ambiguity with multiple GET actions** – If two GET actions have templates that could match the same URL with equal precedence, an `AmbiguousMatchException` is thrown.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic GET Actions

```csharp
[Route("api/[controller]")]
[ApiController]
public class CustomersController : ControllerBase
{
    [HttpGet]                         // GET api/Customers
    public IActionResult GetAll() => Ok("All customers");

    [HttpGet("{id:int}")]            // GET api/Customers/5
    public IActionResult GetById(int id) => Ok($"Customer {id}");
}
```

**Step‑by‑step explanation:**

- `GetAll` matches `GET api/Customers` (no extra template).
- `GetById` matches `GET api/Customers/5` (template `{id:int}` appended).

**Expected Output:**

| Method | URL | Action |
|--------|-----|--------|
| GET | `/api/Customers` | `GetAll` |
| GET | `/api/Customers/3` | `GetById(3)` |
| POST | `/api/Customers` | **405** (no POST action) |

**Why this output?** `[HttpGet]` constrains matching to GET. A POST to the same URL finds no matching endpoint and returns 405 Method Not Allowed.

---

#### Example 2: Multiple GET Templates on One Action

```csharp
[Route("api/[controller]")]
[ApiController]
public class ReportsController : ControllerBase
{
    [HttpGet("daily")]
    [HttpGet("daily/{date:datetime}")]
    public IActionResult GetDaily(DateTime? date = null) =>
        Ok($"Daily report for {date?.ToString("yyyy-MM-dd") ?? "today"}");
}
```

**Step‑by‑step explanation:**

- Two `[HttpGet]` attributes create two endpoints: `api/Reports/daily` and `api/Reports/daily/{date:datetime}`.
- The same action handles both, using an optional `DateTime?` parameter.

**Expected Output:**

| URL | Response |
|-----|----------|
| `GET /api/Reports/daily` | `Daily report for today` |
| `GET /api/Reports/daily/2025-03-15` | `Daily report for 2025-03-15` |

**Why this output?** Multiple `[HttpGet]` attributes allow one action to serve both a parameterless and a parameterised GET endpoint. The `datetime` constraint validates the date segment.

---

#### Example 3: `[HttpGet]` Without a Controller Prefix

```csharp
[ApiController]
public class StatusController : ControllerBase
{
    [HttpGet("health")]               // → GET /health (no controller prefix)
    public IActionResult Health() => Ok("Healthy");
}
```

**Step‑by‑step explanation:**

- No `[Route]` on the controller, so the action template `health` is used as‑is.
- The endpoint matches `GET /health`.

**Expected Output:**

| URL | Action |
|-----|--------|
| `GET /health` | `Health` |

**Why this output?** Without a controller‑level prefix, the action template becomes the full route. This is useful for health checks, metrics, and other infrastructure endpoints that should not be nested under a controller name.

### Real‑World Cases

- **Read‑only resource retrieval:** `GET /api/products` (list) and `GET /api/products/42` (detail) are the canonical GET endpoints in a REST API.
- **Health checks:** `[HttpGet("health")]` on a controller without a route prefix exposes `/health` for load balancers and monitoring.
- **Search endpoints:** `[HttpGet("search")]` with query parameters (e.g., `?q=routing`) handles read‑only search without creating a new resource.

### References

- `HttpGetAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpgetattribute
- HTTP verb attributes in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes

---

## 4. `[HttpPost]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HttpPost]` is an attribute that constrains an action to respond only to HTTP POST requests and optionally supplies a route template. |
| **Technical Definition** | `HttpPostAttribute` derives from `HttpMethodAttribute` and implements `IActionConstraint`. It sets the HTTP method constraint to `POST`. During matching, only POST requests are considered for endpoints decorated with this attribute. |
| **Beginner‑Friendly Explanation** | `[HttpPost]` is the **“submit” sign** on an action. When a form is submitted or an API client sends data to create a new resource, the POST action handles it. GET requests to the same URL are ignored. |

### Purposes (each begins with “To”)

- **To** restrict an action to HTTP POST requests, following RESTful semantics for resource creation.
- **To** optionally supply a route template (often the same as the GET collection URL).
- **To** ensure that data‑mutating creation operations are only triggered by POST, not by GET (which could happen accidentally via browser navigation or crawlers).
- **To** work with `[FromBody]` and model binding to accept JSON or form data.
- **To** support multiple POST templates for different creation scenarios.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpPost]                             // Inherits controller prefix
[HttpPost("template")]                 // Appends template
[HttpPost("template", Order = 1)]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `template` (optional) | Route template appended to the controller prefix. |
| `Order` / `Name` | Same semantics as `[HttpGet]`. |

#### Syntax Rules

- **POST actions typically share the collection URL** with GET list actions (e.g., `POST /api/Products` to create a product, while `GET /api/Products` lists products). The HTTP method distinguishes them.
- **Model binding** – POST data is usually bound with `[FromBody]` (for JSON) or `[FromForm]` (for form‑encoded data).
- **Status code** – Successful creation typically returns `201 Created` with a `Location` header pointing to the new resource.

#### Constraints and Limitations

- **Non‑idempotent** – POST is not idempotent; repeating the same request may create duplicate resources. This is semantically correct but should be documented.
- **CSRF protection** – For browser‑facing POST actions, use `[ValidateAntiForgeryToken]` or equivalent CSRF protection.
- **No automatic `id` in POST URL** – POST creation typically targets the collection, not an individual resource; an `id` is usually generated by the server.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic POST Creation Action

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpPost]                         // POST api/Products
    public IActionResult Create([FromBody] ProductDto dto)
    {
        var id = 42; // Simulate database-generated ID
        return CreatedAtAction(nameof(GetById), new { id }, dto);
    }

    [HttpGet("{id:int}")]
    public IActionResult GetById(int id) => Ok(id);
}
```

**Step‑by‑step explanation:**

1. `[HttpPost]` matches `POST api/Products`.
2. `[FromBody]` binds the JSON request body to `ProductDto`.
3. `CreatedAtAction` returns `201 Created` with a `Location` header pointing to `GetById`.

**Expected Output:**

| Method | URL | Status | Response |
|--------|-----|--------|----------|
| POST | `/api/Products` (with JSON body) | 201 Created | `Location: /api/Products/42` |
| GET | `/api/Products/42` | 200 OK | Product data |

**Why this output?** The POST action creates a resource and returns the canonical URL for retrieving it. The GET action on the same controller retrieves the created resource.

---

#### Example 2: POST with a Specific Template

```csharp
[Route("api/[controller]")]
[ApiController]
public class OrdersController : ControllerBase
{
    [HttpPost("checkout")]             // POST api/Orders/checkout
    public IActionResult Checkout([FromBody] CartDto cart) => Ok("Order placed");

    [HttpPost("cancel/{id:int}")]      // POST api/Orders/cancel/5
    public IActionResult Cancel(int id) => Ok($"Order {id} cancelled");
}
```

**Step‑by‑step explanation:**

- `[HttpPost("checkout")]` matches `POST api/Orders/checkout`.
- `[HttpPost("cancel/{id:int}")]` matches `POST api/Orders/cancel/5`.

**Expected Output:**

| URL | Action |
|-----|--------|
| `POST /api/Orders/checkout` | `Checkout` |
| `POST /api/Orders/cancel/5` | `Cancel(5)` |

**Why this output?** POST actions can have specific templates that represent operations (checkout, cancel) rather than simple resource creation.

---

#### Example 3: POST and GET on the Same URL

```csharp
[Route("api/[controller]")]
[ApiController]
public class CommentsController : ControllerBase
{
    [HttpGet]                          // GET api/Comments
    public IActionResult GetAll() => Ok("All comments");

    [HttpPost]                         // POST api/Comments
    public IActionResult Create([FromBody] string comment) => Created("", comment);
}
```

**Step‑by‑step explanation:**

- Both actions use the same template (`api/Comments`), but different HTTP method attributes.
- `[HttpGet]` handles retrieval; `[HttpPost]` handles creation.
- The routing middleware distinguishes them by HTTP verb.

**Expected Output:**

| Method | URL | Action |
|--------|-----|--------|
| GET | `/api/Comments` | `GetAll` |
| POST | `/api/Comments` | `Create` |

**Why this output?** The HTTP method attributes act as discriminators. The same URL can have different behaviour depending on the verb, which is fundamental to RESTful API design.

### Real‑World Cases

- **Resource creation:** `POST /api/customers` with a JSON body creates a new customer.
- **Form submission:** A Razor view posts to `POST /Orders/Checkout` with form data, protected by `[ValidateAntiForgeryToken]`.
- **Action‑oriented endpoints:** `POST /api/orders/{id}/cancel` and `POST /api/orders/{id}/refund` model operations that are not simple CRUD.

### References

- `HttpPostAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httppostattribute
- HTTP verb attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes

---

## 5. `[HttpPut]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HttpPut]` is an attribute that constrains an action to respond only to HTTP PUT requests and optionally supplies a route template. |
| **Technical Definition** | `HttpPutAttribute` derives from `HttpMethodAttribute` and sets the HTTP method constraint to `PUT`. PUT is defined as **idempotent**: replacing a resource with the same data multiple times produces the same result. |
| **Beginner‑Friendly Explanation** | `[HttpPut]` is for **full replacement**. When a client wants to completely replace an existing resource (e.g., update all fields of a product), it sends a PUT request. The action decorated with `[HttpPut]` handles that replacement. |

### Purposes (each begins with “To”)

- **To** restrict an action to HTTP PUT requests, enforcing idempotent full‑replacement semantics.
- **To** ensure that update operations that replace the entire resource are distinguished from partial updates (PATCH).
- **To** optionally supply a route template, typically including the resource identifier (`{id:int}`).
- **To** support model binding of the replacement resource from the request body.
- **To** return appropriate status codes (204 No Content or 200 OK) after a successful replacement.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpPut("{id:int}")]                  // PUT api/Products/5
[HttpPut]                              // PUT to controller prefix (less common for updates)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `template` | Typically includes `{id}` to identify the resource to replace. |
| `Order` / `Name` | Same semantics as other verb attributes. |

#### Syntax Rules

- **Idempotency** – PUT must be idempotent: applying the same request multiple times yields the same server state.
- **Full replacement** – The request body should contain the complete representation of the resource, not just the changed fields.
- **Route template** – Usually includes the resource ID so the server knows which resource to replace.

#### Constraints and Limitations

- **Not for partial updates** – Use `[HttpPatch]` for partial updates. Using PUT for partial updates violates the principle of full replacement and can lead to data loss if clients omit fields expecting them to remain unchanged.
- **Concurrency** – PUT operations should consider concurrency (e.g., ETags, row versions) to avoid lost updates.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Full Resource Replacement with PUT

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpPut("{id:int}")]              // PUT api/Products/5
    public IActionResult Replace(int id, [FromBody] ProductDto dto)
    {
        // Replace the entire product with the supplied data
        return NoContent(); // 204
    }
}
```

**Step‑by‑step explanation:**

1. `[HttpPut("{id:int}")]` matches `PUT api/Products/5`.
2. The entire `ProductDto` is bound from the request body.
3. The action replaces the resource and returns `204 No Content`.

**Expected Output:**

| Method | URL | Status |
|--------|-----|--------|
| PUT | `/api/Products/5` (with full JSON body) | 204 No Content |
| PUT | `/api/Products/abc` | **404** (constraint fails) |

**Why this output?** The `int` constraint on `{id:int}` rejects non‑integer IDs. The PUT action semantically replaces the entire resource.

---

#### Example 2: PUT with a Custom Template

```csharp
[Route("api/[controller]")]
[ApiController]
public class InventoryController : ControllerBase
{
    [HttpPut("stock/{sku}")]           // PUT api/Inventory/stock/ABC-123
    public IActionResult UpdateStock(string sku, [FromBody] int quantity) =>
        Ok($"Stock for {sku} set to {quantity}");
}
```

**Step‑by‑step explanation:**

- The template `stock/{sku}` allows PUT to a non‑ID identifier (SKU).
- The action replaces the stock quantity for the given SKU.

**Expected Output:**

| Method | URL | Response |
|--------|-----|----------|
| PUT | `/api/Inventory/stock/ABC-123` | `Stock for ABC-123 set to 10` |

**Why this output?** PUT is not limited to integer IDs; any resource identifier can be used in the template.

### Real‑World Cases

- **Full product update:** `PUT /api/products/42` replaces all fields of product 42 with the supplied representation.
- **Configuration replacement:** `PUT /api/settings/theme` replaces the entire theme configuration with the new one.
- **Inventory level set:** `PUT /api/inventory/stock/SKU-123` sets the stock quantity to an absolute value (idempotent).

### References

- `HttpPutAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpputattribute
- HTTP verb attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes

---

## 6. `[HttpPatch]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HttpPatch]` is an attribute that constrains an action to respond only to HTTP PATCH requests and optionally supplies a route template. |
| **Technical Definition** | `HttpPatchAttribute` derives from `HttpMethodAttribute` and sets the HTTP method constraint to `PATCH`. PATCH is used for **partial updates**: the request body contains only the fields to be modified, not the complete resource. |
| **Beginner‑Friendly Explanation** | `[HttpPatch]` is for **partial changes**. Instead of sending the whole product to update one field (like changing only the price), you send just the field that changed. The PATCH action applies only those changes. |

### Purposes (each begins with “To”)

- **To** restrict an action to HTTP PATCH requests, supporting partial updates of a resource.
- **To** distinguish partial‑update semantics from full replacement (PUT) and creation (POST).
- **To** allow clients to send minimal payloads containing only the fields that need to change.
- **To** optionally supply a route template that identifies the resource to be partially updated.
- **To** work with JSON Patch documents (`application/json-patch+json`) or merge‑patch content types.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpPatch("{id:int}")]                // PATCH api/Products/5
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `template` | Typically includes the resource ID. |
| `Order` / `Name` | Same semantics as other verb attributes. |

#### Syntax Rules

- **Partial payload** – The request body should contain only the changes, not the full resource.
- **Content types** – Common PATCH content types include `application/json-patch+json` (RFC 6902) and `application/merge-patch+json` (RFC 7386).
- **Route template** – Must identify the resource to patch; usually includes `{id}`.

#### Constraints and Limitations

- **Not universally supported** – Some clients and proxies do not support PATCH; consider providing PUT as an alternative if PATCH is not viable.
- **Complexity** – JSON Patch operations (add, remove, replace, move, copy, test) require careful handling and validation.
- **Idempotency** – PATCH is not guaranteed to be idempotent (depending on the patch operations), unlike PUT.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic PATCH Action

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpPatch("{id:int}")]            // PATCH api/Products/5
    public IActionResult PartialUpdate(int id, [FromBody] JsonPatchDocument<ProductDto> patchDoc)
    {
        // Apply patch operations to the product
        return NoContent();
    }
}
```

**Step‑by‑step explanation:**

1. `[HttpPatch("{id:int}")]` matches `PATCH api/Products/5`.
2. `[FromBody] JsonPatchDocument<ProductDto>` binds a JSON Patch document.
3. The action applies the patch operations and returns `204 No Content`.

**Expected Output:**

| Method | URL | Status |
|--------|-----|--------|
| PATCH | `/api/Products/5` (with JSON Patch body) | 204 No Content |

**Why this output?** PATCH allows partial modification: the client sends only the operations to perform, and the server applies them to the existing resource.

---

#### Example 2: PATCH with a Merge Patch Document

```csharp
[HttpPatch("{id:int}")]
public IActionResult MergeUpdate(int id, [FromBody] ProductPatchDto patch)
{
    // Apply only non-null fields from patch
    return Ok();
}

public class ProductPatchDto
{
    public string? Name { get; set; }
    public decimal? Price { get; set; }
}
```

**Step‑by‑step explanation:**

- The action accepts a DTO with nullable properties.
- Only the properties present in the request body are applied; `null` properties are left unchanged.

**Expected Output:**

| Request Body | Effect |
|--------------|--------|
| `{"price": 19.99}` | Only price is updated; name remains unchanged. |
| `{"name": "New Name"}` | Only name is updated; price remains unchanged. |

**Why this output?** Partial update semantics: the action selectively applies only the fields provided, leaving others intact.

### Real‑World Cases

- **Price update:** `PATCH /api/products/42` with `{"price": 19.99}` changes only the price.
- **Profile editing:** `PATCH /api/users/7` with `{"email": "new@example.com"}` updates only the email address.
- **Status change:** `PATCH /api/orders/100` with `{"status": "shipped"}` updates only the order status.

### References

- `HttpPatchAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httppatchattribute
- HTTP verb attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes

---

## 7. `[HttpDelete]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HttpDelete]` is an attribute that constrains an action to respond only to HTTP DELETE requests and optionally supplies a route template. |
| **Technical Definition** | `HttpDeleteAttribute` derives from `HttpMethodAttribute` and sets the HTTP method constraint to `DELETE`. DELETE requests are intended to remove the identified resource. |
| **Beginner‑Friendly Explanation** | `[HttpDelete]` is the **“remove” sign**. When a client wants to delete a resource (like removing a product), it sends a DELETE request. The action decorated with `[HttpDelete]` performs the deletion. |

### Purposes (each begins with “To”)

- **To** restrict an action to HTTP DELETE requests, following RESTful semantics for resource removal.
- **To** optionally supply a route template that identifies the resource to delete (typically `{id:int}`).
- **To** ensure that deletion cannot be triggered by GET or other safe methods.
- **To** return appropriate status codes (204 No Content or 200 OK) after deletion.
- **To** support cascade deletes or soft deletes depending on business logic.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HttpDelete("{id:int}")]               // DELETE api/Products/5
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `template` | Usually includes the resource identifier. |
| `Order` / `Name` | Same semantics as other verb attributes. |

#### Syntax Rules

- **Idempotency** – DELETE is idempotent: deleting the same resource multiple times produces the same result (the resource is gone). The second delete may return 404, but the server state is the same.
- **Route template** – Must identify the resource to delete.
- **Status code** – Common responses are `204 No Content` (success, no body) or `200 OK` (success with a confirmation body).

#### Constraints and Limitations

- **Destructive** – DELETE is irreversible in many implementations; consider soft deletes for auditability.
- **Authorization** – Deletion should be tightly authorized; use `[Authorize]` and role/policy checks.
- **CSRF** – Browser‑facing DELETE endpoints may need CSRF protection if triggered from JavaScript.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic DELETE Action

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpDelete("{id:int}")]           // DELETE api/Products/5
    public IActionResult Delete(int id) => NoContent(); // 204
}
```

**Step‑by‑step explanation:**

1. `[HttpDelete("{id:int}")]` matches `DELETE api/Products/5`.
2. The action performs the deletion (simulated here) and returns `204 No Content`.

**Expected Output:**

| Method | URL | Status |
|--------|-----|--------|
| DELETE | `/api/Products/5` | 204 No Content |
| DELETE | `/api/Products/abc` | **404** (constraint fails) |

**Why this output?** The `int` constraint ensures only numeric IDs reach the delete action. The action returns 204 to indicate successful deletion without a response body.

---

#### Example 2: DELETE with a Named Route

```csharp
[HttpDelete("{id:int}", Name = "DeleteProduct")]
public IActionResult Delete(int id) => NoContent();
```

**Step‑by‑step explanation:**

- The `Name` property assigns a route name (`"DeleteProduct"`).
- This name can be used with `Url.RouteUrl("DeleteProduct", new { id = 5 })` to generate the URL.

**Expected Output:**

| URL Generation Call | Result |
|---------------------|--------|
| `Url.RouteUrl("DeleteProduct", new { id = 5 })` | `/api/Products/5` |

**Why this output?** Named routes make URL generation robust: client code refers to the route by name, and if the template changes, generated links update automatically.

### Real‑World Cases

- **Product deletion:** `DELETE /api/products/42` removes product 42 from the catalogue.
- **Account closure:** `DELETE /api/users/7` deletes or deactivates user 7.
- **Cart item removal:** `DELETE /api/cart/items/99` removes item 99 from the shopping cart.

### References

- `HttpDeleteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpdeleteattribute
- HTTP verb attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes

---

## 8. Route Parameters

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route parameters are named placeholders in an attribute route template that capture values from the URL and make them available to the action method. |
| **Technical Definition** | In attribute routing, route parameters are defined inside `{}` in the template string. During matching, the routing middleware tokenises the URL path, assigns segments to parameters, and stores the values in `RouteData.Values`. Model binding then converts these string values to the action method’s parameter types. Attribute routes support the same inline syntax as conventional routes for optional parameters, default values, and constraints. |
| **Beginner‑Friendly Explanation** | Route parameters are the **blanks** in an attribute route. In `[HttpGet("{id:int}")]`, the `{id:int}` is a blank that must be filled with an integer. If the URL is `/api/Books/5`, the blank is filled with `5`, and the action receives `id = 5`. |

### Purposes (each begins with “To”)

- **To** capture variable data from the URL path and pass it to the action method.
- **To** create hierarchical, resource‑oriented URLs (e.g., `/customers/5/orders/10`).
- **To** support optional and default values so that one action can handle multiple URL shapes.
- **To** enable model binding to automatically populate action parameters from captured values.
- **To** allow URL generation from route values, closing the loop between matching and link creation.

### Syntax Rules and Structure

#### General Syntax

```
{parameterName}
{parameterName?}          // optional
{parameterName=default}   // default value
{parameterName:constraint} // inline constraint
{*parameterName}          // catch-all
```

#### Component Breakdown

| Syntax | Meaning | Route Value Produced? |
|--------|---------|----------------------|
| `{id}` | Required parameter | Yes, when segment present. |
| `{id?}` | Optional parameter | Only if segment present; otherwise no value. |
| `{id=5}` | Parameter with default | Always; default used if segment absent. |
| `{id:int}` | Parameter with constraint | Yes, if value satisfies constraint. |
| `{*path}` | Catch‑all parameter | Captures remainder of URL. |

#### Syntax Rules

- Parameter names are **case‑insensitive** during matching but are exposed with the casing used in the template.
- Optional parameters (`?`) and default values (`=`) must come **after** the parameter name; constraints (`:`) come after the name and before `?` or `=`.
- **Optional route parameters must come after all required parameters and literals** in the same template.
- Catch‑all parameters (`{*name}`) must be the **last** segment.

#### Constraints and Limitations

- **Empty segments** – Optional parameters do not match empty segments; `/api/products/` does not match `{id?}`.
- **Default values always produce a value** – Unlike optional parameters, a parameter with a default always produces a route value, even when absent from the URL.
- **Model binding failures** – If a captured value cannot be converted to the action parameter type (and no constraint prevented the match), model binding may fail.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Required and Optional Parameters in Attribute Routing

```csharp
[Route("api/[controller]")]
[ApiController]
public class ArticlesController : ControllerBase
{
    [HttpGet("{category}")]
    [HttpGet("{category}/{page:int?}")]
    public IActionResult List(string category, int? page) =>
        Ok($"Category: {category}, Page: {page?.ToString() ?? "1"}");
}
```

**Step‑by‑step explanation:**

- Two `[HttpGet]` templates: `{category}` and `{category}/{page:int?}`.
- `category` is required in both.
- `page` is optional and constrained to integers.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/api/Articles/tech` | `Category: tech, Page: 1` |
| `/api/Articles/tech/2` | `Category: tech, Page: 2` |
| `/api/Articles` | **404** (missing required `category`) |

**Why this output?** The required `category` parameter must be present. The optional `page` parameter is `null` when absent, and the action substitutes `"1"` as a default display value.

---

#### Example 2: Default Value Parameter

```csharp
[HttpGet("archive/{year:int}/{month:int=1}")]
public IActionResult Archive(int year, int month) =>
    Ok($"Year: {year}, Month: {month}");
```

**Step‑by‑step explanation:**

- `{year:int}` is required and must be an integer.
- `{month:int=1}` is optional with a default value of `1`; it always produces a value.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/api/Archive/archive/2025` | `Year: 2025, Month: 1` |
| `/api/Archive/archive/2025/6` | `Year: 2025, Month: 6` |

**Why this output?** The default value (`1`) ensures `month` always has a value, even when the segment is absent.

---

#### Example 3: Catch‑All Parameter in Attribute Routing

```csharp
[HttpGet("files/{*path}")]
public IActionResult GetFile(string path) => Ok($"File: {path}");
```

**Step‑by‑step explanation:**

- `{*path}` captures everything after `files/`, including slashes.
- The captured value is URL‑decoded and assigned to `path`.

**Expected Output:**

| URL | `path` Value |
|-----|--------------|
| `/api/Files/files/docs/report.pdf` | `docs/report.pdf` |
| `/api/Files/files/images/logo.png` | `images/logo.png` |

**Why this output?** Catch‑all parameters consume the remainder of the URL, making them ideal for file paths or SPA fallback routes.

### Real‑World Cases

- **Hierarchical resources:** `[HttpGet("customers/{customerId:int}/orders/{orderId:int}")]` captures both identifiers for nested resources.
- **Blog pagination:** `[HttpGet("posts/{category}/{page:int?}")]` supports optional page numbers.
- **File serving:** `[HttpGet("files/{*path}")]` serves nested file paths.

### References

- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- Optional parameters and defaults – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-parameters-and-defaults

---

## 9. Route Constraints

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route constraint is a rule that a route parameter’s value must satisfy for the route to match. |
| **Technical Definition** | In attribute routing, constraints are applied inline using the syntax `{parameter:constraint}`. The constraint implements `IRouteConstraint` and is evaluated during matching. If the constraint returns `false`, the endpoint is discarded. Multiple constraints can be chained with `:` (e.g., `{id:int:min(1)}`). Constraints are used **to disambiguate similar routes**, not for input validation. |
| **Beginner‑Friendly Explanation** | A route constraint is a **gatekeeper** for a parameter. In `{id:int}`, the `int` constraint says: “Only allow this route to match if the value is a whole number.” If someone visits `/api/products/abc`, the `int` gatekeeper says “No,” and the route is skipped. |

### Purposes (each begins with “To”)

- **To** reject URLs that do not conform to the expected data type or format before the action executes.
- **To** disambiguate between overlapping route templates (e.g., `{id:int}` vs. `{slug}`).
- **To** improve performance by eliminating non‑matching endpoints early in the matching process.
- **To** enforce format rules (length, range, regex pattern) at the routing layer.
- **To** provide a first line of defence against malformed input, while recognising that constraints are not a substitute for full validation.

### Syntax Rules and Structure

#### General Syntax (Inline)

```
{parameter:constraintName}
{parameter:constraintName(argument)}
{parameter:constraint1:constraint2}   // multiple constraints
```

#### Built‑in Constraints (Selected)

| Constraint | Example | Matches | Notes |
|------------|---------|---------|-------|
| `int` | `{id:int}` | `123`, `-45` | 32‑bit integer. |
| `bool` | `{active:bool}` | `true`, `FALSE` | Case‑insensitive. |
| `guid` | `{id:guid}` | `CD2C1638-...` | Valid GUID. |
| `long` | `{ticks:long}` | `123456789` | 64‑bit integer. |
| `minlength(n)` | `{name:minlength(4)}` | `Rick` | Minimum string length. |
| `maxlength(n)` | `{name:maxlength(8)}` | `MyFile` | Maximum string length. |
| `length(n)` | `{code:length(5)}` | `ABCDE` | Exact string length. |
| `min(n)` | `{age:min(18)}` | `19` | Minimum numeric value. |
| `max(n)` | `{age:max(120)}` | `91` | Maximum numeric value. |
| `range(min,max)` | `{age:range(18,120)}` | `91` | Numeric range. |
| `alpha` | `{name:alpha}` | `Rick` | Alphabetic characters only. |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | Regular expression. |
| `required` | `{name:required}` | `Rick` | Non‑null value required for URL generation. |

#### Syntax Rules

- Constraints are **case‑insensitive** (`int` and `Int` are equivalent).
- Multiple constraints can be chained: `{id:int:min(1)}`.
- Constraint arguments are passed in parentheses: `{name:minlength(3)}`.
- **Regex constraints** should be anchored (`^...$`) and used with a timeout when processing untrusted input to prevent ReDoS attacks.

#### Constraints and Limitations

- **Not input validation** – Constraints are for route matching, not for business validation. Invalid input that passes a constraint can still cause a 400 or 500 error inside the action. Use constraints to **disambiguate routes**, and perform full validation in the action or with model validation.
- **Invariant culture** – Constraints like `datetime` and `decimal` use the **invariant culture**, which may not match user locale expectations.
- **Regex performance** – Complex regex constraints can be slow and vulnerable to ReDoS; keep them simple and anchored.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: `int` and `min` Constraints

```csharp
[HttpGet("products/{id:int:min(1)}")]
public IActionResult GetProduct(int id) => Ok($"Product {id}");
```

**Step‑by‑step explanation:**

- `{id:int:min(1)}` applies two constraints:
  1. `int` – the value must be an integer.
  2. `min(1)` – the integer must be ≥ 1.

**Expected Output:**

| URL | Match? | Reason |
|-----|--------|--------|
| `/products/5` | Yes | Integer ≥ 1. |
| `/products/0` | No | `min(1)` fails. |
| `/products/-3` | No | `min(1)` fails. |
| `/products/abc` | No | `int` fails. |

**Why this output?** Both constraints must pass for the endpoint to match. This prevents invalid IDs from reaching the action.

---

#### Example 2: `guid` Constraint

```csharp
[HttpGet("orders/{orderId:guid}")]
public IActionResult GetOrder(Guid orderId) => Ok(orderId);
```

**Step‑by‑step explanation:**

- `{orderId:guid}` requires the segment to be a valid GUID.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/orders/3fa85f64-5717-4562-b3fc-2c963f66afa6` | Yes |
| `/orders/123` | **404** (not a GUID) |

**Why this output?** The `guid` constraint validates the format, ensuring only GUID‑format order IDs reach the action.

---

#### Example 3: `regex` Constraint for a Slug

```csharp
[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost(string slug) => Ok($"Post: {slug}");
```

**Step‑by‑step explanation:**

- `regex(^[a-z0-9-]+$)` allows only lowercase letters, digits, and hyphens.
- The `^` and `$` anchors ensure the entire segment matches.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/blog/my-first-post` | Yes |
| `/blog/My-Post` | **404** (uppercase `M` not allowed) |
| `/blog/post_1` | **404** (underscore not allowed) |

**Why this output?** The regex constraint enforces a URL‑friendly slug format.

### Real‑World Cases

- **API ID validation:** `{id:int}` on product IDs prevents non‑numeric IDs from reaching database queries.
- **GUID‑based resources:** `{id:guid}` on order or user IDs ensures only GUID‑format identifiers are accepted.
- **SEO slugs:** `{slug:regex(^[a-z0-9-]+$)}` guarantees clean, lowercase, hyphenated blog post URLs.
- **Pagination limits:** `{page:int:min(1)}` ensures page numbers are positive integers.

### References

- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- `IRouteConstraint` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint
- Inline constraints in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#specifying-attribute-route-optional-parameters-default-values-and-constraints

---

## 10. Attribute Routing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Attribute routing best practices are guidelines for designing attribute‑routed controllers and actions that are predictable, maintainable, RESTful, and secure. |
| **Technical Definition** | Best practices encompass template design (resource‑oriented, plural nouns), token usage, constraint application, route ordering (via precedence or `Order`), separation of conventional and attribute routing, and avoiding overly broad or ambiguous templates. |
| **Beginner‑Friendly Explanation** | Best practices are the **rules of thumb** that keep attribute routing clean. For example: use plural nouns for resource collections (`/api/products`, not `/api/product`), put constraints on IDs, don’t cram multiple unrelated routes onto one action, and use attribute routing for APIs and conventional routing for HTML pages. |

### Purposes (each begins with “To”)

- **To** keep the API URL space simple, consistent, and well defined.
- **To** model REST resources correctly using HTTP verbs and plural noun resource names.
- **To** avoid route conflicts and ambiguity by using constraints and clear template design.
- **To** separate concerns: attribute routing for REST APIs, conventional routing for HTML‑serving controllers.
- **To** improve maintainability by co‑locating routes with the actions they describe.
- **To** ensure that URL generation (links, `CreatedAtAction`, etc.) works reliably through consistent template design.

### Syntax Rules and Structure (Guideline Checklist)

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use attribute routing for REST APIs** | Conventional routing cannot express HTTP verb semantics precisely. | `[HttpGet("api/products/{id:int}")]` |
| **Use conventional routing for HTML views** | Reduces repetitive attributes for similar page structures. | `MapControllerRoute` for `HomeController` |
| **Pluralise resource names** | Follows REST conventions for collections. | `/api/products` (not `/api/product`) |
| **Constrain parameters** | Rejects invalid URLs early; disambiguates routes. | `{id:int}`, `{slug:regex(...)}` |
| **Keep templates simple** | Complex templates are harder to read, debug, and generate links for. | Avoid unnecessary chained segments. |
| **Avoid multiple routes per action unless needed** | Multiple routes make the URL space harder to reason about. | Use only for backward compatibility. |
| **Use `[ApiController]` for API controllers** | Enables automatic 400 responses, binding source inference, and API‑specific behaviours. | `[ApiController]` on all API controllers. |
| **Name routes for URL generation** | Named routes make link generation robust and refactoring‑friendly. | `[HttpGet("{id:int}", Name = "GetProduct")]` |
| **Do not use constraints for validation** | Constraints are for matching; use model validation for input rules. | Validate `id > 0` in the action. |
| **Guard against ReDoS in regex constraints** | Untrusted input with complex regex can cause denial‑of‑service. | Use simple, anchored patterns; pass a timeout. |

#### Constraints and Limitations

- **Attribute routing is exclusive** – Once a controller has `[Route]`, all its actions are attribute‑routed and cannot be reached conventionally. Mixing requires careful planning.
- **Order overuse** – Rely on template precedence (literals > constrained parameters > unconstrained parameters) rather than `Order` wherever possible.
- **URL space complexity** – Every additional template increases the surface area for conflicts; keep it minimal and well documented.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: RESTful API Controller Following Best Practices

```csharp
[Route("api/[controller]")]          // Plural resource: api/Products
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet]                         // GET api/Products
    public IActionResult GetAll() => Ok();

    [HttpGet("{id:int}", Name = "GetProduct")] // GET api/Products/5
    public IActionResult GetById(int id) => Ok(id);

    [HttpPost]                        // POST api/Products
    public IActionResult Create([FromBody] ProductDto dto)
    {
        var id = 42;
        return CreatedAtAction(nameof(GetById), new { id }, dto);
    }

    [HttpPut("{id:int}")]            // PUT api/Products/5
    public IActionResult Replace(int id, [FromBody] ProductDto dto) => NoContent();

    [HttpPatch("{id:int}")]          // PATCH api/Products/5
    public IActionResult PartialUpdate(int id, [FromBody] JsonPatchDocument<ProductDto> patch) => NoContent();

    [HttpDelete("{id:int}")]         // DELETE api/Products/5
    public IActionResult Delete(int id) => NoContent();
}
```

**Step‑by‑step explanation:**

1. `[Route("api/[controller]")]` gives a consistent plural resource prefix (`api/Products`).
2. Each action uses the correct HTTP verb attribute.
3. `{id:int}` constraints ensure only integer IDs are routed.
4. `Name = "GetProduct"` enables reliable URL generation for the created resource.
5. `CreatedAtAction` uses the named route to produce the `Location` header.

**Expected Output:**

| Method | URL | Status |
|--------|-----|--------|
| GET | `/api/Products` | 200 |
| GET | `/api/Products/5` | 200 |
| POST | `/api/Products` | 201 (Location: `/api/Products/42`) |
| PUT | `/api/Products/5` | 204 |
| PATCH | `/api/Products/5` | 204 |
| DELETE | `/api/Products/5` | 204 |

**Why this output?** The controller models a REST resource with plural naming, correct HTTP verbs, and constrained IDs, following Microsoft’s guidance for attribute‑routed APIs.

---

#### Example 2: Mixing Conventional and Attribute Routing Correctly

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapControllers(); // Attribute-routed controllers

// HTML controller (conventional)
public class HomeController : Controller
{
    public IActionResult Index() => View();
}

// API controller (attribute-routed)
[Route("api/[controller]")]
[ApiController]
public class ApiOrdersController : ControllerBase
{
    [HttpGet] // GET api/ApiOrders
    public IActionResult GetAll() => Ok();
}
```

**Step‑by‑step explanation:**

1. Conventional routing is used for `HomeController`, which serves HTML views.
2. Attribute routing is used for `ApiOrdersController`, which serves JSON APIs.
3. `MapControllers()` discovers attribute‑routed controllers; conventional routes handle the rest.

**Expected Output:**

| URL | Routing Strategy | Action |
|-----|------------------|--------|
| `/` | Conventional | `HomeController.Index` |
| `/api/ApiOrders` | Attribute | `ApiOrdersController.GetAll` |

**Why this output?** The two strategies coexist without conflict because they apply to different controllers. This is the recommended pattern for hybrid MVC + API applications.

---

#### Example 3: Avoiding Ambiguity with Constraints

```csharp
[Route("api/[controller]")]
[ApiController]
public class SearchController : ControllerBase
{
    [HttpGet("{id:int}")]            // api/Search/5
    public IActionResult ById(int id) => Ok($"ID: {id}");

    [HttpGet("{slug}")]              // api/Search/hello-world
    public IActionResult BySlug(string slug) => Ok($"Slug: {slug}");
}
```

**Step‑by‑step explanation:**

- `/api/Search/5` matches both templates, but `{id:int}` has **higher precedence** because of the constraint.
- `/api/Search/hello-world` does not satisfy `int`, so only the slug template matches.
- Without the `int` constraint, both templates would have equal precedence for numeric segments, causing ambiguity.

**Expected Output:**

| URL | Action | Reason |
|-----|--------|--------|
| `/api/Search/5` | `ById(5)` | `int` constraint gives higher precedence. |
| `/api/Search/hello-world` | `BySlug("hello-world")` | `int` constraint fails; slug route matches. |

**Why this output?** Constraints act as discriminators, allowing overlapping templates to coexist deterministically.

### Real‑World Cases

- **E‑commerce API:** A `ProductsController` with `[Route("api/products")]` and verb attributes provides a complete, RESTful CRUD API.
- **Versioned API:** `[Route("api/v{version:apiVersion}/[controller]")]` allows v1 and v2 controllers to coexist with attribute routing.
- **Hybrid application:** Conventional routes serve Razor views (`/`, `/About`), while attribute‑routed controllers expose `/api/...` JSON endpoints.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis
- Mixed routing: attribute vs conventional – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#mixed-routing-attribute-routing-vs-conventional-routing

---

## Summary Table: Attribute Routing at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Attribute Routing** | Co‑located route definitions | `[Route]`, `[HttpGet]`, etc. on controllers/actions | REST APIs and fine‑grained URL control |
| **`[Route]`** | Define templates on controllers/actions | `[Route("api/[controller]")]`, `[Route("{id:int}")]` | Setting prefixes and action templates |
| **`[HttpGet]`** | Handle read‑only retrieval | `[HttpGet]`, `[HttpGet("{id:int}")]` | Listing and fetching resources |
| **`[HttpPost]`** | Handle resource creation | `[HttpPost]`, `[HttpPost("checkout")]` | Creating resources and operations |
| **`[HttpPut]`** | Handle full replacement | `[HttpPut("{id:int}")]` | Idempotent full updates |
| **`[HttpPatch]`** | Handle partial updates | `[HttpPatch("{id:int}")]` | Selective field updates |
| **`[HttpDelete]`** | Handle resource removal | `[HttpDelete("{id:int}")]` | Deleting resources |
| **Route Parameters** | Capture URL values | `{id}`, `{id?}`, `{id=5}`, `{*path}` | Passing data from URL to actions |
| **Route Constraints** | Validate and disambiguate | `{id:int}`, `{id:guid}`, `{slug:regex(...)}` | Rejecting invalid URLs early |
| **Best Practices** | Maintainability and correctness | Plural nouns, constraints, verb attributes, separation | Production‑ready API design |

---

## References (Full List)

- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis
- HTTP verb attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#http-verb-attributes
- Specifying attribute route optional parameters, default values, and constraints – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#specifying-attribute-route-optional-parameters-default-values-and-constraints
- Token replacement in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- Optional parameters and defaults – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-parameters-and-defaults
- `RouteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routeattribute
- `HttpGetAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpgetattribute
- `HttpPostAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httppostattribute
- `HttpPutAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpputattribute
- `HttpPatchAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httppatchattribute
- `HttpDeleteAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.httpdeleteattribute
- `IRouteTemplateProvider` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.routing.iroutetemplateprovider
- `IRouteConstraint` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint
- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Mixed routing: attribute vs conventional – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#mixed-routing-attribute-routing-vs-conventional-routing