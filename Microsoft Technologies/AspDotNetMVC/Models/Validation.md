# ASP.NET Core MVC Validation: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** and **Razor Pages** validation. Examples use .NET 6–9 with minimal hosting. Validation infrastructure is stable across these versions; version-specific changes (e.g., .NET 10 `Microsoft.Extensions.Validation`) are flagged where relevant. MVC and Razor Pages continue to use `System.ComponentModel.DataAnnotations` and `ModelState`; the newer `Microsoft.Extensions.Validation` package is **not supported** for MVC/Razor Pages and serves Blazor/Minimal APIs only.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Validation is the process of ensuring that user input conforms to the business rules and data-integrity requirements of an application before that data is processed or persisted. |
| **Technical Definition** | In ASP.NET Core MVC and Razor Pages, validation is a two-phase pipeline that runs during **model binding**: (1) **Model binding errors** (type-conversion failures) are recorded, and (2) **Model validation** evaluates data-annotation attributes and `IValidatableObject` implementations. Results are aggregated in **`ModelState`**. Controllers and page handlers inspect `ModelState.IsValid` to decide whether to proceed or re-render the form. Validation can execute **server-side only** or be mirrored **client-side** via jQuery Unobtrusive Validation using generated `data-val-*` attributes. |
| **Beginner-Friendly Explanation** | Validation is the **quality-control checkpoint** for your data. When a user submits a form, validation checks: “Is this field filled in? Is that number within range? Does this email look valid?” If anything fails, the app shows error messages and refuses to save the data. Without validation, bad data (empty names, negative prices, malformed emails) would enter your system. |

### Key Characteristics

- **Two-phase pipeline** – Binding errors (type conversion) and validation errors (business rules) are both captured in `ModelState`.
- **Declarative-first** – Rules are expressed with data-annotation attributes (`[Required]`, `[Range]`, etc.) on model properties.
- **Extensible** – Custom `ValidationAttribute` subclasses handle property-level rules; `IValidatableObject` handles cross-property rules.
- **Client-side mirroring** – Most built-in attributes emit `data-val-*` HTML attributes that jQuery Unobtrusive Validation interprets for instant browser feedback.
- **Server is authoritative** – Client-side validation is a UX optimization; server-side validation always runs and cannot be bypassed.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- `System.ComponentModel.DataAnnotations` (included in the shared framework).
- Basic understanding of **model binding** and **controllers/Razor Page handlers**.
- For client-side validation: jQuery and jQuery Unobtrusive Validation scripts (included in default MVC templates).

### Related Programming Areas

- **Model binding** – Populates model properties and records conversion errors in `ModelState`.
- **Data annotations** – Supply the declarative rules that validation evaluates.
- **Tag Helpers / HTML Helpers** – Render validation messages (`asp-validation-for`, `ValidationMessageFor`) and client-side `data-val-*` attributes.
- **Anti-forgery** – Protects POST actions from CSRF; complements validation in secure form handling.

### Core Concepts / Features

1. **Validation** – The overall pipeline and `ModelState`.
2. **Server-side validation** – Always-executed validation on the server.
3. **Client-side validation** – jQuery Unobtrusive Validation in the browser.
4. **Validation attributes** – Built-in data-annotation attributes.
5. **Custom validation** – Extending validation with custom logic.
6. **`IValidatableObject`** – Class-level, cross-property validation.
7. **`ValidationAttribute`** – Base class for custom property-level attributes.
8. **Validation best practices** – Security, UX, and maintainability guidelines.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. Validation (The Overall Pipeline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Validation is the end-to-end process of checking user input against rules and recording any violations so the application can respond appropriately. |
| **Technical Definition** | Validation in MVC/Razor Pages runs **after model binding** and **before** the action/handler executes. The `ObjectModelValidator` traverses the bound model, evaluates all `ValidationAttribute` instances and `IValidatableObject.Validate` calls, and writes `ModelError` entries into `ModelState`. Binding-time conversion errors are also stored in `ModelState`. The action/handler must check `ModelState.IsValid` and react (re-render, return 400, redirect, etc.). Web API controllers with `[ApiController]` automatically return HTTP 400 when `ModelState` is invalid. |
| **Beginner-Friendly Explanation** | Validation is the **whole checking process** from the moment a form is submitted to the moment your code decides whether to save the data. It catches both “this isn’t even a number” errors (binding) and “this number is out of range” errors (business rules), and collects them in one place (`ModelState`) so the page can show all problems at once. |

### Purposes

- **To** prevent invalid, malformed, or malicious data from entering the application’s business logic or database.
- **To** collect all validation problems in a single, structured container (`ModelState`) for unified error reporting.
- **To** provide a consistent validation experience across controllers, Razor Pages, and APIs.
- **To** enable both immediate client-side feedback and authoritative server-side enforcement.
- **To** integrate seamlessly with data annotations, custom rules, and UI helpers.

### Syntax Rules and Structure

#### General Syntax (Controller Check)

```csharp
[HttpPost]
public async Task<IActionResult> Create(Movie movie)
{
    if (!ModelState.IsValid)
    {
        return View(movie);   // Re-render with errors
    }

    _context.Movies.Add(movie);
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ModelState.IsValid` | `true` only if **all** binding and validation rules passed. |
| `return View(movie)` | Re-renders the form; Tag/HTML Helpers display `ModelState` errors. |
| `RedirectToAction` | PRG pattern after successful validation. |

#### General Syntax (Razor Page Handler)

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }
    // ...
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ModelState.IsValid` | Same semantics as in controllers. |
| `return Page()` | Re-renders the page with validation messages. |

#### Syntax Rules

- Validation runs **automatically**; you do not call a `Validate()` method manually.
- **Binding errors** (e.g., entering `"x"` in an `int` field) are added to `ModelState` before attribute validation runs.
- `ModelState` is a `ModelStateDictionary` keyed by property name (or `""` for model-level errors).

#### Constraints and Limitations

- **`[ApiController]` changes behaviour** – Web API controllers automatically return 400 without requiring an explicit `ModelState.IsValid` check.
- **Re-running validation** – If you modify model values after binding, call `ModelState.ClearValidationState` and `TryValidateModel` to re-evaluate.
- **Validation is synchronous** – `IValidatableObject.Validate` is synchronous; async validation in MVC/Razor Pages requires custom patterns.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: End-to-End Validation Flow in a Controller

```csharp
// Model with validation attributes
public class Movie
{
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Title { get; set; } = null!;

    [Range(0, 999.99)]
    public decimal Price { get; set; }
}

// Controller action
[HttpPost]
public async Task<IActionResult> Create(Movie movie)
{
    if (!ModelState.IsValid)          // Step 1: Check validation results
    {
        return View(movie);            // Step 2: Re-render form with errors
    }

    _context.Movies.Add(movie);        // Step 3: Persist valid data
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index)); // Step 4: PRG redirect
}
```

**Step-by-step explanation:**

1. Model binding populates `movie` and runs all validation attributes.
2. If any rule fails, `ModelState.IsValid` is `false`.
3. The view is re-rendered; `asp-validation-for`/`ValidationMessageFor` display errors.
4. If valid, the entity is saved and the user is redirected.

**Expected Output:** Invalid submissions re-display the form with error messages. Valid submissions save and redirect.

**Why this output?** `ModelState` aggregates both binding and validation errors; the action inspects it and branches accordingly.

### Real-World Cases

- **Registration forms:** Validate username uniqueness (custom), password strength (`[MinLength]`), and email format (`[EmailAddress]`).
- **E-commerce checkout:** Validate shipping address (`[Required]`), quantity (`[Range]`), and payment token (custom).
- **Admin CRUD:** Validate required fields and range constraints before database writes.

### References

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Validation in ASP.NET Core (overview) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation

---

## 2. Server-Side Validation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Server-side validation is validation that executes on the web server (in the ASP.NET Core process) and cannot be bypassed by the client. |
| **Technical Definition** | Server-side validation is performed by MVC’s `ObjectModelValidator` during model binding. It evaluates all `ValidationAttribute` instances on the bound model and any `IValidatableObject` implementations, populating `ModelState` with `ModelError` entries. It runs regardless of whether client-side validation is enabled or JavaScript is available. |
| **Beginner-Friendly Explanation** | Server-side validation is the **final, authoritative check** on the server. Even if a malicious user disables JavaScript or crafts a custom HTTP request, the server still validates every rule. Client-side validation is a convenience; server-side validation is the security guarantee. |

### Purposes

- **To** guarantee that no invalid data reaches business logic or the database, regardless of client behaviour.
- **To** enforce security-critical rules (e.g., authorization-dependent validation) that cannot be trusted to the browser.
- **To** serve as the single source of truth for validation outcomes (`ModelState`).
- **To** support scenarios where JavaScript is disabled or unavailable.
- **To** validate data submitted from non-browser clients (APIs, mobile apps, automated tests).

### Syntax Rules and Structure

#### General Syntax

Server-side validation is **implicit** — it runs automatically when a model is bound. The developer’s responsibility is to **check `ModelState.IsValid`** and respond appropriately.

```csharp
if (!ModelState.IsValid)
{
    return View(model);   // or Page(), or BadRequest(ModelState)
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ModelState.IsValid` | Aggregated result of all server-side validation. |
| `View(model)` / `Page()` | Re-render the form with error messages. |
| `BadRequest(ModelState)` | Return 400 with error details (APIs). |

#### Syntax Rules

- Server-side validation **always runs** during model binding; it does not require configuration.
- Client-side validation is **additive**; it never replaces server-side validation.
- For Web APIs with `[ApiController]`, `ModelState` is checked automatically and a 400 response is returned.

#### Constraints and Limitations

- **Synchronous** – `IValidatableObject.Validate` is synchronous; async server-side validation requires custom patterns or newer APIs (e.g., `IAsyncValidatableObject` in .NET 10+ for Minimal APIs, not MVC).
- **No automatic UI** – Server-side validation populates `ModelState`, but the developer must render messages via `asp-validation-for`/`ValidationMessageFor`.
- **Performance** – Complex custom validation runs on every request; optimize expensive checks.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Controller with Server-Side-Only Validation

```csharp
// Model: no client-side adapters needed for custom logic
public class Booking
{
    [Required]
    public DateTime CheckIn { get; set; }

    [Required]
    public DateTime CheckOut { get; set; }
}

// Controller action: always checks ModelState
[HttpPost]
public IActionResult Book(Booking booking)
{
    if (!ModelState.IsValid)
    {
        return View(booking);   // Server-side errors displayed
    }
    // Additional business validation
    if (booking.CheckOut <= booking.CheckIn)
    {
        ModelState.AddModelError(nameof(booking.CheckOut),
            "Check-out must be after check-in.");
        return View(booking);
    }
    return RedirectToAction("Confirmation");
}
```

**Step-by-step explanation:**

1. Model binding runs `[Required]` validation.
2. `ModelState.IsValid` is checked.
3. Additional cross-field validation is performed imperatively and added to `ModelState`.
4. The form is re-rendered with all errors.

**Expected Output:** Invalid dates produce server-side error messages even if JavaScript is disabled.

**Why this output?** All validation logic executes on the server; the browser cannot bypass it.

### Real-World Cases

- **Financial transactions:** Amount limits and authorization checks that must not be trusted to the client.
- **Healthcare data:** Mandatory fields and format rules for regulatory compliance.
- **APIs consumed by mobile apps:** No browser-based client validation is possible; server-side validation is the only defence.

### References

- Model validation in ASP.NET Core MVC (server-side pipeline) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation

---

## 3. Client-Side Validation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Client-side validation is validation that runs in the user’s browser using JavaScript before the form is submitted to the server. |
| **Technical Definition** | ASP.NET Core emits `data-val-*` attributes on form inputs based on validation attributes and client-side adapters. **jQuery Unobtrusive Validation** (a Microsoft add-on to the jQuery Validation plugin) reads these attributes and performs immediate validation, displaying error messages without a round-trip to the server. It is a UX optimization; server-side validation remains authoritative. |
| **Beginner-Friendly Explanation** | Client-side validation is **instant feedback in the browser**. If you type an invalid email, the page tells you immediately — without waiting for the server. It makes forms feel responsive, but it is **not** a security mechanism because users can disable JavaScript or modify requests. |

### Purposes

- **To** provide immediate feedback, reducing server round-trips and improving user experience.
- **To** catch simple errors (missing fields, wrong format) before form submission.
- **To** reduce server load by preventing obviously invalid submissions.
- **To** complement — never replace — server-side validation.

### Syntax Rules and Structure

#### General Syntax (Enabling Client-Side Validation)

```cshtml
@* In the view, include the validation scripts partial *@
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `_ValidationScriptsPartial` | Includes jQuery Validation and jQuery Unobtrusive Validation. |
| `data-val-*` attributes | Generated automatically by Tag/HTML Helpers from validation attributes. |

#### General Syntax (Generated HTML Example)

```html
<input type="text"
       data-val="true"
       data-val-required="The Title field is required."
       data-val-length-max="100"
       data-val-length-min="3"
       data-val-length="Title must be 3–100 characters."
       id="Title" name="Title" value="" />
```

**Component breakdown:**

| Attribute | Meaning |
|-----------|---------|
| `data-val="true"` | Marks the input for unobtrusive validation. |
| `data-val-required` | Error message for `[Required]`. |
| `data-val-length-*` | Parameters and message for `[StringLength]`. |

#### Syntax Rules

- Client-side validation requires **jQuery**, **jQuery Validation**, and **jQuery Unobtrusive Validation** scripts.
- The `_ValidationScriptsPartial` partial is included by default in MVC templates.
- Not all validation attributes have client-side adapters; custom rules require a custom adapter.

#### Constraints and Limitations

- **Never a security boundary** — client validation can be bypassed.
- **Custom attributes** need a JavaScript adapter (`$.validator.unobtrusive.adapters.add`) to participate.
- **`[Range]` with `DateTime`** has known limitations in jQuery validation; disable client-side date-range validation if problematic.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Built-in Client-Side Validation

```cshtml
@model Movie

<form asp-action="Create" method="post">
    <label asp-for="Title"></label>
    <input asp-for="Title" />
    <span asp-validation-for="Title"></span>

    <label asp-for="Price"></label>
    <input asp-for="Price" />
    <span asp-validation-for="Price"></span>

    <button type="submit">Save</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

**Step-by-step explanation:**

1. The Tag Helpers render `data-val-*` attributes based on `[Required]`, `[StringLength]`, and `[Range]`.
2. `_ValidationScriptsPartial` loads jQuery Unobtrusive Validation.
3. When the user tabs out of a field or submits, the browser validates immediately and displays messages.

**Expected Output:** Invalid input triggers browser-side error messages without a server round-trip.

**Why this output?** jQuery Unobtrusive Validation reads the `data-val-*` attributes and applies the corresponding rules.

---

#### Example 2: Custom Client-Side Adapter (Sketch)

```javascript
// Register a custom validation method
$.validator.addMethod('classicmovie', function (value, element, params) {
    // Custom logic
});

// Register an adapter that reads data-val-classicmovie-* attributes
$.validator.unobtrusive.adapters.add('classicmovie', ['year'], function (options) {
    options.rules['classicmovie'] = [/* params */];
    options.messages['classicmovie'] = options.message;
});
```

**Expected Output:** The custom rule participates in client-side validation alongside built-in rules.

**Why this output?** The adapter bridges `data-val-*` attributes to the jQuery Validation engine.

### Real-World Cases

- **Public-facing forms:** Instant feedback improves conversion and satisfaction.
- **Mobile-friendly forms:** Reduces round-trips on high-latency connections.
- **Multi-field forms:** Immediate validation helps users correct errors progressively.

### References

- Model validation in ASP.NET Core MVC (client-side validation) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- jQuery Unobtrusive Validation (GitHub) – https://github.com/aspnet/jquery-validation-unobtrusive
- jQuery Validation documentation – https://jqueryvalidation.org/documentation/

---

## 4. Validation Attributes

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Validation attributes are declarative .NET attributes that specify a rule a model property must satisfy. |
| **Technical Definition** | Validation attributes inherit from `ValidationAttribute` (in `System.ComponentModel.DataAnnotations`). MVC’s validator discovers them via reflection and invokes their `IsValid` method during model validation. Each attribute can supply an `ErrorMessage` and, where applicable, a client-side adapter that emits `data-val-*` attributes. |
| **Beginner-Friendly Explanation** | Validation attributes are **stickers you put on model properties** to say what rules apply. `[Required]` means “must be filled in”; `[Range(1, 100)]` means “must be between 1 and 100.” The framework reads these stickers and enforces them automatically. |

### Purposes

- **To** declare validation rules in a single, readable place (the model).
- **To** avoid duplicating validation logic across controllers and views.
- **To** automatically participate in both server-side and client-side validation.
- **To** provide standardised error messages that views can display.
- **To** integrate with scaffolding and UI generation.

### Syntax Rules and Structure

#### General Syntax

```csharp
[AttributeName(parameters, ErrorMessage = "...")]
public <Type> <PropertyName> { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[AttributeName]` | The validation attribute (e.g., `[Required]`, `[Range]`). |
| `parameters` | Positional arguments (e.g., `min`, `max` for `[Range]`). |
| `ErrorMessage` | Optional custom message. |

#### Common Built-in Validation Attributes

| Attribute | Purpose | Client-Side Adapter? |
|-----------|---------|----------------------|
| `[Required]` | Non-null value | Yes |
| `[StringLength(max, MinimumLength = min)]` | String length range | Yes |
| `[MaxLength(n)]` / `[MinLength(n)]` | Max/min length | Yes |
| `[Range(min, max)]` | Numeric/date range | Yes (with caveats for dates) |
| `[RegularExpression(pattern)]` | Regex match | Yes |
| `[EmailAddress]` | Email format | Yes |
| `[Phone]` | Phone format | Yes |
| `[Url]` | URL format | Yes |
| `[Compare(other)]` | Equality with another property | Yes |

#### Syntax Rules

- Attributes are applied **above** the property.
- Multiple attributes can be stacked.
- **Value types** (`int`, `decimal`, `DateTime`) are inherently required; `[Required]` is redundant.
- `ErrorMessage` supports `{0}` (property name), `{1}`, `{2}` placeholders for some attributes.

#### Constraints and Limitations

- **Built-in attributes cover common cases**; complex logic requires custom attributes or `IValidatableObject`.
- **`[Range]` with `DateTime`** may not work reliably with jQuery validation; prefer server-side or custom adapters.
- **Client-side adapters** are not available for every attribute; custom attributes need custom adapters.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Stacking Validation Attributes

```csharp
public class Product
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100, MinimumLength = 3,
        ErrorMessage = "Name must be 3–100 characters.")]
    public string Name { get; set; } = null!;

    [Range(1, 10000, ErrorMessage = "Price must be 1–10,000.")]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }

    [EmailAddress(ErrorMessage = "Invalid contact email.")]
    public string? ContactEmail { get; set; }
}
```

**Expected Output:** Empty name → “Name is required.” Short name → length error. Price outside range → range error. Bad email → email error.

**Why this output?** Each attribute contributes an independent rule; all failures are collected in `ModelState`.

### Real-World Cases

- **User registration:** `[Required]`, `[EmailAddress]`, `[StringLength]`, `[Compare]`.
- **Product catalogue:** `[Range]` for price, `[StringLength]` for descriptions.
- **Contact forms:** `[Phone]`, `[Url]`, `[RegularExpression]` for structured input.

### References

- Model validation in ASP.NET Core MVC (validation attributes) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- System.ComponentModel.DataAnnotations namespace – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations

---

## 5. Custom Validation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Custom validation is user-defined validation logic that extends the built-in attributes when built-in rules cannot express a business rule. |
| **Technical Definition** | Custom validation can be implemented in two primary ways: (1) subclassing `ValidationAttribute` and overriding `IsValid` for **property-level** rules, or (2) implementing `IValidatableObject` on the model class for **cross-property** rules. Custom attributes can optionally implement `IClientModelValidator` or use an `AttributeAdapterBase<T>` to participate in client-side validation. |
| **Beginner-Friendly Explanation** | Custom validation is **writing your own rule** when the built-in stickers aren’t enough. For example, “Classic movies must have a release year no later than 1960” isn’t covered by `[Range]`, so you write a `ClassicMovieAttribute` or implement `IValidatableObject`. |

### Purposes

- **To** enforce domain-specific rules that built-in attributes cannot express.
- **To** keep complex validation logic in a reusable, testable class.
- **To** optionally mirror custom rules on the client side via adapters.
- **To** support cross-property and conditional validation.

### Syntax Rules and Structure

#### General Syntax (Custom `ValidationAttribute`)

```csharp
public class CustomAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        // Return ValidationResult.Success or new ValidationResult("message")
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: ValidationAttribute` | Inherits the base validation class. |
| `IsValid` | Override; receives the property value and a `ValidationContext`. |
| `ValidationResult.Success` | Indicates the rule passed. |
| `new ValidationResult(message)` | Indicates failure. |

#### General Syntax (Custom Client Adapter)

Implement `IClientModelValidator` or derive an `AttributeAdapterBase<T>` to emit `data-val-*` attributes.

#### Syntax Rules

- `IsValid` must return `ValidationResult.Success` or a `ValidationResult` with an error message.
- `ValidationContext.ObjectInstance` provides access to the entire model (for cross-property logic).
- Custom attributes are discovered automatically when applied to properties.

#### Constraints and Limitations

- **Server-side only by default** — custom attributes run only on the server unless a client adapter is implemented.
- **Synchronous** — `IsValid` is synchronous; async validation is not supported in MVC/Razor Pages without custom patterns.
- **Attribute arguments must be constant expressions** — you cannot pass runtime-computed values to attribute constructors.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom `ValidationAttribute` for a Domain Rule

```csharp
public class ClassicMovieAttribute : ValidationAttribute
{
    public int Year { get; }

    public ClassicMovieAttribute(int year) => Year = year;

    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        var movie = (Movie)context.ObjectInstance;
        var releaseYear = ((DateTime)value!).Year;

        if (movie.Genre == Genre.Classic && releaseYear > Year)
        {
            return new ValidationResult(
                $"Classic movies must have a release year no later than {Year}.");
        }
        return ValidationResult.Success;
    }
}

// Usage
[ClassicMovie(1960)]
[DataType(DataType.Date)]
public DateTime ReleaseDate { get; set; }
```

**Step-by-step explanation:**

1. The attribute receives a threshold year in its constructor.
2. `IsValid` casts the value to `DateTime` and inspects the full `Movie` via `context.ObjectInstance`.
3. If the genre is Classic and the year exceeds the threshold, an error is returned.

**Expected Output:** A Classic movie released in 1970 produces the error “Classic movies must have a release year no later than 1960.”

**Why this output?** The custom attribute encodes a rule that depends on another property (`Genre`) and a constructor parameter, which built-in attributes cannot express.

#### Example 2: Custom Client Adapter (High-Level)

```javascript
$.validator.addMethod('classicmovie', function (value, element, params) {
    // Client-side equivalent of the server rule
});
$.validator.unobtrusive.adapters.add('classicmovie', ['year'], function (options) {
    options.rules['classicmovie'] = [/* genre element, year */];
    options.messages['classicmovie'] = options.message;
});
```

**Expected Output:** The custom rule runs in the browser alongside server-side validation.

**Why this output?** The adapter reads `data-val-classicmovie-*` attributes and integrates them with jQuery Validation.

### Real-World Cases

- **Age restrictions:** “Users must be at least 18” (cross-field with a date).
- **Business hours:** “Delivery time must be within business hours” (conditional on day of week).
- **Promo codes:** “Code must be valid and not expired” (server-side custom attribute).

### References

- Model validation in ASP.NET Core MVC (custom attributes) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- `ValidationAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute
- Custom client-side validation – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 6. `IValidatableObject`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IValidatableObject` is an interface that enables **class-level** (cross-property) validation logic on a model. |
| **Technical Definition** | `IValidatableObject` defines a single method, `Validate(ValidationContext)`, which returns `IEnumerable<ValidationResult>`. MVC calls this method **after** all property-level attribute validation has run, allowing the model to inspect its own property values and yield errors that reference specific properties or the model as a whole. |
| **Beginner-Friendly Explanation** | `IValidatableObject` lets the **model validate itself as a whole**. Instead of checking one property at a time, you can compare `StartDate` and `EndDate` and say “End must be after Start.” You implement `Validate` on the model class and yield any errors. |

### Purposes

- **To** express cross-property rules (e.g., end date > start date) that no single attribute can enforce.
- **To** keep class-level validation logic inside the model, close to the data it validates.
- **To** yield multiple errors, each optionally associated with specific property names.
- **To** complement property-level attributes without replacing them.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class MyModel : IValidatableObject
{
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (/* condition */)
        {
            yield return new ValidationResult(
                "Error message",
                new[] { nameof(PropertyName) });   // optional property association
        }
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IValidatableObject` | Interface implemented by the model class. |
| `Validate` | Method called after attribute validation. |
| `yield return new ValidationResult(...)` | Emits an error; the second argument optionally associates it with property names. |
| `ValidationResult.Success` | Not used here; an empty sequence means success. |

#### Syntax Rules

- `Validate` is called **after** property-level attribute validation.
- Return an **empty sequence** (or `yield break`) to indicate success.
- Multiple `ValidationResult` objects can be returned.
- The `memberNames` array associates errors with specific properties; `null` or empty means a model-level error.

#### Constraints and Limitations

- **Synchronous** — `Validate` cannot be `async`; it returns `IEnumerable<ValidationResult>`.
- **No client-side support** — `IValidatableObject` runs only on the server; there is no automatic client-side mirroring.
- **Attribute validation runs first** — if a property-level rule fails, `IValidatableObject.Validate` may still run (it is not skipped).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cross-Property Date Validation

```csharp
public class Booking : IValidatableObject
{
    [Required]
    public DateTime CheckIn { get; set; }

    [Required]
    public DateTime CheckOut { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (CheckOut <= CheckIn)
        {
            yield return new ValidationResult(
                "Check-out date must be after check-in date.",
                new[] { nameof(CheckOut) });
        }
    }
}
```

**Step-by-step explanation:**

1. `[Required]` validation runs for both properties.
2. `Validate` is called; it compares the two values.
3. If the condition fails, a `ValidationResult` is yielded and associated with `CheckOut`.

**Expected Output:** If `CheckOut` is before `CheckIn`, `ModelState` contains the error for `CheckOut`, and the view displays it next to that field.

**Why this output?** `IValidatableObject` enables cross-property logic, and the `memberNames` array ties the error to a specific field.

### Real-World Cases

- **Booking systems:** Check-out must be after check-in.
- **Password confirmation:** New password and confirm password must match (though `[Compare]` covers this for two fields).
- **Survey logic:** “If you selected ‘Other’, you must provide a comment.”

### References

- Model validation in ASP.NET Core MVC (`IValidatableObject`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- `IValidatableObject` API – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject

---

## 7. `ValidationAttribute`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ValidationAttribute` is the abstract base class for all validation attributes in .NET, including built-in and custom property-level rules. |
| **Technical Definition** | `ValidationAttribute` (in `System.ComponentModel.DataAnnotations`) defines the `IsValid` method and `ErrorMessage` property. Subclasses override `IsValid` to implement a rule. MVC’s validator discovers attributes inheriting from `ValidationAttribute` and invokes them. Custom attributes can additionally implement `IClientModelValidator` to generate `data-val-*` attributes for client-side validation. |
| **Beginner-Friendly Explanation** | `ValidationAttribute` is the **template for making your own validation stickers**. If you need a rule that isn’t built in, you create a class that inherits from `ValidationAttribute`, write the logic in `IsValid`, and then use your new attribute like any built-in one. |

### Purposes

- **To** provide a reusable, declarative way to express a property-level validation rule.
- **To** plug custom logic into the standard validation pipeline.
- **To** optionally extend the rule to client-side validation via adapters.
- **To** supply consistent error messages that integrate with `ModelState` and UI helpers.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class MyRuleAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        // Return ValidationResult.Success or new ValidationResult("message")
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: ValidationAttribute` | Inherits the base class. |
| `IsValid` | Must be overridden. |
| `value` | The property value being validated. |
| `context` | Provides access to `ObjectInstance`, `MemberName`, etc. |

#### Syntax Rules

- `IsValid` is `protected override`.
- Return `ValidationResult.Success` for valid input.
- Return `new ValidationResult(errorMessage)` for invalid input.
- The attribute can accept constructor parameters for configuration (e.g., `maxLength`).

#### Constraints and Limitations

- **Server-side only by default** — client-side support requires implementing `IClientModelValidator` or deriving an adapter.
- **Attribute arguments must be constant expressions** — no runtime-computed values in the constructor.
- **Synchronous** — `IsValid` cannot be async.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Reusable `ValidationAttribute`

```csharp
public class AllowedExtensionsAttribute : ValidationAttribute
{
    private readonly string[] _extensions;

    public AllowedExtensionsAttribute(params string[] extensions)
        => _extensions = extensions;

    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        if (value is IFormFile file)
        {
            var extension = Path.GetExtension(file.FileName).ToLowerInvariant();
            if (!_extensions.Contains(extension))
            {
                return new ValidationResult(
                    $"Only {string.Join(", ", _extensions)} files are allowed.");
            }
        }
        return ValidationResult.Success;
    }
}

// Usage
[AllowedExtensions(".jpg", ".png")]
public IFormFile? Photo { get; set; }
```

**Step-by-step explanation:**

1. The attribute constructor accepts allowed extensions.
2. `IsValid` checks the uploaded file’s extension.
3. If the extension is not allowed, a `ValidationResult` is returned.

**Expected Output:** Uploading a `.pdf` when only `.jpg`/`.png` are allowed produces the error message.

**Why this output?** The custom attribute inherits from `ValidationAttribute` and participates in the standard validation pipeline.

### Real-World Cases

- **File uploads:** Allowed extensions and size limits.
- **Domain-specific formats:** License plate numbers, ISBNs, national ID formats.
- **Business constraints:** “Price must be a multiple of 0.05.”

### References

- `ValidationAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute
- Custom validation attributes – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 8. Validation Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Validation best practices are guidelines that help developers implement validation that is secure, user-friendly, maintainable, and performant. |
| **Technical Definition** | Best practices span server/client parity, use of ViewModels to prevent over-posting, antiforgery integration, error-message design, performance of custom validation, and testing. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for validation — they help you avoid security holes, confusing error messages, and maintenance headaches. |

### Purposes

- **To** guarantee security by treating server-side validation as authoritative and never trusting the client alone.
- **To** improve UX by combining client-side immediacy with clear server-side messages.
- **To** prevent over-posting and mass-assignment attacks through ViewModels/DTOs.
- **To** keep validation logic DRY, testable, and maintainable.
- **To** integrate antiforgery protection with validation for secure POST handling.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Server-side authority** | Always check `ModelState.IsValid` on the server. | Rely solely on client-side validation. |
| **Client-side parity** | Include `_ValidationScriptsPartial` for UX; mirror custom rules with adapters. | Assume client validation is a security feature. |
| **ViewModels/DTOs** | Use dedicated input models with only the properties the user should edit. | Bind directly to entities containing sensitive fields (`IsAdmin`). |
| **Antiforgery** | Apply `[ValidateAntiForgeryToken]` / rely on default Razor Pages antiforgery. | Disable antiforgery without compelling reason. |
| **Error messages** | Use clear, actionable `ErrorMessage` strings with `{0}` placeholders. | Display raw exception messages or technical jargon. |
| **Custom validation** | Prefer `IValidatableObject` for cross-property rules; `ValidationAttribute` for reusable property rules. | Put cross-property logic in a single-property attribute. |
| **Performance** | Keep custom validation lightweight; cache expensive lookups. | Perform database calls inside `IsValid` without caching. |
| **Testing** | Unit-test custom attributes and `IValidatableObject` separately. | Test validation only through UI integration tests. |
| **Async limitations** | Be aware `IValidatableObject.Validate` is synchronous in MVC. | Attempt `async` inside `IValidatableObject` without a custom pattern. |

### Annotated Code Example (Best-Practice Validation Setup)

```csharp
// ViewModel with validation attributes and cross-property rule
public class RegisterInputModel : IValidatableObject
{
    [Required, StringLength(50, MinimumLength = 3)]
    [Display(Name = "Username")]
    public string Username { get; set; } = null!;

    [Required, EmailAddress]
    public string Email { get; set; } = null!;

    [Required, MinLength(8), DataType(DataType.Password)]
    public string Password { get; set; } = null!;

    [DataType(DataType.Password)]
    [Compare(nameof(Password), ErrorMessage = "Passwords do not match.")]
    public string ConfirmPassword { get; set; } = null!;

    public IEnumerable<ValidationResult> Validate(ValidationContext context)
    {
        if (Username.Equals(Email, StringComparison.OrdinalIgnoreCase))
        {
            yield return new ValidationResult(
                "Username and email must be different.",
                new[] { nameof(Username) });
        }
    }
}

// Controller
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Register(RegisterInputModel input)
{
    if (!ModelState.IsValid)
        return View(input);

    // ...
}
```

**Why this follows best practices:**

- **ViewModel** isolates input and prevents over-posting.
- **Built-in attributes** handle simple rules; **`IValidatableObject`** handles the cross-field rule.
- **`[ValidateAntiForgeryToken]`** secures the POST action.
- **`ModelState.IsValid`** is checked before any processing.
- **`ErrorMessage`** and `Compare` provide clear feedback.

### Real-World Cases

- **Public APIs:** DTOs with validation attributes; automatic 400 responses via `[ApiController]`.
- **Admin dashboards:** ViewModels hide audit fields; `IValidatableObject` enforces workflow rules.
- **High-traffic forms:** Client-side validation reduces server load, but server-side checks remain mandatory.

### References

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Validation in ASP.NET Core (overview) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Validation** | End-to-end checking pipeline | `ModelState.IsValid` | Every form/API submission |
| **Server-side validation** | Authoritative enforcement | Automatic during model binding | Always — cannot be disabled |
| **Client-side validation** | Immediate browser feedback | `_ValidationScriptsPartial`, `data-val-*` | UX optimization; requires JS |
| **Validation attributes** | Declarative property rules | `[Required]`, `[Range]`, etc. | Most common rules |
| **Custom validation** | Domain-specific rules | `ValidationAttribute` or `IValidatableObject` | Built-ins insufficient |
| **`IValidatableObject`** | Cross-property validation | `Validate(ValidationContext)` | Multi-field rules |
| **`ValidationAttribute`** | Base for reusable property rules | Override `IsValid` | Reusable custom rules |
| **Best practices** | Security, UX, maintainability | ViewModels, antiforgery, server checks | Every production app |

---

## References (Full List)

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Validation in ASP.NET Core (overview) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- `ValidationAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute
- `IValidatableObject` interface – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject
- jQuery Unobtrusive Validation (GitHub) – https://github.com/aspnet/jquery-validation-unobtrusive
- jQuery Validation documentation – https://jqueryvalidation.org/documentation/
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- `System.ComponentModel.DataAnnotations` namespace – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations