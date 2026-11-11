# C# SignalR — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**SignalR** is an open-source ASP.NET Core library that adds **real-time, bi-directional communication** between a server and connected clients. Unlike traditional HTTP request/response, SignalR maintains a persistent connection so the server can **push** messages to clients instantly, and clients can invoke server methods—without polling. SignalR abstracts the underlying transport (WebSockets, Server-Sent Events, Long Polling, or a custom transport) and falls back gracefully based on the client and server capabilities.

**Technical Definition**
SignalR is a framework for building real-time web functionality. It uses **Hubs**—high-level pipelines that allow the server and client to call methods on each other directly. On the server, a `Hub` derives from `Microsoft.AspNetCore.SignalR.Hub` or `Hub<T>`. On the client, a `HubConnection` (from `Microsoft.AspNetCore.SignalR.Client`) manages the connection, invokes server methods, and handles server-to-client calls. Transports are negotiated via HTTP: the server and client agree on **WebSockets** (preferred), **Server-Sent Events**, or **Long Polling**. SignalR manages connection lifetime, reconnection, message framing (JSON or MessagePack), and **groups** (named sets of connections). Scale-out requires a **backplane** (Azure SignalR Service, Redis) or **sticky sessions**.

**Beginner-Friendly Explanation**
SignalR is **real-time chat plumbing for your app**. Normally, a web browser asks the server for data and waits for the reply. With SignalR, the server can **push** a message to every connected browser the instant something happens—like a chat message, a stock price tick, or a live notification. The server-side unit that handles real-time calls is called a **Hub**. Clients connect to the Hub, call its methods, and receive calls from it. SignalR figures out the best transport (usually WebSockets) and handles the messy details of keeping the connection alive.

---

### Key Characteristics

- **Bi-directional communication** — Server can call client methods and vice versa.
- **Automatic transport negotiation** — WebSockets, Server-Sent Events, Long Polling, or custom transports.
- **Hubs** — High-level API for RPC-style communication.
- **Groups** — Named sets of connections for targeted messaging (e.g., chat rooms, user sessions).
- **Strongly typed hubs** — `Hub<T>` provides compile-time type safety for client methods.
- **Multiple protocols** — JSON (default) and MessagePack.
- **Scale-out support** — Azure SignalR Service, Redis backplane, or sticky sessions.
- **Reconnection and lifecycle events** — `Reconnecting`, `Reconnected`, `Closed` events on the client.

---

### Prerequisites

- Basic understanding of **ASP.NET Core** (`Program.cs`, middleware, DI).
- Familiarity with **async/await** and `Task`.
- Knowledge of **WebSockets** concepts (helpful but not required).
- Awareness of **HTTP** and the difference between polling and pushing.
- Understanding of **C# generics** for strongly typed hubs.

---

### Related Programming Areas

- **Chat applications** — Real-time messaging between users.
- **Live dashboards** — Push updates to connected clients.
- **Notifications** — Server-initiated alerts.
- **Collaborative editing** — Real-time document changes.
- **Gaming** — Real-time state synchronization.

---

### Core Concepts / Features

1. SignalR (Overarching Concept)
2. Real-Time Communication
3. Hubs
4. Clients
5. Groups
6. SignalR Best Practices

---

## 1. SignalR (Overarching Concept)

### Definitions

**Core Definition:** SignalR is a library for adding real-time, bi-directional communication to ASP.NET Core applications.

**Technical Definition:** SignalR provides a persistent connection between server and client. Server-side `Hub` classes expose methods that clients can call; the server can call methods on connected clients via `IHubContext<T>` or `Clients`/`Groups` properties. Transports are negotiated automatically. The library is extensible via protocols (JSON, MessagePack) and scale-out backplanes.

**Beginner-Friendly Explanation:** SignalR is **the real-time channel between your server and its clients**. It lets the server push updates without the client asking.

---

### Purposes

- To **push data** from server to clients in real time.
- To allow **clients to call server methods** without traditional REST endpoints.
- To **abstract transport details** (WebSockets, SSE, Long Polling).
- To support **scale-out** across multiple servers.
- To simplify **bi-directional RPC** between server and client.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR();

var app = builder.Build();

app.MapHub<ChatHub>("/chat");

app.Run();
```

```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message) =>
        await Clients.All.SendAsync("ReceiveMessage", user, message);
}
```

**Component breakdown**

- `AddSignalR()` — Registers SignalR services.
- `MapHub<T>(path)` — Maps a hub to a URL.
- `Hub` — Base class for hubs.
- `Clients.All.SendAsync(...)` — Sends a message to all clients.

#### Syntax Rules

- Hubs are registered with `MapHub<T>()`.
- Hub methods can be synchronous or asynchronous; async is preferred.
- Clients invoke hub methods by name.

#### Constraints and Limitations

- SignalR requires a persistent connection; serverless environments need Azure SignalR Service.
- Scale-out requires a backplane or sticky sessions.

---

### Annotated Code Examples and Expected Outputs

#### Example: Minimal Chat Hub

```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message) =>
        await Clients.All.SendAsync("ReceiveMessage", user, message);
}
```

**Expected Output (client-side event)**
```
ReceiveMessage: { user: "Alice", message: "Hello" }
```

**Why this output occurs:** The server invokes the client method `ReceiveMessage` on all connected clients.

---

### Real-World Cases

- **Chat apps** — Users exchange messages instantly.
- **Live dashboards** — Metrics update without refresh.
- **Notification systems** — Server pushes alerts to connected browsers.

---

### References

- Introduction to ASP.NET Core SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction

---

## 2. Real-Time Communication

### Definitions

**Core Definition:** Real-time communication is the ability to exchange data between server and clients with minimal delay, typically via persistent connections and push-based messaging.

**Technical Definition:** SignalR uses **WebSockets** as the preferred transport for full-duplex communication. If WebSockets are unavailable, it falls back to **Server-Sent Events** (server-to-client push) or **Long Polling** (client polls for updates). The transport is negotiated via HTTP. Messages are framed and can be serialized as JSON or MessagePack.

**Beginner-Friendly Explanation:** Real-time communication means **the server can talk to the client at any moment**, not just when the client asks.

---

### Purposes

- To **eliminate polling** and reduce latency.
- To **push updates** instantly.
- To support **live collaboration**.
- To **reduce server load** compared to frequent polling.

---

### Syntax Rules and Structure (Transport Negotiation)

| Transport | Direction | When Used |
|---|---|---|
| **WebSockets** | Full-duplex | Preferred |
| **Server-Sent Events** | Server → Client | WebSockets unavailable |
| **Long Polling** | Client ↔ Server | Fallback |

**Component breakdown**

- **WebSockets** — Full-duplex persistent connection.
- **SSE** — Server-to-client streaming.
- **Long Polling** — Client holds request open until data is available.

#### Syntax Rules

- SignalR negotiates the transport automatically.
- The client library handles reconnection and transport fallback.
- Message protocol (JSON or MessagePack) is configured at startup.

#### Constraints and Limitations

- WebSockets require proxy support in some environments.
- Long Polling has higher overhead.

---

### Annotated Code Examples and Expected Outputs

#### Example: Transport Selection on the Client

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chat", { transport: signalR.HttpTransportType.WebSockets })
    .build();
```

**Expected Output (conceptual)**
```
// The client forces WebSockets; if unavailable, the connection fails.
```

**Why this pattern is used:** Explicit transport selection can be useful for diagnostics or environments with known constraints.

---

### Real-World Cases

- **Live chat** — Instant messages.
- **Stock tickers** — Real-time price updates.
- **Multiplayer games** — Real-time state sync.

---

### References

- SignalR transports — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction#transports

---

## 3. Hubs

### Definitions

**Core Definition:** A Hub is a high-level SignalR class that allows the server and client to invoke methods on each other.

**Technical Definition:** Hubs derive from `Microsoft.AspNetCore.SignalR.Hub` or `Hub<T>`. Public methods on the Hub can be invoked by clients. The Hub exposes properties (`Clients`, `Groups`, `Context`) for sending messages. Lifecycle events (`OnConnectedAsync`, `OnDisconnectedAsync`) allow per-connection logic. Hubs are transient per connection.

**Beginner-Friendly Explanation:** A Hub is **the server-side controller for real-time calls**. Clients call its methods; the server can call back.

---

### Purposes

- To **expose server methods** to clients.
- To **send messages** to clients or groups.
- To handle **connection lifecycle**.
- To provide **RPC-style** communication.

---

### Syntax Rules and Structure

```csharp
public class ChatHub : Hub
{
    public override Task OnConnectedAsync() =>
        Clients.All.SendAsync("UserConnected", Context.ConnectionId);

    public async Task SendMessage(string user, string msg) =>
        await Clients.All.SendAsync("ReceiveMessage", user, msg);
}
```

**Component breakdown**

- `Hub` — Base class.
- `OnConnectedAsync` / `OnDisconnectedAsync` — Lifecycle.
- `Clients` — Sender for connected clients.
- `Groups` — Sender for groups.
- `Context` — Connection metadata.

#### Syntax Rules

- Public methods are invokable by clients.
- Lifecycle methods can be overridden.
- Hubs should be **stateless** (per-connection state in `Context.Items`).

#### Constraints and Limitations

- Hub instances are per-connection; do not store long-lived state in fields.
- Methods should be idempotent and safe to retry.

---

### Annotated Code Examples and Expected Outputs

#### Example: Hub with Connection Tracking

```csharp
public class PresenceHub : Hub
{
    private static readonly Dictionary<string, string> _users = new();

    public override async Task OnConnectedAsync()
    {
        _users[Context.ConnectionId] = "anonymous";
        await Clients.All.SendAsync("UserJoined", Context.ConnectionId);
    }

    public override async Task OnDisconnectedAsync(Exception? ex)
    {
        _users.Remove(Context.ConnectionId);
        await Clients.All.SendAsync("UserLeft", Context.ConnectionId);
    }
}
```

**Expected Output (conceptual)**
```
// When a client connects, all clients receive UserJoined.
// When disconnected, UserLeft is broadcast.
```

**Why this pattern is used:** Lifecycle methods let the server track presence and notify other clients.

---

### Real-World Cases

- **Chat rooms** — Broadcast messages.
- **Presence** — Track online users.
- **Notifications** — Push alerts.

---

### References

- Hubs — SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs

---

## 4. Clients

### Definitions

**Core Definition:** Clients are the connected applications that receive messages from a Hub or call its methods.

**Technical Definition:** The server sends messages via `Clients` (`All`, `Caller`, `Others`, `AllExcept`, `Client(connectionId)`, `Clients(IReadOnlyList<string>)`). Clients use `HubConnection` (`Microsoft.AspNetCore.SignalR.Client`) to invoke hub methods (`InvokeAsync`, `SendAsync`) and register handlers (`On<T>`). Client libraries exist for JavaScript, .NET, Java, and Python.

**Beginner-Friendly Explanation:** Clients are **the people on the other end of the phone line**—the browser, mobile app, or desktop app connected to the Hub.

---

### Purposes

- To **receive messages** from the server.
- To **call hub methods**.
- To **manage connection lifecycle**.
- To **handle reconnection** automatically.

---

### Syntax Rules and Structure

```csharp
// Server
await Clients.All.SendAsync("ReceiveMessage", user, message);
await Clients.Caller.SendAsync("Echo", message);
await Clients.Others.SendAsync("SomeoneSpoke", user, message);
await Clients.Client(Context.ConnectionId).SendAsync("Welcome", user);
```

```csharp
// .NET client
var connection = new HubConnectionBuilder()
    .WithUrl("https://localhost:5001/chat")
    .WithAutomaticReconnect()
    .Build();

connection.On<string, string>("ReceiveMessage", (user, message) =>
    Console.WriteLine($"{user}: {message}"));

await connection.StartAsync();
await connection.InvokeAsync("SendMessage", "Alice", "Hello");
```

**Component breakdown**

- `Clients.All` — All connected clients.
- `Clients.Caller` — The calling client.
- `Clients.Others` — All except the caller.
- `Clients.Client(id)` — Specific connection.
- `HubConnection` — Client connection object.
- `On<T>` — Client handler for server messages.
- `InvokeAsync` — Call server method and wait for result.
- `SendAsync` — Fire-and-forget server call.

#### Syntax Rules

- `InvokeAsync` awaits the server’s response; `SendAsync` does not.
- `On` registers handlers for server-to-client calls.
- `WithAutomaticReconnect()` enables reconnection.

#### Constraints and Limitations

- `InvokeAsync` requires the hub method to return a value or `Task`.
- Client handlers must be registered before starting the connection.

---

### Annotated Code Examples and Expected Outputs

#### Example: .NET Client

```csharp
using Microsoft.AspNetCore.SignalR.Client;

var connection = new HubConnectionBuilder()
    .WithUrl("https://localhost:5001/chat")
    .WithAutomaticReconnect()
    .Build();

connection.On<string, string>("ReceiveMessage", (user, msg) =>
    Console.WriteLine($"{user}: {msg}"));

await connection.StartAsync();
await connection.InvokeAsync("SendMessage", "Alice", "Hello");
```

**Expected Output (console)**
```
Alice: Hello
```

**Why this output occurs:** The client registers a handler, starts the connection, invokes the hub method, and receives the broadcast via the handler.

---

### Real-World Cases

- **Browser apps** — JavaScript client for chat.
- **Mobile apps** — .NET MAUI or native clients.
- **Desktop apps** — WPF/WinForms clients.

---

### References

- ASP.NET Core SignalR .NET client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/dotnet-client
- ASP.NET Core SignalR JavaScript client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/javascript-client

---

## 5. Groups

### Definitions

**Core Definition:** Groups are **named sets of connections** that allow targeted messaging (e.g., chat rooms, user sessions).

**Technical Definition:** `Groups` on the Hub allows adding and removing connections to named groups (`AddToGroupAsync`, `RemoveFromGroupAsync`). Messages sent via `Clients.Group(name)` reach all connections in that group. Connections can belong to multiple groups. Groups are not persisted; they are managed by the SignalR app.

**Beginner-Friendly Explanation:** Groups are **chat rooms**. A connection can join multiple rooms, and messages sent to a room reach only its members.

---

### Purposes

- To **target messages** to a subset of clients.
- To implement **chat rooms** and **channels**.
- To scope notifications to **user sessions** or **tenant groups**.
- To **reduce bandwidth** by avoiding broadcast to all clients.

---

### Syntax Rules and Structure

```csharp
public class ChatHub : Hub
{
    public Task JoinRoom(string room) => Groups.AddToGroupAsync(Context.ConnectionId, room);
    public Task LeaveRoom(string room) => Groups.RemoveFromGroupAsync(Context.ConnectionId, room);
    public Task SendToRoom(string room, string message) =>
        Clients.Group(room).SendAsync("ReceiveMessage", message);
}
```

**Component breakdown**

- `AddToGroupAsync` — Adds the connection to a group.
- `RemoveFromGroupAsync` — Removes the connection.
- `Clients.Group(name)` — Sends to a group.
- `Clients.OthersInGroup(name)` — Sends to others in the group.

#### Syntax Rules

- Group names are arbitrary strings.
- Connections are removed from groups automatically on disconnect.
- A connection can belong to multiple groups.

#### Constraints and Limitations

- Groups are not persisted across server restarts.
- Group membership is per-connection, not per-user.

---

### Annotated Code Examples and Expected Outputs

#### Example: Join and Send to Room

```csharp
public async Task SendToRoom(string room, string user, string message)
{
    await Groups.AddToGroupAsync(Context.ConnectionId, room);
    await Clients.Group(room).SendAsync("ReceiveMessage", user, message);
}
```

**Expected Output (clients in the room)**
```
ReceiveMessage: { user: "Alice", message: "Hello room" }
```

**Why this pattern is used:** Only clients in the specified group receive the message.

---

### Real-World Cases

- **Chat rooms** — `Clients.Group("room-1")`.
- **Live sports** — Subscribers to a match.
- **Multi-tenant apps** — Tenant-specific groups.

---

### References

- Manage users and groups in SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/groups

---

## 6. SignalR Best Practices

### Definitions

**Core Definition:** SignalR best practices are the guidelines for building scalable, reliable, and secure real-time applications.

**Technical Definition:** Best practices derive from Microsoft's SignalR guidance: **use async hub methods**; **keep hubs stateless** (per-connection state in `Context.Items`); **use groups** for targeted messaging; **use strongly typed hubs** for compile-time safety; **avoid blocking calls**; **configure automatic reconnection** on clients; **scale out** with Azure SignalR Service or a Redis backplane; **enable MessagePack** for performance; **secure connections** with authentication and authorization; and **test** hubs and clients.

**Beginner-Friendly Explanation:** The best practices are: **keep hubs small**, **use groups**, **use async**, **secure the connection**, **enable reconnection**, and **scale out with a backplane**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Async hub methods
public async Task SendMessage(string msg) => await Clients.All.SendAsync("ReceiveMessage", msg);

// 2. Stateless hubs
public override Task OnConnectedAsync()
{
    Context.Items["userId"] = Context.UserIdentifier;
    return base.OnConnectedAsync();
}

// 3. Groups for targeted messaging
await Groups.AddToGroupAsync(Context.ConnectionId, room);

// 4. Strongly typed hubs
public class ChatHub : Hub<IChatClient>
{
    public Task Send(string msg) => Clients.All.ReceiveMessage(msg);
}

// 5. Client reconnection
var connection = new HubConnectionBuilder()
    .WithUrl("/chat")
    .WithAutomaticReconnect()
    .Build();

// 6. Scale-out backplane
services.AddSignalR().AddStackExchangeRedis("redis:6379");

// 7. Secure connections
app.MapHub<ChatHub>("/chat").RequireAuthorization();
```

**Component breakdown**

- **Async methods** — Avoid blocking threads.
- **Stateless hubs** — Per-connection state in `Context.Items`.
- **Groups** — Targeted messaging.
- **Strongly typed hubs** — `Hub<T>` for compile-time safety.
- **Reconnection** — `WithAutomaticReconnect` on clients.
- **Backplane** — Redis or Azure SignalR Service for scale-out.
- **Security** — `RequireAuthorization()` on hubs.

#### Rules and Constraints

- **Do** use async hub methods.
- **Do** keep hubs stateless.
- **Do** use groups for targeted messaging.
- **Do** use strongly typed hubs.
- **Do** enable automatic reconnection on clients.
- **Do** scale out with a backplane.
- **Do not** block on async calls in hubs.
- **Do not** store per-connection state in hub fields.
- **Do not** expose hubs without authentication.

---

### Annotated Code Examples and Expected Outputs

#### Example: Strongly Typed Hub with Authorization

```csharp
public interface IChatClient
{
    Task ReceiveMessage(string user, string message);
}

[Authorize]
public class ChatHub : Hub<IChatClient>
{
    public Task SendMessage(string user, string message) =>
        Clients.All.ReceiveMessage(user, message);
}

app.MapHub<ChatHub>("/chat");
```

**Expected Output (conceptual)**
```
// Unauthenticated clients receive 401; authenticated clients receive strongly typed messages.
```

**Why this pattern is used:** `Hub<T>` provides compile-time safety for client method names, and `[Authorize]` restricts access.

---

### Real-World Cases

- **Enterprise chat** — Authenticated, group-based messaging.
- **Live dashboards** — Scaled-out with Redis backplane.
- **Mobile apps** — Automatic reconnection on network changes.

---

### References

- SignalR best practices — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction
- Hubs — SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs
- Groups — SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/groups

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| ASP.NET SignalR (Framework) | **Legacy**; replaced by ASP.NET Core SignalR. |
| MessagePack protocol | **Optional**; requires `Microsoft.AspNetCore.SignalR.Protocols.MessagePack`. |
| Azure SignalR Service | **Recommended** for serverless and large scale-out. |
| Redis backplane | **Recommended** for self-hosted scale-out. |
| Sticky sessions | **Required** without a backplane. |
| WebSockets | **Preferred transport**; fallback to SSE/Long Polling. |
| Strongly typed hubs (`Hub<T>`) | **Recommended** for compile-time safety. |

---

## References (Consolidated)

- Introduction to ASP.NET Core SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction
- Hubs — SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs
- ASP.NET Core SignalR .NET client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/dotnet-client
- ASP.NET Core SignalR JavaScript client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/javascript-client
- Manage users and groups in SignalR — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/groups
- SignalR transports — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction#transports
- SignalR best practices — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction

---