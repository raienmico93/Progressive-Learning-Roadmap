# Route Constraints in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10/11). Route constraints behave consistently across **conventional routing** and **attribute routing**. The constraints covered are the built‑in constraints provided by the framework, plus guidance on implementing **custom constraints**. All built‑in constraints are available in both inline form (`{id:int}`) and via the `constraints` object literal in `MapControllerRoute` / `MapAreaControllerRoute`.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route constraint is a rule that a route parameter’s value must satisfy for the route to be considered a match. |
| **Technical Definition** | Route constraints execute when a match has occurred to the incoming URL and the URL path is tokenized into route values. They generally inspect the route value associated via the route template and make a true/false decision about whether the value is acceptable. Some constraints use data outside the route value (e.g., `HttpMethodRouteConstraint` checks the HTTP verb). Constraints are used in **routing requests** and **link generation**. A constraint implements `IRouteConstraint` (built‑in or custom) and is applied inline using the syntax `{parameter:constraintName}` or via the `constraints` parameter of `MapControllerRoute`. |
| **Beginner‑Friendly Explanation** | A route constraint is a **gatekeeper** for a URL segment. In `{id:int}`, the `int` gatekeeper says: “Only allow this route to match if the value is a whole number.” If someone visits `/products/abc`, the `int` gatekeeper says “No,” and the route is skipped. Constraints help your application reject bad URLs early and choose the right endpoint when multiple routes could match. |

### Key Characteristics

- **Declarative and inline** – Constraints are written directly in the route template (`{id:int}`) or supplied as an object in conventional routing.
- **Match‑time evaluation** – Constraints run **after** a template match has occurred, during the tokenisation of route values.
- **Disambiguation‑focused** – Their primary role is to **distinguish between similar routes**, not to validate business input.
- **Invariant culture** – Constraints that convert to CLR types (`int`, `DateTime`, `decimal`, etc.) always use the **invariant culture**.
- **Non‑mutating** – Framework‑provided constraints **do not modify** the values stored in route values; all parsed values remain strings.
- **Chainable** – Multiple colon‑delimited constraints can be applied to a single parameter (`{id:int:min(1)}`）.
- **Extensible** – Custom constraints can be created by implementing `IRouteConstraint` and registering them in `ConstraintMap`.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()` or `AddControllers()`).
- Routing middleware configured (`UseRouting()` and `MapControllers()` / `MapControllerRoute`).
- Basic understanding of **route templates** and **route parameters**.

### Related Programming Areas

- **Route Parameters** – Constraints are applied to parameters.
- **Model Binding** – Constraints prevent invalid values from reaching model binding, but they are **not** a substitute for validation.
- **URL Generation** – Constraints participate in link generation; values that fail constraints may not produce valid links.
- **Attribute Routing** – Constraints are frequently used inline in `[HttpGet("{id:int}")]`.
- **Custom Constraints** – Advanced scenarios where built‑in constraints are insufficient.
- **Security** – Regex constraints must be guarded against ReDoS (Regular Expression Denial of Service).

### Core Concepts / Features

1. **Route Constraints (General)** – The mechanism and its role in routing.
2. **Numeric Constraints** – `int`, `long`, `decimal`, `double`, `float`, `min`, `max`, `range`.
3. **Boolean and Date Constraints** – `bool`, `datetime`.
4. **GUID Constraint** – `guid`.
5. **String Length Constraints** – `minlength`, `maxlength`, `length`.
6. **String Content Constraints** – `alpha`, `regex`.
7. **Special Constraints** – `required`.
8. **Custom Constraints** – Implementing `IRouteConstraint`.
9. **Route Constraint Best Practices** – Guidelines for correct and secure usage.

---

## 1. Route Constraints (General Mechanism)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route constraints are predicates that validate route parameter values during matching, allowing the router to disambiguate between multiple candidate templates. |
| **Technical Definition** | A route constraint implements `IRouteConstraint`. The `Match` method receives the `HttpContext`, the `IRouter`, the route key (parameter name), the `RouteValueDictionary`, and the `RouteDirection`. It returns `true` if the value is acceptable. Built‑in constraints are registered in `RouteOptions.ConstraintMap` and are resolved by name during template parsing. Constraints execute **after** a template match and **before** endpoint selection, narrowing the set of candidates. |
| **Beginner‑Friendly Explanation** | Think of route constraints as **filters** applied to the blanks in a URL pattern. In `products/{id:int}`, the `int` filter ensures that only numbers fill the `{id}` blank. If the URL contains letters, the filter rejects it, and the router looks for a different route or returns 404. |

### Purposes (each begins with “To”)

- **To** disambiguate similar routes so that the correct endpoint is chosen deterministically.
- **To** reject URLs that cannot possibly be valid for a given endpoint, avoiding unnecessary model‑binding errors.
- **To** improve routing performance by eliminating non‑matching endpoints early.
- **To** express data‑type expectations (integer, GUID, date) directly in the URL template.
- **To** participate in link generation, ensuring that generated URLs satisfy the same rules as incoming requests.

### Syntax Rules and Structure

#### General Syntax (Inline)

```
{parameter:constraintName}
{parameter:constraintName(argument)}
{parameter:constraint1:constraint2}   // chained
```

#### General Syntax (Object Literal in Conventional Routing)

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}",
    constraints: new { id = @"\d+" } // regex constraint via object literal
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `parameter` | The route parameter name. |
| `constraintName` | The name of the constraint (e.g., `int`, `guid`, `minlength`). |
| `argument` | An optional argument (e.g., `4` in `minlength(4)`). |
| `constraints` object | Property names match route parameter names; string values are interpreted as regular expressions. |

#### Syntax Rules

- Constraints are **case‑insensitive** (`int` and `Int` are equivalent).
- Multiple constraints are **colon‑delimited** and evaluated in order: `{id:int:min(1)}`.
- Regex constraints must escape routing delimiter characters (`{`, `}`, `[`, `]`) by doubling them.
- Anchors (`^` and `$`) are strongly recommended for regex constraints so that the entire segment is matched.

#### Constraints and Limitations

- **Not for input validation** – Microsoft explicitly warns: “Don’t use constraints for input validation. If constraints are used for input validation, invalid input results in a `404 Not Found` response. Invalid input should produce a `400 Bad Request` with an appropriate error message. Route constraints are used to **disambiguate similar routes**, not to validate the inputs for a particular route”.
- **Invariant culture** – Constraints that convert to CLR types (`int`, `DateTime`, `decimal`, etc.) always use the invariant culture; they assume the URL is **not localizable**.
- **Values are not modified** – Constraints only verify convertibility; the stored route values remain strings.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Chaining `int` and `min`

```csharp
[HttpGet("users/{id:int:min(1)}")]
public IActionResult GetUser(int id) => Ok($"User {id}");
```

**Step‑by‑step explanation:**

1. `{id:int}` ensures the segment is a 32‑bit integer.
2. `min(1)` further requires the integer to be ≥ 1.
3. Both constraints must return `true` for the route to match.

**Expected Output:**

| URL | Match? | Reason |
|-----|--------|--------|
| `/users/5` | Yes | Integer ≥ 1. |
| `/users/0` | No | `min(1)` fails. |
| `/users/abc` | No | `int` fails. |

**Why this output?** Constraints are evaluated sequentially; a failure in any constraint discards the candidate endpoint. This prevents invalid IDs from reaching the action.

---

#### Example 2: Regex Constraint via Object Literal (Conventional Routing)

```csharp
app.MapControllerRoute(
    name: "people",
    pattern: "people/{ssn}",
    constraints: new { ssn = "^\\d{3}-\\d{2}-\\d{4}$" },
    defaults: new { controller = "People", action = "List" });
```

**Step‑by‑step explanation:**

1. The `constraints` object maps the `ssn` parameter to a regex string.
2. The regex is interpreted directly (no `regex()` wrapper needed in object‑literal form).
3. The string must match the full Social Security Number pattern.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/people/123-45-6789` | Yes |
| `/people/123456789` | No |

**Why this output?** The object‑literal form treats string values as regular expressions. The anchors ensure the entire segment matches.

---

#### Example 3: Custom Constraint (Preview)

```csharp
// Registration
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("noZeroes", typeof(NoZeroesRouteConstraint));
});

// Usage
[HttpGet("{id:noZeroes}")]
public IActionResult Get(string id) => Ok(id);
```

**Step‑by‑step explanation:**

1. A custom constraint is registered under the key `"noZeroes"`.
2. The template `{id:noZeroes}` applies the constraint.
3. The constraint’s `Match` method decides whether the value is acceptable.

**Expected Output:** Depends on the custom logic (see Section 8 for a full example).

**Why this output?** Custom constraints extend the built‑in set, allowing domain‑specific disambiguation rules.

### Real‑World Cases

- **API ID validation:** `{id:int}` prevents non‑numeric IDs from reaching database queries.
- **SEO slugs:** `{slug:regex(^[a-z0-9-]+$)}` ensures URL‑safe blog post slugs.
- **Date‑based archives:** `{year:int}/{month:int:range(1,12)}` validates year and month segments.
- **Multi‑tenant routing:** `{tenant:alpha}` ensures tenant names contain only letters.

### References

- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- Route constraint reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraint-reference
- Don’t use constraints for input validation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#dont-use-constraints-for-input-validation

---

## 2. Built‑in Route Constraints Reference

The following table summarises all built‑in constraints. Detailed explanations and examples follow in grouped sections.

| Constraint | Example | Example Matches | Notes |
|------------|---------|-----------------|-------|
| `int` | `{id:int}` | `123456789`, `-123456789` | Matches any integer. |
| `bool` | `{active:bool}` | `true`, `FALSE` | Matches `true` or `false`, case‑insensitive. |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | Matches a valid `DateTime` in the invariant culture. |
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | Matches a valid `decimal` in the invariant culture. |
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | Matches a valid `double` in the invariant culture. |
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | Matches a valid `float` in the invariant culture. |
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | Matches a valid `Guid`. |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | Matches a valid `long`. |
| `minlength(n)` | `{username:minlength(4)}` | `Rick` | String must be at least `n` characters. |
| `maxlength(n)` | `{filename:maxlength(8)}` | `MyFile` | String must be no more than `n` characters. |
| `length(n)` | `{filename:length(12)}` | `somefile.txt` | String must be exactly `n` characters. |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | String must be between `min` and `max` characters. |
| `min(n)` | `{age:min(18)}` | `19` | Integer must be ≥ `n`. |
| `max(n)` | `{age:max(120)}` | `91` | Integer must be ≤ `n`. |
| `range(min,max)` | `{age:range(18,120)}` | `91` | Integer must be between `min` and `max`. |
| `alpha` | `{name:alpha}` | `Rick` | String must consist of one or more alphabetical characters (a‑z, case‑insensitive). |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | String must match the regular expression. |
| `required` | `{name:required}` | `Rick` | Used to enforce that a non‑parameter value is present during URL generation. |

> **Note:** Additional constraints such as `file`, `nonfile`, and `min`/`max` for non‑integer types exist but are less commonly used. The table above covers the constraints requested in this cheat sheet.

---

### 2.1 Numeric Constraints: `int`, `long`, `decimal`, `double`, `float`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Numeric constraints verify that a route parameter’s value can be converted to a specific numeric CLR type. |
| **Technical Definition** | These constraints attempt to parse the route value using the **invariant culture**. If parsing succeeds, the constraint returns `true`; otherwise, `false`. The stored route value remains a string; the constraint does not change the value. |
| **Beginner‑Friendly Explanation** | These constraints check: “Is this segment a number?” `int` checks whole numbers, `decimal` checks numbers with decimal places, `double` and `float` check floating‑point numbers, and `long` checks very large whole numbers. |

#### Purposes (each begins with “To”)

- **To** ensure that a route parameter represents a numeric value before the action executes.
- **To** distinguish numeric IDs from slugs or other string identifiers in overlapping routes.
- **To** prevent model‑binding errors caused by non‑numeric values reaching numeric action parameters.
- **To** participate in link generation, ensuring generated URLs contain valid numeric segments.

#### Syntax Rules and Structure

```
{parameter:int}
{parameter:long}
{parameter:decimal}
{parameter:double}
{parameter:float}
```

**Component breakdown:**

| Constraint | CLR Type | Example | Matches |
|------------|----------|---------|---------|
| `int` | `System.Int32` | `{id:int}` | `42`, `-7` |
| `long` | `System.Int64` | `{ticks:long}` | `1234567890123` |
| `decimal` | `System.Decimal` | `{price:decimal}` | `19.99`, `-1,000.01` |
| `double` | `System.Double` | `{weight:double}` | `1.234`, `-1,001.01e8` |
| `float` | `System.Single` | `{weight:float}` | `1.234`, `-1,001.01e8` |

#### Constraints and Limitations

- **Invariant culture** – Parsing always uses the invariant culture; numbers with locale‑specific separators may not match.
- **Negative numbers** – `int` and `long` accept negative values; combine with `min(0)` or `min(1)` to restrict to positive numbers.
- **Decimal separators** – `decimal`, `double`, and `float` expect a period (`.`) as the decimal separator under the invariant culture.

#### Annotated Code Examples

##### Example 1: `int` with `min(1)`

```csharp
[HttpGet("products/{id:int:min(1)}")]
public IActionResult GetProduct(int id) => Ok($"Product {id}");
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/products/10` | Yes |
| `/products/0` | No (`min(1)` fails) |
| `/products/abc` | No (`int` fails) |

**Why this output?** The `int` constraint ensures the segment is a whole number; `min(1)` further restricts it to positive values. Both must pass for the endpoint to match.

##### Example 2: `decimal` for Prices

```csharp
[HttpGet("products/{price:decimal}")]
public IActionResult FilterByPrice(decimal price) => Ok($"Price: {price}");
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/products/19.99` | Yes |
| `/products/abc` | No |

**Why this output?** The `decimal` constraint parses the value using the invariant culture; `19.99` is valid, while `abc` is not.

##### Example 3: `guid` (covered in section 2.3)

#### Real‑World Cases

- **E‑commerce product IDs:** `{id:int:min(1)}` ensures positive integer IDs.
- **Financial calculations:** `{amount:decimal}` ensures monetary values are valid decimals.
- **Scientific measurements:** `{value:double}` accepts floating‑point numbers in scientific notation.

#### References

- `int` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#int
- `decimal` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#decimal
- Invariant culture note – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints

---

### 2.2 Boolean and Date Constraints: `bool`, `datetime`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `bool` validates that a value is `true` or `false` (case‑insensitive). `datetime` validates that a value is a parseable `DateTime` in the invariant culture. |
| **Technical Definition** | `BoolRouteConstraint` uses `bool.TryParse`. `DateTimeRouteConstraint` uses `DateTime.TryParse` with `CultureInfo.InvariantCulture`. Both return `true` only if parsing succeeds. |
| **Beginner‑Friendly Explanation** | `bool` checks: “Is this segment either `true` or `false`?” `datetime` checks: “Does this segment look like a date or date‑time?” |

#### Purposes

- **To** validate boolean flags in the URL (e.g., `{active:bool}`).
- **To** validate date parameters in archive or scheduling routes (e.g., `{date:datetime}`).
- **To** disambiguate date segments from arbitrary strings in overlapping routes.

#### Syntax Rules

```
{parameter:bool}
{parameter:datetime}
```

#### Constraints and Limitations

- **`datetime` uses invariant culture** – Dates formatted according to local cultures may not match.
- **`bool` is case‑insensitive** – `TRUE`, `true`, `False`, and `false` all match.
- **Date format ambiguity** – The invariant culture expects formats like `yyyy-MM-dd`; other formats may fail.

#### Annotated Code Example

```csharp
[HttpGet("events/{date:datetime}")]
public IActionResult GetEvents(DateTime date) => Ok($"Events on {date:yyyy-MM-dd}");

[HttpGet("features/{enabled:bool}")]
public IActionResult Feature(bool enabled) => Ok($"Enabled: {enabled}");
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/events/2025-03-15` | Yes |
| `/events/15-03-2025` | May fail (invariant culture) |
| `/features/true` | Yes |
| `/features/yes` | No |

**Why this output?** `datetime` parses according to the invariant culture; `bool` accepts only `true` or `false` (case‑insensitive).

#### Real‑World Cases

- **Event calendars:** `{date:datetime}` restricts archive URLs to valid dates.
- **Feature flags:** `{enabled:bool}` toggles features via URL.

#### References

- `bool` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#bool
- `datetime` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#datetime

---

### 2.3 GUID Constraint: `guid`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `guid` validates that a route parameter’s value is a valid `System.Guid`. |
| **Technical Definition** | `GuidRouteConstraint` uses `Guid.TryParse` to validate the segment. |
| **Beginner‑Friendly Explanation** | `guid` checks: “Does this segment look like a globally unique identifier?” |

#### Purposes

- **To** validate GUID‑format identifiers in APIs and URLs.
- **To** disambiguate GUID IDs from integer IDs in overlapping routes.
- **To** prevent invalid GUID strings from reaching actions that expect `Guid` parameters.

#### Syntax Rules

```
{parameter:guid}
```

#### Constraints and Limitations

- **Format sensitivity** – Only standard GUID formats are accepted (e.g., `CD2C1638-1638-72D5-1638-DEADBEEF1638`).
- **Case‑insensitive** – GUID parsing is case‑insensitive.

#### Annotated Code Example

```csharp
[HttpGet("orders/{orderId:guid}")]
public IActionResult GetOrder(Guid orderId) => Ok(orderId);
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/orders/3fa85f64-5717-4562-b3fc-2c963f66afa6` | Yes |
| `/orders/123` | No |

**Why this output?** `guid` rejects non‑GUID strings, ensuring only valid GUIDs reach the action.

#### Real‑World Cases

- **Distributed systems:** GUIDs as primary keys.
- **Public APIs:** Non‑sequential identifiers to mitigate enumeration attacks.

#### References

- `guid` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#guid

---

### 2.4 String Length Constraints: `minlength`, `maxlength`, `length`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | These constraints validate the **length** of a string route parameter. |
| **Technical Definition** | `MinLengthRouteConstraint`, `MaxLengthRouteConstraint`, and `LengthRouteConstraint` check the `string.Length` of the captured value against the specified bounds. |
| **Beginner‑Friendly Explanation** | These constraints check: “Is this segment the right length?” `minlength(4)` means at least 4 characters; `maxlength(8)` means at most 8; `length(12)` means exactly 12. |

#### Purposes

- **To** enforce minimum, maximum, or exact string lengths in route parameters.
- **To** disambiguate short codes from longer slugs in overlapping routes.
- **To** provide a first line of defence against overly long or overly short segments.

#### Syntax Rules

```
{parameter:minlength(n)}
{parameter:maxlength(n)}
{parameter:length(n)}
{parameter:length(min,max)}
```

#### Constraints and Limitations

- **Character count, not byte count** – Length is measured in UTF‑16 code units.
- **No content validation** – Length constraints do not check whether the string contains valid characters; combine with `alpha` or `regex` for content rules.

#### Annotated Code Example

```csharp
[HttpGet("users/{username:minlength(4):maxlength(20)}")]
public IActionResult GetUser(string username) => Ok(username);
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/users/alice` | Yes (5 characters) |
| `/users/al` | No (too short) |
| `/users/abcdefghijklmnopqrst` | No (too long) |

**Why this output?** Both `minlength(4)` and `maxlength(20)` must pass. The segment must be between 4 and 20 characters.

#### Real‑World Cases

- **Usernames:** `{username:minlength(4):maxlength(20)}`.
- **Product codes:** `{code:length(8)}` for fixed‑length SKUs.
- **File names:** `{filename:maxlength(255)}` to respect filesystem limits.

#### References

- `minlength` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#minlength
- `maxlength` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#maxlength
- `length` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#length

---

### 2.5 Numeric Range Constraints: `min`, `max`, `range`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | These constraints validate that a numeric route parameter falls within specified bounds. |
| **Technical Definition** | `MinRouteConstraint`, `MaxRouteConstraint`, and `RangeRouteConstraint` parse the value as a `long` (or appropriate numeric type) and compare it to the configured minimum/maximum. |
| **Beginner‑Friendly Explanation** | `min(18)` means “at least 18”; `max(120)` means “no more than 120”; `range(18,120)` means “between 18 and 120”. |

#### Purposes

- **To** restrict numeric parameters to a valid range (e.g., age, month, page number).
- **To** disambiguate routes based on numeric thresholds.
- **To** prevent out‑of‑range values from reaching actions.

#### Syntax Rules

```
{parameter:min(n)}
{parameter:max(n)}
{parameter:range(min,max)}
```

#### Constraints and Limitations

- **`min` and `max` are inclusive** – `min(18)` accepts 18 and above.
- **Numeric type** – These constraints operate on numeric values; combine with `int` or `long` for explicit type validation.

#### Annotated Code Example

```csharp
[HttpGet("archive/{year:int:range(2000,2100)}/{month:int:range(1,12)}")]
public IActionResult Archive(int year, int month) => Ok($"{year}-{month:00}");
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/archive/2025/6` | Yes |
| `/archive/1999/6` | No (year out of range) |
| `/archive/2025/13` | No (month out of range) |

**Why this output?** The `range` constraints reject values outside the specified bounds.

#### Real‑World Cases

- **Age verification:** `{age:int:range(18,120)}`.
- **Pagination:** `{page:int:min(1)}`.
- **Date components:** `{month:int:range(1,12)}`.

#### References

- `min` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#min
- `max` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#max
- `range` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#range

---

### 2.6 String Content Constraints: `alpha`, `regex`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `alpha` validates that a string consists only of alphabetical characters. `regex` validates that a string matches a regular expression. |
| **Technical Definition** | `AlphaRouteConstraint` uses a regex equivalent to `^[a-zA-Z]+$` (case‑insensitive). `RegexRouteConstraint` compiles the supplied expression with `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant`. |
| **Beginner‑Friendly Explanation** | `alpha` checks: “Does this segment contain only letters?” `regex` checks: “Does this segment match a custom pattern?” |

#### Purposes

- **To** enforce that a parameter contains only letters (`alpha`).
- **To** enforce complex patterns such as slugs, codes, or formatted identifiers (`regex`).
- **To** disambiguate routes based on content format.

#### Syntax Rules

```
{parameter:alpha}
{parameter:regex(expression)}
```

#### Constraints and Limitations

- **ReDoS risk** – Regex constraints applied to untrusted input can cause Denial of Service. Always pass a timeout when using `System.Text.RegularExpressions`.
- **Anchors required for full match** – Use `^` and `$` to match the entire segment.
- **Escape routing delimiters** – Double `{`, `}`, `[`, `]` inside regex constraints.

#### Annotated Code Example

```csharp
[HttpGet("authors/{name:alpha}")]
public IActionResult GetAuthor(string name) => Ok(name);

[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost(string slug) => Ok(slug);
```

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/authors/Rick` | Yes |
| `/authors/Rick123` | No (`alpha` fails) |
| `/blog/my-first-post` | Yes |
| `/blog/My-Post` | No (regex fails due to uppercase) |

**Why this output?** `alpha` accepts only letters; the regex constraint enforces lowercase letters, digits, and hyphens for the slug.

#### Real‑World Cases

- **Author names:** `{name:alpha}`.
- **SEO slugs:** `{slug:regex(^[a-z0-9-]+$)}`.
- **Social Security Numbers:** `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}`.

#### References

- `alpha` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#alpha
- `regex` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regex
- Regex timeout warning – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regular-expressions-in-constraints

---

### 2.7 Special Constraint: `required`

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `required` enforces that a non‑parameter value is present during **URL generation**. |
| **Technical Definition** | `RequiredRouteConstraint` returns `false` during link generation if the route value is missing or empty. It does not affect request matching in the same way as other constraints; its primary role is to ensure that generated links contain a value for the parameter. |
| **Beginner‑Friendly Explanation** | `required` is mainly used when **building URLs**. It says: “You must supply a value for this segment when generating a link.” |

#### Purposes

- **To** ensure that URL generation fails gracefully when a required parameter is missing.
- **To** prevent links with empty segments from being generated.

#### Syntax Rules

```
{parameter:required}
```

#### Constraints and Limitations

- **Primarily affects link generation** – It is not a substitute for `{parameter}` (which already requires the segment for matching).
- **Does not validate content** – It only checks presence, not format.

#### Annotated Code Example

```csharp
[HttpGet("profile/{username:required}")]
public IActionResult Profile(string username) => Ok(username);
```

**Expected Output:**

- During matching, `{username:required}` behaves similarly to `{username}` (a segment is required).
- During link generation, omitting `username` produces an empty or invalid link rather than a URL with an empty segment.

**Why this output?** `required` adds a check during link generation, ensuring that the parameter is supplied.

#### Real‑World Cases

- **Profile URLs:** Ensuring that a username is always present when generating profile links.
- **Canonical URLs:** Preventing links with missing essential segments.

#### References

- `required` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#required

---

## 3. Custom Constraints

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A custom constraint is a user‑defined implementation of `IRouteConstraint` that provides application‑specific validation logic for route parameters. |
| **Technical Definition** | Implement `IRouteConstraint` and override `Match(HttpContext? httpContext, IRouter? route, string routeKey, RouteValueDictionary values, RouteDirection routeDirection)`. Register the constraint type in `RouteOptions.ConstraintMap` with a unique key. Then use the key in templates (`{id:customName}`) or in the `constraints` object. |
| **Beginner‑Friendly Explanation** | Custom constraints let you write your own “gatekeeper” rules when the built‑in constraints aren’t enough. For example, you could create a constraint that only allows even numbers, or one that checks a value against a database. |

### Purposes (each begins with “To”)

- **To** implement domain‑specific validation rules that cannot be expressed with built‑in constraints.
- **To** disambiguate routes based on custom business logic.
- **To** encapsulate reusable validation logic that applies to multiple routes.
- **To** extend the routing system without modifying framework code.

### Syntax Rules and Structure

#### Implementation

```csharp
public class MyCustomConstraint : IRouteConstraint
{
    public bool Match(HttpContext? httpContext, IRouter? route,
                      string routeKey, RouteValueDictionary values,
                      RouteDirection routeDirection)
    {
        if (values.TryGetValue(routeKey, out var value))
        {
            // Custom logic
            return /* true or false */;
        }
        return false;
    }
}
```

#### Registration

```csharp
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

Alternatively, via `AddRouting`:

```csharp
builder.Services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

#### Usage

```csharp
[HttpGet("{id:customName}")]
public IActionResult Get(string id) => Ok(id);
```

### Constraints and Limitations

- **Custom constraints are rarely needed** – Microsoft advises: “Custom route constraints are rarely needed. Before implementing a custom route constraint, consider alternatives like model binding”.
- **Regex timeout** – If your custom constraint uses regex, always pass a timeout to prevent ReDoS.
- **Registration required** – The constraint must be registered in `ConstraintMap` before it can be used in templates.

### Complete Annotated Step‑by‑Step Code Example

```csharp
// 1. Define the custom constraint: disallow zero values.
public class NoZeroesRouteConstraint : IRouteConstraint
{
    private static readonly Regex _regex = new Regex(
        @"^[1-9][0-9]*$",
        RegexOptions.CultureInvariant | RegexOptions.IgnoreCase,
        TimeSpan.FromMilliseconds(100)); // Timeout to mitigate ReDoS

    public bool Match(HttpContext? httpContext, IRouter? route,
                      string routeKey, RouteValueDictionary values,
                      RouteDirection routeDirection)
    {
        if (values.TryGetValue(routeKey, out var value))
        {
            var stringValue = Convert.ToString(value, CultureInfo.InvariantCulture);
            return stringValue != null && _regex.IsMatch(stringValue);
        }
        return false;
    }
}

// 2. Register the constraint.
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("noZeroes", typeof(NoZeroesRouteConstraint));
});

var app = builder.Build();
app.UseRouting();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();

// 3. Use the constraint.
[Route("api/[controller]")]
[ApiController]
public class TestController : ControllerBase
{
    [HttpGet("{id:noZeroes}")]
    public IActionResult Get(string id) => Ok($"Valid ID: {id}");
}
```

**Step‑by‑step explanation:**

1. `NoZeroesRouteConstraint` implements `IRouteConstraint`. Its regex `^[1-9][0-9]*$` matches positive integers without leading zeros.
2. The constraint is registered under the key `"noZeroes"`.
3. The route `api/Test/{id:noZeroes}` applies the constraint.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/api/Test/5` | Yes |
| `/api/Test/0` | No (zero not allowed) |
| `/api/Test/05` | No (leading zero) |
| `/api/Test/abc` | No |

**Why this output?** The custom constraint’s `Match` method returns `true` only for positive integers without leading zeros, effectively disallowing zero and non‑numeric values.

### Real‑World Cases

- **Positive‑only IDs:** A custom constraint that rejects `0` and negative numbers.
- **Business‑day validation:** A constraint that checks whether a date falls on a working day.
- **Tenant validation:** A constraint that verifies the tenant slug exists in a database (though this may be better handled in middleware for performance reasons).

### References

- Custom route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#custom-route-constraints
- `IRouteConstraint` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint
- ConstraintMap registration – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#custom-route-constraints

---

## 4. Route Constraint Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route constraint best practices are guidelines for using constraints correctly, securely, and maintainably, recognising their role as **route disambiguators** rather than input validators. |
| **Technical Definition** | Best practices encompass constraint selection (built‑in vs custom), regex safety (timeouts, anchoring), culture awareness, avoidance of over‑constraining, and the separation of routing concerns from validation and business logic. |
| **Beginner‑Friendly Explanation** | Best practices are the “rules of thumb” that keep constraint usage safe and predictable. For example: don’t use constraints to validate user input, always anchor your regexes, and keep constraints simple. |

### Purposes (each begins with “To”)

- **To** prevent security vulnerabilities such as ReDoS in regex constraints.
- **To** ensure deterministic route matching by using constraints as disambiguators.
- **To** avoid confusing 404 responses when 400 responses would be more appropriate.
- **To** maintain performance by keeping constraints lightweight.
- **To** keep the URL space simple and well defined.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Don’t use constraints for input validation** | Constraints produce 404s; invalid input should produce 400s with meaningful errors. | Use model validation in the action for business rules. |
| **Anchor regex constraints** | Unanchored regexes match substrings, leading to unintended matches. | `regex(^[a-z0-9-]+$)` |
| **Pass a timeout to regex** | Prevents ReDoS attacks on untrusted input. | Use `TimeSpan.FromMilliseconds(100)`. |
| **Be aware of invariant culture** | `datetime`, `decimal`, etc. use invariant culture; localised formats may fail. | Document expected formats. |
| **Prefer built‑in constraints** | Custom constraints add complexity and startup cost. | Use `int`, `guid`, `minlength` before writing custom code. |
| **Keep URL space simple** | Multiple routes on actions can make routing hard to reason about. | Use multiple routes only for backward compatibility. |
| **Combine constraints when needed** | Chaining narrows matches without custom code. | `{id:int:min(1)}` |
| **Avoid over‑constraining** | Overly strict constraints make routes brittle. | Use constraints only where they add clear value. |

### Constraints and Limitations

- **Order matters for chained constraints** – Constraints are evaluated in order; place the most restrictive or cheapest first for performance.
- **Custom constraints should be a last resort** – Model binding and action‑level validation are often better alternatives.

### Annotated Code Example: Applying Best Practices

```csharp
// ✅ Good: Anchored regex with timeout (via custom constraint)
[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost(string slug) => Ok(slug);

// ✅ Good: Use constraints for disambiguation, not validation
[HttpGet("products/{id:int}")]
public IActionResult GetProduct(int id)
{
    if (id <= 0) return BadRequest("ID must be positive."); // Validation in action
    return Ok(id);
}

// ❌ Avoid: Overly broad custom constraint for simple validation
// public class PositiveIntConstraint : IRouteConstraint { ... } // Unnecessary if action validates
```

**Why this example?** It demonstrates anchored regex, action‑level validation for business rules, and avoidance of unnecessary custom constraints.

### Real‑World Cases

- **API security:** Anchored regex constraints with timeouts prevent ReDoS.
- **Multi‑tenant routing:** Simple `alpha` constraints for tenant names, with validation in middleware.
- **Pagination:** `min(1)` constraints on page numbers, with upper bounds validated in the action.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Don’t use constraints for input validation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#dont-use-constraints-for-input-validation
- Regex timeout warning – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regular-expressions-in-constraints
- Custom constraints guidance – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#custom-route-constraints

---

## Summary Table: Route Constraints at a Glance

| Constraint | Purpose | Syntax | Best For |
|------------|---------|--------|----------|
| `int` | Validate integer | `{id:int}` | Numeric IDs |
| `bool` | Validate boolean | `{active:bool}` | Flags |
| `datetime` | Validate date/time | `{dob:datetime}` | Archives, scheduling |
| `decimal` | Validate decimal | `{price:decimal}` | Monetary values |
| `double` / `float` | Validate floating‑point | `{weight:double}` | Scientific data |
| `guid` | Validate GUID | `{id:guid}` | Distributed IDs |
| `long` | Validate 64‑bit integer | `{ticks:long}` | Large numeric IDs |
| `minlength` / `maxlength` / `length` | Validate string length | `{name:minlength(4)}` | Usernames, codes |
| `min` / `max` / `range` | Validate numeric range | `{age:range(18,120)}` | Age, month, pagination |
| `alpha` | Validate alphabetic | `{name:alpha}` | Author names |
| `regex` | Validate pattern | `{slug:regex(...)}` | Slugs, SSNs |
| `required` | Enforce presence during link generation | `{name:required}` | Canonical URLs |
| **Custom** | Domain‑specific rules | `{id:customName}` | Last‑resort disambiguation |

---

## References (Full List)

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- Route constraint reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraint-reference
- Don’t use constraints for input validation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#dont-use-constraints-for-input-validation
- Regular expressions in constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regular-expressions-in-constraints
- Custom route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#custom-route-constraints
- `IRouteConstraint` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- `int` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#int
- `bool` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#bool
- `datetime` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#datetime
- `decimal` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#decimal
- `double` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#double
- `float` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#float
- `guid` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#guid
- `long` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#long
- `minlength` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#minlength
- `maxlength` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#maxlength
- `length` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#length
- `min` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#min
- `max` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#max
- `range` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#range
- `alpha` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#alpha
- `regex` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regex
- `required` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#required