# Tag Helpers in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Tag Helpers are enabled by default in MVC views via the `_ViewImports.cshtml` file (`@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`). Most built-in Tag Helpers reside in the `Microsoft.AspNetCore.Mvc.TagHelpers` namespace. The `asp-page` and `asp-page-handler` attributes are specific to **Razor Pages** but are covered for completeness. The `asp-fallback-*`, `asp-src-include`, and `asp-src-exclude` attributes belong to the `ScriptTagHelper` and `LinkTagHelper`. Custom Tag Helpers derive from the `TagHelper` base class and are discovered automatically from the assembly.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Tag Helper is a **server-side component** that attaches to HTML-like elements in Razor views, enabling C# code to create, update, or render HTML markup without breaking the HTML designer's workflow. |
| **Technical Definition** | Tag Helpers are classes implementing `ITagHelper` (typically deriving from `TagHelper`) that are discovered via `@addTagHelper` directives. They target specific HTML elements or attributes using `[HtmlTargetElement]` and execute their `Process` or `ProcessAsync` method during Razor rendering. Built-in Tag Helpers (e.g., `AnchorTagHelper`, `FormTagHelper`, `InputTagHelper`) reside in `Microsoft.AspNetCore.Mvc.TagHelpers` and provide features like URL generation, anti-forgery tokens, and model binding metadata. |
| **Beginner-Friendly Explanation** | Tag Helpers are **HTML attributes with superpowers**. Instead of writing messy C# code inside your HTML (like `@Html.ActionLink(...)`), you write natural-looking HTML like `<a asp-controller="Products" asp-action="Details">`. The `asp-` attributes are Tag Helpers—they tell the server to do something smart with that HTML tag before sending it to the browser. |

### Key Characteristics

- **HTML-friendly syntax** – Tag Helpers look like normal HTML attributes, making views readable for designers and front-end developers.
- **IntelliSense support** – Visual Studio provides IntelliSense for Tag Helper attributes.
- **Server-side rendering** – Tag Helpers execute on the server and produce plain HTML output.
- **Declarative** – They are applied as attributes in Razor markup, not as method calls.
- **Targeted** – Each Tag Helper targets specific elements or attributes via `[HtmlTargetElement]`.
- **Extensible** – Custom Tag Helpers can be authored by deriving from `TagHelper`.
- **Scoped** – Availability is controlled by `@addTagHelper`, `@removeTagHelper`, and the `!` opt-out character.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` in `_ViewImports.cshtml` (default in templates).
- Basic understanding of **Razor syntax**, **HTML**, and **C#**.

### Related Programming Areas

- **HTML Helpers** – The older method-based alternative; Tag Helpers do not replace all HTML Helpers.
- **Razor Syntax** – Tag Helpers are part of Razor markup.
- **Model Binding** – `asp-for` and `asp-items` integrate with model metadata.
- **Routing** – `asp-action`, `asp-controller`, `asp-route` use URL generation.
- **Anti-forgery** – `FormTagHelper` generates request verification tokens.
- **View Components** – Another UI composition mechanism.

### Core Concepts / Features

1. **Tag Helpers (General)** – The concept, role, and mechanisms.
2. **Built-in Tag Helpers** – The `asp-*` attributes provided by ASP.NET Core.
3. **Custom Tag Helpers** – Authoring your own Tag Helpers.
4. **`TagHelper`** – The base class for custom Tag Helpers.
5. **`[HtmlTargetElement]`** – The attribute that controls which elements a Tag Helper targets.
6. **`Process()`** – The synchronous execution method.
7. **`ProcessAsync()`** – The asynchronous execution method.
8. **Tag Helper Best Practices** – Guidelines for clean, maintainable Tag Helpers.

---

## 1. Tag Helpers (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Tag Helpers are server-side components that attach to HTML elements and attributes in Razor views to produce dynamic HTML output. |
| **Technical Definition** | A Tag Helper is a class implementing `ITagHelper` (typically deriving from `TagHelper`) that is registered via `@addTagHelper` and discovered by the Razor compiler. It declares one or more `[HtmlTargetElement]` attributes specifying the HTML elements or attributes it targets. During rendering, the Razor engine invokes the Tag Helper's `Process` or `ProcessAsync` method, which can modify the `TagHelperOutput` (the HTML being generated). Built-in Tag Helpers reside in `Microsoft.AspNetCore.Mvc.TagHelpers`. |
| **Beginner-Friendly Explanation** | Tag Helpers are **smart HTML attributes**. You write `<a asp-action="Details">` instead of `@Html.ActionLink(...)`, and the server figures out the correct URL. They make Razor views look like plain HTML while still running C# code behind the scenes. |

### Purposes (each begins with “To”)

- **To** provide an HTML-friendly syntax for server-side rendering, improving readability for designers.
- **To** reduce the "context switch" between HTML and C# in Razor views.
- **To** leverage IntelliSense and tooling support for `asp-` attributes.
- **To** encapsulate complex rendering logic (URL generation, anti-forgery, model metadata) in reusable components.
- **To** offer a modern alternative to HTML Helpers while complementing them.

### Syntax Rules and Structure

#### General Syntax (Built-in)

```cshtml
<a asp-controller="Products" asp-action="Details" asp-route-id="@Model.Id">Details</a>
```

#### General Syntax (Custom)

```csharp
[HtmlTargetElement("my-widget")]
public class MyWidgetTagHelper : TagHelper
{
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        // Modify output
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `asp-*` attributes | Built-in Tag Helper attributes (e.g., `asp-action`, `asp-for`). |
| `[HtmlTargetElement]` | Declares the target element/attribute for a custom Tag Helper. |
| `Process` / `ProcessAsync` | The execution methods for custom Tag Helpers. |
| `TagHelperContext` | Provides contextual information about the element being processed. |
| `TagHelperOutput` | Represents the HTML output being generated. |

#### Syntax Rules

- **Tag Helpers are enabled via `@addTagHelper`** – default in `_ViewImports.cshtml` for built-in helpers.
- **`asp-` prefix** is conventional for built-in Tag Helpers but not required for custom ones.
- **Tag Helpers can target elements (`<a>`, `<form>`) or attributes (`asp-for`)** .
- **Tag Helper scope** is controlled by `@addTagHelper`, `@removeTagHelper`, and the `!` opt-out character.

#### Constraints and Limitations

- **Not a 1:1 replacement for HTML Helpers** – some HTML Helpers have no Tag Helper equivalent.
- **Custom Tag Helpers require registration** – via `@addTagHelper` in `_ViewImports.cshtml` or per-view.
- **Tag Helper conflicts** – multiple Tag Helpers targeting the same element can conflict; `[HtmlTargetElement]` order and `!` opt-out help resolve this.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Built-in Tag Helper (Anchor)

```cshtml
<a asp-controller="Products" asp-action="Details" asp-route-id="5">View Product</a>
```

**Step-by-step explanation:**

1. `asp-controller` specifies the controller (`Products`).
2. `asp-action` specifies the action (`Details`).
3. `asp-route-id` adds `id = 5` to the route values.
4. The `AnchorTagHelper` generates the URL using routing.

**Expected Output:** `<a href="/Products/Details/5">View Product</a>`

**Why this output?** The Tag Helper replaces the `asp-*` attributes with a standard `href` attribute generated by URL routing.

---

#### Example 2: Custom Tag Helper (Conditional Rendering)

```csharp
[HtmlTargetElement(Attributes = nameof(Condition))]
public class ConditionTagHelper : TagHelper
{
    public bool Condition { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        if (!Condition)
        {
            output.SuppressOutput();
        }
    }
}
```

```cshtml
<div condition="false">This will not render.</div>
<div condition="true">This will render.</div>
```

**Expected Output:** Only the second `<div>` is rendered.

**Why this output?** The custom Tag Helper suppresses output when `Condition` is `false`.

---

#### Example 3: Using `ProcessAsync` for Asynchronous Logic

```csharp
[HtmlTargetElement("p")]
public class AutoLinkerHttpTagHelper : TagHelper
{
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        var childContent = await output.GetChildContentAsync();
        output.Content.SetHtmlContent(Regex.Replace(
            childContent.GetContent(),
            @"\b(?:https?://)(\S+)\b",
            "<a target=\"_blank\" href=\"$0\">$0</a>"));
    }
}
```

**Expected Output:** URLs inside `<p>` elements are automatically converted to clickable links.

**Why this output?** `ProcessAsync` retrieves child content and replaces URL patterns with anchor tags.

### Real-World Cases

- **Navigation links** – `asp-controller`, `asp-action`, `asp-route-*` generate correct URLs.
- **Forms** – `FormTagHelper` generates `action` attributes and anti-forgery tokens.
- **Model-bound inputs** – `asp-for` generates `id`, `name`, and validation attributes.
- **CDN fallback** – `asp-fallback-*` provides local fallbacks when CDNs are unavailable.
- **Cache busting** – `asp-append-version` appends a hash to static asset URLs.

### References

- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring
- Tag Helpers in forms in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 2. Built-in Tag Helpers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Built-in Tag Helpers are the `asp-*` attributes provided by ASP.NET Core MVC for common rendering tasks such as URL generation, form creation, model binding, validation, and asset management. |
| **Technical Definition** | Built-in Tag Helpers reside in `Microsoft.AspNetCore.Mvc.TagHelpers` and include `AnchorTagHelper`, `FormTagHelper`, `InputTagHelper`, `LabelTagHelper`, `SelectTagHelper`, `TextAreaTagHelper`, `ValidationMessageTagHelper`, `ValidationSummaryTagHelper`, `ScriptTagHelper`, `LinkTagHelper`, `ImageTagHelper`, `EnvironmentTagHelper`, `CacheTagHelper`, and `DistributedCacheTagHelper`. Each targets specific HTML elements or attributes via `[HtmlTargetElement]`. |
| **Beginner-Friendly Explanation** | Built-in Tag Helpers are the **ready-made `asp-` attributes** that come with ASP.NET Core. They handle common tasks like creating links, building forms, binding inputs to models, showing validation messages, and managing CSS/JS files. |

### Purposes (each begins with “To”)

- **To** provide ready-made solutions for common view rendering tasks.
- **To** generate correct URLs, form actions, and anti-forgery tokens automatically.
- **To** bind HTML inputs to model properties with validation metadata.
- **To** manage static assets (scripts, styles, images) with cache busting and CDN fallbacks.
- **To** reduce the amount of manual C# code in views.

### Syntax Rules and Structure

#### General Syntax

```cshtml
<a asp-action="Index">Home</a>
<form asp-controller="Account" asp-action="Login" method="post">
    <input asp-for="Email" />
    <span asp-validation-for="Email"></span>
</form>
```

#### Component Breakdown (Selected Built-in Helpers)

| Tag Helper | Target | Key Attributes |
|------------|--------|----------------|
| `AnchorTagHelper` | `<a>` | `asp-controller`, `asp-action`, `asp-route-*`, `asp-area` |
| `FormTagHelper` | `<form>` | `asp-controller`, `asp-action`, `asp-route`, `asp-antiforgery` |
| `InputTagHelper` | `<input>` | `asp-for` |
| `LabelTagHelper` | `<label>` | `asp-for` |
| `SelectTagHelper` | `<select>` | `asp-for`, `asp-items` |
| `ValidationMessageTagHelper` | `<span>` | `asp-validation-for` |
| `ValidationSummaryTagHelper` | `<div>` | `asp-validation-summary` |
| `ScriptTagHelper` | `<script>` | `asp-src-include`, `asp-src-exclude`, `asp-fallback-*` |
| `LinkTagHelper` | `<link>` | `asp-append-version`, `asp-fallback-*` |
| `ImageTagHelper` | `<img>` | `asp-append-version` |

#### Syntax Rules

- **`asp-` attributes are case-insensitive**.
- **Multiple Tag Helpers can target the same element** (e.g., `<form>` with `asp-controller`, `asp-action`, and `asp-antiforgery`).
- **Attributes without `asp-` prefix** are passed through to the rendered HTML.

#### Constraints and Limitations

- **Not all HTML Helpers have Tag Helper equivalents**.
- **Some Tag Helpers require specific target elements** (e.g., `asp-for` on `<input>`, `<select>`, `<textarea>`).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: AnchorTagHelper (`asp-action`, `asp-controller`, `asp-route-*`)

```cshtml
<a asp-controller="Products" asp-action="Details" asp-route-id="5">Details</a>
```

**Expected Output:** `<a href="/Products/Details/5">Details</a>`

**Why this output?** The `AnchorTagHelper` uses routing to generate the `href`.

---

#### Example 2: FormTagHelper (`asp-action`, `asp-controller`, `asp-antiforgery`)

```cshtml
<form asp-controller="Account" asp-action="Login" asp-antiforgery="true" method="post">
    <!-- inputs -->
</form>
```

**Expected Output:** The form `action` attribute is generated, and a hidden anti-forgery token input is included.

**Why this output?** `FormTagHelper` generates the `action` URL and a request verification token.

---

#### Example 3: InputTagHelper (`asp-for`)

```cshtml
<input asp-for="Email" />
```

**Expected Output:** `<input type="email" id="Email" name="Email" />` (with validation attributes derived from the model).

**Why this output?** `InputTagHelper` reads the model expression and generates `id`, `name`, and `type` attributes.

---

#### Example 4: SelectTagHelper (`asp-items`)

```cshtml
<select asp-for="CategoryId" asp-items="Model.Categories"></select>
```

**Expected Output:** A `<select>` populated with `<option>` elements from `Model.Categories`.

**Why this output?** `SelectTagHelper` binds to the model property and uses `asp-items` to render options.

---

#### Example 5: ValidationMessageTagHelper (`asp-validation-for`)

```cshtml
<span asp-validation-for="Email" class="text-danger"></span>
```

**Expected Output:** A validation message placeholder that displays errors for the `Email` property.

**Why this output?** `ValidationMessageTagHelper` displays model validation errors for the specified property.

---

#### Example 6: ScriptTagHelper (`asp-fallback-*`)

```cshtml
<script src="https://cdn.example.com/jquery.js"
        asp-fallback-src="~/lib/jquery/jquery.js"
        asp-fallback-test="window.jQuery"></script>
```

**Expected Output:** If the CDN script fails to load (checked by `asp-fallback-test`), the local fallback is used.

**Why this output?** `ScriptTagHelper` renders JavaScript that tests the CDN and loads the fallback if needed.

---

#### Example 7: ScriptTagHelper (`asp-src-include`, `asp-src-exclude`)

```cshtml
<script asp-src-include="~/js/**/*.js" asp-src-exclude="~/js/admin/*.js"></script>
```

**Expected Output:** Multiple `<script>` tags for all matching `.js` files, excluding admin scripts.

**Why this output?** `ScriptTagHelper` uses glob patterns to include/exclude files relative to the web root.

---

#### Example 8: ImageTagHelper (`asp-append-version`)

```cshtml
<img src="~/images/logo.png" asp-append-version="true" />
```

**Expected Output:** `<img src="/images/logo.png?v=abc123" />` (with a hash based on file content).

**Why this output?** `ImageTagHelper` appends a version hash for cache busting.

### Real-World Cases

- **Navigation menus** – `asp-controller`/`asp-action` for all links.
- **Login/registration forms** – `FormTagHelper` + `InputTagHelper` + validation helpers.
- **Dropdown lists** – `SelectTagHelper` with `asp-items`.
- **CDN with local fallback** – `ScriptTagHelper` / `LinkTagHelper` with `asp-fallback-*`.
- **Cache-busted assets** – `asp-append-version` on CSS/JS/image tags.

### References

- Tag Helpers in forms in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms
- Exploring .NET Core – ASP.NET Core MVC TagHelper (built-in helpers) – https://learn.microsoft.com/en-us/archive/technet-wiki/35978.exploring-net-core-asp-net-core-1-0-mvc-taghelper
- Script Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/script-tag-helper
- Link Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/link-tag-helper
- Image Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/image-tag-helper

---

## 3. Custom Tag Helpers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Custom Tag Helpers are user-defined classes that implement `ITagHelper` (typically deriving from `TagHelper`) to provide application-specific HTML transformations. |
| **Technical Definition** | A custom Tag Helper is a public class deriving from `TagHelper` (or implementing `ITagHelper`) that targets specific HTML elements or attributes via `[HtmlTargetElement]` and overrides `Process` or `ProcessAsync`. It is registered via `@addTagHelper` in `_ViewImports.cshtml` or per-view. Custom Tag Helpers can access `TagHelperContext` (contextual information) and modify `TagHelperOutput` (the HTML being generated). |
| **Beginner-Friendly Explanation** | Custom Tag Helpers are **your own smart HTML attributes**. If you have a recurring piece of server-side logic that you want to apply to certain HTML elements, you can write a Tag Helper for it. For example, you could create a `condition` attribute that removes an element if the condition is false. |

### Purposes (each begins with “To”)

- **To** encapsulate reusable server-side HTML transformations.
- **To** extend the built-in Tag Helper set with application-specific behaviour.
- **To** keep views clean and readable by moving logic into a dedicated class.
- **To** provide an alternative to HTML Helpers and partial views for certain scenarios.
- **To** leverage the Tag Helper execution pipeline (context, output, async support).

### Syntax Rules and Structure

#### General Syntax

```csharp
[HtmlTargetElement("element-name")]
public class MyCustomTagHelper : TagHelper
{
    public string MyAttribute { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        // Modify output
    }
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `[HtmlTargetElement]` | Specifies the target element (e.g., `"p"`, `"div"`) or attributes. |
| Properties | Bound to Tag Helper attributes in the view (e.g., `my-attribute="value"`). |
| `Process` / `ProcessAsync` | The execution methods. |
| `TagHelperContext` | Provides the `TagName`, attributes, and other contextual info. |
| `TagHelperOutput` | Represents the HTML element and content being generated. |

#### Syntax Rules

- **Class must be public and non-abstract**.
- **Class name conventionally ends with `TagHelper`** (best practice, not required).
- **`[HtmlTargetElement]` is required** to associate the Tag Helper with HTML elements or attributes.
- **Properties are PascalCase** in C# and kebab-case in views (e.g., `MyAttribute` → `my-attribute`).
- **Register the Tag Helper** via `@addTagHelper *, MyAssembly` in `_ViewImports.cshtml`.

#### Constraints and Limitations

- **Tag Helper conflicts** – two Tag Helpers targeting the same element with the same attribute can conflict; use `[HtmlTargetElement]` attributes to disambiguate.
- **`ProcessAsync` should be used for I/O-bound work**.
- **Custom Tag Helpers are discovered per assembly** – ensure the assembly is referenced and registered.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Condition Tag Helper (Suppress Output)

```csharp
[HtmlTargetElement(Attributes = nameof(Condition))]
public class ConditionTagHelper : TagHelper
{
    public bool Condition { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        if (!Condition)
        {
            output.SuppressOutput();
        }
    }
}
```

```cshtml
<div condition="false">This will not render.</div>
<div condition="true">This will render.</div>
```

**Expected Output:** Only the second `<div>` is rendered.

**Why this output?** The Tag Helper suppresses output when `Condition` is `false`.

---

#### Example 2: Auto-Linker Tag Helper (Async)

```csharp
[HtmlTargetElement("p")]
public class AutoLinkerHttpTagHelper : TagHelper
{
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        var childContent = await output.GetChildContentAsync();
        output.Content.SetHtmlContent(Regex.Replace(
            childContent.GetContent(),
            @"\b(?:https?://)(\S+)\b",
            "<a target=\"_blank\" href=\"$0\">$0</a>"));
    }
}
```

**Expected Output:** URLs inside `<p>` elements become clickable links.

**Why this output?** `ProcessAsync` retrieves the child content and applies regex replacement.

---

#### Example 3: Email Tag Helper

```csharp
[HtmlTargetElement("email")]
public class EmailTagHelper : TagHelper
{
    public string Address { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        output.Attributes.SetAttribute("href", $"mailto:{Address}");
        output.Content.SetContent(Address);
    }
}
```

```cshtml
<email address="Support"></email>
```

**Expected Output:** `<a href="mailto:Support">Support</a>`

**Why this output?** The Tag Helper transforms the `<email>` element into an anchor with a `mailto:` link.

### Real-World Cases

- **Conditional rendering** – suppress elements based on authorization or business rules.
- **Custom widgets** – e.g., `<my-widget>` that renders a complex component.
- **Markdown processing** – convert Markdown content to HTML at render time.
- **Internationalisation** – a Tag Helper that localises content based on culture.

### References

- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring
- Tag Helper best practices (naming convention) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring#getting-started-with-tag-helpers

---

## 4. `TagHelper`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `TagHelper` is the abstract base class for custom Tag Helpers, providing the `Process` and `ProcessAsync` methods to implement and the `TagHelperOutput` mechanism to modify rendered HTML. |
| **Technical Definition** | `Microsoft.AspNetCore.Razor.TagHelpers.TagHelper` is an abstract class implementing `ITagHelper`. It defines virtual `Process` and `ProcessAsync` methods. Classes deriving from `TagHelper` are discovered by the Razor compiler when registered via `@addTagHelper`. The base class also provides the `Order` property for controlling execution order relative to other Tag Helpers. |
| **Beginner-Friendly Explanation** | `TagHelper` is the **base class you inherit from** when writing your own Tag Helper. It gives you the structure to override `Process` or `ProcessAsync` and modify the HTML output. |

### Purposes (each begins with “To”)

- **To** provide a standard base class for custom Tag Helpers.
- **To** define the `Process` and `ProcessAsync` methods that the Razor engine invokes.
- **To** supply the `TagHelperOutput` object for modifying HTML.
- **To** support the `Order` property for execution ordering.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class MyTagHelper : TagHelper
{
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        // ...
    }
}
```

#### Component Breakdown

| Member | Description |
|--------|-------------|
| `Process` | Synchronous execution method. |
| `ProcessAsync` | Asynchronous execution method (override one or both). |
| `Order` | Controls execution order relative to other Tag Helpers. |
| `TagHelperOutput` | The HTML element being generated. |

#### Syntax Rules

- **Override `Process` or `ProcessAsync`** (at least one).
- **`ProcessAsync` is preferred** for I/O-bound work.
- **`Order`** – lower values execute first.

#### Constraints and Limitations

- **Class must be public and non-abstract**.
- **Must be registered via `@addTagHelper`** to be discovered.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `TagHelper` Derivation

```csharp
public class HelloTagHelper : TagHelper
{
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.Content.SetContent("Hello, world!");
    }
}
```

**Expected Output:** The targeted element’s content is replaced with “Hello, world!”.

**Why this output?** `Process` sets the output content.

---

#### Example 2: Using `Order`

```csharp
public class FirstTagHelper : TagHelper
{
    public override int Order => 1;
    public override void Process(TagHelperContext context, TagHelperOutput output) { /* ... */ }
}

public class SecondTagHelper : TagHelper
{
    public override int Order => 2;
    public override void Process(TagHelperContext context, TagHelperOutput output) { /* ... */ }
}
```

**Expected Output:** `FirstTagHelper` executes before `SecondTagHelper`.

**Why this output?** Lower `Order` values execute first.

### Real-World Cases

- **Conditional rendering** – suppress output based on `Order`.
- **Custom widgets** – derive from `TagHelper` and modify output.
- **Asset management** – append versions or transform URLs.

### References

- `TagHelper` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper
- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring

---

## 5. `[HtmlTargetElement]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HtmlTargetElement]` is an attribute that specifies which HTML elements or attributes a custom Tag Helper targets. |
| **Technical Definition** | `Microsoft.AspNetCore.Razor.TagHelpers.HtmlTargetElementAttribute` is applied to a Tag Helper class (one or more times) to declare the element name and/or attribute names that the Tag Helper processes. It supports properties such as `Tag` (element name), `Attributes` (comma-separated attribute names), and `ParentTag`. |
| **Beginner-Friendly Explanation** | `[HtmlTargetElement]` is the **targeting rule** for a Tag Helper. It tells the framework: “This Tag Helper applies to `<p>` elements” or “this Tag Helper applies to elements with a `condition` attribute.” |

### Purposes (each begins with “To”)

- **To** associate a custom Tag Helper with specific HTML elements or attributes.
- **To** restrict a Tag Helper’s scope to avoid unintended transformations.
- **To** support multiple target declarations on a single Tag Helper.
- **To** disambiguate Tag Helper conflicts by targeting more specific elements/attributes.

### Syntax Rules and Structure

#### General Syntax

```csharp
[HtmlTargetElement("p")]
[HtmlTargetElement(Attributes = "condition")]
[HtmlTargetElement("div", Attributes = "my-attr")]
public class MyTagHelper : TagHelper { ... }
```

#### Component Breakdown

| Property | Description |
|----------|-------------|
| `Tag` (first positional parameter) | The HTML element name (e.g., `"p"`, `"div"`). |
| `Attributes` | Comma-separated attribute names the Tag Helper requires. |
| `ParentTag` | Restricts the Tag Helper to elements inside a specific parent element. |

#### Syntax Rules

- **Multiple `[HtmlTargetElement]` attributes** can be applied to one class.
- **`nameof` should be used** for attribute names to avoid magic strings and improve refactoring safety.
- **`Tag` and `Attributes` can be combined** for precise targeting.

#### Constraints and Limitations

- **Overly broad targeting** can cause unintended transformations; be specific.
- **Conflicting Tag Helpers** – if two Tag Helpers target the same element/attribute, use `Order` and `[HtmlTargetElement]` to resolve.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Target by Element

```csharp
[HtmlTargetElement("p")]
public class AutoLinkerHttpTagHelper : TagHelper { ... }
```

**Expected Output:** The Tag Helper applies only to `<p>` elements.

**Why this output?** The `Tag` parameter restricts targeting to `<p>`.

---

#### Example 2: Target by Attribute

```csharp
[HtmlTargetElement(Attributes = nameof(Condition))]
public class ConditionTagHelper : TagHelper
{
    public bool Condition { get; set; }
    // ...
}
```

**Expected Output:** The Tag Helper applies to any element with a `condition` attribute.

**Why this output?** The `Attributes` parameter targets the attribute, not the element.

---

#### Example 3: Combined Targeting

```csharp
[HtmlTargetElement("div", Attributes = "my-attr")]
public class MyDivTagHelper : TagHelper { ... }
```

**Expected Output:** The Tag Helper applies only to `<div>` elements that have a `my-attr` attribute.

**Why this output?** Both `Tag` and `Attributes` are specified.

### Real-World Cases

- **Conditional rendering** – target any element with a `condition` attribute.
- **Email helper** – target `<email>` elements.
- **Markdown helper** – target `<markdown>` elements.

### References

- `HtmlTargetElementAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.htmltargetelementattribute
- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring

---

## 6. `Process()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Process()` is the synchronous method in a Tag Helper that contains the logic to modify the HTML output. |
| **Technical Definition** | `public override void Process(TagHelperContext context, TagHelperOutput output)` is the synchronous execution method. It is called by the Razor engine during rendering. Implementations modify `output` to change the generated HTML. |
| **Beginner-Friendly Explanation** | `Process()` is the **“do the work” method** for a Tag Helper. It runs synchronously and is where you write the code that changes the HTML element. |

### Purposes (each begins with “To”)

- **To** implement the synchronous logic of a custom Tag Helper.
- **To** modify the `TagHelperOutput` (tag name, attributes, content) before rendering.
- **To** provide a simple entry point for Tag Helpers that do not require async operations.

### Syntax Rules and Structure

#### General Syntax

```csharp
public override void Process(TagHelperContext context, TagHelperOutput output)
{
    // Modify output
}
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `context` | Provides contextual information (`TagName`, `AllAttributes`, `Items`). |
| `output` | The HTML element and content being generated. |

#### Syntax Rules

- **Must be overridden** in a custom Tag Helper (or `ProcessAsync`).
- **Return type is `void`**.
- **Modifications to `output`** are reflected in the rendered HTML.

#### Constraints and Limitations

- **Synchronous** – blocks the thread; prefer `ProcessAsync` for I/O-bound work.
- **Do not perform long-running operations** – use `ProcessAsync` instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Modifying Tag Name

```csharp
public override void Process(TagHelperContext context, TagHelperOutput output)
{
    output.TagName = "a";
}
```

**Expected Output:** The element is rendered as an `<a>` tag instead of its original tag.

**Why this output?** `TagName` is changed in `Process`.

---

#### Example 2: Setting Content

```csharp
public override void Process(TagHelperContext context, TagHelperOutput output)
{
    output.Content.SetContent("New content");
}
```

**Expected Output:** The element’s content is replaced with “New content”.

**Why this output?** `Content.SetContent` sets the inner HTML.

---

#### Example 3: Suppressing Output

```csharp
public override void Process(TagHelperContext context, TagHelperOutput output)
{
    if (!Condition)
    {
        output.SuppressOutput();
    }
}
```

**Expected Output:** The element is not rendered when `Condition` is `false`.

**Why this output?** `SuppressOutput` removes the element from the output.

### Real-World Cases

- **Transform elements** – change `<email>` to `<a>`.
- **Conditional rendering** – suppress elements based on business rules.
- **Content replacement** – inject server-side content.

### References

- `TagHelper.Process` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.process
- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring

---

## 7. `ProcessAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ProcessAsync()` is the asynchronous method in a Tag Helper that contains the logic to modify the HTML output without blocking the thread. |
| **Technical Definition** | `public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)` is the asynchronous execution method. It is preferred for I/O-bound work such as database queries or HTTP calls. It returns a `Task` that the Razor engine awaits. |
| **Beginner-Friendly Explanation** | `ProcessAsync()` is the **async version of `Process()`**. Use it when your Tag Helper needs to do something slow (like fetch data) without blocking the server. |

### Purposes (each begins with “To”)

- **To** implement asynchronous logic in a custom Tag Helper.
- **To** avoid blocking the request thread during I/O-bound operations.
- **To** retrieve child content asynchronously (`await output.GetChildContentAsync()`).
- **To** perform async transformations before rendering.

### Syntax Rules and Structure

#### General Syntax

```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    var childContent = await output.GetChildContentAsync();
    // ...
}
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `context` | Contextual information. |
| `output` | The HTML output being generated. |
| `GetChildContentAsync()` | Asynchronously retrieves the element’s child content. |

#### Syntax Rules

- **Return type is `Task`**.
- **Use `await`** for async operations.
- **`GetChildContentAsync()`** is essential for content-based transformations.

#### Constraints and Limitations

- **`async void` is never appropriate** – must return `Task`.
- **Do not use `.Result` or `.Wait()`** – causes deadlocks.
- **Use `ProcessAsync` when the Tag Helper needs to await I/O**.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Auto-Linker (Async)

```csharp
[HtmlTargetElement("p")]
public class AutoLinkerHttpTagHelper : TagHelper
{
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        var childContent = await output.GetChildContentAsync();
        output.Content.SetHtmlContent(Regex.Replace(
            childContent.GetContent(),
            @"\b(?:https?://)(\S+)\b",
            "<a target=\"_blank\" href=\"$0\">$0</a>"));
    }
}
```

**Expected Output:** URLs in `<p>` elements become anchor tags.

**Why this output?** `ProcessAsync` retrieves child content asynchronously and applies regex replacement.

---

#### Example 2: Conditional Rendering with Async Check

```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    var isAllowed = await _authorizationService.IsAllowedAsync(User);
    if (!isAllowed)
    {
        output.SuppressOutput();
    }
}
```

**Expected Output:** The element is suppressed if authorization fails.

**Why this output?** The async authorization check determines whether to render the element.

### Real-World Cases

- **Authorization-based rendering** – check permissions asynchronously.
- **Database-driven content** – fetch data before rendering.
- **External API calls** – enrich HTML with remote data.

### References

- `TagHelper.ProcessAsync` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.processasync
- Author Tag Helpers in ASP.NET Core (async examples) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring

---

## 8. Tag Helper Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Tag Helper best practices are guidelines for authoring, registering, and using Tag Helpers that are **readable, maintainable, performant, and free of conflicts**. |
| **Technical Definition** | Best practices include: naming custom Tag Helper classes with the `TagHelper` suffix, using `nameof` for attribute targeting, preferring `ProcessAsync` for async work, avoiding overly broad targeting, using `Order` to resolve conflicts, scoping Tag Helpers via `@addTagHelper`/`@removeTagHelper`, and understanding the distinction between Tag Helpers and HTML Helpers. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for Tag Helpers. For example: name your classes `SomethingTagHelper`, use `nameof` instead of magic strings, and use `ProcessAsync` when you need to await something. |

### Purposes (each begins with “To”)

- **To** ensure Tag Helpers are **discoverable and predictable** through naming conventions.
- **To** prevent **runtime errors** from typos in attribute names by using `nameof`.
- **To** avoid **thread-blocking** by preferring `ProcessAsync` for I/O.
- **To** manage **scope** and **conflicts** through `@addTagHelper`/`@removeTagHelper` and `[HtmlTargetElement]`.
- **To** keep views **readable and maintainable** by using Tag Helpers where they excel.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Suffix custom Tag Helper classes with `TagHelper`** | Convention improves discoverability. | `ConditionTagHelper`, `EmailTagHelper` |
| **Use `nameof` for attribute targeting** | Compile-time safety and refactoring support. | `[HtmlTargetElement(Attributes = nameof(Condition))]` |
| **Prefer `ProcessAsync` for async operations** | Avoids blocking the request thread. | `public override async Task ProcessAsync(...)` |
| **Use `[HtmlTargetElement]` precisely** | Avoids unintended transformations and conflicts. | `[HtmlTargetElement("p")]` |
| **Use `Order` to resolve conflicts** | Controls execution order when multiple Tag Helpers target the same element. | `public override int Order => 1;` |
| **Scope Tag Helpers via `@addTagHelper` / `@removeTagHelper`** | Controls availability and avoids unexpected behaviour. | `@addTagHelper *, MyAssembly` |
| **Do not replace all HTML Helpers** | Some HTML Helpers have no Tag Helper equivalent. | Use HTML Helpers where appropriate. |
| **Keep Tag Helpers focused** | One responsibility per Tag Helper. | `ConditionTagHelper` only handles conditions. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended Naming and Targeting

```csharp
[HtmlTargetElement(Attributes = nameof(Condition))]
public class ConditionTagHelper : TagHelper
{
    public bool Condition { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        if (!Condition) output.SuppressOutput();
    }
}
```

**Expected Output:** The Tag Helper is discovered by convention and targets the `condition` attribute safely.

**Why this output?** The `TagHelper` suffix and `nameof` usage follow best practices.

---

#### Example 2: Using `ProcessAsync` for I/O

```csharp
public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
{
    var data = await _service.GetDataAsync();
    output.Content.SetContent(data);
}
```

**Expected Output:** The Tag Helper fetches data asynchronously and sets the content.

**Why this output?** `ProcessAsync` avoids blocking the thread during the service call.

---

#### Example 3: Scoping Tag Helpers

```cshtml
@* _ViewImports.cshtml *@
@addTagHelper *, MyApp.TagHelpers
@removeTagHelper *, SomeOtherAssembly
```

**Expected Output:** Only Tag Helpers from `MyApp.TagHelpers` are available; others are removed.

**Why this output?** `@addTagHelper` and `@removeTagHelper` control the Tag Helper scope.

### Real-World Cases

- **Custom widget libraries** – Tag Helpers packaged in a Razor Class Library.
- **Conditional rendering** – using `nameof` for attribute targeting.
- **Async authorization** – using `ProcessAsync` for permission checks.

### References

- Tag Helper best practices (naming convention) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring#getting-started-with-tag-helpers
- Avoid Tag Helper conflicts – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring#avoid-tag-helper-conflicts
- Managing Tag Helper scope – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro#managing-tag-helper-scope

---

## Summary Table: Tag Helpers at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Tag Helpers** | Server-side HTML attributes | HTML-friendly, IntelliSense-supported | Common rendering tasks |
| **Built-in Tag Helpers** | Ready-made `asp-*` attributes | Links, forms, validation, assets | Most views |
| **`asp-action` / `asp-controller`** | URL generation | Anchor/Form Tag Helpers | Links and form actions |
| **`asp-for`** | Model binding | Generates `id`, `name`, validation attrs | Input, label, select, textarea |
| **`asp-items`** | Populate select options | SelectTagHelper | Dropdown lists |
| **`asp-validation-for` / `asp-validation-summary`** | Validation messages | Validation Tag Helpers | Forms |
| **`asp-fallback-*`** | CDN fallback | ScriptTagHelper / LinkTagHelper | External assets |
| **`asp-append-version`** | Cache busting | ImageTagHelper / LinkTagHelper | Static assets |
| **`asp-src-include` / `asp-src-exclude`** | Glob asset inclusion | ScriptTagHelper | Bundling scripts |
| **Custom Tag Helpers** | Application-specific transformations | Derive from `TagHelper` | Reusable server-side logic |
| **`TagHelper`** | Base class | Override `Process` / `ProcessAsync` | Every custom Tag Helper |
| **`[HtmlTargetElement]`** | Targeting rule | Element/attribute targeting | Every custom Tag Helper |
| **`Process()` / `ProcessAsync()`** | Execution methods | Async preferred for I/O | Logic implementation |
| **Best Practices** | Clean, safe, maintainable | Suffix `TagHelper`, `nameof`, `ProcessAsync`, scope | All custom Tag Helpers |

---

## References (Full List)

- Tag Helpers in ASP.NET Core (official docs) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Author Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring
- Tag Helpers in forms in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms
- Exploring .NET Core – ASP.NET Core MVC TagHelper (built-in helpers) – https://learn.microsoft.com/en-us/archive/technet-wiki/35978.exploring-net-core-asp-net-core-1-0-mvc-taghelper
- Script Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/script-tag-helper
- Link Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/link-tag-helper
- Image Tag Helper in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/image-tag-helper
- `SelectTagHelper` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.selecttaghelper
- `TagHelper` class API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper
- `HtmlTargetElementAttribute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.htmltargetelementattribute
- `TagHelper.Process` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.process
- `TagHelper.ProcessAsync` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.processasync
- Managing Tag Helper scope – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro#managing-tag-helper-scope
- Avoid Tag Helper conflicts – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/authoring#avoid-tag-helper-conflicts