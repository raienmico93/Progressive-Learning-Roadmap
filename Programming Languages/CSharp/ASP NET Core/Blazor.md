# C# Blazor — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Blazor** is a modern .NET web UI framework that lets developers build **interactive web applications using C# and Razor syntax** instead of JavaScript. Blazor applications are composed of **components**—reusable, self-contained UI units that combine markup and C# logic. Blazor runs in three primary hosting models: **Blazor Server** (executes on the server over a SignalR connection), **Blazor WebAssembly** (executes entirely in the browser via WebAssembly), and **Blazor Hybrid** (runs natively in desktop and mobile apps via .NET MAUI, WPF, or Windows Forms). Blazor enables **full-stack C#** development, sharing code and libraries between client and server.

**Technical Definition**
Blazor is a component-based UI framework built on ASP.NET Core and Razor. Components are classes (conventionally `.razor` files) that derive from `ComponentBase` (or implement `IComponent`). The **render tree** is a lightweight in-memory representation of the UI; the **renderer** applies diffs to the DOM (or native UI) when state changes. Blazor Server uses a **SignalR circuit** to send UI updates to the browser and receive events. Blazor WebAssembly compiles .NET to WebAssembly and runs the app in the browser sandbox. Blazor Hybrid embeds a `BlazorWebView` control in native apps. Components support **parameters**, **event callbacks**, **lifecycle methods** (`OnInitialized`, `OnParametersSet`, `OnAfterRender`), **dependency injection**, **routing**, and **state management**.

**Beginner-Friendly Explanation**
Blazor is **C# for the browser and beyond**. Instead of writing JavaScript to make a web page interactive, you write C# components—small building blocks like a button, a form, or a page—that combine HTML and C# logic. Blazor can run in three places: on the **server** (the browser sends events to the server, and the server sends back UI updates over a real-time connection), in the **browser** (the C# is compiled to WebAssembly and runs locally), or inside a **native app** (desktop or mobile using .NET MAUI). Blazor lets you share code between client and server and use the same language everywhere.

---

### Key Characteristics

- **Component-based** — UI is built from reusable `.razor` components.
- **C# instead of JavaScript** — Full .NET ecosystem for UI logic.
- **Three hosting models** — Server, WebAssembly, Hybrid.
- **Render tree diffing** — Efficient UI updates based on state changes.
- **Dependency injection** — Components receive services via `[Inject]`.
- **Lifecycle methods** — `OnInitializedAsync`, `OnParametersSetAsync`, `OnAfterRenderAsync`, `DisposeAsync`.
- **Routing** — `@page` directive and `Router` component.
- **State management** — Component state, cascading parameters, DI scoped services, and persistent state in WebAssembly.
- **Streaming rendering** — Blazor Server can stream UI updates (`@attribute [StreamRendering]`).
- **Hot reload** — Faster development loop.

---

### Prerequisites

- Solid understanding of **C#** (classes, async/await, LINQ, events).
- Familiarity with **ASP.NET Core** (`Program.cs`, DI, configuration).
- Knowledge of **Razor syntax** (`@`, `@code`, `@foreach`, tag helpers).
- Basic understanding of **HTML** and **CSS**.
- Awareness of **HTTP** and **WebSockets/SignalR** (for Blazor Server).

---

### Related Programming Areas

- **ASP.NET Core** — Blazor is part of the ASP.NET Core family.
- **SignalR** — Powers Blazor Server’s real-time circuit.
- **WebAssembly** — Powers Blazor WebAssembly.
- **.NET MAUI** — Powers Blazor Hybrid.
- **Razor Pages and MVC** — Alternative server-rendered models.

---

### Core Concepts / Features

1. Blazor (Overarching Concept)
2. Blazor Server
3. Blazor WebAssembly
4. Blazor Hybrid
5. Components
6. Routing
7. State Management
8. Blazor Best Practices

---

## 1. Blazor (Overarching Concept)

### Definitions

**Core Definition:** Blazor is a .NET UI framework for building interactive web and hybrid apps using C# components.

**Technical Definition:** Blazor is built on the Razor component model. A Blazor app is a tree of components; each component is a `.razor` file containing markup and C# code. The renderer tracks state changes and updates the DOM (or native UI). The hosting model determines where the component code executes (server, browser, or native).

**Beginner-Friendly Explanation:** Blazor is **C# for building interactive UIs**. You write components; the framework handles rendering and updates.

---

### Purposes

- To build **interactive web UIs** with C# instead of JavaScript.
- To **share code** between client and server.
- To leverage the **.NET ecosystem** for UI logic.
- To support **web, desktop, and mobile** from one component model.
- To provide **type-safe**, maintainable UI code.

---

### Syntax Rules and Structure

```razor
@* Counter.razor *@
<h1>Counter</h1>
<p>Current count: @count</p>
<button @onclick="Increment">Click me</button>

@code {
    private int count;
    private void Increment() => count++;
}
```

**Component breakdown**

- Markup — HTML with `@` expressions.
- `@code` — C# logic block.
- `@onclick` — Event binding.
- `@count` — Data binding.

#### Syntax Rules

- Components are `.razor` files.
- `@code` holds C# fields and methods.
- Event bindings use `@on{event}`.

#### Constraints and Limitations

- Blazor WebAssembly requires downloading the .NET runtime to the browser (larger initial payload).
- Blazor Server requires a persistent SignalR connection.

---

### Annotated Code Examples and Expected Outputs

#### Example: Simple Counter Component

```razor
@page "/counter"
<h1>Counter</h1>
<p>Current count: @currentCount</p>
<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;
    private void IncrementCount() => currentCount++;
}
```

**Expected Output (rendered)**
```html
<h1>Counter</h1>
<p>Current count: 0</p>
<button class="btn btn-primary">Click me</button>
```

**Why this output occurs:** The component renders `currentCount`; clicking the button invokes `IncrementCount`, which updates the count and re-renders.

---

### Real-World Cases

- **Dashboards** — Interactive data visualizations.
- **Internal tools** — CRUD apps with rich UI.
- **Progressive web apps** — Offline-capable WebAssembly apps.
- **Cross-platform apps** — Blazor Hybrid for desktop/mobile.

---

### References

- Blazor overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/

---

## 2. Blazor Server

### Definitions

**Core Definition:** Blazor Server runs the component logic on the server and communicates UI updates to the browser over a SignalR connection.

**Technical Definition:** In Blazor Server, the component code executes on the server. The browser establishes a **SignalR circuit** with the server. UI events are sent from the browser to the server; the server processes them, re-renders the component, and sends a **diff** of the render tree back to the browser, which applies it to the DOM. State lives on the server for the duration of the circuit. Blazor Server is supported in .NET 8+ alongside **static SSR** and **interactive server** components.

**Beginner-Friendly Explanation:** Blazor Server is **“the brain runs on the server.”** The browser is a thin client; all C# logic runs on the server and the UI updates are streamed over a real-time connection.

---

### Purposes

- To run **full .NET** on the server without WebAssembly.
- To support **thin clients** and older browsers.
- To enable **fast startup** (small browser payload).
- To keep **sensitive logic** on the server.
- To share code with the server-side ASP.NET Core app.

---

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();

app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();
```

```razor
@* Component with interactive server mode *@
@rendermode InteractiveServer
```

**Component breakdown**

- `AddInteractiveServerComponents()` — Registers server interactivity.
- `AddInteractiveServerRenderMode()` — Enables server render mode.
- `@rendermode InteractiveServer` — Marks a component as interactive server.

#### Syntax Rules

- Interactive server components require a SignalR circuit.
- State is stored per circuit on the server.
- Use `@rendermode` to control interactivity in .NET 8+.

#### Constraints and Limitations

- **Requires a persistent connection**; latency affects interactivity.
- **Scales with concurrent circuits**; server resources per user.
- **Not offline-capable**.
- **No direct browser APIs**; calls must be proxied to JS interop.

---

### Annotated Code Examples and Expected Outputs

#### Example: Interactive Server Component

```razor
@page "/server-counter"
@rendermode InteractiveServer

<h1>Server Counter</h1>
<p>Count: @count</p>
<button @onclick="() => count++">Increment</button>

@code {
    private int count;
}
```

**Expected Output (browser)**

- Initial render: `Count: 0`.
- Click “Increment” → server increments `count` and sends a diff; browser shows `Count: 1`.

**Why this output occurs:** The click is sent to the server; the server updates state and sends the UI diff over SignalR.

---

### Real-World Cases

- **Intranet apps** — Internal tools with server-side data access.
- **Data-heavy dashboards** — Server-side processing.
- **Apps with thin clients** — Older browsers or low-powered devices.

---

### References

- ASP.NET Core Blazor hosting models — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models
- Blazor Server — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models#blazor-server

---

## 3. Blazor WebAssembly

### Definitions

**Core Definition:** Blazor WebAssembly runs the entire Blazor app in the browser using .NET compiled to WebAssembly.

**Technical Definition:** Blazor WebAssembly downloads the .NET runtime and the app’s assemblies to the browser, where they execute in the WebAssembly sandbox. UI updates happen locally without a server round-trip. It supports **offline** scenarios (with PWA), **client-side state**, and **static hosting**. In .NET 8+, Blazor Web App can use **Auto** render mode (server prerendering + WebAssembly interactivity).

**Beginner-Friendly Explanation:** Blazor WebAssembly is **“the brain runs in the browser.”** The app downloads the .NET runtime and runs entirely on the client.

---

### Purposes

- To run **C# directly in the browser**.
- To support **offline** and **PWA** scenarios.
- To **offload work** from the server.
- To provide **fast UI interactions** after initial load.
- To host on **static file servers** or CDNs.

---

### Syntax Rules and Structure

```csharp
// Program.cs (client)
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");
await builder.Build().RunAsync();
```

**Component breakdown**

- `WebAssemblyHostBuilder` — Client host builder.
- `RootComponents.Add<App>` — Root component.
- `RunAsync()` — Starts the app.

#### Syntax Rules

- The app runs in the browser sandbox.
- API calls use `HttpClient` to reach the server.
- State is client-side and lost on refresh unless persisted.

#### Constraints and Limitations

- **Larger initial download** (runtime + assemblies).
- **No direct server-side resources** (database, file system).
- **Browser sandbox** limitations (no threads, limited APIs).
- **Debugging** is more limited than server-side.

---

### Annotated Code Examples and Expected Outputs

#### Example: WebAssembly Counter

```razor
@page "/wasm-counter"

<h1>WASM Counter</h1>
<p>Count: @count</p>
<button @onclick="() => count++">Increment</button>

@code {
    private int count;
}
```

**Expected Output (browser)**

- Initial render: `Count: 0`.
- Click → `Count: 1` (no server round-trip).

**Why this output occurs:** The component runs locally in WebAssembly; state changes are processed in the browser.

---

### Real-World Cases

- **PWAs** — Offline-capable apps.
- **Client-heavy apps** — Rich interactivity.
- **Static-hosted apps** — Deployed to CDN.

---

### References

- Blazor WebAssembly — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models#blazor-webassembly

---

## 4. Blazor Hybrid

### Definitions

**Core Definition:** Blazor Hybrid runs Blazor components natively inside desktop and mobile apps using `BlazorWebView`.

**Technical Definition:** Blazor Hybrid hosts Razor components in a native app (via .NET MAUI, WPF, or Windows Forms) using a `BlazorWebView` control. Components execute in the .NET process, not in a browser or server, and render to an embedded web view. Hybrid apps have full access to native device APIs and platform capabilities. .NET MAUI Blazor Hybrid is the primary supported scenario.

**Beginner-Friendly Explanation:** Blazor Hybrid is **“Blazor inside a native app.”** The UI is built with Blazor components, but it runs in a desktop or mobile app with native capabilities.

---

### Purposes

- To build **cross-platform native apps** with Blazor.
- To reuse **Blazor components** across web and native.
- To access **native device APIs** (camera, file system, sensors).
- To deliver **offline-first** apps.
- To share code with web apps.

---

### Syntax Rules and Structure

```csharp
// MauiProgram.cs (MAUI)
builder.Services.AddMauiBlazorWebView();

// MainPage.xaml
<BlazorWebView HostPage="wwwroot/index.html">
    <BlazorWebView.RootComponents>
        <RootComponent Selector="#app" ComponentType="{x:Type local:Main}" />
    </BlazorWebView.RootComponents>
</BlazorWebView>
```

**Component breakdown**

- `AddMauiBlazorWebView()` — Registers the Blazor WebView.
- `<BlazorWebView>` — Hosts the components.
- `<RootComponent>` — Root component mapping.

#### Syntax Rules

- Hybrid apps use `BlazorWebView` (MAUI, WPF, WinForms).
- Components run in the .NET process with native access.
- `wwwroot` hosts the HTML shell.

#### Constraints and Limitations

- **Platform-specific** project setup.
- **Native APIs** require platform-specific code or MAUI Essentials.
- **Not browser-hosted**; deployment is via app stores or installers.

---

### Annotated Code Examples and Expected Outputs

#### Example: MAUI Blazor Hybrid Component

```razor
@* Main.razor *@
<h1>Hybrid Counter</h1>
<p>Count: @count</p>
<button @onclick="() => count++">Increment</button>

@code {
    private int count;
}
```

**Expected Output (native app)**

- Initial render: `Count: 0`.
- Click → `Count: 1`.

**Why this output occurs:** The component runs in the native .NET process and renders to the embedded web view.

---

### Real-World Cases

- **Cross-platform apps** — MAUI Blazor Hybrid for iOS, Android, Windows, macOS.
- **Line-of-business apps** — Desktop apps with Blazor UI.
- **Offline apps** — Native access with local storage.

---

### References

- ASP.NET Core Blazor Hybrid — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hybrid/

---

## 5. Components

### Definitions

**Core Definition:** A component is a reusable, self-contained UI unit defined in a `.razor` file with markup and C# logic.

**Technical Definition:** Components derive from `ComponentBase` (or implement `IComponent`). They accept **parameters** (`[Parameter]`), expose **event callbacks** (`EventCallback<T>`), and participate in the **render tree**. Lifecycle methods (`OnInitialized`, `OnInitializedAsync`, `OnParametersSet`, `OnParametersSetAsync`, `ShouldRender`, `OnAfterRender`, `OnAfterRenderAsync`) control initialization, parameter changes, and post-render logic. Components can be **generic** (`@typeparam T`) and support **child content** (`RenderFragment`).

**Beginner-Friendly Explanation:** A component is **a reusable piece of UI**—a button, a form, a page. You write it once and reuse it everywhere.

---

### Purposes

- To **reuse UI** across pages and apps.
- To **encapsulate** state and logic.
- To **compose** complex UIs from small pieces.
- To share components across hosting models.
- To **test** UI logic in C#.

---

### Syntax Rules and Structure

```razor
@* MyComponent.razor *@
<h3>@Title</h3>
<button @onclick="OnClick">Click</button>

@code {
    [Parameter] public string Title { get; set; } = "";
    [Parameter] public EventCallback OnClick { get; set; }
}
```

**Component breakdown**

- `[Parameter]` — Input from parent.
- `EventCallback` — Output event.
- `@code` — Logic.

#### Syntax Rules

- Parameters are public properties with `[Parameter]`.
- Event callbacks are `EventCallback` or `EventCallback<T>`.
- Child content uses `RenderFragment`.
- Lifecycle methods are overridable.

#### Constraints and Limitations

- Components must not mutate parameters.
- State should be local or in DI services.
- `OnAfterRender` runs only in interactive modes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Component with Parameter and EventCallback

```razor
@* Counter.razor *@
<p>@Label: @Value</p>
<button @onclick="() => OnIncrement.InvokeAsync()">Increment</button>

@code {
    [Parameter] public string Label { get; set; } = "Count";
    [Parameter] public int Value { get; set; }
    [Parameter] public EventCallback OnIncrement { get; set; }
}
```

```razor
@* Parent.razor *@
<Counter Label="Clicks" Value="@count" OnIncrement="() => count++" />

@code {
    private int count;
}
```

**Expected Output (initial render)**
```
Clicks: 0
[Increment]
```

**Why this output occurs:** The parent passes `Value` and handles `OnIncrement`; the child renders the label and raises the event.

---

### Real-World Cases

- **Layouts** — Shared shell components.
- **Forms** — Reusable form controls.
- **Charts** — Chart components with parameters.
- **Navigation** — Menu and breadcrumb components.

---

### References

- ASP.NET Core Razor components — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/components/

---

## 6. Routing

### Definitions

**Core Definition:** Routing maps URLs to Blazor components using the `@page` directive and the `Router` component.

**Technical Definition:** The `@page` directive on a `.razor` file assigns a route template. The `Router` component (`Microsoft.AspNetCore.Components.Routing.Router`) matches the URL to components. Route parameters (`@page "/products/{id:int}"`) bind to component parameters. Navigation is performed via the `NavigationManager`. Layouts (`@layout`) wrap pages. In .NET 8+, routing supports enhanced navigation and streaming rendering.

**Beginner-Friendly Explanation:** Routing is **which component appears at which URL**. `@page "/products"` makes a component the products page.

---

### Purposes

- To **map URLs to components**.
- To support **route parameters** and constraints.
- To enable **navigation** programmatically.
- To share **layouts** across pages.
- To support **nested routes** and areas.

---

### Syntax Rules and Structure

```razor
@page "/products"
@page "/products/{Id:int}"

<h1>Product @Id</h1>

@code {
    [Parameter] public int Id { get; set; }
}
```

**Component breakdown**

- `@page` — Route template.
- `{Id:int}` — Route parameter with constraint.
- `[Parameter]` — Bound from route.

#### Syntax Rules

- Multiple `@page` directives are allowed.
- Route parameters must match component parameters.
- `NavigationManager.NavigateTo` performs programmatic navigation.
- `@layout` assigns a layout.

#### Constraints and Limitations

- Route constraints are limited to built-in types.
- Catch-all parameters (`{*path}`) are supported.

---

### Annotated Code Examples and Expected Outputs

#### Example: Route Parameter

```razor
@page "/product/{Id:int}"

<h1>Product @Id</h1>

@code {
    [Parameter] public int Id { get; set; }
}
```

**Expected Output (URL `/product/42`)**
```html
<h1>Product 42</h1>
```

**Why this output occurs:** The route parameter `{Id:int}` binds to the `Id` parameter.

---

### Real-World Cases

- **Product pages** — `/product/{id}`.
- **Blog posts** — `/blog/{slug}`.
- **Admin areas** — `/admin/users`.
- **Localized routes** — `/en/products`.

---

### References

- ASP.NET Core Blazor routing — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/routing

---

## 7. State Management

### Definitions

**Core Definition:** State management is how a Blazor app stores, updates, and shares data across components and pages.

**Technical Definition:** State can be **component-local** (fields), **shared via cascading parameters** (`CascadingValue`), **DI services** (scoped to circuit in Server; singleton in WASM), **persistent storage** (`localStorage`, `sessionStorage`, IndexedDB via JS interop), or **server state** (database). In .NET 8+, **persistent component state** (`[PersistentState]`) survives prerendering. State changes trigger re-renders via `StateHasChanged()`.

**Beginner-Friendly Explanation:** State management is **keeping track of data across the UI**—who is logged in, what is in the cart, which tab is open.

---

### Purposes

- To **share data** between components.
- To **persist state** across navigation or reloads.
- To **synchronize** UI with server data.
- To avoid **unnecessary re-renders**.
- To **test** state logic independently.

---

### Syntax Rules and Structure

```razor
@* Cascading parameter *@
<CascadingValue Value="@theme">
    <ChildComponent />
</CascadingValue>

@* Child *@
@code {
    [CascadingParameter] public Theme Theme { get; set; } = default!;
}
```

```csharp
// DI service (scoped in Server, singleton in WASM)
builder.Services.AddScoped<CartService>();
```

**Component breakdown**

- `CascadingValue` — Passes state down the tree.
- `[CascadingParameter]` — Receives it.
- DI services — Shared state via `AddScoped`/`AddSingleton`.
- `[PersistentState]` — Survives prerendering (.NET 8+).

#### Syntax Rules

- Use cascading parameters for theme, auth, or layout state.
- Use DI services for shared app state.
- Use JS interop for browser storage.
- Call `StateHasChanged()` after async state changes (if not automatic).

#### Constraints and Limitations

- Cascading values can cause widespread re-renders if not used carefully.
- Server-side scoped services are per circuit; not shared across users.
- WebAssembly state is lost on refresh unless persisted.

---

### Annotated Code Examples and Expected Outputs

#### Example: Shared Cart Service

```csharp
public class CartService
{
    public List<string> Items { get; } = new();
    public event Action? OnChange;
    public void Add(string item) { Items.Add(item); OnChange?.Invoke(); }
}
```

```razor
@inject CartService Cart
@implements IDisposable

<p>Items: @Cart.Items.Count</p>
<button @onclick="() => Cart.Add(\"Apple\")">Add</button>

@code {
    protected override void OnInitialized() => Cart.OnChange += StateHasChanged;
    public void Dispose() => Cart.OnChange -= StateHasChanged;
}
```

**Expected Output (after clicking)**

- Initial: `Items: 0`.
- After click: `Items: 1`.

**Why this pattern is used:** The service holds shared state; the component subscribes to changes and re-renders.

---

### Real-World Cases

- **Shopping carts** — Shared cart state.
- **Authentication** — User identity via cascading auth state.
- **Themes** — Cascading theme values.
- **Forms** — Multi-step form state.

---

### References

- ASP.NET Core Blazor state management — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management

---

## 8. Blazor Best Practices

### Definitions

**Core Definition:** Blazor best practices are the guidelines for building performant, maintainable, secure, and testable Blazor apps.

**Technical Definition:** Best practices include: **choose the right hosting model** (Server for thin clients, WASM for offline, Hybrid for native); **keep components small and focused**; **use `@key` for list rendering**; **avoid unnecessary re-renders** (`ShouldRender`); **use cascading parameters sparingly**; **prefer DI services for shared state**; **use `IDisposable`/`IAsyncDisposable` for subscriptions**; **secure WebAssembly apps** (no secrets in client); **enable prerendering carefully**; and **test components with bUnit**.

**Beginner-Friendly Explanation:** The best practices are: **pick the right hosting model**, **keep components small**, **use `@key`**, **avoid re-render storms**, **secure WASM**, and **test your components**.

---

### Guidelines and Structure

#### General Guidelines

```razor
@* 1. Use @key for lists *@
@foreach (var item in items)
{
    <ItemComponent @key="item.Id" Item="item" />
}

@* 2. Avoid unnecessary re-renders *@
@code {
    protected override bool ShouldRender() => _needsRender;
}

@* 3. Dispose subscriptions *@
@implements IDisposable
@code {
    protected override void OnInitialized() => _service.OnChange += StateHasChanged;
    public void Dispose() => _service.OnChange -= StateHasChanged;
}

@* 4. Use cascading parameters sparingly *@
<CascadingValue Value="@theme" IsFixed="true">
    <Layout />
</CascadingValue>
```

**Component breakdown**

- **`@key`** — Preserves component identity in lists.
- **`ShouldRender`** — Controls re-render frequency.
- **Disposal** — Prevents memory leaks.
- **`IsFixed`** — Optimizes cascading values that never change.

#### Rules and Constraints

- **Do** choose the right hosting model.
- **Do** use `@key` in lists.
- **Do** dispose subscriptions.
- **Do** keep components small.
- **Do** secure WebAssembly apps (no secrets).
- **Do not** put secrets in client-side code.
- **Do not** overuse cascading parameters.
- **Do not** mutate parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Structured Component

```razor
@inject ICartService Cart
@implements IDisposable

<ul>
@foreach (var item in Cart.Items)
{
    <li @key="item.Id">@item.Name</li>
}
</ul>

@code {
    protected override void OnInitialized() => Cart.OnChange += StateHasChanged;
    public void Dispose() => Cart.OnChange -= StateHasChanged;
}
```

**Expected Output (conceptual)**
```
// Efficient list rendering with @key; no memory leaks due to Dispose.
```

**Why this pattern is used:** `@key` preserves identity; disposal prevents leaks; DI provides shared state.

---

### Real-World Cases

- **Large lists** — `@key` for stable rendering.
- **Real-time apps** — Disposal of SignalR subscriptions.
- **Secure apps** — No secrets in WASM.

---

### References

- ASP.NET Core Blazor best practices — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/
- ASP.NET Core Blazor state management — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management
- ASP.NET Core Blazor routing — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/routing
- ASP.NET Core Razor components — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/components/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Blazor Server (.NET 7 and earlier) | **Superseded** by Blazor Web App (.NET 8+) with render modes. |
| Blazor WebAssembly standalone | **Supported**; still valid for static-hosted apps. |
| Blazor Hybrid | **Supported**; MAUI is the primary scenario. |
| `@rendermode` | **.NET 8+**; controls interactivity. |
| `[PersistentState]` | **.NET 8+**; survives prerendering. |
| `@key` | **Recommended** for lists. |
| Secrets in WASM | **Unsafe**; never embed secrets in client code. |

---

## References (Consolidated)

- Blazor overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/
- ASP.NET Core Blazor hosting models — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models
- Blazor Server — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models#blazor-server
- Blazor WebAssembly — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models#blazor-webassembly
- ASP.NET Core Blazor Hybrid — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/hybrid/
- ASP.NET Core Razor components — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/components/
- ASP.NET Core Blazor routing — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/routing
- ASP.NET Core Blazor state management — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management

---