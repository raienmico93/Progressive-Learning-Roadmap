# ASP.NET Core SignalR: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core SignalR** (.NET 6–10), the modern, actively maintained real‑time communication library. The legacy ASP.NET SignalR (pre‑Core) is **effectively dead** and not covered. SignalR uses **WebSockets** as its preferred transport, with automatic fallback to **Server‑Sent Events** and **Long Polling**. It is built for **server‑to‑client RPC** and supports **scale‑out** via a **Redis backplane** or **Azure SignalR Service**. SignalR is **not a replacement for REST**; it complements HTTP APIs for push‑based, low‑latency scenarios. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | SignalR is an open‑source library that simplifies adding real‑time web functionality to apps, enabling server‑side code to push content to connected clients instantly. |
| **Technical Definition** | SignalR provides an API for creating **server‑to‑client remote procedure calls (RPCs)** . It handles connection management, transport negotiation (WebSocket → SSE → Long Polling), and message dispatch across machine boundaries. The central abstraction is the **Hub**, a high‑level pipeline that allows clients and servers to call methods on each other. SignalR supports **groups**, **user targeting**, **streaming**, **strongly‑typed hubs**, **Hub filters**, and **scale‑out** via backplanes. |
| **Beginner‑Friendly Explanation** | SignalR is the **“live update”** system for your app. Instead of the client constantly asking “anything new?” (polling), the server pushes updates the moment they happen. A chat message appears instantly; a stock price updates without a refresh; a dashboard reflects changes in real time. SignalR handles the messy parts—connection drops, transport fallbacks, and scaling across servers—so you focus on the messages. |

### Key Characteristics

- **Server‑to‑client push** – Server code invokes methods on connected clients instantly.
- **Automatic transport negotiation** – WebSockets preferred; SSE and Long Polling as fallbacks.
- **Hub‑based abstraction** – A high‑level pipeline for bidirectional method calls.
- **Groups and user targeting** – Send messages to specific subsets of connections.
- **Streaming support** – Server‑to‑client and client‑to‑server streaming via `IAsyncEnumerable<T>`.
- **Scale‑out ready** – Redis backplane or Azure SignalR Service for multi‑instance deployments.
- **Strongly‑typed hubs** – `Hub<TClient>` provides compile‑time safety for client method calls.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **C# async/await**, **dependency injection**, and **middleware**.
- Basic understanding of **WebSockets** and **HTTP transports**.
- A JavaScript/TypeScript or .NET client (for the client examples).

### Related Programming Areas

- **WebSockets** – The preferred underlying transport.
- **Real‑time communication** – The broader category (chat, notifications, dashboards).
- **ASP.NET Core DI** – Hubs and `IHubContext` integrate with the service container.
- **Authentication/Authorization** – JWT/cookies and `[Authorize]` on hubs.
- **Scaling** – Redis backplane, Azure SignalR Service.

### Core Concepts / Features

1. **SignalR** – The library, architecture, and transports.
2. **Real‑time communication** – The use cases and patterns.
3. **Hubs** – The central abstraction, lifecycle, and methods.
4. **Clients** – Targeting mechanisms and client SDKs.
5. **Groups** – Group management and message routing.
6. **SignalR best practices** – Security, scaling, reconnection, and performance.

---

## 1. SignalR (The Library)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | SignalR is an open‑source ASP.NET Core library for adding real‑time, server‑push functionality to web applications. |
| **Technical Definition** | SignalR provides an **RPC abstraction** over persistent connections. It negotiates the best available transport (WebSocket > SSE > Long Polling), manages connections and reconnections, and dispatches messages between server‑side **Hubs** and client‑side proxies. On ASP.NET Core, SignalR is registered via `AddSignalR()` and hubs are mapped via `MapHub<T>()`. It supports **JSON** (default) and **MessagePack** hub protocols, **streaming**, **filters**, and **scale‑out** through backplanes. |
| **Beginner‑Friendly Explanation** | SignalR is the **“real‑time engine”**. You define a Hub (a C# class) with methods that clients can call, and methods that call clients. SignalR handles the connection, picks the best transport, and delivers messages. If the connection drops, it can reconnect automatically. If you run multiple servers, SignalR can use a backplane to keep all clients in sync. |

### Purposes

- **To** add **server‑push** functionality without polling.
- **To** abstract away **transport complexity** (WebSocket vs. SSE vs. Long Polling).
- **To** provide **automatic connection management** and reconnection.
- **To** enable **scale‑out** across multiple server instances via backplanes.
- **To** support **strongly‑typed, bidirectional RPC** between client and server.

### Syntax Rules and Structure

#### General Syntax (Server Registration)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR();           // Register SignalR services

var app = builder.Build();
app.MapHub<ChatHub>("/chathub");          // Map the hub endpoint
app.Run();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddSignalR()` | Registers SignalR services in DI. |
| `MapHub<ChatHub>("/chathub")` | Maps the hub to a route. |

#### Transport Negotiation

| Priority | Transport | Notes |
|----------|-----------|-------|
| 1 | **WebSockets** | Preferred; lowest latency, full duplex. |
| 2 | **Server‑Sent Events (SSE)** | Fallback; server‑to‑client only. |
| 3 | **Long Polling** | Last resort; highest overhead. |

**Syntax rules:**

- **WebSockets must be enabled** on the server (IIS/nginx/Azure) for the preferred transport.
- **`SkipNegotiation = true`** can be used to force WebSockets directly (performance optimisation).
- **Transport selection is automatic** based on client and server capabilities.

**Constraints and limitations:**

- **WebSocket support** may be blocked by proxies, firewalls, or older servers; fallback transports handle this but with higher latency.
- **SignalR is stateful** – connections are long‑lived; load balancers must support sticky sessions or a backplane.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR(options =>
{
    options.EnableDetailedErrors = false;   // Security: no stack traces to clients
});

var app = builder.Build();

app.MapHub<ChatHub>("/hubs/chat");

app.Run();
```

**Step‑by‑step explanation:**

1. `AddSignalR()` registers SignalR services with detailed errors disabled.
2. `MapHub<ChatHub>("/hubs/chat")` exposes the hub at the specified route.
3. Clients connect to `/hubs/chat` using the SignalR client library.

**Expected Output:** A SignalR endpoint is available at `/hubs/chat`; clients can connect and invoke hub methods.

**Why this output?** The hub mapping creates the endpoint; the SignalR client negotiates the transport and establishes the connection.

### Real‑World Cases

- **Chat applications:** Instant message delivery between users.
- **Live dashboards:** Real‑time metrics and monitoring.
- **Notifications:** Push alerts to specific users or groups.
- **Collaborative editing:** Real‑time document changes.

### References

- Overview of ASP.NET Core SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction
- SignalR transports – https://learn.microsoft.com/en-us/aspnet/core/signalr/transports

---

## 2. Real‑Time Communication (The Use Cases)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Real‑time communication is the ability for a server to push data to clients instantly as events occur, without the client requesting it. |
| **Technical Definition** | Real‑time communication in SignalR is implemented through **persistent connections** (WebSocket, SSE, or Long Polling) and **server‑to‑client RPC**. Unlike request‑response HTTP, the server can invoke client methods at any time. SignalR manages the connection lifecycle, including reconnection and state recovery. |
| **Beginner‑Friendly Explanation** | Real‑time communication is the **“live feed”** experience. Think of a live sports score, a chat window, or a collaborative document—updates appear the moment they happen, without you refreshing the page. SignalR makes this possible by keeping a connection open and pushing messages as events occur. |

### Purposes

- **To** deliver **instant updates** without polling.
- **To** support **collaborative, multi‑user** experiences.
- **To** enable **live notifications** and alerts.
- **To** reduce **server load** compared to frequent polling.
- **To** improve **user experience** with low‑latency interactions.

### Syntax Rules and Structure

#### When to Use SignalR

| Scenario | Example | SignalR Fit |
|----------|---------|-------------|
| High‑frequency updates | Gaming, auctions, GPS tracking | ✅ Excellent |
| Collaborative apps | Whiteboards, team meetings | ✅ Excellent |
| Notifications | Chat, email alerts, social | ✅ Excellent |
| Simple CRUD | Form submissions | ❌ Use REST instead |

**Syntax rules:**

- **SignalR is for push‑based scenarios** – do not replace regular REST APIs for CRUD.
- **Real‑time is best when the client needs to react to server events** – not when the client simply requests data.

**Constraints and limitations:**

- **Stateful connections** complicate load balancing; use a backplane for scale‑out.
- **Not a replacement for message queues** – SignalR is for client‑facing real‑time, not durable messaging.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Server pushes a notification to all clients
public class NotificationHub : Hub
{
    public async Task SendNotification(string message)
    {
        await Clients.All.SendAsync("ReceiveNotification", message);
    }
}
```

```javascript
// Client receives the notification
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/notificationHub")
    .build();

connection.on("ReceiveNotification", (message) => {
    console.log("Notification:", message);
});

await connection.start();
```

**Step‑by‑step explanation:**

1. The hub method `SendNotification` is callable by clients.
2. It invokes `ReceiveNotification` on all connected clients.
3. The client registers a handler for `ReceiveNotification` and logs the message.

**Expected Output:** When any client calls `SendNotification("Hello")`, every connected client’s console logs “Notification: Hello”.

**Why this output?** SignalR broadcasts the message to all connections; each client’s registered handler fires.

### Real‑World Cases

- **Stock tickers:** Real‑time price updates.
- **Multiplayer games:** State synchronisation.
- **IoT dashboards:** Live sensor data.

### References

- Real‑time web functionality with SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction

---

## 3. Hubs (The Central Abstraction)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Hub is a high‑level pipeline that allows clients and servers to call methods on each other, serving as the central communication point in SignalR. |
| **Technical Definition** | A Hub is a class that inherits from `Hub` (or `Hub<TClient>` for strongly‑typed hubs). It defines **public methods** that clients can invoke, and uses `Clients`, `Groups`, and `Context` properties to interact with connections. Hubs are **transient**—a new instance is created per method invocation—so they must not store state in instance fields. Lifecycle events (`OnConnectedAsync`, `OnDisconnectedAsync`) allow connection management. |
| **Beginner‑Friendly Explanation** | A Hub is the **“switchboard”** between clients and the server. Clients call hub methods to send messages or perform actions. The hub calls client methods to push updates. It also provides access to **who is calling** (`Context`), **who to send to** (`Clients`), and **group management** (`Groups`). Hubs are created fresh for each call, so they should not hold state—use the database or external storage for that. |

### Purposes

- **To** provide a **central, typed entry point** for real‑time communication.
- **To** define **server methods** callable by clients.
- **To** define **client methods** callable by the server (via `Clients`).
- **To** manage **connection lifecycle** (`OnConnectedAsync`, `OnDisconnectedAsync`).
- **To** enable **strongly‑typed** hubs (`Hub<TClient>`) for compile‑time safety.

### Syntax Rules and Structure

#### General Syntax (Basic Hub)

```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: Hub` | Inherits the base hub class. |
| `public async Task SendMessage(...)` | A method callable by clients. |
| `Clients.All.SendAsync(...)` | Invokes a method on all connected clients. |

#### General Syntax (Strongly‑Typed Hub)

```csharp
public interface IChatClient
{
    Task ReceiveMessage(string user, string message);
}

public class ChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.ReceiveMessage(user, message);   // Compile-time checked
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Hub<IChatClient>` | Strongly‑typed hub; client methods are defined in the interface. |
| `Clients.All.ReceiveMessage(...)` | Invokes the interface method on all clients. |

#### Lifecycle Methods

```csharp
public override async Task OnConnectedAsync()
{
    await Groups.AddToGroupAsync(Context.ConnectionId, "SignalR Users");
    await base.OnConnectedAsync();
}

public override async Task OnDisconnectedAsync(Exception? exception)
{
    await base.OnDisconnectedAsync(exception);
}
```

**Syntax rules:**

- **Hubs are transient** – do not store state in instance fields.
- **Use `Context.ConnectionId`** to identify the current connection.
- **Use `Context.UserIdentifier`** (from `IUserIdProvider`) to identify the user.
- **`OnConnectedAsync`** is the place to add connections to groups.
- **`OnDisconnectedAsync`** is where cleanup logic runs; group removal is automatic.

**Constraints and limitations:**

- **Strongly‑typed hubs disable `SendAsync`** – you must use the interface methods.
- **Hub method names are not stripped of `Async`** – client method names must match exactly.
- **Long‑running hub methods block the connection** – offload to background services.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public interface IChatClient
{
    Task ReceiveMessage(string user, string message);
    Task UserJoined(string user);
    Task UserLeft(string user);
}

public class ChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.ReceiveMessage(user, message);
    }

    public async Task JoinRoom(string room)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, room);
        await Clients.Group(room).UserJoined(
            Context.User?.Identity?.Name ?? "Anonymous");
    }

    public async Task LeaveRoom(string room)
    {
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, room);
        await Clients.Group(room).UserLeft(
            Context.User?.Identity?.Name ?? "Anonymous");
    }

    public override async Task OnConnectedAsync()
    {
        await base.OnConnectedAsync();
    }

    public override async Task OnDisconnectedAsync(Exception? exception)
    {
        await base.OnDisconnectedAsync(exception);
    }
}
```

**Step‑by‑step explanation:**

1. `IChatClient` defines the client methods with compile‑time safety.
2. `ChatHub` inherits `Hub<IChatClient>`.
3. `SendMessage` broadcasts to all clients.
4. `JoinRoom` adds the caller to a group and notifies the group.
5. `LeaveRoom` removes the caller and notifies the group.
6. Lifecycle methods are overridden for connection management.

**Expected Output:** Clients can send messages, join/leave rooms, and receive group notifications; the strongly‑typed interface prevents typo‑based bugs.

**Why this output?** The hub dispatches calls to the appropriate `Clients` targeting methods; the interface ensures client method names are correct at compile time.

### Real‑World Cases

- **Chat rooms:** Join/leave, message broadcasting.
- **Notifications:** Per‑user or per‑group alerts.
- **Collaborative tools:** User presence (joined/left).

### References

- Use hubs in ASP.NET Core SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs
- Strongly‑typed hubs – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs#strongly-typed-hubs

---

## 4. Clients (Targeting Mechanisms)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | In SignalR, “Clients” refers to the set of connected client connections that a hub method can target when sending messages. |
| **Technical Definition** | The `IHubClients` interface (exposed as `Hub.Clients`) provides properties and methods for targeting: `All`, `Caller`, `Others`, `Client(connectionId)`, `Group(groupName)`, `User(userId)`, `Users(userIds)`, `OthersInGroup(groupName)`, `GroupExcept(groupName, excludedConnectionIds)`, `Groups(groupNames)`, and more. Each targeting method returns an `IClientProxy` (or `IHubClients<T>`) that invokes the specified client method. |
| **Beginner‑Friendly Explanation** | “Clients” is **who you are talking to**. `Clients.All` means everyone. `Clients.Caller` means just the client who called the hub method. `Clients.Others` means everyone except the caller. `Clients.Group("room")` means everyone in that group. You pick the audience, then call a method on them—like `Clients.Group("room").SendAsync("ReceiveMessage", ...)`. |

### Purposes

- **To** **target messages precisely** to the intended audience.
- **To** support **broadcast** (`All`), **caller‑only** (`Caller`), and **exclusion** (`Others`) patterns.
- **To** enable **group‑based** and **user‑based** communication.
- **To** avoid sending messages to clients that should not receive them (security and efficiency).

### Syntax Rules and Structure

#### Targeting Properties and Methods

| Target | Description |
|--------|-------------|
| `Clients.All` | All connected clients. |
| `Clients.Caller` | The client that invoked the hub method. |
| `Clients.Others` | All clients except the caller. |
| `Clients.Client(connectionId)` | A specific connection. |
| `Clients.Group(groupName)` | All connections in a group. |
| `Clients.OthersInGroup(groupName)` | Group members except the caller. |
| `Clients.User(userId)` | All connections for a user. |
| `Clients.Users(userIds)` | Connections for multiple users. |
| `Clients.GroupExcept(group, excludedIds)` | Group members except specified connections. |
| `Clients.Groups(groupNames)` | Multiple groups. |

#### General Syntax

```csharp
// Broadcast to all
await Clients.All.SendAsync("ReceiveMessage", user, message);

// Send only to the caller
await Clients.Caller.SendAsync("ReceiveMessage", user, message);

// Send to a group
await Clients.Group("SignalR Users").SendAsync("ReceiveMessage", user, message);

// Send to a specific user
await Clients.User(userId).SendAsync("ReceiveMessage", user, message);
```

**Syntax rules:**

- **`SendAsync` is used with untyped hubs**; strongly‑typed hubs use the interface methods.
- **`Clients.User(userId)`** requires an `IUserIdProvider` to map connections to users.
- **`Clients.Caller`** is useful for acknowledgements and caller‑specific responses.

**Constraints and limitations:**

- **`Clients.All` includes the caller** —use `Clients.Others` if the caller should be excluded.
- **User targeting** depends on authentication; unauthenticated connections have no `UserIdentifier`.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class ChatHub : Hub
{
    public async Task SendToAll(string message)
        => await Clients.All.SendAsync("ReceiveMessage", "System", message);

    public async Task SendToCaller(string message)
        => await Clients.Caller.SendAsync("ReceiveMessage", "System", message);

    public async Task SendToGroup(string group, string message)
        => await Clients.Group(group).SendAsync("ReceiveMessage", "System", message);

    public async Task SendToUser(string userId, string message)
        => await Clients.User(userId).SendAsync("ReceiveMessage", "System", message);
}
```

**Expected Output:**

| Hub Method Call | Recipients |
|-----------------|------------|
| `SendToAll("hi")` | All connected clients |
| `SendToCaller("hi")` | Only the caller |
| `SendToGroup("room1", "hi")` | Members of `room1` |
| `SendToUser("user42", "hi")` | Connections for `user42` |

**Why this output?** Each targeting property selects a different subset of connections; the message is delivered only to those connections.

### Real‑World Cases

- **Chat:** `Clients.All` for broadcast, `Clients.Group` for rooms, `Clients.Caller` for acknowledgements.
- **Notifications:** `Clients.User(userId)` for personal alerts.
- **Presence:** `Clients.Others` to notify others when a user joins.

### References

- `IHubClients` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.signalr.ihubclients
- Client targeting – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs#client-targeting

---

## 5. Groups (Group Management)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Groups are named collections of SignalR connections that allow messages to be sent to a subset of connected clients. |
| **Technical Definition** | Groups are managed via `IGroupManager` (exposed as `Hub.Groups`). `AddToGroupAsync(connectionId, groupName)` adds a connection; `RemoveFromGroupAsync` removes it. Group membership is **not persisted** across server restarts; it is automatically cleaned up on disconnect (explicit `RemoveFromGroupAsync` is not required in `OnDisconnectedAsync`). A connection can belong to multiple groups. Messages sent to a group are delivered to all connections in that group. |
| **Beginner‑Friendly Explanation** | Groups are **“chat rooms”** or **“channels”** . You add a connection to a group (e.g., `"room1"`), and then you can send a message to everyone in that group with `Clients.Group("room1")`. A connection can be in several groups at once. Groups are perfect for scoping messages—only interested clients receive them. |

### Purposes

- **To** **scope messages** to a subset of clients (rooms, topics, teams).
- **To** avoid sending every message to every client (efficiency).
- **To** simplify **per‑room** or **per‑tenant** communication.
- **To** support **dynamic membership** as users join and leave.

### Syntax Rules and Structure

#### General Syntax (Group Management)

```csharp
public async Task JoinRoom(string room)
{
    await Groups.AddToGroupAsync(Context.ConnectionId, room);
}

public async Task LeaveRoom(string room)
{
    await Groups.RemoveFromGroupAsync(Context.ConnectionId, room);
}

public async Task SendToRoom(string room, string message)
{
    await Clients.Group(room).SendAsync("ReceiveMessage", message);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Groups.AddToGroupAsync(connectionId, groupName)` | Adds a connection to a group. |
| `Groups.RemoveFromGroupAsync(...)` | Removes a connection from a group. |
| `Clients.Group(groupName)` | Targets all connections in the group. |

**Syntax rules:**

- **Group names are strings**; choose meaningful names (room IDs, tenant IDs, user IDs).
- **A connection can belong to multiple groups**.
- **Group membership is automatically removed on disconnect** —no need to call `RemoveFromGroupAsync` in `OnDisconnectedAsync`.
- **Groups are not persisted** —if the server restarts, groups must be re‑established (clients typically re‑join on reconnect).

**Constraints and limitations:**

- **Groups are not a security boundary** —you must still authorize access to group‑scoped resources (a user could try to join a group they are not allowed to).
- **Group membership is per‑connection** —a user with multiple connections must be added to the group on each connection.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class ChatHub : Hub
{
    public async Task JoinRoom(string room)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, room);
        await Clients.Group(room).SendAsync("UserJoined", Context.ConnectionId);
    }

    public async Task LeaveRoom(string room)
    {
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, room);
        await Clients.Group(room).SendAsync("UserLeft", Context.ConnectionId);
    }

    public async Task SendToRoom(string room, string message)
    {
        await Clients.Group(room).SendAsync("ReceiveMessage", room, message);
    }
}
```

**Step‑by‑step explanation:**

1. `JoinRoom` adds the caller’s connection to the group and notifies the group.
2. `LeaveRoom` removes the connection and notifies the group.
3. `SendToRoom` sends a message to all connections in the group.

**Expected Output:** Members of `room` receive the messages; non‑members do not.

**Why this output?** `Clients.Group(room)` targets only the connections that have been added to the group.

### Real‑World Cases

- **Chat rooms:** Each room is a group.
- **Multi‑tenant SaaS:** Each tenant is a group for tenant‑scoped notifications.
- **Live sports:** Each match is a group for score updates.

### References

- Manage users and groups in SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/groups
- `IGroupManager` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.signalr.igroupmanager

---

## 6. SignalR Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | SignalR best practices are guidelines for building secure, scalable, and reliable real‑time applications with SignalR. |
| **Technical Definition** | Best practices span **security** (authentication on the upgrade, `[Authorize]` on hubs, group access control), **scalability** (Redis backplane or Azure SignalR Service), **reconnection** (automatic reconnect, state recovery), **performance** (MessagePack, `SkipNegotiation` for WebSockets, avoiding long‑running hub methods), **state management** (hubs are transient), and **error handling** (`HubException` for client‑visible errors). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for SignalR—they help you avoid a hub that was anonymous in development and shipped that way, a scale‑out setup that loses messages, a reconnection that loses state, and a hub method that blocks the connection. |

### Purposes

- **To** secure real‑time endpoints against unauthorized access.
- **To** scale across multiple server instances without message loss.
- **To** handle disconnections and reconnections gracefully.
- **To** optimise performance for high‑throughput scenarios.
- **To** avoid common anti‑patterns (state in hubs, long‑running methods, missing auth).

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Authentication** | Apply `[Authorize]` on hubs; pass JWT in the `access_token` query string for WebSocket upgrades. | Ship a hub that was anonymous in development to production. |
| **Group security** | Authorize group joins (verify the user belongs to the room/tenant). | Allow arbitrary group joining without checks. |
| **Scaling** | Use a **Redis backplane** or **Azure SignalR Service** for multi‑instance deployments. | Assume in‑memory broadcast works across servers. |
| **Reconnection** | Implement `withAutomaticReconnect()` on clients; re‑join groups and re‑fetch state on reconnect. | Assume connections never drop. |
| **Hub state** | Keep hubs **stateless** (transient); use external storage. | Store state in hub instance fields. |
| **Long‑running work** | Offload to background services; use streaming for incremental results. | Block the hub method for seconds/minutes. |
| **Client methods** | Use **strongly‑typed hubs** (`Hub<TClient>`) for compile‑time safety. | Rely on magic strings for client method names. |
| **Errors** | Use `HubException` for client‑visible errors; log others. | Leak stack traces or internal details. |
| **Protocol** | Consider **MessagePack** for smaller payloads (~30–40% reduction). | Use JSON for high‑throughput binary data. |
| **Transport** | Prefer **WebSockets**; use `SkipNegotiation` when appropriate. | Force Long Polling when WebSockets are available. |

### Annotated Code Example (Best‑Practice Setup)

```csharp
// Server: authenticated, strongly-typed hub
[Authorize]
public class ChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string message)
    {
        var user = Context.User?.Identity?.Name ?? "Anonymous";
        await Clients.All.ReceiveMessage(user, message);
    }

    public async Task JoinRoom(string room)
    {
        // Authorization check: verify the user may join this room
        if (!await _authService.CanJoinRoomAsync(Context.UserIdentifier!, room))
        {
            throw new HubException("Not authorized to join this room.");
        }
        await Groups.AddToGroupAsync(Context.ConnectionId, room);
    }
}

// Program.cs: SignalR + Redis backplane
builder.Services.AddSignalR()
    .AddStackExchangeRedis("localhost:6379");   // Scale-out

// Client: automatic reconnect + state recovery
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect([0, 2000, 10000, 30000])   // Custom backoff
    .build();

connection.onreconnected(async (connectionId) => {
    await connection.invoke("JoinRoom", currentRoom);  // Re-join groups
    await refreshState();                              // Re-fetch state
});
```

**Why this follows best practices:**

- **`[Authorize]` on the hub** prevents anonymous access.
- **Group joins are authorized** —users cannot join rooms they are not allowed to.
- **The Redis backplane** enables scale‑out across instances.
- **Automatic reconnect** with a custom backoff schedule handles transient failures.
- **State recovery** re‑joins groups and re‑fetches data after reconnection.
- **`HubException`** is used for client‑visible authorization errors without leaking details.
- **Strongly‑typed hub** (`Hub<IChatClient>`) provides compile‑time checking.

### Real‑World Cases

- **Production chat:** Auth on upgrade, Redis backplane, reconnect logic.
- **Live dashboards:** MessagePack for high‑frequency updates, `SkipNegotiation` for WebSockets.
- **Multi‑tenant SaaS:** Group‑per‑tenant with authorization checks.

### References

- SignalR security considerations – https://learn.microsoft.com/en-us/aspnet/core/signalr/security
- SignalR scale‑out (Redis backplane, Azure SignalR Service) – https://learn.microsoft.com/en-us/aspnet/core/signalr/scale-out
- SignalR reconnection – https://learn.microsoft.com/en-us/aspnet/core/signalr/javascript-client#reconnect-clients
- SignalR performance – https://learn.microsoft.com/en-us/aspnet/core/signalr/performance
- ASP.NET Core SignalR Quick Reference (anti‑patterns) – https://github.com/claude-dev-suite/claude-dev-suite/blob/e10ab644ba43abe2abc923399226df4af7337214/skills/backend-frameworks/aspnet-signalr/SKILL.md
- SignalR (.NET 8–10) skill (best practices) – https://github.com/chrishuffman5/domain-expert/blob/main/plugins/api-realtime/skills/signalr/SKILL.md

---

## Summary Table: SignalR Core Concepts at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **SignalR** | Real‑time server‑push library | `AddSignalR()`, `MapHub<T>()` | Chat, notifications, live dashboards |
| **Real‑time communication** | Instant server‑to‑client updates | Hub methods + `Clients` targeting | Push‑based scenarios (not CRUD) |
| **Hubs** | Central communication pipeline | `Hub`, `Hub<TClient>`, lifecycle methods | Every SignalR app |
| **Clients** | Message targeting | `Clients.All`, `Caller`, `Group`, `User` | Precise audience selection |
| **Groups** | Named connection subsets | `Groups.AddToGroupAsync`, `Clients.Group` | Chat rooms, tenants, topics |
| **Best Practices** | Security, scaling, reliability | `[Authorize]`, Redis backplane, reconnect, stateless hubs | Every production SignalR deployment |

---

## References (Full List)

- Overview of ASP.NET Core SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction
- Use hubs in ASP.NET Core SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs
- Strongly‑typed hubs – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs#strongly-typed-hubs
- Client targeting – https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs#client-targeting
- `IHubClients` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.signalr.ihubclients
- Manage users and groups in SignalR – https://learn.microsoft.com/en-us/aspnet/core/signalr/groups
- `IGroupManager` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.signalr.igroupmanager
- SignalR transports – https://learn.microsoft.com/en-us/aspnet/core/signalr/transports
- SignalR security considerations – https://learn.microsoft.com/en-us/aspnet/core/signalr/security
- SignalR scale‑out – https://learn.microsoft.com/en-us/aspnet/core/signalr/scale-out
- SignalR reconnection – https://learn.microsoft.com/en-us/aspnet/core/signalr/javascript-client#reconnect-clients
- SignalR performance – https://learn.microsoft.com/en-us/aspnet/core/signalr/performance
- ASP.NET Core SignalR Quick Reference (GitHub) – https://github.com/claude-dev-suite/claude-dev-suite/blob/e10ab644ba43abe2abc923399226df4af7337214/skills/backend-frameworks/aspnet-signalr/SKILL.md
- SignalR (.NET 8–10) skill (GitHub) – https://github.com/chrishuffman5/domain-expert/blob/main/plugins/api-realtime/skills/signalr/SKILL.md
- SignalR authentication and scaling notes (GitHub) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/tree/df63d1347c197204d82516c55db92560c62a61a0/curriculum/week-11-signalr-and-realtime
- SignalR vs WebSocket – https://websocket.org/comparisons/signalr/