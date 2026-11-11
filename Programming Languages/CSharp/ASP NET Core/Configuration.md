# C# Configuration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Configuration in .NET is the mechanism by which an application reads its settings from **external sources**—JSON files, environment variables, command-line arguments, secret stores, and cloud services—rather than hard-coding them. It provides a **layered, key-value model** where later sources override earlier ones, and it integrates with **strongly typed binding** (the Options pattern) so that settings are consumed as validated, type-safe objects rather than raw strings.

**Technical Definition**
The .NET configuration system is built on **`IConfiguration`** and its concrete implementation **`IConfigurationRoot`**. Configuration is assembled from **`IConfigurationProvider`** instances (JSON, environment variables, command line, User Secrets, Azure Key Vault, etc.) that are chained together. Providers are read in the order they are added; for a matching key, the **last provider wins**. The default ASP.NET Core host adds providers in this order: `appsettings.json` → `appsettings.{Environment}.json` → User Secrets (Development only) → environment variables → command-line arguments. The **Options pattern** bridges configuration and dependency injection: `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>` provide strongly typed access with different lifetimes and reload semantics.

**Beginner-Friendly Explanation**
Configuration is **where your app reads its settings**. Instead of writing `const string ConnectionString = "..."` in code, you put the value in `appsettings.json`, override it with an environment variable in production, and read it as a strongly typed object. The configuration system checks **multiple sources in order**—later sources win—so you can keep defaults in JSON, per-environment overrides in another JSON file, secrets in a secure store, and deployment-specific values in environment variables. The Options pattern turns those raw strings into **typed classes** with validation, so your services never touch `IConfiguration` directly.

---

### Key Characteristics

- **Layered providers** — Configuration is composed from multiple sources; later providers override earlier ones.
- **Key-value model** — Hierarchical keys use `:` (JSON) or `__` (environment variables, cross-platform).
- **`reloadOnChange`** — JSON providers reload automatically when the file changes (enabled by default).
- **Strongly typed binding** — The Options pattern binds configuration sections to POCOs with validation.
- **Three Options interfaces** — `IOptions<T>` (singleton, startup value), `IOptionsSnapshot<T>` (scoped, per-request), `IOptionsMonitor<T>` (singleton, change notifications).
- **Secrets separation** — User Secrets for development, Azure Key Vault or environment variables for production.
- **Never commit secrets** — Sensitive values must not live in source control.

---

### Prerequisites

- Basic understanding of **ASP.NET Core** (`Program.cs`, `WebApplicationBuilder`) or the **Generic Host**.
- Familiarity with **dependency injection** (`IServiceCollection`, `IServiceProvider`).
- Knowledge of **JSON** syntax.
- Awareness of **environment variables** and **`IDisposable`**.
- Understanding of **POCOs** and **DataAnnotations** (for validation).

---

### Related Programming Areas

- **Dependency Injection** — The Options pattern is built on DI.
- **Options Pattern** — The strongly typed consumption model for configuration.
- **Azure Key Vault** — Production secrets management.
- **User Secrets** — Development secrets management.
- **Environment-specific configuration** — `appsettings.{Environment}.json`.

---

### Core Concepts / Features

1. Configuration (Overarching Concept)
2. `IConfiguration`
3. `appsettings.json`
4. Environment Variables
5. User Secrets
6. Azure Key Vault
7. Configuration Binding
8. Options Pattern
9. `IOptions<T>`
10. `IOptionsSnapshot<T>`
11. `IOptionsMonitor<T>`
12. Configuration Best Practices

---

## 1. Configuration (Overarching Concept)

### Definitions

**Core Definition**
Configuration is the externalized settings that an application reads at startup and, optionally, at runtime.

**Technical Definition**
Configuration is a set of key-value pairs aggregated from providers. The `IConfiguration` abstraction exposes indexers (`config["Key"]`), section access (`GetSection`), and value binding (`GetValue<T>`). The configuration root is built by `ConfigurationBuilder` and consumed by the host, DI, and application code.

**Beginner-Friendly Explanation**
Configuration is **all the knobs and dials your app needs**—connection strings, feature flags, API keys, timeouts—kept outside the code so they can change per environment.

---

### Purposes

- To **separate settings from code**.
- To support **multiple environments** (Development, Staging, Production).
- To allow **deployment-time overrides** without rebuilding.
- To **secure sensitive values** via secret stores.
- To enable **runtime changes** via reloadable providers.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);
// Configuration is available as builder.Configuration
var conn = builder.Configuration["ConnectionStrings:DefaultConnection"];
```

**Component breakdown**

- `builder.Configuration` — `IConfiguration` instance.
- `["Key:SubKey"]` — Indexer access.
- `GetSection("Section")` — Returns an `IConfigurationSection`.

#### Syntax Rules

- Configuration is assembled before `builder.Build()`.
- Keys are case-insensitive.
- Hierarchical keys use `:` in JSON and code; environment variables use `__`.

#### Constraints and Limitations

- Configuration values are strings; type conversion is performed by binding or `GetValue<T>`.
- Direct indexer access is discouraged in application code (use the Options pattern).

---

### Annotated Code Examples and Expected Outputs

#### Example: Reading Configuration Directly (Not Recommended for Services)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", (IConfiguration config) =>
    config["Logging:LogLevel:Default"] ?? "not set");

app.Run();
```

**Expected Output (HTTP response)**
```
Information
```

**Why this output occurs**
The default `appsettings.json` contains `"Logging": { "LogLevel": { "Default": "Information" } }`; the indexer reads it using the `:` separator.

---

### Real-World Cases

- **Connection strings** — `ConnectionStrings:DefaultConnection`.
- **Feature flags** — `Features:NewDashboard`.
- **Timeouts and limits** — `HttpClient:TimeoutSeconds`.
- **Logging levels** — `Logging:LogLevel:Default`.

---

### References

- Configuration in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/

---

## 2. `IConfiguration`

### Definitions

**Core Definition**
`IConfiguration` is the interface that exposes configuration key-value pairs and sections.

**Technical Definition**
`Microsoft.Extensions.Configuration.IConfiguration` provides an indexer (`string this[string key]`), `GetSection(string)`, `GetChildren()`, and `GetReloadToken()`. The concrete root (`IConfigurationRoot`) aggregates providers and resolves keys by querying them in reverse order (last provider wins). Extension methods (`GetValue<T>`, `GetConnectionString`) add convenience.

**Beginner-Friendly Explanation**
`IConfiguration` is **the configuration API**—you ask it for a key and it returns the value from the highest-priority provider.

---

### Purposes

- To **read individual values** by key.
- To **navigate sections** via `GetSection`.
- To **observe reloads** via `GetReloadToken`.
- To **bind sections** to POCOs via `Get<T>` or `Bind`.

---

### Syntax Rules and Structure

```csharp
string? value = config["Key:SubKey"];
IConfigurationSection section = config.GetSection("Section");
T? settings = section.Get<T>();
```

**Component breakdown**

- `config["Key"]` — Indexer; returns `null` if missing.
- `GetSection("Name")` — Returns a section (never `null`).
- `Get<T>()` — Binds and returns a new instance.

#### Syntax Rules

- Keys are case-insensitive.
- `GetSection` returns an empty section if missing (not `null`).
- `Get<T>` allocates a new object each call; for repeated access, use the Options pattern.

#### Constraints and Limitations

- `IConfiguration` is read-only; writing configuration back is not supported by default.
- Direct injection of `IConfiguration` into services is an anti-pattern; prefer `IOptions<T>`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Reading a Section

```csharp
var section = builder.Configuration.GetSection("Position");
var title = section["Title"];   // "Editor"
```

**Expected Output (conceptual)**
```
Title = "Editor"
```

**Why this output occurs**
`GetSection` returns a view over the configuration subtree; the indexer reads the child key.

---

### Real-World Cases

- **Middleware/startup code** — Reading configuration before DI is available.
- **Health checks** — Checking configured endpoints.
- **Diagnostics** — Enumerating providers.

---

### References

- `IConfiguration` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration
- Configuration in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/

---

## 3. `appsettings.json`

### Definitions

**Core Definition**
`appsettings.json` is the default JSON configuration file for .NET applications, containing the base settings for all environments.

**Technical Definition**
The JSON configuration provider loads `appsettings.json` and, optionally, `appsettings.{Environment}.json`. The environment-specific file **overrides** matching keys in the base file. `reloadOnChange` is enabled by default for these files. Comments are supported using JSONC syntax (JavaScript/C# style comments).

**Beginner-Friendly Explanation**
`appsettings.json` is **the default settings file**. You put common values there, and override them per environment in `appsettings.Development.json` or `appsettings.Production.json`.

---

### Purposes

- To provide **default settings** for all environments.
- To define **hierarchical sections** (e.g., `Logging`, `ConnectionStrings`).
- To support **per-environment overrides** without code changes.
- To keep **non-secret defaults** in source control.

---

### Syntax Rules and Structure

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=LocalSqlServer\\MSSQLDev;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

**Component breakdown**

- Nested objects — Become hierarchical sections.
- Keys — Case-insensitive.
- Values — Strings, numbers, booleans, arrays.

#### Syntax Rules

- `appsettings.json` is loaded first; `appsettings.{Environment}.json` overrides it.
- `reloadOnChange` is `true` by default.
- Comments are allowed (JSONC) but may produce IDE warnings.

#### Constraints and Limitations

- **Never store secrets** in `appsettings.json`.
- Arrays are replaced entirely by later providers; partial overrides are not supported.

---

### Annotated Code Examples and Expected Outputs

#### Example: Environment Override

`appsettings.json`:
```json
{ "Position": { "Title": "Editor" } }
```
`appsettings.Development.json`:
```json
{ "Position": { "Title": "Editor (Dev)" } }
```

```csharp
var title = builder.Configuration["Position:Title"];
```

**Expected Output (Development)**
```
Editor (Dev)
```

**Why this output occurs**
The Development-specific file is loaded after the base file and overrides the `Title` key.

---

### Real-World Cases

- **Logging configuration** — `Logging:LogLevel`.
- **Allowed hosts** — `AllowedHosts`.
- **Feature defaults** — `Features:*`.
- **Connection string placeholders** — Non-sensitive defaults.

---

### References

- App settings file configuration — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/#app-settings-file-configuration

---

## 4. Environment Variables

### Definitions

**Core Definition**
Environment variables are operating-system-level key-value pairs that configuration providers read and map into the configuration system.

**Technical Definition**
The `EnvironmentVariablesConfigurationProvider` reads variables **without** the `ASPNETCORE_` or `DOTNET_` prefixes by default. Hierarchical keys use `__` (double underscore) on all platforms; the provider converts `__` to `:`. Variables are loaded **after** JSON files and User Secrets, so they **override** those sources. Custom prefixes can be added via `AddEnvironmentVariables(prefix: "...")`.

**Beginner-Friendly Explanation**
Environment variables are **settings injected by the operating system or container**. They are ideal for production because they override JSON files and keep secrets out of source control.

---

### Purposes

- To **override configuration** without modifying files.
- To inject **secrets in production** (e.g., CI/CD, Kubernetes).
- To configure **containers** via environment variables.
- To provide **per-machine** or **per-deployment** values.

---

### Syntax Rules and Structure

```bash
# Linux/macOS
export ConnectionStrings__DefaultConnection="Server=prod;..."

# Windows PowerShell
$env:ConnectionStrings__DefaultConnection = "Server=prod;..."
```

```csharp
// Explicitly add a prefixed provider
builder.Configuration.AddEnvironmentVariables(prefix: "MyApp_");
```

**Component breakdown**

- `__` — Hierarchical separator (cross-platform).
- `prefix:` — Optional prefix stripped when reading keys.

#### Syntax Rules

- `__` is converted to `:` by the provider.
- Variables are loaded after JSON and User Secrets, so they override them.
- `launchSettings.json` variables override system environment variables in development.

#### Constraints and Limitations

- Environment variables are stored in **plain text** and may be visible to other processes.
- Bash does not support `:` in variable names; use `__`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overriding a Connection String

```bash
export ConnectionStrings__DefaultConnection="Server=prod;Database=App;"
```

```csharp
var conn = builder.Configuration.GetConnectionString("DefaultConnection");
```

**Expected Output (Production)**
```
Server=prod;Database=App;
```

**Why this output occurs**
The environment variable uses `__` for hierarchy; the provider maps it to `ConnectionStrings:DefaultConnection` and overrides the JSON value.

---

### Real-World Cases

- **Kubernetes** — Secrets and ConfigMaps injected as environment variables.
- **Docker** — `docker run -e "Key=Value"`.
- **CI/CD** — Pipeline variables for deployment-specific settings.

---

### References

- Environment Variables Configuration Provider — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/#environment-variables-configuration-provider
- Safe storage of app secrets in development — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets

---

## 5. User Secrets

### Definitions

**Core Definition**
User Secrets is a development-time secret store that keeps sensitive values **outside the project tree** so they are not committed to source control.

**Technical Definition**
The Secret Manager tool stores secrets in a JSON file under the user profile (`%APPDATA%\Microsoft\UserSecrets\<UserSecretsId>\secrets.json` on Windows, `~/.microsoft/usersecrets/<UserSecretsId>/secrets.json` on Linux/macOS). The project file must contain a `<UserSecretsId>` GUID. Secrets are loaded **only in the Development environment** and are overridden by environment variables in production.

**Beginner-Friendly Explanation**
User Secrets is **a private settings file on your machine** that never goes into Git. You use it for API keys and connection strings during development.

---

### Purposes

- To **keep secrets out of source control** during development.
- To **avoid polluting `appsettings.json`** with sensitive values.
- To provide **per-developer** overrides without affecting others.
- To enable **safe local testing** with real credentials.

---

### Syntax Rules and Structure

```xml
<PropertyGroup>
  <UserSecretsId>a1b2c3d4-e5f6-7890-abcd-ef1234567890</UserSecretsId>
</PropertyGroup>
```

```bash
dotnet user-secrets init
dotnet user-secrets set "Movies:ServiceApiKey" "12345"
```

```csharp
// Automatically added in Development by the default host
// Or explicitly:
builder.Configuration.AddUserSecrets<Program>();
```

**Component breakdown**

- `<UserSecretsId>` — Unique GUID linking the project to its secret store.
- `dotnet user-secrets set` — Writes a secret.
- `AddUserSecrets<T>()` — Loads the secrets.

#### Syntax Rules

- User Secrets are loaded **after** JSON files and **before** environment variables.
- They are enabled **only in Development** by default.
- Hierarchical keys use `:` in the secret name.

#### Constraints and Limitations

- **Not for production**; production secrets must use Azure Key Vault or environment variables.
- Secrets are **not encrypted** on disk (they rely on OS file permissions).
- The `UserSecretsId` must be unique per project.

---

### Annotated Code Examples and Expected Outputs

#### Example: Storing and Reading a Secret

```bash
dotnet user-secrets set "AppSecret" "super-secret-value"
```

```csharp
var secret = builder.Configuration["AppSecret"];
```

**Expected Output (Development)**
```
super-secret-value
```

**Why this output occurs**
The Secret Manager provider loads the secret from the user profile; it is overridden by environment variables in production.

---

### Real-World Cases

- **Local development** — API keys, OAuth client secrets.
- **Developer-specific overrides** — Personal database connections.
- **Testing third-party integrations** — Sandbox credentials.

---

### References

- Safe storage of app secrets in development — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Secret Manager — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets#secret-manager

---

## 6. Azure Key Vault

### Definitions

**Core Definition**
Azure Key Vault is a cloud service for securely storing and accessing secrets, keys, and certificates, integrated into .NET configuration via a provider.

**Technical Definition**
The `Azure.Extensions.AspNetCore.Configuration.Secrets` package provides a configuration provider that loads secrets from Key Vault. Authentication uses `Azure.Identity` (Managed Identity, Service Principal, or certificate). Key Vault secrets are loaded as configuration keys; hierarchical secrets use `--` as a separator (converted to `:`). The provider is added **after** other sources so Key Vault values override JSON and environment variables when intended.

**Beginner-Friendly Explanation**
Azure Key Vault is **a secure cloud safe for production secrets**. Your app authenticates with Azure and reads secrets at startup, so no secret ever lives in code or files.

---

### Purposes

- To **secure production secrets** with hardware-backed protection.
- To **centralize secret management** across services.
- To **audit access** to secrets.
- To **rotate secrets** without redeploying.
- To meet **compliance requirements** (FIPS 140-2).

---

### Syntax Rules and Structure

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());
```

**Component breakdown**

- `AddAzureKeyVault` — Extension method from `Azure.Extensions.AspNetCore.Configuration.Secrets`.
- `DefaultAzureCredential` — Authentication chain (Managed Identity, environment, etc.).

#### Syntax Rules

- Requires `Azure.Extensions.AspNetCore.Configuration.Secrets` and `Azure.Identity` packages.
- Secrets are loaded as configuration keys; `--` in secret names maps to `:`.
- The provider is added after other providers to override their values.

#### Constraints and Limitations

- Requires network access and Azure authentication.
- Secret retrieval adds startup latency.
- Secrets are cached in memory after loading.

---

### Annotated Code Examples and Expected Outputs

#### Example: Loading Key Vault Secrets

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri("https://my-vault.vault.azure.net/"),
    new DefaultAzureCredential());

// Secret named "ConnectionStrings--Default" maps to ConnectionStrings:Default
var conn = builder.Configuration.GetConnectionString("Default");
```

**Expected Output (Production)**
```
Server=prod;Database=App;
```

**Why this output occurs**
Key Vault secrets are loaded into configuration; `--` is converted to `:`, and `GetConnectionString` reads the mapped key.

---

### Real-World Cases

- **Azure App Service** — Managed Identity accessing Key Vault.
- **AKS** — Pod identity or workload identity.
- **Multi-tenant SaaS** — Per-tenant secrets in separate vaults.

---

### References

- Use Azure Key Vault configuration provider in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration

---

## 7. Configuration Binding

### Definitions

**Core Definition**
Configuration binding is the process of mapping configuration sections to strongly typed POCOs.

**Technical Definition**
`ConfigurationBinder.Get<T>()` and `ConfigurationBinder.Bind()` map configuration keys to object properties by name (case-insensitive). `GetSection("Section").Get<T>()` returns a new instance; `Bind(instance)` populates an existing instance. The Options pattern extends binding with DI and lifetimes.

**Beginner-Friendly Explanation**
Binding is **turning raw configuration into a C# object**. Instead of `config["Position:Title"]`, you bind the `Position` section to a `PositionOptions` object with a `Title` property.

---

### Purposes

- To **eliminate string-based access** in application code.
- To **validate** configuration at startup.
- To **group related settings** into cohesive classes.
- To enable **IntelliSense** and compile-time safety.
- To support **testing** with POCO instances.

---

### Syntax Rules and Structure

```csharp
public class PositionOptions
{
    public string Title { get; set; } = "";
    public string Name { get; set; } = "";
}

var options = builder.Configuration.GetSection("Position").Get<PositionOptions>();
```

**Component breakdown**

- `GetSection("Position")` — Selects the section.
- `Get<T>()` — Binds and returns a new instance.
- `Bind(existing)` — Binds into an existing object.

#### Syntax Rules

- Property names must match configuration keys (case-insensitive).
- Nested sections bind to nested objects.
- Arrays and collections bind to `T[]`, `List<T>`, etc.

#### Constraints and Limitations

- `Get<T>()` allocates a new object each call; use the Options pattern for repeated access.
- Binding failures may throw or leave properties at their default values (depending on strictness).

---

### Annotated Code Examples and Expected Outputs

#### Example: Binding a Section

```csharp
public class SmtpOptions
{
    public string Host { get; set; } = "";
    public int Port { get; set; }
}

var smtp = builder.Configuration.GetSection("Smtp").Get<SmtpOptions>();
// smtp.Host, smtp.Port
```

**Expected Output (conceptual)**
```
Host = "smtp.example.com", Port = 587
```

**Why this output occurs**
The binder maps `Smtp:Host` and `Smtp:Port` to the corresponding properties.

---

### Real-World Cases

- **Startup binding** — Reading configuration once at startup.
- **Tooling** — CLI tools binding configuration to options.
- **Testing** — Binding `IConfiguration` to test POCOs.

---

### References

- Configuration binding — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration#binding
- `ConfigurationBinder.Get<T>` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationbinder.get

---

## 8. Options Pattern

### Definitions

**Core Definition**
The Options pattern is the recommended way to consume configuration as **strongly typed objects** through dependency injection, with lifetimes and validation.

**Technical Definition**
The pattern registers a POCO (`TOptions`) with the DI container via `services.Configure<TOptions>(configuration.GetSection("Section"))`. Consumers request `IOptions<TOptions>`, `IOptionsSnapshot<TOptions>`, or `IOptionsMonitor<TOptions>`. The pattern supports **named options**, **post-configuration**, **validation** (DataAnnotations, `IValidateOptions<T>`), and **`ValidateOnStart`**.

**Beginner-Friendly Explanation**
The Options pattern is **the clean way to use configuration**. Instead of reading strings, your service asks for `IOptions<MySettings>`, and the container provides a validated, typed object.

---

### Purposes

- To **decouple** services from `IConfiguration`.
- To provide **strongly typed**, validated settings.
- To support **different lifetimes** (singleton, scoped, change-tracking).
- To enable **named options** for multiple configurations.
- To **validate at startup** with `ValidateOnStart`.

---

### Syntax Rules and Structure

```csharp
public class MySettings
{
    public string ApiKey { get; set; } = "";
}

builder.Services.Configure<MySettings>(
    builder.Configuration.GetSection("MySettings"));

// Consumption
public class MyService(IOptions<MySettings> options)
{
    private readonly MySettings _settings = options.Value;
}
```

**Component breakdown**

- `Configure<T>(section)` — Registers the binding.
- `IOptions<T>` — Singleton access to the value.
- `options.Value` — The bound POCO.

#### Syntax Rules

- Register with `Configure<T>(IConfigurationSection)`.
- Inject one of the three `IOptions` interfaces.
- Use `ValidateDataAnnotations()` and `ValidateOnStart()` for validation.

#### Constraints and Limitations

- `IOptions<T>` does not reflect configuration changes after startup.
- Validation runs lazily unless `ValidateOnStart` is called.

---

### Annotated Code Examples and Expected Outputs

#### Example: Validated Options

```csharp
public class KeyOptions
{
    [Required] public string Key1 { get; set; } = "";
    [Range(1, 100)] public int Key2 { get; set; }
}

builder.Services.AddOptions<KeyOptions>()
    .Bind(builder.Configuration.GetSection("Key"))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

**Expected Output (conceptual)**
```
// Startup fails if Key1 is missing or Key2 is out of range.
```

**Why this pattern is used**
`ValidateOnStart` catches invalid configuration before the app starts serving requests.

---

### Real-World Cases

- **API clients** — Base URL, API key, timeout.
- **Feature flags** — Typed flags with defaults.
- **Third-party integrations** — SMTP, payment gateways.

---

### References

- Options pattern in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options

---

## 9. `IOptions<T>`

### Definitions

**Core Definition**
`IOptions<T>` provides **singleton access** to a configured options object computed once at startup.

**Technical Definition**
`IOptions<T>.Value` returns the cached `TOptions` instance. It is registered as a singleton and **does not reflect configuration changes** after startup. It is suitable for settings that are fixed for the lifetime of the application.

**Beginner-Friendly Explanation**
`IOptions<T>` is **“set it once and forget it”**—the value is computed at startup and never changes.

---

### Purposes

- To access **stable settings** that should not change at runtime.
- To provide the **simplest** Options interface.
- To avoid per-request overhead.
- To ensure consistent values throughout the application.

---

### Syntax Rules and Structure

```csharp
public class MyService(IOptions<MySettings> options)
{
    private readonly MySettings _settings = options.Value;
}
```

**Component breakdown**

- `IOptions<T>` — Singleton service.
- `Value` — Cached instance.

#### Syntax Rules

- `Value` is computed on first access.
- Changes to the underlying configuration are **not observed**.
- Validation runs on first access to `Value` (or at startup with `ValidateOnStart`).

#### Constraints and Limitations

- Not suitable for **hot-reload** scenarios.
- Not suitable for **per-request** values (use `IOptionsSnapshot<T>`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Singleton Options

```csharp
public class EmailService(IOptions<SmtpOptions> options)
{
    private readonly SmtpOptions _smtp = options.Value;

    public void Send() => Console.WriteLine(_smtp.Host);
}
```

**Expected Output (conceptual)**
```
smtp.example.com
```

**Why this output occurs**
`Value` returns the singleton instance bound at startup.

---

### Real-World Cases

- **Application-wide settings** — App name, version.
- **Immutable configuration** — Settings that should not change.
- **Simple services** — Avoiding reload complexity.

---

### References

- `IOptions<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptions

---

## 10. `IOptionsSnapshot<T>`

### Definitions

**Core Definition**
`IOptionsSnapshot<T>` provides **scoped access** to options, recomputed **once per scope** (typically per HTTP request).

**Technical Definition**
`IOptionsSnapshot<T>` is registered as **scoped**. `Value` is computed when the snapshot is constructed (or on first access within the scope) and cached for the lifetime of the scope. Changes to configuration are reflected in **new scopes**, not within the current one. In non-HTTP scopes (desktop apps, Windows services), it behaves like `IOptions<T>` unless a scope is explicitly created.

**Beginner-Friendly Explanation**
`IOptionsSnapshot<T>` is **“fresh settings per request”**—each HTTP request gets the current configuration values.

---

### Purposes

- To provide **per-request configuration** in web apps.
- To reflect **configuration changes** between requests.
- To support **multi-tenant** scenarios with per-request values.
- To avoid the overhead of change tracking for stable settings.

---

### Syntax Rules and Structure

```csharp
public class MyController(IOptionsSnapshot<MySettings> options)
{
    private readonly MySettings _settings = options.Value;
}
```

**Component breakdown**

- `IOptionsSnapshot<T>` — Scoped service.
- `Value` — Recomputed per scope.

#### Syntax Rules

- Registered as scoped; resolved per request.
- `Value` is cached within the scope.
- Changes are visible in new scopes.

#### Constraints and Limitations

- Cannot be injected into **singletons** (captive dependency).
- Overhead of recomputing per scope.

---

### Annotated Code Examples and Expected Outputs

#### Example: Per-Request Options

```csharp
app.MapGet("/settings", (IOptionsSnapshot<MySettings> options) =>
    options.Value.ApiKey);
```

**Expected Output (conceptual)**
```
// Returns the current ApiKey for each request.
```

**Why this output occurs**
Each request scope constructs a new snapshot, reflecting the latest configuration.

---

### Real-World Cases

- **Web APIs** — Per-request tenant settings.
- **Feature toggles** — Per-request flag evaluation.
- **Localization** — Per-request culture settings.

---

### References

- `IOptionsSnapshot<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptionssnapshot

---

## 11. `IOptionsMonitor<T>`

### Definitions

**Core Definition**
`IOptionsMonitor<T>` provides **singleton access** to the current options value and **notifies subscribers** when the value changes.

**Technical Definition**
`IOptionsMonitor<T>` is a singleton service. `CurrentValue` returns the latest options; `OnChange(Action<TOptions, string?>)` registers a callback invoked when configuration reloads. It is suitable for **long-lived services** (background workers, singletons) that need to react to configuration changes without restart.

**Beginner-Friendly Explanation**
`IOptionsMonitor<T>` is **“always the latest settings”**—you get the current value and can subscribe to changes.

---

### Purposes

- To access **live configuration** in singletons and background services.
- To **react to changes** without restarting the app.
- To support **hot-reload** scenarios.
- To provide the **latest value** at any time.

---

### Syntax Rules and Structure

```csharp
public class BackgroundWorker(IOptionsMonitor<MySettings> monitor)
{
    public void Start()
    {
        var current = monitor.CurrentValue;
        monitor.OnChange(updated => { /* react */ });
    }
}
```

**Component breakdown**

- `CurrentValue` — Latest options.
- `OnChange(...)` — Change notification callback.
- `Get(string name)` — Named options access.

#### Syntax Rules

- Registered as singleton.
- `CurrentValue` reflects the latest configuration.
- `OnChange` callbacks are invoked on configuration reload.

#### Constraints and Limitations

- Overhead of change tracking even when not needed.
- Callbacks run on the configuration reload thread; keep them fast and thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example: Monitoring Changes

```csharp
var monitor = app.Services.GetRequiredService<IOptionsMonitor<MySettings>>();
monitor.OnChange(settings => Console.WriteLine($"Changed: {settings.ApiKey}"));
```

**Expected Output (when config reloads)**
```
Changed: new-key-value
```

**Why this output occurs**
The monitor detects configuration reload and invokes the callback with the new value.

---

### Real-World Cases

- **Background workers** — Reacting to connection string changes.
- **Caches** — Resizing caches when limits change.
- **Feature flags** — Live toggle evaluation.

---

### References

- `IOptionsMonitor<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptionsmonitor

---

## 12. Configuration Best Practices

### Definitions

**Core Definition**
Configuration best practices are the guidelines for **layering, securing, binding, and validating** configuration in .NET applications.

**Technical Definition**
Best practices derive from Microsoft's configuration and Options guidance: **never read `IConfiguration` directly in services**—use the Options pattern; **never commit secrets**—use User Secrets in development and Azure Key Vault or environment variables in production; **layer configuration** from lowest to highest priority (JSON → environment-specific JSON → User Secrets → environment variables → Key Vault); **use `__` for hierarchical environment variables** (cross-platform); **validate options at startup** with `ValidateOnStart`; and **choose the right Options interface** for the lifetime: `IOptions<T>` for stable, `IOptionsSnapshot<T>` for per-request, `IOptionsMonitor<T>` for live updates.

**Beginner-Friendly Explanation**
The best practices are: **don't hard-code secrets**, **use the Options pattern**, **layer your sources**, **validate at startup**, and **pick the right lifetime**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Options pattern, not IConfiguration
builder.Services.AddOptions<MySettings>()
    .Bind(builder.Configuration.GetSection("MySettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();

// 2. Secrets separation
// Development: dotnet user-secrets
// Production: Azure Key Vault or environment variables

// 3. Layered sources (highest wins)
// appsettings.json → appsettings.{Env}.json → User Secrets → Env Vars → Key Vault

// 4. Cross-platform environment variables
// ConnectionStrings__DefaultConnection

// 5. Right Options interface
// IOptions<T>          — stable
// IOptionsSnapshot<T>  — per-request
// IOptionsMonitor<T>  — live updates
```

**Component breakdown**

- **Options pattern** — Strongly typed, validated settings.
- **Secrets separation** — User Secrets / Key Vault.
- **Layering** — Overrides in priority order.
- **`__` separator** — Cross-platform hierarchy.
- **Interface choice** — Lifetime and reload semantics.

#### Rules and Constraints

- **Do** use the Options pattern in services.
- **Do** keep secrets out of `appsettings.json`.
- **Do** layer configuration sources correctly.
- **Do** use `__` for environment variable hierarchy.
- **Do** validate options at startup.
- **Do not** read `IConfiguration` directly in application logic.
- **Do not** commit secrets to source control.

---

### Annotated Code Examples and Expected Outputs

#### Example: Production-Ready Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Layered sources (default host already adds JSON + env vars)
builder.Configuration.AddUserSecrets<Program>();   // Development only
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());

// Options with validation
builder.Services.AddOptions<SmtpOptions>()
    .Bind(builder.Configuration.GetSection("Smtp"))
    .ValidateDataAnnotations()
    .ValidateOnStart();

var app = builder.Build();
```

**Expected Output (conceptual)**
```
// Configuration is layered, secrets are separated, and options are validated at startup.
```

**Why this pattern is used**
It follows all best practices: layered sources, secrets separation, strong typing, and startup validation.

---

### Real-World Cases

- **Production APIs** — Key Vault + environment variables + validated options.
- **Microservices** — Environment-specific JSON + User Secrets in development.
- **Multi-tenant SaaS** — Per-tenant secrets in Key Vault, monitored options.

---

### References

- Options pattern in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
- Configuration in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/
- Safe storage of app secrets in development — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Use Azure Key Vault configuration provider — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Direct `IConfiguration` injection into services | **Discouraged**; use Options pattern. |
| Secrets in `appsettings.json` | **Unsafe**; use User Secrets/Key Vault. |
| `:` in environment variable names | **Not cross-platform**; use `__`. |
| `IOptions<T>` reload | **Does not reflect changes** after startup. |
| `IOptionsSnapshot<T>` in singletons | **Captive dependency**; use `IOptionsMonitor<T>`. |
| `ValidateOnStart` | **Recommended** for fail-fast validation. |
| Azure Key Vault | **Production-only**; requires authentication and network access. |

---

## References (Consolidated)

- Configuration in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/
- Options pattern in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
- `IConfiguration` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration
- Configuration binding — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration#binding
- `ConfigurationBinder.Get<T>` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationbinder.get
- Safe storage of app secrets in development — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Secret Manager — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets#secret-manager
- Use Azure Key Vault configuration provider in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
- Environment Variables Configuration Provider — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/#environment-variables-configuration-provider
- `IOptions<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptions
- `IOptionsSnapshot<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptionssnapshot
- `IOptionsMonitor<T>` — Options pattern — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#ioptionsmonitor

---