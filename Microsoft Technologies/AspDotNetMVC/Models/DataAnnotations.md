# ASP.NET Core MVC Data Annotations: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (controllers + Razor views) and applies equally to **Razor Pages**. Examples use .NET 6–9 with minimal hosting (`Program.cs`). Data annotations live primarily in `System.ComponentModel.DataAnnotations`; EF Core–specific mapping attributes live in `System.ComponentModel.DataAnnotations.Schema`. The `Microsoft.Extensions.Validation` package is **not supported** for MVC/Razor Pages validation — MVC continues to use `System.ComponentModel.DataAnnotations` and `ModelState`. Attributes that are EF Core–only are explicitly identified.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Data annotations are attributes applied to model classes and properties to declaratively specify validation rules, display formatting, and database‑mapping behaviour. |
| **Technical Definition** | Data annotations (`System.ComponentModel.DataAnnotations` and `.Schema`) are attribute classes read by MVC’s `ObjectModelValidator` during model binding and by EF Core during model configuration. **Validation attributes** inherit from `ValidationAttribute` and populate `ModelState`; **formatting attributes** (e.g., `[DataType]`, `[Display]`) supply metadata to views without enforcing rules; **mapping attributes** (e.g., `[Key]`, `[ForeignKey]`) configure the EF Core model. |
| **Beginner‑Friendly Explanation** | Data annotations are **rules and labels you stick onto your model properties**. Instead of writing `if (string.IsNullOrEmpty(Title))` in every controller, you write `[Required]` above `Title`, and ASP.NET Core enforces it everywhere. They also tell the view how to display a value and tell EF Core how to map it to a database column. |

### Key Characteristics

- **Declarative** – rules and metadata are expressed as attributes, not imperative code.
- **Dual‑purpose** – validation attributes feed `ModelState`; non‑validation attributes feed display, scaffolding, and EF Core mapping.
- **DRY by design** – rules are declared once on the model and enforced across MVC, Razor Pages, EF Core, and client‑side validation.
- **Client and server aware** – many validation attributes emit `data-val-*` attributes for jQuery unobtrusive validation alongside server‑side checks.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- `System.ComponentModel.DataAnnotations` (included in the shared framework).
- For EF Core attributes: `Microsoft.EntityFrameworkCore` package.
- For `[Column]` and `[ForeignKey]`: `System.ComponentModel.DataAnnotations.Schema`.

### Related Programming Areas

- **Model validation** – `ModelState.IsValid`, `ValidationMessageFor`, `ValidationSummary`.
- **Model binding** – metadata influences binding sources and behaviour.
- **Entity Framework Core** – mapping attributes configure keys, relationships, concurrency, and table/column mapping.
- **Razor views & scaffolding** – `DisplayFor`, `EditorFor`, and scaffolding tools read display and scaffolding attributes.

### Core Concepts / Features

The following 24 attributes are each treated as a distinct feature. They are grouped by purpose for navigation, but **each receives its own complete section** below.

| Group | Attributes |
|-------|------------|
| **Validation** | `[Required]`, `[StringLength]`, `[MaxLength]`, `[MinLength]`, `[Range]`, `[RegularExpression]`, `[EmailAddress]`, `[Phone]`, `[Url]`, `[Compare]` |
| **Display & Formatting** | `[DataType]`, `[Display]`, `[DisplayName]`, `[DisplayFormat]` |
| **Scaffolding & UI Hints** | `[ScaffoldColumn]`, `[HiddenInput]`, `[ReadOnly]`, `[Editable]` |
| **EF Core Relational & Mapping** | `[Key]`, `[ForeignKey]`, `[NotMapped]`, `[Timestamp]`, `[ConcurrencyCheck]` |
| **Best Practices** | Cross‑cutting guidelines |

---

## 1. Data Annotations (General Infrastructure)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Data annotations are a namespace of attributes that supply validation, formatting, and mapping metadata for model classes and properties. |
| **Technical Definition** | `System.ComponentModel.DataAnnotations` contains `ValidationAttribute` derivatives and formatting attributes. `System.ComponentModel.DataAnnotations.Schema` contains EF Core mapping attributes. MVC’s validation pipeline evaluates validation attributes during model binding; EF Core’s model builder reads mapping attributes to override conventions. |
| **Beginner‑Friendly Explanation** | Data annotations are **the toolbox of attributes** you use to describe your data. Some enforce rules (`[Required]`), some control display (`[Display]`), and some tell the database how to store the data (`[Key]`). |

### Purposes

- **To** centralise validation, display, and mapping rules on the model.
- **To** enable automatic client‑side and server‑side validation.
- **To** drive scaffolding and UI generation.
- **To** configure EF Core without fluent API code.
- **To** keep models readable and self‑documenting.

### Syntax Rules and Structure

```csharp
[AttributeName(parameter1, NamedProperty = value)]
public <Type> <PropertyName> { get; set; }
```

**Component breakdown:**

| Part | Description |
|------|-------------|
| `[AttributeName]` | The annotation attribute. |
| `parameter1` | Positional constructor argument. |
| `NamedProperty = value` | Optional named property (e.g., `ErrorMessage`). |

**Syntax rules:**

- Attributes are placed above the property or class.
- Multiple attributes can be combined on one property.
- `ErrorMessage` is available on all validation attributes.

**Constraints and limitations:**

- **Formatting attributes do not validate**; they only influence rendering.
- **Value types** (`int`, `decimal`, `DateTime`) are inherently required and do not need `[Required]`.
- **Non‑nullable reference types** in nullable‑enabled projects are treated as `[Required(AllowEmptyStrings = true)]` implicitly.

### Annotated Code Example

```csharp
using System.ComponentModel.DataAnnotations;

public class Movie
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Title is required.")]
    [StringLength(60, MinimumLength = 3)]
    public string? Title { get; set; }

    [Range(1, 100)]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }
}
```

**Expected Output:** Submitting an empty title produces the error “Title is required.” A price outside 1–100 is rejected. The `Price` field renders as a currency input.

**Why this output?** MVC evaluates each validation attribute during binding and writes failures to `ModelState`; `DataType` guides the view’s field template.

### Real‑World Cases

- **CRUD forms:** `[Required]` and `[StringLength]` on name fields.
- **Registration:** `[EmailAddress]`, `[Compare]` for password confirmation.
- **EF Core entities:** `[Key]` and `[ForeignKey]` to configure relationships.

### References

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Mapping attributes (aka data annotations) for relationships – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/mapping-attributes

---

## 2. `[Required]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Required]` specifies that a property must have a non‑null value. |
| **Technical Definition** | `RequiredAttribute` inherits from `ValidationAttribute`. On the server, a value is considered missing only if it is `null`; empty strings satisfy `[Required]` unless `AllowEmptyStrings = false`. Non‑nullable reference types are implicitly treated as `[Required(AllowEmptyStrings = true)]` in nullable contexts. |
| **Beginner‑Friendly Explanation** | `[Required]` means “this field cannot be left blank.” If the user submits the form without filling it in, validation fails. |

### Purposes

- **To** enforce that a field is present.
- **To** produce a clear error message when the field is null.
- **To** participate in client‑side validation via `data-val-required`.
- **To** influence EF Core schema (`NOT NULL` column).

### Syntax Rules and Structure

```csharp
[Required]
[Required(ErrorMessage = "Custom message.")]
[Required(AllowEmptyStrings = false)]
public string? Name { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ErrorMessage` | Custom validation message. |
| `AllowEmptyStrings` | If `false`, empty strings also fail (default `true` for `[Required]`). |

**Syntax rules:**

- Applies to reference types and nullable value types.
- Does **not** trim whitespace; whitespace‑only strings satisfy `[Required]` unless combined with `[RegularExpression]`.

**Constraints and limitations:**

- **Value types** are inherently required; `[Required]` on `int` is redundant.
- **Non‑nullable `string`** in nullable‑enabled projects is already treated as required implicitly; adding `[Required]` is explicit but not necessary.

### Annotated Code Example

```csharp
[Required(ErrorMessage = "Name is required.")]
public string? Name { get; set; }
```

**Expected Output:** Empty submission → `ModelState` error “Name is required.”

**Why this output?** `RequiredAttribute` returns `false` for `null`, and the `ErrorMessage` is used in `ModelState`.

### Real‑World Cases

- **Registration:** Username and password fields.
- **Checkout:** Shipping address line 1.
- **Admin forms:** Required category selection.

### References

- `[Required]` attribute – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#required-attribute

---

## 3. `[StringLength]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[StringLength]` limits the length of a string property. |
| **Technical Definition** | `StringLengthAttribute(maxLength)` sets the maximum length; `MinimumLength` sets the minimum. It is a validation attribute and does **not** affect the database column type by itself. |
| **Beginner‑Friendly Explanation** | `[StringLength]` says “this text must be between X and Y characters.” |

### Purposes

- **To** enforce both minimum and maximum string length.
- **To** provide a single attribute for length validation (unlike `[MaxLength]`/`[MinLength]`).
- **To** generate client‑side length validation.

### Syntax Rules and Structure

```csharp
[StringLength(maxLength)]
[StringLength(maxLength, MinimumLength = minLength, ErrorMessage = "...")]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `maxLength` | Maximum allowed characters (required). |
| `MinimumLength` | Optional minimum characters. |

**Syntax rules:**

- `MinimumLength` must be ≤ `maxLength`.
- `ErrorMessage` can use `{0}` (property name), `{1}` (max), `{2}` (min) placeholders.

**Constraints and limitations:**

- Does **not** trim whitespace.
- Does **not** affect the database column type; use `[MaxLength]` or `[Column]` for schema.

### Annotated Code Example

```csharp
[StringLength(60, MinimumLength = 3, ErrorMessage = "{0} must be {2}–{1} characters.")]
public string? Title { get; set; }
```

**Expected Output:** “Title must be 3–60 characters.”

**Why this output?** The placeholders are filled by `String.Format` with the property name, max, and min.

### Real‑World Cases

- **Movie titles:** 3–60 characters.
- **Usernames:** 4–20 characters.
- **Post codes:** exact length (combine with `MinimumLength = maxLength`).

### References

- `[StringLength]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#stringlength-attribute

---

## 4. `[MaxLength]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[MaxLength]` sets the maximum length of a string or array property. |
| **Technical Definition** | `MaxLengthAttribute` validates the maximum length and **also influences EF Core schema** (`nvarchar(maxLength)`). It is distinct from `[StringLength]` in that it does not support `MinimumLength` and directly affects database mapping. |
| **Beginner‑Friendly Explanation** | `[MaxLength]` says “this field cannot contain more than N characters.” It also tells the database to allocate only N characters. |

### Purposes

- **To** enforce a maximum length on strings and arrays.
- **To** configure the database column length in EF Core.
- **To** provide client‑side maxlength validation.

### Syntax Rules and Structure

```csharp
[MaxLength(100)]
public string? Description { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Length` | Maximum allowed length. |

**Syntax rules:**

- Works on `string` and arrays (`T[]`).
- No `MinimumLength` support.

**Constraints and limitations:**

- Does **not** enforce a minimum.
- For strings, `[StringLength]` is often preferred when a minimum is needed.

### Annotated Code Example

```csharp
[MaxLength(200)]
public string? Summary { get; set; }
```

**Expected Output:** Strings longer than 200 characters fail validation. EF Core maps the column to `nvarchar(200)`.

**Why this output?** `MaxLengthAttribute` validates length; EF Core reads it during model configuration.

### Real‑World Cases

- **SEO meta descriptions:** Max 160 characters.
- **Short codes:** Max 10 characters.
- **Array limits:** Max 5 tags.

### References

- `MaxLengthAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.maxlengthattribute

---

## 5. `[MinLength]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[MinLength]` sets the minimum length of a string or array property. |
| **Technical Definition** | `MinLengthAttribute` validates the minimum length. It does **not** affect EF Core schema by itself. |
| **Beginner‑Friendly Explanation** | `[MinLength]` says “this field must contain at least N characters.” |

### Purposes

- **To** enforce a minimum length on strings and arrays.
- **To** provide client‑side minlength validation.
- **To** complement `[MaxLength]` when both bounds are needed.

### Syntax Rules and Structure

```csharp
[MinLength(5)]
public string? Password { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Length` | Minimum allowed length. |

**Syntax rules:**

- Works on `string` and arrays.
- No maximum support.

**Constraints and limitations:**

- Does **not** enforce a maximum.
- Does **not** affect the database schema.

### Annotated Code Example

```csharp
[MinLength(8, ErrorMessage = "Password must be at least 8 characters.")]
public string? Password { get; set; }
```

**Expected Output:** A 5‑character password fails validation.

**Why this output?** `MinLengthAttribute` returns `false` when the length is below 8.

### Real‑World Cases

- **Passwords:** Minimum 8 characters.
- **Tags:** Minimum 2 characters.
- **Array selections:** At least one item.

### References

- `MinLengthAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.minlengthattribute

---

## 6. `[Range]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Range]` validates that a numeric or `DateTime` value falls within a specified range. |
| **Technical Definition** | `RangeAttribute(min, max)` checks inclusive bounds. It supports numeric types and `DateTime` (though `DateTime` with jQuery validation has known limitations). |
| **Beginner‑Friendly Explanation** | `[Range]` says “this number must be between X and Y.” |

### Purposes

- **To** enforce numeric bounds (e.g., price, age, quantity).
- **To** enforce date bounds (with caveats).
- **To** provide client‑side range validation for numeric types.

### Syntax Rules and Structure

```csharp
[Range(min, max)]
[Range(typeof(decimal), "0.01", "9999.99")]
[Range(1, 100, ErrorMessage = "Must be 1–100.")]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `min`, `max` | Inclusive bounds. |
| `typeof(...)` | Required for `decimal`/`DateTime` when using string bounds. |

**Syntax rules:**

- For `double`/`int`, simple literals work.
- For `decimal`, use `typeof(decimal)` with string bounds.
- For `DateTime`, use `typeof(DateTime)` with date strings — but jQuery validation may fail; disable client‑side date validation if needed.

**Constraints and limitations:**

- **`DateTime` range with jQuery validation is unreliable**; prefer server‑side validation or custom attributes.
- Bound values must be compile‑time constants.

### Annotated Code Example

```csharp
[Range(1, 100, ErrorMessage = "Price must be 1–100.")]
[DataType(DataType.Currency)]
public decimal Price { get; set; }
```

**Expected Output:** Price `0` or `101` fails; price `50` passes.

**Why this output?** `RangeAttribute` checks `min ≤ value ≤ max`.

### Real‑World Cases

- **Product price:** 1–10,000.
- **Age verification:** 18–120.
- **Quantity:** 1–99.

### References

- `[Range]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#range-attribute

---

## 7. `[RegularExpression]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[RegularExpression]` validates that a property value matches a specified .NET regular expression. |
| **Technical Definition** | `RegularExpressionAttribute(pattern)` uses `Regex.IsMatch` (anchored implicitly at start and end). It supports client‑side validation via `data-val-regex`. |
| **Beginner‑Friendly Explanation** | `[RegularExpression]` says “this field must follow a specific pattern,” such as “only letters” or “a valid postal code.” |

### Purposes

- **To** enforce character‑level format rules (e.g., only letters, digits, hyphens).
- **To** validate structured strings (postal codes, phone formats).
- **To** provide client‑side pattern validation.

### Syntax Rules and Structure

```csharp
[RegularExpression(@"pattern")]
[RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$", ErrorMessage = "Must start with a capital letter.")]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `pattern` | .NET regex; anchors are implicit. |

**Syntax rules:**

- Use verbatim strings (`@"..."`) for regex.
- The pattern is **not** anchored by default; `^` and `$` are added implicitly.

**Constraints and limitations:**

- **Complex regexes can cause ReDoS** (Regular Expression Denial of Service); keep patterns simple.
- Does **not** trim whitespace.

### Annotated Code Example

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$", ErrorMessage = "Genre must start with a capital letter.")]
[StringLength(30)]
public string? Genre { get; set; }
```

**Expected Output:** `"Sci-Fi"` fails (hyphen not allowed); `"Science Fiction"` passes.

**Why this output?** The regex allows uppercase first letter, then letters and spaces only.

### Real‑World Cases

- **Genre/rating:** Uppercase‑starting words.
- **Postal code:** `^\d{5}(-\d{4})?$`.
- **Username:** `^[a-zA-Z0-9_]{4,20}$`.

### References

- `[RegularExpression]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#regularexpression-attribute

---

## 8. `[EmailAddress]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[EmailAddress]` validates that a string resembles an email address. |
| **Technical Definition** | `EmailAddressAttribute` uses a simplified regex (`^[^@\s]+@[^@\s]+\.[^@\s]+$`). It validates format only, not deliverability. |
| **Beginner‑Friendly Explanation** | `[EmailAddress]` says “this must look like an email address” (something@something.something). |

### Purposes

- **To** enforce a basic email format.
- **To** provide client‑side email validation.
- **To** combine with `[Required]` for mandatory email fields.

### Syntax Rules and Structure

```csharp
[EmailAddress]
[EmailAddress(ErrorMessage = "Invalid email.")]
public string? Email { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ErrorMessage` | Custom message. |

**Syntax rules:**

- Does **not** require the string to be non‑null; combine with `[Required]` if mandatory.
- Does **not** check domain existence.

**Constraints and limitations:**

- Only format validation; `user@localhost` may pass or fail depending on the implementation.
- Internationalised email addresses may not be fully supported.

### Annotated Code Example

```csharp
[Required]
[EmailAddress(ErrorMessage = "Enter a valid email address.")]
public string? Email { get; set; }
```

**Expected Output:** `"not-an-email"` fails; `"user@example.com"` passes.

**Why this output?** The regex checks for `@` and a dot‑separated domain.

### Real‑World Cases

- **Registration forms:** Email field.
- **Newsletter signup:** Email address.
- **Contact forms:** Reply‑to email.

### References

- `[EmailAddress]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#emailaddress-attribute

---

## 9. `[Phone]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Phone]` validates that a string resembles a telephone number. |
| **Technical Definition** | `PhoneAttribute` uses a permissive regex that allows digits, spaces, parentheses, hyphens, and a leading `+`. It does **not** enforce a specific national format. |
| **Beginner‑Friendly Explanation** | `[Phone]` says “this must look like a phone number” — it allows common phone‑number characters. |

### Purposes

- **To** reject clearly non‑phone input.
- **To** allow flexible phone formats (international, local).
- **To** provide client‑side phone validation.

### Syntax Rules and Structure

```csharp
[Phone]
[Phone(ErrorMessage = "Invalid phone number.")]
public string? PhoneNumber { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ErrorMessage` | Custom message. |

**Syntax rules:**

- Does **not** enforce a country code or length.
- Combine with `[StringLength]` for maximum length.

**Constraints and limitations:**

- Very permissive; `"123"` may pass in some implementations.
- Does **not** validate against a real numbering plan.

### Annotated Code Example

```csharp
[Phone(ErrorMessage = "Enter a valid phone number.")]
[StringLength(20)]
public string? Phone { get; set; }
```

**Expected Output:** `"abc"` fails; `"+1 (555) 123-4567"` passes.

**Why this output?** The regex allows digits, spaces, parentheses, hyphens, and `+`.

### Real‑World Cases

- **Contact details:** Phone number fields.
- **Two‑factor authentication:** Phone number for SMS.
- **Customer profiles:** Contact phone.

### References

- `[Phone]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#phone-attribute

---

## 10. `[Url]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Url]` validates that a string is a well‑formed URL. |
| **Technical Definition** | `UrlAttribute` checks that the string starts with `http://`, `https://`, or `ftp://` and contains a valid host component. |
| **Beginner‑Friendly Explanation** | `[Url]` says “this must be a web address starting with http:// or https://.” |

### Purposes

- **To** enforce URL format for links and website fields.
- **To** prevent malformed URLs from being saved.
- **To** provide client‑side URL validation.

### Syntax Rules and Structure

```csharp
[Url]
[Url(ErrorMessage = "Enter a valid URL.")]
public string? Website { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ErrorMessage` | Custom message. |

**Syntax rules:**

- Requires a scheme (`http://`, `https://`, `ftp://`).
- Does **not** verify that the URL is reachable.

**Constraints and limitations:**

- Relative URLs and scheme‑less URLs (`www.example.com`) fail.
- Does **not** validate SSL certificates or DNS.

### Annotated Code Example

```csharp
[Url(ErrorMessage = "Website must be a valid URL.")]
public string? Website { get; set; }
```

**Expected Output:** `"example.com"` fails; `"https://example.com"` passes.

**Why this output?** The attribute requires an explicit scheme.

### Real‑World Cases

- **Company website:** URL field.
- **Social media links:** Profile URLs.
- **Portfolio:** Project links.

### References

- `[Url]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#url-attribute

---

## 11. `[Compare]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Compare]` validates that two properties have the same value. |
| **Technical Definition** | `CompareAttribute(otherProperty)` compares the annotated property’s value to another property on the same model. It is commonly used for password confirmation. |
| **Beginner‑Friendly Explanation** | `[Compare]` says “this field must match that other field,” like “Confirm Password must match Password.” |

### Purposes

- **To** ensure two fields are identical (password confirmation, email confirmation).
- **To** provide client‑side equality validation.
- **To** avoid duplicating comparison logic in controllers.

### Syntax Rules and Structure

```csharp
[Compare(nameof(OtherProperty))]
[Compare("Password", ErrorMessage = "Passwords do not match.")]
public string? ConfirmPassword { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `otherProperty` | Name of the property to compare against. |

**Syntax rules:**

- The other property must exist on the same model.
- Use `nameof` for compile‑time safety.

**Constraints and limitations:**

- Does **not** work across different models.
- Comparison is case‑sensitive by default.

### Annotated Code Example

```csharp
[Required, DataType(DataType.Password)]
public string? Password { get; set; }

[DataType(DataType.Password)]
[Compare(nameof(Password), ErrorMessage = "Passwords do not match.")]
public string? ConfirmPassword { get; set; }
```

**Expected Output:** Mismatched values → error “Passwords do not match.”

**Why this output?** `CompareAttribute` compares the two bound values during validation.

### Real‑World Cases

- **Registration:** Password + Confirm Password.
- **Email change:** New email + Confirm email.
- **Account settings:** New password + Confirm new password.

### References

- `[Compare]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#compare-attribute

---

## 12. `[DataType]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[DataType]` specifies the semantic type of a property (e.g., Date, Currency, EmailAddress, Password) so views can choose an appropriate rendering template. |
| **Technical Definition** | `DataTypeAttribute(DataType enum)` is a **formatting** attribute, not a validation attribute. It guides MVC’s field template selection and enables HTML5 input types where applicable. |
| **Beginner‑Friendly Explanation** | `[DataType]` tells the view “this is a date” or “this is currency,” so the view can show a date picker or a currency‑formatted input. |

### Purposes

- **To** enable the correct field template (date picker, password mask, currency formatting).
- **To** activate HTML5 input types (`type="date"`, `type="email"`, `type="password"`).
- **To** convey semantic meaning without enforcing validation.

### Syntax Rules and Structure

```csharp
[DataType(DataType.Date)]
[DataType(DataType.Currency)]
[DataType(DataType.Password)]
[DataType(DataType.MultilineText)]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DataType` | Enum: `Date`, `DateTime`, `Currency`, `EmailAddress`, `Password`, `Url`, `PhoneNumber`, `MultilineText`, etc. |

**Syntax rules:**

- Does **not** validate; use validation attributes for rules.
- Often combined with `[DisplayFormat]` for precise formatting.

**Constraints and limitations:**

- Without `[DataType]`, `[DisplayFormat]` alone uses the string template and may not produce a date picker.
- Browser support for HTML5 input types varies.

### Annotated Code Example

```csharp
[DataType(DataType.Date)]
[Display(Name = "Release Date")]
public DateTime ReleaseDate { get; set; }

[DataType(DataType.Currency)]
public decimal Price { get; set; }
```

**Expected Output:** `ReleaseDate` renders with a date picker; `Price` renders with currency formatting.

**Why this output?** MVC’s `EditorFor` reads `DataType` and selects the appropriate template.

### Real‑World Cases

- **Dates:** Date pickers for release dates, birth dates.
- **Passwords:** Masked inputs.
- **Currency:** Formatted price inputs.
- **Multiline text:** Textareas for descriptions.

### References

- `[DataType]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#datatype-attribute

---

## 13. `[Display]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Display]` provides display metadata: name, description, group, and order. |
| **Technical Definition** | `DisplayAttribute` sets `Name`, `Description`, `GroupName`, `Order`, and `Prompt`. It is read by `DisplayNameFor`, `LabelFor`, and scaffolding. |
| **Beginner‑Friendly Explanation** | `[Display]` controls the **label text** and **order** of a field in the UI. |

### Purposes

- **To** set a human‑readable label (`Name`).
- **To** provide tooltip/description text.
- **To** control field order in generated UI.
- **To** group related fields.

### Syntax Rules and Structure

```csharp
[Display(Name = "Release Date", Order = 2, Description = "The date the movie was released.")]
public DateTime ReleaseDate { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Name` | Label text. |
| `Order` | Sort order (lower appears first). |
| `Description` | Tooltip/help text. |
| `GroupName` | Grouping for UI. |

**Syntax rules:**

- `Name` overrides the default property‑name‑derived label.
- `Order` affects scaffolding and `DisplayFor` ordering.

**Constraints and limitations:**

- Does **not** validate.
- Does **not** format values (use `[DisplayFormat]` or `[DataType]`).

### Annotated Code Example

```csharp
[Display(Name = "Release Date", Order = 1)]
[DataType(DataType.Date)]
public DateTime ReleaseDate { get; set; }

[Display(Name = "Price", Order = 2)]
[Range(1, 100)]
public decimal Price { get; set; }
```

**Expected Output:** Labels read “Release Date” and “Price,” with Release Date appearing first.

**Why this output?** `DisplayNameFor` reads `Name`; `Order` influences generated form layout.

### Real‑World Cases

- **Admin panels:** Friendly field labels.
- **Ordered forms:** Field sequence control.
- **Tooltips:** Help text for complex fields.

### References

- `DisplayAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.displayattribute

---

## 14. `[DisplayName]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[DisplayName]` sets a simple display name for a property or class. |
| **Technical Definition** | `DisplayNameAttribute(name)` is a lightweight alternative to `[Display]` for changing the label text. It does **not** support `Order`, `Description`, or `GroupName`. |
| **Beginner‑Friendly Explanation** | `[DisplayName]` is a quick way to change a field’s label without using the fuller `[Display]` attribute. |

### Purposes

- **To** set a human‑readable name with minimal syntax.
- **To** override the default property‑name‑derived label.
- **To** work in contexts where `[Display]` is not available or overkill.

### Syntax Rules and Structure

```csharp
[DisplayName("Release Date")]
public DateTime ReleaseDate { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `displayName` | The label text. |

**Syntax rules:**

- Positional constructor argument.
- Cannot set `Order` or `Description`.

**Constraints and limitations:**

- Does **not** support the additional metadata of `[Display]`.
- If both `[Display]` and `[DisplayName]` are present, `[Display]` takes precedence.

### Annotated Code Example

```csharp
[DisplayName("Release Date")]
[DataType(DataType.Date)]
public DateTime ReleaseDate { get; set; }
```

**Expected Output:** Label reads “Release Date.”

**Why this output?** `DisplayNameFor` uses the `DisplayName` value.

### Real‑World Cases

- **Quick label changes:** Simple renaming.
- **Legacy codebases:** Where `[DisplayName]` is already used.
- **Minimal models:** Where full `[Display]` metadata is unnecessary.

### References

- `DisplayNameAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.displaynameattribute

---

## 15. `[DisplayFormat]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[DisplayFormat]` specifies how a property value is formatted for display and editing. |
| **Technical Definition** | `DisplayFormatAttribute` sets `DataFormatString`, `ApplyFormatInEditMode`, `NullDisplayText`, and `ConvertEmptyStringToNull`. It works with `DisplayFor` and `EditorFor`; it is a **formatting** attribute, not a validation attribute. |
| **Beginner‑Friendly Explanation** | `[DisplayFormat]` says “show this date as `yyyy-MM-dd`” or “show this price with a currency symbol.” |

### Purposes

- **To** control the exact display format (dates, numbers, currency).
- **To** apply formatting in edit mode (`ApplyFormatInEditMode`).
- **To** specify text for null values (`NullDisplayText`).
- **To** convert empty strings to null (`ConvertEmptyStringToNull`).

### Syntax Rules and Structure

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
[DisplayFormat(NullDisplayText = "N/A")]
public DateTime? ReleaseDate { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DataFormatString` | Composite format string (`{0:...}`). |
| `ApplyFormatInEditMode` | Whether to format in edit mode (default `false`). |
| `NullDisplayText` | Text shown when the value is `null`. |
| `ConvertEmptyStringToNull` | Whether empty strings become `null`. |

**Syntax rules:**

- Use `{0:format}` syntax.
- Often combined with `[DataType]` for best results.

**Constraints and limitations:**

- Without `[DataType]`, `DisplayFormat` uses the string template and may not produce the desired HTML5 input type.
- `ApplyFormatInEditMode = true` may not be desirable for currency symbols in editable fields.

### Annotated Code Example

```csharp
[DataType(DataType.Date)]
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
```

**Expected Output:** Displays and edits `ReleaseDate` as `2025-03-15`.

**Why this output?** `DisplayFormat` applies the composite format string when rendering.

### Real‑World Cases

- **Dates:** ISO format display.
- **Currency:** Two decimal places.
- **Null values:** “Not specified” placeholder.

### References

- `[DisplayFormat]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation#displayformat-attribute

---

## 16. `[ScaffoldColumn]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ScaffoldColumn]` specifies whether a property is included when scaffolding generates UI. |
| **Technical Definition** | `ScaffoldColumnAttribute(bool)` is read by scaffolding tools. `[ScaffoldColumn(false)]` hides the property from generated views. |
| **Beginner‑Friendly Explanation** | `[ScaffoldColumn(false)]` tells the scaffolding tool “don’t create a form field for this property.” |

### Purposes

- **To** hide sensitive or internal properties from generated CRUD UI.
- **To** prevent scaffolding of computed or navigation properties.
- **To** control which columns appear in scaffolded views.

### Syntax Rules and Structure

```csharp
[ScaffoldColumn(false)]
public string? InternalNotes { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `scaffold` | `true` (default) or `false`. |

**Syntax rules:**

- Applies to properties.
- Only affects scaffolding, not validation or binding.

**Constraints and limitations:**

- **Does not** prevent model binding; use `[Bind]` or a ViewModel for security.
- Scaffolding is a one‑time code generation; manual views ignore it.

### Annotated Code Example

```csharp
public class Product
{
    public int Id { get; set; }
    public string? Name { get; set; }

    [ScaffoldColumn(false)]
    public DateTime CreatedAt { get; set; }
}
```

**Expected Output:** Scaffolded `Create`/`Edit` views do **not** include `CreatedAt`.

**Why this output?** The scaffolder checks `ScaffoldColumnAttribute` and skips the property.

### Real‑World Cases

- **Audit fields:** Hide `CreatedAt`/`ModifiedAt`.
- **Internal flags:** Hide `IsDeleted`.
- **Computed properties:** Hide `FullName` if derived.

### References

- `ScaffoldColumnAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.scaffoldcolumnattribute

---

## 17. `[HiddenInput]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[HiddenInput]` renders a property as a hidden input in generated views. |
| **Technical Definition** | `HiddenInputAttribute` sets `DisplayValue` (whether to render the value) and is read by `EditorFor`/scaffolding to produce `<input type="hidden">`. |
| **Beginner‑Friendly Explanation** | `[HiddenInput]` says “store this value in the form, but don’t show it to the user.” |

### Purposes

- **To** render ID fields as hidden inputs.
- **To** include concurrency tokens in forms without user interaction.
- **To** preserve values across round‑trips.

### Syntax Rules and Structure

```csharp
[HiddenInput(DisplayValue = false)]
public int Id { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DisplayValue` | Whether to render the value in display views (default `true`). |

**Syntax rules:**

- Typically used with `EditorFor` or scaffolding.
- `DisplayValue = false` hides the value in display views but still renders the hidden input in edit views.

**Constraints and limitations:**

- **Not a security mechanism** — hidden values are visible in page source.
- Does **not** replace explicit `Html.HiddenFor`.

### Annotated Code Example

```csharp
[HiddenInput(DisplayValue = false)]
public int Id { get; set; }
```

**Expected Output:** `EditorFor(m => m.Id)` renders `<input type="hidden" ...>`.

**Why this output?** `HiddenInputAttribute` influences the editor template selection.

### Real‑World Cases

- **Edit forms:** Preserve entity ID.
- **Concurrency:** Include `RowVersion` as hidden.
- **Wizard steps:** Carry state between pages.

### References

- `HiddenInputAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.hiddeninputattribute

---

## 18. `[ReadOnly]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ReadOnly]` indicates that a property should be treated as read‑only in the UI. |
| **Technical Definition** | `ReadOnlyAttribute(bool)` is a metadata attribute read by some UI frameworks and custom code; MVC’s `EditorFor` may render a read‑only input, but the property is **still bound** unless explicitly prevented. |
| **Beginner‑Friendly Explanation** | `[ReadOnly]` says “show this value but don’t let the user change it.” |

### Purposes

- **To** display computed or system‑generated values without allowing edits.
- **To** mark fields as non‑editable in generated UI.
- **To** signal intent to custom view logic.

### Syntax Rules and Structure

```csharp
[ReadOnly(true)]
public DateTime CreatedAt { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `isReadOnly` | `true` or `false`. |

**Syntax rules:**

- Metadata only; does **not** prevent model binding.

**Constraints and limitations:**

- **Not a security boundary** — clients can still post values unless binding is restricted.
- MVC’s default templates may not honour `[ReadOnly]` consistently.

### Annotated Code Example

```csharp
[ReadOnly(true)]
public string? SystemCode { get; set; }
```

**Expected Output:** Custom view logic can render `SystemCode` as read‑only.

**Why this output?** The attribute provides metadata for UI frameworks.

### Real‑World Cases

- **Audit fields:** Display creation timestamps read‑only.
- **System codes:** Show generated codes without edits.
- **Calculated fields:** Display totals read‑only.

### References

- `ReadOnlyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.readonlyattribute

---

## 19. `[Editable]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Editable]` specifies whether a property can be edited in the UI. |
| **Technical Definition** | `EditableAttribute(bool)` controls editability for UI generation and is read by `EditorFor`. `[Editable(false)]` makes the property non‑editable. |
| **Beginner‑Friendly Explanation** | `[Editable(false)]` says “this field cannot be changed by the user.” |

### Purposes

- **To** prevent editing of certain properties in generated forms.
- **To** mark properties as display‑only.
- **To** complement `[ReadOnly]` with a clearer intent.

### Syntax Rules and Structure

```csharp
[Editable(false)]
public string? ReferenceNumber { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `allowEdit` | `false` to disable editing. |

**Syntax rules:**

- UI metadata only; does **not** prevent binding.

**Constraints and limitations:**

- **Not a security mechanism**; use ViewModels/DTOs to control binding.
- Behaviour depends on the view template.

### Annotated Code Example

```csharp
[Editable(false)]
public string? InvoiceNumber { get; set; }
```

**Expected Output:** `EditorFor` renders a read‑only display for `InvoiceNumber`.

**Why this output?** `EditableAttribute` metadata influences editor template behaviour.

### Real‑World Cases

- **Invoice numbers:** System‑generated, non‑editable.
- **Created by:** Audit field.
- **Status:** Workflow‑controlled field.

### References

- `EditableAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.editableattribute

---

## 20. `[Key]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Key]` explicitly marks a property as the primary key for an EF Core entity. |
| **Technical Definition** | `KeyAttribute` (in `System.ComponentModel.DataAnnotations.Schema`) overrides EF Core’s convention (`Id` or `<Type>Id`). It is used when the key property has a non‑conventional name. |
| **Beginner‑Friendly Explanation** | `[Key]` tells the database “this column uniquely identifies each row.” |

### Purposes

- **To** designate a primary key with a non‑standard name.
- **To** configure composite keys (multiple `[Key]` properties).
- **To** override EF Core conventions.

### Syntax Rules and Structure

```csharp
[Key]
public int MovieId { get; set; }        // Non-conventional key name
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| No parameters | Marks the property as the key. |

**Syntax rules:**

- One `[Key]` per entity for a single key; multiple `[Key]` for a composite key.
- Key properties are required by EF Core.

**Constraints and limitations:**

- EF Core conventions already recognise `Id` and `<Type>Id`; `[Key]` is only needed for other names.
- Does **not** validate uniqueness at the MVC layer.

### Annotated Code Example

```csharp
public class Movie
{
    [Key]
    public int FilmId { get; set; }     // Convention would look for Id/MovieId

    public string? Title { get; set; }
}
```

**Expected Output:** EF Core treats `FilmId` as the primary key.

**Why this output?** `KeyAttribute` overrides the convention.

### Real‑World Cases

- **Legacy databases:** Non‑standard key names.
- **Composite keys:** Order lines with `OrderId` + `ProductId`.
- **Natural keys:** ISBN, SKU.

### References

- `KeyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.keyattribute

---

## 21. `[ForeignKey]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ForeignKey]` specifies which property is the foreign key for a navigation property. |
| **Technical Definition** | `ForeignKeyAttribute(name)` (in `.Schema`) tells EF Core which property holds the FK value for a navigation. It is used when the FK property name does not follow convention (`<Navigation>Id`). |
| **Beginner‑Friendly Explanation** | `[ForeignKey]` says “this property is the link to another table.” |

### Purposes

- **To** configure relationships when the FK property has a non‑conventional name.
- **To** disambiguate multiple navigation properties to the same entity.
- **To** override EF Core conventions.

### Syntax Rules and Structure

```csharp
[ForeignKey(nameof(Genre))]
public int GenreId { get; set; }
public Genre? Genre { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `name` | Name of the FK property or navigation property. |

**Syntax rules:**

- Can be placed on the FK property or the navigation property.
- Use `nameof` for compile‑time safety.

**Constraints and limitations:**

- Overkill when convention already works (`GenreId`).
- Does **not** enforce referential integrity at the MVC layer.

### Annotated Code Example

```csharp
public class Movie
{
    public int Id { get; set; }

    [ForeignKey(nameof(Director))]
    public int DirectorId { get; set; }

    public Director? Director { get; set; }
}
```

**Expected Output:** EF Core configures `DirectorId` as the FK to `Director`.

**Why this output?** `ForeignKeyAttribute` explicitly links the property.

### Real‑World Cases

- **Multiple relationships:** `BillingAddressId` and `ShippingAddressId` to `Address`.
- **Legacy schemas:** Non‑conventional FK names.
- **Explicit configuration:** Clarity in complex models.

### References

- `ForeignKeyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.foreignkeyattribute

---

## 22. `[NotMapped]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[NotMapped]` excludes a property from EF Core’s database mapping. |
| **Technical Definition** | `NotMappedAttribute` (in `.Schema`) tells EF Core to ignore the property when creating the model and migrations. |
| **Beginner‑Friendly Explanation** | `[NotMapped]` says “this property is for the app only — don’t create a database column for it.” |

### Purposes

- **To** exclude computed or UI‑only properties from the database.
- **To** avoid unwanted columns in migrations.
- **To** keep the domain model clean.

### Syntax Rules and Structure

```csharp
[NotMapped]
public string? FullName => $"{FirstName} {LastName}";
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| No parameters | Excludes the property. |

**Syntax rules:**

- Applies to properties.
- Also excludes from validation if it is a computed property.

**Constraints and limitations:**

- Does **not** affect MVC binding; the property can still be bound if present in a form.
- Computed properties should typically be get‑only.

### Annotated Code Example

```csharp
public class Customer
{
    public int Id { get; set; }
    public string? FirstName { get; set; }
    public string? LastName { get; set; }

    [NotMapped]
    public string FullName => $"{FirstName} {LastName}";
}
```

**Expected Output:** EF Core does **not** create a `FullName` column.

**Why this output?** `NotMappedAttribute` excludes the property from the model.

### Real‑World Cases

- **Computed properties:** `FullName`, `Age`.
- **UI‑only flags:** `IsSelected` for checkboxes.
- **Temporary data:** Search filters.

### References

- `NotMappedAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.notmappedattribute

---

## 23. `[Timestamp]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Timestamp]` marks a property as a row version / concurrency token. |
| **Technical Definition** | `TimestampAttribute` (in `.Schema`) configures the property as a concurrency token. In SQL Server, this maps to `rowversion`; EF Core includes the column in the `WHERE` clause of `UPDATE` and `DELETE` statements. |
| **Beginner‑Friendly Explanation** | `[Timestamp]` says “use this hidden value to detect if someone else changed the row while I was editing it.” |

### Purposes

- **To** implement optimistic concurrency.
- **To** detect conflicting updates.
- **To** avoid lost updates in multi‑user scenarios.

### Syntax Rules and Structure

```csharp
[Timestamp]
public byte[]? RowVersion { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| No parameters | Marks the property as a concurrency token. |

**Syntax rules:**

- Property type is typically `byte[]` (SQL Server `rowversion`).
- Only **one** `[Timestamp]` property per entity.

**Constraints and limitations:**

- **Database‑specific:** SQL Server uses `rowversion`; other providers may behave differently.
- Must be configured correctly in migrations.

### Annotated Code Example

```csharp
public class Department
{
    public int Id { get; set; }
    public string? Name { get; set; }

    [Timestamp]
    public byte[]? RowVersion { get; set; }
}
```

**Expected Output:** Updating a department includes `RowVersion` in the `WHERE` clause; a mismatch throws `DbUpdateConcurrencyException`.

**Why this output?** `TimestampAttribute` configures the concurrency token.

### Real‑World Cases

- **Multi‑user editing:** Prevent lost updates.
- **Booking systems:** Detect double‑booking conflicts.
- **Inventory:** Avoid stock‑level conflicts.

### References

- `TimestampAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.timestampattribute
- Concurrency conflicts – https://learn.microsoft.com/en-us/aspnet/core/data/ef-rp/concurrency

---

## 24. `[ConcurrencyCheck]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ConcurrencyCheck]` marks one or more properties as concurrency tokens for optimistic concurrency. |
| **Technical Definition** | `ConcurrencyCheckAttribute` (in `.Schema`) configures the property as a concurrency token. Unlike `[Timestamp]`, it can be applied to **multiple properties** and does not require a row‑version column type. |
| **Beginner‑Friendly Explanation** | `[ConcurrencyCheck]` says “check this column’s value before updating to make sure nobody else changed it.” |

### Purposes

- **To** implement optimistic concurrency on one or more columns.
- **To** use existing business columns as concurrency tokens.
- **To** avoid adding a dedicated `rowversion` column.

### Syntax Rules and Structure

```csharp
[ConcurrencyCheck]
public string? Status { get; set; }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| No parameters | Marks the property as a concurrency token. |

**Syntax rules:**

- Can be applied to **multiple** properties.
- The original value is included in the `WHERE` clause of `UPDATE`/`DELETE`.

**Constraints and limitations:**

- **Less efficient** than `[Timestamp]` for high‑concurrency tables because it compares actual values.
- Requires the client to post the original value (often via hidden field).

### Annotated Code Example

```csharp
public class Order
{
    public int Id { get; set; }

    [ConcurrencyCheck]
    public string? Status { get; set; }

    [ConcurrencyCheck]
    public int Version { get; set; }
}
```

**Expected Output:** Updates include `Status` and `Version` in the `WHERE` clause; conflicts throw `DbUpdateConcurrencyException`.

**Why this output?** `ConcurrencyCheckAttribute` configures both properties as tokens.

### Real‑World Cases

- **Workflow systems:** Status as a concurrency token.
- **Legacy schemas:** No rowversion column available.
- **Multi‑column tokens:** Combining several business fields.

### References

- `ConcurrencyCheckAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.concurrencycheckattribute

---

## 25. Data Annotation Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Best practices are guidelines for using data annotations safely, consistently, and maintainably. |
| **Technical Definition** | These practices span security (over‑posting, hidden fields), schema awareness (validation attributes affecting EF Core), client/server parity, error‑message design, and separation of concerns. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for data annotations — they help you avoid security holes and confusing behaviour. |

### Purposes

- **To** prevent over‑posting and mass‑assignment attacks.
- **To** avoid unintended database schema changes from validation attributes.
- **To** keep client‑side and server‑side validation consistent.
- **To** write clear, actionable error messages.
- **To** separate domain models from view‑specific input models.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Binding security** | Use ViewModels/DTOs with only the properties the user should edit. | Bind directly to entities that contain `IsAdmin`, `Role`, or audit fields. |
| **Schema awareness** | Know that `[Required]` creates `NOT NULL` and `[StringLength]`/`[MaxLength]` affect column length. | Assume validation attributes have no database impact. |
| **`[Required]` on value types** | Omit `[Required]` on `int`, `decimal`, `DateTime`. | Add `[Required]` to value types (redundant). |
| **Nullable reference types** | Use `string?` for optional strings; `string` is implicitly required. | Mix nullable and non‑nullable inconsistently. |
| **`[DataType]` + `[DisplayFormat]`** | Combine them for correct field templates and formatting. | Rely on `[DisplayFormat]` alone for date pickers. |
| **`[HiddenInput]` / `[ReadOnly]`** | Treat them as UI hints only. | Treat them as security boundaries. |
| **`[Timestamp]` vs `[ConcurrencyCheck]`** | Prefer `[Timestamp]` (rowversion) for SQL Server. | Use `[ConcurrencyCheck]` on many columns in high‑throughput tables. |
| **Client‑side validation** | Include jQuery unobtrusive scripts for client feedback. | Rely solely on client‑side validation. |
| **Error messages** | Use `ErrorMessage` with `{0}`, `{1}` placeholders for clarity. | Use generic “Invalid value” messages. |
| **Custom validation** | Use `IValidatableObject` for cross‑property rules. | Overload `[RegularExpression]` with complex multi‑field logic. |

### Annotated Code Example (Best‑Practice Model)

```csharp
public class RegisterInputModel   // ViewModel, not domain entity
{
    [Required, StringLength(50, MinimumLength = 3)]
    [Display(Name = "Username")]
    public string Username { get; set; } = null!;

    [Required, EmailAddress]
    public string Email { get; set; } = null!;

    [Required, DataType(DataType.Password), MinLength(8)]
    public string Password { get; set; } = null!;

    [DataType(DataType.Password)]
    [Compare(nameof(Password), ErrorMessage = "Passwords do not match.")]
    public string ConfirmPassword { get; set; } = null!;
}
```

**Why this follows best practices:**

- **ViewModel** isolates input from the domain entity, preventing over‑posting.
- **`[DataType(DataType.Password)]`** ensures masked inputs.
- **`[Compare]`** enforces password confirmation without controller logic.
- **`[MinLength]`** enforces a minimum without affecting the database schema.
- **`ErrorMessage`** provides clear, field‑specific feedback.

### Real‑World Cases

- **Public registration forms:** ViewModels prevent privilege escalation.
- **Admin CRUD:** DTOs hide audit fields while annotations enforce validation.
- **APIs:** Request models with validation annotations and response DTOs without them.

### References

- Part 9, add validation to an ASP.NET Core MVC app (DRY principle) – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Mapping attributes (aka data annotations) for relationships – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/mapping-attributes

---

## Summary Table: All 24 Attributes at a Glance

| Attribute | Category | Primary Effect | Affects EF Core Schema? |
|-----------|----------|----------------|--------------------------|
| `[Required]` | Validation | Non‑null value | Yes (`NOT NULL`) |
| `[StringLength]` | Validation | Min/max string length | No (use `[MaxLength]` for schema) |
| `[MaxLength]` | Validation | Max length (strings/arrays) | Yes (`nvarchar(n)`) |
| `[MinLength]` | Validation | Min length (strings/arrays) | No |
| `[Range]` | Validation | Numeric/date bounds | No |
| `[RegularExpression]` | Validation | Pattern match | No |
| `[EmailAddress]` | Validation | Email format | No |
| `[Phone]` | Validation | Phone format | No |
| `[Url]` | Validation | URL format | No |
| `[Compare]` | Validation | Equality with another property | No |
| `[DataType]` | Formatting | Semantic type / field template | No |
| `[Display]` | Formatting | Name, order, description | No |
| `[DisplayName]` | Formatting | Simple display name | No |
| `[DisplayFormat]` | Formatting | Format string, null text | No |
| `[ScaffoldColumn]` | UI Hint | Include/exclude from scaffolding | No |
| `[HiddenInput]` | UI Hint | Render as hidden input | No |
| `[ReadOnly]` | UI Hint | Mark as read‑only | No |
| `[Editable]` | UI Hint | Enable/disable editing | No |
| `[Key]` | EF Core Mapping | Primary key | Yes |
| `[ForeignKey]` | EF Core Mapping | Foreign key property | Yes |
| `[NotMapped]` | EF Core Mapping | Exclude from database | Yes (excludes) |
| `[Timestamp]` | EF Core Mapping | Row version concurrency token | Yes |
| `[ConcurrencyCheck]` | EF Core Mapping | Concurrency token (any column) | Yes |

---

## References (Full List)

- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- Mapping attributes (aka data annotations) for relationships – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/mapping-attributes
- `RequiredAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.requiredattribute
- `StringLengthAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.stringlengthattribute
- `MaxLengthAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.maxlengthattribute
- `MinLengthAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.minlengthattribute
- `RangeAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.rangeattribute
- `RegularExpressionAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.regularexpressionattribute
- `EmailAddressAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute
- `PhoneAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.phoneattribute
- `UrlAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.urlattribute
- `CompareAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.compareattribute
- `DataTypeAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.datatypeattribute
- `DisplayAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.displayattribute
- `DisplayNameAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.displaynameattribute
- `DisplayFormatAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.displayformatattribute
- `ScaffoldColumnAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.scaffoldcolumnattribute
- `HiddenInputAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.hiddeninputattribute
- `ReadOnlyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.readonlyattribute
- `EditableAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.editableattribute
- `KeyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.keyattribute
- `ForeignKeyAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.foreignkeyattribute
- `NotMappedAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.notmappedattribute
- `TimestampAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.timestampattribute
- `ConcurrencyCheckAttribute` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.schema.concurrencycheckattribute
- Concurrency conflicts in EF Core – https://learn.microsoft.com/en-us/aspnet/core/data/ef-rp/concurrency