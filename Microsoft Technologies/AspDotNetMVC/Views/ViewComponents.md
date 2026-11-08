# View Components in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). View components are a stable feature with no breaking changes across these versions. The **`ViewComponent` base class**, **`Invoke`/`InvokeAsync` methods**, the **`@await Component.InvokeAsync(...)` syntax**, and the **`vc:` Tag Helper** are the primary mechanisms. View components are **not** controllers and do **not** participate in the controller lifecycle (filters do not apply). When considering view components, Microsoft also recommends evaluating **Razor components** for client-side UI logic and composition.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A view component is a **reusable UI unit** that combines C# logic with Razor markup, rendering a chunk of HTML (rather than a whole response) and is invoked from a view or layout. |
| **Technical Definition** | A view component is a public, non-nested, non-abstract class that either derives from `ViewComponent`, is decorated with `[ViewComponent]`, or has a name ending with the `ViewComponent` suffix. It defines its logic in an **`InvokeAsync` method returning `Task<IViewComponentResult>`** or a **synchronous `Invoke` method returning `IViewComponentResult`**. Parameters come directly from the invocation, **not** from model binding. View components support **constructor dependency injection**, do **not** participate in the controller lifecycle (filters do not apply), and are typically invoked from a layout or view using `@await Component.InvokeAsync(...)` or the `vc:` Tag Helper. |
| **Beginner-Friendly Explanation** | A view component is a **self-contained widget** for your UI. Unlike a partial view (which is just markup), a view component has **both logic and markup**—it can query a database, call a service, and render the result. Think of it as a mini-controller paired with a mini-view that you can drop anywhere: a shopping cart summary, a login panel, a dynamic menu, or a sidebar. |

### Key Characteristics

- **Logic + markup** – Unlike partial views, view components contain C# logic (database queries, service calls) and render their own markup.
- **No model binding** – Parameters are passed directly at invocation time, not bound from HTTP requests.
- **Not HTTP endpoints** – View components are never directly reachable via a URL; they are invoked from code (typically a view).
- **No controller lifecycle** – Filters do not apply to view components.
- **Constructor DI supported** – Services can be injected via the constructor.
- **Name derivation** – The component name is the class name with the `ViewComponent` suffix removed (or explicitly set via `Name`).
- **`[NonViewComponent]` opt-out** – Prevents a class with the `ViewComponent` suffix from being treated as a view component.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- Basic understanding of **controllers**, **views**, **Razor syntax**, and **dependency injection**.
- Tag Helpers enabled (via `_ViewImports.cshtml`) for the `vc:` Tag Helper.

### Related Programming Areas

- **Partial Views** – Simpler alternative for markup-only reuse; view components add logic.
- **Razor Components (Blazor)** – Alternative for client-side UI logic and composition.
- **Tag Helpers** – The `vc:` Tag Helper invokes view components declaratively.
- **Dependency Injection** – View components support constructor injection.
- **Layouts** – View components are commonly invoked from layouts (e.g., login panel, cart summary).

### Core Concepts / Features

1. **View Components (General)** – The concept, role, and when to use them.
2. **`ViewComponent`** – The base class and class requirements.
3. **`Invoke()`** – The synchronous logic method.
4. **`InvokeAsync()`** – The asynchronous logic method (recommended).
5. **View Component Views** – View discovery, naming, and search paths.
6. **View Component Best Practices** – Guidelines for clean, performant, and maintainable view components.

---

## 1. View Components (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A view component is a reusable UI unit that renders a chunk of HTML and encapsulates its own logic, invoked from a view or layout. |
| **Technical Definition** | View components are similar to partial views but **much more powerful** because they can contain business logic. They are intended for reusable rendering logic that is **too complex for a partial view**—dynamic navigation menus, tag clouds that query the database, shopping carts, sign-in panels, recently published articles, and sidebar content. A view component consists of two parts: the **class** (typically deriving from `ViewComponent`) and the **result** it returns (typically a view). |
| **Beginner-Friendly Explanation** | A view component is a **smart widget**. A partial view is like a sticker—it just shows markup. A view component is like a mini-app—it can fetch data, apply logic, and then show markup. Use view components when the UI piece needs to **do something** (query a database, check user state), not just display static content. |

### Purposes (each begins with “To”)

- **To** encapsulate **reusable rendering logic** that is too complex for a partial view.
- **To** separate concerns between the logic (view component class) and the markup (view component view), mirroring the controller/view separation.
- **To** enable **testable** UI components by isolating logic in a dedicated class.
- **To** render **dynamic content** (menus, carts, sign-in panels) that depends on services or database queries.
- **To** provide a **cleaner alternative to HTML Helpers** for logic-bearing UI fragments.

### Syntax Rules and Structure

#### General Syntax (Class)

```csharp
public class PriorityList : ViewComponent
{
    private readonly ToDoContext _db;
    public PriorityList(ToDoContext db) => _db = db;

    public async Task<IViewComponentResult> InvokeAsync(int maxPriority, bool isDone)
    {
        var items = await GetItemsAsync(maxPriority, isDone);
        return View(items);
    }
}
```

#### General Syntax (Invocation in a View)

```cshtml
@await Component.InvokeAsync("PriorityList", new { maxPriority = 4, isDone = true })
```

Or using the `vc:` Tag Helper:

```cshtml
<vc:priority-list max-priority="4" is-done="true"></vc:priority-list>
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Class | Derives from `ViewComponent`, uses `[ViewComponent]`, or ends with `ViewComponent`. |
| `Invoke` / `InvokeAsync` | The logic method; returns `IViewComponentResult` or `Task<IViewComponentResult>`. |
| `View(...)` | Returns a `ViewViewComponentResult` that renders the associated view. |
| Invocation | `@await Component.InvokeAsync(...)` or `<vc:...>` Tag Helper. |
| View location | `/Views/{Controller}/Components/{Name}/Default.cshtml` or `/Views/Shared/Components/{Name}/Default.cshtml`. |

#### Syntax Rules

- **Class must be public, non-nested, and non-abstract**.
- **Name** – The component name is the class name minus the `ViewComponent` suffix (or explicitly set via `Name`).
- **`InvokeAsync` is preferred** – The asynchronous method returns `Task<IViewComponentResult>`.
- **No model binding** – Parameters come from the invocation, not from HTTP.
- **No HTTP endpoint** – View components are not directly routable.

#### Constraints and Limitations

- **No filters** – View components do not participate in the controller lifecycle; action filters do not apply.
- **Not for full responses** – View components render a chunk, not a whole page.
- **Consider Razor components** – For client-side UI logic and composition, Microsoft recommends evaluating Razor components.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating and Invoking a View Component
`ViewComponents/PriorityList.cs`
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

public class PriorityList : ViewComponent
{
    private readonly ToDoContext _db;
    public PriorityList(ToDoContext db) => _db = db;

    public async Task<IViewComponentResult> InvokeAsync(int maxPriority, bool isDone)
    {
        var items = await _db.ToDo
            .Where(x => x.IsDone == isDone && x.Priority <= maxPriority)
            .ToListAsync();
        return View(items);   // Renders Views/Shared/Components/PriorityList/Default.cshtml
    }
}
```

`Views/Shared/Components/PriorityList/Default.cshtml`
```cshtml
@model IEnumerable<TodoItem>

<h3>Priority Items</h3>
<ul>
    @foreach (var item in Model)
    {
        <li>@item.Name</li>
    }
</ul>
```

```cshtml
@* Invocation in any view or layout *@
@await Component.InvokeAsync("PriorityList", new { maxPriority = 4, isDone = true })
```

**Step-by-step explanation:**

1. The `PriorityList` class derives from `ViewComponent` and uses constructor injection for `ToDoContext`.
2. `InvokeAsync` queries the database for matching items and returns `View(items)`.
3. The default view is discovered at `Views/Shared/Components/PriorityList/Default.cshtml`.
4. The view is invoked with named parameters (`maxPriority`, `isDone`).

**Expected Output:** A list of priority todo items rendered wherever the invocation appears.

**Why this output?** The view component encapsulates both the query logic and the rendering, making it reusable across views and layouts.

---

#### Example 2: Invoking with `vc:` Tag Helper

```cshtml
@* Invocation using the Tag Helper *@
<vc:priority-list max-priority="4" is-done="true"></vc:priority-list>
```

**Expected Output:** The same rendered output as `@await Component.InvokeAsync(...)`.

**Why this output?** The `vc:` Tag Helper is a declarative alternative to `InvokeAsync`; parameters are passed as HTML attributes and mapped to PascalCase method parameters.

---

#### Example 3: Invoking from a Controller

```csharp
public IActionResult IndexVC(int maxPriority = 2, bool isDone = false)
{
    return ViewComponent("PriorityList", new { maxPriority, isDone });
}
```

**Expected Output:** The response is the rendered view component output, without a full page layout.

**Why this output?** `ViewComponent(...)` in a controller action returns a `ViewComponentResult` that renders the component directly, useful for AJAX or fragment responses.

### Real-World Cases

- **Dynamic navigation menus** – Render different menu items based on user roles or database-driven configuration.
- **Shopping cart summary** – Show cart item count and total on every page via a layout-invoked view component.
- **Sign-in panel** – Display sign-in or sign-out links depending on authentication state.
- **Tag cloud** – Query the database for popular tags and render a cloud.
- **Recently published articles** – Fetch and display the latest posts in a sidebar.

### References

- View components in ASP.NET Core (official docs) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components
- View components (GitHub source) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/view-components.md
- Razor components overview – https://learn.microsoft.com/en-us/aspnet/core/blazor/components/

---

## 2. `ViewComponent`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ViewComponent` is the base class for view components, providing helper methods (e.g., `View()`) and properties (e.g., `ViewData`, `User`) used by the component’s logic. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ViewComponent` is the abstract base class for view components. Deriving from it is **optional** (a POCO with `[ViewComponent]` or a `ViewComponent` suffix also works), but most developers take advantage of its methods and properties. It provides `View(...)` overloads that return `ViewViewComponentResult`, as well as access to `HttpContext`, `User`, `ViewData`, and `ViewBag`. |
| **Beginner-Friendly Explanation** | `ViewComponent` is the **base class that gives your widget superpowers**. By deriving from it, you get the `View()` method (to render markup), access to `ViewData`, and the current `User`—just like a controller, but for a UI fragment. |

### Purposes (each begins with “To”)

- **To** provide the `View()` method for rendering the component’s associated view.
- **To** expose `HttpContext`, `User`, `ViewData`, and `ViewBag` to the component’s logic.
- **To** offer a conventional base class that integrates with the framework’s discovery and execution pipeline.
- **To** support constructor dependency injection in the component class.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class MyComponent : ViewComponent
{
    public IViewComponentResult Invoke()
    {
        return View();   // Renders Default.cshtml
    }
}
```

#### Component Breakdown

| Member | Purpose |
|--------|---------|
| `View(...)` | Returns a `ViewViewComponentResult`; overloads accept view name, model. |
| `ViewData` / `ViewBag` | Pass data to the view. |
| `User` | The current `ClaimsPrincipal`. |
| `HttpContext` | The current HTTP context. |
| Constructor | Supports DI for services. |

#### Syntax Rules

- **Deriving from `ViewComponent` is optional** – you can use `[ViewComponent]` or the suffix convention instead.
- **Class must be public, non-nested, and non-abstract**.
- **Name derivation** – suffix removed unless `Name` is explicitly set.

#### Constraints and Limitations

- **No filters** – view components are not part of the controller lifecycle.
- **Not for full HTTP responses** – view components render UI fragments.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Deriving from `ViewComponent`

```csharp
public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;
    public CartSummaryViewComponent(ICartService cartService) => _cartService = cartService;

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var count = await _cartService.GetItemCountAsync();
        return View(count);   // Renders Default.cshtml with the count
    }
}
```

**Expected Output:** The cart item count is rendered via the default view.

**Why this output?** The `ViewComponent` base class provides the `View()` method and DI support for the service.

---

#### Example 2: Using `[ViewComponent]` Without Deriving

```csharp
[ViewComponent(Name = "CartSummary")]
public class CartWidget
{
    public IViewComponentResult Invoke() => new HtmlContentViewComponentResult(
        new HtmlString("<p>Cart is empty</p>"));
}
```

**Expected Output:** The component is discovered as `CartSummary` and returns an `HtmlContentViewComponentResult`.

**Why this output?** The `[ViewComponent]` attribute marks the class and sets the component name, even without inheriting from `ViewComponent`.

### Real-World Cases

- **`CartSummaryViewComponent`** – derives from `ViewComponent` to render cart count on every page.
- **`SignInViewComponent`** – uses `User` property to determine authentication state.
- **`MenuViewComponent`** – derives from `ViewComponent` and uses DI to fetch menu items.

### References

- ViewComponent class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewcomponent
- View components in ASP.NET Core (class requirements) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components

---

## 3. `Invoke()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Invoke()` is the **synchronous logic method** of a view component that returns an `IViewComponentResult`. |
| **Technical Definition** | `Invoke` is a public method on a view component that returns `IViewComponentResult`. It is the synchronous counterpart of `InvokeAsync`. Parameters come directly from the invocation (e.g., anonymous object properties passed to `Component.InvokeAsync`), **not** from model binding. |
| **Beginner-Friendly Explanation** | `Invoke()` is the **synchronous “do the work” method** for a view component. Use it when your logic doesn’t need `await`—for example, when fetching data from an in-memory cache or performing simple calculations. |

### Purposes (each begins with “To”)

- **To** define the synchronous logic of a view component.
- **To** return an `IViewComponentResult` (typically a view) synchronously.
- **To** support scenarios where no asynchronous operations are required.
- **To** provide a simple, readable entry point for view component execution.

### Syntax Rules and Structure

#### General Syntax

```csharp
public IViewComponentResult Invoke(int maxPriority, bool isDone)
{
    var items = GetItems(maxPriority, isDone);
    return View(items);
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Method name | `Invoke` (case-insensitive matching by the framework). |
| Return type | `IViewComponentResult`. |
| Parameters | Directly from invocation; no model binding. |

#### Syntax Rules

- **Public method** – must be public for the framework to find it.
- **Parameters match invocation** – parameter names correspond to the properties of the anonymous object passed to `Component.InvokeAsync` (or the `vc:` Tag Helper attributes).
- **Return `View(...)`** – renders the component’s view.

#### Constraints and Limitations

- **Synchronous** – blocks the thread; prefer `InvokeAsync` for I/O-bound work.
- **No filters** – view components are not part of the controller filter pipeline.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Synchronous Invoke

```csharp
public class GreetingViewComponent : ViewComponent
{
    public IViewComponentResult Invoke(string name)
    {
        var greeting = $"Hello, {name}!";
        return View((object)greeting);
    }
}
```

```cshtml
@await Component.InvokeAsync("Greeting", new { name = "Alice" })
```

**Expected Output:** A greeting message rendered by the component’s default view.

**Why this output?** `Invoke` receives `name` from the invocation and returns a view with the greeting model.

---

#### Example 2: Synchronous Data Fetch (In-Memory)

```csharp
public class StatsViewComponent : ViewComponent
{
    private readonly IStatsCache _cache;
    public StatsViewComponent(IStatsCache cache) => _cache = cache;

    public IViewComponentResult Invoke()
    {
        var stats = _cache.GetCurrent();   // synchronous
        return View(stats);
    }
}
```

**Expected Output:** Cached statistics rendered by the view.

**Why this output?** The synchronous `Invoke` is appropriate for in-memory/cached data without I/O.

### Real-World Cases

- **Static menu rendering** – menu items loaded from configuration or cache.
- **Simple badges** – rendering a count from in-memory data.
- **Localised greetings** – simple string formatting without I/O.

### References

- View component methods (`Invoke`/`InvokeAsync`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-component-methods
- View components in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components

---

## 4. `InvokeAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `InvokeAsync()` is the **asynchronous logic method** of a view component that returns a `Task<IViewComponentResult>`. |
| **Technical Definition** | `InvokeAsync` is a public method returning `Task<IViewComponentResult>`. It is the **recommended** method for view components with asynchronous operations (database queries, HTTP calls). Parameters come directly from the invocation, not from model binding. The asynchronous signature allows the request thread to be freed while I/O operations complete. |
| **Beginner-Friendly Explanation** | `InvokeAsync()` is the **async version** of `Invoke()`. Use it when your widget needs to talk to a database or an external service—the `await` keyword lets the server handle other requests while waiting, keeping your app responsive. |

### Purposes (each begins with “To”)

- **To** define asynchronous logic for a view component without blocking the request thread.
- **To** support database queries, HTTP calls, and other I/O-bound operations.
- **To** return a `Task<IViewComponentResult>` that the framework awaits.
- **To** enable scalable rendering of dynamic UI fragments under load.

### Syntax Rules and Structure

#### General Syntax

```csharp
public async Task<IViewComponentResult> InvokeAsync(int maxPriority, bool isDone)
{
    var items = await _db.ToDo
        .Where(x => x.IsDone == isDone && x.Priority <= maxPriority)
        .ToListAsync();
    return View(items);
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Method name | `InvokeAsync`. |
| Return type | `Task<IViewComponentResult>`. |
| `await` | Used for I/O-bound operations. |
| `View(...)` | Returns the view result. |

#### Syntax Rules

- **`async Task<IViewComponentResult>`** – the signature.
- **`await`** – used for asynchronous calls.
- **Parameters** – passed from the invocation (e.g., `new { maxPriority = 4, isDone = true }`).

#### Constraints and Limitations

- **`async void` is never appropriate** – view components must return a `Task` that can be awaited.
- **Do not use `.Result` or `.Wait()`** – causes sync-over-async and potential deadlocks.
- **No filters** – view components are outside the controller lifecycle.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Asynchronous Database Query

```csharp
public class PriorityList : ViewComponent
{
    private readonly ToDoContext _db;
    public PriorityList(ToDoContext db) => _db = db;

    public async Task<IViewComponentResult> InvokeAsync(int maxPriority, bool isDone)
    {
        var items = await _db.ToDo
            .Where(x => x.IsDone == isDone && x.Priority <= maxPriority)
            .ToListAsync();
        return View(items);
    }
}
```

```cshtml
@await Component.InvokeAsync("PriorityList", new { maxPriority = 4, isDone = true })
```

**Expected Output:** The priority todo list is rendered asynchronously.

**Why this output?** `InvokeAsync` frees the request thread during the database query; when the query completes, the view is rendered.

---

#### Example 2: Asynchronous Service Call

```csharp
public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;
    public CartSummaryViewComponent(ICartService cartService) => _cartService = cartService;

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var cart = await _cartService.GetCartAsync();
        return View(cart);
    }
}
```

**Expected Output:** The cart summary is rendered asynchronously from the service.

**Why this output?** The async method awaits the service call, keeping the thread free.

---

#### Example 3: Asynchronous with Custom View Name

```csharp
public async Task<IViewComponentResult> InvokeAsync(int maxPriority, bool isDone)
{
    string viewName = "Default";
    if (maxPriority > 3 && isDone)
        viewName = "PVC";   // Use an alternative view

    var items = await GetItemsAsync(maxPriority, isDone);
    return View(viewName, items);
}
```

**Expected Output:** The component renders the `PVC.cshtml` view when conditions match.

**Why this output?** The `View(viewName, model)` overload allows conditional view selection.

### Real-World Cases

- **Database-backed menus** – async query for menu items.
- **Cart summary** – async fetch of cart contents on every page.
- **Sign-in panel** – async check of authentication state.
- **Tag cloud** – async query for popular tags.

### References

- `InvokeAsync` method – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-component-methods
- View components in ASP.NET Core (async examples) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components

---

## 5. View Component Views

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View component views are Razor files (`.cshtml`) that are rendered by a view component’s `View()` method, discovered through a dedicated set of search paths. |
| **Technical Definition** | View component views are discovered by the Razor view engine using view component–specific search paths. The **default view name is `Default`** (i.e., `Default.cshtml`), but a different view name can be specified when calling `View(viewName, model)`. The runtime searches the following paths: `/Views/{ControllerName}/Components/{ViewComponentName}/{ViewName}.cshtml`, `/Views/Shared/Components/{ViewComponentName}/{ViewName}.cshtml`, `/Pages/Shared/Components/{ViewComponentName}/{ViewName}.cshtml`, and `/Areas/{AreaName}/Views/Shared/Components/{ViewComponentName}/{ViewName}.cshtml`. |
| **Beginner-Friendly Explanation** | View component views are **the HTML templates for your widgets**. By convention, you put them in `Views/Shared/Components/{ComponentName}/Default.cshtml`. If the component is controller-specific, you can put them in `Views/{Controller}/Components/{ComponentName}/`. The framework knows where to look. |

### Purposes (each begins with “To”)

- **To** provide the markup rendered by a view component.
- **To** enforce a consistent folder structure for component views across the application.
- **To** support controller-specific and shared component views via distinct search paths.
- **To** allow alternative views (beyond `Default.cshtml`) for different rendering scenarios.
- **To** enable area-specific component views for modular applications.

### Syntax Rules and Structure

#### Search Paths (Default Order)

```
1. /Views/{ControllerName}/Components/{ViewComponentName}/{ViewName}.cshtml
2. /Views/Shared/Components/{ViewComponentName}/{ViewName}.cshtml
3. /Pages/Shared/Components/{ViewComponentName}/{ViewName}.cshtml
4. /Areas/{AreaName}/Views/Shared/Components/{ViewComponentName}/{ViewName}.cshtml
```

#### Recommended Location

```
/Views/Shared/Components/{ViewComponentName}/Default.cshtml
```

#### Component Breakdown

| Path | Use Case |
|------|----------|
| `/Views/{Controller}/Components/...` | Controller-specific component view. |
| `/Views/Shared/Components/...` | Shared across all controllers (recommended for reusable components). |
| `/Pages/Shared/Components/...` | Razor Pages shared components. |
| `/Areas/{Area}/Views/Shared/Components/...` | Area-specific shared components. |

#### Syntax Rules

- **Default view name is `Default`** – `View()` without arguments renders `Default.cshtml`.
- **Custom view names** – `View("PVC", model)` renders `PVC.cshtml`.
- **View discovery follows the search paths above** – the first match wins.
- **Custom search paths** – can be configured via `RazorViewEngineOptions.ViewLocationFormats`.

#### Constraints and Limitations

- **Missing view causes runtime error** – `InvalidOperationException` lists searched locations.
- **Naming mismatch** – the component name is derived from the class (suffix removed) unless explicitly set.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default View Discovery

```cshtml
@* Views/Shared/Components/PriorityList/Default.cshtml *@
@model IEnumerable<TodoItem>

<h3>Priority Items</h3>
<ul>
    @foreach (var item in Model) { <li>@item.Name</li> }
</ul>
```

```csharp
// In PriorityList.cs:
return View(items);   // Renders Views/Shared/Components/PriorityList/Default.cshtml
```

**Expected Output:** The priority list is rendered using the default view.

**Why this output?** `View()` without a view name uses `Default.cshtml` and searches the component-specific paths.

---

#### Example 2: Controller-Specific Component View

```
Views/
├── ToDo/
│   └── Components/
│       └── PriorityList/
│           └── Default.cshtml    ← Used only from ToDoController
└── Shared/
    └── Components/
        └── PriorityList/
            └── Default.cshtml    ← Fallback for all other controllers
```

**Expected Output:** `ToDoController` renders the controller-specific view; other controllers fall back to the shared view.

**Why this output?** The search order prioritizes the controller-specific path before the shared path.

---

#### Example 3: Custom View Name

```csharp
return View("PVC", items);   // Renders PVC.cshtml instead of Default.cshtml
```

**Expected Output:** The `PVC.cshtml` view is rendered.

**Why this output?** The explicit view name overrides the default `Default` name.

---

#### Example 4: Missing View Error

```text
InvalidOperationException: The view 'Components/PriorityList/Default' wasn't found.
The following locations were searched:
/Views/ToDo/Components/PriorityList/Default.cshtml
/Views/Shared/Components/PriorityList/Default.cshtml
```

**Expected Output:** A detailed error listing all searched locations.

**Why this output?** The framework reports every location it checked, aiding debugging.

### Real-World Cases

- **Shared `Default.cshtml`** – reusable components like cart summary, sign-in panel, and menu.
- **Controller-specific views** – customised rendering for a specific controller’s context.
- **Area-specific views** – admin area uses a different cart view than the public site.
- **Alternative views** – `PVC.cshtml` for a “print view” or “compact view” of the same data.

### References

- View search path for view components – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-search-path
- Customising the view search path – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#customize-the-view-search-path
- View components in ASP.NET Core (views) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components

---

## 6. View Component Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View component best practices are guidelines for creating, invoking, and maintaining view components that are **reusable, performant, testable, and cleanly separated** from other concerns. |
| **Technical Definition** | Best practices include: preferring `InvokeAsync` over `Invoke`, naming views `Default.cshtml` and placing them in `Views/Shared/Components/{Name}/`, using the `vc:` Tag Helper for declarative invocation, avoiding hard-coded component names (use `nameof`), considering Razor components for client-side UI logic, and keeping view components focused on a single responsibility. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for view components. For example: use `InvokeAsync` (async is better), put your views in the standard `Components` folder, use the `vc:` tag in your markup, and don’t hard-code the component name—use `nameof()` so the compiler catches typos. |

### Purposes (each begins with “To”)

- **To** maximise **reusability** by following standard naming and location conventions.
- **To** ensure **asynchronous, non-blocking** execution by preferring `InvokeAsync`.
- **To** improve **readability** by using the `vc:` Tag Helper in views.
- **To** prevent **runtime errors** from hard-coded component names by using `nameof`.
- **To** keep view components **focused and testable** by isolating their logic.
- **To** choose the **right tool**: view components for logic-heavy UI, partial views for markup-only reuse, and Razor components for client-side interactivity.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Prefer `InvokeAsync` over `Invoke`** | Non-blocking; scalable for I/O. | `public async Task<IViewComponentResult> InvokeAsync(...)` |
| **Name the view `Default.cshtml`** | Framework default; standard convention. | `Views/Shared/Components/CartSummary/Default.cshtml` |
| **Place shared views in `Views/Shared/Components/`** | Reusable across controllers. | Shared component views. |
| **Use the `vc:` Tag Helper** | Declarative, readable HTML-like syntax. | `<vc:priority-list max-priority="4" is-done="true" />` |
| **Use `nameof` for component names** | Compile-time safety. | `@await Component.InvokeAsync(nameof(PriorityList), ...)` |
| **Avoid hard-coded strings** | Typos become runtime errors. | Use `nameof(PriorityList)` instead of `"PriorityList"`. |
| **Consider Razor components for client-side logic** | View components are server-rendered; Razor components offer richer client-side composition. | Use Razor components for interactive UI. |
| **Keep components focused** | One responsibility per component. | `CartSummary` only shows cart summary; not full checkout. |
| **Test the `InvokeAsync` logic** | The class is a POCO and easily testable. | Unit test `InvokeAsync` with mocked services. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended Pattern (Async + `nameof` + `vc:` Tag Helper)

```csharp
// ViewComponents/CartSummary.cs
public class CartSummary : ViewComponent
{
    private readonly ICartService _cartService;
    public CartSummary(ICartService cartService) => _cartService = cartService;

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var cart = await _cartService.GetCartAsync();
        return View(cart);   // Renders Default.cshtml
    }
}
```

```cshtml
@* Using the vc: Tag Helper (recommended) *@
<vc:cart-summary></vc:cart-summary>

@* Or using InvokeAsync with nameof *@
@await Component.InvokeAsync(nameof(CartSummary))
```

**Expected Output:** The cart summary is rendered asynchronously, with compile-time safety for the component name.

**Why this output?** `InvokeAsync` is non-blocking; `nameof` catches typos at compile time; the Tag Helper is the most readable invocation syntax.

---

#### Example 2: Avoiding Hard-Coded Strings

```csharp
// ❌ BAD: Hard-coded string
@await Component.InvokeAsync("PriorityList", new { maxPriority = 4, isDone = true })

// ✅ GOOD: nameof for compile-time safety
@await Component.InvokeAsync(nameof(PriorityList), new { maxPriority = 4, isDone = true })
```

**Expected Output:** The `nameof` version fails at compile time if the class is renamed; the hard-coded string fails at runtime.

**Why this output?** `nameof` provides compile-time safety and refactoring support.

---

#### Example 3: Choosing the Right Tool

```cshtml
@* ✅ View component: logic + markup (database query) *@
<vc:recent-articles count="5"></vc:recent-articles>

@* ✅ Partial view: markup only (static content) *@
<partial name="_Footer" />

@* ✅ Razor component: client-side interactivity (Blazor) *@
<Counter IncrementAmount="10" />
```

**Expected Output:** Each tool is used for its intended purpose: logic-heavy UI (view component), static fragments (partial), and interactive client-side UI (Razor component).

**Why this output?** View components are for server-side logic + markup; partials for markup reuse; Razor components for client-side composition.

### Real-World Cases

- **E-commerce** – `CartSummary` view component with `InvokeAsync`, `Default.cshtml` in `Shared/Components`, invoked via `<vc:cart-summary>`.
- **Blog** – `RecentArticles` view component querying the database, using `nameof` for invocation.
- **Admin dashboard** – `StatsWidget` view component with async service calls; tested in isolation.

### References

- View components best practices (recommended view name and path) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-search-path
- Avoid hard-coded strings (`nameof`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#avoid-hardcoded-strings
- Consider Razor components – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#consider-razor-components
- `InvokeAsync` preference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-component-methods

---

## Summary Table: View Components at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **View Components** | Reusable UI units with logic | Logic + markup; no model binding; not HTTP endpoints | Dynamic menus, carts, sign-in panels, tag clouds |
| **`ViewComponent`** | Base class for components | Provides `View()`, `ViewData`, `User`, DI support | Most view components (optional) |
| **`Invoke()`** | Synchronous logic method | Returns `IViewComponentResult` | In-memory/synchronous work |
| **`InvokeAsync()`** | Asynchronous logic method (recommended) | Returns `Task<IViewComponentResult>` | Database queries, HTTP calls, I/O |
| **View Component Views** | Render component markup | `Default.cshtml` in `Views/Shared/Components/{Name}/` | Every view component |
| **Best Practices** | Reusable, performant, safe | Async, `nameof`, `vc:` Tag Helper, standard paths, consider Razor components | All production view components |

---

## References (Full List)

- View components in ASP.NET Core (official docs) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components
- View components (GitHub source) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/view-components.md
- ViewComponent class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewcomponent
- View component methods (`Invoke`/`InvokeAsync`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-component-methods
- View search path for view components – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#view-search-path
- Customising the view search path – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#customize-the-view-search-path
- Avoid hard-coded strings (`nameof`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#avoid-hardcoded-strings
- Consider Razor components – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components#consider-razor-components
- View components in ASP.NET Core (Slovenian, .NET 10) – https://learn.microsoft.com/sl-si/aspnet/core/mvc/views/view-components?view=aspnetcore-10.0
- View components in ASP.NET Core (German, .NET 6) – https://learn.microsoft.com/de-de/aspnet/core/mvc/views/view-components?view=aspnetcore-6.0
- View components in ASP.NET Core (Chinese, .NET 11) – https://learn.microsoft.com/zh-hk/aspnet/core/mvc/views/view-components?view=aspnetcore-11.0
- Razor components overview – https://learn.microsoft.com/en-us/aspnet/core/blazor/components/
- Integrating ASP.NET Core Razor components with MVC or Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/blazor/components/prerendering-and-integration