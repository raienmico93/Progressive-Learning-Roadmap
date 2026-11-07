# Controller Fundamentals in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). The distinction between `Controller` and `ControllerBase` has been stable since ASP.NET Core 1.0, but the **minimal hosting model** (`Program.cs` with top-level statements) is the default from .NET 6 onward. Attribute routing (`[Route]`, `[HttpGet]`, etc.) is the recommended approach for APIs, while conventional routing remains suitable for view-based controllers. `ControllerBase` is the required base class for web API controllers; `Controller` adds view support for MVC pages.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controllers are the **entry point for HTTP requests** in ASP.NET Core MVC. They are classes that contain public methods (actions), coordinate application logic, and return responses such as views, JSON, or status codes. |
| **Technical Definition** | A controller is a public class that inherits from either `ControllerBase` (for APIs) or `Controller` (for MVC with views). Actions are public methods exposed as HTTP endpoints via routing. Controllers are discovered automatically from the `Controllers` directory and use **model binding** to populate action parameters. `Controller` derives from `ControllerBase` and adds view-rendering capabilities (`View()`, `PartialView()`, `ViewComponent()`). |
| **Beginner-Friendly Explanation** | Controllers are the **traffic directors** of your application. When a request arrives for `/Products/Details/5`, the `ProductsController` handles it. The `Details` action method runs, fetches the product data, and decides what to send back—usually a view (HTML page) or JSON data. Think of a controller as a **coordinator**: it receives the request, calls the right business logic, and returns the result to the user. |

### Key Characteristics

- **Two base classes** – `ControllerBase` for APIs (no view support); `Controller` for MVC pages (adds view support).
- **Automatic discovery** – Controllers are found by convention in the `Controllers` directory; no explicit registration is required.
- **Action methods** – Public methods on the controller become HTTP endpoints through routing.
- **Model binding** – Action parameters are populated automatically from route values, query strings, and request bodies.
- **Thin by design** – Controllers should coordinate, not contain business logic; business rules belong in services.
- **Testable** – Controllers that follow the **Explicit Dependencies Principle** (constructor injection) are easy to unit test.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllers()` or `AddControllersWithViews()`).
- Routing configured (`MapControllers()` or `MapControllerRoute()`).
- Basic understanding of **C# classes**, **interfaces**, and **dependency injection**.

### Related Programming Areas

- **Routing** – Maps URLs to controller actions.
- **Model Binding** – Populates action parameters from request data.
- **Dependency Injection** – Services are injected into controllers via constructors.
- **Filters** – Run before/after action methods for cross-cutting concerns.
- **Action Results** – `IActionResult`, `ActionResult<T>`, and specific return types.
- **API Conventions** – `[ApiController]` enables opinionated API behaviours.

### Core Concepts / Features

1. **Controllers (General)** – The role and responsibilities of controllers.
2. **`Controller`** – The MVC base class with view support.
3. **`ControllerBase`** – The API base class without view support.
4. **Controller Creation** – How to create, name, and organise controllers.
5. **Controller Methods (Actions)** – Public methods that handle requests and return responses.
6. **Controller Best Practices** – Guidelines for thin, testable, and maintainable controllers.

Each concept is elaborated with Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Controllers (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A controller is a class responsible for handling HTTP requests, coordinating application logic, and returning responses to the client. |
| **Technical Definition** | Controllers are public classes with one or more public methods called **actions**. By convention, controllers reside in the `Controllers` directory and their names end with the `Controller` suffix. Actions are exposed as HTTP endpoints through routing. Controllers use **model binding** to populate parameters and return `IActionResult` implementations (views, JSON, status codes). |
| **Beginner-Friendly Explanation** | Controllers are the **managers** of your web application. Each controller handles a group of related pages or API endpoints. For example, a `ProductsController` handles everything related to products: listing, viewing details, creating, editing, and deleting. It doesn't do the heavy lifting itself—it delegates to services and then decides how to present the result. |

### Purposes (each begins with “To”)

- **To** receive HTTP requests and route them to the appropriate application logic.
- **To** coordinate between the model (data) and the view (presentation), acting as the intermediary in the MVC pattern.
- **To** return appropriate HTTP responses (HTML views, JSON data, status codes, redirects).
- **To** leverage model binding to automatically populate action parameters from request data.
- **To** keep application logic organised by grouping related actions into cohesive controller classes.
- **To** remain **thin** by delegating business logic to services, improving testability and maintainability.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class ProductsController : Controller
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllAsync();
        return View(products);
    }

    public async Task<IActionResult> Details(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null) return NotFound();
        return View(product);
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Controller class | Public class ending with `Controller`; resides in `Controllers/`. |
| Base class | `Controller` (with views) or `ControllerBase` (API only). |
| Constructor injection | Services are injected via DI, following the Explicit Dependencies Principle. |
| Action methods | Public methods returning `IActionResult` (or `Task<IActionResult>`). |

#### Syntax Rules

- **Folder:** Controllers reside in the `Controllers` directory at the project root by default.
- **Naming:** Controller class names must end with the `Controller` suffix (e.g., `ProductsController`).
- **Discovery:** Controllers are automatically discovered from the assembly; no registration is needed.
- **Thin controllers:** Business logic belongs in services, not controllers.

#### Constraints and Limitations

- **No direct `DbContext` usage in controllers** – Controllers should call services, not EF Core directly, to avoid tight coupling.
- **Action method complexity** – Overly complex action methods indicate that logic should be extracted to services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: MVC Controller with View Support

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;

public class ProductsController : Controller
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService) => _productService = productService;

    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllAsync();
        return View(products); // Looks for /Views/Products/Index.cshtml
    }
}
```

**Step-by-step explanation:**

1. `ProductsController` inherits from `Controller`, gaining view-rendering helpers (`View()`, `RedirectToAction()`, etc.).
2. `IProductService` is injected via the constructor.
3. `Index()` calls the service and returns `View(products)`.
4. The Razor view engine searches `/Views/Products/Index.cshtml` first.

**Expected Output:** An HTML page listing products is rendered using the `Index.cshtml` view.

**Why this output?** The controller delegates data retrieval to the service and passes the model to the view, following the thin-controller principle.

---

#### Example 2: API Controller with `ControllerBase`

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsApiController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsApiController(IProductService productService) => _productService = productService;

    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> Get()
    {
        var products = await _productService.GetAllAsync();
        return Ok(products);
    }
}
```

**Step-by-step explanation:**

1. `ControllerBase` provides API-oriented helpers (`Ok()`, `NotFound()`, `CreatedAtAction()`) but **no** `View()` method.
2. `[ApiController]` enables opinionated API behaviours (automatic model validation, binding source inference).
3. The action returns `Ok(products)` as JSON.

**Expected Output:** `GET /api/ProductsApi` returns a JSON array of products.

**Why this output?** API controllers return data, not views; `ControllerBase` is the appropriate base class.

### Real-World Cases

- **E-commerce:** `ProductsController` handles listing, details, and search.
- **Admin panel:** `AdminController` inside an area manages administrative tasks.
- **REST API:** `CustomersController` exposes JSON endpoints for mobile clients.

### References

- Create web APIs with ASP.NET Core (ControllerBase) – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#2
- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 2. `Controller`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Controller` is the **base class for MVC controllers that render views**, providing view-rendering helpers and view-related context in addition to everything `ControllerBase` offers. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller` inherits from `ControllerBase` and adds support for **views**, **partial views**, and **view components**. It provides the `View()`, `PartialView()`, and `ViewComponent()` methods, as well as properties like `ViewData`, `ViewBag`, and `TempData`. It is intended for controllers that serve **web pages** (HTML), not web API requests. |
| **Beginner-Friendly Explanation** | `Controller` is the **base class for page-serving controllers**. If your controller needs to return an HTML page (a view), it should inherit from `Controller`. This gives you the `View()` method, which the framework uses to find and render the appropriate `.cshtml` file. Think of it as the “MVC mode” base class. |

### Purposes (each begins with “To”)

- **To** provide view-rendering capabilities (`View()`, `PartialView()`, `ViewComponent()`) for controllers that return HTML pages.
- **To** expose view-related properties (`ViewData`, `ViewBag`, `TempData`) for passing data to views.
- **To** support MVC applications that serve Razor views alongside API endpoints.
- **To** inherit all API-friendly helpers from `ControllerBase` while adding view support.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        ViewData["Title"] = "Home Page";
        return View();
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `Controller` base class | Inherits from `ControllerBase`; adds view support. |
| `View()` | Returns a view matching the action name (`Views/Home/Index.cshtml`). |
| `ViewData` / `ViewBag` | Mechanisms for passing data to views (prefer strongly typed models for complex data). |
| `TempData` | Persists data across redirects. |

#### Syntax Rules

- **Use `Controller` when the action returns a view** (HTML page).
- **Use `Controller` when the controller mixes views and API endpoints** (though separation is preferred).
- **`Controller` inherits from `ControllerBase`** , so all `ControllerBase` helpers are available.

#### Constraints and Limitations

- **Do not use `Controller` for pure API controllers** – Microsoft explicitly states: “Web API controllers should typically derive from `ControllerBase` rather than from `Controller`”.
- **`Controller` adds view-related overhead** (view engines, `ViewData`, etc.) that is unnecessary for APIs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `HomeController` with Views

```csharp
public class HomeController : Controller
{
    public IActionResult Index() => View();
    public IActionResult Privacy() => View();
}
```

**Step-by-step explanation:**

1. `HomeController` inherits from `Controller`.
2. `Index()` returns `View()`, which renders `Views/Home/Index.cshtml`.
3. `Privacy()` returns `View()`, rendering `Views/Home/Privacy.cshtml`.

**Expected Output:** Visiting `/` renders the home page; visiting `/Home/Privacy` renders the privacy page.

**Why this output?** The `View()` method uses the Razor view engine to locate and render the view matching the action name.

---

#### Example 2: Controller with `ViewData`

```csharp
public IActionResult About()
{
    ViewData["Message"] = "Your application description page.";
    return View();
}
```

**Expected Output:** Visiting `/Home/About` renders `Views/Home/About.cshtml` with the message accessible via `ViewData["Message"]`.

**Why this output?** `ViewData` passes data from the controller to the view; the view reads it using `@ViewData["Message"]`.

### Real-World Cases

- **Home page and static pages:** `HomeController` serves the landing page, privacy policy, and about page.
- **Product pages:** `ProductsController` returns product listing and detail views.
- **Form handling:** A controller returns a view for GET requests and processes form submissions for POST requests.

### References

- `Controller` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controller
- Controller vs ControllerBase (Stack Overflow summary) – https://stackoverflow.com/revisions/7fd9e474-ec4b-45e0-9407-0210f9fae3a3/view-source

---

## 3. `ControllerBase`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ControllerBase` is the **base class for web API controllers**, providing HTTP-oriented helpers and properties **without** view support. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase` is the foundational class for controllers that handle HTTP requests and return data (JSON, XML, files, status codes). It provides methods such as `Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`, and `PhysicalFile()`, as well as properties like `User`, `Request`, and `Response`. It does **not** include view-rendering methods (`View()`, `PartialView()`). |
| **Beginner-Friendly Explanation** | `ControllerBase` is the **base class for API controllers**. If your controller returns JSON, status codes, or files—not HTML pages—it should inherit from `ControllerBase`. It gives you all the helpers for building HTTP responses (like `Ok()`, `NotFound()`, `BadRequest()`) without the extra baggage of view rendering. |

### Purposes (each begins with “To”)

- **To** provide HTTP-oriented helpers (`Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`) for building API responses.
- **To** expose request/response context (`HttpContext`, `Request`, `Response`, `User`) to API controllers.
- **To** serve as the lightweight base class for REST APIs, avoiding the overhead of view-related features.
- **To** support model validation and binding helpers (`TryUpdateModelAsync`, `TryValidateModel`).

### Syntax Rules and Structure

#### General Syntax

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetById(id);
        if (product == null) return NotFound();
        return Ok(product);
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ControllerBase` | Base class for APIs; no view support. |
| `Ok()` / `NotFound()` / `BadRequest()` | Helper methods returning `IActionResult` with appropriate status codes. |
| `CreatedAtAction()` | Returns `201 Created` with a `Location` header. |
| `[ApiController]` | Enables opinionated API behaviours (automatic 400 responses, binding source inference). |

#### Syntax Rules

- **Use `ControllerBase` for web API controllers** – Microsoft explicitly recommends this.
- **Do not use `View()`** – `ControllerBase` does not have view-rendering methods.
- **Combine with `[ApiController]`** – enables automatic model state validation and other API-specific behaviours.

#### Constraints and Limitations

- **No view support** – If an action needs to return a Razor view, the controller must inherit from `Controller` instead.
- **`[ApiController]` requires attribute routing** – The attribute enforces attribute routing as a requirement.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic API Controller with `ControllerBase`

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly ICustomerService _customerService;

    public CustomersController(ICustomerService customerService) => _customerService = customerService;

    [HttpGet]
    public ActionResult<IEnumerable<CustomerDto>> GetAll()
    {
        var customers = _customerService.GetAll();
        return Ok(customers);
    }

    [HttpGet("{id:int}")]
    public ActionResult<CustomerDto> GetById(int id)
    {
        var customer = _customerService.GetById(id);
        if (customer == null) return NotFound();
        return Ok(customer);
    }
}
```

**Step-by-step explanation:**

1. `CustomersController` inherits from `ControllerBase`.
2. `[ApiController]` enables automatic model validation and binding source inference.
3. `GetAll()` returns `Ok(customers)` → HTTP 200 with JSON data.
4. `GetById(id)` returns `NotFound()` if the customer doesn't exist, or `Ok(customer)` if found.

**Expected Output:**

| Method | URL | Response |
|--------|-----|----------|
| GET | `/api/Customers` | 200 OK with JSON array |
| GET | `/api/Customers/5` | 200 OK with JSON object (if found) |
| GET | `/api/Customers/999` | 404 Not Found |

**Why this output?** `ControllerBase` provides `Ok()` and `NotFound()` helpers that return the appropriate HTTP status codes with the serialized response.

---

#### Example 2: POST with `CreatedAtAction`

```csharp
[HttpPost]
public ActionResult<CustomerDto> Create(CustomerDto dto)
{
    var created = _customerService.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** `POST /api/Customers` returns `201 Created` with a `Location` header pointing to `/api/Customers/42`.

**Why this output?** `CreatedAtAction` generates the canonical URL for the newly created resource using the named action.

### Real-World Cases

- **REST APIs:** `ProductsController` with `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]` actions.
- **Microservices:** Lightweight API controllers returning JSON.
- **Mobile backends:** `ControllerBase`-derived controllers serving JSON to mobile clients.

### References

- `ControllerBase` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase
- Create web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#2

---

## 4. Controller Creation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller creation is the process of defining a new controller class, choosing the appropriate base class (`Controller` or `ControllerBase`), and organising it within the application’s folder structure. |
| **Technical Definition** | A controller is created by adding a public class ending with `Controller` to the `Controllers` directory, inheriting from either `Controller` or `ControllerBase`, and optionally applying attributes such as `[ApiController]`, `[Route]`, and HTTP verb attributes. The class is discovered automatically by the framework at startup. Constructor injection is used to receive dependencies following the **Explicit Dependencies Principle**. |
| **Beginner-Friendly Explanation** | Creating a controller is **making a new manager for a group of related pages**. You create a new class, give it a name ending in `Controller`, put it in the `Controllers` folder, and choose whether it serves HTML pages (`Controller`) or API data (`ControllerBase`). Then you add methods (actions) for each thing it should handle. |

### Purposes (each begins with “To”)

- **To** define a new entry point for a group of related HTTP requests.
- **To** choose the appropriate base class (`Controller` for views, `ControllerBase` for APIs) based on the controller’s purpose.
- **To** follow naming and folder conventions so the framework can discover the controller automatically.
- **To** inject required services via the constructor, adhering to the Explicit Dependencies Principle.
- **To** apply routing attributes that define the controller’s URL space.

### Syntax Rules and Structure

#### General Syntax (MVC Controller)

```csharp
public class HomeController : Controller
{
    public IActionResult Index() => View();
}
```

#### General Syntax (API Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok();
}
```

#### Component Breakdown

| Decision | Options | When to Use |
|----------|---------|-------------|
| **Base class** | `Controller` | When the controller returns views (HTML pages). |
| | `ControllerBase` | When the controller returns data (JSON, status codes) for APIs. |
| **Folder** | `Controllers/` | Default location for discovery. |
| **Naming** | `*Controller` suffix | Required for convention-based discovery. |
| **Attributes** | `[ApiController]`, `[Route]`, `[HttpGet]` | For API controllers; enables opinionated behaviours and attribute routing. |
| **Dependencies** | Constructor injection | For services used across multiple actions. |

#### Syntax Rules

- **Place controllers in `Controllers/`** for automatic discovery.
- **End class names with `Controller`** (e.g., `ProductsController`).
- **Use `Controller` for view-returning actions; `ControllerBase` for API actions**.
- **Apply `[ApiController]`** to API controllers for automatic model validation and binding source inference.

#### Constraints and Limitations

- **Don’t mix views and APIs in the same controller** – Use separate controllers (`ProductsController` for views, `ProductsApiController` for APIs) to keep concerns separated.
- **Avoid parameter injection for services used by multiple actions** – use constructor injection for shared dependencies.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating an MVC Controller

```csharp
// Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    public IActionResult Index() => View();
    public IActionResult Privacy() => View();
}
```

**Step-by-step explanation:**

1. The class is named `HomeController` and placed in `Controllers/`.
2. It inherits from `Controller`, enabling `View()`.
3. `Index()` and `Privacy()` return views.

**Expected Output:** `/` and `/Home/Privacy` render the corresponding Razor views.

**Why this output?** The framework discovers `HomeController` by convention and maps the default route to its actions.

---

#### Example 2: Creating an API Controller

```csharp
// Controllers/ProductsApiController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsApiController : ControllerBase
{
    private readonly IProductService _service;

    public ProductsApiController(IProductService service) => _service = service;

    [HttpGet]
    public ActionResult<IEnumerable<ProductDto>> Get() =>
        Ok(_service.GetAll());
}
```

**Step-by-step explanation:**

1. The class inherits from `ControllerBase`.
2. `[ApiController]` enables automatic 400 responses and binding inference.
3. `[Route("api/[controller]")]` sets the URL prefix to `api/ProductsApi`.
4. The constructor injects `IProductService`.

**Expected Output:** `GET /api/ProductsApi` returns a JSON array of products.

**Why this output?** `ControllerBase` provides `Ok()`, and `[ApiController]` ensures model validation and binding sources are handled automatically.

### Real-World Cases

- **Feature-based controllers:** `OrdersController` (MVC) and `OrdersApiController` (API) for the same domain but different consumers.
- **Admin controllers:** `Admin/DashboardController` inside an area.
- **Minimal API alternative:** For simple endpoints, minimal APIs can replace controllers entirely, but controllers remain preferred for larger, organised APIs.

### References

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Create web APIs with ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#2
- Explicit Dependencies Principle – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions#explicit-dependencies-principle

---

## 5. Controller Methods (Actions)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller methods (actions) are **public methods on a controller** that handle HTTP requests and return responses (views, JSON, status codes, redirects). |
| **Technical Definition** | Action methods are public, non-static methods on a controller class. They are exposed as HTTP endpoints through routing and can return `IActionResult`, `ActionResult<T>`, or a specific type (POCO). The return type determines how the response is formatted: `IActionResult` allows multiple return types (200, 404, etc.), while a specific type is automatically serialized to JSON in API controllers. |
| **Beginner-Friendly Explanation** | Actions are the **individual operations** a controller can perform. If `ProductsController` is the manager, then `Index()`, `Details()`, `Create()`, and `Delete()` are its specific tasks. Each action is triggered by a URL and decides what to send back—an HTML page, a JSON object, or a status code like “404 Not Found.” |

### Purposes (each begins with “To”)

- **To** handle a specific HTTP request and execute the corresponding application logic.
- **To** return an appropriate response (`IActionResult`, `ActionResult<T>`, or a specific type) based on the outcome.
- **To** use model binding to automatically populate parameters from route values, query strings, and request bodies.
- **To** allow multiple return types (e.g., `Ok()` or `NotFound()`) when the outcome is conditional.
- **To** keep actions focused and small by delegating complex logic to services.

### Syntax Rules and Structure

#### General Syntax (MVC Action)

```csharp
public async Task<IActionResult> Details(int id)
{
    var product = await _service.GetByIdAsync(id);
    if (product == null) return NotFound();
    return View(product);
}
```

#### General Syntax (API Action)

```csharp
[HttpGet("{id:int}")]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return Ok(product);
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Access modifier | `public` (required for action discovery). |
| Return type | `IActionResult` (multiple types), `ActionResult<T>` (typed with multiple types), or a specific type (e.g., `ProductDto`). |
| Parameters | Populated by model binding from route/query/body. |
| Action body | Delegates to services; returns response. |

#### Syntax Rules

- **Actions must be `public`** – non-public methods are not discovered as endpoints.
- **Use `IActionResult` when multiple return types are possible** (e.g., `Ok()` or `NotFound()`).
- **Use `ActionResult<T>`** for typed responses with multiple status codes (API controllers).
- **Use a specific type** when the action always returns the same type and status code.
- **Async actions** should return `Task<IActionResult>` (or `Task<ActionResult<T>>`).

#### Constraints and Limitations

- **Overly complex actions** – If an action contains significant logic, extract it to a service.
- **`void` return types** – Rarely appropriate; use `IActionResult` or a specific type instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Action with `IActionResult` and Multiple Outcomes

```csharp
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null)
        return NotFound();
    return View(product);
}
```

**Step-by-step explanation:**

1. The action returns `IActionResult`, allowing different return types.
2. If `product` is `null`, `NotFound()` returns 404.
3. Otherwise, `View(product)` returns the rendered view.

**Expected Output:**

| Scenario | Response |
|----------|----------|
| Product exists | 200 OK with HTML page |
| Product does not exist | 404 Not Found |

**Why this output?** `IActionResult` allows the action to return different HTTP results based on runtime conditions.

---

#### Example 2: Action with `ActionResult<T>` (API)

```csharp
[HttpGet("{id:int}")]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return Ok(product);
}
```

**Expected Output:**

| Scenario | Response |
|----------|----------|
| Product exists | 200 OK with JSON |
| Product does not exist | 404 Not Found |

**Why this output?** `ActionResult<ProductDto>` provides type safety while allowing multiple status codes, which is recommended for web APIs.

---

#### Example 3: Action with a Specific Return Type

```csharp
[HttpGet]
public IEnumerable<ProductDto> GetAll()
{
    return _service.GetAll();
}
```

**Expected Output:** `GET /api/Products` returns 200 OK with a JSON array.

**Why this output?** A specific return type is appropriate when the action always returns the same type and success status code; MVC automatically serializes the result to JSON.

### Real-World Cases

- **CRUD operations:** `Index()` (list), `Details(id)` (single), `Create(dto)` (POST), `Edit(id, dto)` (PUT), `Delete(id)` (DELETE).
- **Search and filtering:** `Search(string query, int page)` returns filtered results.
- **File downloads:** `Download(int id)` returns a `FileResult`.

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 6. Controller Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller best practices are guidelines for writing controllers that are **thin, testable, maintainable, and aligned with the separation of concerns** principle. |
| **Technical Definition** | Best practices include: thin controllers (delegating business logic to services), the **Explicit Dependencies Principle** (constructor injection), using `ControllerBase` for APIs and `Controller` for views, returning appropriate action result types, avoiding business logic in controllers, using filters for cross-cutting concerns, and keeping actions focused on HTTP concerns. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep controllers clean and easy to work with. The most important rule is: **controllers should be thin**. They should coordinate—call a service, get the result, pass it to the view—not do the actual work themselves. This makes controllers easy to test and keeps business logic reusable. |

### Purposes (each begins with “To”)

- **To** keep controllers focused on HTTP concerns (receiving requests, returning responses) and delegate business logic to services.
- **To** make controllers easy to unit test by following the **Explicit Dependencies Principle** (constructor injection).
- **To** choose the correct base class (`Controller` vs. `ControllerBase`) based on the controller’s purpose.
- **To** use appropriate action result types (`IActionResult`, `ActionResult<T>`, specific types) for clear API contracts.
- **To** avoid duplication by using filters for cross-cutting concerns (validation, logging, error handling).
- **To** maintain consistency and readability across the codebase.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Keep controllers thin** | Business logic belongs in services; thin controllers are easier to test and maintain. | Controller calls `_service.GetAllAsync()` and returns `View(result)`. |
| **Use `ControllerBase` for APIs** | `Controller` adds unnecessary view support for API controllers. | `public class ProductsApiController : ControllerBase` |
| **Use `Controller` for views** | `ControllerBase` lacks `View()` and view-related helpers. | `public class HomeController : Controller` |
| **Follow the Explicit Dependencies Principle** | Declare dependencies via constructor injection; makes testing and reasoning easier. | `public ProductsController(IProductService service)` |
| **Use `IActionResult` for multiple outcomes** | Allows returning `Ok()`, `NotFound()`, `BadRequest()`, etc.. | `if (product == null) return NotFound(); return Ok(product);` |
| **Use `ActionResult<T>` for typed API responses** | Provides type safety while allowing multiple status codes. | `public ActionResult<ProductDto> GetById(int id)` |
| **Avoid business logic in controllers** | Logic in controllers is hard to reuse and test; move it to services. | Move discount calculations, inventory checks, etc., to `IOrderService`. |
| **Use filters for cross-cutting concerns** | Reduces duplication in actions. | `[ServiceFilter(typeof(ValidationFilter))]` |
| **Do not use `try/catch` in controllers** | Exception handling belongs in middleware or filters, not in every action. | Use global exception middleware instead. |

#### Constraints and Limitations

- **Over-engineering** – Applying advanced patterns to simple controllers can add unnecessary complexity.
- **Ignoring the Explicit Dependencies Principle** – Using service locator or `HttpContext.RequestServices` directly makes testing harder.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Thin Controller (Before and After)

```csharp
// ❌ Fat controller (business logic inside)
public IActionResult Details(int id)
{
    var product = _db.Products.Find(id);
    if (product == null) return NotFound();
    product.Price = product.Price * 1.1m; // Business logic!
    return View(product);
}

// ✅ Thin controller (business logic in service)
public IActionResult Details(int id)
{
    var product = _productService.GetWithTax(id);
    if (product == null) return NotFound();
    return View(product);
}
```

**Expected Output:** Both actions return a view, but the thin controller delegates the tax calculation to the service, keeping the controller focused on HTTP concerns.

**Why this output?** Thin controllers are easier to test and maintain; business rules can be reused across MVC views and APIs.

---

#### Example 2: Explicit Dependencies with Constructor Injection

```csharp
// ✅ Constructor injection (Explicit Dependencies Principle)
public class OrdersController : Controller
{
    private readonly IOrderService _orderService;
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(IOrderService orderService, ILogger<OrdersController> logger)
    {
        _orderService = orderService;
        _logger = logger;
    }

    public async Task<IActionResult> Index()
    {
        var orders = await _orderService.GetAllAsync();
        return View(orders);
    }
}
```

**Expected Output:** The controller receives its dependencies through the constructor, making it testable and transparent about what it needs.

**Why this output?** Explicit dependencies make unit testing straightforward (mock `IOrderService`) and avoid hidden dependencies.

---

#### Example 3: Using Filters for Cross-Cutting Concerns

```csharp
[ApiController]
[Route("api/[controller]")]
[ServiceFilter(typeof(ValidationFilter))]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public ActionResult<ProductDto> Create(ProductDto dto)
    {
        // Validation filter runs before this action
        return Ok(dto);
    }
}
```

**Expected Output:** Invalid requests are short-circuited by the validation filter with a 400 response; valid requests reach the action.

**Why this output?** Filters centralise cross-cutting concerns, reducing duplication and keeping actions focused.

### Real-World Cases

- **E-commerce:** Thin `CheckoutController` delegates to `IOrderService`; validation filter handles model state.
- **API:** `ProductsApiController : ControllerBase` with `ActionResult<T>` returns; no view-related code.
- **MVC:** `HomeController : Controller` returns views; no business logic in actions.

### References

- Handle requests with controllers in ASP.NET Core MVC (Explicit Dependencies Principle) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions#explicit-dependencies-principle
- Do you use Thin Controllers, Fat Models, and Dumb Views? – https://github.com/sswconsulting/ssw.rules.content/blob/6bef495e78b5100b8ba830e146280af37109b2af/public/uploads/rules/do-you-use-thin-controllers-fat-models-and-dumb-views/rule.mdx
- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Create web APIs with ASP.NET Core (ControllerBase recommendation) – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#2

---

## Summary Table: Controller Fundamentals at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Controllers** | Handle HTTP requests and coordinate responses | Thin, focused classes ending in `Controller` | Every request-handling class |
| **`Controller`** | Base class with view support | Inherits from `ControllerBase`; adds `View()`, `ViewData` | MVC pages that return HTML |
| **`ControllerBase`** | Base class without view support | Provides `Ok()`, `NotFound()`, `BadRequest()` | Web API controllers returning JSON |
| **Controller Creation** | Define a new controller | Place in `Controllers/`, choose base class, inject services | Every new functional area |
| **Controller Methods (Actions)** | Handle specific HTTP requests | Public methods returning `IActionResult` or typed results | Every endpoint |
| **Best Practices** | Maintainability and testability | Thin controllers, constructor injection, correct base class | All production controllers |

---

## References (Full List)

- Create web APIs with ASP.NET Core (ControllerBase recommendation) – https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0#2
- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- `Controller` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controller
- `ControllerBase` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase
- Controller vs ControllerBase (Stack Overflow summary) – https://stackoverflow.com/revisions/7fd9e474-ec4b-45e0-9407-0210f9fae3a3/view-source
- Do you use Thin Controllers, Fat Models, and Dumb Views? (SSW) – https://github.com/sswconsulting/ssw.rules.content/blob/6bef495e78b5100b8ba830e146280af37109b2af/public/uploads/rules/do-you-use-thin-controllers-fat-models-and-dumb-views/rule.mdx
- ASP.NET Core web API controllers (Microsoft Learn training) – https://learn.microsoft.com/pt-pt/training/modules/build-web-api-aspnet-core/4-aspnet-controllers
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing