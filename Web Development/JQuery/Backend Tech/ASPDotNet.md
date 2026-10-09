# jQuery with ASP.NET — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** jQuery with ASP.NET is the practice of combining jQuery's client-side DOM manipulation and AJAX capabilities with ASP.NET Core's server-side framework to build dynamic, data-driven web applications. ASP.NET Core provides the backend routing, controllers, Razor Pages, model binding, validation, and real-time communication, while jQuery handles asynchronous requests, DOM updates, and interactive UI behavior on the frontend.

**Technical Definition:** jQuery with ASP.NET Core refers to the integration of two technologies: jQuery (a JavaScript library for DOM manipulation, event handling, and AJAX) and ASP.NET Core (a cross-platform, open-source web framework). The integration is achieved through HTTP requests initiated by jQuery's `$.ajax()`, `$.get()`, and `$.post()` methods, which target ASP.NET Core endpoints — Web API controllers, Minimal APIs, Razor Pages handler methods, or MVC controller actions. ASP.NET Core processes the requests using model binding, validates input via DataAnnotations and ModelState, and returns responses — typically JSON via `JsonResult` or `System.Text.Json`. jQuery then parses the responses and updates the DOM. For real-time communication, SignalR provides WebSocket-based bidirectional messaging with jQuery-compatible client libraries.

**Beginner-Friendly Explanation:** ASP.NET Core is the backend — it manages the database, handles security, and defines what URLs the application responds to. jQuery is the frontend — it talks to ASP.NET in the background and updates the page without reloading. This cheat sheet covers how to make the two work together smoothly: sending secure requests, handling validation errors, performing CRUD operations, and enabling real-time updates.

### Key Characteristics

- **Multiple AJAX endpoint types:** ASP.NET Core supports Web API controllers, Minimal APIs, Razor Pages handlers, and MVC controller actions as AJAX targets.
- **JSON serialization by default:** ASP.NET Core's `System.Text.Json` serializes to camelCase by default, which can mismatch PascalCase model properties.
- **Antiforgery protection:** ASP.NET Core automatically protects state-changing requests with antiforgery tokens, which jQuery must send via headers.
- **Server-side validation:** DataAnnotations and ModelState provide automatic client-side (via jQuery Unobtrusive Validation) and server-side validation.
- **SignalR for real-time:** SignalR enables WebSocket-based bidirectional communication with a jQuery-compatible client library.

### Prerequisites

- Proficiency in jQuery fundamentals: selectors, events, AJAX, and DOM manipulation.
- Working knowledge of ASP.NET Core: routing, controllers, Razor Pages, model binding, and validation.
- Understanding of HTTP methods, status codes, and JSON.
- Familiarity with C# and the .NET ecosystem.

### Related Programming Areas

- **MVC and Razor Pages:** ASP.NET Core's server-side rendering and page handler patterns.
- **Web API and RESTful Design:** JSON API endpoints for SPA and AJAX consumption.
- **Model Binding and Validation:** Mapping request data to ViewModels and validating input.
- **Real-Time Web:** SignalR for WebSocket-based communication.
- **Security:** Antiforgery tokens, CORS, and authentication.

### Core Concepts / Features

This cheat sheet covers six core concepts: AJAX endpoints, JSON serialization, form processing, server-side validation, antiforgery request tokens, and SignalR hub communication.

---

## Core Concept 1: AJAX Endpoints — Targeting Web API, Minimal APIs, or Razor Pages Handlers

### Definitions

**Core Definition:** AJAX endpoints in ASP.NET Core are server-side handlers that return JSON or partial HTML responses instead of full HTML views, designed to be consumed by jQuery AJAX requests rather than browser navigation.

**Technical Definition:** ASP.NET Core provides several mechanisms for defining AJAX endpoints: (1) **Web API controllers** — classes inheriting from `ControllerBase` with `[ApiController]` attribute, returning JSON via `JsonResult` or typed objects; (2) **Minimal APIs** — route handlers defined in `Program.cs` using `MapGet`, `MapPost`, etc.; (3) **Razor Pages handlers** — `OnGet`, `OnPost`, or named handlers (`OnPostGetCustomers`) in a PageModel that return `JsonResult` or `IActionResult`; and (4) **MVC controller actions** — traditional controller methods that return `JsonResult`. Each endpoint type has different routing conventions and attribute requirements.

**Beginner-Friendly Explanation:** An AJAX endpoint is like a specific department in a company. When you call the "customer service" extension, you get the customer service team. In ASP.NET Core, you can have different "departments" for different tasks: one for Web API calls, one for Razor Pages, one for Minimal APIs. jQuery dials the right extension (URL) and gets the data it needs.

### Purposes

- To define clear, RESTful endpoints for AJAX requests from jQuery.
- To return JSON or partial HTML without full page reloads.
- To leverage ASP.NET Core's model binding, validation, and dependency injection.
- To support both traditional MVC and modern Minimal API patterns.
- To enable Razor Pages handler methods for page-specific AJAX operations.

### Syntax Rules and Structure

**Complete General Syntax (Razor Pages Handler):**
```csharp
public class IndexModel : PageModel
{
    public void OnGet() { }

    [ValidateAntiForgeryToken]
    public IActionResult OnPostGetCustomers()
    {
        var customers = new List<CustomerModel>
        {
            new CustomerModel { CustomerId = 1, Name = "John Hammond", Country = "United States" },
            new CustomerModel { CustomerId = 2, Name = "Mudassar Khan", Country = "India" }
        };
        return new JsonResult(customers);
    }
}
```

**Complete General Syntax (Web API Controller):**
```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new[] { new { Id = 1, Name = "John" } });
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create([FromBody] CustomerModel model)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        return Ok(model);
    }
}
```

**Complete General Syntax (jQuery AJAX Target):**
```javascript
$.ajax({
    url: "/Index?handler=GetCustomers",
    type: "POST",
    dataType: "json",
    headers: { "XSRF-TOKEN": $('input[name="__RequestVerificationToken"]').val() },
    success: function(customers) { ... }
});
```

| Endpoint Type | URL Pattern | Return Type |
|---------------|-------------|-------------|
| Razor Pages Handler | `/Page?handler=HandlerName` | `JsonResult` |
| Web API | `/api/controller/action` | `Ok()`, `JsonResult` |
| Minimal API | `/endpoint` | `Results.Ok()`, `Results.Json()` |
| MVC Action | `/Controller/Action` | `JsonResult` |

**Syntax Rules:**

- For Razor Pages handlers, the URL pattern is `/PageName?handler=HandlerName`.
- Handler methods must be named `OnPostHandlerName` or `OnGetHandlerName`.
- Use `[ValidateAntiForgeryToken]` on POST handlers.
- Web API controllers should use `[ApiController]` for automatic model validation and binding.

**Constraints and Limitations:**

- Razor Pages handlers must be in the PageModel; they cannot be in separate files.
- Minimal APIs are defined in `Program.cs` and may become unwieldy for large applications.
- Web API controllers require the `[ApiController]` attribute for automatic 400 responses on validation failures.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Razor Pages Handler with jQuery AJAX**

```html
<!-- Pages/Index.cshtml -->
@page
@model IndexModel
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Razor Pages AJAX</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <button id="loadBtn">Load Customers</button>
    <ul id="customerList"></ul>

    <script>
        $(function() {
            $("#loadBtn").click(function() {
                $.ajax({
                    url: "/Index?handler=GetCustomers",
                    type: "POST",
                    dataType: "json",
                    headers: {
                        "XSRF-TOKEN": $('input[name="__RequestVerificationToken"]').val()
                    },
                    success: function(customers) {
                        var html = "";
                        $.each(customers, function(i, c) {
                            html += "<li>" + c.name + " — " + c.country + "</li>";
                        });
                        $("#customerList").html(html);
                    },
                    error: function(jqXHR) {
                        $("#customerList").html("<li>Error: " + jqXHR.status + "</li>");
                    }
                });
            });
        });
    </script>
</body>
</html>
```

```csharp
// Pages/Index.cshtml.cs
public class IndexModel : PageModel
{
    public void OnGet() { }

    [ValidateAntiForgeryToken]
    public IActionResult OnPostGetCustomers()
    {
        var customers = new List<CustomerModel>
        {
            new CustomerModel { CustomerId = 1, Name = "John Hammond", Country = "United States" },
            new CustomerModel { CustomerId = 2, Name = "Mudassar Khan", Country = "India" },
            new CustomerModel { CustomerId = 3, Name = "Suzanne Mathews", Country = "France" }
        };
        return new JsonResult(customers);
    }
}

public class CustomerModel
{
    public int CustomerId { get; set; }
    public string Name { get; set; }
    public string Country { get; set; }
}
```

**Expected Output:** Clicking "Load Customers" sends a POST request to the Razor Page handler. The handler returns a JSON array of customers. jQuery renders each customer as a list item.

**Why this output:** The `OnPostGetCustomers` handler is invoked via the URL `/Index?handler=GetCustomers`. The `[ValidateAntiForgeryToken]` attribute ensures the request includes a valid antiforgery token. The `JsonResult` serializes the list to JSON, which jQuery parses and renders.

### Real-World Cases

- **Data tables:** Loading paginated records via Razor Pages handlers.
- **Autocomplete:** Returning suggestions from a Minimal API endpoint.
- **SPA backends:** Exposing RESTful Web API controllers for jQuery consumption.
- **Dashboard widgets:** Fetching statistics from MVC controller actions.

---

## Core Concept 2: JSON — Managing PascalCase vs. camelCase Serialization Mismatches

### Definitions

**Core Definition:** JSON serialization in jQuery with ASP.NET Core is the process of converting C# objects to JSON strings on the server and parsing them into JavaScript objects on the client. A common issue is the mismatch between C#'s PascalCase property names and ASP.NET Core's default camelCase JSON output.

**Technical Definition:** ASP.NET Core's default JSON serializer is `System.Text.Json`, which uses a camelCase naming policy by default. This means a C# property `Name` is serialized as `"name"` in JSON. On the JavaScript side, jQuery parses the JSON into an object with camelCase properties. If the JavaScript code expects PascalCase (e.g., `user.Name`), it will receive `undefined`. To resolve this mismatch, developers can either (1) configure `System.Text.Json` to use `null` naming policy (preserving PascalCase), or (2) adapt the JavaScript code to use camelCase.

**Beginner-Friendly Explanation:** C# uses `PascalCase` for property names (e.g., `Name`, `Email`), but ASP.NET Core's JSON serializer converts them to `camelCase` (e.g., `name`, `email`) by default. JavaScript also uses camelCase, so the default behavior is usually correct. If you need PascalCase in JavaScript, you can change the serializer setting. Otherwise, use camelCase in your jQuery code.

### Purposes

- To provide a consistent JSON format between server and client.
- To avoid `undefined` errors caused by property name mismatches.
- To configure serialization globally for all AJAX responses.
- To support integration with JavaScript libraries that expect specific casing.

### Syntax Rules and Structure

**Complete General Syntax (Preserving PascalCase in Program.cs):**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages()
    .AddJsonOptions(options =>
        options.JsonSerializerOptions.PropertyNamingPolicy = null);

var app = builder.Build();
```

| Option | Behavior |
|--------|----------|
| `PropertyNamingPolicy = null` | Preserves PascalCase (`Name`, `Email`) |
| Default (not set) | Uses camelCase (`name`, `email`) |
| `JsonNamingPolicy.CamelCase` | Explicitly uses camelCase |

**Complete General Syntax (jQuery Consuming PascalCase):**
```javascript
$.getJSON("/api/users", function(users) {
    // With PropertyNamingPolicy = null, properties are PascalCase
    $.each(users, function(i, user) {
        console.log(user.Name, user.Email);
    });
});
```

**Syntax Rules:**

- Configure `PropertyNamingPolicy = null` in `AddJsonOptions()` or `AddControllersWithViews()` to preserve PascalCase.
- Alternatively, use camelCase in JavaScript and leave the default serializer policy.
- `System.Text.Json` is the default serializer in ASP.NET Core 3.0+; `Newtonsoft.Json` can be used as an alternative.
- The `[JsonPropertyName]` attribute can be used for per-property overrides.

**Constraints and Limitations:**

- Changing the global naming policy affects all JSON responses in the application.
- `Newtonsoft.Json` uses PascalCase by default, which differs from `System.Text.Json`.
- Some JavaScript libraries (e.g., Kendo UI Grid) expect camelCase by default and may fail with PascalCase.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Configuring PascalCase Serialization**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages()
    .AddJsonOptions(options =>
        options.JsonSerializerOptions.PropertyNamingPolicy = null);

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();
app.Run();
```

```csharp
// Pages/Index.cshtml.cs
public class IndexModel : PageModel
{
    [ValidateAntiForgeryToken]
    public IActionResult OnPostGetPerson()
    {
        var person = new PersonModel
        {
            Name = "John Hammond",
            DateTime = DateTime.Now.ToString()
        };
        return new JsonResult(person);
    }
}

public class PersonModel
{
    public string Name { get; set; }
    public string DateTime { get; set; }
}
```

```html
<!-- Pages/Index.cshtml -->
<script>
$(function() {
    $("#loadBtn").click(function() {
        $.ajax({
            url: "/Index?handler=GetPerson",
            type: "POST",
            dataType: "json",
            headers: { "XSRF-TOKEN": $('input[name="__RequestVerificationToken"]').val() },
            success: function(person) {
                // PascalCase properties are preserved
                $("#result").text(person.Name + " — " + person.DateTime);
            }
        });
    });
});
</script>
```

**Expected Output:** The result displays "John Hammond — [current datetime]" with PascalCase property access working correctly.

**Why this output:** The `AddJsonOptions` configuration sets `PropertyNamingPolicy = null`, which preserves the original PascalCase property names in the JSON response. jQuery accesses `person.Name` and `person.DateTime` directly.

### Real-World Cases

- **Kendo UI Grid integration:** Configuring PascalCase to match the grid's data source expectations.
- **Legacy JavaScript code:** Maintaining PascalCase for compatibility with existing code.
- **API consistency:** Ensuring all endpoints use the same serialization policy.

---

## Core Concept 3: Form Processing — Mapping Form Serializations to Backend ViewModels

### Definitions

**Core Definition:** Form processing with jQuery and ASP.NET Core is the practice of serializing HTML form data on the client side and sending it to an ASP.NET Core endpoint, where model binding maps the incoming data to a strongly-typed ViewModel or model.

**Technical Definition:** jQuery's `.serialize()` method converts form data into a URL-encoded string (e.g., `name=John&email=john@example.com`). ASP.NET Core's model binding automatically maps this data to action method parameters or ViewModel properties by name. For complex objects, the form field names must match the ViewModel property names (including nested property paths like `Address.Street`). For JSON payloads, the data is sent with `contentType: "application/json"` and serialized using `JSON.stringify()`, and the endpoint uses `[FromBody]` to bind the JSON to the model.

**Beginner-Friendly Explanation:** A form is like a paper application. jQuery takes the form data and writes it in a standardized format. ASP.NET Core reads that format and fills in a matching "form" on the server side (the ViewModel). The field names in the HTML form must match the property names in the ViewModel, or the server will not know where to put the data.

### Purposes

- To send form data to the server without a full page reload.
- To leverage ASP.NET Core's strong model binding and validation.
- To support both URL-encoded and JSON payloads.
- To map complex nested ViewModels from form fields.
- To provide server-side validation feedback without losing user input.

### Syntax Rules and Structure

**Complete General Syntax (URL-Encoded Form Data):**
```javascript
$("#myForm").on("submit", function(e) {
    e.preventDefault();
    $.ajax({
        url: "/api/save",
        type: "POST",
        data: $(this).serialize(),
        dataType: "json",
        success: function(response) { ... }
    });
});
```

**Complete General Syntax (JSON Payload):**
```javascript
var formData = {
    Name: $("#name").val(),
    Email: $("#email").val(),
    Address: {
        Street: $("#street").val(),
        City: $("#city").val()
    }
};

$.ajax({
    url: "/api/save",
    type: "POST",
    contentType: "application/json",
    data: JSON.stringify(formData),
    success: function(response) { ... }
});
```

**Complete General Syntax (ASP.NET Core ViewModel):**
```csharp
public class UserViewModel
{
    public string Name { get; set; }
    public string Email { get; set; }
    public AddressViewModel Address { get; set; }
}

public class AddressViewModel
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

| jQuery Method | ASP.NET Core Binding |
|---------------|---------------------|
| `.serialize()` | `[FromForm]` |
| `JSON.stringify()` | `[FromBody]` |
| `FormData` | `[FromForm]` + `IFormFile` |

**Syntax Rules:**

- Field names in the HTML form must match ViewModel property names.
- For nested objects, use dot notation in field names: `<input name="Address.Street">`.
- Use `contentType: "application/json"` and `JSON.stringify()` for JSON payloads.
- Use `[FromBody]` for JSON binding and `[FromForm]` for form data binding.

**Constraints and Limitations:**

- `.serialize()` does not include file inputs; use `FormData` for file uploads.
- JSON payloads require `[FromBody]` on the server; without it, binding fails.
- Case sensitivity of property names depends on the serializer configuration.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Mapping Form Data to a Nested ViewModel**

```html
<!-- Pages/Create.cshtml -->
<form id="userForm">
    <input type="text" name="Name" placeholder="Name">
    <input type="email" name="Email" placeholder="Email">
    <input type="text" name="Address.Street" placeholder="Street">
    <input type="text" name="Address.City" placeholder="City">
    <button type="submit">Save</button>
</form>
<div id="result"></div>

<script>
$(function() {
    $("#userForm").on("submit", function(e) {
        e.preventDefault();
        $.ajax({
            url: "/Index?handler=SaveUser",
            type: "POST",
            data: $(this).serialize(),
            dataType: "json",
            headers: { "XSRF-TOKEN": $('input[name="__RequestVerificationToken"]').val() },
            success: function(response) {
                $("#result").text("Saved: " + response.name + " (" + response.email + ")");
            },
            error: function(jqXHR) {
                $("#result").text("Error: " + jqXHR.status);
            }
        });
    });
});
</script>
```

```csharp
// Pages/Create.cshtml.cs
public class CreateModel : PageModel
{
    [ValidateAntiForgeryToken]
    public IActionResult OnPostSaveUser([FromForm] UserViewModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        return new JsonResult(new { model.Name, model.Email });
    }
}

public class UserViewModel
{
    public string Name { get; set; }
    public string Email { get; set; }
    public AddressViewModel Address { get; set; }
}

public class AddressViewModel
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

**Expected Output:** Submitting the form with valid data displays "Saved: John (john@example.com)". The nested `Address` ViewModel is bound from the `Address.Street` and `Address.City` form fields.

**Why this output:** The form field names (`Name`, `Email`, `Address.Street`, `Address.City`) match the ViewModel property paths. ASP.NET Core's model binding maps the URL-encoded form data to the `UserViewModel` and its nested `AddressViewModel`.

### Real-World Cases

- **Registration forms:** Mapping user details to a `RegisterViewModel`.
- **Checkout forms:** Mapping shipping and billing addresses to nested ViewModels.
- **Profile editing:** Mapping partial updates to a `ProfileViewModel`.
- **Survey forms:** Mapping dynamic form fields to a collection property.

---

## Core Concept 4: Server-Side Validation — Intercepting ModelState Errors and Displaying Them Inline

### Definitions

**Core Definition:** Server-side validation in jQuery with ASP.NET Core is the practice of validating input on the server using DataAnnotations and ModelState, returning a 400 (Bad Request) response with validation errors, and displaying those errors inline next to the corresponding form fields using jQuery.

**Technical Definition:** ASP.NET Core uses DataAnnotations attributes (`[Required]`, `[StringLength]`, `[EmailAddress]`, `[Range]`) on ViewModel properties to define validation rules. When a request is received, the framework runs these validations and populates the `ModelState` dictionary. If `ModelState.IsValid` is false, the server returns a 400 status code with a JSON payload containing the errors. jQuery's `error` callback receives the `jqXHR` object, whose `responseJSON` property contains the ModelState errors. The frontend iterates over the errors and displays each message next to the corresponding input. For client-side validation, ASP.NET Core's jQuery Unobtrusive Validation automatically parses DataAnnotations-generated `data-*` attributes and performs validation before submission.

**Beginner-Friendly Explanation:** When you submit a form, ASP.NET Core checks whether the data meets the rules (e.g., "name is required", "email must be valid"). If something is wrong, it sends back a list of errors, each labeled with the field name. jQuery reads this list and shows the error message next to the right field, so the user knows exactly what to fix.

### Purposes

- To display field-specific validation errors without page reloads.
- To provide immediate feedback on form submission.
- To keep validation rules in one place (the ViewModel) and apply them consistently.
- To handle complex validation scenarios (unique emails, conditional rules) that only the server can check.
- To support both client-side and server-side validation seamlessly.

### Syntax Rules and Structure

**Complete General Syntax (ViewModel with DataAnnotations):**
```csharp
using System.ComponentModel.DataAnnotations;

public class RegisterViewModel
{
    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100, MinimumLength = 2, ErrorMessage = "Name must be 2-100 characters.")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Email is required.")]
    [EmailAddress(ErrorMessage = "Please enter a valid email address.")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Password is required.")]
    [StringLength(100, MinimumLength = 8, ErrorMessage = "Password must be at least 8 characters.")]
    public string Password { get; set; }
}
```

**Complete General Syntax (jQuery Error Handling):**
```javascript
$.ajax({
    url: "/api/register",
    type: "POST",
    data: formData,
    dataType: "json",
    success: function(response) { ... },
    error: function(jqXHR) {
        if (jqXHR.status === 400) {
            var errors = jqXHR.responseJSON;
            $.each(errors, function(field, messages) {
                $("[name='" + field + "']").addClass("is-invalid");
                $("[data-error-for='" + field + "']").text(messages[0]);
            });
        }
    }
});
```

| Component | Description |
|-----------|-------------|
| `jqXHR.status` | HTTP status code; 400 for validation errors. |
| `jqXHR.responseJSON` | Object mapping field names to error message arrays. |
| `errors[field][0]` | The first error message for a field. |
| `[data-error-for]` | Custom attribute for error message containers. |

**Syntax Rules:**

- Add DataAnnotations attributes to ViewModel properties for automatic validation.
- Use `[ValidateAntiForgeryToken]` on POST handlers.
- Check `jqXHR.status === 400` before parsing `responseJSON`.
- Clear previous errors before displaying new ones.
- Use `$.validator.unobtrusive.parse(form)` to enable client-side validation on dynamically loaded forms.

**Constraints and Limitations:**

- ModelState errors are only returned as JSON when the request expects JSON (i.e., `Accept: application/json` or `X-Requested-With: XMLHttpRequest`).
- jQuery Unobtrusive Validation does not work automatically on dynamically generated forms; `$.validator.unobtrusive.parse()` must be called after inserting the form.
- Some validation attributes (e.g., `[Range]` with `DateTime`) may not work correctly with client-side validation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Inline Validation Error Display**

```html
<!-- Pages/Register.cshtml -->
<form id="registerForm">
    <div>
        <input type="text" name="Name" placeholder="Name">
        <span class="error-message" data-error-for="Name"></span>
    </div>
    <div>
        <input type="email" name="Email" placeholder="Email">
        <span class="error-message" data-error-for="Email"></span>
    </div>
    <div>
        <input type="password" name="Password" placeholder="Password">
        <span class="error-message" data-error-for="Password"></span>
    </div>
    <button type="submit">Register</button>
</form>
<div id="successMessage"></div>

<script>
$(function() {
    $("#registerForm").on("submit", function(e) {
        e.preventDefault();
        var $form = $(this);

        // Clear previous errors
        $form.find(".is-invalid").removeClass("is-invalid");
        $form.find(".error-message").text("");
        $("#successMessage").text("");

        $.ajax({
            url: "/Register?handler=Register",
            type: "POST",
            data: $form.serialize(),
            dataType: "json",
            headers: { "XSRF-TOKEN": $('input[name="__RequestVerificationToken"]').val() },
            success: function(response) {
                $("#successMessage").text("Registration successful!");
                $form[0].reset();
            },
            error: function(jqXHR) {
                if (jqXHR.status === 400) {
                    var errors = jqXHR.responseJSON;
                    $.each(errors, function(field, messages) {
                        $("[name='" + field + "']").addClass("is-invalid");
                        $("[data-error-for='" + field + "']").text(messages[0]);
                    });
                } else {
                    $("#successMessage").text("An unexpected error occurred.");
                }
            }
        });
    });
});
</script>
```

```csharp
// Pages/Register.cshtml.cs
public class RegisterModel : PageModel
{
    [ValidateAntiForgeryToken]
    public IActionResult OnPostRegister([FromForm] RegisterViewModel model)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        return new JsonResult(new { success = true });
    }
}
```

**Expected Output:** Submitting the form with an empty name, invalid email, and short password displays "Name is required", "Please enter a valid email address", and "Password must be at least 8 characters" next to the respective fields. The fields are highlighted with a red border. When the form is corrected and submitted, "Registration successful!" is displayed.

**Why this output:** The `RegisterViewModel` has DataAnnotations attributes that define validation rules. ASP.NET Core validates the incoming form data, and if `ModelState.IsValid` is false, returns a 400 response with the errors. jQuery's `error` callback checks for 400, iterates over `responseJSON`, and displays the first message for each field.

### Real-World Cases

- **Registration forms:** Validating email uniqueness, password strength, and required fields.
- **Checkout forms:** Validating credit card numbers, expiry dates, and shipping addresses.
- **Profile updates:** Validating name, email, and password changes.
- **Data entry forms:** Validating numeric ranges, date formats, and required selections.

---

## Core Concept 5: Antiforgery Request Tokens — Extracting and Attaching `__RequestVerificationToken` Headers

### Definitions

**Core Definition:** Antiforgery request tokens in jQuery with ASP.NET Core are the mechanism by which the server generates a unique token per session, embeds it in the page (via a hidden form field or JavaScript-readable cookie), and validates it on state-changing requests to prevent Cross-Site Request Forgery (CSRF) attacks.

**Technical Definition:** ASP.NET Core's antiforgery system generates two tokens: a cookie token (stored in a cookie named `.AspNetCore.Antiforgery.*`) and a request token (embedded in the page as a hidden input named `__RequestVerificationToken` or provided via `IAntiforgery.GetAndStoreTokens()`). For AJAX requests, the request token must be sent in a custom header. The header name is configurable via `services.AddAntiforgery(options => options.HeaderName = "X-CSRF-TOKEN")`. jQuery reads the token from the hidden input or a `<meta>` tag and sets it as a default header via `$.ajaxSetup()` or per-request via `beforeSend`. The `[ValidateAntiForgeryToken]` attribute or `[AutoValidateAntiforgeryToken]` on the controller/PageModel validates the token.

**Beginner-Friendly Explanation:** ASP.NET Core has a security guard that only lets in requests carrying a special ticket. The ticket is generated by the server and hidden in the page. jQuery reads the ticket and shows it to the guard on every state-changing request. Without the ticket, the request is rejected with a 400 error.

### Purposes

- To protect against Cross-Site Request Forgery attacks on all state-changing AJAX routes.
- To automatically include the antiforgery token in every AJAX request.
- To configure the header name to match the server's expectation.
- To use `@Html.AntiForgeryToken()` or `IAntiforgery` to generate the token in Razor views.
- To support both form-encoded and JSON payloads.

### Syntax Rules and Structure

**Complete General Syntax (Server Configuration):**
```csharp
// Program.cs
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";
    options.Cookie.Name = "MyApp.Antiforgery";
});
```

**Complete General Syntax (Razor View — Token Generation):**
```html
@Html.AntiForgeryToken()

<!-- Or via a meta tag -->
<meta name="csrf-token" content="@requestToken">
```

```csharp
// PageModel
public void OnGet()
{
    var tokens = _antiforgery.GetAndStoreTokens(HttpContext);
    ViewData["RequestToken"] = tokens.RequestToken;
}
```

**Complete General Syntax (jQuery Global Setup):**
```javascript
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRF-TOKEN",
                $('input[name="__RequestVerificationToken"]').val());
        }
    }
});
```

| Component | Description |
|-----------|-------------|
| `options.HeaderName` | The header name the server expects (default: `"X-CSRF-TOKEN"`). |
| `@Html.AntiForgeryToken()` | Generates a hidden input with the token. |
| `IAntiforgery.GetAndStoreTokens()` | Generates and stores tokens, returning the request token. |
| `[ValidateAntiForgeryToken]` | Attribute that validates the token on POST actions. |

**Syntax Rules:**

- Configure the header name in `AddAntiforgery()` to match the jQuery header.
- Generate the token in the Razor view using `@Html.AntiForgeryToken()` or `IAntiforgery`.
- Use `$.ajaxSetup()` with `beforeSend` to add the token to all state-changing requests.
- Exclude safe HTTP methods (GET, HEAD, OPTIONS, TRACE) and cross-domain requests.
- The `[ValidateAntiForgeryToken]` attribute must be present on POST handlers.

**Constraints and Limitations:**

- The token must be present in the DOM before the AJAX request is made; if the form is loaded dynamically, the token must be included in the partial view.
- In subdomain applications, cookie configuration may require additional settings (`options.Cookie.Name` and `options.Cookie.Domain`).
- The `RequestVerificationToken` header may be stripped by some proxies; verify the header name matches the server configuration.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Antiforgery Token with jQuery AJAX**

```html
<!-- Pages/Index.cshtml -->
@page
@model IndexModel
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="csrf-token" content="@ViewData["RequestToken"]" />
    <title>Antiforgery Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <button id="updateBtn">Update Profile</button>
    <div id="result"></div>

    <script>
        $(function() {
            // Read the token from the meta tag
            var csrfToken = $('meta[name="csrf-token"]').attr("content");

            // Configure global AJAX setup
            $.ajaxSetup({
                beforeSend: function(xhr, settings) {
                    if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
                        xhr.setRequestHeader("X-CSRF-TOKEN", csrfToken);
                    }
                }
            });

            $("#updateBtn").click(function() {
                $.ajax({
                    url: "/Index?handler=UpdateProfile",
                    type: "POST",
                    data: { name: "Jane Developer" },
                    success: function(response) {
                        $("#result").text("Profile updated: " + response.name);
                    },
                    error: function(jqXHR) {
                        $("#result").text("Error: " + jqXHR.status);
                    }
                });
            });
        });
    </script>
</body>
</html>
```

```csharp
// Pages/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IAntiforgery _antiforgery;

    public IndexModel(IAntiforgery antiforgery)
    {
        _antiforgery = antiforgery;
    }

    public void OnGet()
    {
        var tokens = _antiforgery.GetAndStoreTokens(HttpContext);
        ViewData["RequestToken"] = tokens.RequestToken;
    }

    [ValidateAntiForgeryToken]
    public IActionResult OnPostUpdateProfile(string name)
    {
        return new JsonResult(new { name = name });
    }
}
```

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";
});

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();
app.Run();
```

**Expected Output:** Clicking "Update Profile" sends a POST request with the `X-CSRF-TOKEN` header. The server validates the token, processes the request, and returns JSON. The result displays "Profile updated: Jane Developer."

**Why this output:** The `GetAndStoreTokens` method generates a request token, which is embedded in the meta tag. jQuery reads the token and sets it as the `X-CSRF-TOKEN` header on all state-changing requests. The `[ValidateAntiForgeryToken]` attribute on the handler validates the token before processing the request.

### Real-World Cases

- **Profile updates:** Any authenticated user action that modifies data.
- **Settings pages:** Saving user preferences via AJAX.
- **Admin panels:** Performing CRUD operations protected by antiforgery tokens.
- **Form submissions:** All POST, PUT, and DELETE AJAX requests.

---

## Core Concept 6: SignalR Hub Communication — Using jQuery Alongside SignalR for Real-Time WebSocket Fallback

### Definitions

**Core Definition:** SignalR hub communication with jQuery is the practice of using SignalR — ASP.NET Core's real-time communication library — alongside jQuery to enable bidirectional, WebSocket-based messaging between the server and the browser, with automatic fallback to other transports (Server-Sent Events, Long Polling) when WebSockets are unavailable.

**Technical Definition:** SignalR provides a hub abstraction that allows the server to call JavaScript functions on connected clients and clients to call server methods. The SignalR JavaScript client library (`@microsoft/signalr`) can be used independently of jQuery, but projects that already use jQuery can integrate it seamlessly. The client is configured with `new signalR.HubConnectionBuilder().withUrl("/hub").build()`, and handlers are registered with `connection.on("EventName", callback)`. The server uses `IHubContext<T>` to send messages to all clients (`Clients.All`), specific clients (`Clients.Client(id)`), or groups (`Clients.Group(name)`). SignalR automatically negotiates the best transport and falls back to Long Polling if WebSockets are blocked.

**Beginner-Friendly Explanation:** SignalR is like a telephone line between the server and the browser. The server can call the browser and say "here is new data," and the browser can call the server and say "give me an update." jQuery can listen for these calls and update the page instantly. If the phone line (WebSocket) is not available, SignalR automatically uses a slower but reliable method (Long Polling) without the user noticing.

### Purposes

- To enable real-time updates (dashboards, notifications, chat) without polling.
- To provide automatic transport fallback for environments where WebSockets are blocked.
- To integrate real-time communication into existing jQuery-based applications.
- To leverage ASP.NET Core's dependency injection and authentication with SignalR.
- To support groups for targeted messaging (per-user, per-room).

### Syntax Rules and Structure

**Complete General Syntax (Server Hub):**
```csharp
public class DashboardHub : Hub
{
    public async Task SendMessage(string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", message);
    }
}
```

**Complete General Syntax (Server Registration):**
```csharp
// Program.cs
builder.Services.AddSignalR();
app.MapHub<DashboardHub>("/dashboardhub");
```

**Complete General Syntax (jQuery Client):**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/7.0.5/signalr.min.js"></script>
<script>
$(function() {
    const connection = new signalR.HubConnectionBuilder()
        .withUrl("/dashboardhub")
        .withAutomaticReconnect()
        .build();

    connection.on("ReceiveMessage", function(message) {
        // Use jQuery to update the DOM
        $("#messages").append("<div>" + message + "</div>");
    });

    connection.start()
        .then(function() {
            console.log("SignalR connected.");
        })
        .catch(function(err) {
            console.error("SignalR connection error:", err);
        });
});
</script>
```

| Component | Description |
|-----------|-------------|
| `HubConnectionBuilder` | Creates a new SignalR connection. |
| `.withUrl("/hub")` | Specifies the hub endpoint. |
| `.withAutomaticReconnect()` | Automatically reconnects on disconnection. |
| `connection.on("Event", callback)` | Registers a handler for server-to-client messages. |
| `connection.invoke("Method", args)` | Calls a server method from the client. |

**Syntax Rules:**

- The SignalR JavaScript client library is separate from jQuery and must be loaded before use.
- Use `connection.on()` to register client-side handlers for server messages.
- Use `connection.invoke()` to call server hub methods.
- Use `withAutomaticReconnect()` for production applications to handle network interruptions.
- The server registers the hub with `app.MapHub<THub>("/path")`.

**Constraints and Limitations:**

- SignalR's JavaScript client does not require jQuery; it can be used with vanilla JavaScript or other frameworks.
- WebSocket transport requires a full-duplex connection; some proxies and firewalls may block it, triggering fallback to Long Polling.
- SignalR connections are not shared across browser tabs by default; each tab establishes its own connection.
- Authentication with SignalR requires the `accessTokenFactory` option or cookie-based authentication.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Real-Time Dashboard with SignalR and jQuery**

```csharp
// Hubs/DashboardHub.cs
using Microsoft.AspNetCore.SignalR;

public class DashboardHub : Hub
{
    public async Task SendMessage(string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", message);
    }

    public async Task SendUpdate(string data)
    {
        await Clients.All.SendAsync("ReceiveUpdate", data);
    }
}
```

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddSignalR();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();
app.MapHub<DashboardHub>("/dashboardhub");
app.Run();
```

```html
<!-- Pages/Index.cshtml -->
@page
@model IndexModel
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>SignalR Dashboard</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/7.0.5/signalr.min.js"></script>
</head>
<body>
    <h1>Real-Time Dashboard</h1>
    <div id="messages"></div>
    <button id="sendBtn">Send Test Message</button>

    <script>
        $(function() {
            // Build the SignalR connection
            const connection = new signalR.HubConnectionBuilder()
                .withUrl("/dashboardhub")
                .withAutomaticReconnect()
                .build();

            // Register handlers for server messages
            connection.on("ReceiveMessage", function(message) {
                $("#messages").append("<div class='message'>" + message + "</div>");
            });

            connection.on("ReceiveUpdate", function(data) {
                $("#messages").append("<div class='update'>Update: " + data + "</div>");
            });

            // Start the connection
            connection.start()
                .then(function() {
                    console.log("SignalR connected.");
                })
                .catch(function(err) {
                    console.error("SignalR connection error:", err);
                });

            // Send a message to the server
            $("#sendBtn").click(function() {
                connection.invoke("SendMessage", "Hello from client at " + new Date().toLocaleTimeString())
                    .catch(function(err) {
                        console.error("Send error:", err);
                    });
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** When the page loads, the SignalR connection is established. Clicking "Send Test Message" invokes the `SendMessage` method on the server, which broadcasts the message to all connected clients. The message appears in the `#messages` div on all connected browsers.

**Why this output:** The `HubConnectionBuilder` creates a connection to the `/dashboardhub` endpoint. The `connection.on` handlers listen for `ReceiveMessage` and `ReceiveUpdate` events from the server. The `connection.invoke` call sends a message to the server, which broadcasts it to all clients via `Clients.All.SendAsync`.

### Real-World Cases

- **Real-time dashboards:** Live metrics, stock prices, and system monitoring.
- **Chat applications:** Instant messaging between users.
- **Notifications:** Real-time alerts and updates.
- **Collaborative editing:** Multiple users editing the same document in real-time.
- **IoT dashboards:** Live sensor data from connected devices.

---

## References

- ASP.NET Core Razor Pages: Using jQuery AJAX — ASPSnippets — https://www.aspsnippets.com/Articles/5360/ASPNet-Core-8-Using-jQuery-AJAX-in-Razor-Pages/
- ASP.NET Core Razor Pages: Return List Collection from Handler — ASPSnippets — https://www.aspsnippets.com/Articles/5406/ASPNet-Core-Razor-Pages-Return-List-collection-from-Handler-method-using-jQuery-AJAX
- Cannot Get Data to Load in Grid (camelCase vs PascalCase) — Telerik — https://www.telerik.com/kendo-jquery-ui/documentation/knowledge-base/grid-is-not-showing-data
- System.Text.Json Deserialize camelCase to PascalCase — ASPSnippets — https://www.aspsnippets.com/Articles/5620/Net-Core-7-SystemTextJson-JsonSerializer-deserialize-camelCase-to-PascalCase/
- Part 8: Add Validation — Microsoft Learn — https://learn.microsoft.com/en-au/aspnet/core/tutorials/razor-pages/validation?view=aspnetcore-8.0
- Validate AntiforgeryToken via jQuery Ajax — Stack Overflow — https://stackoverflow.com/feeds/question/70145708
- Building Real-Time Dashboards with SignalR — Intertoons — https://intertoons.com/building-real-time-dashboards-with-asp-net-core-signalr-and-sql-server.html
- Model Validation in ASP.NET Core MVC (Dynamic Forms) — Microsoft Learn — https://learn.microsoft.com/cs-cz/aspnet/core/mvc/models/validation?view=aspnetcore-8.0
- Preventing CSRF Attacks in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-au/aspnet/core/security/anti-request-forgery
- SignalR JavaScript Client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/javascript-client
- ASP.NET Core SignalR Hubs — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs
- jQuery.ajaxSetup() — jQuery API Documentation — https://api.jquery.com/jQuery.ajaxSetup/
- jQuery.serialize() — jQuery API Documentation — https://api.jquery.com/serialize/