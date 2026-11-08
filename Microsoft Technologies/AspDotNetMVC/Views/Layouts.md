# Layouts in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Layouts are conceptually shared with **Razor Pages**; the primary difference is the folder (`Pages/Shared` vs. `Views/Shared`), which is noted where relevant. The `_ViewStart.cshtml`, `_ViewImports.cshtml`, `@RenderBody()`, `RenderSection`/`RenderSectionAsync`, and `@section` mechanisms have been stable across modern ASP.NET Core versions. **`RenderSectionAsync`** is recommended in async contexts; the synchronous `RenderSection` is still supported. Layouts are optional—apps can define zero, one, or multiple layouts.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A layout is a **shared Razor template** (`.cshtml`) that defines the common HTML skeleton—header, navigation, footer, scripts, styles—used by multiple views, which inject their unique content into designated placeholders. |
| **Technical Definition** | A layout is a `.cshtml` file (conventionally `_Layout.cshtml`) that acts as a **top-level template** for views. The layout calls `@RenderBody()` to render the child view's content and may call `@RenderSection("Name", required: false)` (or `@await RenderSectionAsync(...)`) to render named sections that views define via `@section Name { ... }`. Layout selection is controlled by the `Layout` property, typically set in `_ViewStart.cshtml` or overridden per view. Shared directives are imported via `_ViewImports.cshtml`. The Razor engine enforces that the body and every defined section are either rendered or explicitly ignored (`IgnoreBody()`, `IgnoreSection()`). |
| **Beginner-Friendly Explanation** | A layout is the **template or “picture frame”** for your pages. Every page on a website usually has the same header, navigation bar, and footer. Instead of copying that HTML into every single view, you put it once in a layout file. Each view then provides only its unique content, and the layout slots that content into the middle. This keeps your HTML DRY (Don't Repeat Yourself) and your site consistent. |

### Key Characteristics

- **Shared HTML skeleton** – Defines `<html>`, `<head>`, `<body>`, navigation, footer, and global CSS/JS references.
- **Content injection via `@RenderBody()`** – The mandatory placeholder for the child view's main content.
- **Named sections** – `@RenderSection` / `@section` allow views to inject page‑specific scripts, styles, or sidebar content.
- **Convention‑based naming** – The default layout is `_Layout.cshtml`, located in `Views/Shared/` (MVC) or `Pages/Shared/` (Razor Pages).
- **Hierarchical configuration** – `_ViewStart.cshtml` sets the default layout for all views in its folder and subfolders; `_ViewImports.cshtml` shares directives (e.g., `@using`, `@addTagHelper`).
- **Optional** – Apps are not required to use layouts; views can set `Layout = null` to opt out.
- **Multiple layouts** – Different sections of an app can use different layouts (e.g., `_AdminLayout.cshtml`, `_Layout.cshtml`).

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views.
- Basic understanding of **Razor syntax**, **views**, and **controllers**.

### Related Programming Areas

- **Views** – Layouts wrap views; views provide content.
- **Razor Syntax** – `@RenderBody()`, `@RenderSection`, `@section`.
- **View Discovery** – Layouts are discovered using the same search paths as partial views.
- **Tag Helpers** – `<partial>`, `asp-append-version`, environment tag helpers are commonly used in layouts.
- **Partial Views** – Reusable fragments (`_Navbar.cshtml`, `_Footer.cshtml`) often included in layouts.
- **Dependency Injection** – `@inject` in `_ViewImports.cshtml` makes services available to all views and layouts.

### Core Concepts / Features

1. **Layouts (General)** – The concept, role, and benefits.
2. **`_Layout.cshtml`** – The default layout file and its structure.
3. **`_ViewStart.cshtml`** – Sets the default layout before each view runs.
4. **`_ViewImports.cshtml`** – Shares directives across views and layouts.
5. **Layout Sections** – Named content regions for page‑specific assets.
6. **`@RenderBody()`** – The mandatory main‑content placeholder.
7. **`@RenderSection()` / `RenderSectionAsync`** – Rendering named sections in the layout.
8. **`@section`** – Defining section content in a view.
9. **Layout Best Practices** – Guidelines for clean, maintainable layouts.

---

## 1. Layouts (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A layout is a shared Razor template that provides a consistent visual and programmatic structure for multiple views. |
| **Technical Definition** | A layout is a `.cshtml` file whose `Layout` property is typically `null` (it is a top‑level template). It renders child view content via `@RenderBody()` and optional named sections. The Razor view engine resolves the layout specified by the view's `Layout` property (set in `_ViewStart` or in the view itself) using the same discovery process as partial views. The engine tracks whether the body and all sections have been rendered, throwing an exception if a defined section is not rendered and not explicitly ignored. |
| **Beginner-Friendly Explanation** | A layout is the **consistent frame** around your pages. Think of a website with a logo, menu, and footer on every page. The layout holds those pieces, and each view fills in only the unique middle part. This makes your site look consistent and saves you from writing the same HTML over and over. |

### Purposes (each begins with “To”)

- **To** eliminate duplicated HTML (headers, footers, navigation) across views.
- **To** enforce a consistent look and feel throughout the application.
- **To** centralise global assets (CSS, JavaScript) so they are loaded once per page.
- **To** provide named extension points (`@RenderSection`) for page‑specific content.
- **To** support multiple layouts for different areas of the app (e.g., admin vs. public).
- **To** enable shared configuration (`_ViewStart`, `_ViewImports`) that applies to all views.

### Syntax Rules and Structure

#### General Layout Structure

```cshtml
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/css/site.css" />
    @RenderSection("Styles", required: false)
</head>
<body>
    <header><!-- navigation --></header>

    <main>
        @RenderBody()
    </main>

    <footer><!-- footer --></footer>

    <script src="~/js/site.js"></script>
    @RenderSection("Scripts", required: false)
</body>
</html>
```

#### Syntax Rules

- **`@RenderBody()` is mandatory** in every layout (unless explicitly ignored).
- **Sections are optional**; `required: false` makes them optional.
- **Layout discovery** uses the same search paths as partial views; a partial name like `_Layout` searches the current folder first, then `Shared`.
- **`Layout = null`** in a view disables the layout for that view.

#### Constraints and Limitations

- **Only one `@RenderBody()` per layout**.
- **A defined section must be rendered or ignored**; otherwise, a runtime exception occurs.
- **Layouts cannot define sections** that they themselves render; sections are defined by views.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Layout with Body and Optional Section

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
</head>
<body>
    <header><h1>My Application</h1></header>

    <main>
        @RenderBody()
    </main>

    <footer><p>&copy; 2025</p></footer>

    @RenderSection("Scripts", required: false)
</body>
</html>
```

**Step-by-step explanation:**

1. `@RenderBody()` marks where each view's content will be inserted.
2. `@RenderSection("Scripts", required: false)` provides an optional slot for page‑specific scripts.
3. The header and footer are shared across all views using this layout.

**Expected Output:** Every view wrapped by this layout shares the header and footer, with its unique content in `<main>`.

**Why this output?** The layout defines the skeleton; the view supplies only the body content.

---

#### Example 2: View Using the Layout

```cshtml
@* Views/Home/Index.cshtml *@
@{
    ViewData["Title"] = "Home";
}

<h1>Welcome</h1>
<p>This is the home page content.</p>

@section Scripts {
    <script>console.log('Home page loaded');</script>
}
```

**Expected Output:** The home page renders inside `_Layout`, with the script injected at the `Scripts` section location.

**Why this output?** The view sets `ViewData["Title"]` and defines the `Scripts` section; the layout renders both.

---

#### Example 3: Disabling the Layout

```cshtml
@{
    Layout = null;
}
<!DOCTYPE html>
<html>
<head><title>Standalone Page</title></head>
<body><p>No layout applied.</p></body>
</html>
```

**Expected Output:** A bare HTML page with no shared header or footer.

**Why this output?** Setting `Layout = null` prevents the view from being wrapped by a layout.

### Real-World Cases

- **Public website** – consistent branding across all pages via `_Layout.cshtml`.
- **Admin panel** – a separate `_AdminLayout.cshtml` with a different navigation structure.
- **Email templates** – layouts for HTML emails (often without full `<html>` wrappers).
- **Print views** – minimal layouts for printer‑friendly pages.

### References

- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview

---

## 2. `_Layout.cshtml`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `_Layout.cshtml` is the **conventionally named default layout file** in an ASP.NET Core application, typically located in `Views/Shared/` (MVC) or `Pages/Shared/` (Razor Pages). |
| **Technical Definition** | `_Layout.cshtml` is a Razor template that serves as the top‑level layout for views that do not specify a custom layout. It contains the full HTML document structure, shared UI components, and calls to `@RenderBody()` and `@RenderSection`. The leading underscore (`_`) follows the convention for files that are not meant to be rendered as standalone views. |
| **Beginner-Friendly Explanation** | `_Layout.cshtml` is the **default master template** for your site. When you create a new MVC project, this file already contains the header, navigation, footer, and the `@RenderBody()` placeholder. Most views automatically use it, so you rarely have to write the shared HTML yourself. |

### Purposes (each begins with “To”)

- **To** provide a ready‑made shared template that new views automatically use.
- **To** define the standard HTML document structure (`<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`).
- **To** include global CSS and JavaScript references.
- **To** render the shared navigation, header, and footer.
- **To** serve as a baseline that can be customised or supplemented with additional layouts.

### Syntax Rules and Structure

#### Typical `_Layout.cshtml` (Template‑Generated)

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-light bg-white border-bottom mb-3">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">MyApp</a>
                <ul class="navbar-nav flex-grow-1">
                    <li class="nav-item"><a class="nav-link" asp-controller="Home" asp-action="Index">Home</a></li>
                    <li class="nav-item"><a class="nav-link" asp-controller="Home" asp-action="Privacy">Privacy</a></li>
                </ul>
            </div>
        </nav>
    </header>

    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2025 - MyApp - <a asp-controller="Home" asp-action="Privacy">Privacy</a>
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

#### Component Breakdown

| Component | Role |
|-----------|------|
| `<!DOCTYPE html>` … `<html>` | Document structure. |
| `<head>` | Meta tags, title, global CSS. |
| `@RenderSectionAsync("Styles", required: false)` | Optional page‑specific CSS. |
| `<header>` / `<nav>` | Shared navigation. |
| `@RenderBody()` | Child view content. |
| `<footer>` | Shared footer. |
| Global `<script>` tags | Shared JavaScript. |
| `@await RenderSectionAsync("Scripts", required: false)` | Optional page‑specific JS. |

#### Syntax Rules

- **`@RenderBody()` must be present** (unless `IgnoreBody()` is called).
- **`@RenderSectionAsync`** is recommended over `RenderSection` in async contexts.
- **The `_` prefix** indicates the file is not a standalone view.

#### Constraints and Limitations

- **Modifying the default layout affects all views** that use it; use multiple layouts for divergent structures.
- **Overly complex layouts** become hard to maintain; extract reusable pieces into partial views.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal `_Layout.cshtml`

```cshtml
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
</head>
<body>
    <header><h1>My Site</h1></header>
    <main>@RenderBody()</main>
    <footer><p>&copy; 2025</p></footer>
    @RenderSection("Scripts", required: false)
</body>
</html>
```

**Expected Output:** A consistent page frame for all views using this layout.

**Why this output?** The layout provides the minimum shared structure; views supply the body and optional scripts.

---

#### Example 2: Layout with Partial Views for Header/Footer

```cshtml
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
</head>
<body>
    <partial name="_Navbar" />
    <main class="container">
        @RenderBody()
    </main>
    <partial name="_Footer" />
    @RenderSection("Scripts", required: false)
</body>
</html>
```

**Expected Output:** The header and footer are rendered via reusable partial views.

**Why this output?** Partials keep the layout focused and allow the header/footer to be maintained independently.

### Real-World Cases

- **Template‑generated projects** – `_Layout.cshtml` comes pre‑configured with Bootstrap and navigation.
- **Custom branding** – editing `_Layout.cshtml` to add a logo, custom fonts, or analytics scripts.
- **Multi‑layout apps** – using `_Layout.cshtml` for public pages and `_AdminLayout.cshtml` for admin.

### References

- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Default layout in template projects – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#what-is-a-layout

---

## 3. `_ViewStart.cshtml`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `_ViewStart.cshtml` is a Razor file whose statements run **before every full view** (not layouts, not partial views) in its directory and subdirectories, commonly used to set the default layout. |
| **Technical Definition** | `_ViewStart.cshtml` is executed hierarchically: the root‑level `_ViewStart` (in `Views/` or `Pages/`) runs first, followed by any `_ViewStart` in the current view's folder path. The most common use is `@{ Layout = "_Layout"; }`, which sets the default layout for all views in scope. |
| **Beginner-Friendly Explanation** | `_ViewStart.cshtml` is the **“before every view” setup file**. If you want all views to use a particular layout, you write `Layout = "_Layout";` once in `_ViewStart.cshtml` instead of repeating it in every single view file. |

### Purposes (each begins with “To”)

- **To** set a default layout for all views in a folder and its subfolders, avoiding repetition.
- **To** run common initialisation code before each view renders.
- **To** enable hierarchical configuration: an area or subfolder can have its own `_ViewStart` that runs after the root one.
- **To** support different layouts for different sections of the app without per‑view declarations.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@{
    Layout = "_Layout";
}
```

#### Hierarchical Execution

```
Views/_ViewStart.cshtml               ← runs first
Views/Products/_ViewStart.cshtml      ← runs next (if present)
Views/Products/Index.cshtml           ← then the view runs
```

#### Syntax Rules

- **`_ViewStart.cshtml` is not a full view** – it has no `@model` and is not rendered as HTML.
- **It runs before full views**, but **not before layouts or partial views**.
- **Multiple `_ViewStart` files** execute from root to leaf; the last one wins for `Layout` assignment.
- **`_ViewStart` is typically placed in `Views/`** (or `Pages/`), not in `Shared/`.

#### Constraints and Limitations

- **Overuse** – only set `Layout` and simple shared values; complex logic belongs elsewhere.
- **Does not affect partial views** – partials do not execute `_ViewStart`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting a Default Layout

```cshtml
@* Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

**Expected Output:** Every full view in `Views/` and its subfolders uses `_Layout.cshtml` unless overridden.

**Why this output?** `_ViewStart` runs before each view and sets the `Layout` property.

---

#### Example 2: Hierarchical Override

```
Views/_ViewStart.cshtml:               Layout = "_Layout";
Views/Admin/_ViewStart.cshtml:        Layout = "_AdminLayout";
```

```cshtml
@* Views/Admin/Dashboard/Index.cshtml *@
@* No explicit Layout — inherits _AdminLayout from the nearest _ViewStart *@
<h1>Admin Dashboard</h1>
```

**Expected Output:** Admin views use `_AdminLayout`; other views use `_Layout`.

**Why this output?** The nearest `_ViewStart` (in `Views/Admin/`) runs after the root one and overrides `Layout`.

### Real-World Cases

- **Template‑generated apps** – `_ViewStart.cshtml` sets `Layout = "_Layout"` for all views.
- **Area‑specific layouts** – `Areas/Admin/Views/_ViewStart.cshtml` sets `_AdminLayout` for admin views.
- **Feature folders** – a `_ViewStart` in a feature folder applies a feature‑specific layout.

### References

- Running code before each view (`_ViewStart.cshtml`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#running-code-before-each-view
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 4. `_ViewImports.cshtml`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `_ViewImports.cshtml` is a Razor file that **imports shared directives** (e.g., `@using`, `@addTagHelper`, `@inject`) for all views and layouts in its folder and subfolders. |
| **Technical Definition** | `_ViewImports.cshtml` supports a specific set of Razor directives: `@addTagHelper`, `@removeTagHelper`, `@tagHelperPrefix`, `@using`, `@model`, `@inherits`, `@inject`, and `@namespace`. It is processed hierarchically: root‑level imports run first, then folder‑level imports. For additive directives (`@addTagHelper`, `@using`), all run; for override directives (`@model`, `@inherits`, `@tagHelperPrefix`), the closest file to the view wins. |
| **Beginner-Friendly Explanation** | `_ViewImports.cshtml` is the **shared imports file** for your views. Instead of writing `@using MyApp.Models` at the top of every view, you put it once in `_ViewImports.cshtml`, and every view in that folder can use those namespaces and Tag Helpers automatically. |

### Purposes (each begins with “To”)

- **To** avoid repeating `@using`, `@addTagHelper`, and `@inject` directives in every view.
- **To** make Tag Helpers available application‑wide (or folder‑wide) with a single `@addTagHelper` directive.
- **To** share common `@using` imports so views can reference models without fully qualified names.
- **To** inject services (e.g., `IViewLocalizer`, `IAuthorizationService`) into all views via `@inject`.
- **To** support hierarchical directive overriding (e.g., a subfolder can use a different `@model` base or Tag Helper prefix).

### Syntax Rules and Structure

#### General Syntax

```cshtml
@using MyApp
@using MyApp.Models
@using MyApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@inject IViewLocalizer Localizer
```

#### Component Breakdown

| Directive | Effect |
|-----------|--------|
| `@using` | Imports a namespace. |
| `@addTagHelper` | Registers Tag Helpers. |
| `@removeTagHelper` | Removes a Tag Helper. |
| `@tagHelperPrefix` | Sets a prefix for Tag Helpers. |
| `@model` | Sets the model type (rare in `_ViewImports`; overridden per view). |
| `@inherits` | Sets the base class for views. |
| `@inject` | Injects a service into all views. |
| `@namespace` | Sets the namespace for generated page classes (Razor Pages). |

#### Syntax Rules

- **`_ViewImports.cshtml` is hierarchical** – root‑level imports apply to all views; folder‑level imports apply only to that folder and its subfolders.
- **`@addTagHelper` and `@using` are additive** – all run; duplicates are ignored.
- **`@model`, `@inherits`, `@tagHelperPrefix`** – the closest file to the view overrides others.
- **`_ViewImports` is typically placed in `Views/`** (or `Pages/`), not `Shared/`.

#### Constraints and Limitations

- **`_ViewImports` does not support functions or section definitions**.
- **Over‑importing** – too many `@using` directives can slow down compilation and pollute IntelliSense.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Sharing `@using` and Tag Helpers

```cshtml
@* Views/_ViewImports.cshtml *@
@using MyApp
@using MyApp.Models
@using MyApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<Product>   @* No fully qualified name needed *@
```

**Expected Output:** The view can reference `Product` without `@using MyApp.Models`.

**Why this output?** `_ViewImports` imports the namespace for all views in scope.

---

#### Example 2: Injecting a Service into All Views

```cshtml
@* Views/_ViewImports.cshtml *@
@inject IAuthorizationService AuthorizationService
```

```cshtml
@* Any view *@
@if (await AuthorizationService.AuthorizeAsync(User, "AdminPolicy"))
{
    <a asp-action="Edit">Edit</a>
}
```

**Expected Output:** The `AuthorizationService` is available in every view.

**Why this output?** `@inject` in `_ViewImports` adds the service to all views in scope.

### Real-World Cases

- **Tag Helper registration** – `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` enables `asp-for`, `asp-action`, etc., application‑wide.
- **Shared model imports** – importing `MyApp.Models` so views can use short type names.
- **Service injection** – making `IViewLocalizer` or `IAuthorizationService` available in all views.

### References

- Importing shared directives (`_ViewImports.cshtml`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 5. Layout Sections

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Layout sections are **named content regions** defined in views (`@section Name { ... }`) and rendered by layouts (`@RenderSection("Name")` or `@await RenderSectionAsync("Name")`), allowing views to inject page‑specific content into designated places in the shared layout. |
| **Technical Definition** | Sections are a Razor feature for injecting content from a view into a layout. The layout declares one or more `@RenderSection` / `@RenderSectionAsync` calls, each with a name and an optional `required` flag. A view defines a matching section using `@section Name { ... }`. The Razor engine enforces that every defined section is either rendered or explicitly ignored (`IgnoreSection`). |
| **Beginner-Friendly Explanation** | Sections are **custom slots in the layout**. The layout says “I have a slot called `Scripts` at the bottom.” A view can then say “Here are the scripts for this page” by defining `@section Scripts { ... }`. Pages that don't need the slot simply omit it (if it's optional). |

### Purposes (each begins with “To”)

- **To** allow views to inject page‑specific content (scripts, styles, sidebar widgets) into a shared layout.
- **To** keep the layout generic while supporting page‑level customisation.
- **To** control exactly where page‑specific assets are rendered in the final HTML.
- **To** support optional sections (`required: false`) so views only provide content when needed.
- **To** enforce that critical sections (`required: true`) are always provided.

### Syntax Rules and Structure

#### Layout Definition

```cshtml
@RenderSection("Scripts", required: false)
@await RenderSectionAsync("Styles", required: false)
```

#### View Definition

```cshtml
@section Scripts {
    <script src="~/js/page.js"></script>
}

@section Styles {
    <link rel="stylesheet" href="~/css/page.css" />
}
```

#### Component Breakdown

| Component | Role |
|-----------|------|
| `@RenderSection("Name", required: false)` | Renders the named section; optional means the view may omit it. |
| `@await RenderSectionAsync("Name", required: false)` | Async version; recommended in async layouts. |
| `@section Name { ... }` | Defines the content for the section in the view. |

#### Syntax Rules

- **Section names are case‑sensitive**.
- **A layout can have multiple sections**.
- **A view can define multiple sections**.
- **A section defined in a view must be rendered or ignored**; otherwise, an exception occurs.
- **`required: true`** causes an exception if the view does not define the section.

#### Constraints and Limitations

- **Sections are not inherited** – a view can only define sections rendered by its immediate layout.
- **Sections cannot be nested**.
- **Overusing sections** makes layouts complex; extract reusable pieces into partial views.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Optional Scripts Section

```cshtml
@* Layout *@
<script src="~/js/global.js"></script>
@RenderSection("Scripts", required: false)
```

```cshtml
@* View *@
@section Scripts {
    <script src="~/js/page-specific.js"></script>
}
```

**Expected Output:** The page‑specific script is rendered after the global script.

**Why this output?** The layout renders the `Scripts` section at the designated location.

---

#### Example 2: Required Section

```cshtml
@* Layout *@
@RenderSection("Sidebar", required: true)
```

```cshtml
@* View *@
@section Sidebar {
    <aside>Sidebar content</aside>
}
```

**Expected Output:** The sidebar is always rendered; if the view omits the section, an exception is thrown.

**Why this output?** `required: true` enforces that the view must define the section.

---

#### Example 3: Multiple Sections

```cshtml
@* Layout *@
<head>
    @RenderSection("Styles", required: false)
</head>
<body>
    @RenderBody()
    @RenderSection("Scripts", required: false)
</body>
```

```cshtml
@* View *@
@section Styles {
    <link rel="stylesheet" href="~/css/special.css" />
}
@section Scripts {
    <script src="~/js/special.js"></script>
}
```

**Expected Output:** The stylesheet is placed in `<head>`; the script at the bottom of `<body>`.

**Why this output?** Each section is rendered at its corresponding placeholder in the layout.

### Real-World Cases

- **Page‑specific CSS/JS** – `@section Styles` and `@section Scripts` for per‑page assets.
- **Sidebar widgets** – `@section Sidebar` rendered in a layout column.
- **Meta tags** – `@section Meta` for page‑specific SEO tags.
- **Breadcrumbs** – `@section Breadcrumbs` rendered near the top of the page.

### References

- Sections in layouts – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections
- `RazorPage.RenderSectionAsync` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorpage.rendersectionasync

---

## 6. `@RenderBody()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@RenderBody()` is the **mandatory placeholder** in a layout where the child view's main content is rendered. |
| **Technical Definition** | `RenderBody()` is a method on `RazorPage` that renders the body of the child view. Every layout must call it (unless `IgnoreBody()` is called). The body corresponds to all markup in the view that is not inside a `@section` block. |
| **Beginner-Friendly Explanation** | `@RenderBody()` is the **main content slot** in the layout. Whatever you write in a view (outside of `@section` blocks) goes exactly where `@RenderBody()` is placed. It's the “meat” of the page inside the layout's frame. |

### Purposes (each begins with “To”)

- **To** inject the child view's main content into the layout's structure.
- **To** provide the single required content area in every layout.
- **To** separate the shared frame (layout) from the unique page content (view).

### Syntax Rules and Structure

#### General Syntax

```cshtml
<main>
    @RenderBody()
</main>
```

#### Syntax Rules

- **Exactly one `@RenderBody()` per layout**.
- **Must be present** unless `IgnoreBody()` is called.
- **Renders all view markup not enclosed in `@section` blocks**.

#### Constraints and Limitations

- **Cannot be omitted** without explicitly ignoring the body.
- **Cannot be called multiple times** in the same layout.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Body Rendering

```cshtml
@* Layout *@
<body>
    <header><h1>My App</h1></header>
    @RenderBody()
    <footer><p>&copy; 2025</p></footer>
</body>
```

```cshtml
@* View *@
<h2>Page Title</h2>
<p>Page content goes here.</p>
```

**Expected Output:** The view's `<h2>` and `<p>` appear between the header and footer.

**Why this output?** `@RenderBody()` inserts the view's non‑section markup at that location.

---

#### Example 2: Body with Surrounding Structure

```cshtml
<div class="container">
    <div class="row">
        <div class="col-md-8">
            @RenderBody()
        </div>
        <div class="col-md-4">
            @RenderSection("Sidebar", required: false)
        </div>
    </div>
</div>
```

**Expected Output:** The view's content occupies the main column; an optional sidebar can be rendered alongside.

**Why this output?** The body is placed inside the layout's grid; sections provide additional content slots.

### Real-World Cases

- **Every layout** – `@RenderBody()` is the core mechanism for injecting page content.
- **Two‑column layouts** – body in one column, sidebar section in another.
- **Dashboard layouts** – body for the main dashboard area, with optional panels in sections.

### References

- `@RenderBody()` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#rendering-the-body

---

## 7. `@RenderSection()` / `RenderSectionAsync`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RenderSection` (and its async counterpart `RenderSectionAsync`) is a method used in a layout to **render the content of a named section** defined by a view. |
| **Technical Definition** | `RenderSection("Name", required)` renders the section named `Name`. If `required` is `true` and the view does not define the section, an exception is thrown. `RenderSectionAsync` is the asynchronous equivalent, recommended in async layouts to avoid blocking. The method returns an `IHtmlContent` (or a `Task` thereof) that is written to the output. |
| **Beginner-Friendly Explanation** | `RenderSection` is the **“put the section content here”** instruction in the layout. The layout says “I'll render a section called `Scripts` here,” and the view fills that section with its scripts. |

### Purposes (each begins with “To”)

- **To** render page‑specific content (scripts, styles, widgets) at specific locations in the layout.
- **To** support optional sections so views only provide content when needed.
- **To** enforce required sections that must be provided by every view.
- **To** keep the layout flexible without hard‑coding page‑specific assets.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@RenderSection("Name", required: false)           @* synchronous *@
@await RenderSectionAsync("Name", required: false) @* asynchronous *@
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `"Name"` | The section name (case‑sensitive). |
| `required` | `true` → view must define the section; `false` → optional. |

#### Syntax Rules

- **`required: false`** makes the section optional; views may omit it.
- **`required: true`** causes an exception if the view omits the section.
- **`RenderSectionAsync`** should be used in async layouts.
- **A section can be rendered only once** per layout.

#### Constraints and Limitations

- **Required sections must be defined by every view** that uses the layout.
- **Sections cannot be nested**.
- **Rendering a non‑existent section** with `required: true` throws.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Optional Section

```cshtml
@* Layout *@
<footer>
    <p>&copy; 2025</p>
</footer>
@RenderSection("Scripts", required: false)
```

```cshtml
@* View *@
@section Scripts {
    <script src="~/js/page.js"></script>
}
```

**Expected Output:** The script is rendered after the footer.

**Why this output?** `RenderSection` outputs the section content at that location.

---

#### Example 2: Required Section

```cshtml
@* Layout *@
@RenderSection("Sidebar", required: true)
```

```cshtml
@* View A *@
@section Sidebar { <aside>Content A</aside> }

@* View B (omits Sidebar) *@
@* Runtime exception: section 'Sidebar' is required *@
```

**Expected Output:** View A renders the sidebar; View B throws an exception.

**Why this output?** `required: true` enforces that every view defines the section.

---

#### Example 3: Async Version

```cshtml
@* Layout *@
@await RenderSectionAsync("Scripts", required: false)
```

**Expected Output:** The section is rendered asynchronously.

**Why this output?** `RenderSectionAsync` returns a `Task` and is awaited; this is the recommended pattern in async layouts.

### Real-World Cases

- **Scripts section** – the most common use case; page‑specific JavaScript at the end of `<body>`.
- **Styles section** – page‑specific CSS in `<head>`.
- **Sidebar section** – optional sidebar content in two‑column layouts.
- **Meta section** – page‑specific meta tags for SEO.

### References

- `@RenderSection` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections
- `RazorPage.RenderSectionAsync` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorpage.rendersectionasync

---

## 8. `@section`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@section` is a Razor directive used in a view to **define the content of a named section** that will be rendered by the layout's `@RenderSection` or `@RenderSectionAsync` call. |
| **Technical Definition** | `@section Name { ... }` defines a section named `Name` with the enclosed Razor markup. When the layout renders that section, the content is emitted at the layout's `RenderSection` location. A view may define multiple sections; each must be rendered (or ignored) by the layout. |
| **Beginner-Friendly Explanation** | `@section` is how a view **fills in a slot** that the layout provides. If the layout has a slot called `Scripts`, the view says `@section Scripts { <script>...</script> }` to put its scripts into that slot. |

### Purposes (each begins with “To”)

- **To** provide page‑specific content for a named section in the layout.
- **To** keep page‑specific assets (scripts, styles) organised and separate from the main body.
- **To** support optional or required sections defined by the layout.
- **To** allow views to contribute content to multiple layout slots.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@section Name {
    @* Razor markup, HTML, C# *@
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `@section` | Directive keyword. |
| `Name` | Section name (case‑sensitive; must match layout's `RenderSection`). |
| `{ ... }` | Content block (markup and code). |

#### Syntax Rules

- **Section names must match** the layout's `RenderSection` names (case‑sensitive).
- **A view can define multiple sections**.
- **A section can contain markup, C#, and Tag Helpers**.
- **Sections cannot be nested**.

#### Constraints and Limitations

- **A view can define a section only once**.
- **If a view defines a section that the layout does not render** , a runtime exception may occur (depending on `IgnoreSection` usage).
- **Sections are not available in partial views** unless explicitly passed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Defining a Scripts Section

```cshtml
@* View *@
<h1>Page Content</h1>

@section Scripts {
    <script src="~/js/page-specific.js"></script>
    <script>console.log('Page loaded');</script>
}
```

**Expected Output:** The scripts are rendered at the layout's `Scripts` section location.

**Why this output?** The view defines the `Scripts` section; the layout renders it.

---

#### Example 2: Defining Multiple Sections

```cshtml
@section Styles {
    <link rel="stylesheet" href="~/css/page.css" />
}
@section Scripts {
    <script src="~/js/page.js"></script>
}
```

**Expected Output:** The stylesheet appears in `<head>`; the script appears at the bottom of `<body>`.

**Why this output?** Each section is placed at its corresponding `RenderSection` call in the layout.

---

#### Example 3: Using a Partial Inside a Section

```cshtml
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

**Expected Output:** The partial's content is rendered within the section.

**Why this output?** Sections can contain any Razor markup, including partials.

### Real-World Cases

- **Per‑page scripts** – `@section Scripts` for validation, charts, or interactive widgets.
- **Per‑page styles** – `@section Styles` for page‑specific CSS.
- **Sidebar widgets** – `@section Sidebar` for optional sidebar content.
- **SEO meta tags** – `@section Meta` for page‑specific meta tags.

### References

- `@section` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 9. Layout Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Layout best practices are guidelines for designing, organising, and maintaining layouts that are **clean, reusable, performant, and secure**. |
| **Technical Definition** | Best practices include: using `_ViewStart.cshtml` to set default layouts, keeping layouts focused on structure (not content), extracting reusable pieces into partial views, using `RenderSectionAsync` in async contexts, marking sections as optional unless they are truly required, avoiding business logic in layouts, loading global assets efficiently, and using multiple layouts for distinct areas (admin vs. public). |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep layouts tidy. For example: don't put page‑specific content in the layout, use `_ViewStart` instead of repeating `Layout = "_Layout"` in every view, and make sections optional unless every page must have them. |

### Purposes (each begins with “To”)

- **To** keep layouts **focused on shared structure**, not page‑specific content.
- **To** maximise **reuse** through partial views and multiple layouts.
- **To** avoid **repetition** by leveraging `_ViewStart.cshtml` and `_ViewImports.cshtml`.
- **To** improve **performance** by loading global assets efficiently and avoiding unnecessary sections.
- **To** maintain **security** by not exposing sensitive data in layouts.
- **To** support **maintainability** through clear separation of concerns and consistent conventions.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use `_ViewStart.cshtml` for default layouts** | Avoids repeating `Layout = "_Layout"` in every view. | `@{ Layout = "_Layout"; }` in `Views/_ViewStart.cshtml` |
| **Keep layouts structural** | Page‑specific content belongs in views/sections. | Navigation, header, footer only; no page‑specific data. |
| **Extract reusable pieces into partial views** | Keeps the layout clean and maintainable. | `<partial name="_Navbar" />` |
| **Use `RenderSectionAsync` in async layouts** | Avoids blocking; recommended pattern. | `@await RenderSectionAsync("Scripts", required: false)` |
| **Make sections optional unless required** | Reduces view boilerplate. | `required: false` for scripts/styles. |
| **Use multiple layouts for distinct areas** | Admin and public pages often need different structures. | `_Layout.cshtml`, `_AdminLayout.cshtml` |
| **Place `_ViewImports` and `_ViewStart` correctly** | Root‑level files apply application‑wide. | `Views/_ViewImports.cshtml`, `Views/_ViewStart.cshtml` |
| **Avoid business logic in layouts** | Layouts are presentation‑only. | No complex calculations or data access. |
| **Load scripts at the end of `<body>`** | Improves perceived performance. | Global JS before `@RenderSectionAsync("Scripts")`. |
| **Use `asp-append-version` for cache busting** | Ensures updated assets are fetched. | `<link href="~/css/site.css" asp-append-version="true" />` |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Well‑Structured Layout with Partials

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <partial name="_Navbar" />
    <main class="container">
        @RenderBody()
    </main>
    <partial name="_Footer" />
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**Expected Output:** A clean layout with reusable partials for navigation and footer.

**Why this output?** Partials keep the layout focused; sections provide page‑specific extension points.

---

#### Example 2: Using `_ViewStart` and Multiple Layouts

```cshtml
@* Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

```cshtml
@* Areas/Admin/Views/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

**Expected Output:** Public views use `_Layout`; admin views use `_AdminLayout`.

**Why this output?** Hierarchical `_ViewStart` files allow different default layouts for different sections.

---

#### Example 3: Optional Sections to Reduce Boilerplate

```cshtml
@* Layout *@
@await RenderSectionAsync("Scripts", required: false)
```

```cshtml
@* View without scripts — no error *@
<h1>Simple Page</h1>
```

**Expected Output:** The simple page renders without requiring a `Scripts` section.

**Why this output?** `required: false` makes the section optional, reducing unnecessary view code.

### Real-World Cases

- **E‑commerce** – `_Layout.cshtml` with partial `_Navbar`, `_Footer`, and optional `Scripts` section for page‑specific tracking.
- **Admin dashboard** – `_AdminLayout.cshtml` with a sidebar and admin‑only navigation.
- **Multi‑tenant apps** – separate layouts for different tenants or brands.
- **Email templates** – minimal layouts optimised for email clients.

### References

- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- `_ViewStart.cshtml` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#running-code-before-each-view
- `_ViewImports.cshtml` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Sections – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections
- `RenderSectionAsync` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorpage.rendersectionasync
- Partial views – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial

---

## Summary Table: Layouts at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Layouts** | Shared HTML skeleton | Consistent frame; DRY principle | Every multi‑page app |
| **`_Layout.cshtml`** | Default layout file | Conventionally named; in `Views/Shared/` | Template‑generated and custom apps |
| **`_ViewStart.cshtml`** | Set default layout before each view | `@{ Layout = "_Layout"; }` | Every MVC project |
| **`_ViewImports.cshtml`** | Share directives | `@using`, `@addTagHelper`, `@inject` | Every MVC project |
| **Layout Sections** | Named content slots | `@section` + `@RenderSection` | Page‑specific scripts, styles, widgets |
| **`@RenderBody()`** | Main content placeholder | Mandatory; one per layout | Every layout |
| **`@RenderSection()` / `RenderSectionAsync`** | Render named sections | `required: false` for optional | Every layout with sections |
| **`@section`** | Define section content in view | Matches layout's section name | When a view provides section content |
| **Best Practices** | Clean, reusable layouts | Partials, multiple layouts, async sections | All production layouts |

---

## References (Full List)

- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- `_ViewStart.cshtml` (Running code before each view) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#running-code-before-each-view
- `_ViewImports.cshtml` (Importing shared directives) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Sections – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections
- `@RenderBody()` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#rendering-the-body
- `RazorPage.RenderSectionAsync` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorpage.rendersectionasync
- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor