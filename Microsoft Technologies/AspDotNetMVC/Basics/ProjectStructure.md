# Project Structure in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10/11) and the **default folder conventions** established by the official project templates. ASP.NET Core templates use the **generic host** (`Program.cs` with minimal hosting in .NET 6+). Folder structures can be customised extensively; the conventions described here are the **defaults** that the framework recognises out of the box. Areas are covered in detail in a separate section. Legacy ASP.NET MVC 5 structures (Global.asax, App_Start, Content, Scripts) are noted only to clarify migration differences.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Project structure in ASP.NET Core MVC is the **conventional folder organisation** that separates controllers, views, models, and supporting infrastructure into well-defined directories, enabling the framework to discover and wire up application components automatically. |
| **Technical Definition** | ASP.NET Core MVC uses **convention-over-configuration** to locate application parts. Controllers are discovered from assemblies; views are discovered by the **RazorViewEngine** using a well-defined search path (`/Views/{Controller}/{Action}.cshtml`, `/Views/Shared/{Action}.cshtml`); models and view models are plain CLR types with no framework-imposed location. Supporting folders (Services, Data, Middleware, Filters, Extensions, Configuration) follow community and Clean Architecture conventions rather than framework-mandated rules. The **`wwwroot`** folder is the web root for static files, served by `UseStaticFiles`. **Areas** partition the application into functional groups with their own Controllers/Views structure. |
| **Beginner-Friendly Explanation** | Project structure is **how you organise your files** so that both the framework and your team can find things easily. ASP.NET Core expects controllers in a `Controllers` folder, views in a `Views` folder grouped by controller name, and static files (CSS, JS, images) in `wwwroot`. Other folders like `Services`, `Data`, and `Models` are organisational choices that keep your code clean and maintainable as the app grows. |

### Key Characteristics

- **Convention-based discovery** – The framework automatically finds controllers and views based on folder and naming conventions; explicit registration is rarely required for core MVC components.
- **Separation of concerns** – Each folder has a clear responsibility, reducing coupling and improving maintainability.
- **Extensible** – Conventions can be overridden (e.g., custom view location formats, custom `IControllerModelConvention`).
- **Feature-folder alternative** – Large applications can organise by feature (Areas, vertical slices) instead of by file type.
- **Layered architecture support** – The default structure can be extended into multi-project solutions (Web, Services, Data, Core) following Clean Architecture principles.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** project template (`dotnet new mvc`).
- Basic understanding of **controllers**, **views**, and **Razor** syntax.
- Familiarity with **C# namespaces** and **dependency injection**.

### Related Programming Areas

- **Routing** – Controllers and actions are discovered and mapped via routing.
- **Razor View Engine** – View discovery and layout resolution depend on folder structure.
- **Dependency Injection** – Services and repositories are registered in `Program.cs` and injected into controllers.
- **Entity Framework Core** – Data access is typically isolated in a `Data` folder.
- **Middleware Pipeline** – Custom middleware lives in a `Middleware` folder and is registered in `Program.cs`.
- **Filters** – Action filters, exception filters, and result filters live in a `Filters` folder.
- **Configuration** – `appsettings.json` and options classes live in the project root or a `Configuration` folder.

### Core Concepts / Features

1. **Controllers** – Handle HTTP requests and orchestrate application logic.
2. **Views** – Razor templates that render HTML, grouped by controller name.
3. **Models** – Domain and data models representing the application’s data.
4. **ViewModels** – Types shaped specifically for views, preventing over-posting and exposing only necessary data.
5. **Services** – Business logic and application services, injected into controllers.
6. **Repositories** – Data-access abstractions (optional; EF Core’s `DbContext` often suffices).
7. **Data** – EF Core `DbContext`, migrations, and entity configurations.
8. **Middleware** – Custom HTTP pipeline components.
9. **Filters** – Cross-cutting concerns (validation, logging, exception handling) applied declaratively.
10. **Extensions** – Extension methods for `IServiceCollection` and `IApplicationBuilder`, keeping `Program.cs` clean.
11. **Configuration** – Options classes and configuration binding.
12. **wwwroot** – Static assets (CSS, JS, images, libraries) served to clients.
13. **Areas** – Functional partitions of the application with their own Controllers/Views.
14. **Project Structure Best Practices** – Guidelines for scalable, maintainable organisation.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Controllers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controllers are classes that handle incoming HTTP requests, coordinate application logic, and return responses (views, JSON, redirects, etc.). |
| **Technical Definition** | A controller is a class that typically inherits from `Controller` (MVC with views) or `ControllerBase` (APIs). It contains **action methods**—public methods that are mapped to routes. Controllers are discovered by the framework from assemblies; no explicit registration is required. Action methods use **model binding** to populate parameters from route values, query strings, and request bodies, and return **`IActionResult`** implementations. |
| **Beginner-Friendly Explanation** | Controllers are the **traffic directors** of your application. When a request comes in for `/Products/Details/5`, the `ProductsController` handles it. The `Details` action method runs, fetches the product data, and decides what to send back—usually a view (HTML page) or JSON data. |

### Purposes (each begins with “To”)

- **To** receive and process incoming HTTP requests and route them to the appropriate business logic.
- **To** coordinate between the model (data) and the view (presentation), acting as the intermediary in the MVC pattern.
- **To** return appropriate HTTP responses (views, JSON, status codes, redirects).
- **To** leverage model binding to automatically populate action parameters from request data.
- **To** keep controllers **thin** by delegating business logic to services, improving testability and maintainability.

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
| `Controller` base class | Provides view-rendering helpers (`View()`, `RedirectToAction()`, `Json()`). |
| `ControllerBase` | Lightweight base for APIs; no view support. |
| Constructor injection | Services are injected via DI, keeping the controller thin. |
| Action methods | Public methods returning `IActionResult` (or `Task<IActionResult>`). |
| `View()` | Returns a view, typically located at `/Views/{Controller}/{Action}.cshtml`. |

#### Syntax Rules

- **Folder:** Controllers reside in the `Controllers` folder at the project root by default.
- **Naming:** Controller classes must end with the `Controller` suffix (e.g., `ProductsController`).
- **Discovery:** Controllers are automatically discovered from the assembly; no registration is needed.
- **Thin controllers:** Business logic belongs in services, not controllers.

#### Constraints and Limitations

- **No direct `DbContext` usage in controllers** – Controllers should call services, not EF Core directly, to avoid tight coupling and improve testability.
- **Action method complexity** – Overly complex action methods indicate that logic should be extracted to services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Controller with Service Injection

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

1. `ProductsController` inherits from `Controller`.
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

1. `ControllerBase` provides API-oriented helpers (`Ok()`, `NotFound()`, `CreatedAtAction()`).
2. `[ApiController]` enables automatic model validation and binding source inference.
3. The action returns `Ok(products)` as JSON.

**Expected Output:** `GET /api/ProductsApi` returns a JSON array of products.

**Why this output?** API controllers return data, not views; `ControllerBase` is the appropriate base class.

### Real-World Cases

- **E-commerce:** `ProductsController` handles listing, details, and search.
- **Admin panel:** `AdminController` inside an area manages administrative tasks.
- **REST API:** `CustomersController` exposes JSON endpoints for mobile clients.

### References

- Controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- ASP.NET Core Best Practices and Coding Standards (thin controllers) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 2. Views

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Views are Razor template files (`.cshtml`) that render HTML markup, combining static HTML with server-side C# code to produce the user interface. |
| **Technical Definition** | Views are `.cshtml` files stored in the `Views` folder. The Razor view engine discovers views using a well-defined search order: (1) `/Views/{ControllerName}/{ActionName}.cshtml`, (2) `/Views/Shared/{ActionName}.cshtml`. Layouts (`_Layout.cshtml`), partial views, and `_ViewStart.cshtml` / `_ViewImports.cshtml` provide shared UI structure, reusable fragments, and shared directives respectively. Views receive a **strongly typed model** from the controller and use Razor syntax (`@Model`, `@foreach`, Tag Helpers) to render it. |
| **Beginner-Friendly Explanation** | Views are the **HTML templates** of your application. They’re what the user actually sees in the browser. Controllers pass data (the model) to views, and the views use that data to generate the final HTML page. Views live in a `Views` folder, organised by controller name, so the framework knows which view belongs to which action. |

### Purposes (each begins with “To”)

- **To** separate the presentation layer (HTML) from business logic, following the **separation of concerns** principle.
- **To** provide a place for Razor syntax that dynamically renders data passed from the controller.
- **To** enable reusable UI components through layouts, partial views, and view components.
- **To** support strongly typed models, providing IntelliSense and compile-time checking in views.
- **To** allow shared views (`/Views/Shared`) to be reused across multiple controllers.

### Syntax Rules and Structure

#### View Discovery Order

```
1. /Views/{ControllerName}/{ActionName}.cshtml
2. /Views/Shared/{ActionName}.cshtml
```

#### Typical Folder Structure

```
Views/
├── _ViewStart.cshtml        // Sets default layout
├── _ViewImports.cshtml      // Shared directives and Tag Helpers
├── Home/
│   ├── Index.cshtml
│   └── About.cshtml
├── Products/
│   ├── Index.cshtml
│   └── Details.cshtml
└── Shared/
    ├── _Layout.cshtml
    ├── _ValidationScriptsPartial.cshtml
    └── Error.cshtml
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `Views/{Controller}/` | Views specific to a controller. |
| `Views/Shared/` | Views shared across controllers (layouts, partials, error pages). |
| `_ViewStart.cshtml` | Executes before each view; sets the default `Layout`. |
| `_ViewImports.cshtml` | Contains `@using` directives and `@addTagHelper` directives. |
| `_Layout.cshtml` | Master page with `@RenderBody()` for page content. |

#### Syntax Rules

- **Views must be `.cshtml` files** and reside in the `Views` folder or a subfolder.
- **View names should match action names** for automatic discovery.
- **`_ViewImports.cshtml` is not inherited** by area views; each area needs its own.
- **`_ViewStart.cshtml`** in a folder applies to all views in that folder and its subfolders.

#### Constraints and Limitations

- **Views are not strongly typed by default** – use `@model` to specify the model type.
- **Over-reliance on `ViewBag`/`ViewData`** – prefer strongly typed models for compile-time safety.
- **View discovery is case-insensitive** on most file systems but sensitive on Linux; use consistent casing.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strongly Typed View

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<Product>

<h2>Products</h2>
<ul>
@foreach (var product in Model)
{
    <li>@product.Name - @product.Price.ToString("C")</li>
}
</ul>
```

**Step-by-step explanation:**

1. `@model IEnumerable<Product>` declares the model type.
2. `@foreach` iterates over the model.
3. Razor syntax (`@product.Name`) renders the data.

**Expected Output:** An HTML list of product names and prices.

**Why this output?** The controller returns `View(products)`, and the view renders the strongly typed model.

---

#### Example 2: Layout and `_ViewStart`

```cshtml
@* Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html>
<head><title>@ViewData["Title"]</title></head>
<body>
    <header>My App</header>
    @RenderBody()
    <footer>&copy; 2025</footer>
</body>
</html>
```

**Step-by-step explanation:**

1. `_ViewStart.cshtml` sets the default layout to `_Layout`.
2. `_Layout.cshtml` provides the shared HTML skeleton.
3. `@RenderBody()` is replaced with the content of the specific view.

**Expected Output:** Every page uses the shared header and footer.

**Why this output?** `_ViewStart` executes before each view, applying the layout automatically.

### Real-World Cases

- **E-commerce:** `Views/Products/Index.cshtml` displays a product grid; `Details.cshtml` shows a single product.
- **Admin dashboard:** Shared `_Layout.cshtml` provides navigation and branding.
- **Partial views:** `_ProductCard.cshtml` is reused across multiple pages.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 3. Models

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Models are classes that represent the application’s **data and business rules**, independent of the UI or data-access technology. |
| **Technical Definition** | In MVC, a model encapsulates the data and behaviour of the application. Models are typically **POCOs** (Plain Old CLR Objects) that may include validation attributes (`[Required]`, `[StringLength]`) and are used by controllers to pass data to views. In layered architectures, **domain models** (entities) live in a `Core` or `Entities` folder, while **data models** (EF Core entities) may live in a `Data` folder. Models are distinct from **ViewModels**, which are shaped for specific views. |
| **Beginner-Friendly Explanation** | Models are the **data blueprint** of your app. A `Product` model has properties like `Id`, `Name`, and `Price`. Models represent what your app works with—they’re the “things” in your system. They don’t know about HTML or databases; they just hold data and sometimes validation rules. |

### Purposes (each begins with “To”)

- **To** represent the application’s data entities (e.g., `Product`, `Order`, `Customer`).
- **To** encapsulate business rules and validation logic (via data annotations).
- **To** serve as the data contract between controllers and views (when ViewModels are not used).
- **To** provide a technology-agnostic representation of domain concepts, enabling separation from EF Core or other data-access concerns.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class Product
{
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Range(0.01, 10000)]
    public decimal Price { get; set; }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Properties | Data fields with getters/setters. |
| Data annotations | Validation rules (`[Required]`, `[Range]`, `[StringLength]`). |
| No framework inheritance | Models typically do not inherit from framework base classes. |

#### Syntax Rules

- **Folder:** Models conventionally reside in the `Models` folder at the project root.
- **Naming:** PascalCase for class and property names.
- **Validation:** Use data annotations for declarative validation.
- **Separation:** Domain models should not contain EF Core attributes if they need to remain technology-agnostic.

#### Constraints and Limitations

- **Over-posting risk** – Using domain models directly in views can expose sensitive properties; use ViewModels for forms.
- **Anemic models** – Models with only properties and no behaviour are common but may indicate logic leaking into services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Domain Model with Validation

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000, ErrorMessage = "Price must be between 0.01 and 10,000.")]
    public decimal Price { get; set; }
}
```

**Step-by-step explanation:**

1. `Product` has three properties.
2. Data annotations enforce validation rules.
3. The model is used by controllers and views.

**Expected Output:** When model validation runs, invalid products produce validation errors.

**Why this output?** Data annotations are evaluated by MVC’s model validation system.

---

#### Example 2: Plain Entity (Technology-Agnostic)

```csharp
// Entities/Product.cs
public class Product
{
    public Guid Id { get; private set; }
    public string Name { get; private set; } = string.Empty;
    public decimal Price { get; private set; }

    private Product() { } // For EF Core

    public static Product Create(string name, decimal price)
    {
        return new Product { Id = Guid.NewGuid(), Name = name, Price = price };
    }
}
```

**Step-by-step explanation:**

1. `Product` is a plain domain entity with no EF Core attributes.
2. The private constructor is for EF Core materialisation.
3. A static factory method enforces creation rules.

**Expected Output:** A domain entity free of database concerns, suitable for Clean Architecture.

**Why this output?** Keeping entities plain allows them to work across multiple databases and reduces coupling to EF Core.

### Real-World Cases

- **E-commerce:** `Product`, `Order`, `Customer` domain models.
- **Blog:** `Post`, `Comment`, `Author` models.
- **SaaS:** `Subscription`, `Invoice`, `Tenant` models.

### References

- Models in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/
- ASP.NET Core Best Practices and Coding Standards (Models folder) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 4. ViewModels

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModels are classes **shaped specifically for a view**, containing exactly the data the view needs—nothing more, nothing less. |
| **Technical Definition** | A ViewModel is a simple class (often with validation attributes) that aggregates data from one or more models and is passed from the controller to the view. ViewModels prevent **over-posting** (clients binding to properties they shouldn’t), reduce the attack surface, and allow views to display data that doesn’t map one-to-one to a domain model. They reside in a `ViewModels` folder (often inside `Models` or alongside it) and are **not** domain entities. |
| **Beginner-Friendly Explanation** | ViewModels are **custom-made data packages for views**. If a page needs a product’s name and its category name, but those come from different database tables, you create a `ProductDetailsViewModel` with just those two properties. The view gets exactly what it needs, and nothing extra—like a waiter bringing exactly what you ordered, not the whole kitchen. |

### Purposes (each begins with “To”)

- **To** prevent **over-posting** attacks by exposing only the properties a form should accept.
- **To** aggregate data from multiple sources into a single object tailored to a specific view.
- **To** decouple views from domain models, allowing the domain model to evolve without breaking the UI.
- **To** provide a place for view-specific validation rules and display formatting.
- **To** keep controllers and views clean by avoiding `ViewBag`/`ViewData` for structured data.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public bool IsInStock { get; set; }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Properties | Only the data the view needs. |
| Validation attributes | `[Required]`, `[Range]`, etc., for form validation. |
| Display attributes | `[Display(Name = "...")]` for labels. |
| No behaviour | ViewModels are typically data-only. |

#### Syntax Rules

- **Folder:** ViewModels conventionally reside in a `ViewModels` folder inside the web project.
- **Naming:** End with `ViewModel` or `Dto` to distinguish from domain models.
- **No domain logic** – ViewModels should not contain business logic.

#### Constraints and Limitations

- **Duplication** – ViewModels may duplicate properties from domain models; mapping (via AutoMapper or manual projection) is required.
- **Maintenance overhead** – Each view may need its own ViewModel; this is intentional but adds classes.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: ViewModel for a Product Detail Page

```csharp
// ViewModels/ProductDetailsViewModel.cs
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
}
```

```csharp
// Controller action
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();

    var viewModel = new ProductDetailsViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryName = product.Category.Name
    };

    return View(viewModel);
}
```

**Step-by-step explanation:**

1. The ViewModel contains exactly the fields the view displays.
2. The controller maps the domain model to the ViewModel.
3. The view uses `@model ProductDetailsViewModel`.

**Expected Output:** The view displays the product name, price, and category name without exposing the entire domain model.

**Why this output?** ViewModels shape data for the view, preventing over-posting and keeping the UI contract explicit.

---

#### Example 2: ViewModel with Validation for a Form

```csharp
public class CreateProductViewModel
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000)]
    public decimal Price { get; set; }
}
```

**Expected Output:** When the form is submitted with invalid data, validation messages appear, and the action is not executed.

**Why this output?** Validation attributes on the ViewModel are evaluated by MVC’s model validation before the action runs.

### Real-World Cases

- **E-commerce:** `CheckoutViewModel` aggregates cart items, shipping address, and payment method.
- **Admin dashboard:** `DashboardViewModel` combines counts, recent orders, and alerts.
- **Forms:** `RegisterViewModel` contains only email, password, and confirm password—not the full `User` entity.

### References

- ASP.NET Core MVC project structure (ViewModels) – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards (over-posting) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 5. Services

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Services are classes that encapsulate **business logic and application operations**, keeping controllers thin and testable. |
| **Technical Definition** | Services implement interfaces (e.g., `IProductService`) and are registered in the DI container. They orchestrate data access (via repositories or `DbContext`), apply business rules, and return results to controllers. Services reside in a `Services` folder or a separate `ProjectName.Services` project in layered architectures. |
| **Beginner-Friendly Explanation** | Services are the **“brains”** of your app. When a controller needs to create an order, it doesn’t do the work itself—it calls `IOrderService.CreateOrderAsync()`. The service checks inventory, calculates totals, applies discounts, saves to the database, and sends a confirmation email. The controller just handles the HTTP part. |

### Purposes (each begins with “To”)

- **To** centralise business logic in a single, testable layer, avoiding duplication across controllers.
- **To** keep controllers thin and focused on HTTP concerns.
- **To** provide a clear contract (via interfaces) for dependency injection and mocking in tests.
- **To** coordinate multiple data sources or external services (e.g., payment gateways, email senders).
- **To** enable reuse of business logic across different entry points (MVC views, APIs, background jobs).

### Syntax Rules and Structure

#### General Syntax

```csharp
public interface IProductService
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task<Product> CreateAsync(CreateProductDto dto);
}

public class ProductService : IProductService
{
    private readonly AppDbContext _db;

    public ProductService(AppDbContext db) => _db = db;

    public async Task<IEnumerable<Product>> GetAllAsync() =>
        await _db.Products.ToListAsync();

    public async Task<Product?> GetByIdAsync(int id) =>
        await _db.Products.FindAsync(id);

    public async Task<Product> CreateAsync(CreateProductDto dto)
    {
        var product = Product.Create(dto.Name, dto.Price);
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
        return product;
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Interface (`IProductService`) | Contract for DI and testing. |
| Implementation (`ProductService`) | Business logic and data orchestration. |
| Constructor injection | Dependencies (`AppDbContext`, `ILogger`) injected. |

#### Syntax Rules

- **Folder:** Services reside in a `Services` folder or a separate `ProjectName.Services` project.
- **Interface-first** – Define an interface for each service to enable DI and mocking.
- **Registration:** Register services in `Program.cs` (`builder.Services.AddScoped<IProductService, ProductService>()`).

#### Constraints and Limitations

- **Not a dumping ground** – Services should have a cohesive responsibility; avoid god-services.
- **Repository vs. direct `DbContext`** – Some teams use repositories; others use `DbContext` directly in services. Both are valid; be consistent.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Service with Interface

```csharp
// Services/IProductService.cs
public interface IProductService
{
    Task<IEnumerable<ProductDto>> GetAllAsync();
}

// Services/ProductService.cs
public class ProductService : IProductService
{
    private readonly AppDbContext _db;

    public ProductService(AppDbContext db) => _db = db;

    public async Task<IEnumerable<ProductDto>> GetAllAsync()
    {
        return await _db.Products
            .Select(p => new ProductDto { Id = p.Id, Name = p.Name, Price = p.Price })
            .ToListAsync();
    }
}

// Program.cs
builder.Services.AddScoped<IProductService, ProductService>();
```

**Step-by-step explanation:**

1. The interface defines the contract.
2. The implementation uses `AppDbContext` to query products and project to DTOs.
3. The service is registered with a **scoped** lifetime.

**Expected Output:** Controllers can inject `IProductService` and call `GetAllAsync()`.

**Why this output?** DI resolves the interface to the implementation, and the service encapsulates data access and projection.

---

#### Example 2: Service with Business Logic

```csharp
public async Task<OrderResult> PlaceOrderAsync(int userId, CheckoutRequest request)
{
    var cart = await _cartService.GetCartAsync(userId);
    if (!cart.Items.Any()) return OrderResult.Fail("Cart is empty.");

    var total = _pricingService.CalculateTotal(cart);
    if (!await _paymentService.ProcessPaymentAsync(userId, total))
        return OrderResult.Fail("Payment failed.");

    var order = Order.Create(userId, cart.Items, total);
    _db.Orders.Add(order);
    await _db.SaveChangesAsync();

    await _emailService.SendOrderConfirmationAsync(userId, order.Id);
    return OrderResult.Ok(order.Id);
}
```

**Step-by-step explanation:**

1. The service orchestrates cart retrieval, pricing, payment, order creation, and email.
2. Business rules (empty cart, payment failure) are centralised.
3. The controller only translates HTTP to a service call.

**Expected Output:** `OrderResult` indicates success or failure; the controller returns an appropriate HTTP response.

**Why this output?** Business logic lives in the service, making it reusable across MVC views and APIs, and testable in isolation.

### Real-World Cases

- **E-commerce:** `OrderService` handles checkout, pricing, and payment.
- **SaaS:** `SubscriptionService` manages plans, renewals, and cancellations.
- **Blog:** `PostService` handles publishing, editing, and commenting rules.

### References

- Modular ASP.NET Core MVC Template (Services layer) – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards (Services) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 6. Repositories

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Repositories are **data-access abstractions** that encapsulate query logic and provide a collection-like interface for retrieving and persisting domain entities. |
| **Technical Definition** | A repository implements an interface (e.g., `IProductRepository`) that exposes methods like `GetByIdAsync`, `ListAsync`, `AddAsync`, and `Update`. It hides the underlying data store (EF Core `DbContext`, Dapper, MongoDB) from the service layer. Repositories typically reside in a `Repositories` folder within the `Data` or `Infrastructure` project. **Note:** Many teams skip repositories and use `DbContext` directly in services, since `DbSet<T>` already acts as a repository and `DbContext` as a unit of work. |
| **Beginner-Friendly Explanation** | Repositories are **specialised librarians** for your data. Instead of writing database queries all over your app, you ask the repository: “Give me product 5” or “Save this order.” The repository knows how to talk to the database, so the rest of your code doesn’t have to. |

### Purposes (each begins with “To”)

- **To** centralise data-access logic and avoid scattering queries throughout services.
- **To** provide a testable abstraction (via interfaces) that can be mocked in unit tests.
- **To** decouple business logic from the specific data-access technology (EF Core, Dapper, etc.).
- **To** enforce consistent query patterns and business rules at the data layer.
- **To** support unit-of-work patterns when multiple repositories must share a transaction.

### Syntax Rules and Structure

#### General Syntax

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> ListAsync();
    Task AddAsync(Product product);
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;

    public ProductRepository(AppDbContext db) => _db = db;

    public async Task<Product?> GetByIdAsync(int id) =>
        await _db.Products.FindAsync(id);

    public async Task<IEnumerable<Product>> ListAsync() =>
        await _db.Products.ToListAsync();

    public async Task AddAsync(Product product)
    {
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Interface (`IProductRepository`) | Contract for DI and testing. |
| Implementation | Encapsulates EF Core queries. |
| `DbContext` injection | The repository receives the `DbContext` via DI. |

#### Syntax Rules

- **Folder:** Repositories reside in `Data/Repositories` or `Infrastructure/Repositories`.
- **Registration:** Register repositories in `Program.cs` with `AddScoped`.
- **Unit of work** – The `DbContext` itself is a unit of work; repositories should not call `SaveChanges` independently if a service orchestrates multiple operations.

#### Constraints and Limitations

- **Leaky abstraction** – Overly generic repositories (`IRepository<T>`) often hide useful EF Core features (projections, `Include`, `ExecuteUpdateAsync`). Many teams prefer direct `DbContext` usage in services.
- **Added complexity** – Repositories add a layer of indirection that may not be justified for simple CRUD apps.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Repository with Interface

```csharp
// Data/Repositories/IProductRepository.cs
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> ListAsync();
    Task AddAsync(Product product);
}

// Data/Repositories/ProductRepository.cs
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;

    public ProductRepository(AppDbContext db) => _db = db;

    public async Task<Product?> GetByIdAsync(int id) =>
        await _db.Products.FindAsync(id);

    public async Task<IEnumerable<Product>> ListAsync() =>
        await _db.Products.ToListAsync();

    public async Task AddAsync(Product product)
    {
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
    }
}

// Program.cs
builder.Services.AddScoped<IProductRepository, ProductRepository>();
```

**Step-by-step explanation:**

1. The interface abstracts data access.
2. The implementation uses `DbContext`.
3. The repository is registered in DI.

**Expected Output:** Services can inject `IProductRepository` and perform data operations without directly depending on EF Core.

**Why this output?** The repository encapsulates EF Core, making the service layer testable with a mocked repository.

---

#### Example 2: Using `DbContext` Directly in a Service (Alternative)

```csharp
public class ProductService : IProductService
{
    private readonly AppDbContext _db;

    public ProductService(AppDbContext db) => _db = db;

    public async Task<IEnumerable<ProductDto>> GetAllAsync() =>
        await _db.Products
            .Select(p => new ProductDto { Id = p.Id, Name = p.Name, Price = p.Price })
            .ToListAsync();
}
```

**Expected Output:** The same functionality, with less indirection.

**Why this output?** Many teams find that `DbContext` already provides repository-like functionality, and repositories add unnecessary complexity for simple scenarios.

### Real-World Cases

- **Large enterprise:** Repositories used to abstract complex data access and support multiple data sources.
- **Simple CRUD apps:** `DbContext` used directly in services, skipping repositories for simplicity.
- **Testing:** Repositories mocked to isolate business logic from the database.

### References

- Clean Architecture project structure (Repositories) – https://github.com/asiaostrich/universal-dev-standards/blob/a8112c5f730d3a5236a8f46991168a6033b26a8c/options/project-structure/dotnet.md
- ASP.NET Core MVC project structure (practical layered architecture) – https://dev.to/rgk/aspnet-core-mvc-project-structure-a-practical-layered-architecture-43o8

---

## 7. Data

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The `Data` folder contains **data-access infrastructure**—typically the EF Core `DbContext`, entity configurations, migrations, and optionally repositories. |
| **Technical Definition** | In layered architectures, `Data` (or `Infrastructure`) is a separate project containing `AppDbContext`, `IEntityTypeConfiguration<T>` classes, migrations, and seed data. The `DbContext` manages database connections, entity tracking, and query execution. Entity configurations use the Fluent API to define table mappings, relationships, and constraints, keeping entities free of database attributes. |
| **Beginner-Friendly Explanation** | The `Data` folder is the **database workshop**. It contains the `DbContext` (the gateway to the database), the configuration that tells EF Core how to map your classes to tables, and the migrations that create or update the database schema. Keeping this separate from your business logic keeps your app clean. |

### Purposes (each begins with “To”)

- **To** isolate EF Core `DbContext` and database configuration from business logic.
- **To** centralise entity configurations using `IEntityTypeConfiguration<T>` for clean, maintainable mappings.
- **To** manage database schema changes through migrations.
- **To** provide seed data for development and testing.
- **To** keep domain entities free of database-specific attributes, preserving technology-agnosticism.

### Syntax Rules and Structure

#### General Syntax

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

```csharp
// Data/Configurations/ProductConfiguration.cs
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.Property(p => p.Price).HasPrecision(18, 2);
        builder.HasIndex(p => p.Name).IsUnique();
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `AppDbContext` | The EF Core database context; manages entity sets. |
| `DbSet<T>` | Represents a table; queryable collection. |
| `IEntityTypeConfiguration<T>` | Fluent API configuration for entity mapping. |
| `Migrations` | Schema versioning and evolution. |

#### Syntax Rules

- **Folder:** `Data` resides in the web project or a separate `ProjectName.Data` project.
- **Configuration:** Use `IEntityTypeConfiguration<T>` instead of attributes on entities for cleaner separation.
- **Registration:** `builder.Services.AddDbContext<AppDbContext>(options => ...)` in `Program.cs`.

#### Constraints and Limitations

- **Migrations** – Must be managed carefully in team environments; use a consistent migration strategy.
- **Connection strings** – Store in `appsettings.json` or environment variables, never in source control.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `DbContext` with Entity Configuration

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}

// Data/Configurations/ProductConfiguration.cs
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.Property(p => p.Price).HasPrecision(18, 2);
        builder.HasIndex(p => p.Name).IsUnique();
    }
}

// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

**Step-by-step explanation:**

1. `AppDbContext` exposes `DbSet<Product>`.
2. `OnModelCreating` applies all `IEntityTypeConfiguration` classes in the assembly.
3. `ProductConfiguration` defines precision and a unique index.
4. The context is registered with SQL Server.

**Expected Output:** EF Core uses the configuration to create the correct schema during migrations.

**Why this output?** Separating configuration from entities keeps entities clean and configuration focused.

---

#### Example 2: Seed Data

```csharp
// Data/Seeds/ProductSeeder.cs
public static class ProductSeeder
{
    public static void Seed(AppDbContext db)
    {
        if (!db.Products.Any())
        {
            db.Products.AddRange(
                Product.Create("Laptop", 999.99m),
                Product.Create("Mouse", 19.99m)
            );
            db.SaveChanges();
        }
    }
}
```

**Expected Output:** The database is populated with initial product data on first run.

**Why this output?** Seeding ensures a consistent starting state for development and testing.

### Real-World Cases

- **E-commerce:** `AppDbContext` with `Products`, `Orders`, `Customers` configurations.
- **Multi-tenant SaaS:** `AppDbContext` with tenant-specific query filters.
- **Blog:** `AppDbContext` with `Posts`, `Comments`, `Authors`.

### References

- ASP.NET Core Best Practices and Coding Standards (Data folder) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- .NET Project Structure (Clean Architecture, Data layer) – https://github.com/asiaostrich/universal-dev-standards/blob/a8112c5f730d3a5236a8f46991168a6033b26a8c/options/project-structure/dotnet.md

---

## 8. Middleware

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware are components that form the **HTTP request pipeline**, processing requests and responses as they flow through the application. |
| **Technical Definition** | Middleware are classes with an `InvokeAsync(HttpContext)` method (or constructor-injected `RequestDelegate`). They are registered in `Program.cs` via `app.Use...()` or `app.UseMiddleware<T>()`. Middleware executes in **registration order**; each component can short-circuit the pipeline or pass control to the next. Common built-in middleware includes exception handling, HTTPS redirection, static files, routing, authentication, and authorization. **Custom middleware** (e.g., request logging, correlation ID, exception handling) resides in a `Middleware` folder. |
| **Beginner-Friendly Explanation** | Middleware are like **security checkpoints** at an airport. Every request passes through them in order: first security (authentication), then passport control (authorization), then the gate (routing to your controller). Each checkpoint can inspect the request, modify it, or stop it entirely. Custom middleware lets you add your own checkpoints—like logging every request for debugging. |

### Purposes (each begins with “To”)

- **To** process HTTP requests and responses globally, before or after the endpoint executes.
- **To** implement cross-cutting concerns (logging, error handling, authentication, CORS) without duplicating code in controllers.
- **To** short-circuit the pipeline for invalid or unauthorized requests.
- **To** enrich requests (e.g., correlation IDs, tenant resolution) before they reach application logic.
- **To** keep `Program.cs` clean by encapsulating middleware logic in dedicated classes.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Request: {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context);
        _logger.LogInformation("Response: {StatusCode}", context.Response.StatusCode);
    }
}

// Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Constructor | Receives `RequestDelegate` (the next middleware) and dependencies. |
| `InvokeAsync` | Contains the middleware logic; calls `_next(context)` to proceed. |
| Registration | `app.UseMiddleware<T>()` adds it to the pipeline. |

#### Syntax Rules

- **Folder:** Custom middleware resides in a `Middleware` folder.
- **Ordering:** Middleware order in `Program.cs` is critical; exception handling should come early, routing before authentication.
- **Constructor injection** – Dependencies are injected into the constructor; `InvokeAsync` receives `HttpContext`.

#### Constraints and Limitations

- **Singleton lifetime** – Middleware constructors are called once; do not inject scoped services into the constructor. Use `InvokeAsync` parameters or `HttpContext.RequestServices` for scoped dependencies.
- **Performance** – Middleware runs on every request; keep logic lightweight.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Request Logging Middleware

```csharp
// Middleware/RequestLoggingMiddleware.cs
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Handling {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context);
    }
}

// Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Step-by-step explanation:**

1. The middleware logs the request method and path.
2. `await _next(context)` passes control to the next middleware.
3. Registration order determines when logging occurs.

**Expected Output:** Each request produces a log entry with its method and path.

**Why this output?** The middleware runs for every request, logging before passing control down the pipeline.

---

#### Example 2: Exception Handling Middleware

```csharp
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(new { error = "An unexpected error occurred." });
        }
    }
}
```

**Expected Output:** Unhandled exceptions are caught, logged, and converted to a 500 JSON response.

**Why this output?** The middleware wraps the rest of the pipeline in a try/catch, providing centralised error handling.

### Real-World Cases

- **Logging:** Correlation ID middleware enriches logs with request identifiers.
- **Error handling:** Global exception middleware returns consistent error responses.
- **Multi-tenancy:** Tenant resolution middleware identifies the tenant from the host or route.
- **Security:** Custom middleware for API key validation or rate limiting.

### References

- ASP.NET Core Middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Clean Architecture project structure (Middleware folder) – https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/a04e954077f81a28b402fb8114cc1d0e341ac8b6/.claude/rules/project-structure.md

---

## 9. Filters

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filters are attributes or classes that run **before or after** specific stages of the MVC action pipeline, enabling cross-cutting concerns to be applied declaratively. |
| **Technical Definition** | Filters implement interfaces such as `IActionFilter`, `IExceptionFilter`, `IResultFilter`, `IAuthorizationFilter`, or `IResourceFilter`. They are applied as attributes (`[MyFilter]`) on controllers or actions, or registered globally. Filters execute at well-defined points in the MVC pipeline: authorization → resource → model binding → action → result. The `Filters` folder typically contains custom validation filters, exception filters, and action filters. |
| **Beginner-Friendly Explanation** | Filters are **automatic helpers** that run around your controller actions. An action filter might log how long an action takes; an exception filter might catch errors and return a friendly page; a validation filter might check the model before the action runs. You attach them with an attribute, and they run automatically—no need to call them from every action. |

### Purposes (each begins with “To”)

- **To** apply cross-cutting concerns (validation, logging, caching, exception handling) declaratively, without repeating code in every action.
- **To** intercept the action pipeline at specific stages (before/after action, before/after result).
- **To** centralise error handling at the controller or action level.
- **To** enforce authorization rules or custom validation logic.
- **To** keep controllers focused on their primary responsibility by moving reusable concerns to filters.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class ValidationFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(context.ModelState);
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage
[ServiceFilter(typeof(ValidationFilter))]
public IActionResult Create(ProductDto dto) { ... }
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `IActionFilter` | Runs before (`OnActionExecuting`) and after (`OnActionExecuted`) the action. |
| `IExceptionFilter` | Catches exceptions thrown by the action. |
| `IResultFilter` | Runs before and after the result is executed. |
| Attribute usage | `[MyFilter]` on controllers or actions. |
| Global registration | `builder.Services.AddControllers(options => options.Filters.Add<MyFilter>());` |

#### Syntax Rules

- **Folder:** Filters reside in a `Filters` folder.
- **Attribute vs. `ServiceFilter`** – Attributes with no dependencies can be applied directly; filters with dependencies use `[ServiceFilter(typeof(T))]` or `[TypeFilter(typeof(T))]`.
- **Global filters** – Registered in `AddControllers` for all actions.

#### Constraints and Limitations

- **Order of execution** – Authorization filters run first, then resource, action, exception, and result filters.
- **Dependency injection** – Attribute-based filters cannot use constructor injection; use `ServiceFilter` or `TypeFilter` for DI.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Validation Filter

```csharp
// Filters/ValidationFilter.cs
public class ValidationFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(context.ModelState);
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage on a controller
[ApiController]
[Route("api/[controller]")]
[ServiceFilter(typeof(ValidationFilter))]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductDto dto) => Ok(dto);
}
```

**Step-by-step explanation:**

1. `ValidationFilter` checks `ModelState` before the action executes.
2. If invalid, it short-circuits with a `400 Bad Request`.
3. `[ServiceFilter]` resolves the filter via DI.

**Expected Output:** Invalid requests receive a 400 response with validation errors; valid requests reach the action.

**Why this output?** The filter runs before the action and prevents execution when the model is invalid.

---

#### Example 2: Exception Filter

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;

    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger) => _logger = logger;

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");
        context.Result = new ObjectResult(new { error = "An error occurred." })
        {
            StatusCode = 500
        };
        context.ExceptionHandled = true;
    }
}

// Global registration
builder.Services.AddControllers(options => options.Filters.Add<GlobalExceptionFilter>());
```

**Expected Output:** All unhandled exceptions are logged and converted to a consistent 500 JSON response.

**Why this output?** The exception filter catches exceptions globally, centralising error handling.

### Real-World Cases

- **API validation:** A global validation filter returns 400 responses automatically.
- **Audit logging:** An action filter logs who called which action and when.
- **Caching:** A result filter caches action outputs for a specified duration.
- **Authorization:** Custom `IAuthorizationFilter` for API key validation.

### References

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Clean Architecture project structure (Filters folder) – https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/a04e954077f81a28b402fb8114cc1d0e341ac8b6/.claude/rules/project-structure.md

---

## 10. Extensions

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The `Extensions` folder contains **extension methods** for `IServiceCollection` and `IApplicationBuilder`, encapsulating DI registration and middleware configuration to keep `Program.cs` clean and modular. |
| **Technical Definition** | Extension methods in the `Extensions` folder typically follow the pattern `public static IServiceCollection AddXxx(this IServiceCollection services, IConfiguration config)` and `public static IApplicationBuilder UseXxx(this IApplicationBuilder app)`. These methods group related registrations (e.g., `AddApplicationServices`, `AddInfrastructure`) and pipeline configurations (e.g., `UseCustomMiddleware`). This pattern is widely used in Clean Architecture to keep `Program.cs` concise and to make configuration reusable across projects. |
| **Beginner-Friendly Explanation** | Extensions are **shortcut methods** that bundle up repetitive setup code. Instead of writing 20 lines of service registrations in `Program.cs`, you write `builder.Services.AddApplicationServices();`—and that extension method contains all the registrations. It keeps `Program.cs` tidy and makes it easy to reuse the same setup in multiple projects. |

### Purposes (each begins with “To”)

- **To** keep `Program.cs` clean and readable by moving DI registrations and middleware configuration into dedicated extension methods.
- **To** group related registrations (e.g., all application services, all infrastructure services) into cohesive units.
- **To** enable reuse of configuration across multiple projects or environments.
- **To** improve testability by isolating registration logic.
- **To** follow the **separation of concerns** principle in application startup.

### Syntax Rules and Structure

#### General Syntax

```csharp
// Extensions/ServiceCollectionExtensions.cs
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        services.AddScoped<IOrderService, OrderService>();
        return services;
    }
}

// Extensions/ApplicationBuilderExtensions.cs
public static class ApplicationBuilderExtensions
{
    public static IApplicationBuilder UseCustomMiddleware(this IApplicationBuilder app)
    {
        app.UseMiddleware<RequestLoggingMiddleware>();
        return app;
    }
}

// Program.cs
builder.Services.AddApplicationServices();
app.UseCustomMiddleware();
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ServiceCollectionExtensions` | Extension methods for `IServiceCollection` (DI registration). |
| `ApplicationBuilderExtensions` | Extension methods for `IApplicationBuilder` (middleware pipeline). |
| `this` parameter | The type being extended (`IServiceCollection` or `IApplicationBuilder`). |

#### Syntax Rules

- **Folder:** Extensions reside in an `Extensions` folder.
- **Naming:** `AddXxx` for service registration; `UseXxx` for middleware.
- **Return the original instance** – Extension methods should return `services` or `app` to allow chaining.

#### Constraints and Limitations

- **Over-extension** – Too many small extension methods can make it hard to trace registrations; group logically.
- **Order sensitivity** – Middleware extension methods must be called in the correct order in `Program.cs`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Service Registration Extension

```csharp
// Extensions/ServiceCollectionExtensions.cs
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IEmailService, EmailService>();
        return services;
    }

    public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(config.GetConnectionString("Default")));
        services.AddScoped<IProductRepository, ProductRepository>();
        return services;
    }
}

// Program.cs
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructure(builder.Configuration);
```

**Step-by-step explanation:**

1. `AddApplicationServices` registers all business services.
2. `AddInfrastructure` registers EF Core and repositories.
3. `Program.cs` remains concise and readable.

**Expected Output:** All services are registered in DI with a clean `Program.cs`.

**Why this output?** Extension methods encapsulate registration logic, making startup configuration modular and maintainable.

---

#### Example 2: Middleware Pipeline Extension

```csharp
// Extensions/ApplicationBuilderExtensions.cs
public static class ApplicationBuilderExtensions
{
    public static IApplicationBuilder UseCustomPipeline(this IApplicationBuilder app)
    {
        app.UseMiddleware<RequestLoggingMiddleware>();
        app.UseMiddleware<ExceptionHandlingMiddleware>();
        return app;
    }
}

// Program.cs
app.UseCustomPipeline();
```

**Expected Output:** The custom middleware pipeline is configured with a single method call.

**Why this output?** The extension method groups related middleware, reducing clutter in `Program.cs`.

### Real-World Cases

- **Clean Architecture:** `AddApplication`, `AddInfrastructure`, `AddPresentation` extension methods for each layer.
- **Modular monolith:** Each module registers its own services via an extension method.
- **Shared libraries:** Extension methods packaged in NuGet for reuse across projects.

### References

- Clean Architecture project structure (Extensions folder) – https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/a04e954077f81a28b402fb8114cc1d0e341ac8b6/.claude/rules/project-structure.md
- ASP.NET Core Best Practices and Coding Standards (Extensions) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 11. Configuration

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Configuration in ASP.NET Core is the system for loading settings from multiple sources (JSON files, environment variables, command-line arguments) into a strongly typed options model. |
| **Technical Definition** | Configuration is built via `WebApplication.CreateBuilder(args)`, which loads `appsettings.json`, `appsettings.{Environment}.json`, environment variables, and command-line arguments in a defined order. The **options pattern** binds configuration sections to POCO classes (`IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`). A `Configuration` folder may house options classes and extension methods that encapsulate configuration binding. |
| **Beginner-Friendly Explanation** | Configuration is **how your app reads its settings**. Things like database connection strings, API keys, and feature flags live in `appsettings.json` (or environment variables in production). Your code reads these settings through strongly typed classes, so you get IntelliSense and compile-time checking instead of magic strings. |

### Purposes (each begins with “To”)

- **To** externalise settings from code, allowing different configurations per environment (Development, Staging, Production).
- **To** provide strongly typed access to settings via the **options pattern**, reducing magic strings and runtime errors.
- **To** support multiple configuration sources (JSON, environment variables, secrets) with a clear precedence order.
- **To** keep sensitive values (connection strings, API keys) out of source control using **User Secrets** or environment variables.
- **To** organise related settings into cohesive options classes.

### Syntax Rules and Structure

#### General Syntax

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=...;Database=..."
  },
  "Jwt": {
    "Issuer": "https://myapp.com",
    "Audience": "https://myapp.com",
    "Key": "super-secret-key"
  }
}
```

```csharp
// Configuration/JwtOptions.cs
public class JwtOptions
{
    public const string SectionName = "Jwt";
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public string Key { get; set; } = string.Empty;
}

// Program.cs
builder.Services.Configure<JwtOptions>(
    builder.Configuration.GetSection(JwtOptions.SectionName));
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `appsettings.json` | Base configuration file. |
| `appsettings.{Environment}.json` | Environment-specific overrides. |
| Options class | POCO that maps to a configuration section. |
| `Configure<T>` | Binds a configuration section to an options class. |

#### Syntax Rules

- **Folder:** Options classes conventionally reside in a `Configuration` folder, though they can also live in `Core/Configuration` in Clean Architecture.
- **`SectionName` constant** – Define a constant for the section name to avoid magic strings.
- **Precedence** – Environment variables override JSON files; command-line arguments override environment variables.

#### Constraints and Limitations

- **Secrets management** – Never commit secrets to source control; use User Secrets (development) or Azure Key Vault / environment variables (production).
- **Options validation** – Use `ValidateDataAnnotations()` or `ValidateOnStart()` to catch misconfiguration early.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strongly Typed Options

```csharp
// Configuration/JwtOptions.cs
public class JwtOptions
{
    public const string SectionName = "Jwt";
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public string Key { get; set; } = string.Empty;
}

// Program.cs
builder.Services.Configure<JwtOptions>(
    builder.Configuration.GetSection(JwtOptions.SectionName));

// Usage in a service
public class TokenService
{
    private readonly JwtOptions _options;

    public TokenService(IOptions<JwtOptions> options) => _options = options.Value;
}
```

**Step-by-step explanation:**

1. `JwtOptions` maps to the `"Jwt"` section.
2. `Configure<JwtOptions>` binds the section.
3. `IOptions<JwtOptions>` is injected where needed.

**Expected Output:** `TokenService` receives strongly typed JWT settings without magic strings.

**Why this output?** The options pattern provides compile-time checking and centralised configuration.

---

#### Example 2: Configuration Validation

```csharp
builder.Services.AddOptions<JwtOptions>()
    .Bind(builder.Configuration.GetSection(JwtOptions.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

**Expected Output:** The application fails fast at startup if required settings are missing or invalid.

**Why this output?** `ValidateOnStart` catches misconfiguration immediately, rather than at first use.

### Real-World Cases

- **Multi-environment:** Different connection strings for Development, Staging, and Production.
- **Feature flags:** `FeatureManagement` section toggles features per environment.
- **Third-party APIs:** API keys and endpoints configured per environment.

### References

- Configuration in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/
- Options pattern in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
- Clean Architecture project structure (Configuration folder) – https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/a04e954077f81a28b402fb8114cc1d0e341ac8b6/.claude/rules/project-structure.md

---

## 12. wwwroot

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `wwwroot` is the **web root folder** that contains static assets (CSS, JavaScript, images, fonts, libraries) served directly to clients. |
| **Technical Definition** | `wwwroot` is the default **web root** in ASP.NET Core, configured via `WebHostDefaults.WebRootKey` or the `UseWebRoot` method. Static files middleware (`app.UseStaticFiles()`) serves files from this folder at the root URL path. Files in `wwwroot` are **not** processed by Razor or MVC; they are served as-is. The folder is published to the output directory during deployment. |
| **Beginner-Friendly Explanation** | `wwwroot` is the **public folder** of your website. It holds the files that the browser downloads directly: stylesheets, JavaScript files, images, and fonts. When a user visits `/css/site.css`, the server simply sends that file from `wwwroot/css/site.css`—no controller or view is involved. |

### Purposes (each begins with “To”)

- **To** store static assets that are served directly to clients without server-side processing.
- **To** keep static files separate from application code, improving organisation.
- **To** provide a predictable location for CSS, JavaScript, images, and client-side libraries.
- **To** enable caching and compression of static assets via middleware.
- **To** ensure static files are included in published output.

### Syntax Rules and Structure

#### Typical Folder Structure

```
wwwroot/
├── css/
│   └── site.css
├── js/
│   └── site.js
├── lib/
│   ├── bootstrap/
│   └── jquery/
├── images/
│   └── logo.png
└── favicon.ico
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `css/` | Stylesheets. |
| `js/` | JavaScript files. |
| `lib/` | Client-side libraries (Bootstrap, jQuery, etc.). |
| `images/` | Image assets. |
| `favicon.ico` | Site icon. |

#### Syntax Rules

- **Folder:** `wwwroot` is at the project root by default.
- **Middleware:** `app.UseStaticFiles()` must be called to serve files.
- **URL path:** Files are served at the root URL path (e.g., `/css/site.css`).
- **Publishing:** All files in `wwwroot` are published to the output directory.

#### Constraints and Limitations

- **No server-side processing** – Files in `wwwroot` are served as-is; Razor syntax is not processed.
- **Security** – Do not place sensitive files (e.g., `appsettings.json`) in `wwwroot`; they would be publicly accessible.
- **Caching** – Static files are cached by browsers; use versioning or cache busting for updates.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Serving Static Files

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseStaticFiles(); // Serves files from wwwroot

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Step-by-step explanation:**

1. `UseStaticFiles()` enables static file serving from `wwwroot`.
2. A request to `/css/site.css` returns `wwwroot/css/site.css`.
3. A request to `/` is handled by MVC routing.

**Expected Output:** Static files are served directly; MVC handles dynamic routes.

**Why this output?** `UseStaticFiles` short-circuits the pipeline for matching static files, bypassing MVC.

---

#### Example 2: Referencing Static Files in a View

```cshtml
@* Views/Shared/_Layout.cshtml *@
<link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
<script src="~/js/site.js" asp-append-version="true"></script>
```

**Expected Output:** The browser loads `site.css` and `site.js` from `wwwroot`, with cache-busting version hashes appended.

**Why this output?** The `asp-append-version` Tag Helper appends a hash to the URL, ensuring browsers fetch updated files when content changes.

### Real-World Cases

- **Bootstrap and jQuery:** Client-side libraries stored in `wwwroot/lib`.
- **Custom CSS/JS:** Application-specific styles and scripts.
- **Images and fonts:** Logos, icons, and web fonts.
- **SPA fallback:** Static SPA files served from `wwwroot` with a fallback route.

### References

- Static files in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files
- ASP.NET Core directory structure (wwwroot) – https://github.com/dotnet/aspnetcore.docs/blob/d58cb4ac9362818d442f860d2d5cd84bce08eae2/aspnetcore/host-and-deploy/directory-structure.md

---

## 13. Areas

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Areas are a **logical partitioning mechanism** that divides an ASP.NET Core MVC application into smaller functional groups, each with its own controllers, views, and models. |
| **Technical Definition** | Areas introduce an additional **`area` route parameter** into the routing hierarchy alongside `controller` and `action`. Controllers are associated with an area via the `[Area("AreaName")]` attribute. Area routes are registered in `Program.cs` using `MapControllerRoute` with an `{area:exists}` token or `MapAreaControllerRoute`. Areas create a separate **namespace for routing** and a **folder structure for views** (`/Areas/<AreaName>/Views/...`). |
| **Beginner-Friendly Explanation** | Areas are **departments** inside your application. A large e-commerce app might have an `Admin` area, a `Shop` area, and a `Blog` area. Each area has its own controllers and views, but they all live inside the same project. Areas let you have a `HomeController` in `Admin` and another `HomeController` in `Shop` without conflict—because they’re in different areas. |

### Purposes (each begins with “To”)

- **To** partition large applications into **high-level functional components** that can be developed independently.
- **To** allow controllers with the **same name** to coexist in different areas, avoiding naming conflicts.
- **To** provide a clear folder structure that mirrors the application’s functional decomposition.
- **To** enable independent development and ownership of different functional areas within a team.
- **To** support area-specific views, layouts, and view imports.

### Syntax Rules and Structure

#### Typical Folder Structure

```
Areas/
├── Products/
│   ├── Controllers/
│   │   └── HomeController.cs
│   └── Views/
│       └── Home/
│           └── Index.cshtml
└── Services/
    ├── Controllers/
    │   └── HomeController.cs
    └── Views/
        └── Home/
            └── Index.cshtml
```

#### Controller Association

```csharp
[Area("Products")]
public class HomeController : Controller { ... }
```

#### Route Registration

```csharp
// Generic area route (must come before default route)
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

// Named area route
app.MapAreaControllerRoute(
    name: "products",
    areaName: "Products",
    pattern: "Products/{controller=Home}/{action=Index}/{id?}");
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `[Area("Name")]` | Associates a controller with an area. |
| `{area:exists}` | Route token that matches any registered area. |
| `MapAreaControllerRoute` | Sets area defaults and constraints for a specific area. |

#### Syntax Rules

- **Folder:** Areas live in the `Areas` folder at the project root.
- **Route order:** Area routes must be registered **before** the default route.
- **View discovery:** Area views are discovered in `/Areas/<AreaName>/Views/...`.

#### Constraints and Limitations

- **No `AreaRegistration` class** – ASP.NET Core does not use the legacy `AreaRegistration` mechanism.
- **`_ViewImports` is not inherited** – each area needs its own `_ViewImports.cshtml`.
- **Over-use** – Areas add routing complexity; use them for **high-level functional modules**, not every small feature.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Area Setup

```csharp
// Areas/Admin/Controllers/DashboardController.cs
[Area("Admin")]
public class DashboardController : Controller
{
    public IActionResult Index() => Content("Admin Dashboard");
}

// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step-by-step explanation:**

1. `[Area("Admin")]` associates the controller with the `Admin` area.
2. The `{area:exists}` route captures `"Admin"` as the area.
3. A request to `/Admin/Dashboard/Index` matches the area route.

**Expected Output:** `GET /Admin/Dashboard/Index` returns `"Admin Dashboard"`.

**Why this output?** The area route extracts the area name and selects the controller with the matching `[Area]` attribute.

---

#### Example 2: Same Controller Name in Different Areas

```csharp
// Areas/Products/Controllers/HomeController.cs
[Area("Products")]
public class HomeController : Controller
{
    public IActionResult Index() => Content("Products Home");
}

// Areas/Services/Controllers/HomeController.cs
[Area("Services")]
public class HomeController : Controller
{
    public IActionResult Index() => Content("Services Home");
}
```

**Expected Output:**

| URL | Response |
|-----|----------|
| `/Products/Home/Index` | `Products Home` |
| `/Services/Home/Index` | `Services Home` |

**Why this output?** Areas create separate routing namespaces, allowing identical controller names to coexist.

### Real-World Cases

- **E-commerce:** Areas for `Admin`, `Shop`, `Checkout`, and `Billing`.
- **SaaS:** Areas for `TenantAdmin`, `SuperAdmin`, and `Public`.
- **CMS:** Areas for `Blog`, `Docs`, and `Shop`.

### References

- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- Areas in ASP.NET Core (GitHub) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/controllers/areas.md

---

## 14. Project Structure Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Project structure best practices are guidelines for organising files and folders in an ASP.NET Core MVC application to maximise maintainability, scalability, testability, and team productivity. |
| **Technical Definition** | Best practices encompass **layer separation** (Web, Services, Data, Core), **feature organisation** (Areas or feature folders for large apps), **thin controllers**, **ViewModels for views**, **interface-driven services**, **configuration externalisation**, **static asset management**, and **consistent naming conventions**. For enterprise applications, **Clean Architecture** (dependencies pointing inward toward the core domain) is the recommended approach. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep your project tidy as it grows. For example: keep controllers thin (they shouldn’t contain business logic), use ViewModels for forms (not domain models), put database code in a `Data` folder, and use Areas when your app has distinct functional modules. Following these practices makes your code easier to understand, test, and change. |

### Purposes (each begins with “To”)

- **To** maintain **separation of concerns** so that changes in one layer don’t ripple through the entire application.
- **To** enable **testability** by isolating business logic in services and data access in repositories.
- **To** support **scalability** as the application and team grow (Areas, feature folders, multi-project solutions).
- **To** enforce **consistency** through naming conventions and folder organisation.
- **To** prevent common pitfalls such as over-posting, fat controllers, and tight coupling to EF Core.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Keep controllers thin** | Business logic belongs in services; thin controllers are easier to test and maintain | Controller calls `_service.GetAllAsync()` and returns `View(result)` |
| **Use ViewModels for views** | Prevents over-posting and decouples views from domain models | `ProductDetailsViewModel` instead of `Product` |
| **Separate data access** | Isolate EF Core `DbContext` and configurations in a `Data` folder | `Data/AppDbContext.cs`, `Data/Configurations/` |
| **Use Areas for high-level modules** | Areas partition large apps into functional groups | `Areas/Admin`, `Areas/Shop` |
| **Interface-driven services** | Enables DI, mocking, and testability | `IProductService` / `ProductService` |
| **Keep `Program.cs` clean** | Use extension methods for DI and middleware | `builder.Services.AddApplicationServices()` |
| **Externalise configuration** | Use `appsettings.json` and the options pattern; never hard-code secrets | `IOptions<JwtOptions>` |
| **Use `wwwroot` for static assets** | Keep static files separate from code | `wwwroot/css/site.css` |
| **Follow Clean Architecture for large apps** | Dependencies point inward toward the core domain | Web → Services → Data → Core |
| **Consider feature folders for large teams** | Reduces scrolling and improves cohesion | `Features/Products/` containing controller, views, and models |

#### Constraints and Limitations

- **Clean Architecture overhead** – Multi-project solutions add complexity; use them when the application’s size justifies it.
- **Repositories are not mandatory** – EF Core’s `DbContext` already provides repository-like functionality; add repositories only when they add clear value.
- **Areas vs. feature folders** – Areas are framework-supported; feature folders require custom conventions. Choose based on team preference and framework support.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Layered Solution Structure (Clean Architecture)

```
SolutionName/
├── src/
│   ├── SolutionName.Api/               # Presentation Layer
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Filters/
│   │   ├── Extensions/
│   │   ├── Program.cs
│   │   └── appsettings.json
│   ├── SolutionName.Core/              # Domain + Application Layer
│   │   ├── Entities/
│   │   ├── Interfaces/
│   │   └── Services/
│   ├── SolutionName.Infrastructure/    # Infrastructure Layer
│   │   ├── Data/
│   │   ├── Repositories/
│   │   └── ExternalServices/
│   └── SolutionName.Shared/            # Shared code
│       ├── DTOs/
│       └── Extensions/
└── tests/
    ├── SolutionName.UnitTests/
    └── SolutionName.IntegrationTests/
```

**Step-by-step explanation:**

1. **Api** contains controllers, middleware, filters, and startup configuration.
2. **Core** contains domain entities, interfaces, and business services (no dependencies on EF Core).
3. **Infrastructure** contains EF Core `DbContext`, repositories, and external service implementations.
4. **Shared** contains DTOs and cross-cutting utilities.
5. Dependencies point inward: Api → Core, Infrastructure → Core; Core has no outward dependencies.

**Expected Output:** A maintainable, testable solution where layers can evolve independently.

**Why this output?** Clean Architecture enforces separation of concerns and dependency inversion, making the application easier to test and maintain.

---

#### Example 2: Feature Folder Structure (Alternative to Areas)

```
Features/
├── Products/
│   ├── ProductsController.cs
│   ├── ProductService.cs
│   ├── IProductService.cs
│   ├── ViewModels/
│   │   └── ProductDetailsViewModel.cs
│   └── Views/
│       ├── Index.cshtml
│       └── Details.cshtml
└── Orders/
    ├── OrdersController.cs
    ├── OrderService.cs
    └── Views/
        └── Checkout.cshtml
```

**Step-by-step explanation:**

1. Each feature has its own folder containing controller, service, ViewModels, and views.
2. This reduces jumping between `Controllers/`, `Views/`, `Models/`, and `Services/` folders.
3. Custom conventions (e.g., `IControllerModelConvention`) can be used to locate views in feature folders.

**Expected Output:** All files related to a feature are in one place, improving cohesion.

**Why this output?** Feature folders reduce cognitive load and make it easier to see all related code for a feature.

### Real-World Cases

- **Enterprise e-commerce:** Clean Architecture with separate Web, Services, Data, and Core projects.
- **Modular monolith:** Areas for `Admin`, `Shop`, and `Blog`, each with its own controllers and views.
- **Small team:** Feature folders within a single project to reduce navigation overhead.

### References

- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- .NET Project Structure (Clean Architecture) – https://github.com/asiaostrich/universal-dev-standards/blob/a8112c5f730d3a5236a8f46991168a6033b26a8c/options/project-structure/dotnet.md
- Modern Web Apps: Develop ASP.NET Core MVC Apps (feature folders) – https://raw.githubusercontent.com/dotnet/docs/8fb16db4d447cfe3a4be029ab35ba08175baf3da/docs/architecture/modern-web-apps-azure/develop-asp-net-core-mvc-apps.md
- ASP.NET Core MVC project structure: a practical layered architecture – https://dev.to/rgk/aspnet-core-mvc-project-structure-a-practical-layered-architecture-43o8

---

## Summary Table: Project Structure at a Glance

| Folder / Concept | Primary Purpose | Key Convention | When to Use |
|------------------|----------------|----------------|-------------|
| **Controllers** | Handle HTTP requests | `Controllers/` folder, `*Controller` suffix | Every MVC application |
| **Views** | Render HTML | `Views/{Controller}/{Action}.cshtml` | MVC with server-rendered UI |
| **Models** | Represent data | `Models/` folder, POCOs with validation | Domain entities and data contracts |
| **ViewModels** | Shape data for views | `ViewModels/` folder, view-specific properties | Forms, complex views, over-posting prevention |
| **Services** | Business logic | `Services/` folder, interface + implementation | Any non-trivial business logic |
| **Repositories** | Data-access abstraction | `Data/Repositories/` folder | When abstraction/testing justifies the extra layer |
| **Data** | EF Core `DbContext` and configurations | `Data/` folder, `IEntityTypeConfiguration<T>` | Any EF Core–backed application |
| **Middleware** | HTTP pipeline components | `Middleware/` folder, `InvokeAsync` | Cross-cutting HTTP concerns |
| **Filters** | Action pipeline concerns | `Filters/` folder, `IActionFilter` etc. | Validation, exception handling, caching |
| **Extensions** | Clean `Program.cs` | `Extensions/` folder, `AddXxx` / `UseXxx` | Grouping DI and middleware configuration |
| **Configuration** | Options and settings | `Configuration/` folder, `IOptions<T>` | Any configurable application |
| **wwwroot** | Static assets | Root folder, `UseStaticFiles()` | CSS, JS, images, libraries |
| **Areas** | Functional partitioning | `Areas/{Name}/Controllers`, `Views` | Large apps with distinct modules |
| **Best Practices** | Maintainability and scalability | Thin controllers, ViewModels, Clean Architecture | All production applications |

---

## References (Full List)

- ASP.NET Core MVC project structure (Views overview) – https://learn.microsoft.com/es-es/aspnet/core/mvc/views/overview?view=aspnetcore-9.0
- ASP.NET Core directory structure – https://github.com/dotnet/aspnetcore.docs/blob/d58cb4ac9362818d442f860d2d5cd84bce08eae2/aspnetcore/host-and-deploy/directory-structure.md
- Modern Web Apps: Develop ASP.NET Core MVC Apps (Areas, feature folders) – https://raw.githubusercontent.com/dotnet/docs/8fb16db4d447cfe3a4be029ab35ba08175baf3da/docs/architecture/modern-web-apps-azure/develop-asp-net-core-mvc-apps.md
- Modular ASP.NET Core MVC Template – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- .NET Project Structure (Clean Architecture) – https://github.com/asiaostrich/universal-dev-standards/blob/a8112c5f730d3a5236a8f46991168a6033b26a8c/options/project-structure/dotnet.md
- Clean Architecture Project Structure (ASP.NET Core) – https://github.com/dinhnguyenngoc/spec-driven-claude-code/blob/a04e954077f81a28b402fb8114cc1d0e341ac8b6/.claude/rules/project-structure.md
- ASP.NET Core Development Guidelines (LobeHub) – https://lobehub.com/skills/mindrally-skills-aspnet-core
- Areas in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/controllers/areas.md
- ASP.NET Core MVC project structure: a practical layered architecture – https://dev.to/rgk/aspnet-core-mvc-project-structure-a-practical-layered-architecture-43o8
- Controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Middleware in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Configuration in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/
- Options pattern in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
- Static files in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files
- Areas in ASP.NET Core (official docs) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas