# ASP.NET Core MVC HTML Helpers: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (Razor views, `.cshtml`). HTML Helpers remain fully supported in ASP.NET Core, although **Tag Helpers** are the recommended approach for new form‑heavy views in many scenarios. However, HTML Helpers are **not deprecated**, and there is **no Tag Helper equivalent for every HTML Helper** — `Html.Raw`, `Html.Encode`, `Html.AntiForgeryToken` (explicit usage), and certain validation helpers are still commonly used.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HTML Helpers are server‑side methods used inside Razor views (`.cshtml`) to generate HTML markup programmatically, typically bound to the view’s model. |
| **Technical Definition** | An HTML Helper is an extension method on `IHtmlHelper` or `IHtmlHelper<TModel>`, located in the `Microsoft.AspNetCore.Mvc.Rendering` namespace. It returns `IHtmlContent` (not a plain string), which is written directly to the response without additional encoding. Strongly typed helpers (`*For`) use lambda expressions (`Expression<Func<TModel, TProperty>>`) to bind to model properties, enabling compile‑time checking, automatic `name`/`id` generation, and model‑state–aware value population. |
| **Beginner‑Friendly Explanation** | HTML Helpers are **shortcuts inside your Razor views** that write HTML for you. Instead of manually typing `<input type="text" name="Name" id="Name" value="..." />`, you write `@Html.TextBoxFor(m => m.Name)`, and ASP.NET Core fills in the correct `name`, `id`, `value`, and validation attributes based on your model. |

### Key Characteristics

- **Strongly typed (`*For` helpers)** – Lambda‑based binding prevents typos and enables refactoring support.
- **Model‑state aware** – Helpers automatically read from `ModelState` so that posted values are re‑displayed after validation failures.
- **Data‑annotation aware** – `DisplayNameFor`, `EditorFor`, `LabelFor`, and validation helpers respect `[Display]`, `[Required]`, `[StringLength]`, `[DataType]`, and other attributes.
- **Returns `IHtmlContent`** – Output is not double‑encoded; `Html.Raw` is only needed when you explicitly want to bypass encoding.
- **Tag Helper coexistence** – HTML Helpers and Tag Helpers can be mixed in the same view; neither replaces the other entirely.

### Prerequisites

- ASP.NET Core MVC project with Razor views (`.cshtml`).
- A strongly typed view (`@model MyViewModel`) for `*For` helpers.
- `@using Microsoft.AspNetCore.Mvc.Rendering` is **not required** in Razor views — `Html` is available by default as a property of `RazorPage<TModel>`.

### Related Programming Areas

- **Model binding & validation** – Helpers generate the `name` attributes that model binding relies on, and display validation messages from `ModelState`.
- **Tag Helpers** – The modern alternative for many form elements (`asp-for`, `asp-validation-for`, etc.).
- **Data annotations** – `[Display]`, `[Required]`, `[StringLength]`, `[DataType]`, `[HiddenInput]` directly influence helper output.
- **CSRF protection** – `@Html.AntiForgeryToken()` works together with `[ValidateAntiForgeryToken]`.
- **XSS prevention** – Razor encodes output by default; `Html.Raw` and `Html.Encode` control this behaviour.

### Core Concepts / Features

1. **HTML Helpers (General)** – The `IHtmlHelper` infrastructure.
2. **`@Html.ActionLink()`** – Generates an anchor (`<a>`) to a controller action.
3. **`@Html.DisplayFor()`** – Renders a read‑only display of a model property.
4. **`@Html.DisplayNameFor()`** – Renders the display name (label text) of a property.
5. **`@Html.EditorFor()`** – Renders an editor using the appropriate template for the property type.
6. **`@Html.LabelFor()`** – Renders a `<label>` with the correct `for` attribute.
7. **`@Html.TextBoxFor()`** – Renders a single‑line text input.
8. **`@Html.TextAreaFor()`** – Renders a multi‑line `<textarea>`.
9. **`@Html.CheckBoxFor()`** – Renders a boolean checkbox (plus a hidden fallback field).
10. **`@Html.RadioButtonFor()`** – Renders a radio button for a property.
11. **`@Html.DropDownListFor()`** – Renders a single‑selection `<select>`.
12. **`@Html.ListBoxFor()`** – Renders a multi‑selection `<select multiple>`.
13. **`@Html.HiddenFor()`** – Renders a hidden input.
14. **`@Html.PasswordFor()`** – Renders a password input (no value re‑population).
15. **`@Html.ValidationMessageFor()`** – Renders the validation message for a single property.
16. **`@Html.ValidationSummary()`** – Renders a summary of all validation errors.
17. **`@Html.BeginForm()` / `@Html.EndForm()`** – Renders `<form>` start and end tags.
18. **`@Html.AntiForgeryToken()`** – Renders the CSRF anti‑forgery token.
19. **`@Html.Raw()`** – Outputs a string without HTML encoding.
20. **`@Html.Encode()`** – HTML‑encodes a value.
21. **HTML Helper Best Practices** – Guidelines for safe, maintainable usage.

---

## 1. HTML Helpers (General Infrastructure)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HTML Helpers are extension methods on `IHtmlHelper` that generate HTML markup inside Razor views. |
| **Technical Definition** | `IHtmlHelper` and `IHtmlHelper<TModel>` are interfaces in `Microsoft.AspNetCore.Mvc.Rendering`. The concrete `HtmlHelper` / `HtmlHelper<TModel>` classes implement them. Extension methods (e.g., `DisplayFor`, `TextBoxFor`, `ValidationMessageFor`) are defined in static classes such as `DisplayNameExtensions`, `EditorExtensions`, `InputExtensions`, and `ValidationExtensions`. All helpers return `IHtmlContent`. |
| **Beginner‑Friendly Explanation** | Think of HTML Helpers as **pre‑built Razor functions** that save you from typing repetitive HTML. They know about your model, so they can fill in values, names, IDs, and validation rules automatically. |

### Purposes

- **To** reduce repetitive HTML markup in views.
- **To** bind HTML form elements to model properties automatically (names, IDs, values).
- **To** integrate model validation (data annotations and `ModelState`) into the rendered HTML.
- **To** provide compile‑time safety through strongly typed `*For` expressions.
- **To** enable consistent URL generation (`ActionLink`, `BeginForm`) using routing.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@Html.MethodName(expression, ...)          // strongly typed (*For)
@Html.MethodName("propertyName", ...)      // weakly typed (string-based)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `@Html` | The `IHtmlHelper<TModel>` instance available in every Razor view. |
| `MethodName` | The helper method (e.g., `TextBoxFor`, `DisplayFor`, `ActionLink`). |
| `expression` | A lambda expression: `m => m.PropertyName`. |
| `"propertyName"` | A string name used by weakly typed overloads. |

#### Syntax Rules

- `*For` helpers **require a strongly typed view** (`@model TModel`).
- `Html` is available by default; no `@using` is required in Razor views.
- Helpers return `IHtmlContent`; Razor writes the result directly — **do not** wrap them in `@Html.Raw()`.
- Weakly typed overloads (e.g., `Html.TextBox("Name")`) exist but are **not recommended** for new code because they lack compile‑time safety.

#### Constraints and Limitations

- HTML Helpers **do not replace Tag Helpers**; many form scenarios are better served by `asp-for` Tag Helpers in modern ASP.NET Core.
- `IHtmlHelper.Partial` (synchronous partial rendering) is flagged by analyzer **MVC1000** — use `PartialAsync` or the Partial Tag Helper instead.
- Custom HTML Helpers should return `IHtmlContent` and use `TagBuilder` rather than raw string concatenation.

### Annotated Code Example

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<Product>

<h2>Products</h2>
<ul>
    @foreach (var product in Model)
    {
        <li>
            @Html.DisplayFor(m => product.Name)   @* strongly typed *@
            — @Html.ActionLink("Details", "Details", new { id = product.Id })
        </li>
    }
</ul>
```

**Expected Output (conceptual):**

```html
<h2>Products</h2>
<ul>
    <li>Laptop — <a href="/Products/Details/1">Details</a></li>
    <li>Mouse — <a href="/Products/Details/2">Details</a></li>
</ul>
```

**Why this output?** `DisplayFor` reads the `Name` property value and encodes it. `ActionLink` uses routing to generate the `href`.

### Real‑World Cases

- **Admin dashboards:** Rapidly build CRUD views with `DisplayFor`, `EditorFor`, and `ValidationMessageFor`.
- **Mixed Tag Helper / HTML Helper codebases:** Legacy views using HTML Helpers continue to work alongside new Tag Helper–based views.
- **Explicit CSRF token placement:** `@Html.AntiForgeryToken()` is still needed when you are not using the Form Tag Helper.

### References

- IHtmlHelper<TModel> Interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper-1
- HtmlHelper Class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper
- Tag Helpers in forms – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 2. `@Html.ActionLink()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ActionLink` generates an HTML anchor (`<a>`) element whose `href` points to a controller action. |
| **Technical Definition** | `HtmlHelper.ActionLink(linkText, actionName, controllerName, routeValues, htmlAttributes)` uses `IUrlHelper.Action` to generate the URL. The returned `IHtmlContent` contains a complete `<a>` tag. |
| **Beginner‑Friendly Explanation** | `ActionLink` is a **shortcut for creating a link to another page in your app**. Instead of hard‑coding `/Products/Details/5`, you say “link to the Details action of Products with id 5,” and ASP.NET Core builds the correct URL. |

### Purposes

- **To** generate navigation links without hard‑coding URLs.
- **To** keep links consistent with the application’s routing configuration.
- **To** support route values (e.g., `id`) and HTML attributes (e.g., CSS classes).
- **To** enable refactoring safety: if routes change, `ActionLink` output updates automatically.

### Syntax Rules and Structure

```cshtml
@Html.ActionLink(linkText, actionName)
@Html.ActionLink(linkText, actionName, controllerName)
@Html.ActionLink(linkText, actionName, controllerName, routeValues, htmlAttributes)
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `linkText` | Visible text of the anchor (required, non‑null). |
| `actionName` | The action method name. |
| `controllerName` | The controller name (without `Controller` suffix). |
| `routeValues` | An anonymous object of route parameters (e.g., `new { id = 5 }`). |
| `htmlAttributes` | An anonymous object of HTML attributes (e.g., `new { @class = "btn" }`). Use `@` to escape reserved keywords. |

#### Syntax Rules

- At minimum, provide `linkText` and `actionName`.
- `controllerName` defaults to the **current controller** if omitted.
- Use `null` for `routeValues` or `htmlAttributes` if not needed.
- HTML attribute names that clash with C# keywords must be prefixed with `@` (e.g., `@class`).

#### Constraints and Limitations

- `ActionLink` generates a **relative URL** by default; for absolute URLs, use `Url.Action(..., protocol: "https")`.
- The link text is **HTML‑encoded** by default; do not pass trusted HTML.
- `ActionLink` depends on routing; if no route matches the supplied values, it may generate an empty `href` or throw.

### Annotated Code Examples

#### Example 1: Basic Link

```cshtml
@Html.ActionLink("View Details", "Details", "Products", new { id = 5 }, null)
```

**Expected Output:**

```html
<a href="/Products/Details/5">View Details</a>
```

**Why this output?** Routing matches `Products/Details/5` to the `Details` action of `ProductsController` with `id = 5`.

#### Example 2: Link with CSS Class

```cshtml
@Html.ActionLink("Delete", "Delete", "Products",
    new { id = item.Id },
    new { @class = "btn btn-danger", onclick = "return confirm('Are you sure?');" })
```

**Expected Output:**

```html
<a href="/Products/Delete/3" class="btn btn-danger" onclick="return confirm('Are you sure?');">Delete</a>
```

**Why this output?** The `htmlAttributes` object is translated into HTML attributes. `@class` escapes the C# keyword `class`.

### Real‑World Cases

- **Navigation menus:** `@Html.ActionLink("Home", "Index", "Home")`.
- **CRUD tables:** Row‑level “Edit” and “Delete” links with route values.
- **Breadcrumbs:** Links generated from route data without hard‑coding URLs.

### References

- HtmlHelper.ActionLink Method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.actionlink
- Routing to controller actions – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing

---

## 3. `@Html.DisplayFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `DisplayFor` renders a read‑only representation of a model property. |
| **Technical Definition** | `DisplayFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression)` uses the model metadata and display templates to render the property value. It respects `[DisplayFormat]`, `[Display]`, and `[DataType]` attributes. |
| **Beginner‑Friendly Explanation** | `DisplayFor` is used when you want to **show a value** (not edit it). It looks at your model’s data annotations and formats the value appropriately (e.g., dates, currency, booleans). |

### Purposes

- **To** display model values without creating editable form fields.
- **To** respect data annotations for formatting (e.g., date format, currency).
- **To** automatically HTML‑encode output to prevent XSS.
- **To** keep display logic consistent across views.

### Syntax Rules and Structure

```cshtml
@Html.DisplayFor(m => m.PropertyName)
@Html.DisplayFor(m => m.PropertyName, "DisplayTemplateName")
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `templateName` (optional) | Name of a custom display template in `Views/Shared/DisplayTemplates/`. |

#### Syntax Rules

- Requires a strongly typed view.
- The default template renders the value as text; booleans render as “True”/“False” unless a `[Display]` attribute overrides.
- Null values render as empty strings.

#### Constraints and Limitations

- `DisplayFor` does **not** generate `<label>` elements; use `DisplayNameFor` or `LabelFor`.
- For complex objects, it may render nothing unless a display template exists.

### Annotated Code Example

```cshtml
@model Product

<p>Name: @Html.DisplayFor(m => m.Name)</p>
<p>Price: @Html.DisplayFor(m => m.Price)</p>
<p>Release Date: @Html.DisplayFor(m => m.ReleaseDate)</p>
```

**Model:**

```csharp
public class Product
{
    public string Name { get; set; }
    [DisplayFormat(DataFormatString = "{0:C}")]
    public decimal Price { get; set; }
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }
}
```

**Expected Output:**

```html
<p>Name: Laptop</p>
<p>Price: $999.99</p>
<p>Release Date: 3/15/2025</p>
```

**Why this output?** `DisplayFor` reads the `[DisplayFormat]` and `[DataType]` attributes to format the price as currency and the date without time.

### Real‑World Cases

- **Product detail pages:** Display formatted price, date, and description.
- **Admin read‑only views:** Show user details without editable inputs.
- **Reports and invoices:** Render currency and dates consistently.

### References

- DisplayFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.displaynameextensions.displayfor
- Display and DisplayName attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 4. `@Html.DisplayNameFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `DisplayNameFor` renders the human‑readable name of a model property, typically used as a label or column header. |
| **Technical Definition** | `DisplayNameFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression)` reads the `DisplayName` from model metadata (derived from the property name or `[Display(Name = "...")]`). |
| **Beginner‑Friendly Explanation** | `DisplayNameFor` gives you the **friendly name** of a property. If your property is `ReleaseDate`, it can output “Release Date” if you added `[Display(Name = "Release Date")]`. |

### Purposes

- **To** display property names consistently across views.
- **To** respect `[Display(Name = "...")]` attributes without hard‑coding strings.
- **To** avoid duplicating label text in multiple places.
- **To** support localisation through resource‑based display names.

### Syntax Rules and Structure

```cshtml
@Html.DisplayNameFor(m => m.PropertyName)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |

#### Syntax Rules

- Returns an `IHtmlContent` string containing the display name (encoded).
- If no `[Display]` attribute is present, the property name is split into words (e.g., `ReleaseDate` → “Release Date”).

#### Constraints and Limitations

- Does **not** generate a `<label>` element; use `LabelFor` for that.
- Does not add a colon or other punctuation automatically.

### Annotated Code Example

```cshtml
@model Product

<h3>@Html.DisplayNameFor(m => m.ReleaseDate)</h3>
<p>@Html.DisplayFor(m => m.ReleaseDate)</p>
```

**Model:**

```csharp
[Display(Name = "Release Date")]
public DateTime ReleaseDate { get; set; }
```

**Expected Output:**

```html
<h3>Release Date</h3>
<p>3/15/2025</p>
```

**Why this output?** `DisplayNameFor` reads the `[Display(Name = "Release Date")]` attribute. Without it, the output would be “ReleaseDate” (no space).

### Real‑World Cases

- **Table headers:** `<th>@Html.DisplayNameFor(m => m.Name)</th>`.
- **Form section headings:** Display the property name above a group of fields.
- **Localised UI:** Use resource‑based display names for multi‑language apps.

### References

- DisplayNameFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.displaynameextensions.displaynamefor
- Display attribute – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.displayattribute

---

## 5. `@Html.EditorFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `EditorFor` renders an editor (input control) for a model property using the most appropriate template for its type. |
| **Technical Definition** | `EditorFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression)` consults model metadata to choose a built‑in editor template (`String`, `Boolean`, `DateTime`, `Collection`, etc.) or a custom template in `Views/Shared/EditorTemplates/`. |
| **Beginner‑Friendly Explanation** | `EditorFor` is the **smart input generator**. You tell it which property to edit, and it decides whether to render a text box, checkbox, date picker, or a custom editor — based on the property’s data type and annotations. |

### Purposes

- **To** automatically select the correct input control for a property’s data type.
- **To** honour `[DataType]` attributes (e.g., `DataType.Password`, `DataType.Date`).
- **To** support custom editor templates for reusable UI components.
- **To** handle collections and complex objects without manual loops.

### Syntax Rules and Structure

```cshtml
@Html.EditorFor(m => m.PropertyName)
@Html.EditorFor(m => m.PropertyName, "TemplateName")
@Html.EditorFor(m => m.PropertyName, new { htmlAttributes = new { @class = "form-control" } })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `templateName` | Optional custom template name. |
| `htmlAttributes` | Anonymous object for HTML attributes (must be nested inside `htmlAttributes` key). |

#### Syntax Rules

- The `htmlAttributes` key is **case‑insensitive** but must be spelled correctly inside the anonymous object.
- For collections, `EditorFor(m => m.Items)` renders an editor for each item.
- `EditorFor` is **strongly typed**; `Html.Editor()` is the weakly typed equivalent and is not recommended.

#### Constraints and Limitations

- The Input Tag Helper (`<input asp-for="...">`) overlaps with `EditorFor` but does **not** handle collections or complex objects. `EditorFor` remains more capable for those scenarios.
- Custom templates must follow the naming convention and folder structure.

### Annotated Code Example

```cshtml
@model RegisterViewModel

@Html.EditorFor(m => m.Email, new { htmlAttributes = new { @class = "form-control", placeholder = "you@example.com" } })
@Html.EditorFor(m => m.Password)
@Html.EditorFor(m => m.BirthDate)
```

**Model:**

```csharp
public class RegisterViewModel
{
    [DataType(DataType.EmailAddress)]
    public string Email { get; set; }

    [DataType(DataType.Password)]
    public string Password { get; set; }

    [DataType(DataType.Date)]
    public DateTime BirthDate { get; set; }
}
```

**Expected Output (simplified):**

```html
<input class="form-control" id="Email" name="Email" placeholder="you@example.com" type="email" value="" />
<input id="Password" name="Password" type="password" />
<input id="BirthDate" name="BirthDate" type="date" value="" />
```

**Why this output?** `EditorFor` reads `[DataType]` to select `type="email"`, `type="password"`, and `type="date"`. The `htmlAttributes` object adds the CSS class and placeholder.

### Real‑World Cases

- **Registration and login forms:** Password, email, and date fields generated with correct HTML5 types.
- **Dynamic forms:** Collections of sub‑models edited in a loop.
- **Admin panels:** Data‑type‑aware editors without manual switch statements.

### References

- EditorFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.editorextensions.editorfor
- Tag Helpers in forms (comparison with EditorFor) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 6. `@Html.LabelFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `LabelFor` renders an HTML `<label>` element whose `for` attribute matches the `id` of the corresponding input. |
| **Technical Definition** | `LabelFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression)` combines `DisplayNameFor` (for the label text) with the generated `id` of the property to produce `<label for="PropertyName">Display Name</label>`. |
| **Beginner‑Friendly Explanation** | `LabelFor` creates a **clickable label** that is linked to an input field. Clicking the label focuses the input, which improves accessibility and usability. |

### Purposes

- **To** improve accessibility by associating labels with inputs.
- **To** display the property’s display name without hard‑coding it.
- **To** ensure the `for` attribute matches the input’s `id` (critical for screen readers).
- **To** keep label text consistent with `DisplayNameFor`.

### Syntax Rules and Structure

```cshtml
@Html.LabelFor(m => m.PropertyName)
@Html.LabelFor(m => m.PropertyName, "Custom label text")
@Html.LabelFor(m => m.PropertyName, new { @class = "control-label" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `labelText` | Optional override for the label text. |
| `htmlAttributes` | Optional HTML attributes for the `<label>`. |

#### Syntax Rules

- The `for` attribute is set to the **HTML `id`** of the property, which may differ from the property name if `IdAttributeDotReplacement` is configured.
- The label text is HTML‑encoded.

#### Constraints and Limitations

- `LabelFor` does **not** generate the input itself; pair it with `EditorFor`, `TextBoxFor`, etc.
- If the input’s `id` is customised, the label may not point to the correct element.

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.Email, new { @class = "form-label" })
@Html.EditorFor(m => m.Email)
```

**Expected Output:**

```html
<label class="form-label" for="Email">Email</label>
<input id="Email" name="Email" type="email" />
```

**Why this output?** `LabelFor` derives the text “Email” from the property name and sets `for="Email"`, matching the input’s generated `id`.

### Real‑World Cases

- **Accessible forms:** Every input should have a corresponding label.
- **Bootstrap‑styled forms:** Apply `control-label` or `form-label` classes.
- **Localised forms:** Combine with resource‑based `[Display]` attributes.

### References

- LabelFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.labelextensions.labelfor
- Label Tag Helper (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 7. `@Html.TextBoxFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `TextBoxFor` renders a single‑line text input (`<input type="text">`) bound to a model property. |
| **Technical Definition** | `TextBoxFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression, object htmlAttributes)` generates an `<input>` element with `type="text"`, `id`, `name`, and `value` derived from the model property and `ModelState`. |
| **Beginner‑Friendly Explanation** | `TextBoxFor` is the **classic text box**. It shows the current value of a property and lets the user edit it. Unlike `EditorFor`, it always renders a text input (not a date picker or checkbox). |

### Purposes

- **To** provide a simple, predictable text input for string or numeric properties.
- **To** automatically populate the input with the current model value.
- **To** generate correct `name` and `id` attributes for model binding.
- **To** allow HTML attribute customisation (CSS classes, placeholders, `readonly`, etc.).

### Syntax Rules and Structure

```cshtml
@Html.TextBoxFor(m => m.PropertyName)
@Html.TextBoxFor(m => m.PropertyName, new { @class = "form-control" })
@Html.TextBoxFor(m => m.PropertyName, new { @readonly = "readonly" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `htmlAttributes` | Anonymous object of HTML attributes. |

#### Syntax Rules

- `TextBoxFor` does **not** automatically set the `type` attribute based on `[DataType]` — it always uses `type="text"`. Use `EditorFor` for data‑type‑aware inputs.
- The `value` is populated from `ModelState` first, then from the model.

#### Constraints and Limitations

- Not suitable for multi‑line text (use `TextAreaFor`) or passwords (use `PasswordFor`).
- The Input Tag Helper is the modern alternative but does not handle collections.

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.UserName)
@Html.TextBoxFor(m => m.UserName, new { @class = "form-control", placeholder = "Enter username" })
```

**Expected Output:**

```html
<label for="UserName">User Name</label>
<input class="form-control" id="UserName" name="UserName" placeholder="Enter username" type="text" value="john.doe" />
```

**Why this output?** `TextBoxFor` uses the model value (`john.doe`) and applies the CSS class and placeholder from `htmlAttributes`.

### Real‑World Cases

- **Search boxes:** Simple text input for search terms.
- **Read‑only fields:** `@Html.TextBoxFor(m => m.Id, new { @readonly = "readonly" })`.
- **Legacy forms:** When `EditorFor` is not desired because a plain text input is required.

### References

- TextBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.textboxfor
- Input Tag Helper (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 8. `@Html.TextAreaFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `TextAreaFor` renders a multi‑line text input (`<textarea>`) bound to a model property. |
| **Technical Definition** | `TextAreaFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression, int rows, int columns, object htmlAttributes)` renders a `<textarea>` with the property value as its inner text and the supplied `rows`/`columns` attributes. |
| **Beginner‑Friendly Explanation** | `TextAreaFor` is for **long text** — descriptions, comments, notes. It gives users a resizable multi‑line box instead of a single‑line text box. |

### Purposes

- **To** provide a multi‑line text input for long‑form content.
- **To** bind the textarea to a model property for model binding.
- **To** control the visible size of the textarea via `rows` and `columns`.
- **To** apply CSS classes and other HTML attributes.

### Syntax Rules and Structure

```cshtml
@Html.TextAreaFor(m => m.PropertyName)
@Html.TextAreaFor(m => m.PropertyName, 5, 40, new { @class = "form-control" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `rows` | Number of visible text lines. |
| `columns` | Number of visible characters per line. |
| `htmlAttributes` | Additional HTML attributes. |

#### Syntax Rules

- The property value is rendered as the **inner text** of the `<textarea>`, not as a `value` attribute.
- `rows` and `columns` are optional; without them, the browser default applies.

#### Constraints and Limitations

- The Textarea Tag Helper (`<textarea asp-for="...">`) is the modern alternative and provides the same strong typing and validation attributes.

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.Description)
@Html.TextAreaFor(m => m.Description, 4, 50, new { @class = "form-control" })
```

**Expected Output:**

```html
<label for="Description">Description</label>
<textarea class="form-control" cols="50" id="Description" name="Description" rows="4">A great product.</textarea>
```

**Why this output?** The property value `A great product.` is placed inside the `<textarea>` element, and `rows="4"` / `cols="50"` control the size.

### Real‑World Cases

- **Product descriptions:** Long‑form text entry in admin panels.
- **Comment boxes:** Blog or support‑ticket comments.
- **Address fields:** Multi‑line address input.

### References

- TextAreaFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.textareafor
- Textarea Tag Helper (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 9. `@Html.CheckBoxFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `CheckBoxFor` renders a boolean checkbox (`<input type="checkbox">`) bound to a model property. |
| **Technical Definition** | `CheckBoxFor<TModel>(Expression<Func<TModel, bool>> expression)` renders the checkbox **plus a hidden input with the same name and value `false`**. This ensures that `false` is posted even when the checkbox is unchecked (browsers do not submit unchecked checkboxes). |
| **Beginner‑Friendly Explanation** | `CheckBoxFor` creates a **tick box** for true/false values. It also adds a hidden field so that your model receives `false` when the user leaves the box unchecked. |

### Purposes

- **To** provide a boolean toggle (yes/no, true/false) bound to a model property.
- **To** ensure that `false` is correctly posted when the checkbox is unchecked.
- **To** respect `ModelState` and re‑display the correct checked state after validation errors.
- **To** support HTML attributes such as `disabled` or CSS classes.

### Syntax Rules and Structure

```cshtml
@Html.CheckBoxFor(m => m.IsActive)
@Html.CheckBoxFor(m => m.IsActive, new { @class = "form-check-input" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting a **bool** property. |
| `htmlAttributes` | Optional HTML attributes. |

#### Syntax Rules

- The property **must be of type `bool`**.
- `CheckBoxFor` renders **two** `<input>` elements: the visible checkbox and a hidden fallback.
- The hidden input’s name matches the checkbox name, and its value is `"false"`.

#### Constraints and Limitations

- The hidden fallback can be disabled globally via `CheckBoxHiddenInputRenderMode.None` if desired.
- Model binding reads the **first** value for a `bool`, which is why the checkbox appears before the hidden input.

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.IsActive)
@Html.CheckBoxFor(m => m.IsActive, new { @class = "form-check-input" })
```

**Expected Output:**

```html
<label for="IsActive">Is Active</label>
<input checked="checked" class="form-check-input" id="IsActive" name="IsActive" type="checkbox" value="true" />
<input name="IsActive" type="hidden" value="false" />
```

**Why this output?** If `IsActive` is `true`, the checkbox is checked. The hidden field ensures `false` is posted if the user unchecks it.

### Real‑World Cases

- **Settings toggles:** Enable/disable notifications, features, or accounts.
- **Terms acceptance:** `@Html.CheckBoxFor(m => m.AcceptedTerms)`.
- **Admin flags:** Mark records as active, featured, or published.

### References

- CheckBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.checkboxfor
- Checkbox hidden input rendering – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 10. `@Html.RadioButtonFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RadioButtonFor` renders a radio button (`<input type="radio">`) bound to a model property and a specific value. |
| **Technical Definition** | `RadioButtonFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression, object value)` renders a radio input whose `checked` state is determined by comparing the model value to the supplied `value`. |
| **Beginner‑Friendly Explanation** | `RadioButtonFor` creates **one option in a group of mutually exclusive choices**. For example, “Gender: ( ) Male ( ) Female” — each radio button represents one value. |

### Purposes

- **To** present a set of mutually exclusive options bound to a single property.
- **To** automatically check the radio button that matches the current model value.
- **To** generate correct `name` attributes so that only one value is posted.
- **To** support enum‑based or string‑based choice properties.

### Syntax Rules and Structure

```cshtml
@Html.RadioButtonFor(m => m.PropertyName, "Value1")
@Html.RadioButtonFor(m => m.PropertyName, "Value1", new { id = "customId" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `value` | The value this radio button represents. |
| `htmlAttributes` | Optional HTML attributes. |

#### Syntax Rules

- All radio buttons for the same property must share the same `name` (which `RadioButtonFor` generates automatically).
- The `checked` attribute is added when the model value equals the supplied `value`.
- Use distinct `id` attributes (or let the helper generate them) to keep labels working correctly.

#### Constraints and Limitations

- You must render **each option separately**; there is no built‑in “radio group” helper.
- For enum properties, consider `Html.EnumRadioButtonFor` (not built‑in) or a loop.

### Annotated Code Example

```cshtml
@Html.RadioButtonFor(m => m.Gender, "M", new { id = "genderM" })
@Html.Label("genderM", "Male")

@Html.RadioButtonFor(m => m.Gender, "F", new { id = "genderF" })
@Html.Label("genderF", "Female")
```

**Expected Output (assuming `Gender = "M"`):**

```html
<input checked="checked" id="genderM" name="Gender" type="radio" value="M" />
<label for="genderM">Male</label>
<input id="genderF" name="Gender" type="radio" value="F" />
<label for="genderF">Female</label>
```

**Why this output?** The first radio button is checked because the model value matches `"M"`. Both share `name="Gender"`, ensuring mutual exclusivity.

### Real‑World Cases

- **Surveys and forms:** Gender, yes/no, rating scales.
- **Shipping options:** Standard vs. Express delivery.
- **Enum selection:** Map enum values to radio buttons.

### References

- RadioButtonFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.radiobuttonfor

---

## 11. `@Html.DropDownListFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `DropDownListFor` renders a single‑selection `<select>` element bound to a model property, populated from a collection of `SelectListItem`. |
| **Technical Definition** | `DropDownListFor<TModel, TProperty>(Expression<Func<TModel, TProperty>> expression, IEnumerable<SelectListItem> selectList, string optionLabel, object htmlAttributes)` generates `<select>` and `<option>` elements, marking the option that matches the model value as `selected`. |
| **Beginner‑Friendly Explanation** | `DropDownListFor` creates a **dropdown menu** where the user picks one option. You provide the list of options, and the helper builds the HTML. |

### Purposes

- **To** provide a compact single‑choice UI for a list of options.
- **To** bind the selected value to a model property automatically.
- **To** support an optional placeholder option (`"-- Select --"`).
- **To** apply CSS classes and other attributes to the `<select>`.

### Syntax Rules and Structure

```cshtml
@Html.DropDownListFor(m => m.PropertyName, Model.Options, "Select...", new { @class = "form-select" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `selectList` | `IEnumerable<SelectListItem>` containing the options. |
| `optionLabel` | Optional text for an empty first option. |
| `htmlAttributes` | Optional HTML attributes. |

#### Syntax Rules

- The `SelectListItem` collection must be supplied; there is no automatic population from the model.
- The `Selected` property of `SelectListItem` is **overridden** by the model value if they conflict.
- For multi‑select, use `ListBoxFor` instead.

#### Constraints and Limitations

- The Select Tag Helper (`<select asp-for="..." asp-items="...">`) is the modern alternative and is often more concise.
- `DropDownListFor` does not support grouping (optgroups) without custom templates.

### Annotated Code Example

```cshtml
@Html.DropDownListFor(
    m => m.SelectedCategoryId,
    Model.Categories,          // IEnumerable<SelectListItem>
    "-- Choose a category --",
    new { @class = "form-select" })
```

**Expected Output (assuming `SelectedCategoryId = 2`):**

```html
<select class="form-select" id="SelectedCategoryId" name="SelectedCategoryId">
    <option value="">-- Choose a category --</option>
    <option value="1">Electronics</option>
    <option selected="selected" value="2">Books</option>
    <option value="3">Clothing</option>
</select>
```

**Why this output?** The option whose `Value` matches `SelectedCategoryId` receives the `selected` attribute. The `optionLabel` adds the placeholder option.

### Real‑World Cases

- **Category selection:** Product categories, blog categories.
- **Country/state pickers:** Large option lists.
- **Status selection:** Order status, user role.

### References

- DropDownListFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectextensions.dropdownlistfor
- Select Tag Helper (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 12. `@Html.ListBoxFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ListBoxFor` renders a multi‑selection `<select multiple>` element bound to a collection property. |
| **Technical Definition** | `ListBoxFor<TModel, TProperty>(Expression<Func<TModel, TProperty>> expression, IEnumerable<SelectListItem> selectList, object htmlAttributes)` generates a `<select multiple="multiple">` where multiple options can be selected and bound to an array or collection property. |
| **Beginner‑Friendly Explanation** | `ListBoxFor` is like a dropdown, but the user can **select multiple options** at once (usually by holding Ctrl or Shift). It’s a list box, not a dropdown. |

### Purposes

- **To** allow selection of multiple values from a list.
- **To** bind a collection property (e.g., `int[]`, `List<string>`) to the selected options.
- **To** provide a visible list of options (unlike a dropdown, which hides them).
- **To** support `multiple` selection without custom JavaScript.

### Syntax Rules and Structure

```cshtml
@Html.ListBoxFor(m => m.SelectedIds, Model.Options, new { @class = "form-select", size = 5 })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting a **collection** property. |
| `selectList` | `IEnumerable<SelectListItem>`. |
| `htmlAttributes` | Optional HTML attributes (`size` controls visible rows). |

#### Syntax Rules

- The target property should be an `IEnumerable` (e.g., `int[]`, `List<int>`).
- The `multiple="multiple"` attribute is added automatically.
- Options whose `Value` appears in the model collection receive `selected="selected"`.

#### Constraints and Limitations

- The Select Tag Helper can also render multi‑select (`<select asp-for="..." asp-items="..." multiple>`), but `ListBoxFor` is still valid.
- `DropDownListFor` with a `multiple` attribute does **not** preserve multi‑selection correctly after postback; use `ListBoxFor` instead.

### Annotated Code Example

```cshtml
@Html.ListBoxFor(m => m.SelectedTagIds, Model.AllTags, new { size = 4 })
```

**Expected Output (assuming tags 1 and 3 are selected):**

```html
<select id="SelectedTagIds" multiple="multiple" name="SelectedTagIds" size="4">
    <option selected="selected" value="1">ASP.NET</option>
    <option value="2">C#</option>
    <option selected="selected" value="3">MVC</option>
    <option value="4">Razor</option>
</select>
```

**Why this output?** The options whose values are in `SelectedTagIds` are marked as selected. The `multiple` attribute allows more than one selection.

### Real‑World Cases

- **Tag selection:** Blog posts with multiple tags.
- **Permission assignment:** Users with multiple roles.
- **Product filters:** Select multiple categories at once.

### References

- ListBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectextensions.listboxfor

---

## 13. `@Html.HiddenFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `HiddenFor` renders a hidden input (`<input type="hidden">`) bound to a model property. |
| **Technical Definition** | `HiddenFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression)` generates a hidden input with `id` and `name` derived from the property, and `value` from `ModelState` or the model. |
| **Beginner‑Friendly Explanation** | `HiddenFor` stores a value in the form **without showing it to the user**. It’s commonly used for IDs, timestamps, or concurrency tokens that must be posted back to the server. |

### Purposes

- **To** persist a model value across a round‑trip without user interaction.
- **To** pass identifiers (e.g., `Id`) in edit forms.
- **To** include concurrency tokens (e.g., `RowVersion`) in the form.
- **To** preserve values that should not be editable.

### Syntax Rules and Structure

```cshtml
@Html.HiddenFor(m => m.Id)
@Html.HiddenFor(m => m.RowVersion)
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |

#### Syntax Rules

- The input is **not visible** to the user but **is submitted** with the form.
- Values are HTML‑encoded.
- For non‑nullable value types, an empty string may be rendered if the value is null — be mindful of model‑binding errors.

#### Constraints and Limitations

- **Security:** Hidden fields are visible to the client via “View Source.” **Never** store sensitive data (passwords, secrets) in hidden fields.
- For concurrency tokens, use `[HiddenInput]` or `HiddenFor` correctly with `[Timestamp]`.

### Annotated Code Example

```cshtml
@Html.HiddenFor(m => m.Id)
@Html.HiddenFor(m => m.RowVersion)
@Html.EditorFor(m => m.Name)
```

**Expected Output:**

```html
<input id="Id" name="Id" type="hidden" value="42" />
<input id="RowVersion" name="RowVersion" type="hidden" value="AAAAAAAAB9E=" />
<input id="Name" name="Name" type="text" value="Laptop" />
```

**Why this output?** `HiddenFor` renders the current `Id` and `RowVersion` values so they are posted back when the form is submitted.

### Real‑World Cases

- **Edit forms:** Preserve the primary key (`Id`) of the entity being edited.
- **Optimistic concurrency:** Post the original `RowVersion` back to detect conflicts.
- **Wizard steps:** Carry hidden state between pages.

### References

- HiddenFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.hiddenfor

---

## 14. `@Html.PasswordFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `PasswordFor` renders a password input (`<input type="password">`) bound to a model property. |
| **Technical Definition** | `PasswordFor<TModel, TValue>(Expression<Func<TModel, TValue>> expression, object htmlAttributes)` generates a password field. Unlike `TextBoxFor`, it **does not re‑populate the value** from `ModelState` after a postback, for security reasons. |
| **Beginner‑Friendly Explanation** | `PasswordFor` creates a **masked input** where characters are hidden (shown as dots or asterisks). It deliberately does **not** refill the password after a failed validation, so the user must re‑enter it. |

### Purposes

- **To** securely collect password input without displaying the characters.
- **To** prevent password values from being re‑rendered in the HTML after a postback.
- **To** bind the password field to a model property for validation (e.g., `[Required]`, `[StringLength]`).
- **To** support HTML attributes (placeholders, CSS classes).

### Syntax Rules and Structure

```cshtml
@Html.PasswordFor(m => m.Password)
@Html.PasswordFor(m => m.Password, new { @class = "form-control", placeholder = "Enter password" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `htmlAttributes` | Optional HTML attributes. |

#### Syntax Rules

- The input’s `value` is **always empty** on initial render and after a postback — this is by design.
- Use `[DataType(DataType.Password)]` on the model for metadata, or use `PasswordFor` directly.

#### Constraints and Limitations

- **Do not** attempt to repopulate passwords via `ModelState`; doing so is a security anti‑pattern.
- For password confirmation fields, use two separate properties and compare them with a custom validation attribute.

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.Password)
@Html.PasswordFor(m => m.Password, new { @class = "form-control" })

@Html.LabelFor(m => m.ConfirmPassword)
@Html.PasswordFor(m => m.ConfirmPassword, new { @class = "form-control" })
```

**Expected Output:**

```html
<label for="Password">Password</label>
<input class="form-control" id="Password" name="Password" type="password" />
<label for="ConfirmPassword">Confirm Password</label>
<input class="form-control" id="ConfirmPassword" name="ConfirmPassword" type="password" />
```

**Why this output?** No `value` attribute is present, even if the model has a password value. This prevents the password from appearing in the page source after a validation error.

### Real‑World Cases

- **Registration forms:** Password and confirm‑password fields.
- **Login forms:** Password entry without value repopulation.
- **Password reset:** New password and confirmation.

### References

- PasswordFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.passwordfor

---

## 15. `@Html.ValidationMessageFor()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ValidationMessageFor` renders the validation error message associated with a specific model property. |
| **Technical Definition** | `ValidationMessageFor<TModel, TProperty>(Expression<Func<TModel, TProperty>> expression, string validationMessage, object htmlAttributes)` reads `ModelState` for the property and renders a `<span>` containing the first error message, or the fallback `validationMessage` if provided. |
| **Beginner‑Friendly Explanation** | `ValidationMessageFor` shows the **error text next to a specific field** when validation fails. For example, if the “Email” field is required and left empty, this helper displays “The Email field is required.” |

### Purposes

- **To** display property‑level validation errors in context.
- **To** improve UX by showing errors near the offending input.
- **To** respect data annotation validation messages (`[Required(ErrorMessage = "...")]`).
- **To** allow a custom fallback message when no specific error exists.

### Syntax Rules and Structure

```cshtml
@Html.ValidationMessageFor(m => m.PropertyName)
@Html.ValidationMessageFor(m => m.PropertyName, "Custom error message", new { @class = "text-danger" })
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `expression` | Lambda selecting the property. |
| `validationMessage` | Optional fallback message. |
| `htmlAttributes` | Optional HTML attributes for the `<span>`. |

#### Syntax Rules

- The helper renders a `<span>` element. If there is no error, the span may be empty or omitted.
- The message is HTML‑encoded.
- Typically placed immediately after the corresponding input.

#### Constraints and Limitations

- The Validation Message Tag Helper (`<span asp-validation-for="...">`) is the modern alternative and behaves similarly.
- Does **not** display model‑level errors (use `ValidationSummary` for those).

### Annotated Code Example

```cshtml
@Html.LabelFor(m => m.Email)
@Html.EditorFor(m => m.Email)
@Html.ValidationMessageFor(m => m.Email, "", new { @class = "text-danger" })
```

**Model:**

```csharp
[Required(ErrorMessage = "Email is required.")]
[EmailAddress(ErrorMessage = "Invalid email format.")]
public string Email { get; set; }
```

**Expected Output (after submitting an empty email):**

```html
<label for="Email">Email</label>
<input class="text-box single-line" id="Email" name="Email" type="email" value="" />
<span class="text-danger field-validation-error" data-valmsg-for="Email" data-valmsg-replace="true">Email is required.</span>
```

**Why this output?** `ValidationMessageFor` reads the `ModelState` error for `Email` and renders the `[Required]` error message inside a `<span>` with the specified CSS class.

### Real‑World Cases

- **Registration forms:** Show field‑specific errors for username, email, password.
- **Checkout forms:** Validate shipping address fields individually.
- **Data entry screens:** Immediate feedback per field.

### References

- ValidationMessageFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.validationextensions.validationmessagefor
- Validation Tag Helpers (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 16. `@Html.ValidationSummary()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ValidationSummary` renders a consolidated list of all validation errors, optionally excluding property‑level errors. |
| **Technical Definition** | `ValidationSummary(bool excludePropertyErrors, string message, object htmlAttributes)` reads `ModelState` and renders a `<div>` containing a `<ul>` of error messages. If `excludePropertyErrors` is `true`, only model‑level errors are shown. |
| **Beginner‑Friendly Explanation** | `ValidationSummary` is a **bulleted list of all errors** at the top (or bottom) of a form. Instead of showing errors next to each field, it gives an overview. |

### Purposes

- **To** provide a single place to see all validation errors.
- **To** display model‑level errors that are not tied to a specific property.
- **To** optionally hide property‑level errors (when `ValidationMessageFor` already shows them).
- **To** improve accessibility by grouping errors in a list.

### Syntax Rules and Structure

```cshtml
@Html.ValidationSummary()
@Html.ValidationSummary(true)                    // exclude property errors
@Html.ValidationSummary(false, "Please fix the errors below.")
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `excludePropertyErrors` | `true` to show only model‑level errors; `false` (default) to show all. |
| `message` | Optional introductory message. |
| `htmlAttributes` | Optional attributes for the wrapping `<div>`. |

#### Syntax Rules

- Default (`false`) includes **all** errors.
- `true` excludes property‑level errors — useful when each field already has its own `ValidationMessageFor`.
- The output is a `<div class="validation-summary-errors">` containing a `<ul>`.

#### Constraints and Limitations

- The Validation Summary Tag Helper (`<div asp-validation-summary="All">`) is the modern alternative.
- Does not automatically scroll to the first error; consider client‑side enhancements.

### Annotated Code Example

```cshtml
@Html.ValidationSummary(true, "Please correct the following errors:")
@Html.LabelFor(m => m.Name)
@Html.EditorFor(m => m.Name)
@Html.ValidationMessageFor(m => m.Name)
```

**Expected Output (when a model‑level error exists and the `Name` field is also invalid):**

```html
<div class="validation-summary-errors">
    <span>Please correct the following errors:</span>
    <ul>
        <li>An account with this email already exists.</li>
    </ul>
</div>
<!-- Name field and its ValidationMessageFor are rendered separately -->
```

**Why this output?** `ValidationSummary(true)` excludes the `Name` property error (which is handled by `ValidationMessageFor`) and shows only the model‑level error.

### Real‑World Cases

- **Registration forms:** Show “Email already in use” model‑level errors at the top.
- **Complex forms:** Summarise errors when many fields fail validation.
- **Accessibility:** Provide a single error list for screen readers.

### References

- ValidationSummary – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.validationextensions.validationsummary
- Validation Summary Tag Helper – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 17. `@Html.BeginForm()` / `@Html.EndForm()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `BeginForm` renders the opening `<form>` tag, and `EndForm` renders the closing `</form>` tag. |
| **Technical Definition** | `Html.BeginForm(actionName, controllerName, routeValues, method, htmlAttributes)` returns an `MvcForm` object that implements `IDisposable`. In a `using` block, `EndForm` is called automatically when the block is disposed. |
| **Beginner‑Friendly Explanation** | `BeginForm` and `EndForm` are the **start and end of a form**. Everything between them is submitted to the server when the user clicks a submit button. |

### Purposes

- **To** create an HTML form that posts to a specific controller action.
- **To** generate the correct `action` URL using routing.
- **To** support GET or POST methods.
- **To** allow HTML attributes (CSS classes, `enctype` for file uploads).

### Syntax Rules and Structure

```cshtml
@using (Html.BeginForm("ActionName", "ControllerName", FormMethod.Post, new { @class = "form" }))
{
    <!-- form fields -->
}
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `actionName` | Target action method. |
| `controllerName` | Target controller. |
| `routeValues` | Optional route parameters. |
| `method` | `FormMethod.Get` or `FormMethod.Post`. |
| `htmlAttributes` | Anonymous object of HTML attributes. |

#### Syntax Rules

- The `using` statement ensures `EndForm()` is called automatically.
- If `actionName` and `controllerName` are omitted, the form posts to the **current action**.
- The `method` defaults to `POST` if not specified.

#### Constraints and Limitations

- The Form Tag Helper (`<form asp-action="..." asp-controller="...">`) is the modern alternative and also **generates the anti‑forgery token automatically**.
- `BeginForm` does **not** automatically add the anti‑forgery token; use `@Html.AntiForgeryToken()` explicitly if needed.

### Annotated Code Example

```cshtml
@using (Html.BeginForm("Create", "Products", FormMethod.Post, new { @class = "form-horizontal" }))
{
    @Html.AntiForgeryToken()
    @Html.LabelFor(m => m.Name)
    @Html.EditorFor(m => m.Name)
    @Html.ValidationMessageFor(m => m.Name)

    <button type="submit">Save</button>
}
```

**Expected Output:**

```html
<form action="/Products/Create" class="form-horizontal" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="..." />
    <label for="Name">Name</label>
    <input id="Name" name="Name" type="text" value="" />
    <span class="field-validation-valid" data-valmsg-for="Name" data-valmsg-replace="true"></span>
    <button type="submit">Save</button>
</form>
```

**Why this output?** `BeginForm` generates the `action` URL via routing and the `method="post"` attribute. The `using` block automatically closes the form with `</form>`.

### Real‑World Cases

- **CRUD create/edit forms:** Post data to `Create` or `Edit` actions.
- **File uploads:** Set `enctype = "multipart/form-data"`.
- **Search forms:** Use `FormMethod.Get` to append query parameters.

### References

- BeginForm – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperformextensions.beginform
- Form Tag Helper (alternative) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms

---

## 18. `@Html.AntiForgeryToken()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AntiForgeryToken` renders a hidden input containing a CSRF anti‑forgery token. |
| **Technical Definition** | `Html.AntiForgeryToken()` calls the anti‑forgery service to generate a token and renders `<input name="__RequestVerificationToken" type="hidden" value="..." />`. The token is validated by the `[ValidateAntiForgeryToken]` filter on the server. |
| **Beginner‑Friendly Explanation** | `AntiForgeryToken` protects your forms against **Cross‑Site Request Forgery (CSRF)** attacks. It adds a secret token to the form that an attacker’s site cannot guess, so the server can verify that the request came from your own page. |

### Purposes

- **To** protect POST actions from CSRF attacks.
- **To** generate the hidden `__RequestVerificationToken` field required by `[ValidateAntiForgeryToken]`.
- **To** work alongside the Form Tag Helper (which adds the token automatically) or explicitly in plain HTML forms.
- **To** ensure that forms submitted from other origins are rejected.

### Syntax Rules and Structure

```cshtml
@Html.AntiForgeryToken()
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| No parameters | The helper generates the token and hidden input automatically. |

#### Syntax Rules

- Must be placed **inside** the `<form>` element.
- The generated input name is `__RequestVerificationToken`.
- The corresponding action must be decorated with `[ValidateAntiForgeryToken]` (or `[AutoValidateAntiforgeryToken]` globally).

#### Constraints and Limitations

- **Do not** use `Html.Raw` on the token output.
- The Form Tag Helper adds the token automatically, so `AntiForgeryToken()` is only needed for plain HTML forms or when you opt out of the Form Tag Helper.
- The token is tied to the user’s session/cookie; it will not validate across different users.

### Annotated Code Example

```cshtml
<form action="/Products/Create" method="post">
    @Html.AntiForgeryToken()
    <input type="text" name="Name" />
    <button type="submit">Save</button>
</form>
```

**Controller:**

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Product product) { ... }
```

**Expected Output:**

```html
<form action="/Products/Create" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8...removed for brevity..." />
    <input type="text" name="Name" />
    <button type="submit">Save</button>
</form>
```

**Why this output?** The hidden input contains a cryptographically random token. When the form is submitted, `[ValidateAntiForgeryToken]` verifies that the token matches the user’s anti‑forgery cookie.

### Real‑World Cases

- **All POST forms:** Login, registration, checkout, admin actions.
- **AJAX requests:** Include the token in the request header (`RequestVerificationToken`) or body.
- **Legacy forms:** When not using the Form Tag Helper.

### References

- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- AntiForgeryToken – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken

---

## 19. `@Html.Raw()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Html.Raw` outputs a string **without HTML encoding**. |
| **Technical Definition** | `Html.Raw(string value)` returns an `IHtmlContent` wrapping the string unchanged. Razor then writes it directly to the response. |
| **Beginner‑Friendly Explanation** | Normally, Razor **encodes** output for safety (e.g., `<` becomes `&lt;`). `Html.Raw` tells Razor: “Don’t encode this — I trust it.” It should be used only when you are certain the content is safe. |

### Purposes

- **To** render trusted HTML markup stored in a string (e.g., formatted article content).
- **To** display raw XML or JSON that has already been sanitised.
- **To** intentionally bypass Razor’s automatic encoding for trusted sources.

### Syntax Rules and Structure

```cshtml
@Html.Raw(Model.TrustedHtml)
@Html.Raw("<strong>Bold text</strong>")
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | The string to output without encoding. |

#### Syntax Rules

- Accepts a `string` or `object`.
- Returns `IHtmlContent`; Razor writes it directly.
- **No encoding is performed** — this is the entire point of the helper.

#### Constraints and Limitations

- **XSS risk:** If the input contains user‑controlled data, `Html.Raw` can introduce a Cross‑Site Scripting vulnerability.
- **Never** use `Html.Raw` with untrusted input. If you need to display user HTML, sanitise it first (e.g., with an HTML sanitiser library).
- For safe encoding, use `Html.Encode` or rely on Razor’s default `@` encoding.

### Annotated Code Example

```cshtml
@* Safe: trusted HTML from a CMS editor that sanitises input *@
<div class="article-content">
    @Html.Raw(Model.SanitizedArticleHtml)
</div>
```

**Expected Output:**

```html
<div class="article-content">
    <p>This is <strong>formatted</strong> content.</p>
</div>
```

**Why this output?** Without `Html.Raw`, Razor would encode the `<p>` and `<strong>` tags as `&lt;p&gt;`, displaying them as text instead of rendering them.

### Real‑World Cases

- **CMS article rendering:** Display HTML content produced by a rich‑text editor (after sanitisation).
- **Email templates:** Render trusted HTML fragments.
- **Debugging:** Output raw JSON or XML for inspection.

### References

- Html.Raw – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelper.raw
- Preventing XSS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting

---

## 20. `@Html.Encode()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Html.Encode` HTML‑encodes a value so that special characters are rendered safely as text. |
| **Technical Definition** | `Html.Encode(object value)` converts `<`, `>`, `&`, `"`, and `'` to their HTML entities and returns the encoded string. Razor already encodes output by default, so `Html.Encode` is primarily useful when you need an **encoded string** rather than `IHtmlContent`. |
| **Beginner‑Friendly Explanation** | `Html.Encode` makes text **safe to display**. If a user enters `<script>alert('xss')</script>`, `Html.Encode` turns it into `&lt;script&gt;alert('xss')&lt;/script&gt;`, which the browser shows as text instead of executing. |

### Purposes

- **To** explicitly HTML‑encode a value when Razor’s automatic encoding is not applied.
- **To** produce encoded strings for use in JavaScript or other contexts where `IHtmlContent` is not suitable.
- **To** safely display user‑supplied text that might contain HTML or script characters.

### Syntax Rules and Structure

```cshtml
@Html.Encode(Model.UserInput)
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | The object/string to encode. |

#### Syntax Rules

- Returns a **plain string** (not `IHtmlContent`).
- Razor’s default `@` syntax already encodes; `Html.Encode` is needed only in specific scenarios (e.g., building an encoded string for a script block).

#### Constraints and Limitations

- **Do not** use `Html.Encode` together with `Html.Raw` unless you intentionally want to double‑encode and then display the encoded text.
- For URL encoding, use `UrlEncoder`; for JavaScript encoding, use `JavaScriptEncoder`.

### Annotated Code Example

```cshtml
@{
    var userInput = "<script>alert('XSS');</script>";
}
<p>Encoded: @Html.Encode(userInput)</p>
```

**Expected Output:**

```html
<p>Encoded: &lt;script&gt;alert('XSS');&lt;/script&gt;</p>
```

**Why this output?** `Html.Encode` converts the angle brackets to entities, so the browser displays the script tag as text instead of executing it.

### Real‑World Cases

- **Displaying raw user input:** Show exactly what the user typed without executing it.
- **Debugging:** Log encoded values to inspect special characters.
- **Building encoded strings for script blocks:** When you need a string, not `IHtmlContent`.

### References

- Html.Encode – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelper.encode
- Cross-Site Scripting (XSS) prevention – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting

---

## 21. HTML Helper Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Best practices are guidelines for using HTML Helpers safely, efficiently, and maintainably. |
| **Technical Definition** | These practices encompass security (XSS/CSRF), performance (avoiding blocking calls), maintainability (strong typing over string‑based helpers), and interoperability with Tag Helpers. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** that experienced developers follow to avoid bugs, security holes, and messy code when using HTML Helpers. |

### Purposes

- **To** prevent XSS vulnerabilities when outputting content.
- **To** avoid CSRF vulnerabilities in forms.
- **To** keep views maintainable and refactor‑safe.
- **To** use the most appropriate helper (HTML Helper vs. Tag Helper) for each scenario.
- **To** avoid performance pitfalls such as synchronous partial rendering.

### Syntax Rules and Structure (Guideline Checklist)

| Practice | Do | Don’t |
|----------|----|-------|
| **Prefer strongly typed helpers** | `@Html.TextBoxFor(m => m.Name)` | `@Html.TextBox("Name")` |
| **Use `EditorFor` for data‑type‑aware inputs** | `@Html.EditorFor(m => m.BirthDate)` | `@Html.TextBoxFor(m => m.BirthDate)` if a date picker is desired |
| **Do not use `Html.Raw` with user input** | `@Html.Encode(userInput)` or plain `@userInput` | `@Html.Raw(userInput)` without sanitisation |
| **Include anti‑forgery tokens in POST forms** | `@Html.AntiForgeryToken()` inside `<form>` with `[ValidateAntiForgeryToken]` | Omit the token |
| **Prefer Tag Helpers where equivalent** | `<input asp-for="Name" />` | Using only HTML Helpers for every form field when Tag Helpers are available |
| **Avoid synchronous `IHtmlHelper.Partial`** | `PartialAsync` or `<partial name="_View" />` | `Html.Partial("_View")` (analyzer MVC1000) |
| **Return `IHtmlContent` from custom helpers** | Use `TagBuilder` and return `IHtmlContent` | Concatenate raw HTML strings |
| **Keep custom helpers stateless** | Pure rendering logic | Injecting services or performing data access inside helpers |

### Constraints and Limitations

- **Tag Helpers do not replace all HTML Helpers.** `Html.Raw`, `Html.Encode`, and explicit `AntiForgeryToken` are still needed in certain scenarios.
- **Do not** use HTML Helpers to perform business logic; keep views presentation‑focused.
- **Do not** rely on `Html.Raw` for “convenience” — it is a security boundary.

### Annotated Code Example (Demonstrating Multiple Best Practices)

```cshtml
@model ProductViewModel

@using (Html.BeginForm("Edit", "Products", FormMethod.Post))
{
    @Html.AntiForgeryToken()                    @* CSRF protection *@
    @Html.HiddenFor(m => m.Id)                  @* strong typing *@

    @Html.LabelFor(m => m.Name)
    @Html.EditorFor(m => m.Name)                @* data-type-aware editor *@
    @Html.ValidationMessageFor(m => m.Name)     @* field-level validation *@

    @Html.LabelFor(m => m.Description)
    @Html.TextAreaFor(m => m.Description, 4, 50, new { @class = "form-control" })

    <button type="submit">Save</button>
}
```

**Why this follows best practices:**

- Strongly typed `*For` helpers prevent typos.
- `EditorFor` selects the correct input type for `Name`.
- `AntiForgeryToken` protects the POST action.
- `HiddenFor` preserves the entity ID without exposing it visually.
- `ValidationMessageFor` displays field‑specific errors.

### Real‑World Cases

- **Enterprise MVC applications:** Consistent use of strongly typed helpers reduces runtime errors and improves refactoring safety.
- **Security‑sensitive forms:** Explicit CSRF tokens and disciplined encoding prevent common vulnerabilities.
- **Migration from MVC 5:** Existing HTML Helper code continues to work, but new development should lean toward Tag Helpers where equivalent functionality exists.

### References

- Tag Helpers in forms – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Prevent Cross-Site Scripting (XSS) – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- MVC1000: Use of IHtmlHelper.Partial should be avoided – https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/mvc1000
- Custom HTML Helpers (TagBuilder guidance) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/html-helpers

---

## Summary Table: All Helpers at a Glance

| Helper | Purpose | Strongly Typed? | Common Alternative |
|--------|---------|-----------------|-------------------|
| `ActionLink` | Anchor to an action | No (string‑based) | `<a asp-action="...">` |
| `DisplayFor` | Read‑only property display | Yes | `<label>` or plain `@Model.Property` |
| `DisplayNameFor` | Property display name | Yes | — |
| `EditorFor` | Data‑type‑aware editor | Yes | `<input asp-for="...">` (not for collections) |
| `LabelFor` | Accessible `<label>` | Yes | `<label asp-for="...">` |
| `TextBoxFor` | Single‑line text input | Yes | `<input asp-for="...">` |
| `TextAreaFor` | Multi‑line textarea | Yes | `<textarea asp-for="...">` |
| `CheckBoxFor` | Boolean checkbox + hidden fallback | Yes | `<input asp-for="...">` (Tag Helper) |
| `RadioButtonFor` | One radio option | Yes | `<input asp-for="..." value="...">` |
| `DropDownListFor` | Single‑select dropdown | Yes | `<select asp-for="..." asp-items="...">` |
| `ListBoxFor` | Multi‑select list box | Yes | `<select asp-for="..." asp-items="..." multiple>` |
| `HiddenFor` | Hidden input | Yes | `<input asp-for="..." type="hidden">` |
| `PasswordFor` | Password input (no value refill) | Yes | `<input asp-for="..." type="password">` |
| `ValidationMessageFor` | Property‑level error message | Yes | `<span asp-validation-for="...">` |
| `ValidationSummary` | All‑errors summary | No | `<div asp-validation-summary="All">` |
| `BeginForm` / `EndForm` | Form wrapper | No | `<form asp-action="...">` |
| `AntiForgeryToken` | CSRF token | No | Automatically added by Form Tag Helper |
| `Raw` | Unencoded output | No | — (use with extreme caution) |
| `Encode` | HTML encoding | No | Razor’s default `@` encoding |

---

## References (Full List)

- IHtmlHelper<TModel> Interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper-1
- HtmlHelper Class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper
- Tag Helpers in forms in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- DisplayFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.displaynameextensions.displayfor
- EditorFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.editorextensions.editorfor
- LabelFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.labelextensions.labelfor
- TextBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.textboxfor
- TextAreaFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.textareafor
- CheckBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.checkboxfor
- RadioButtonFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.radiobuttonfor
- DropDownListFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectextensions.dropdownlistfor
- ListBoxFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectextensions.listboxfor
- HiddenFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.hiddenfor
- PasswordFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.inputextensions.passwordfor
- ValidationMessageFor – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.validationextensions.validationmessagefor
- ValidationSummary – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.validationextensions.validationsummary
- BeginForm – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperformextensions.beginform
- AntiForgeryToken – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken
- Html.Raw – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelper.raw
- Html.Encode – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelper.encode
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Prevent Cross-Site Scripting (XSS) – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- MVC1000: Use of IHtmlHelper.Partial should be avoided – https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/mvc1000