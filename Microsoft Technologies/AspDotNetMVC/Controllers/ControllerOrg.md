# Controller Organization in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Controller discovery and naming conventions have been stable since ASP.NET Core 1.0. While the framework imposes minimal hard requirements (any public class suffixed with `Controller` or inheriting from `Controller` is discovered), the **conventions** described here are the foundation of maintainable projects. Areas and feature folders are alternative organizational strategies covered where relevant. Namespace conventions matter less in ASP.NET Core than in legacy MVC, but they remain important for tooling and team consistency.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller organization is the practice of structuring controller files, naming classes, and arranging folders in an ASP.NET Core MVC project to maximise **discoverability, cohesion, and maintainability** as the application grows. |
| **Technical Definition** | Controller organization encompasses **file location** (the `Controllers` folder at the project root), **class naming** (`<Name>Controller` convention), **namespace structure** (mirroring folders), and **grouping strategies** (traditional layer-first vs. feature folders vs. Areas). The framework discovers controllers via assembly scanning; a class is treated as a controller if its name ends with `Controller`, inherits from a class whose name ends with `Controller`, or is decorated with `[Controller]` (and not `[NonController]`). Controllers are activated and disposed **per request** and should follow the **Explicit Dependencies Principle**. |
| **Beginner-Friendly Explanation** | Controller organization is **how you arrange your controller files** so that you and your team can find things quickly. The framework expects controllers in a `Controllers` folder with names ending in `Controller` (like `ProductsController`). As your app grows, you choose how to group controllers—all in one folder, split by feature, or organised into Areas. Good organisation makes the codebase easier to navigate and maintain. |

### Key Characteristics

- **Convention-based discovery** – The framework finds controllers automatically; no registration required.
- **Suffix convention** – `Controller` suffix is **highly recommended** (though not strictly mandatory in ASP.NET Core).
- **Per-request lifetime** – Controllers are instantiated per request, not singletons.
- **Thin by design** – Controllers should delegate business logic to services, following the **Single Responsibility Principle (SRP)** .
- **Multiple grouping strategies** – Traditional layer-first, feature folders, or Areas, each with trade-offs.
- **Namespace mirroring** – Namespaces conventionally mirror folder structure (e.g., `MyApp.Areas.Admin.Controllers`).

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project (`dotnet new mvc` or Visual Studio template).
- Basic understanding of **controllers**, **routing**, and **dependency injection**.

### Related Programming Areas

- **Routing** – Controllers are mapped to URLs via routing.
- **Areas** – A built-in partitioning mechanism for large applications.
- **Feature Folders** – A community-driven organizational pattern.
- **Dependency Injection** – Controllers receive services via constructor injection.
- **Namespace Conventions** – Mirror folder structure for clarity.

### Core Concepts / Features

1. **Controller Organization (General)** – The overall strategy and framework requirements.
2. **Controller Naming** – Suffix conventions, naming patterns, and versioning.
3. **Controller Folders** – Traditional `Controllers/`, feature folders, and Areas.
4. **Controller Best Practices** – SRP, thin controllers, naming, and structural guidelines.

---

## 1. Controller Organization (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller organization is the structural arrangement of controller classes and their files to support **discoverability and separation of concerns** in an ASP.NET Core MVC project. |
| **Technical Definition** | Controllers are discovered from the application assembly by scanning for public classes that satisfy at least one of: name ending in `Controller`, inheritance from a class ending in `Controller`, or `[Controller]` attribute application. By convention, controllers reside in the root-level `Controllers` folder. The framework does **not** require a specific namespace; controllers can be placed in any folder and namespace and still be discovered. However, tooling and team conventions strongly favour the `Controllers` folder with a matching namespace. |
| **Beginner-Friendly Explanation** | Controller organization is **how you decide where to put your controller files and what to name them**. The framework is flexible—it will find your controllers as long as they're named correctly. But following consistent conventions makes your project easier to navigate, especially as it grows. |

### Purposes (each begins with “To”)

- **To** make controllers easy to find and navigate for all team members.
- **To** group related actions logically, enabling common routing, caching, and authorization rules to be applied collectively.
- **To** support the **Single Responsibility Principle** by keeping controllers focused and cohesive.
- **To** scale the application’s structure from a small project to a large enterprise codebase.
- **To** facilitate tooling (IDE navigation, scaffolding, static analysis) that relies on conventions.

### Syntax Rules and Structure

#### Framework Requirements (Minimum)

| Condition | Satisfied by |
|-----------|--------------|
| Name ends with `Controller` | `ProductsController` ✅ |
| Inherits from a class ending with `Controller` | `class MyController : Controller` ✅ |
| Has `[Controller]` attribute | `[Controller] public class MyClass` ✅ |
| Must **not** have `[NonController]` | `[NonController]` excludes the class ❌ |

#### Conventional Structure

```
ProjectName/
├── Controllers/               ← Root-level folder (convention)
│   ├── HomeController.cs
│   ├── ProductsController.cs
│   └── OrdersController.cs
├── Models/
├── Views/
└── Program.cs
```

#### Syntax Rules

- **Place controllers in `Controllers/`** for tooling and team consistency.
- **Suffix class names with `Controller`** – highly recommended, though not strictly mandatory.
- **Namespace should mirror the folder path** (e.g., `MyApp.Controllers`).
- **Controllers must be `public`** to be discovered.
- **`[NonController]`** can be used to exclude a public class from discovery.

#### Constraints and Limitations

- **Namespace is unimportant for discovery** – ASP.NET Core scans the assembly, not the namespace.
- **No hard requirement for folder location** – controllers can technically reside anywhere.
- **Convention matters for humans**, even when the framework is flexible.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Conventional Controller in `Controllers/`

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers
{
    public class ProductsController : Controller
    {
        public IActionResult Index() => View();
    }
}
```

**Step-by-step explanation:**

1. The class is named `ProductsController` (suffix convention).
2. It resides in `Controllers/` (folder convention).
3. Its namespace is `MyApp.Controllers` (mirrors folder).
4. It inherits from `Controller` (base class convention).

**Expected Output:** The controller is discovered automatically; `/Products/Index` routes to `Index()`.

**Why this output?** The framework scans the assembly for classes satisfying the controller criteria; this class satisfies all conventions.

---

#### Example 2: Controller Without `Controller` Suffix (Using `[Controller]`)

```csharp
// Controllers/Analytics.cs
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers
{
    [Controller]
    public class Analytics : ControllerBase
    {
        [HttpGet("/analytics")]
        public IActionResult Get() => Ok("Analytics data");
    }
}
```

**Expected Output:** The class is discovered as a controller despite lacking the suffix, because `[Controller]` is applied.

**Why this output?** The `[Controller]` attribute explicitly marks the class for discovery, overriding the naming convention.

### Real-World Cases

- **Small MVC app** – a flat `Controllers/` folder with `HomeController`, `AccountController`, and `ProductsController`.
- **Growing app** – controllers grouped by feature (see Section 3) to reduce navigation friction.
- **API-only project** – controllers in `Controllers/` with `[ApiController]` and route attributes.

### References

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- ASP.NET Core Best Practices and Coding Standards (directory structure) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- Stack Overflow: Is controller suffix mandatory? – https://stackoverflow.com/questions/78656522/is-controller-suffix-is-mandatory-in-net-core-net-6

---

## 2. Controller Naming

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller naming is the convention of suffixing controller class names with `Controller` (e.g., `ProductsController`), enabling the framework to discover them and the routing system to derive clean URL segments. |
| **Technical Definition** | ASP.NET Core provides a built-in convention: a controller’s name is the class name with the `Controller` suffix trimmed when it appears exactly at the end. For versioned APIs, the convention is extended to `<Name>[#]Controller`, where trailing numbers are removed for grouping purposes (e.g., `Values2Controller` → `Values`). While the suffix is not strictly mandatory if the class inherits from `Controller` or has `[Controller]`, it is **highly recommended** for consistency and tooling compatibility. |
| **Beginner-Friendly Explanation** | Controller naming is the **rule that controller classes end with `Controller`** — like `ProductsController` instead of just `Products`. This tells the framework “this is a controller,” and it also makes the URL cleaner: `/Products` instead of `/ProductsController`. |

### Purposes (each begins with “To”)

- **To** signal to the framework that a class is a controller (via the suffix convention).
- **To** produce clean URL segments (the suffix is trimmed from route values).
- **To** maintain consistency across the codebase, improving readability.
- **To** support tooling (scaffolding, IDE navigation, static analysis) that relies on naming conventions.
- **To** enable versioned API naming patterns (`ValuesController`, `Values2Controller`).

### Syntax Rules and Structure

#### General Syntax

```csharp
public class <Name>Controller : Controller { ... }
public class <Name>[#]Controller : ControllerBase { ... }   // Versioned API
```

#### Component Breakdown

| Pattern | Example | Route Segment |
|---------|---------|---------------|
| `<Name>Controller` | `ProductsController` | `Products` |
| `<Name>[#]Controller` | `Values2Controller` | `Values` (number trimmed) |
| `[Controller]` attribute (no suffix) | `Analytics` | `Analytics` |

#### Syntax Rules

- **Suffix is `Controller`** – `ProductsController` not `Products`.
- **Trimmed for routing** – the suffix is removed when the route value is computed.
- **Trailing numbers are trimmed** for versioned controllers (`Values2Controller` → `Values`).
- **Namespaces are irrelevant** for discovery but should follow conventions.

#### Constraints and Limitations

- **Suffix is not strictly mandatory** if the class inherits from `Controller` or has `[Controller]`.
- **Inconsistency causes confusion** – mixing suffixed and non-suffixed controllers makes the codebase harder to navigate.
- **Razor Pages** use a different naming model (page names, not controller names).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Standard Naming

```csharp
public class ProductsController : Controller { ... }
public class OrdersController : Controller { ... }
```

**Expected Output:** Routes are `/Products/...` and `/Orders/...` (suffix trimmed).

**Why this output?** The framework trims the `Controller` suffix when deriving the controller route value.

---

#### Example 2: Versioned API Naming

```csharp
public class ValuesController : ControllerBase { ... }
public class Values2Controller : ControllerBase { ... }
```

**Expected Output:** Both controllers are grouped under `Values` for API versioning purposes.

**Why this output?** API Versioning’s naming convention trims the suffix and trailing numbers, treating both as the same logical resource.

---

#### Example 3: `[Controller]` Attribute Without Suffix

```csharp
[Controller]
public class Analytics : ControllerBase { ... }
```

**Expected Output:** The controller is discovered, and its route segment is `Analytics`.

**Why this output?** The `[Controller]` attribute explicitly marks the class, bypassing the suffix requirement.

### Real-World Cases

- **Standard MVC app** – `HomeController`, `AccountController`, `ProductsController`.
- **Versioned API** – `CustomersController`, `Customers2Controller` for v1 and v2.
- **Internal tooling** – `Dashboard`, `Reports` classes with `[Controller]` for brevity.

### References

- Controller Naming Conventions (API Versioning) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/naming-conventions.html
- Is controller suffix mandatory? – https://learn.microsoft.com/en-us/answers/questions/1663066/is-controller-suffix-mandatory-in-net-core-net-6
- Create a controller – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/tutorials/first-mvc-app/adding-controller.md

---

## 3. Controller Folders

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller folders are the directory structures used to organise controller files—ranging from the default flat `Controllers/` folder to feature-based folders and Areas. |
| **Technical Definition** | Three primary strategies exist: (1) **Traditional layer-first** – all controllers in a single root-level `Controllers/` folder, organized by technical role. (2) **Feature folders** – folders organized around features, each containing its own controller, views, and models. (3) **Areas** – a built-in ASP.NET Core feature that partitions the app into functional groups, each with its own `Controllers/` and `Views/` folders. ASP.NET Core’s discovery is assembly-based, so folder structure is a **human organizational choice**, not a framework requirement. |
| **Beginner-Friendly Explanation** | Controller folders are **where you put your controller files**. In a small app, you dump them all in one `Controllers` folder. As the app grows, you might group them by feature (a `Products` folder containing `ProductsController`, its views, and models) or use Areas for major sections like `Admin` or `Shop`. The framework will find them either way—but good organization helps your team. |

### Purposes (each begins with “To”)

- **To** organise controllers in a way that matches the team’s mental model of the application.
- **To** reduce navigation friction as the number of controllers grows.
- **To** increase **cohesion** by keeping related files (controller, view, view model) together in feature folders.
- **To** support modular development and potential extraction of areas into separate assemblies.
- **To** leverage the built-in **Areas** feature for large applications with distinct functional modules.

### Syntax Rules and Structure

#### Strategy 1: Traditional Layer-First (Default)

```
ProjectName/
├── Controllers/           ← All controllers
│   ├── HomeController.cs
│   ├── ProductsController.cs
│   └── OrdersController.cs
├── Models/
├── Views/
└── Program.cs
```

- **Pros:** Familiar, simple, framework-default.
- **Cons:** Lacks cohesion; files related to a feature are scattered across `Controllers/`, `Views/`, and `Models/`.

#### Strategy 2: Feature Folders

```
ProjectName/
├── Features/
│   ├── Products/
│   │   ├── ProductsController.cs
│   │   ├── Views/
│   │   └── ViewModels/
│   └── Orders/
│       ├── OrdersController.cs
│       └── Views/
```

- **Pros:** High cohesion; everything related to a feature in one place.
- **Cons:** Requires custom view location configuration (e.g., `AddFeatureFolders` NuGet package).

#### Strategy 3: Areas

```
ProjectName/
├── Areas/
│   ├── Products/
│   │   ├── Controllers/
│   │   │   └── HomeController.cs
│   │   └── Views/
│   └── Services/
│       ├── Controllers/
│       └── Views/
```

- **Pros:** Built-in framework support; clean URL partitioning (`/Products/...`).
- **Cons:** More ceremony (area routes, `[Area]` attribute); overkill for small apps.

#### Syntax Rules

- **Traditional:** Controllers in root `Controllers/`.
- **Feature folders:** Controllers co-located with their feature’s views and models; requires custom view discovery.
- **Areas:** Controllers in `Areas/<AreaName>/Controllers/` with `[Area("AreaName")]` attribute; area routes must precede default routes.
- **Namespace mirroring** is recommended for all strategies.

#### Constraints and Limitations

- **Framework does not enforce folder location** – only naming/attributes matter for discovery.
- **Feature folders require configuration** – the default view engine won’t find views outside `Views/` without customisation.
- **Areas add routing complexity** – area routes must be ordered correctly and `[Area]` attributes applied.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Traditional Layer-First Structure

```csharp
// Controllers/ProductsController.cs
namespace MyApp.Controllers
{
    public class ProductsController : Controller { ... }
}
```

**Expected Output:** Controller is discovered; views are in `Views/Products/`.

**Why this output?** The default view engine searches `Views/{Controller}/` for views; the flat structure keeps all controllers in one place.

---

#### Example 2: Feature Folder Structure

```
Features/
└── Products/
    ├── ProductsController.cs
    ├── Views/
    │   └── Index.cshtml
    └── ViewModels/
        └── ProductListViewModel.cs
```

```csharp
// Features/Products/ProductsController.cs
namespace MyApp.Features.Products
{
    public class ProductsController : Controller { ... }
}
```

**Expected Output:** Controller is discovered; views are resolved via custom feature folder conventions.

**Why this output?** The feature folder groups everything related to products together, increasing cohesion.

---

#### Example 3: Area Structure

```
Areas/
└── Admin/
    ├── Controllers/
    │   └── DashboardController.cs
    └── Views/
        └── Dashboard/
            └── Index.cshtml
```

```csharp
// Areas/Admin/Controllers/DashboardController.cs
namespace MyApp.Areas.Admin.Controllers
{
    [Area("Admin")]
    public class DashboardController : Controller { ... }
}
```

**Expected Output:** `/Admin/Dashboard/Index` routes to the area controller.

**Why this output?** The `[Area]` attribute and area route configuration enable partitioned routing.

### Real-World Cases

- **Small app (< 10 controllers)** – traditional `Controllers/` folder is sufficient.
- **Medium app (10–30 controllers)** – feature folders reduce navigation friction.
- **Large app with distinct modules** – Areas for `Admin`, `Shop`, `Blog`, etc..
- **Modular monolith** – feature folders or Areas plus separate projects for services/data.

### References

- Feature Slices for ASP.NET Core MVC – https://learn.microsoft.com/en-us/archive/msdn-magazine/2016/september/asp-net-core-feature-slices-for-asp-net-core-mvc
- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- ASP.NET Core Best Practices and Coding Standards (directory structure) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- .NET Core Standards (feature folders) – https://raw.githubusercontent.com/thenerdery/dot-net-standards/refs/heads/master/DotNetCore.md

---

## 4. Controller Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Controller best practices are guidelines for organising and writing controllers that are **thin, focused, testable, and consistent**, following the **Single Responsibility Principle (SRP)** and the **Explicit Dependencies Principle**. |
| **Technical Definition** | Best practices include: keeping controllers thin (delegating business logic to services), following SRP (one reason to change), using constructor injection for shared dependencies and action injection for single-action dependencies, naming controllers consistently with the `Controller` suffix, mirroring namespaces to folders, avoiding fat controllers, and choosing an organisational strategy (traditional/feature/Areas) that matches the application’s size and team structure. |
| **Beginner-Friendly Explanation** | Controller best practices are the **rules of thumb** that keep controllers clean and maintainable. The most important rule: **controllers should be thin**—they should only handle HTTP concerns, not business logic. Use services for real work, follow naming conventions, and keep each controller focused on one area of responsibility. |

### Purposes (each begins with “To”)

- **To** keep controllers thin and focused on HTTP concerns (request validation, response selection).
- **To** follow the **Single Responsibility Principle** – each controller should have one reason to change.
- **To** improve testability by delegating business logic to services that can be independently tested.
- **To** maintain consistency across the codebase through naming and structural conventions.
- **To** prevent **fat controllers** that are hard to read, maintain, and test.
- **To** enable scalability by choosing an organisational strategy that grows with the application.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Suffix names with `Controller`** | Framework discovery and clean route segments. | `ProductsController`, not `Products` |
| **Place controllers in `Controllers/`** | Tooling and team conventions. | Root-level `Controllers/` folder |
| **Mirror namespaces to folders** | Clarity and tooling support. | `MyApp.Controllers`, `MyApp.Areas.Admin.Controllers` |
| **Keep controllers thin** | Business logic belongs in services. | Controller calls `_service.GetAllAsync()` and returns `View(result)` |
| **Follow SRP** | One reason to change per controller. | Split `MediaController` into `MovieController` and `PhotoController` |
| **Use constructor injection for shared services** | Explicit dependencies, testability. | `public ProductsController(IProductService svc)` |
| **Use action injection for single-action services** | Avoids noisy constructors. | `[FromServices] ICsvExporter` in one action |
| **Choose feature folders for medium/large apps** | Higher cohesion, less navigation friction. | `Features/Products/ProductsController.cs` |
| **Use Areas for distinct modules** | Built-in partitioning with URL prefixes. | `Areas/Admin/Controllers/DashboardController.cs` |
| **Avoid fat controllers** | Hard to read, test, and maintain. | Extract logic to services and filters |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Thin Controller with Service Delegation

```csharp
// Controllers/ProductsController.cs
public class ProductsController : Controller
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
        => _productService = productService;

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

**Step-by-step explanation:**

1. Controller is thin: only handles HTTP concerns.
2. Business logic is delegated to `IProductService`.
3. Constructor injection supplies the dependency (Explicit Dependencies Principle).
4. Actions return `View`, `NotFound`, etc.

**Expected Output:** HTML pages listing products and showing details.

**Why this output?** The controller orchestrates HTTP concerns; the service handles data retrieval and business rules.

---

#### Example 2: Feature Folder Organization

```
Features/
├── Products/
│   ├── ProductsController.cs
│   ├── Views/
│   │   └── Index.cshtml
│   └── ProductListViewModel.cs
└── Orders/
    ├── OrdersController.cs
    └── Views/
        └── Checkout.cshtml
```

**Expected Output:** All product-related files are co-located, improving cohesion.

**Why this output?** Feature folders group by purpose rather than technical type, reducing navigation friction.

---

#### Example 3: Area Organization for Admin Section

```
Areas/
└── Admin/
    ├── Controllers/
    │   ├── DashboardController.cs   // [Area("Admin")]
    │   └── UsersController.cs       // [Area("Admin")]
    └── Views/
        ├── Dashboard/
        └── Users/
```

```csharp
[Area("Admin")]
public class DashboardController : Controller
{
    public IActionResult Index() => View();
}
```

**Expected Output:** `/Admin/Dashboard/Index` routes to the area controller.

**Why this output?** The `[Area]` attribute and area routing configuration partition the admin functionality.

### Real-World Cases

- **E-commerce** – `ProductsController`, `OrdersController`, `CartController` with thin actions delegating to services.
- **Admin panel** – `Areas/Admin` with `DashboardController`, `UsersController`, `SettingsController`.
- **API** – `[ApiController]` classes in `Controllers/` with `[Route]` attributes, returning `ActionResult<T>`.

### References

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Overview of ASP.NET Core MVC (SRP for controllers) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- SonarAnalyzer: Controllers should not have mixed responsibilities – https://github.com/sonarsource/sonar-dotnet/blob/master/analyzers/rspec/cs/S6960.html
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- Feature Slices for ASP.NET Core MVC – https://learn.microsoft.com/en-us/archive/msdn-magazine/2016/september/asp-net-core-feature-slices-for-asp-net-core-mvc
- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas

---

## Summary Table: Controller Organization at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Controller Organization** | Arrange files for discoverability | Controllers in `Controllers/`, suffix `Controller`, public class | Every ASP.NET Core MVC project |
| **Controller Naming** | Framework discovery + clean URLs | `<Name>Controller`; versioned `<Name>[#]Controller` | Every controller |
| **Controller Folders** | Organise by layer, feature, or Area | Traditional (small), Feature folders (medium/large), Areas (large modules) | Match strategy to app size and team |
| **Best Practices** | Thin, focused, testable controllers | SRP, Explicit Dependencies, avoid fat controllers | All production controllers |

---

## References (Full List)

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Overview of ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Feature Slices for ASP.NET Core MVC – https://learn.microsoft.com/en-us/archive/msdn-magazine/2016/september/asp-net-core-feature-slices-for-asp-net-core-mvc
- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- Controller Naming Conventions (API Versioning) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/naming-conventions.html
- Is controller suffix mandatory? (Microsoft Q&A) – https://learn.microsoft.com/en-us/answers/questions/1663066/is-controller-suffix-mandatory-in-net-core-net-6
- Stack Overflow: Is controller suffix mandatory? – https://stackoverflow.com/questions/78656522/is-controller-suffix-is-mandatory-in-net-core-net-6
- ASP.NET Core Best Practices and Coding Standards (directory structure, naming) – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- .NET Core Standards (feature folders) – https://raw.githubusercontent.com/thenerdery/dot-net-standards/refs/heads/master/DotNetCore.md
- SonarAnalyzer: Controllers should not have mixed responsibilities – https://github.com/sonarsource/sonar-dotnet/blob/master/analyzers/rspec/cs/S6960.html
- Modular ASP.NET Core MVC Template (project structure) – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- Stack Overflow: How to get a list of all routes in ASP.NET Core? – https://stackoverflow.com/questions/28435734/how-to-get-a-list-of-all-routes-in-asp-net-core