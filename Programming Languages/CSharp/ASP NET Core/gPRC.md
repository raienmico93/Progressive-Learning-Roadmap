# C# gRPC — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**gRPC** (Google Remote Procedure Call) is a **high-performance, open-source, contract-first RPC framework** that runs over HTTP/2. In .NET, gRPC services are defined using **Protocol Buffers** (`.proto` files), and the framework generates strongly typed server stubs and client proxies. gRPC is designed for **low-latency, high-throughput** service-to-service communication, making it the recommended choice for **microservices** and **internal APIs** in .NET.

**Technical Definition**
gRPC is a language-agnostic RPC framework that uses **Protocol Buffers** as its Interface Definition Language (IDL) and message format. It runs over **HTTP/2**, which provides multiplexing, header compression, and bi-directional streaming. In .NET, gRPC is implemented via **`Grpc.AspNetCore`** (server) and **`Grpc.Net.Client`** (client). A `.proto` file defines `service` contracts and `message` types; the `Grpc.Tools` package generates C# base classes (`*Base`), client stubs (`*Client`), and message types at build time. gRPC supports four communication patterns: **unary**, **server streaming**, **client streaming**, and **bi-directional streaming**. It uses **binary serialization** (Protocol Buffers) rather than text-based JSON, yielding smaller payloads and faster serialization.

**Beginner-Friendly Explanation**
gRPC is **a fast phone line between services**. Instead of one service sending an HTTP request and waiting for a JSON reply, two services agree on a **contract** written in a `.proto` file, and the framework generates all the code for them to call each other as if they were local methods. Because it runs over HTTP/2 and uses a compact binary format, gRPC is **much faster and smaller** than REST/JSON for service-to-service communication. It is the standard for microservices in .NET, but it is not ideal for browser clients or public APIs where human-readable JSON and broad compatibility matter.

---

### Key Characteristics

- **Contract-first** — `.proto` files define services and messages; code is generated.
- **HTTP/2-based** — Multiplexing, header compression, and streaming.
- **Binary serialization** — Protocol Buffers produce smaller, faster payloads than JSON.
- **Strongly typed** — Generated C# classes and interfaces.
- **Four RPC patterns** — Unary, server streaming, client streaming, and bi-directional streaming.
- **Cross-language** — Clients and servers can be written in any supported language.
- **Code generation** — `Grpc.Tools` generates stubs and message types at build time.
- **Not browser-native** — Requires **gRPC-Web** or **Connect** for browser clients.

---

### Prerequisites

- Basic understanding of **ASP.NET Core** (`Program.cs`, DI, middleware).
- Familiarity with **HTTP/2** concepts.
- Knowledge of **Protocol Buffers** syntax (`.proto`).
- Awareness of **async/await** and `IAsyncStreamReader<T>` / `IServerStreamWriter<T>`.
- Understanding of **service-to-service communication** in microservices.

---

### Related Programming Areas

- **Microservices** — gRPC is the recommended inter-service communication protocol in .NET .
- **Protocol Buffers** — The IDL and serialization format.
- **HTTP/2** — The transport layer.
- **gRPC-Web** — Browser-compatible gRPC.
- **Connect** — An alternative RPC protocol compatible with gRPC.

---

### Core Concepts / Features

1. gRPC (Overarching Concept)
2. Protocol Buffers
3. Services
4. Clients
5. Streaming
6. gRPC Best Practices

---

## 1. gRPC (Overarching Concept)

### Definitions

**Core Definition:** gRPC is a high-performance RPC framework that uses HTTP/2 and Protocol Buffers for efficient service-to-service communication.

**Technical Definition:** gRPC defines services via `.proto` files. The `Grpc.Tools` package generates C# base classes and clients. The server registers services with `MapGrpcService<T>()`; clients create `GrpcChannel` instances and call strongly typed clients. gRPC supports unary, server-streaming, client-streaming, and bi-directional-streaming calls. It is designed for **internal, high-throughput, low-latency** communication and is **not a drop-in replacement for REST** in all scenarios.

**Beginner-Friendly Explanation:** gRPC is **a fast RPC system for microservices**. You write a `.proto` contract, and the framework generates client and server code.

---

### Purposes

- To provide **fast, compact** inter-service communication.
- To enforce **strongly typed contracts** between services.
- To support **streaming** natively.
- To enable **cross-language** communication.
- To reduce **bandwidth and CPU** compared to JSON/REST.

---

### Syntax Rules and Structure

```protobuf
// greet.proto
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest { string name = 1; }
message HelloReply { string message = 1; }
```

```csharp
// Program.cs
builder.Services.AddGrpc();
var app = builder.Build();
app.MapGrpcService<GreeterService>();
```

**Component breakdown**

- `service` — Defines the RPC contract.
- `rpc` — A method on the service.
- `message` — A structured data type.
- `AddGrpc()` — Registers gRPC services.
- `MapGrpcService<T>()` — Maps a service implementation.

#### Syntax Rules

- `.proto` files use `syntax = "proto3"`.
- Services and messages are defined in the same file or imported.
- Generated code is produced at build time by `Grpc.Tools`.

#### Constraints and Limitations

- **Not browser-native**; requires gRPC-Web or Connect for browsers.
- **Requires HTTP/2**; some proxies may need configuration.
- **Not human-readable**; binary payloads are harder to debug than JSON.

---

### Annotated Code Examples and Expected Outputs

#### Example: Unary gRPC Service

```protobuf
syntax = "proto3";
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}
message HelloRequest { string name = 1; }
message HelloReply { string message = 1; }
```

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        => Task.FromResult(new HelloReply { Message = $"Hello {request.Name}" });
}
```

**Expected Output (client)**
```
Hello Alice
```

**Why this output occurs:** The generated `GreeterBase` is overridden; the client stub calls `SayHello` and receives the reply.

---

### Real-World Cases

- **Microservices** — Inter-service communication in .NET .
- **Polyglot systems** — Services written in different languages.
- **Low-latency APIs** — Real-time data pipelines.
- **Streaming** — Real-time telemetry.

---

### References

- gRPC services — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/
- Overview for gRPC on .NET — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/

---

## 2. Protocol Buffers

### Definitions

**Core Definition:** Protocol Buffers (protobuf) is a language-neutral, platform-neutral, extensible mechanism for serializing structured data, and the IDL used to define gRPC contracts.

**Technical Definition:** Protobuf messages are defined in `.proto` files with numbered fields. The binary wire format is compact and backward/forward compatible when fields are added/removed correctly. Field numbers are used on the wire; names are only for code generation. `proto3` is the current syntax; `proto2` is legacy. Protobuf supports scalar types, enums, nested messages, repeated fields, maps, and `oneof`.

**Beginner-Friendly Explanation:** Protocol Buffers is **the compact binary language gRPC speaks**. You define messages in `.proto` files, and the framework generates C# classes that serialize to/from bytes.

---

### Purposes

- To **define strongly typed messages**.
- To provide **compact, fast serialization**.
- To enable **schema evolution** (add/remove fields without breaking).
- To serve as the **IDL** for gRPC services.
- To support **multiple languages**.

---

### Syntax Rules and Structure

```protobuf
syntax = "proto3";

message Person {
  string name = 1;
  int32 age = 2;
  repeated string emails = 3;
  map<string, string> attributes = 4;
  oneof contact {
    string phone = 5;
    string address = 6;
  }
}
```

**Component breakdown**

- `syntax` — `proto3` or `proto2`.
- `message` — A structured type.
- Field number — Unique positive integer on the wire.
- `repeated` — List field.
- `map<K, V>` — Dictionary field.
- `oneof` — Exactly one field set.

#### Syntax Rules

- Field numbers **must not change** once in use.
- Field numbers **1–15** are most efficient (1 byte).
- Do not reuse removed field numbers; use `reserved`.
- `proto3` fields have default values (0, false, empty string).

#### Constraints and Limitations

- Field names are not preserved in wire format.
- Changing field types is a breaking change.
- Unknown fields are preserved in `proto3` (for forward compatibility).

---

### Annotated Code Examples and Expected Outputs

#### Example: Using Generated Protobuf Classes

```csharp
var person = new Person
{
    Name = "Alice",
    Age = 30,
    Emails = { "alice@example.com" },
    Attributes = { ["role"] = "admin" }
};

byte[] bytes = person.ToByteArray();
Person decoded = Person.Parser.ParseFrom(bytes);
```

**Expected Output (conceptual)**
```
decoded.Name == "Alice", decoded.Age == 30
```

**Why this output occurs:** Protobuf serializes to compact binary; parsing reconstructs the same message.

---

### Real-World Cases

- **gRPC contracts** — All gRPC services use protobuf.
- **Event schemas** — Kafka, Pub/Sub with protobuf.
- **Data storage** — Compact binary persistence.

---

### References

- Protocol Buffers — Google Developers — https://protobuf.dev/
- Protocol Buffers language guide (proto3) — https://protobuf.dev/programming-guides/proto3/

---

## 3. Services

### Definitions

**Core Definition:** A gRPC service is a contract and implementation that defines RPC methods callable by clients.

**Technical Definition:** Services are defined in `.proto` files with `service` blocks. The generated `*Base` class is overridden on the server. Services are registered with `MapGrpcService<T>()`. Service implementations can inject services via constructor DI. `ServerCallContext` provides access to headers, cancellation, and metadata.

**Beginner-Friendly Explanation:** A gRPC service is **the server-side implementation of a contract**. You write the methods that clients call.

---

### Purposes

- To **implement RPC contracts**.
- To **expose server functionality** to clients.
- To **share contracts** across languages.
- To **version services** without breaking clients.

---

### Syntax Rules and Structure

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        => Task.FromResult(new HelloReply { Message = $"Hello {request.Name}" });
}
```

**Component breakdown**

- `GreeterBase` — Generated base class.
- `SayHello` — Overridden RPC method.
- `ServerCallContext` — Request metadata, cancellation.

#### Syntax Rules

- Override the generated methods.
- Use async signatures.
- Register with `MapGrpcService<T>()`.

#### Constraints and Limitations

- Services are per-request; do not store request-scoped state in fields.
- Service implementations should be stateless.

---

### Annotated Code Examples and Expected Outputs

#### Example: Service with DI

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly IGreetingService _greetings;
    public GreeterService(IGreetingService greetings) => _greetings = greetings;

    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        => Task.FromResult(new HelloReply { Message = _greetings.Greet(request.Name) });
}
```

**Expected Output (conceptual)**
```
Hello Alice (from injected service)
```

**Why this pattern is used:** DI allows services to use other application services.

---

### Real-World Cases

- **User services** — Authentication and profiles.
- **Order services** — Order management.
- **Inventory services** — Stock tracking.

---

### References

- Create gRPC services and methods — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/services

---

## 4. Clients

### Definitions

**Core Definition:** gRPC clients are generated proxies that call gRPC services using strongly typed methods.

**Technical Definition:** The `Grpc.Net.Client` package provides `GrpcChannel`. Generated `*Client` classes call the service methods. Clients use `GrpcChannel.ForAddress(url)`. For ASP.NET Core apps, `Grpc.Net.ClientFactory` provides `AddGrpcClient<T>()` to register typed clients. Streaming clients use `AsyncServerStreamingCall<T>`, `AsyncClientStreamingCall<T>`, and `AsyncDuplexStreamingCall<T>`.

**Beginner-Friendly Explanation:** A gRPC client is **the generated code that calls the server**. You create a channel, call a method, and get a typed response.

---

### Purposes

- To **call gRPC services** with typed methods.
- To **stream** data to/from the server.
- To integrate with **DI** via `IHttpClientFactory`.
- To configure **deadlines, retries, and cancellation**.

---

### Syntax Rules and Structure

```csharp
using var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);
var reply = await client.SayHelloAsync(new HelloRequest { Name = "Alice" });
```

```csharp
// DI registration
builder.Services.AddGrpcClient<Greeter.GreeterClient>(o =>
    o.Address = new Uri("https://localhost:5001"));
```

**Component breakdown**

- `GrpcChannel` — The connection.
- `GreeterClient` — Generated client.
- `SayHelloAsync` — Async method.
- `AddGrpcClient<T>()` — DI-registered client.

#### Syntax Rules

- Reuse `GrpcChannel` instances; do not create per request.
- Use `AddGrpcClient<T>()` in ASP.NET Core.
- Set deadlines via `CallOptions`.

#### Constraints and Limitations

- Clients are not thread-safe? `GrpcChannel` is thread-safe; clients are lightweight.
- HTTP/2 is required.

---

### Annotated Code Examples and Expected Outputs

#### Example: DI-Registered Client

```csharp
public class MyService
{
    private readonly Greeter.GreeterClient _client;
    public MyService(Greeter.GreeterClient client) => _client = client;

    public async Task<string> GreetAsync(string name)
    {
        var reply = await _client.SayHelloAsync(new HelloRequest { Name = name });
        return reply.Message;
    }
}
```

**Expected Output (conceptual)**
```
Hello Alice
```

**Why this pattern is used:** DI-managed clients share channels and support configuration.

---

### Real-World Cases

- **Microservice clients** — Calling other services.
- **API gateways** — gRPC-to-REST translation.
- **Worker services** — Calling gRPC backends.

---

### References

- Call gRPC services with the .NET client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/client
- gRPC client factory integration in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/clientfactory

---

## 5. Streaming

### Definitions

**Core Definition:** gRPC streaming allows multiple messages to be sent over a single call in either or both directions.

**Technical Definition:** gRPC supports four patterns: **unary** (one request, one response), **server streaming** (one request, many responses), **client streaming** (many requests, one response), and **bi-directional streaming** (many requests, many responses). Streaming methods use `IAsyncStreamReader<T>`, `IServerStreamWriter<T>`, and `IAsyncEnumerable<T>`.

**Beginner-Friendly Explanation:** gRPC streaming is **a live pipeline**. Instead of one request/response, you can send or receive a stream of messages.

---

### Purposes

- To **stream large datasets** efficiently.
- To implement **real-time telemetry** and **chat**.
- To support **long-running operations**.
- To reduce **memory usage** by processing messages incrementally.

---

### Syntax Rules and Structure

```protobuf
service StreamService {
  rpc ServerStream (Request) returns (stream Response);
  rpc ClientStream (stream Request) returns (Response);
  rpc BiDiStream (stream Request) returns (stream Response);
}
```

```csharp
// Server streaming
public override async Task ServerStream(Request request,
    IServerStreamWriter<Response> responseStream, ServerCallContext context)
{
    for (int i = 0; i < 10; i++)
        await responseStream.WriteAsync(new Response { Value = i });
}
```

**Component breakdown**

- `stream` — Marks a streaming parameter or return.
- `IServerStreamWriter<T>` — Server writes to client.
- `IAsyncStreamReader<T>` — Reads client stream.
- `IAsyncEnumerable<T>` — Alternative to stream readers.

#### Syntax Rules

- Streaming methods must use `stream` in the `.proto`.
- Server writes via `IServerStreamWriter<T>`.
- Clients read via `IAsyncStreamReader<T>` or `IAsyncEnumerable<T>`.
- Cancellation is honored via `ServerCallContext.CancellationToken`.

#### Constraints and Limitations

- Streaming requires HTTP/2.
- Long-lived streams need careful resource management.

---

### Annotated Code Examples and Expected Outputs

#### Example: Server Streaming

```csharp
public override async Task ServerStream(
    Request request,
    IServerStreamWriter<Response> responseStream,
    ServerCallContext context)
{
    for (int i = 0; i < 5; i++)
    {
        if (context.CancellationToken.IsCancellationRequested) break;
        await responseStream.WriteAsync(new Response { Value = i });
    }
}
```

**Expected Output (client receives)**
```
0
1
2
3
4
```

**Why this output occurs:** The server writes five messages to the stream; the client reads them incrementally.

---

### Real-World Cases

- **Real-time telemetry** — Sensor data streams.
- **Chat** — Bi-directional messaging.
- **File uploads** — Client streaming.
- **Large datasets** — Server streaming.

---

### References

- gRPC streaming services — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/streaming

---

## 6. gRPC Best Practices

### Definitions

**Core Definition:** gRPC best practices are the guidelines for building performant, secure, and maintainable gRPC services and clients.

**Technical Definition:** Best practices include: **use gRPC for internal service-to-service communication**; **use REST/JSON for public APIs**; **reuse `GrpcChannel` instances**; **set deadlines**; **use streaming for large datasets**; **use `proto3`**; **reserve removed field numbers**; **version services carefully**; **enable TLS**; **use `ServerCallContext.CancellationToken`**; and **avoid blocking calls**.

**Beginner-Friendly Explanation:** The best practices are: **use gRPC for internal services**, **reuse channels**, **set deadlines**, **version contracts carefully**, and **secure the connection**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Reuse channels
var channel = GrpcChannel.ForAddress("https://localhost:5001");
// Reuse across requests; do not create per call.

// 2. Set deadlines
var reply = await client.SayHelloAsync(
    new HelloRequest { Name = "Alice" },
    deadline: DateTime.UtcNow.AddSeconds(5));

// 3. Streaming for large data
await foreach (var item in client.ServerStream(new Request()))
    Process(item);

// 4. proto3 and reserved fields
// syntax = "proto3";
// reserved 5, 6; reserved "old_field";

// 5. TLS
builder.WebHost.ConfigureKestrel(o =>
    o.ListenAnyIP(5001, l => l.UseHttps()));

// 6. Cancellation
context.CancellationToken.ThrowIfCancellationRequested();
```

**Component breakdown**

- **Reuse channels** — Avoids connection overhead.
- **Deadlines** — Prevents hanging calls.
- **Streaming** — For large or real-time data.
- **`proto3` + `reserved`** — Schema evolution safety.
- **TLS** — Encrypts traffic.
- **Cancellation** — Cooperatively stops work.

#### Rules and Constraints

- **Do** use gRPC for internal, high-throughput services.
- **Do** reuse `GrpcChannel` instances.
- **Do** set deadlines on all calls.
- **Do** reserve removed field numbers.
- **Do** enable TLS.
- **Do not** use gRPC for browser clients without gRPC-Web/Connect.
- **Do not** block on async gRPC calls.
- **Do not** change field numbers or types in published contracts.

---

### Annotated Code Examples and Expected Outputs

#### Example: Deadline and Cancellation

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
var reply = await client.SayHelloAsync(
    new HelloRequest { Name = "Alice" },
    deadline: DateTime.UtcNow.AddSeconds(5),
    cancellationToken: cts.Token);
```

**Expected Output (conceptual)**
```
// Call completes within 3 seconds or throws OperationCanceledException/DeadlineExceeded.
```

**Why this pattern is used:** Deadlines and cancellation prevent resource leaks from hanging calls.

---

### Real-World Cases

- **Microservices** — Internal gRPC with deadlines and TLS.
- **API gateways** — gRPC-Web/Connect for browser clients.
- **Streaming pipelines** — Server streaming for telemetry.

---

### References

- gRPC services — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/
- gRPC client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/client
- gRPC streaming — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/streaming
- gRPC-Web — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/grpcweb

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `proto2` syntax | **Legacy**; `proto3` is recommended. |
| gRPC for browser clients | **Requires gRPC-Web or Connect**. |
| `GrpcChannel` per request | **Inefficient**; reuse channels. |
| Missing deadlines | **Risk of hanging calls**. |
| Changing field numbers | **Breaking change**. |
| `Grpc.AspNetCore.Server` on .NET Framework | **Not supported**; gRPC requires .NET Core+. |
| HTTP/2 support | **Required**; some proxies need configuration. |

---

## References (Consolidated)

- gRPC services — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/
- Overview for gRPC on .NET — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/
- Create gRPC services and methods — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/services
- Call gRPC services with the .NET client — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/client
- gRPC client factory integration in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/clientfactory
- gRPC streaming services — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/streaming
- gRPC-Web — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/grpc/grpcweb
- Protocol Buffers — Google Developers — https://protobuf.dev/
- Protocol Buffers language guide (proto3) — https://protobuf.dev/programming-guides/proto3/

---