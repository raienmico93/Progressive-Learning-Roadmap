# Areas in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10/11). Areas in ASP.NET Core differ significantly from legacy ASP.NET MVC 5: **there is no `AreaRegistration` class** and no `RegisterAllAreas()` method. Area registration is achieved through **routing configuration** in `Program.cs` and the **`[Area]` attribute** on controllers. Legacy `AreaRegistration` APIs (`System.Web.Mvc`) are noted only to clarify migration differences. All examples use **endpoint routing** (`UseRouting` / `MapControllerRoute` / `MapAreaControllerRoute`), which became the default in ASP.NET Core 3.0.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An **Area** is a logical partition of an ASP.NET Core MVC application that groups related functionality—controllers, views, models, and Razor Pages—under a separate routing namespace and folder structure. |
| **Technical Definition** | Areas introduce an additional route parameter, `area`, into the routing hierarchy alongside `controller` and `action` (or `page` for Razor Pages). The `[Area("AreaName")]` attribute associates a controller with a specific area. Area routes are registered in `Program.cs` using `MapControllerRoute` with an `{area:...}` token or `MapAreaControllerRoute`. Areas enable multiple controllers with the **same name** to coexist as long as they belong to **different areas** . |
| **Beginner‑Friendly Explanation** | An Area is like a **separate department** inside a company. The company (your application) has different departments (areas) like **Sales**, **Support**, and **Admin**. Each department has its own staff (controllers), documents (views), and filing system (models), but they all work under the same company name. Without areas, you’d have to give every employee a unique name; with areas, you can have a “Manager” in Sales and a “Manager” in Support without confusion. |

### Key Characteristics

- **Logical partitioning** – Splits large applications into smaller, independently workable functional groups .
- **Routing hierarchy** – Adds an `area` route parameter, creating URLs like `/{area}/{controller}/{action}/{id?}` .
- **Namespace isolation** – Allows controllers with identical names in different areas (`Products.HomeController` and `Services.HomeController`) .
- **Folder‑based view discovery** – Views are located under `/Areas/<AreaName>/Views/...` with a well‑defined search order .
- **Conventional routing preferred** – Area routes are typically registered using conventional routing (not attribute routing) and must be placed **before** non‑area routes because they are more specific .
- **Supports Razor Pages** – Razor Pages can also be organised into areas under `Areas/<AreaName>/Pages` .
- **Link generation awareness** – `asp-area` Tag Helper, `Url.Action`, and `Html.ActionLink` accept an `area` route value to generate area‑aware links .

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()`).
- Routing middleware configured (`UseRouting()`, `UseAuthorization()`).
- Basic understanding of **controllers**, **views**, **route templates**, and **conventional routing**.

### Related Programming Areas

- **Routing** – Areas extend routing with an `area` parameter.
- **Controllers** – Controllers are associated with areas via `[Area]`.
- **Views** – Area views follow a dedicated folder structure and search order.
- **URL Generation** – Area‑aware link generation via `asp-area`, `Url.Action`, and `Html.ActionLink`.
- **Razor Pages** – Areas can contain Razor Pages under `Areas/<AreaName>/Pages`.
- **Modularisation** – Areas provide a built‑in modularisation boundary for large applications.

### Core Concepts / Features

1. **Areas (General)** – The concept, folder structure, and when to use them.
2. **Area Registration** – How areas are registered in ASP.NET Core (routing configuration + `[Area]` attribute).
3. **Area Routing** – `MapControllerRoute` with `{area:exists}`, `MapAreaControllerRoute`, and route ordering.
4. **Area Views** – Folder structure, view discovery order, `_ViewStart`, and `_ViewImports`.
5. **Area Best Practices** – Guidelines for effective and maintainable use of areas.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real‑World Cases, and References.

---

## 1. Areas (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Areas are a built‑in ASP.NET feature used to organise related functionality into a group as a separate **routing namespace** and **folder structure** for views and Razor Pages. |
| **Technical Definition** | An area is a structure inside an application. The ASP.NET Core runtime uses naming conventions and routing configuration to associate controllers, views, and models with an area. Using areas creates a routing hierarchy by adding an `area` route parameter to `controller` and `action` (or `page`). Controllers without the `[Area]` attribute do not belong to any area and are not matched when an `area` route value is supplied . |
| **Beginner‑Friendly Explanation** | Areas are **folders with superpowers**. In a normal MVC app, all controllers live in one `Controllers` folder and all views in one `Views` folder. As the app grows, this becomes messy. Areas let you create sub‑folders like `Areas/Admin/Controllers` and `Areas/Admin/Views`, and the framework automatically routes `/Admin/...` URLs to the controllers inside that area. |

### Purposes (each begins with “To”)

- **To** partition a large ASP.NET Core web application into smaller, logically separated functional groups .
- **To** allow multiple controllers with the **same name** to coexist without naming conflicts, as long as they belong to different areas .
- **To** provide a clear folder structure that mirrors the application’s functional decomposition, improving maintainability .
- **To** enable independent development and ownership of different functional areas within a team .
- **To** support area‑specific views, layouts, and view imports through a well‑defined discovery mechanism .
- **To** facilitate modular application design, including potential extraction of areas into separate assemblies or Razor Class Libraries.

### Syntax Rules and Structure

#### Typical Folder Structure

```
ProjectName/
├── Areas/
│   ├── Products/
│   │   ├── Controllers/
│   │   │   ├── HomeController.cs
│   │   │   └── ManageController.cs
│   │   └── Views/
│   │       ├── Home/
│   │       │   └── Index.cshtml
│   │       └── Manage/
│   │           ├── Index.cshtml
│   │           └── About.cshtml
│   └── Services/
│       ├── Controllers/
│       │   └── HomeController.cs
│       └── Views/
│           └── Home/
│               └── Index.cshtml
├── Controllers/
├── Views/
└── Program.cs
```

**Component breakdown:**

| Folder | Purpose |
|--------|---------|
| `Areas/<AreaName>/Controllers` | Controllers belonging to the area. |
| `Areas/<AreaName>/Views` | Views belonging to the area. |
| `Areas/<AreaName>/Models` | (Optional) Models specific to the area. |
| `Areas/<AreaName>/Pages` | (For Razor Pages) Pages belonging to the area. |

> **Important:** While the above layout is typical, **only the view files are required** to use this folder structure. Controllers can reside elsewhere as long as they carry the `[Area]` attribute .

#### Syntax Rules

- **`[Area]` attribute required** – A controller belongs to an area only if it is decorated with `[Area("AreaName")]` .
- **Area routes before default routes** – Area routes are more specific and must be registered **earlier** in the route table than non‑area routes .
- **No `AreaRegistration` class** – Unlike legacy ASP.NET MVC 5, ASP.NET Core does **not** use `AreaRegistration` classes or `RegisterAllAreas()`. Registration is purely through routing configuration .
- **Case sensitivity** – Area names are case‑insensitive in routing but conventionally use PascalCase.

#### Constraints and Limitations

- **Legacy API removal** – `AreaRegistration`, `AreaRegistrationContext`, and `RegisterAllAreas()` are **not available** in ASP.NET Core; code relying on them must be migrated to routing configuration .
- **Order dependence** – Area routes must precede default routes; otherwise, the default route may swallow area URLs .
- **No automatic area discovery** – Areas are not “scanned” automatically; each area route must be explicitly registered or covered by a generic area pattern.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Minimal Area Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseRouting();
app.UseAuthorization();

// Area route must come BEFORE the default route
app.MapControllerRoute(
    name: "MyArea",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

```csharp
// Areas/Products/Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Areas.Products.Controllers;

[Area("Products")]                    // Associates controller with "Products" area
public class HomeController : Controller
{
    public IActionResult Index() => Content("Products Area Home");
}
```

**Step‑by‑step explanation:**

1. The `{area:exists}` route is registered **before** the default route. The `exists` constraint ensures the area segment corresponds to a known area.
2. `[Area("Products")]` marks `HomeController` as belonging to the `Products` area.
3. A request to `/Products/Home/Index` matches the area route with `area = "Products"`, `controller = "Home"`, `action = "Index"`.

**Expected Output:**

| URL | Matched Controller | Response |
|-----|-------------------|----------|
| `/Products/Home/Index` | `Areas/Products/Controllers/HomeController` | `Products Area Home` |
| `/` | Root `HomeController` (if exists) | Root home page |

**Why this output?** The `{area:exists}` token captures `"Products"` and the routing system restricts matching to controllers decorated with `[Area("Products")]` .

---

#### Example 2: Same Controller Name in Multiple Areas

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

// Root Controllers/HomeController.cs (no area)
public class HomeController : Controller
{
    public IActionResult Index() => Content("Root Home");
}
```

**Step‑by‑step explanation:**

1. Three controllers share the name `HomeController` but belong to different areas (or no area).
2. The `area` route value disambiguates them.
3. `/Products/Home/Index` selects the `Products` area controller; `/Services/Home/Index` selects the `Services` area controller; `/Home/Index` selects the root controller.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/Products/Home/Index` | `Products Home` |
| `/Services/Home/Index` | `Services Home` |
| `/Home/Index` | `Root Home` |

**Why this output?** Areas create separate routing namespaces, allowing identical controller names to coexist without conflict .

---

#### Example 3: Area with Razor Pages

```csharp
// Areas/Products/Pages/Index.cshtml
@page
@model MyApp.Areas.Products.Pages.IndexModel
<h1>Products Razor Page</h1>

// Program.cs (same routing configuration)
app.MapControllerRoute(
    name: "MyArea",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. Razor Pages in an area reside under `Areas/Products/Pages`.
2. The area route pattern also applies to Razor Pages via the `area` route value.
3. The page is accessible at `/Products/Index` (or `/Products` if the page is named `Index`).

**Expected Output:**

| URL | Rendered Page |
|-----|---------------|
| `/Products/Index` | `Products Razor Page` |

**Why this output?** Razor Pages within areas use the same `area` route parameter for matching .

### Real‑World Cases

- **E‑commerce application:** Areas for `Checkout`, `Billing`, `Search`, and `Admin` separate distinct business units .
- **SaaS platform:** Areas for `TenantAdmin`, `SuperAdmin`, and `Public` provide clear boundaries between administrative levels.
- **CMS:** Areas for `Blog`, `Shop`, and `Docs` allow independent content teams to work without conflicts.

### References

- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- Routing to controller actions: Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#areas
- Sample code (Areas) – https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/areas/samples

---

## 2. Area Registration

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Area registration is the process of configuring ASP.NET Core to recognise an area: associating controllers with the area via the `[Area]` attribute and registering an area‑aware route in `Program.cs`. |
| **Technical Definition** | In ASP.NET Core, area registration consists of two parts: (1) decorating controllers with `[Area("AreaName")]` so they are included in the area’s routing namespace, and (2) adding a route to the endpoint routing table that includes an `area` parameter (either `{area:exists}` in a shared pattern or `MapAreaControllerRoute` with a specific `areaName`). Unlike legacy ASP.NET MVC 5, there is **no `AreaRegistration` class** or `RegisterAllAreas()` method; registration is entirely routing‑driven . |
| **Beginner‑Friendly Explanation** | Area registration is **telling the framework that an area exists**. You do two things: (1) put a label (`[Area("Admin")]`) on each controller that belongs to the area, and (2) add a routing rule in `Program.cs` that says “URLs starting with `/Admin` should look for controllers with the `Admin` label.” That’s it—no separate registration classes needed. |

### Purposes (each begins with “To”)

- **To** associate controllers with a specific area so they are matched only when the corresponding `area` route value is present .
- **To** register area‑aware routes that include the `area` parameter in the routing table.
- **To** ensure that area routes are evaluated before non‑area routes, preserving correct matching precedence .
- **To** provide a migration path from legacy `AreaRegistration`‑based code to modern routing configuration.
- **To** support both a single generic area pattern (`{area:exists}`) and multiple named area routes (`MapAreaControllerRoute`) depending on URL design .

### Syntax Rules and Structure

#### General Syntax (Controller Attribute)

```csharp
[Area("AreaName")]
public class MyController : Controller { ... }
```

#### General Syntax (Routing Configuration – Generic Area Pattern)

```csharp
app.MapControllerRoute(
    name: "MyArea",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

#### General Syntax (Routing Configuration – Named Area Routes)

```csharp
app.MapAreaControllerRoute(
    name: "MyAreaProducts",
    areaName: "Products",
    pattern: "Products/{controller=Home}/{action=Index}/{id?}");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[Area("AreaName")]` | Attribute on the controller class; associates it with the named area. |
| `{area:exists}` | Route token that captures the area name and applies the `exists` constraint (the area must exist). |
| `MapAreaControllerRoute` | Convenience method that creates a route with a default `area` value and an `area` constraint for the specified `areaName` . |

#### Syntax Rules

- **`[Area]` attribute is mandatory** for controllers to belong to an area .
- **Area routes must precede default routes** in `Program.cs` .
- **`MapAreaControllerRoute` is equivalent** to calling `MapControllerRoute` with `defaults: new { area = "AreaName" }` and `constraints: new { area = "AreaName" }` .
- **`{area:exists}` is the least complicated mechanism** for adding area routing when the URL space is uniform across all areas .
- **No `AreaRegistration` class** – ASP.NET Core does not use `AreaRegistration` or `RegisterAllAreas()` .

#### Constraints and Limitations

- **Legacy incompatibility** – Code that relies on `AreaRegistration.RegisterAllAreas()` will not compile or run in ASP.NET Core; it must be replaced with routing configuration .
- **Order sensitivity** – If the default route is registered before the area route, area URLs may be misinterpreted as controller/action combinations .
- **Explicit registration** – Each area route must be explicitly added; there is no automatic discovery of areas beyond the `[Area]` attribute and routing configuration.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Registering an Area with the `[Area]` Attribute and Generic Area Route

```csharp
// 1. Controller registration
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Areas.Admin.Controllers;

[Area("Admin")]                       // Step 1: Associate controller with area
public class DashboardController : Controller
{
    public IActionResult Index() => Content("Admin Dashboard");
}

// 2. Route registration in Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseRouting();
app.UseAuthorization();

// Step 2: Area route (must come before default)
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Step‑by‑step explanation:**

1. `[Area("Admin")]` registers `DashboardController` in the `Admin` area.
2. The `{area:exists}` route captures the first segment as the area name and restricts matching to controllers in that area.
3. The default route handles non‑area URLs.

**Expected Output:**

| URL | Matched Controller | Response |
|-----|-------------------|----------|
| `/Admin/Dashboard/Index` | `Areas/Admin/Controllers/DashboardController` | `Admin Dashboard` |
| `/Admin` | `Areas/Admin/Controllers/HomeController.Index` (if exists) | Depends on `HomeController` |
| `/Home/Index` | Root `HomeController` | Root home page |

**Why this output?** The `area` route value `"Admin"` is extracted from the URL and used to select the controller with the matching `[Area]` attribute .

---

#### Example 2: Registering Areas with `MapAreaControllerRoute`

```csharp
// Program.cs
app.MapAreaControllerRoute(
    name: "MyAreaProducts",
    areaName: "Products",
    pattern: "Products/{controller=Home}/{action=Index}/{id?}");

app.MapAreaControllerRoute(
    name: "MyAreaServices",
    areaName: "Services",
    pattern: "Services/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. Two named area routes are created, one for `Products` and one for `Services`.
2. Each route provides a default `area` value and a constraint for the specified area name.
3. The default route handles non‑area URLs.

**Expected Output:**

| URL | Area | Controller |
|-----|------|------------|
| `/Products/Home/Index` | `Products` | `Areas/Products/Controllers/HomeController` |
| `/Services/Home/Index` | `Services` | `Areas/Services/Controllers/HomeController` |

**Why this output?** `MapAreaControllerRoute` sets the `area` default and constraint, so the URL prefix (`Products/` or `Services/`) directly maps to the corresponding area .

---

#### Example 3: Migrating from Legacy `AreaRegistration` (Conceptual)

```csharp
// ❌ Legacy ASP.NET MVC 5 (does NOT work in ASP.NET Core):
// public class AdminAreaRegistration : AreaRegistration
// {
//     public override string AreaName => "Admin";
//     public override void RegisterArea(AreaRegistrationContext context)
//     {
//         context.MapRoute(
//             "Admin_default",
//             "Admin/{controller}/{action}/{id}",
//             new { action = "Index", id = UrlParameter.Optional });
//     }
// }
// AreaRegistration.RegisterAllAreas(); // Not available in ASP.NET Core

// ✅ ASP.NET Core equivalent:
[Area("Admin")]
public class DashboardController : Controller { ... }

// Program.cs
app.MapAreaControllerRoute(
    name: "Admin_default",
    areaName: "Admin",
    pattern: "Admin/{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. The legacy `AreaRegistration` class is replaced by the `[Area]` attribute on controllers.
2. The `RegisterArea` route mapping is replaced by `MapAreaControllerRoute` or `MapControllerRoute` with an area token.
3. `RegisterAllAreas()` is not needed; routing configuration in `Program.cs` handles registration.

**Expected Output:** The ASP.NET Core application correctly routes `/Admin/Dashboard/Index` to the `DashboardController` in the `Admin` area.

**Why this output?** ASP.NET Core’s routing system replaces the legacy `AreaRegistration` mechanism with endpoint routing and the `[Area]` attribute .

### Real‑World Cases

- **Migrating from ASP.NET MVC 5:** Replacing `AreaRegistration` classes with `[Area]` attributes and `MapAreaControllerRoute` calls.
- **Modular monolith:** Registering each module’s area with a dedicated `MapAreaControllerRoute` for explicit URL prefixes.
- **Multi‑tenant application:** Using `MapAreaControllerRoute` to register tenant‑specific areas with default area values.

### References

- Areas in ASP.NET Core: Associate the controller with an Area – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#associate-the-controller-with-an-area
- Areas in ASP.NET Core: Add Area route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#add-area-route
- `MapAreaControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapareacontrollerroute
- Legacy `AreaRegistration` (ASP.NET MVC 5) – https://learn.microsoft.com/en-us/dotnet/api/system.web.mvc.arearegistration

---

## 3. Area Routing

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Area routing is the configuration and matching behaviour that uses the `area` route parameter to select controllers and actions belonging to a specific area. |
| **Technical Definition** | Area routing extends conventional routing by adding an `area` route value to the `controller`, `action`, and optional `id` parameters. Two primary mechanisms exist: (1) a generic pattern using `{area:exists}` in `MapControllerRoute`, which matches **all** controllers with an `[Area]` attribute, and (2) `MapAreaControllerRoute`, which creates a named route with a default `area` value and an area constraint for a specific area . Area routes are order‑dependent and must be placed **earlier** in the route table than non‑area routes . |
| **Beginner‑Friendly Explanation** | Area routing is **how the framework knows which area a URL belongs to**. When a request comes in for `/Admin/Users/List`, the router looks at the first segment (`Admin`), says “That’s the Admin area,” and then looks for a `UsersController` inside the Admin area. Without area routing, the framework wouldn’t know that `Admin` is an area name rather than a controller name. |

### Purposes (each begins with “To”)

- **To** match incoming URLs to controllers and actions within a specific area based on the `area` route value .
- **To** distinguish between controllers with the same name in different areas .
- **To** provide URL prefixes (e.g., `/Products/`, `/Admin/`) that correspond to functional modules.
- **To** support both a single generic area pattern (uniform URL space) and multiple named area routes (specific prefixes) .
- **To** enable area‑aware link generation by preserving the `area` route value as an ambient value .

### Syntax Rules and Structure

#### General Syntax (Generic Area Pattern)

```csharp
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

#### General Syntax (Named Area Route)

```csharp
app.MapAreaControllerRoute(
    name: "routeName",
    areaName: "AreaName",
    pattern: "Prefix/{controller=Home}/{action=Index}/{id?}");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `{area:exists}` | Captures the area name; the `exists` constraint ensures the area is registered. |
| `areaName` | The name of the area (must match the `[Area]` attribute value). |
| `pattern` | The URL pattern; typically includes a literal prefix or the `{area}` token. |
| `defaults` / `constraints` | `MapAreaControllerRoute` automatically sets `defaults: new { area = areaName }` and `constraints: new { area = areaName }` . |

#### Syntax Rules

- **Area routes before default routes** – Area routes are more specific and must be registered earlier .
- **`{area:exists}` matches all area controllers** – It is the simplest mechanism for uniform area URL spaces .
- **`MapAreaControllerRoute` is equivalent** to `MapControllerRoute` with area defaults and constraints .
- **Ambient values** – When executing an action inside an area, the `area` route value becomes an **ambient value** for URL generation, meaning links generated from within the area default to that area unless overridden .

#### Constraints and Limitations

- **Order sensitivity** – Registering the default route before area routes causes area URLs to be misinterpreted .
- **Area name must match** – The `areaName` in `MapAreaControllerRoute` must match the `[Area("...")]` value on controllers.
- **Attribute routing interaction** – `MapAreaControllerRoute` maps both conventionally routed and attribute‑routed controllers, but area routes typically use conventional routing .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Generic Area Route with `{area:exists}`

```csharp
// Program.cs
app.MapControllerRoute(
    name: "MyArea",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. The first route includes `{area:exists}` as the first segment.
2. A request to `/Products/Home/Index` populates `area = "Products"`, `controller = "Home"`, `action = "Index"`.
3. The default route handles URLs without an area segment.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/Products/Home/Index` | `area = "Products"`, `controller = "Home"`, `action = "Index"` |
| `/Home/Index` | `controller = "Home"`, `action = "Index"` (no area) |

**Why this output?** The `{area:exists}` token captures the first segment and applies the `exists` constraint, ensuring the area is registered .

---

#### Example 2: Named Area Route with `MapAreaControllerRoute`

```csharp
app.MapAreaControllerRoute(
    name: "blog_route",
    areaName: "Blog",
    pattern: "Manage/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default_route",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. `MapAreaControllerRoute` creates a route named `"blog_route"` for the `"Blog"` area.
2. The URL pattern `Manage/{controller}/{action}/{id?}` maps `/Manage/Users/AddUser` to `area = "Blog"`, `controller = "Users"`, `action = "AddUser"`.
3. The default route handles non‑area URLs.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/Manage/Users/AddUser` | `area = "Blog"`, `controller = "Users"`, `action = "AddUser"` |

**Why this output?** `MapAreaControllerRoute` supplies the `area` default and constraint, so the `Manage/` prefix is mapped to the `Blog` area without requiring an explicit `{area}` token in the pattern .

---

#### Example 3: Area Ambient Values in Link Generation

```csharp
// Inside a view in the Products area:
<a asp-controller="Home" asp-action="About">Products/Home/About</a>
// Generates: /Products/Home/About (area ambient value preserved)

<a asp-area="" asp-controller="Home" asp-action="About">/Home/About</a>
// Generates: /Home/About (area explicitly cleared)
```

**Step‑by‑step explanation:**

1. When generating links from within an area, the `area` route value is an ambient value.
2. Omitting `asp-area` preserves the current area (sticky behaviour) .
3. Setting `asp-area=""` explicitly clears the area, generating a root‑level URL.

**Expected Output:**

| Tag Helper | Generated URL |
|------------|---------------|
| `<a asp-controller="Home" asp-action="About">` | `/Products/Home/About` (if current area is Products) |
| `<a asp-area="" asp-controller="Home" asp-action="About">` | `/Home/About` |

**Why this output?** Area route values are ambient; `asp-area=""` overrides the ambient value .

### Real‑World Cases

- **Uniform area URL space:** Using `{area:exists}` when all areas follow the same `/{area}/{controller}/{action}` pattern.
- **Custom area prefixes:** Using `MapAreaControllerRoute` to give an area a URL prefix that differs from its name (e.g., area `"Blog"` with prefix `"Manage"`) .
- **Area‑aware navigation:** Using `asp-area` to generate links that stay within or explicitly leave an area.

### References

- Routing to controller actions: Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#areas
- Areas in ASP.NET Core: Add Area route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#add-area-route
- `MapAreaControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapareacontrollerroute
- Link generation with MVC areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#link-generation-with-mvc-areas

---

## 4. Area Views

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Area views are Razor view files (`.cshtml`) that reside within an area’s folder structure and are discovered by the Razor view engine using an area‑specific search order. |
| **Technical Definition** | The Razor view engine searches for area views in the following order: (1) `/Areas/<Area-Name>/Views/<Controller-Name>/<Action-Name>.cshtml`, (2) `/Areas/<Area-Name>/Views/Shared/<Action-Name>.cshtml`, (3) `/Views/Shared/<Action-Name>.cshtml`, and (4) `/Pages/Shared/<Action-Name>.cshtml` . `_ViewStart.cshtml` and `_ViewImports.cshtml` require special handling in areas: `_ViewStart` in the area’s `Views` folder controls the area’s layout, while `_ViewImports` is **not** inherited from the root and must be added to each area’s view folder to enable Tag Helpers and imports . |
| **Beginner‑Friendly Explanation** | Area views are **the HTML files for an area**. They live in a special folder (`Areas/AreaName/Views`) so the framework knows they belong to that area. When an action returns `View()`, the framework looks for the view in the area’s folder first, then in shared folders. You can also give an area its own layout and its own set of Tag Helper imports. |

### Purposes (each begins with “To”)

- **To** provide area‑specific views that are isolated from root‑level views, reducing naming conflicts .
- **To** enable area‑specific layouts via `_ViewStart.cshtml` in the area’s `Views` folder .
- **To** allow area‑specific Tag Helper and namespace imports via `_ViewImports.cshtml` in the area’s `Views` folder .
- **To** support a well‑defined view discovery order that falls back to shared views when an area‑specific view is not found .
- **To** facilitate independent UI development for each functional area.

### Syntax Rules and Structure

#### View Discovery Order

```
1. /Areas/<Area-Name>/Views/<Controller-Name>/<Action-Name>.cshtml
2. /Areas/<Area-Name>/Views/Shared/<Action-Name>.cshtml
3. /Views/Shared/<Action-Name>.cshtml
4. /Pages/Shared/<Action-Name>.cshtml
```

#### Folder Structure for Area Views

```
Areas/
└── Products/
    └── Views/
        ├── _ViewStart.cshtml          // Area-specific layout configuration
        ├── _ViewImports.cshtml        // Area-specific imports (Tag Helpers, namespaces)
        ├── Home/
        │   └── Index.cshtml
        └── Shared/
            └── _Layout.cshtml         // (Optional) Area-specific layout
```

#### Syntax Rules

- **`_ViewStart.cshtml`** – Placed in `Areas/<AreaName>/Views/` to specify the layout for that area. If absent, the root `_ViewStart.cshtml` applies .
- **`_ViewImports.cshtml`** – **Not inherited** from the root. Must be added to `Areas/<AreaName>/Views/` to enable Tag Helpers and `@using` directives for views in that area .
- **View location formats can be customised** – The default area folder name (`"Areas"`) can be changed by configuring `RazorViewEngineOptions.AreaViewLocationFormats` .

#### Constraints and Limitations

- **`_ViewImports` is not inherited** – This is a common pitfall; Tag Helpers that work in root views may not work in area views unless `_ViewImports.cshtml` is added to the area .
- **Layout resolution** – If `_ViewStart.cshtml` in an area references a layout that exists only in the root `Views/Shared`, the layout may not resolve correctly; the layout should be in the area’s `Shared` folder or referenced with a full path .
- **Publishing** – All `.cshtml` files and `wwwroot` files are published to output when `<Project Sdk="Microsoft.NET.Sdk.Web">` is included .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic Area View Discovery

```csharp
// Areas/Products/Controllers/HomeController.cs
[Area("Products")]
public class HomeController : Controller
{
    public IActionResult Index() => View();
}
```

```cshtml
@* Areas/Products/Views/Home/Index.cshtml *@
<h1>Products Area Home View</h1>
```

**Step‑by‑step explanation:**

1. The action `HomeController.Index()` calls `View()`.
2. The view engine searches `/Areas/Products/Views/Home/Index.cshtml` first.
3. The file is found and rendered.

**Expected Output:** The page displays `Products Area Home View`.

**Why this output?** The view discovery order prioritises the area‑specific controller folder .

---

#### Example 2: Area‑Specific `_ViewStart.cshtml`

```cshtml
@* Areas/Products/Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";   // Uses Areas/Products/Views/Shared/_Layout.cshtml
}
```

```cshtml
@* Areas/Products/Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html>
<head><title>Products Area</title></head>
<body>
    @RenderBody()
</body>
</html>
```

**Step‑by‑step explanation:**

1. `_ViewStart.cshtml` in the area’s `Views` folder sets the layout for all views in that area.
2. The layout file is expected in the area’s `Shared` folder (or the root `Views/Shared` if not found in the area).

**Expected Output:** All views in the `Products` area use the area‑specific layout.

**Why this output?** `_ViewStart.cshtml` in the area overrides the root `_ViewStart.cshtml` for views in that area .

---

#### Example 3: Area‑Specific `_ViewImports.cshtml`

```cshtml
@* Areas/Products/Views/_ViewImports.cshtml *@
@using MyApp.Areas.Products
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

**Step‑by‑step explanation:**

1. The root `_ViewImports.cshtml` is **not** inherited by area views.
2. Adding `_ViewImports.cshtml` to the area’s `Views` folder enables Tag Helpers and imports for all views in that area.

**Expected Output:** Tag Helpers (e.g., `asp-action`, `asp-controller`) work correctly in area views.

**Why this output?** `_ViewImports.cshtml` must be explicitly added to the area to provide imports .

### Real‑World Cases

- **Area‑specific branding:** Each area uses its own `_Layout.cshtml` and `_ViewStart.cshtml` to apply distinct branding.
- **Modular UI development:** Teams working on different areas can maintain their own `_ViewImports.cshtml` without affecting other areas.
- **Custom area folder names:** Changing `AreaViewLocationFormats` to use `"Modules"` instead of `"Areas"` for a modular architecture .

### References

- Areas in ASP.NET Core: Area folder structure – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#area-folder-structure
- Areas in ASP.NET Core: Shared layout for Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#shared-layout-for-areas-using-the-_viewstartcshtml-file
- Areas in ASP.NET Core: _ViewImports – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#viewimportscshtml
- Changing the default area folder – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#change-default-area-folder-where-views-are-stored
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Importing Shared Directives – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives

---

## 5. Area Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Area best practices are guidelines for using areas effectively—maximising their organisational benefits while avoiding common pitfalls related to routing, views, and link generation. |
| **Technical Definition** | Best practices encompass route ordering (area routes before default routes), view imports management (`_ViewImports` per area), layout configuration (`_ViewStart` placement), link generation awareness (ambient `area` values), naming conventions, and the decision boundary between areas and other modularisation techniques (e.g., feature folders, separate projects). |
| **Beginner‑Friendly Explanation** | Best practices are the “rules of thumb” that keep areas tidy. For example: put area routes before the default route, don’t forget `_ViewImports` in each area, and use `asp-area=""` when you want a link to leave the current area. |

### Purposes (each begins with “To”)

- **To** ensure correct route matching by placing area routes before default routes .
- **To** avoid Tag Helper and namespace resolution issues by adding `_ViewImports.cshtml` to each area .
- **To** control layout inheritance correctly through `_ViewStart.cshtml` placement .
- **To** generate area‑aware links correctly by understanding ambient values and `asp-area` .
- **To** maintain a clean and scalable application structure by using areas only when appropriate .
- **To** facilitate migration from legacy ASP.NET MVC 5 by replacing `AreaRegistration` with modern routing configuration .

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Register area routes before default routes** | Area routes are more specific; order determines precedence . | `MapControllerRoute("{area:exists}/...")` before `MapControllerRoute("{controller}/...")` |
| **Use `{area:exists}` for uniform URL spaces** | Simplest mechanism for areas that follow the same pattern . | `pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}"` |
| **Use `MapAreaControllerRoute` for custom prefixes** | Explicitly sets area defaults and constraints . | `MapAreaControllerRoute("blog", "Blog", "Manage/{controller}/{action}")` |
| **Add `_ViewImports.cshtml` to each area** | Root `_ViewImports` is not inherited . | `Areas/Products/Views/_ViewImports.cshtml` |
| **Place `_ViewStart.cshtml` in the area’s `Views` folder** | Controls layout for the area . | `Areas/Products/Views/_ViewStart.cshtml` |
| **Use `asp-area=""` to leave an area** | Area is sticky for link generation . | `<a asp-area="" asp-controller="Home" asp-action="Index">` |
| **Use areas for high‑level functional modules** | Areas are ideal for large, independently workable components . | `Admin`, `Shop`, `Blog` |
| **Avoid over‑using areas** | Areas add routing and structural overhead; use feature folders within areas for finer organisation. | Areas for top‑level modules; feature folders inside. |
| **Migrate legacy `AreaRegistration` to `[Area]` + routing** | `AreaRegistration` is not available in ASP.NET Core . | Replace `RegisterAllAreas()` with route configuration. |

#### Constraints and Limitations

- **Route order is critical** – Misordering area and default routes leads to 404s or incorrect controller selection .
- **`_ViewImports` must be duplicated** – This is a common source of “Tag Helpers not working” errors in areas .
- **Layout resolution can be tricky** – Area‑specific layouts must reside in the area’s `Shared` folder or be referenced explicitly .
- **Ambient values can cause unexpected links** – Links generated without `asp-area` inside an area default to that area; explicitly clear the area when linking outside .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Correct Route Ordering

```csharp
// ✅ CORRECT: Area route first
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// ❌ INCORRECT: Default route first would swallow area URLs
// app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
// app.MapControllerRoute("areas", "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. Area route is registered first, so `/Products/Home/Index` is matched as area `Products`.
2. Default route handles non‑area URLs.
3. If the default route were first, `/Products/Home/Index` might be interpreted as controller `Products`, action `Home`, which would fail or invoke the wrong action.

**Expected Output:**

| URL | Correct Matching |
|-----|------------------|
| `/Products/Home/Index` | Area route (Products area) |
| `/Home/Index` | Default route (root) |

**Why this output?** Route order determines precedence; area routes must come first because they are more specific .

---

#### Example 2: Area‑Specific `_ViewImports` and `_ViewStart`

```cshtml
@* Areas/Admin/Views/_ViewImports.cshtml *@
@using MyApp.Areas.Admin
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

```cshtml
@* Areas/Admin/Views/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

**Step‑by‑step explanation:**

1. `_ViewImports.cshtml` enables Tag Helpers and imports for all views in the `Admin` area.
2. `_ViewStart.cshtml` sets the layout to `_AdminLayout.cshtml`, which should be placed in `Areas/Admin/Views/Shared/`.

**Expected Output:** Admin area views use `_AdminLayout` and have Tag Helpers available.

**Why this output?** `_ViewImports` is not inherited from the root, and `_ViewStart` in the area controls the layout .

---

#### Example 3: Area‑Aware Link Generation

```cshtml
@* Inside a view in the Products area *@
<a asp-controller="Home" asp-action="About">Products/Home/About</a>
@* Generates: /Products/Home/About (area ambient value preserved) *@

<a asp-area="" asp-controller="Home" asp-action="About">/Home/About</a>
@* Generates: /Home/About (area explicitly cleared) *@
```

**Step‑by‑step explanation:**

1. The first link omits `asp-area`, so the current area (`Products`) is used.
2. The second link sets `asp-area=""`, clearing the area and generating a root‑level URL.

**Expected Output:**

| Tag Helper | Generated URL |
|------------|---------------|
| `<a asp-controller="Home" asp-action="About">` | `/Products/Home/About` |
| `<a asp-area="" asp-controller="Home" asp-action="About">` | `/Home/About` |

**Why this output?** Area route values are ambient; `asp-area=""` overrides them .

### Real‑World Cases

- **E‑commerce platform:** Areas for `Admin`, `Shop`, and `Checkout`, with area routes registered first and `_ViewImports` in each area.
- **Modular monolith:** Each area has its own `_ViewStart` and `_Layout` for distinct branding, while shared Tag Helpers are imported via area‑specific `_ViewImports`.
- **Migration project:** Replacing `AreaRegistration` classes with `[Area]` attributes and `MapAreaControllerRoute` calls, ensuring route order is correct.

### References

- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- Routing to controller actions: Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#areas
- `MapAreaControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapareacontrollerroute
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Importing Shared Directives – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Anchor Tag Helper (`asp-area`) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.anchortaghelper.area
- Legacy `AreaRegistration` (ASP.NET MVC 5) – https://learn.microsoft.com/en-us/dotnet/api/system.web.mvc.arearegistration

---

## Summary Table: Areas at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Areas** | Partition app into functional groups | `[Area("Name")]`, folder structure | Large apps with distinct business modules |
| **Area Registration** | Associate controllers and routes with areas | `[Area("Name")]`, `MapAreaControllerRoute` | Every area must be registered |
| **Area Routing** | Match URLs to area controllers | `{area:exists}/...`, `MapAreaControllerRoute` | URL prefixes for functional modules |
| **Area Views** | Area‑specific UI files | `/Areas/<Name>/Views/...`, `_ViewStart`, `_ViewImports` | Any area with Razor views |
| **Best Practices** | Correct, maintainable area usage | Order routes first, add `_ViewImports`, use `asp-area=""` | All production applications |

---

## References (Full List)

- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas
- Areas in ASP.NET Core: Area folder structure – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#area-folder-structure
- Areas in ASP.NET Core: Associate the controller with an Area – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#associate-the-controller-with-an-area
- Areas in ASP.NET Core: Add Area route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#add-area-route
- Areas in ASP.NET Core: Link generation with MVC areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#link-generation-with-mvc-areas
- Areas in ASP.NET Core: Shared layout for Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#shared-layout-for-areas-using-the-_viewstartcshtml-file
- Areas in ASP.NET Core: `_ViewImports` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#viewimportscshtml
- Areas in ASP.NET Core: Change default area folder – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#change-default-area-folder-where-views-are-stored
- Areas in ASP.NET Core: Areas with Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas#areas-with-razor-pages
- Routing to controller actions in ASP.NET Core: Areas – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#areas
- `MapAreaControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapareacontrollerroute
- `AnchorTagHelper.Area` Property (`asp-area`) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.anchortaghelper.area
- `FormTagHelper.Area` Property (`asp-area`) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.formtaghelper.area
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Importing Shared Directives – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Legacy `AreaRegistration` (ASP.NET MVC 5) – https://learn.microsoft.com/en-us/dotnet/api/system.web.mvc.arearegistration
- Legacy `AreaRegistrationContext` (ASP.NET MVC 5) – https://learn.microsoft.com/en-us/dotnet/api/system.web.mvc.arearegistrationcontext
- Sample code (Areas) – https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/areas/samples