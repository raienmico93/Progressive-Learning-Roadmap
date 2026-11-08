# ASP.NET Core MVC ModelState: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (controllers + Razor views) and **Razor Pages**, which share the same `ModelState` infrastructure. Examples use .NET 6–9 with minimal hosting (`Program.cs`). The core `ModelState` behaviour is stable across these versions. Web API controllers decorated with `[ApiController]` have special automatic 400 handling, which is flagged where relevant. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ModelState` is a dictionary-like object that represents the state of a model-binding and validation attempt, including all errors that occurred. |
| **Technical Definition** | `ModelState` is an instance of `ModelStateDictionary` (in `Microsoft.AspNetCore.Mvc.ModelBinding`). It is keyed by property or parameter name (with `""` for model-level errors) and stores `ModelStateEntry` objects, each containing a `ModelErrorCollection`. Errors originate from two subsystems: **model binding** (data-conversion errors such as `"x"` in an `int` field) and **model validation** (business-rule violations such as a rating of `0` when 1–5 is required). Both occur **before** the action or page handler executes. |
| **Beginner‑Friendly Explanation** | `ModelState` is the **error log for a form submission**. When a user submits data, model binding tries to convert the values and validation checks the business rules. Every problem—whether “that’s not a number” or “that number is out of range”—is written into `ModelState`. Your code then checks `ModelState` to decide whether to proceed or to show the errors back to the user. |

### Key Characteristics

- **Dual-source error tracking** – Captures both binding errors and validation errors in one place.
- **Dictionary-based** – Keyed by property/parameter name; supports field-level and model-level errors.
- **Automatically populated** – Binding and validation run implicitly; you do not manually add binding errors.
- **Extensible** – Custom errors can be added with `AddModelError`.
- **Framework-integrated** – Read by Tag Helpers (`asp-validation-for`), HTML Helpers (`ValidationMessageFor`), and `[ApiController]` filters.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- Basic understanding of **model binding** and **validation attributes**.
- Familiarity with **controllers/Razor Page handlers** and `ModelState.IsValid`.

### Related Programming Areas

- **Model binding** – Populates `ModelState` with conversion errors.
- **Model validation** – Adds business-rule errors to `ModelState`.
- **Razor views** – `asp-validation-for` and `ValidationSummary` render `ModelState` errors.
- **Web API `[ApiController]`** – Automatically returns HTTP 400 when `ModelState` is invalid.
- **Custom validation** – `IValidatableObject` and `ValidationAttribute` contribute errors to `ModelState`.

### Core Concepts / Features

1. **`ModelState`** – The dictionary object and its structure.
2. **`ModelState.IsValid`** – The boolean flag used to branch action logic.
3. **Model state errors** – The `ModelError` entries and how to inspect/add them.
4. **Model state best practices** – Security, correctness, and maintainability guidelines.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real‑World Cases**, and **References**.

---

## 1. `ModelState`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ModelState` is the container that holds the results—values and errors—of attempting to bind and validate a model from an HTTP request. |
| **Technical Definition** | `ModelState` is a property on `ControllerBase` and `PageModel` of type `ModelStateDictionary`. It implements `IReadOnlyDictionary<string, ModelStateEntry>`. Each entry corresponds to a property or parameter name (or `""` for model-level errors) and contains an `Errors` collection of `ModelError` objects, along with attempted and raw values. Binding errors and validation errors are both aggregated here before the action executes. |
| **Beginner‑Friendly Explanation** | `ModelState` is the **scorecard for the request’s data**. For every field, it records whether the value could be read, what the value was, and whether it passed the rules. If any field fails, `ModelState` holds the error message so the page can show it. |

### Purposes

- **To** provide a single, structured place to collect all binding and validation problems for a request.
- **To** enable action methods to decide whether to proceed or re-render the form with errors.
- **To** supply error data to view helpers (`asp-validation-for`, `ValidationMessageFor`, `ValidationSummary`) for user feedback.
- **To** support custom error injection (`AddModelError`) for rules not covered by attributes.
- **To** integrate with `[ApiController]`’s automatic 400 response generation.

### Syntax Rules and Structure

#### General Syntax (Accessing `ModelState`)

```csharp
// In a controller
ModelState.IsValid
ModelState["PropertyName"].Errors
ModelState.AddModelError("key", "message");

// In a Razor Page
ModelState.IsValid
ModelState.AddModelError("key", "message");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ModelState` | The `ModelStateDictionary` instance available on `Controller`/`PageModel`. |
| `IsValid` | `true` when **all** entries have zero errors. |
| `["key"]` | Access a specific `ModelStateEntry`; the key is the property or parameter name. |
| `AddModelError` | Manually add an error (e.g., for custom business rules). |

#### Structure of `ModelStateDictionary`

| Member | Purpose |
|--------|---------|
| `IsValid` | Aggregated validity flag. |
| `Keys` | All keys (property/parameter names). |
| `Values` | All `ModelStateEntry` objects. |
| `AddModelError(key, message)` | Adds a `ModelError` to the entry for `key`. |
| `ClearValidationState(modelName)` | Clears validation state for a model (used before re-validation). |
| `TryValidateModel(model)` | Re-runs validation on a model after modification. |

#### Syntax Rules

- `ModelState` is **populated automatically** during model binding and validation; you do not create it.
- Keys are **case-insensitive** in practice but conventionally match property names exactly.
- The empty string key (`""`) or `string.Empty` represents **model-level errors** not tied to a specific property.
- `AddModelError` throws `TooManyModelErrorsException` if the maximum error count is exceeded (default 200 in some configurations).

#### Constraints and Limitations

- **`ModelState` is per-request** – it is not persisted across requests.
- **Re-validation requires explicit clearing** – if you modify model values, call `ClearValidationState` followed by `TryValidateModel`.
- **`ModelState` is not a security boundary by itself** – you must still check `IsValid` before using the bound data.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Inspecting `ModelState` in a Controller

```csharp
[HttpPost]
public IActionResult Create(Movie movie)
{
    // Step 1: Check aggregate validity
    if (!ModelState.IsValid)
    {
        // Step 2: Optionally inspect specific errors
        var titleErrors = ModelState["Title"]?.Errors;
        var allErrors = ModelState.Values
            .SelectMany(v => v.Errors)
            .Select(e => e.ErrorMessage)
            .ToList();

        // Step 3: Re-render the form with errors
        return View(movie);
    }

    // Step 4: Proceed with valid data
    _context.Movies.Add(movie);
    _context.SaveChanges();
    return RedirectToAction(nameof(Index));
}
```

**Step‑by‑step explanation:**

1. `ModelState.IsValid` is `false` if any binding or validation rule failed.
2. Individual entries (`ModelState["Title"]`) expose the errors for that property; `ModelState.Values.SelectMany(v => v.Errors)` collects all errors across properties.
3. The view is re-rendered; Tag Helpers/HTML Helpers display the messages.
4. If valid, the entity is saved and the user is redirected (PRG pattern).

**Expected Output:** Invalid submissions re-display the form with error messages; valid submissions save and redirect.

**Why this output?** `ModelState` aggregates all errors from binding and validation, and the action inspects it to choose the correct response.

---

#### Example 2: Manually Adding a Custom Error

```csharp
[HttpPost]
public IActionResult Register(RegisterViewModel input)
{
    if (!ModelState.IsValid)
    {
        return View(input);
    }

    if (_userService.EmailExists(input.Email))
    {
        ModelState.AddModelError(nameof(input.Email),
            "An account with this email already exists.");
        return View(input);
    }

    // Proceed with registration
    return RedirectToAction("Welcome");
}
```

**Step‑by‑step explanation:**

1. Attribute-based validation runs automatically.
2. If attribute validation passes, a business-rule check queries the database.
3. If the email already exists, `AddModelError` adds an error keyed to the `Email` property.
4. The view displays the custom error alongside the field.

**Expected Output:** A duplicate email produces the custom error message under the Email field.

**Why this output?** `AddModelError` inserts a `ModelError` into the `ModelStateEntry` for `Email`, making `IsValid` `false` and providing the message for the view.

### Real‑World Cases

- **Registration forms:** Check username/email uniqueness and add errors to `ModelState`.
- **Checkout flows:** Verify inventory availability after binding and add model-level errors.
- **Admin CRUD:** Inspect `ModelState` to log detailed binding failures for diagnostics.

### References

- Model validation in ASP.NET Core MVC (Model state) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- `ModelStateDictionary` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstatedictionary

---

## 2. `ModelState.IsValid`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ModelState.IsValid` is a boolean property that is `true` only when every entry in `ModelState` has zero errors. |
| **Technical Definition** | `IsValid` is a computed property on `ModelStateDictionary` that returns `false` if **any** `ModelStateEntry` in the dictionary has a non‑empty `Errors` collection. It reflects the combined outcome of model binding (conversion errors) and model validation (attribute and `IValidatableObject` errors). Web apps are responsible for inspecting `ModelState.IsValid` and reacting appropriately; Web API controllers with `[ApiController]` do not need to check it because an automatic HTTP 400 response is generated. |
| **Beginner‑Friendly Explanation** | `ModelState.IsValid` answers one simple question: **“Did everything pass?”** If even one field has a problem—a missing required value, an invalid number, a failed custom rule—`IsValid` is `false`. If it is `true`, you can safely use the data. |

### Purposes

- **To** provide a single, reliable branch point for deciding whether to proceed with business logic.
- **To** prevent invalid or malformed data from reaching the database or domain layer.
- **To** trigger re-rendering of the form with error messages when validation fails.
- **To** act as the standard guard clause in MVC actions and Razor Page handlers.
- **To** integrate with `[ApiController]`’s automatic 400 response when not explicitly checked.

### Syntax Rules and Structure

#### General Syntax (Controller)

```csharp
[HttpPost]
public async Task<IActionResult> Create(Movie movie)
{
    if (!ModelState.IsValid)
    {
        return View(movie);   // Re-render with errors
    }

    // Proceed with valid data
    _context.Movies.Add(movie);
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `!ModelState.IsValid` | `true` when at least one error exists. |
| `return View(movie)` | Re-renders the form; validation messages are displayed. |
| `RedirectToAction` | PRG redirect after successful save. |

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

#### General Syntax (Web API with `[ApiController]`)

```csharp
[ApiController]
[Route("api/[controller]")]
public class MoviesController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(Movie movie)
    {
        // No explicit ModelState check needed;
        // invalid ModelState → automatic 400 response
        _context.Movies.Add(movie);
        _context.SaveChanges();
        return Ok(movie);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[ApiController]` | Enables automatic model validation and 400 response. |
| No `IsValid` check | The framework short-circuits invalid requests before the action body runs. |

#### Syntax Rules

- `IsValid` is **read‑only**; you cannot set it directly.
- It becomes `true` only when **all** entries have zero errors.
- **Re‑validation** – after modifying model values, call `ModelState.ClearValidationState(nameof(Model))` then `TryValidateModel(Model)` to refresh `IsValid`.
- **`[ApiController]`** – Web API controllers do not need an explicit `IsValid` check because the `ModelStateInvalidFilter` returns HTTP 400 automatically.

#### Constraints and Limitations

- **`IsValid` is not a substitute for authorization** – it only reflects data validity.
- **`IsValid` does not distinguish binding vs. validation errors** – both make it `false`.
- **Custom `IValidatableObject` validation runs after attribute validation** – `IsValid` reflects both.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Standard `IsValid` Guard Clause

```csharp
[HttpPost]
public IActionResult Create(ProductInputModel input)
{
    if (!ModelState.IsValid)
    {
        return View(input);   // Errors displayed by Tag Helpers
    }

    var product = new Product { Name = input.Name, Price = input.Price };
    _context.Products.Add(product);
    _context.SaveChanges();
    return RedirectToAction(nameof(Index));
}
```

**Step‑by‑step explanation:**

1. Model binding populates `input` and records any conversion errors.
2. Validation attributes run and add errors if rules fail.
3. `IsValid` is `false` → re-render the view; `asp-validation-for` shows messages.
4. `IsValid` is `true` → map to the entity, save, and redirect.

**Expected Output:** Invalid input re-displays the form with errors; valid input saves and redirects.

**Why this output?** `IsValid` aggregates all binding and validation results; the action branches on this single flag.

---

#### Example 2: Re‑validation After Modifying the Model

```csharp
public async Task<IActionResult> OnPostTryValidateAsync()
{
    var modifiedReleaseDate = DateTime.Now.Date;
    Movie.ReleaseDate = modifiedReleaseDate;

    ModelState.ClearValidationState(nameof(Movie));

    if (!TryValidateModel(Movie, nameof(Movie)))
    {
        return Page();
    }

    _context.Movies.Add(Movie);
    await _context.SaveChangesAsync();
    return RedirectToPage("./Index");
}
```

**Step‑by‑step explanation:**

1. A property is modified after initial binding.
2. `ClearValidationState` removes stale validation results for the model.
3. `TryValidateModel` re-runs validation with the new value and updates `ModelState`.
4. The refreshed `ModelState.IsValid` (via `TryValidateModel`’s return) is used to branch.

**Expected Output:** If the modified value violates a rule, validation fails and the page re-renders; otherwise, the entity is saved.

**Why this output?** Validation is automatic only at binding time; explicit re-validation is required after programmatic changes.

### Real‑World Cases

- **CRUD controllers:** The standard guard clause before database writes.
- **Razor Pages:** `OnPostAsync` checks `IsValid` before `SaveChangesAsync`.
- **Web APIs:** `[ApiController]` removes the need for explicit checks while still returning detailed 400 responses.

### References

- Model validation in ASP.NET Core MVC (`ModelState.IsValid`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Create web APIs with ASP.NET Core (Automatic HTTP 400 responses) – https://learn.microsoft.com/en-us/aspnet/core/web-api/

---

## 3. Model State Errors

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model state errors are the individual `ModelError` objects stored inside `ModelStateEntry` instances, each representing a binding failure or a validation rule violation. |
| **Technical Definition** | Each `ModelStateEntry` exposes an `Errors` property of type `ModelErrorCollection` (a list of `ModelError`). A `ModelError` contains an `ErrorMessage` string, an optional `Exception`, and optionally the `ModelMetadata` of the associated property. Errors are added automatically during binding/validation or manually via `ModelState.AddModelError(key, message)`. The maximum error count can trigger `TooManyModelErrorsException`. |
| **Beginner‑Friendly Explanation** | Model state errors are the **individual complaint messages** stored in `ModelState`. If the `Title` field is missing, that is one error; if the `Price` is out of range, that is another. You can read them to show the user exactly what went wrong, or add your own when a business rule fails. |

### Purposes

- **To** expose detailed, per-property error information for UI rendering and diagnostics.
- **To** allow custom business-rule errors to be injected alongside attribute-based errors.
- **To** support programmatic inspection of failures (logging, API error payloads).
- **To** enable field-level (`asp-validation-for`) and summary-level (`ValidationSummary`) error display.
- **To** provide structured error data for Web API 400 responses.

### Syntax Rules and Structure

#### General Syntax (Reading Errors)

```csharp
// All errors across all properties
var allErrors = ModelState.Values
    .SelectMany(v => v.Errors)
    .Select(e => e.ErrorMessage)
    .ToList();

// Errors for a specific property
var titleErrors = ModelState["Title"]?.Errors
    .Select(e => e.ErrorMessage)
    .ToList();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ModelState.Values` | All `ModelStateEntry` objects. |
| `.SelectMany(v => v.Errors)` | Flattens the `ModelErrorCollection` of each entry. |
| `e.ErrorMessage` | The human-readable error message. |
| `ModelState["Title"]` | The entry for the `Title` property. |

#### General Syntax (Adding Errors)

```csharp
ModelState.AddModelError("key", "Error message");
ModelState.AddModelError(nameof(Model.Property), "Error message");
ModelState.AddModelError(string.Empty, "Model-level error.");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `key` | Property/parameter name, or `""` for model-level. |
| `errorMessage` | The message to display. |

#### Structure of `ModelError`

| Member | Description |
|--------|-------------|
| `ErrorMessage` | The display message. |
| `Exception` | Optional underlying exception (for binding errors). |
| `ModelMetadata` | Optional metadata about the property. |

#### Syntax Rules

- **Binding errors** are added automatically; their `ErrorMessage` may be framework-generated (e.g., “The value 'x' is not valid for Age.”).
- **Validation attribute errors** use the `ErrorMessage` specified on the attribute or a default message.
- **`AddModelError`** should be used after checking existing errors to avoid duplicates.
- **`GetFieldValidationState`** (or `GetValidationState`) can inspect aggregate state for nested objects without direct indexer lookups.

#### Constraints and Limitations

- **Maximum error count** – adding too many errors throws `TooManyModelErrorsException`.
- **Invalid → valid transitions** – once a key is marked invalid, calling `MarkFieldValid` or `MarkFieldSkipped` throws `InvalidOperationException`.
- **`ModelState` errors are not automatically localized** – custom messages should be localized manually.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Reading and Displaying All Errors

```csharp
[HttpPost]
public IActionResult Diagnose(Movie movie)
{
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(kvp => kvp.Value.Errors.Count > 0)
            .Select(kvp => new
            {
                Field = kvp.Key,
                Messages = kvp.Value.Errors.Select(e => e.ErrorMessage)
            })
            .ToList();

        // Log or return errors for diagnostics
        return BadRequest(errors);
    }

    return Ok(movie);
}
```

**Step‑by‑step explanation:**

1. `ModelState` is filtered to entries with errors.
2. For each entry, the key (property name) and the list of `ErrorMessage` strings are projected.
3. The result is returned as a structured error payload.

**Expected Output:** A JSON array of objects like `{ "Field": "Price", "Messages": ["Price must be 1–100."] }`.

**Why this output?** `ModelStateEntry.Errors` exposes the `ModelErrorCollection`, and LINQ flattens and projects the messages.

---

#### Example 2: Adding a Model-Level Error

```csharp
[HttpPost]
public IActionResult Checkout(OrderInputModel input)
{
    if (!ModelState.IsValid)
    {
        return View(input);
    }

    if (!_inventoryService.AllItemsAvailable(input.Items))
    {
        ModelState.AddModelError(string.Empty,
            "One or more items are out of stock.");
        return View(input);
    }

    return RedirectToAction("Confirmation");
}
```

**Expected Output:** If inventory is insufficient, a model-level error appears in the validation summary (not attached to a single field).

**Why this output?** The empty-string key associates the error with the model as a whole, which `ValidationSummary` displays.

### Real‑World Cases

- **API error responses:** Return `ModelState` as a structured 400 payload (default with `[ApiController]`).
- **Custom business rules:** Add errors for inventory, uniqueness, or workflow-state violations.
- **Diagnostics:** Log flattened `ModelState` errors for failed requests.

### References

- `ModelStateDictionary.AddModelError` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstatedictionary.addmodelerror
- `ModelStateEntry.Errors` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstateentry.errors
- Reading validation errors in ModelState – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 4. Model State Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model state best practices are guidelines for using `ModelState` correctly, securely, and maintainably in ASP.NET Core MVC and Razor Pages applications. |
| **Technical Definition** | These practices cover mandatory validation checks, correct handling of binding vs. validation errors, safe custom error injection, proper re-validation after model changes, API-specific behaviour (`[ApiController]`), and avoidance of invalid state transitions in `ModelStateDictionary`. |
| **Beginner‑Friendly Explanation** | Best practices are the **“rules of thumb”** for `ModelState` — they help you avoid security holes, confusing behaviour, and bugs. For example, always check `IsValid` before saving data, and never treat a hidden field as a security measure. |

### Purposes

- **To** ensure invalid data never reaches business logic or the database.
- **To** avoid security vulnerabilities from missing validation checks.
- **To** handle Web API and MVC/Razor Pages differences correctly.
- **To** prevent runtime exceptions from invalid `ModelState` state transitions.
- **To** keep error reporting consistent and useful for users and API clients.

### Syntax Rules and Structure (Guideline Checklist)

| Area | Do | Don’t |
|------|----|-------|
| **Validation gate** | Always check `ModelState.IsValid` before executing business logic in MVC/Razor Pages. | Assume client-side validation is sufficient. |
| **API controllers** | Rely on `[ApiController]`’s automatic 400 for invalid state; omit explicit checks. | Add redundant `IsValid` checks in `[ApiController]` actions (harmless but unnecessary). |
| **Custom errors** | Use `AddModelError` with the correct property key or `string.Empty` for model-level errors. | Add duplicate errors for the same field without checking. |
| **Re-validation** | Call `ClearValidationState` then `TryValidateModel` after modifying model values. | Assume `IsValid` updates automatically after programmatic changes. |
| **Nested/complex models** | Use `GetFieldValidationState` to inspect aggregate state for nested objects. | Rely solely on direct indexer lookups for complex graphs. |
| **Error state transitions** | Transition fields from *Unvalidated* to *Valid* only when appropriate. | Call `MarkFieldValid` or `MarkFieldSkipped` on a field already marked *Invalid* (throws `InvalidOperationException`). |
| **Over-posting prevention** | Use ViewModels/DTOs; `ModelState` does not prevent over-posting. | Bind directly to entities with sensitive fields. |
| **Error messages** | Use clear, user-facing messages; avoid leaking stack traces or internal details. | Return raw exception messages in production API responses. |

### Annotated Code Example (Best-Practice Setup)

```csharp
[HttpPost]
public async Task<IActionResult> Create(ProductInputModel input)
{
    // 1. Mandatory validation gate
    if (!ModelState.IsValid)
    {
        return View(input);
    }

    // 2. Business-rule check; add custom error if needed
    if (await _productService.NameExistsAsync(input.Name))
    {
        ModelState.AddModelError(nameof(input.Name),
            "A product with this name already exists.");
        return View(input);
    }

    // 3. Map to entity and save
    var product = new Product { Name = input.Name, Price = input.Price };
    _context.Products.Add(product);
    await _context.SaveChangesAsync();

    // 4. PRG redirect
    return RedirectToAction(nameof(Index));
}
```

**Why this follows best practices:**

- **`ModelState.IsValid`** is checked before any business logic.
- **Custom business error** is added to the correct field key.
- **ViewModel** (`ProductInputModel`) prevents over-posting.
- **PRG pattern** avoids duplicate submissions.

### Real-World Cases

- **Public APIs:** `[ApiController]` automatically returns structured 400 responses; no manual `IsValid` check is needed.
- **Admin CRUD:** ViewModels + `ModelState` guard clauses prevent invalid data from reaching the database.
- **Complex forms:** `GetFieldValidationState` helps aggregate nested validation results.

### References

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Create web APIs with ASP.NET Core (Automatic HTTP 400 responses) – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- Security cards: Enforce Strict Input Validation and ModelState Checks – https://github.com/reware-labs/securitycards

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **`ModelState`** | Container for binding and validation results | `ModelState`, `ModelStateDictionary` | Every request with model binding |
| **`ModelState.IsValid`** | Single branch point for validity | `if (!ModelState.IsValid) { ... }` | Before business logic in MVC/Razor Pages |
| **Model state errors** | Detailed per-field error messages | `ModelState["Key"].Errors`, `AddModelError` | Custom rules, diagnostics, API payloads |
| **Best practices** | Security, correctness, maintainability | Guard clauses, `[ApiController]`, re-validation, DTOs | Every production application |

---

## References (Full List)

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- `ModelStateDictionary` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstatedictionary
- `ModelStateDictionary.AddModelError` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstatedictionary.addmodelerror
- `ModelStateEntry.Errors` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstateentry.errors
- Create web APIs with ASP.NET Core (Automatic HTTP 400 responses) – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- Security cards: Enforce Strict Input Validation and ModelState Checks – https://github.com/reware-labs/securitycards
- Model state (GitHub documentation source) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/models/validation/includes/validation6.md