# ASP.NET Core gRPC: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **gRPC on .NET** (ASP.NET Core 6–10) and the **Grpc.AspNetCore** server package, **Grpc.Net.Client** client package, and **Grpc.Tools** code‑generation tooling. gRPC is a language‑agnostic, high‑performance RPC framework that uses **HTTP/2** for transport and **Protocol Buffers** (Protobuf) as its default interface definition language (IDL) and message format. It supports four communication patterns: **unary**, **server streaming**, **client streaming**, and **bidirectional streaming**. gRPC is ideal for **lightweight microservices**, **polyglot systems**, and **point‑to‑point real‑time streaming**, but it is **not a replacement for REST** in browser‑facing APIs without gRPC‑Web. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | gRPC is a modern, high‑performance, contract‑first Remote Procedure Call (RPC) framework that enables client and server applications to communicate transparently over HTTP/2 using strongly typed service definitions. |
| **Technical Definition** | gRPC uses **Protocol Buffers** as its IDL: services and messages are defined in `.proto` files, and language‑specific strongly typed servers and clients are generated via `Grpc.Tools`. On .NET, `Grpc.AspNetCore` hosts services, `Grpc.Net.Client` provides the client runtime, and `Grpc.Net.ClientFactory` integrates clients with `HttpClientFactory`. gRPC supports four method types—**unary**, **server streaming**, **client streaming**, and **bidirectional streaming**—and provides built‑in support for **deadlines**, **cancellation**, **metadata**, **interceptors**, **TLS**, and **authentication/authorization**. |
| **Beginner‑Friendly Explanation** | gRPC is the **“fast phone line”** between services. Instead of sending JSON over HTTP/1.1 like a REST API, gRPC sends compact binary messages over HTTP/2. You write a contract (`.proto` file) that says “this service has these methods, and they take these messages.” The tooling generates the server and client code for you, so calling a remote service feels like calling a local method. It is fast, strongly typed, and supports streaming—but it is not designed to be called directly from a browser without a bridge. |

### Key Characteristics

- **Contract‑first** – Services and messages are defined in `.proto` files; code is generated, not hand‑written.
- **HTTP/2‑native** – Multiplexing, flow control, header compression, and binary framing.
- **Binary serialization** – Protobuf is smaller and faster than JSON, reducing network usage.
- **Four method types** – Unary, server streaming, client streaming, and bidirectional streaming.
- **Strongly typed** – Generated clients and servers provide compile‑time checking and IntelliSense.
- **Cross‑platform and polyglot** – Tooling for C#, Java, Go, Python, C++, and more.
- **ASP.NET Core integration** – Full access to DI, logging, authentication, and authorization.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- `Grpc.AspNetCore` package (server), `Grpc.Net.Client` / `Grpc.Net.ClientFactory` (client), `Grpc.Tools` (code generation).
- Working knowledge of **HTTP/2**, **Protocol Buffers** basics, and **C# async/await**.
- Familiarity with **dependency injection** and **middleware** (for ASP.NET Core hosting).

### Related Programming Areas

- **Microservices** – gRPC is a primary inter‑service communication mechanism.
- **Protocol Buffers** – The IDL and serialization format.
- **ASP.NET Core** – Hosting, DI, logging, authentication, authorization.
- **Streaming** – Real‑time data flows between client and server.
- **REST/JSON** – The alternative for browser‑facing and public APIs.

### Core Concepts / Features

1. **gRPC** – The framework, architecture, and .NET tooling.
2. **Protocol Buffers** – The `.proto` syntax, messages, fields, and services.
3. **Services** – Implementing gRPC services on ASP.NET Core.
4. **Clients** – Creating and configuring gRPC clients.
5. **Streaming** – Unary, server, client, and bidirectional patterns.
6. **gRPC best practices** – Performance, security, and maintainability.

---

## 1. gRPC (The Framework)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | gRPC is a language‑agnostic, high‑performance RPC framework that uses HTTP/2 for transport and Protocol Buffers for service definition and serialization. |
| **Technical Definition** | gRPC is based on the idea of defining a **service** with methods that can be called remotely, along with their **parameters and return types**. By default, gRPC uses Protocol Buffers as the IDL. The gRPC framework generates client‑side **stubs** (proxies) and server‑side **base classes** from the `.proto` file. On .NET, `Grpc.AspNetCore` integrates gRPC services into the ASP.NET Core pipeline, and `Grpc.Net.Client` provides the HTTP/2‑based client transport. |
| **Beginner‑Friendly Explanation** | gRPC is the **“USB‑C cable”** between services. It is a standardized, efficient way for two programs to talk to each other. You define the “connector shape” in a `.proto` file, and the tooling builds both ends. Unlike REST, which sends text‑based JSON, gRPC sends compact binary data over HTTP/2, making it faster and more efficient—especially for internal service‑to‑service calls. |

### Purposes

- **To** provide **high‑performance, low‑latency** communication between services.
- **To** enforce **strongly typed contracts** between client and server via generated code.
- **To** support **streaming** (server, client, and bidirectional) for real‑time data flows.
- **To** enable **polyglot** systems where services are written in different languages.
- **To** integrate seamlessly with **ASP.NET Core** features (DI, logging, auth).

### Syntax Rules and Structure

#### The gRPC Architecture

```text
Client (generated stub) ──HTTP/2──▶ Server (generated base class)
        │                                   │
        ▼                                   ▼
   Protocol Buffers message           Service method implementation
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| **Client stub** | Generated proxy that serializes requests and deserializes responses. |
| **Server base class** | Generated abstract class that the developer implements. |
| **HTTP/2 transport** | Provides multiplexing, flow control, and header compression. |
| **Protocol Buffers** | Binary serialization for messages. |

#### The `.proto` Contract (Minimal Example)

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest { string name = 1; }
message HelloReply { string message = 1; }
```

#### General Syntax (Server Registration)

```csharp
builder.Services.AddGrpc();
app.MapGrpcService<GreeterService>();
```

**Syntax rules:**

- **`.proto` files are added to the project** via `<Protobuf Include="Protos\greet.proto" />`.
- **`Grpc.Tools`** generates C# types from the `.proto` files at build time.
- **Server services inherit from the generated `{Service}Base`** class and override methods.
- **`AddGrpc()`** registers gRPC services; **`MapGrpcService<T>()`** exposes them.

**Constraints and limitations:**

- **gRPC requires HTTP/2** – browser support is limited without gRPC‑Web.
- **Not ideal for public browser APIs** – JSON/REST is more compatible.
- **Message size limits** – default `MaxReceiveMessageSize` is 4 MB.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddGrpc();

var app = builder.Build();
app.MapGrpcService<GreeterService>();
app.Run();

// GreeterService.cs
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;
    public GreeterService(ILogger<GreeterService> logger) => _logger = logger;

    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        _logger.LogInformation("Saying hello to {Name}", request.Name);
        return Task.FromResult(new HelloReply { Message = $"Hello {request.Name}" });
    }
}
```

**Step‑by‑step explanation:**

1. `AddGrpc()` registers gRPC server services.
2. `MapGrpcService<GreeterService>()` exposes the service at the route defined in the `.proto` file.
3. `GreeterService` inherits from the generated `Greeter.GreeterBase` and overrides `SayHello`.
4. The `ServerCallContext` provides access to metadata, cancellation, and deadlines.

**Expected Output:** A gRPC client calling `SayHello` with `Name = "World"` receives `Message = "Hello World"`.

**Why this output?** The generated server base class handles HTTP/2 framing and Protobuf deserialization; the overridden method executes the business logic and returns a generated reply message.

### Real‑World Cases

- **Microservices:** Internal service‑to‑service calls where performance matters.
- **Polyglot systems:** Services written in C#, Go, and Python communicating via a shared `.proto`.
- **Real‑time streaming:** Server‑push notifications, telemetry, collaborative editing.

### References

- Overview for gRPC on .NET – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/grpc/index.md
- gRPC services with ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/grpc/aspnetcore.md
- gRPC core concepts – https://grpc.io/docs/what-is-grpc/core-concepts/

---

## 2. Protocol Buffers (The IDL)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Protocol Buffers is a language‑neutral, platform‑neutral, extensible mechanism for serializing structured data, defined by `.proto` files and used by gRPC as its IDL and message format. |
| **Technical Definition** | A `.proto` file declares a `syntax` (proto3), a `package`, `option` settings (e.g., `csharp_namespace`), `message` types (analogous to classes), `enum` types, and `service` definitions. Each field in a message has a **name**, a **type**, and a **unique field number** used in the binary wire format. Proto3 supports **scalar types** (`int32`, `string`, `bool`, etc.), **repeated** fields (lists), **map** fields (dictionaries), **oneof** fields (mutually exclusive), and **nested messages**. |
| **Beginner‑Friendly Explanation** | Protocol Buffers is the **“schema language”** for gRPC. It is how you describe the shape of your data. A `message` is like a C# class; a field is like a property. You give each field a number—this number is what travels over the wire, which is why it is compact. You also define `service` blocks that list the remote methods. The tooling turns all of this into C# classes and service bases. |

### Purposes

- **To** define **strongly typed, language‑agnostic** data contracts.
- **To** produce **compact binary** payloads that are smaller and faster than JSON.
- **To** enable **backward‑compatible schema evolution** through field numbers and `reserved`.
- **To** support **multiple data shapes**: scalars, lists, maps, enums, and unions (`oneof`).
- **To** serve as the **single source of truth** for service and message definitions.

### Syntax Rules and Structure

#### General Syntax (Message and Fields)

```protobuf
syntax = "proto3";
option csharp_namespace = "MyApp.Messages";

message Order {
  int32 id = 1;
  string customer_name = 2;
  repeated OrderItem items = 3;
  map<string, string> metadata = 4;
  OrderStatus status = 5;
  oneof payment {
    CardPayment card = 6;
    CashPayment cash = 7;
  }
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;
  PENDING = 1;
  SHIPPED = 2;
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `syntax = "proto3"` | Required first non‑comment line. |
| `option csharp_namespace` | C# namespace for generated code. |
| `message Order` | A data type with fields. |
| `int32 id = 1` | Field name, type, and **unique field number**. |
| `repeated OrderItem items` | A list. |
| `map<string, string> metadata` | A dictionary. |
| `oneof payment` | Only one of the fields can be set. |
| `enum OrderStatus` | Enum with a zero value (required first). |

#### Scalar Types

| Proto Type | C# Type | Notes |
|------------|---------|-------|
| `int32` | `int` | Variable‑length; use for small numbers. |
| `int64` | `long` | Variable‑length. |
| `string` | `string` | UTF‑8. |
| `bool` | `bool` | |
| `double` / `float` | `double` / `float` | |
| `bytes` | `ByteString` | Binary data. |

#### Field Number Rules

- **Field numbers must be unique within a message**.
- **Numbers 1–15 are encoded in one byte** —use them for frequently set fields.
- **Numbers 19,000–19,999 are reserved**.
- **Never reuse a field number** after deleting a field—use `reserved` to prevent accidental reuse.

#### General Syntax (Service Definition)

```protobuf
service OrderService {
  rpc GetOrder (GetOrderRequest) returns (Order);
  rpc ListOrders (ListOrdersRequest) returns (stream Order);
  rpc UploadOrders (stream Order) returns (UploadSummary);
  rpc StreamOrders (stream Order) returns (stream OrderEvent);
}
```

**Syntax rules:**

- **Each `rpc` declares a method** with a request type and a response type.
- **`stream`** before a type indicates a streaming parameter or return.
- **Services and messages are generated** as C# types by `Grpc.Tools`.

**Constraints and limitations:**

- **Changing a field number is equivalent to deleting and recreating the field** —never do this.
- **Enums must have a zero value** and conventionally name it `*_UNSPECIFIED`.
- **`oneof` fields share memory** —setting one clears the others.

### Annotated Complete Step‑by‑Step Code Example

```protobuf
// Protos/order.proto
syntax = "proto3";
option csharp_namespace = "MyApp.Orders";

message Order {
  int32 id = 1;
  string customer_name = 2;
  repeated OrderItem items = 3;
}

message OrderItem {
  string sku = 1;
  int32 quantity = 2;
}

service OrderService {
  rpc GetOrder (GetOrderRequest) returns (Order);
  rpc ListOrders (ListOrdersRequest) returns (stream Order);
}

message GetOrderRequest { int32 id = 1; }
message ListOrdersRequest { string customer_name = 1; }
```

**Step‑by‑step explanation:**

1. The `syntax` declares proto3.
2. `Order` has an `id`, `customer_name`, and a list of `OrderItem` messages.
3. `OrderService` defines `GetOrder` (unary) and `ListOrders` (server streaming).
4. `Grpc.Tools` generates `Order`, `OrderItem`, `GetOrderRequest`, `ListOrdersRequest`, and the `OrderService` base class/client.

**Expected Output:** Generated C# classes allow the server to implement `GetOrder` and `ListOrders`, and the client to call them with strongly typed messages.

**Why this output?** The `.proto` file is the contract; the tooling generates the C# types and service scaffolding at build time.

### Real‑World Cases

- **API contracts:** A shared `.proto` repository used by multiple services.
- **Backward‑compatible evolution:** Adding optional fields without breaking old clients.
- **Polyglot communication:** The same `.proto` generates C#, Java, and Go types.

### References

- Protocol Buffers Language Guide (proto3) – https://protobuf.dev/programming-guides/proto3/
- Create Protobuf messages for .NET apps – https://learn.microsoft.com/en-us/aspnet/core/grpc/protobuf

---

## 3. Services (Implementing gRPC Services)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A gRPC service is a class that inherits from the generated `{Service}Base` class and implements the remote methods defined in the `.proto` file. |
| **Technical Definition** | On ASP.NET Core, gRPC services are registered via `AddGrpc()` and exposed via `MapGrpcService<TService>()`. The service class overrides generated methods (e.g., `SayHello`, `ListOrders`) and receives a `ServerCallContext` that provides access to **metadata**, **deadline**, **cancellation token**, **peer information**, and **response trailers**. Services have full access to ASP.NET Core features: **dependency injection**, **logging**, **authentication**, **authorization**, and **interceptors**. |
| **Beginner‑Friendly Explanation** | A gRPC service is the **“implementation”** of the contract. The `.proto` file says “there is a method called `SayHello`.” The service class is where you write what `SayHello` actually does. It inherits from a generated base class, so you only override the methods you care about. The framework handles HTTP/2, serialization, and routing. |

### Purposes

- **To** implement the **business logic** behind each RPC method.
- **To** leverage **ASP.NET Core DI** for repositories, loggers, and other services.
- **To** access **request metadata** (headers, tokens) and **cancellation** via `ServerCallContext`.
- **To** control **response trailers**, **status codes**, and **error details**.
- **To** apply **authentication and authorization** through ASP.NET Core policies.

### Syntax Rules and Structure

#### General Syntax (Service Implementation)

```csharp
public class OrderService : OrderService.OrderServiceBase
{
    private readonly IOrderRepository _repo;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IOrderRepository repo, ILogger<OrderService> logger)
    {
        _repo = repo;
        _logger = logger;
    }

    public override async Task<Order> GetOrder(
        GetOrderRequest request, ServerCallContext context)
    {
        var order = await _repo.FindAsync(request.Id);
        if (order is null)
        {
            throw new RpcException(new Status(StatusCode.NotFound, "Order not found"));
        }
        return order;
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: OrderService.OrderServiceBase` | Inherits the generated base class. |
| `override` | Overrides the generated method. |
| `ServerCallContext` | Provides metadata, cancellation, deadline. |
| `RpcException` | Throws a gRPC status with a code and message. |

#### General Syntax (Registration)

```csharp
builder.Services.AddGrpc(options =>
{
    options.MaxReceiveMessageSize = 16 * 1024 * 1024; // 16 MB
    options.EnableDetailedErrors = false;
});
app.MapGrpcService<OrderService>();
```

**Syntax rules:**

- **Services must inherit from the generated `{Service}Base`**.
- **Method signatures must match the generated abstract methods** —request, response, and `ServerCallContext`.
- **`RpcException`** is the idiomatic way to return error statuses.
- **`context.CancellationToken`** should be passed to async operations.

**Constraints and limitations:**

- **`EnableDetailedErrors` should remain `false` in production** —it can leak sensitive information.
- **Default `MaxReceiveMessageSize` is 4 MB** —increase with care for large payloads.
- **Services are registered as Scoped by default** —do not capture them in singletons.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;

    public GreeterService(ILogger<GreeterService> logger) => _logger = logger;

    public override Task<HelloReply> SayHello(
        HelloRequest request, ServerCallContext context)
    {
        _logger.LogInformation("Greeting {Name}", request.Name);
        return Task.FromResult(new HelloReply
        {
            Message = $"Hello {request.Name}"
        });
    }

    public override async Task ListReplies(
        HelloRequest request,
        IServerStreamWriter<HelloReply> responseStream,
        ServerCallContext context)
    {
        for (var i = 0; i < 5; i++)
        {
            if (context.CancellationToken.IsCancellationRequested) break;
            await responseStream.WriteAsync(new HelloReply
            {
                Message = $"Reply {i} to {request.Name}"
            });
        }
    }
}
```

**Expected Output:** `SayHello` returns a single reply; `ListReplies` streams five replies to the client.

**Why this output?** The base class dispatches unary and streaming calls to the overridden methods; `IServerStreamWriter` is used for server streaming.

### Real‑World Cases

- **Order services:** `GetOrder`, `CreateOrder`, `ListOrders`.
- **Real‑time feeds:** Server streaming stock prices or notifications.
- **File transfer:** Client streaming chunks for uploads.

### References

- gRPC services with ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/grpc/aspnetcore.md
- Create gRPC services and methods – https://learn.microsoft.com/en-us/aspnet/core/grpc/services

---

## 4. Clients (Calling gRPC Services)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A gRPC client is a generated strongly typed proxy that translates local method calls into gRPC requests over a channel to a remote service. |
| **Technical Definition** | On .NET, clients are generated from `.proto` files as `{Service}Client` classes. A client is created from a **`GrpcChannel`** (`GrpcChannel.ForAddress(...)`), which represents a long‑lived connection to the server. The `Grpc.Net.ClientFactory` package integrates gRPC clients with `HttpClientFactory` via `AddGrpcClient<TClient>()`, providing centralized configuration (address, HTTP handler, retries, interceptors). Calls can be unary, server streaming, client streaming, or bidirectional; each accepts `CallOptions` for **deadline**, **cancellation token**, **headers**, and **credentials**. |
| **Beginner‑Friendly Explanation** | A gRPC client is the **“remote control”** for a service. The generated client class has methods that look like local C# methods—but under the hood, it serializes the request, sends it over HTTP/2, and deserializes the response. You create the client from a channel (the connection), and you can configure TLS, deadlines, and authentication on that channel. |

### Purposes

- **To** call remote gRPC services with **strongly typed** methods.
- **To** centralize **client configuration** (address, TLS, retries) via the client factory.
- **To** support **deadlines** and **cancellation** for reliable calls.
- **To** attach **metadata** (auth tokens, correlation IDs) to requests.
- **To** consume **streaming** responses or send streaming requests.

### Syntax Rules and Structure

#### General Syntax (Manual Client)

```csharp
using var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);
var reply = await client.SayHelloAsync(new HelloRequest { Name = "World" });
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GrpcChannel.ForAddress(...)` | Creates a long‑lived channel. |
| `new Greeter.GreeterClient(channel)` | Creates the generated client. |
| `SayHelloAsync(...)` | Calls the unary method. |

#### General Syntax (Client Factory — Recommended for ASP.NET Core)

```csharp
builder.Services.AddGrpcClient<Greeter.GreeterClient>(options =>
{
    options.Address = new Uri("https://localhost:5001");
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddGrpcClient<TClient>()` | Registers the generated client with DI. |
| `options.Address` | The service address. |
| Integration | Uses `HttpClientFactory` for handler lifetime and resilience. |

#### General Syntax (Deadline and Cancellation)

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
var reply = await client.SayHelloAsync(
    new HelloRequest { Name = "World" },
    deadline: DateTime.UtcNow.AddSeconds(5),
    cancellationToken: cts.Token);
```

**Syntax rules:**

- **Reuse channels** —they are long‑lived and expensive to create.
- **Always set a deadline** on RPCs to avoid hanging calls.
- **Pass `CancellationToken`** to propagate cancellation.
- **Use the client factory** in ASP.NET Core for centralized configuration.

**Constraints and limitations:**

- **Channels must match the server’s TLS configuration** —HTTPS to HTTPS; plaintext only in development.
- **Deadlines are propagated** to the server via the `grpc-timeout` header.
- **Client factory clients are transient** —do not dispose them manually; the factory manages handlers.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Program.cs
builder.Services.AddGrpcClient<Greeter.GreeterClient>(options =>
{
    options.Address = new Uri("https://localhost:5001");
});

// Usage in a service
public class GreetingService
{
    private readonly Greeter.GreeterClient _client;
    public GreetingService(Greeter.GreeterClient client) => _client = client;

    public async Task<string> GreetAsync(string name, CancellationToken ct)
    {
        var reply = await _client.SayHelloAsync(
            new HelloRequest { Name = name },
            deadline: DateTime.UtcNow.AddSeconds(5),
            cancellationToken: ct);
        return reply.Message;
    }
}
```

**Step‑by‑step explanation:**

1. `AddGrpcClient` registers the generated `GreeterClient` with DI.
2. The `GreetingService` receives the client via constructor injection.
3. `SayHelloAsync` is called with a 5‑second deadline and a cancellation token.

**Expected Output:** `GreetAsync` returns the server’s reply message or throws a deadline‑exceeded/cancellation exception.

**Why this output?** The client factory configures the channel and handler; the generated client serializes the request and sends it over HTTP/2.

### Real-World Cases

- **Microservice communication:** A service calls another service via a generated client.
- **API gateways:** A gateway aggregates data from multiple gRPC services.
- **Background workers:** Workers call gRPC services for data processing.

### References

- Call gRPC services with the .NET client – https://learn.microsoft.com/en-us/aspnet/core/grpc/client
- gRPC client factory integration in .NET – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/clientfactory.md
- Reliable gRPC services with deadlines and cancellation – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/deadlines.md

---

## 5. Streaming (Communication Patterns)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | gRPC streaming allows the client and server to exchange sequences of messages over a single RPC call, enabling real‑time, long‑lived data flows. |
| **Technical Definition** | gRPC defines four method types: **Unary** (1 request → 1 response), **Server streaming** (1 request → stream of responses), **Client streaming** (stream of requests → 1 response), and **Bidirectional streaming** (stream of requests ↔ stream of responses). Streaming is declared in the `.proto` file with the `stream` keyword. On the server, server streaming uses `IServerStreamWriter<T>`; client streaming receives an `IAsyncStreamReader<T>`; bidirectional streaming combines both. |
| **Beginner‑Friendly Explanation** | Streaming is the **“conveyor belt”** between client and server. Instead of one request and one response, either side can send a continuous flow of messages. **Server streaming** is like a news feed—one request, many updates. **Client streaming** is like uploading a file in chunks—many sends, one final confirmation. **Bidirectional** is like a phone call—both sides talk whenever they want. |

### Purposes

- **To** support **real‑time data flows** (feeds, notifications, telemetry).
- **To** handle **large payloads** by chunking them across multiple messages.
- **To** reduce **network overhead** by keeping a single connection open for many messages.
- **To** enable **interactive scenarios** (chat, collaborative editing, gaming).
- **To** implement **backpressure** through flow control.

### Syntax Rules and Structure

#### The Four Patterns

| Pattern | Request | Response | `.proto` Syntax | Server Signature |
|---------|---------|----------|-----------------|-------------------|
| **Unary** | 1 | 1 | `rpc M (Req) returns (Res);` | `Task<Res> M(Req, ServerCallContext)` |
| **Server streaming** | 1 | stream | `rpc M (Req) returns (stream Res);` | `Task M(Req, IServerStreamWriter<Res>, ServerCallContext)` |
| **Client streaming** | stream | 1 | `rpc M (stream Req) returns (Res);` | `Task<Res> M(IAsyncStreamReader<Req>, ServerCallContext)` |
| **Bidirectional** | stream | stream | `rpc M (stream Req) returns (stream Res);` | `Task M(IAsyncStreamReader<Req>, IServerStreamWriter<Res>, ServerCallContext)` |

#### General Syntax (Server Streaming)

```protobuf
rpc ListOrders (ListOrdersRequest) returns (stream Order);
```

```csharp
public override async Task ListOrders(
    ListOrdersRequest request,
    IServerStreamWriter<Order> responseStream,
    ServerCallContext context)
{
    await foreach (var order in _repo.StreamAsync(request.CustomerName))
    {
        if (context.CancellationToken.IsCancellationRequested) break;
        await responseStream.WriteAsync(order);
    }
}
```

#### General Syntax (Client Streaming)

```protobuf
rpc UploadOrders (stream Order) returns (UploadSummary);
```

```csharp
public override async Task<UploadSummary> UploadOrders(
    IAsyncStreamReader<Order> requestStream,
    ServerCallContext context)
{
    var count = 0;
    await foreach (var order in requestStream.ReadAllAsync(context.CancellationToken))
    {
        await _repo.SaveAsync(order);
        count++;
    }
    return new UploadSummary { Count = count };
}
```

#### General Syntax (Bidirectional)

```protobuf
rpc Chat (stream ChatMessage) returns (stream ChatMessage);
```

```csharp
public override async Task Chat(
    IAsyncStreamReader<ChatMessage> requestStream,
    IServerStreamWriter<ChatMessage> responseStream,
    ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync(context.CancellationToken))
    {
        await responseStream.WriteAsync(new ChatMessage
        {
            User = message.User,
            Text = $"Echo: {message.Text}"
        });
    }
}
```

**Syntax rules:**

- **`stream` before the type** declares a streaming parameter or return.
- **Server streaming** writes to `IServerStreamWriter<T>`.
- **Client streaming** reads from `IAsyncStreamReader<T>`.
- **Bidirectional** combines both.
- **Always respect `context.CancellationToken`** —client disconnects and deadlines cancel the stream.

**Constraints and limitations:**

- **Streaming RPCs are long‑lived** —deadlines are harder to set per message; implement custom logic.
- **Backpressure** must be managed —slow consumers can cause memory buildup.
- **Load balancing** is less effective for long‑lived streams; connection‑level balancing is required.
- **Not all clients/proxies support streaming** —check gateway and client capabilities.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Server: server streaming
public override async Task StreamPrices(
    PriceRequest request,
    IServerStreamWriter<PriceUpdate> responseStream,
    ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        var update = await _market.GetLatestAsync(request.Symbol);
        await responseStream.WriteAsync(update);
        await Task.Delay(1000, context.CancellationToken);
    }
}

// Client: consuming a server stream
using var call = client.StreamPrices(
    new PriceRequest { Symbol = "MSFT" },
    cancellationToken: ct);

await foreach (var update in call.ResponseStream.ReadAllAsync(ct))
{
    Console.WriteLine($"{update.Symbol}: {update.Price}");
}
```

**Step-by-step explanation:**

1. The server method writes updates to `IServerStreamWriter` until cancellation.
2. The client calls `StreamPrices` and reads from `ResponseStream` using `ReadAllAsync`.
3. Cancellation propagates from client to server via the cancellation token.

**Expected Output:** The client receives a continuous stream of price updates until it cancels.

**Why this output?** Server streaming keeps the HTTP/2 stream open and pushes messages as they become available.

### Real-World Cases

- **Live dashboards:** Server streaming metrics or prices.
- **File uploads:** Client streaming chunks with a final summary.
- **Chat/collaboration:** Bidirectional streaming for real‑time messages.
- **Telemetry:** Client streaming sensor readings to a collector.

### References

- Core concepts (streaming types) – https://grpc.io/docs/what-is-grpc/core-concepts/
- Streaming patterns and best practices – https://grpc.io/docs/languages/cpp/best_practices/
- gRPC streaming best practices – https://learn.microsoft.com/en-us/aspnet/core/grpc/performance

---

## 6. gRPC Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | gRPC best practices are guidelines for building secure, performant, and maintainable gRPC services and clients on .NET. |
| **Technical Definition** | Best practices span **channel and stub reuse**, **deadlines and cancellation**, **message size limits**, **compression**, **TLS and authentication**, **interceptors** for cross‑cutting concerns, **streaming discipline** (cancellation, backpressure, disposal), **schema evolution** (field numbers, `reserved`), and **observability** (logging, metrics, tracing). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for gRPC—they help you avoid resource leaks, hung calls, insecure endpoints, and breaking changes to your `.proto` contracts. |

### Purposes

- **To** ensure **reliable, low‑latency** communication.
- **To** protect **data in transit** and **authenticate/authorize** callers.
- **To** prevent **resource exhaustion** (message size, streams, connections).
- **To** maintain **backward compatibility** as the API evolves.
- **To** enable **observability** through logging, metrics, and tracing.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Channels** | Reuse `GrpcChannel` instances; use the client factory in ASP.NET Core. | Create a new channel per call—it is expensive. |
| **Deadlines** | Always set a deadline on RPCs. | Leave calls without a deadline—they can hang indefinitely. |
| **Cancellation** | Propagate `CancellationToken` to all async calls. | Ignore cancellation—resources leak and work continues. |
| **Message size** | Tune `MaxReceiveMessageSize`/`MaxSendMessageSize` deliberately. | Set them to `null` (unlimited) without understanding the memory risk. |
| **Compression** | Enable gzip for large or repetitive payloads (client must advertise `grpc-accept-encoding`). | Compress already‑compressed data. |
| **TLS** | Use HTTPS in production; configure `CallCredentials` only over TLS. | Use plaintext gRPC in production. |
| **Authentication** | Use JWT/OAuth tokens in metadata; apply `[Authorize]` policies. | Roll custom token validation in service methods. |
| **Interceptors** | Use interceptors for logging, auth, retries, metrics. | Duplicate cross‑cutting logic in every service method. |
| **Streaming** | Dispose streams when done; respect cancellation; manage backpressure. | Keep streams open indefinitely without cleanup. |
| **Schema evolution** | Use `reserved` for removed fields; never change field numbers. | Reuse field numbers—it corrupts the wire format. |
| **Errors** | Throw `RpcException` with appropriate `StatusCode`. | Return 200 with error details in the body. |
| **Observability** | Enable logging, metrics, and distributed tracing. | Ignore latency and error rates. |

### Annotated Code Example (Best‑Practice Configuration)

```csharp
// Server configuration
builder.Services.AddGrpc(options =>
{
    options.MaxReceiveMessageSize = 16 * 1024 * 1024;   // 16 MB
    options.EnableDetailedErrors = false;                // Never leak details
    options.Interceptors.Add<LoggingInterceptor>();      // Cross-cutting
    options.ResponseCompressionAlgorithm = "gzip";
});

// Client factory with deadline and auth interceptor
builder.Services
    .AddGrpcClient<OrderService.OrderServiceClient>(options =>
    {
        options.Address = new Uri("https://orders:5001");
    })
    .AddInterceptor<AuthInterceptor>()
    .ConfigureChannel(channel =>
    {
        channel.HttpHandler = new SocketsHttpHandler
        {
            EnableMultipleHttp2Connections = true
        };
    });

// Service method with deadline and cancellation
public override async Task<Order> GetOrder(
    GetOrderRequest request, ServerCallContext context)
{
    using var cts = CancellationTokenSource.CreateLinkedTokenSource(
        context.CancellationToken);
    cts.CancelAfter(TimeSpan.FromSeconds(10));   // Custom deadline
    var order = await _repo.FindAsync(request.Id, cts.Token);
    return order ?? throw new RpcException(
        new Status(StatusCode.NotFound, "Order not found"));
}
```

**Why this follows best practices:**

- **Message size limits** are explicitly set (16 MB) rather than unlimited.
- **Detailed errors are disabled** in production.
- **Interceptors** handle logging and authentication without duplicating code.
- **The client factory** centralizes address, interceptors, and HTTP handler configuration.
- **Deadlines and cancellation** are propagated to repository calls.
- **`RpcException`** with `StatusCode.NotFound` is the idiomatic error pattern.

### Real-World Cases

- **Microservices at scale:** Channel reuse, deadlines, and interceptors are essential.
- **Security‑sensitive services:** TLS + JWT + `[Authorize]` policies.
- **High‑throughput streaming:** Backpressure, cancellation, and compression tuned.

### References

- gRPC performance best practices – https://learn.microsoft.com/en-us/aspnet/core/grpc/performance
- gRPC configuration (message sizes, compression, interceptors) – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/grpc/configuration.md
- Security in gRPC applications – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/security.md
- gRPC interceptors on .NET – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/interceptors.md
- Reliable gRPC services with deadlines and cancellation – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/deadlines.md
- Best Practices (grpc.io) – https://grpc.io/docs/languages/cpp/best_practices/

---

## Summary Table: gRPC Core Concepts at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **gRPC** | High‑performance RPC framework | HTTP/2 + Protobuf; `Grpc.AspNetCore` / `Grpc.Net.Client` | Microservices, polyglot systems, real‑time streaming |
| **Protocol Buffers** | IDL and binary serialization | `.proto` files; `syntax = "proto3"`; messages and services | Every gRPC contract |
| **Services** | Server‑side implementation | Inherit `{Service}Base`; `MapGrpcService<T>()` | Implementing RPC methods on ASP.NET Core |
| **Clients** | Client‑side proxy | `GrpcChannel`; `AddGrpcClient<T>()` | Calling gRPC services from .NET |
| **Streaming** | Long‑lived message flows | `stream` keyword; `IServerStreamWriter` / `IAsyncStreamReader` | Real‑time feeds, uploads, chat, telemetry |
| **Best Practices** | Reliable, secure, maintainable gRPC | Deadlines, cancellation, TLS, interceptors, schema evolution | Every production gRPC deployment |

---

## References (Full List)

- Overview for gRPC on .NET – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/grpc/index.md
- gRPC services with ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/grpc/aspnetcore.md
- gRPC core concepts, architecture and lifecycle – https://grpc.io/docs/what-is-grpc/core-concepts/
- Protocol Buffers Language Guide (proto3) – https://protobuf.dev/programming-guides/proto3/
- Create Protobuf messages for .NET apps – https://learn.microsoft.com/en-us/aspnet/core/grpc/protobuf
- gRPC services with C# – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/services.md
- Call gRPC services with the .NET client – https://learn.microsoft.com/en-us/aspnet/core/grpc/client
- gRPC client factory integration in .NET – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/clientfactory.md
- gRPC for .NET configuration – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/grpc/configuration.md
- Reliable gRPC services with deadlines and cancellation – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/deadlines.md
- Security in gRPC applications – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/security.md
- gRPC interceptors on .NET – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/grpc/interceptors.md
- gRPC performance best practices – https://learn.microsoft.com/en-us/aspnet/core/grpc/performance
- Best Practices (grpc.io) – https://grpc.io/docs/languages/cpp/best_practices/
- Streaming patterns and best practices – https://github.com/jander99/api-design-guide/blob/main/guides/api-design/grpc/streaming-patterns.md
- Protobuf Language Specification (Proto3) – https://protobuf.dev/programming-guides/proto3/