# ASP.NET Core Identity: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core Identity** (.NET 6–10), the membership system that handles authentication, authorization, and identity management for first-party user accounts. It covers the setup of Identity services and stores, user and role management through `UserManager` and `RoleManager`, the claims model and customization via `ClaimsPrincipalFactory`, and security-focused best practices. Identity uses **cookie authentication** by default for browser-based apps and can also issue **bearer tokens** for SPA/API scenarios via `AddIdentityApiEndpoints`. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ASP.NET Core Identity is a membership system that provides APIs for authentication, authorization, and identity management—including user accounts, login flows, password management, email confirmation, multi-factor authentication (MFA), and account lockout. |
| **Technical Definition** | Identity is registered via `AddIdentity<TUser, TRole>` (or `AddDefaultIdentity`, or `AddIdentityApiEndpoints<TUser>` for API/SPA scenarios). It uses EF Core stores (`AddEntityFrameworkStores<TContext>`) where `TContext` inherits from `IdentityDbContext<TUser>`. Core managers are `UserManager<TUser>` (user CRUD, password hashing, claims, roles, security stamps) and `RoleManager<TRole>` (role CRUD, role claims). The `UserClaimsPrincipalFactory<TUser>` builds the `ClaimsPrincipal` attached to `HttpContext.User` from the user’s identity and persisted claims. |
| **Beginner-Friendly Explanation** | ASP.NET Core Identity is the **built-in “user account system”** for ASP.NET Core apps. Instead of building your own login, registration, password reset, and role management from scratch, Identity gives you all of that out of the box. It stores users in a database, hashes passwords securely, manages roles, and integrates with middleware so that `HttpContext.User` is populated with the logged-in user’s identity. |

### Key Characteristics

- **First-party membership** – Designed for apps that manage their own user accounts (as opposed to delegating entirely to an external identity provider).
- **Password hashing built-in** – Uses PBKDF2 with HMAC-SHA256, a 128-bit salt, a 256-bit subkey, and 10,000 iterations by default.
- **Claims + roles** – Users can have claims (key-value assertions) and roles; roles can also have claims.
- **Account protection** – Supports account lockout, email confirmation, two-factor authentication (2FA), and security stamps for invalidation.
- **Pluggable stores** – EF Core is the default store, but custom stores can be implemented.
- **API/SPA support** – `AddIdentityApiEndpoints` enables cookie **and** proprietary bearer token authentication for SPA backends.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- `Microsoft.AspNetCore.Identity.EntityFrameworkCore` NuGet package (for EF Core stores).
- An EF Core database provider (SQL Server, SQLite, PostgreSQL, in-memory for testing).
- Working knowledge of **middleware**, **dependency injection**, and **claims-based identity**.

### Related Programming Areas

- **Authentication fundamentals** – Identity plugs into the authentication middleware as a scheme.
- **Cookie authentication** – Identity’s default browser authentication mechanism.
- **Authorization** – `[Authorize]` attributes and policies evaluate the `ClaimsPrincipal` built by Identity.
- **Entity Framework Core** – `IdentityDbContext` extends `DbContext` with Identity tables (`AspNetUsers`, `AspNetRoles`, etc.).
- **Data Protection** – Protects cookies and tokens issued by Identity.

### Core Concepts / Features

1. **ASP.NET Core Identity** – The membership system, its architecture, and setup.
2. **Identity setup** – Service registration, `IdentityDbContext`, and middleware integration.
3. **User management** – `UserManager` operations (create, find, update, delete).
4. **Role management** – `RoleManager` and `UserManager.AddToRoleAsync`.
5. **Claims** – User claims, role claims, and `ClaimsPrincipalFactory` customization.
6. **Identity best practices** – Security, configuration, and maintainability guidelines.

---

## 1. ASP.NET Core Identity (The Membership System)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ASP.NET Core Identity is a membership system that manages user accounts, passwords, roles, and claims for ASP.NET Core applications. |
| **Technical Definition** | Identity is a set of services and stores registered in the DI container. `UserManager<TUser>` orchestrates user operations (create, find, update, delete, password management, claims, roles) and delegates persistence to an `IUserStore<TUser>`. `RoleManager<TRole>` manages roles and role claims. The `SignInManager<TUser>` handles sign-in, sign-out, and 2FA. `IdentityDbContext<TUser>` maps Identity entities to EF Core tables. Identity integrates with authentication middleware to populate `HttpContext.User`. |
| **Beginner-Friendly Explanation** | Identity is the **complete user-account toolkit**. It knows how to store users, hash their passwords, check their credentials on login, assign them roles, and attach claims to them. You do not write password hashing or session management yourself—Identity does it correctly and securely. |

### Purposes

- **To** provide a **battle-tested, secure membership system** so developers do not roll their own authentication and password storage.
- **To** manage the full lifecycle of user accounts: registration, login, password reset, email confirmation, 2FA, and account lockout.
- **To** support **role-based and claims-based authorization** through a unified identity model.
- **To** integrate seamlessly with ASP.NET Core’s **authentication middleware**, **DI**, and **EF Core**.
- **To** offer both **cookie-based authentication** (for web apps) and **bearer-token authentication** (for SPAs/APIs) through `AddIdentityApiEndpoints`.

### Syntax Rules and Structure

#### General Syntax (Registration)

```csharp
builder.Services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddIdentity<TUser, TRole>` | Registers Identity services with the specified user and role types. |
| `AddEntityFrameworkStores<TContext>` | Configures EF Core as the persistence store. |
| `AddDefaultTokenProviders` | Enables token generation for password reset, email confirmation, and 2FA. |

#### General Syntax (API/SPA Variant)

```csharp
builder.Services.AddIdentityApiEndpoints<IdentityUser>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddIdentityApiEndpoints<TUser>` | Registers Identity with API endpoints; enables **cookies** and **proprietary bearer tokens** by default. |
| `AddEntityFrameworkStores<TContext>` | Configures EF Core persistence. |

**Syntax rules:**

- **`AddIdentity` must be called before `AddAuthentication`** if you are configuring authentication schemes manually; `AddIdentity` itself configures the default cookie schemes.
- **`AddEntityFrameworkStores` requires the context to inherit from `IdentityDbContext<TUser>`** (or a compatible base).
- **`AddDefaultTokenProviders`** is required for password reset, email confirmation, and 2FA.

**Constraints and limitations:**

- **Identity is not a replacement for external identity providers**—it is for first-party accounts. For social login, combine Identity with external authentication handlers.
- **The default password policy is strict** (uppercase, lowercase, digit, non-alphanumeric, minimum 6 characters) and should be reviewed for your application.

### Annotated Code Example (Basic Setup)

```csharp
// Program.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// 1. Register the EF Core context (Identity-aware)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// 2. Register Identity services and stores
builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
{
    options.Password.RequiredLength = 8;
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

// 3. Register authorization
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();   // Identity populates HttpContext.User
app.UseAuthorization();
app.MapDefaultControllerRoute();

app.Run();

// ApplicationDbContext inherits IdentityDbContext
public class ApplicationDbContext : IdentityDbContext<IdentityUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }
}
```

**Step-by-step explanation:**

1. The `ApplicationDbContext` inherits `IdentityDbContext<IdentityUser>`, adding Identity tables (`AspNetUsers`, `AspNetRoles`, `AspNetUserClaims`, etc.) to the model.
2. `AddIdentity` registers `UserManager`, `RoleManager`, `SignInManager`, and the default cookie authentication schemes.
3. `AddEntityFrameworkStores` connects Identity to the EF Core context.
4. `AddDefaultTokenProviders` enables password reset and email confirmation tokens.
5. Middleware order: `UseAuthentication()` before `UseAuthorization()`.

**Expected Output:** The application has Identity tables in the database, and `[Authorize]`-protected endpoints challenge unauthenticated users with the default login redirect.

**Why this output?** Identity registers its services and stores; the authentication middleware uses Identity’s cookie schemes to populate `HttpContext.User`.

### Real-World Cases

- **Consumer web applications** needing registration, login, password reset, and 2FA.
- **SPA backends** using `AddIdentityApiEndpoints` for cookie or bearer-token authentication.
- **Internal business apps** requiring role-based access control with first-party accounts.

### References

- Introduction to Identity on ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- How to use Identity to secure a Web API backend for SPAs – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-api-authorization
- Configure ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration

---

## 2. Identity Setup (Service Registration and Configuration)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Identity setup is the process of registering Identity services, configuring stores, and integrating Identity with the middleware pipeline. |
| **Technical Definition** | Setup involves installing `Microsoft.AspNetCore.Identity.EntityFrameworkCore`, creating an `IdentityDbContext<TUser>` subclass, registering `AddIdentity` (or `AddIdentityApiEndpoints`) with `AddEntityFrameworkStores`, and configuring `IdentityOptions` (password, lockout, sign-in, user, claims identity). `IdentityOptions` must be configured **after** `AddIdentity`/`AddDefaultIdentity` is called. |
| **Beginner-Friendly Explanation** | Identity setup is **installing and wiring up the user-account system**. You add the NuGet package, create a database context that includes Identity tables, tell ASP.NET Core to use Identity, and optionally tweak the password and lockout rules. After that, Identity is ready to manage users. |

### Purposes

- **To** install and configure the packages and services required for Identity.
- **To** create an EF Core context that includes Identity’s tables.
- **To** customise **password policy**, **lockout**, **sign-in requirements**, and **claim types**.
- **To** integrate Identity with the middleware pipeline (`UseAuthentication`, `UseAuthorization`).
- **To** support both **cookie** and **bearer token** authentication for different client types.

### Syntax Rules and Structure

#### General Syntax (Options Configuration)

```csharp
builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
{
    // Password
    options.Password.RequiredLength = 8;
    options.Password.RequireDigit = true;
    options.Password.RequireUppercase = true;

    // Lockout
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);

    // Sign-in
    options.SignIn.RequireConfirmedEmail = true;

    // Claims identity
    options.ClaimsIdentity.UserIdClaimType = ClaimTypes.NameIdentifier;
    options.ClaimsIdentity.RoleClaimType = ClaimTypes.Role;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();
```

**Component breakdown:**

| Option Group | Key Settings |
|--------------|--------------|
| **Password** | `RequiredLength`, `RequireDigit`, `RequireUppercase`, `RequireLowercase`, `RequireNonAlphanumeric` |
| **Lockout** | `MaxFailedAccessAttempts`, `DefaultLockoutTimeSpan`, `AllowedForNewUsers` |
| **SignIn** | `RequireConfirmedEmail`, `RequireConfirmedPhoneNumber`, `RequireConfirmedAccount` |
| **ClaimsIdentity** | `UserIdClaimType`, `RoleClaimType`, `UserNameClaimType`, `SecurityStampClaimType` |

#### Syntax Rules

- **`IdentityOptions` must be configured after `AddIdentity`/`AddDefaultIdentity`**; otherwise, the options may be overwritten.
- **Default password policy**: uppercase, lowercase, digit, non-alphanumeric; minimum length 6.
- **`RequireConfirmedEmail = true`** requires users to confirm their email before signing in.

**Constraints and limitations:**

- **Overly strict password rules** can frustrate users; balance security and usability.
- **Lockout must be enabled per user** (`LockoutEnabled`) and triggered with `lockoutOnFailure: true` during sign-in.

### Annotated Code Example (Customised Identity Configuration)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Password.RequiredLength = 10;
    options.Password.RequireNonAlphanumeric = true;
    options.Lockout.MaxFailedAccessAttempts = 3;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(30);
    options.SignIn.RequireConfirmedEmail = true;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddAuthorization();
var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output:** Users must have passwords of at least 10 characters with non-alphanumeric characters; three failed attempts lock the account for 30 minutes; email confirmation is required before sign-in.

**Why this output?** The `IdentityOptions` delegate configures the password validator, lockout behaviour, and sign-in requirements used by `UserManager` and `SignInManager`.

### Real-World Cases

- **Consumer apps** requiring strong passwords and email confirmation.
- **Enterprise apps** enforcing lockout after repeated failures.
- **SPA backends** using `AddIdentityApiEndpoints` with custom options.

### References

- Configure ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration
- IdentityOptions class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityoptions

---

## 3. User Management

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | User management is the set of operations for creating, finding, updating, and deleting user accounts, as well as managing passwords, claims, roles, and security stamps. |
| **Technical Definition** | `UserManager<TUser>` provides the primary APIs: `CreateAsync(user, password)`, `FindByIdAsync`, `FindByNameAsync`, `FindByEmailAsync`, `UpdateAsync`, `DeleteAsync`, `AddClaimAsync`, `AddToRoleAsync`, `CheckPasswordAsync`, `ResetPasswordAsync`, `GenerateEmailConfirmationTokenAsync`, and more. All operations are asynchronous and return `IdentityResult` (success or errors). |
| **Beginner-Friendly Explanation** | User management is **everything you do with user accounts**. You create a user with `CreateAsync` (which hashes the password), find them by ID, name, or email, update their profile, delete them, assign them roles, or reset their password. `UserManager` handles the hard parts—password hashing, security stamps, and persistence—for you. |

### Purposes

- **To** create, read, update, and delete user accounts (`CreateAsync`, `FindByIdAsync`, `UpdateAsync`, `DeleteAsync`).
- **To** manage passwords securely (hashing, verification, reset) without custom code.
- **To** assign and remove **claims** and **roles** for authorization.
- **To** manage **security stamps** for invalidating sessions when credentials change.
- **To** enforce **email confirmation** and **lockout** policies.

### Syntax Rules and Structure

#### General Syntax (Core Operations)

```csharp
// Create
var user = new IdentityUser { UserName = "alice@example.com", Email = "alice@example.com" };
IdentityResult result = await _userManager.CreateAsync(user, "P@ssw0rd!");

// Find
var user = await _userManager.FindByIdAsync(userId);
var user = await _userManager.FindByNameAsync(userName);
var user = await _userManager.FindByEmailAsync(email);

// Update
user.Email = "new@example.com";
IdentityResult result = await _userManager.UpdateAsync(user);

// Delete
IdentityResult result = await _userManager.DeleteAsync(user);

// Password check
bool valid = await _userManager.CheckPasswordAsync(user, password);
```

**Component breakdown:**

| Operation | Method | Returns |
|-----------|--------|---------|
| Create | `CreateAsync(user, password)` | `IdentityResult` |
| Find by ID | `FindByIdAsync(id)` | `TUser?` |
| Find by name | `FindByNameAsync(name)` | `TUser?` |
| Update | `UpdateAsync(user)` | `IdentityResult` |
| Delete | `DeleteAsync(user)` | `IdentityResult` |
| Check password | `CheckPasswordAsync(user, password)` | `bool` |

**Syntax rules:**

- **All operations are asynchronous**—use `await`.
- **Always check `IdentityResult.Succeeded`**; on failure, iterate `result.Errors` for details.
- **`CreateAsync` hashes the password**—never store or handle plain-text passwords yourself.
- **`DeleteAsync` removes the user**; related claims and roles are removed by the store.

**Constraints and limitations:**

- **`UserManager` is scoped**—do not capture it in a Singleton; resolve it per request or create a scope for background work.
- **Deleting a user is permanent**—consider soft-delete patterns if auditability is required.
- **Security stamps** are updated on password/email changes; use `UpdateSecurityStampAsync` to invalidate all existing sessions.

### Annotated Code Example (User CRUD and Password Management)

```csharp
public class UserService
{
    private readonly UserManager<IdentityUser> _userManager;

    public UserService(UserManager<IdentityUser> userManager)
        => _userManager = userManager;

    public async Task<IdentityResult> RegisterAsync(string email, string password)
    {
        var user = new IdentityUser { UserName = email, Email = email };
        return await _userManager.CreateAsync(user, password);   // Password is hashed
    }

    public async Task<IdentityResult> ChangeEmailAsync(string userId, string newEmail)
    {
        var user = await _userManager.FindByIdAsync(userId);
        if (user is null) return IdentityResult.Failed(
            new IdentityError { Description = "User not found." });

        user.Email = newEmail;
        user.UserName = newEmail;
        return await _userManager.UpdateAsync(user);
    }

    public async Task<IdentityResult> DeleteUserAsync(string userId)
    {
        var user = await _userManager.FindByIdAsync(userId);
        if (user is null) return IdentityResult.Failed(
            new IdentityError { Description = "User not found." });

        return await _userManager.DeleteAsync(user);
    }
}
```

**Step-by-step explanation:**

1. `RegisterAsync` creates an `IdentityUser` and calls `CreateAsync`, which hashes the password and persists the user.
2. `ChangeEmailAsync` finds the user, updates the email and username, and calls `UpdateAsync`.
3. `DeleteUserAsync` finds the user and calls `DeleteAsync`.

**Expected Output:** `IdentityResult.Succeeded` is `true` for valid operations; errors are returned for invalid input (e.g., duplicate email, weak password).

**Why this output?** `UserManager` orchestrates validation, password hashing, and persistence, returning a structured result for the caller to inspect.

### Real-World Cases

- **Registration flows:** `CreateAsync` with password validation and email confirmation token generation.
- **Admin panels:** Find, update, and delete users; assign roles.
- **Profile management:** Update email, phone number, and claims.

### References

- UserManager<TUser> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1
- IdentityResult class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityresult

---

## 4. Role Management

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Role management is the set of operations for creating, deleting, and assigning roles to users, enabling role-based authorization. |
| **Technical Definition** | `RoleManager<TRole>` provides `CreateAsync`, `DeleteAsync`, `RoleExistsAsync`, and role-claim methods. `UserManager<TUser>` provides `AddToRoleAsync`, `RemoveFromRoleAsync`, `GetRolesAsync`, and `IsInRoleAsync`. Roles are persisted in the `AspNetRoles` and `AspNetUserRoles` tables. The `UserClaimsPrincipalFactory<TUser, TRole>` includes role membership claims in the `ClaimsPrincipal` when `AddRoles<TRole>` is registered. |
| **Beginner-Friendly Explanation** | Role management is **creating “job titles” and assigning them to users**. A role is a label like “Admin” or “Manager.” You create the role once, then add users to it. Authorization can then say “only Admins can access this page,” and Identity will check whether the user has the Admin role. |

### Purposes

- **To** create and delete roles (`RoleManager.CreateAsync`, `DeleteAsync`).
- **To** assign users to roles and remove them (`UserManager.AddToRoleAsync`, `RemoveFromRoleAsync`).
- **To** check role membership (`IsInRoleAsync`, `GetRolesAsync`).
- **To** enable `[Authorize(Roles = "Admin")]` and role-based policies.
- **To** support role claims (claims attached to a role rather than a user) when `RoleManager.SupportsRoleClaims` is true.

### Syntax Rules and Structure

#### General Syntax (Role Operations)

```csharp
// Create a role
var role = new IdentityRole("Admin");
IdentityResult result = await _roleManager.CreateAsync(role);

// Assign a user to a role
var user = await _userManager.FindByNameAsync(userName);
IdentityResult result = await _userManager.AddToRoleAsync(user, "Admin");

// Check membership
bool isAdmin = await _userManager.IsInRoleAsync(user, "Admin");
```

**Component breakdown:**

| Operation | Manager | Method |
|-----------|---------|--------|
| Create role | `RoleManager` | `CreateAsync(role)` |
| Delete role | `RoleManager` | `DeleteAsync(role)` |
| Add user to role | `UserManager` | `AddToRoleAsync(user, roleName)` |
| Remove user from role | `UserManager` | `RemoveFromRoleAsync(user, roleName)` |
| Check membership | `UserManager` | `IsInRoleAsync(user, roleName)` |

**Syntax rules:**

- **Roles must exist before users can be added**—call `RoleManager.CreateAsync` first (or check `RoleExistsAsync`).
- **`AddToRoleAsync` does not create the role**—it throws or returns an error if the role does not exist.
- **Role claims** require `RoleManager` and are included in the `ClaimsPrincipal` only when `AddRoles<TRole>` is registered.

**Constraints and limitations:**

- **Roles are coarse-grained**—for fine-grained permissions, use claims or policies.
- **Role names are unique**—creating a duplicate role fails.
- **Deleting a role** removes the role from all users; ensure no authorization logic depends on it.

### Annotated Code Example (Creating Roles and Assigning Users)

```csharp
public class RoleSeeder
{
    public static async Task SeedAsync(IServiceProvider services)
    {
        var roleManager = services.GetRequiredService<RoleManager<IdentityRole>>();
        var userManager = services.GetRequiredService<UserManager<IdentityUser>>();

        // 1. Ensure the role exists
        if (!await roleManager.RoleExistsAsync("Admin"))
        {
            await roleManager.CreateAsync(new IdentityRole("Admin"));
        }

        // 2. Find the user and add them to the role
        var user = await userManager.FindByNameAsync("alice@example.com");
        if (user is not null && !await userManager.IsInRoleAsync(user, "Admin"))
        {
            await userManager.AddToRoleAsync(user, "Admin");
        }
    }
}
```

**Expected Output:** The “Admin” role exists in the database, and Alice is a member of it. `[Authorize(Roles = "Admin")]` now grants Alice access.

**Why this output?** `RoleManager.CreateAsync` persists the role; `UserManager.AddToRoleAsync` creates the user-role association. The `ClaimsPrincipal` factory includes role claims when roles are registered.

### Real-World Cases

- **Admin vs. user separation:** Create “Admin” and “User” roles; protect admin controllers with `[Authorize(Roles = "Admin")]`.
- **Multi-tier memberships:** “Basic”, “Premium”, “Enterprise” roles controlling feature access.
- **Role seeding:** Ensure default roles exist at startup for consistent authorization.

### References

- RoleManager<TRole> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.rolemanager-1
- UserManager.AddToRoleAsync – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.addtoroleasync

---

## 5. Claims

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Claims are key-value assertions about a user (e.g., `Name: Alice`, `Role: Admin`, `Tenant: Acme`) that are included in the user’s identity and used for authorization decisions. |
| **Technical Definition** | In Identity, claims are persisted in the `AspNetUserClaims` table and managed via `UserManager.AddClaimAsync`, `RemoveClaimAsync`, and `GetClaimsAsync`. The `UserClaimsPrincipalFactory<TUser>` builds the `ClaimsPrincipal` from the user’s ID, username, security stamp, and persisted claims. When `AddRoles<TRole>` is registered, `UserClaimsPrincipalFactory<TUser, TRole>` additionally includes role membership claims and role-entity claims. Custom claims can be added at sign-in time by overriding `GenerateClaimsAsync` in a custom factory. |
| **Beginner-Friendly Explanation** | Claims are **facts about the user** that travel with their identity. “Alice’s email is alice@example.com” is a claim. “Alice has the Admin role” is also a claim. You can add your own claims—like “Alice belongs to tenant 42”—and then write authorization policies that check for them. Claims give you **finer control than roles alone**. |

### Purposes

- **To** carry **fine-grained attributes** about the user (tenant, department, subscription tier) for authorization.
- **To** support **policy-based authorization** that checks claim values.
- **To** persist custom user attributes without altering the Identity schema.
- **To** include **role claims** in the `ClaimsPrincipal` for role-based checks.
- **To** customise the `ClaimsPrincipal` at sign-in time via a custom `ClaimsPrincipalFactory`.

### Syntax Rules and Structure

#### General Syntax (Adding and Retrieving Claims)

```csharp
// Add a claim to a user
await _userManager.AddClaimAsync(user, new Claim("tenant", "acme"));

// Get claims
var claims = await _userManager.GetClaimsAsync(user);

// Remove a claim
await _userManager.RemoveClaimAsync(user, claim);
```

**Component breakdown:**

| Operation | Method |
|-----------|--------|
| Add claim | `AddClaimAsync(user, claim)` |
| Get claims | `GetClaimsAsync(user)` |
| Remove claim | `RemoveClaimAsync(user, claim)` |

#### General Syntax (Custom ClaimsPrincipalFactory)

```csharp
public class ApplicationClaimsPrincipalFactory
    : UserClaimsPrincipalFactory<ApplicationUser, IdentityRole>
{
    public ApplicationClaimsPrincipalFactory(
        UserManager<ApplicationUser> userManager,
        RoleManager<IdentityRole> roleManager,
        IOptions<IdentityOptions> options)
        : base(userManager, roleManager, options) { }

    protected override async Task<ClaimsIdentity> GenerateClaimsAsync(ApplicationUser user)
    {
        var identity = await base.GenerateClaimsAsync(user);
        identity.AddClaim(new Claim("tenant", user.TenantId));
        return identity;
    }
}

// Registration
builder.Services.AddScoped<IUserClaimsPrincipalFactory<ApplicationUser>,
    ApplicationClaimsPrincipalFactory>();
```

**Syntax rules:**

- **Persisted claims** are stored in the database and loaded automatically into the `ClaimsPrincipal`.
- **Custom factory claims** are generated at sign-in time and are not persisted—useful for computed attributes.
- **Role claims** are included only when `AddRoles<TRole>` is registered; the base factory does not inspect `SupportsUserRole`.

**Constraints and limitations:**

- **Claims are not encrypted**—they are encoded in the cookie or token. Do not store secrets in claims.
- **Large claim sets** can exceed cookie size limits; keep claims minimal.
- **Custom factory claims** are recalculated on each sign-in; persisted claims require explicit add/remove operations.

### Annotated Code Example (Adding a Tenant Claim and Authorizing with a Policy)

```csharp
// 1. Add a tenant claim to the user
var user = await _userManager.FindByNameAsync("alice@example.com");
await _userManager.AddClaimAsync(user, new Claim("tenant", "acme"));

// 2. Register a policy requiring the claim
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AcmeTenant", policy =>
        policy.RequireClaim("tenant", "acme"));
});

// 3. Protect an endpoint with the policy
[Authorize(Policy = "AcmeTenant")]
public IActionResult AcmeDashboard() => View();
```

**Step-by-step explanation:**

1. The `tenant` claim is persisted in `AspNetUserClaims`.
2. The `AcmeTenant` policy requires the claim `tenant` with value `"acme"`.
3. `[Authorize(Policy = "AcmeTenant")]` allows only users with that claim to access the action.

**Expected Output:** Alice can access `AcmeDashboard`; a user without the `tenant=acme` claim receives **403 Forbidden**.

**Why this output?** The `ClaimsPrincipal` includes the persisted claim, and the policy evaluates it during authorization.

### Real-World Cases

- **Multi-tenant SaaS:** A `tenant` claim scopes data access to the correct tenant.
- **Subscription tiers:** A `subscription` claim (`basic`, `premium`) drives feature gating.
- **Department-based access:** A `department` claim controls access to departmental resources.

### References

- Persisted claims and ClaimsPrincipalFactory – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- Policy-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- UserClaimsPrincipalFactory and role claims – https://github.com/dotnet/aspnetcore/discussions/65794

---

## 6. Identity Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Identity best practices are guidelines for configuring and using ASP.NET Core Identity securely, maintainably, and in alignment with ASP.NET Core’s security recommendations. |
| **Technical Definition** | Best practices encompass **using Identity for first-party accounts** (rather than custom auth), **configuring password and lockout policies**, **enabling email confirmation and 2FA**, **storing secrets securely**, **scaffolding only the Identity UI pages that need customisation**, **using policies and claims rather than encoding authorization in page logic**, and **persisting Data Protection keys in multi-instance deployments**. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for Identity—they help you avoid security holes (weak passwords, no lockout), maintainability problems (over-scaffolding the UI), and deployment issues (Data Protection keys not shared across servers). |

### Purposes

- **To** leverage Identity’s **built-in security features** (PBKDF2 hashing, lockout, 2FA) instead of reimplementing them.
- **To** enforce **strong password policies** and **account lockout** to resist brute-force attacks.
- **To** require **email confirmation** and offer **2FA** for sensitive applications.
- **To** keep secrets (connection strings, signing keys) out of source code and use secure stores.
- **To** minimise **scaffolding** to reduce upgrade and merge costs.
- **To** use **policy-based authorization** with claims for fine-grained control.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Auth implementation** | Use ASP.NET Core Identity for first-party accounts, login, password management, email confirmation, and MFA. | Implement password recovery or session management from scratch. |
| **Password policy** | Configure `RequiredLength` (≥ 8), require digit, uppercase, lowercase, and non-alphanumeric. | Leave the default minimum length of 6. |
| **Lockout** | Enable `LockoutEnabled` for users; use `lockoutOnFailure: true` on sign-in. | Allow unlimited failed login attempts. |
| **Email confirmation** | Set `SignIn.RequireConfirmedEmail = true` for apps where email ownership matters. | Allow unconfirmed emails to sign in for sensitive apps. |
| **2FA** | Enable and encourage two-factor authentication for sensitive accounts. | Rely on passwords alone for high-security apps. |
| **Secrets** | Store secrets in Azure Key Vault or User Secrets; never in source code or plain `appsettings.json`. | Commit production credentials to source control. |
| **UI scaffolding** | Scaffold only the Identity pages that need customisation. | Fully scaffold the Identity UI—it increases upgrade cost. |
| **Authorization** | Use policies and claims for fine-grained authorization. | Encode all authorization decisions in page/controller logic. |
| **Data Protection** | Persist Data Protection keys in a shared store for multi-instance deployments. | Rely on ephemeral local key storage in web farms. |
| **UserManager lifetime** | Resolve `UserManager` per request (it is Scoped); create a scope for background work. | Capture `UserManager` in a Singleton. |

### Annotated Code Example (Hardened Identity Configuration)

```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Strong password policy
    options.Password.RequiredLength = 12;
    options.Password.RequireDigit = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireNonAlphanumeric = true;

    // Lockout
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.AllowedForNewUsers = true;

    // Require confirmed email
    options.SignIn.RequireConfirmedEmail = true;
    options.SignIn.RequireConfirmedPhoneNumber = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

// Secret management: load connection string from a secure provider
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());
```

**Why this follows best practices:**

- **Strong password policy** (length 12, all character classes) resists brute-force and dictionary attacks.
- **Lockout** is enabled with a 15-minute window after 5 failures.
- **Email confirmation** is required before sign-in.
- **Connection string is loaded from configuration** (backed by a vault in production), not hard-coded.
- **Fallback policy** requires authentication for all endpoints by default.

### Real-World Cases

- **Public-facing consumer apps:** Strong passwords, lockout, email confirmation, and optional 2FA.
- **Enterprise applications:** Integration with corporate identity providers alongside Identity for first-party accounts.
- **Multi-instance deployments:** Shared Data Protection key ring to ensure cookies and tokens are valid across all instances.

### References

- Security And Identity (ASP.NET Core guidance) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- ASP.NET Core Security Best Practices 2026 – https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP DotNet Security Cheat Sheet – https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Configure ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration
- Security cards: Identity and secret handling – https://github.com/reware-labs/securitycards

---

## Summary Table: ASP.NET Core Identity at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **ASP.NET Core Identity** | Membership system for first-party accounts | `AddIdentity`, `AddIdentityApiEndpoints` | Apps managing their own users, passwords, and roles |
| **Identity setup** | Service registration and configuration | `IdentityDbContext`, `AddEntityFrameworkStores`, `IdentityOptions` | Every Identity application |
| **User management** | CRUD and password operations | `UserManager.CreateAsync`, `FindByIdAsync`, `DeleteAsync` | Registration, admin panels, profile updates |
| **Role management** | Create roles and assign users | `RoleManager.CreateAsync`, `UserManager.AddToRoleAsync` | Role-based authorization |
| **Claims** | Fine-grained user attributes | `AddClaimAsync`, `ClaimsPrincipalFactory` | Multi-tenant, subscription, department-based access |
| **Best practices** | Security and maintainability | Strong passwords, lockout, 2FA, secret management, minimal scaffolding | Every production Identity deployment |

---

## References (Full List)

- Introduction to Identity on ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- How to use Identity to secure a Web API backend for SPAs – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-api-authorization
- Configure ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration
- IdentityOptions class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityoptions
- UserManager<TUser> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1
- RoleManager<TRole> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.rolemanager-1
- UserManager.AddToRoleAsync – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.addtoroleasync
- IdentityResult class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.identityresult
- Security And Identity (ASP.NET Core guidance) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- ASP.NET Core Security Best Practices 2026 – https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP DotNet Security Cheat Sheet – https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Security cards: Identity and secret handling – https://github.com/reware-labs/securitycards
- UserClaimsPrincipalFactory and role claims (GitHub discussion) – https://github.com/dotnet/aspnetcore/discussions/65794
- ClaimsPrincipalFactory customization – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- Policy-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Identity API endpoints for SPA/API (Neon guide) – https://github.com/neondatabase/website/blob/e0d62939b056ff2080c9cbdb02b787548be4fc37/content/guides/aspnet-identity-auth.md
- Secure user data with authorization (user and role seeding) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/secure-data
- Adding claims with ClaimsPrincipalFactory (Stack Overflow) – https://stackoverflow.com/revisions/61ee112c-719e-43af-816f-80bc7703f8d7/view-source
- Custom ClaimsPrincipalFactory with roles (GitHub issue) – https://github.com/dotnet/aspnetcore/issues/46593