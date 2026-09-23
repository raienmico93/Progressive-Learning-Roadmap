# Java Modern HTTP Client (Java 11+): A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

The **Java Modern HTTP Client** is a standard API introduced in Java 11 (JEP 321) that provides a high-level, feature-rich client for making HTTP requests. It replaces the legacy `HttpURLConnection` with a modern, fluent, and reactive-streams-based architecture supporting HTTP/1.1, HTTP/2, and WebSocket protocols.

### Technical Definition

The `java.net.http` package provides three core abstractions: `HttpClient` (the request sender and configuration holder), `HttpRequest` (an immutable request object built via a builder), and `HttpResponse` (the response container with status, headers, and body). The client supports both synchronous (`send`) and asynchronous (`sendAsync`) execution, with request and response bodies handled as reactive streams through `BodyPublisher` and `BodySubscriber` interfaces. The API is fully asynchronous-capable via `CompletableFuture` integration and supports HTTP/2 multiplexing, redirect policies, proxy configuration, and WebSocket upgrades.

### Beginner-Friendly Explanation

Think of the Java 11 HTTP Client as a modern smartphone compared to the old rotary phone (`HttpURLConnection`). It can make calls (requests) faster, handle multiple conversations at once (HTTP/2 multiplexing), and even do video calls (WebSocket). You can either wait on hold until the call connects (synchronous) or leave a message and get a callback when it's done (asynchronous with `CompletableFuture`).

### Key Characteristics

- **Builder pattern**: All core classes (`HttpClient`, `HttpRequest`) use fluent builders for configuration.
- **Immutable requests**: Once built, `HttpRequest` is immutable and can be sent multiple times.
- **Reactive streams**: Bodies are handled via `Flow.Publisher` and `Flow.Subscriber` abstractions.
- **HTTP/2 support**: Automatic protocol negotiation via ALPN with fallback to HTTP/1.1.
- **WebSocket support**: Built-in WebSocket client via `HttpClient.newWebSocketBuilder()`.
- **Asynchronous by design**: `CompletableFuture` enables composable, non-blocking request pipelines.

### Prerequisites

- Java 11 or later (the API was standardized in Java 11).
- Basic understanding of HTTP concepts (methods, headers, status codes).
- Familiarity with `CompletableFuture` (Java 8) for asynchronous programming.

### Related Programming Areas

- **REST API Clients**: Consuming and producing JSON/XML payloads.
- **Microservices**: Inter-service communication with HTTP/2.
- **Real-time Applications**: WebSocket for bidirectional communication.
- **Web Scraping**: Fetching and parsing web resources.

### Core Concepts / Features

1. `HttpClient`, `HttpRequest`, and `HttpResponse` Core Architecture
2. Synchronous vs. Asynchronous Requests (`CompletableFuture` Integration)
3. Body Handlers and Publishers
4. Advanced Capabilities: HTTP/2, WebSocket, and Redirect Policies
5. Resource Management: Connection Pooling and Thread Executors


## Core Concept 1: `HttpClient`, `HttpRequest`, and `HttpResponse` Core Architecture

### Definitions

**Core Definition**: `HttpClient` is the entry point for sending HTTP requests; `HttpRequest` encapsulates an immutable request; `HttpResponse` contains the server's response.

**Technical Definition**: `HttpClient` is an abstract class that provides configuration for all requests it sends, including protocol version, redirect policy, proxy, and authenticator. `HttpRequest` is built via `HttpRequest.newBuilder()` and contains the URI, method, headers, timeout, and optional body publisher. `HttpResponse<T>` is returned by `send()` or `sendAsync()` and contains the status code, headers, and body (converted by a `BodyHandler`).

**Beginner-Friendly Explanation**: `HttpClient` is like a postal worker—you configure it once (which post office to use, whether to forward mail), and then it can deliver many letters (`HttpRequest`). Each letter contains the address (`URI`), the message (`body`), and special instructions (`headers`). The reply (`HttpResponse`) contains the answer and a status stamp.

### Purposes

- To provide a unified, modern API for all HTTP communication.
- To enable reusable client configuration across multiple requests.
- To separate request construction from execution.
- To offer a consistent abstraction across HTTP/1.1 and HTTP/2.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Create client (reusable)
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .followRedirects(HttpClient.Redirect.NORMAL)
    .connectTimeout(Duration.ofSeconds(10))
    .build();

// Build request (immutable)
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/resource"))
    .header("Content-Type", "application/json")
    .timeout(Duration.ofMinutes(1))
    .GET()
    .build();

// Send synchronously
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

// Or asynchronously
CompletableFuture<HttpResponse<String>> future =
    client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `HttpClient.newBuilder()` | Creates a client builder for configuration. |
| `HttpRequest.newBuilder()` | Creates a request builder. |
| `.uri(URI)` | Sets the request URI (required). |
| `.version(Version)` | Sets preferred HTTP protocol (default HTTP/2) . |
| `.followRedirects(Redirect)` | Sets redirect policy (default NEVER) . |
| `.send(request, BodyHandler)` | Synchronous execution. |
| `.sendAsync(request, BodyHandler)` | Asynchronous execution . |

**Syntax Rules:**

- `HttpClient` is immutable after building; it can be reused for multiple requests .
- `HttpRequest` is immutable after building; it can be sent multiple times .
- A `BodyHandler` must be supplied for every request sent .
- The default protocol version is HTTP/2 .

**Constraints and Limitations:**

- A single `HttpClient` instance has its own connection pool; there is no global sharing between clients .
- HTTP/2 connections are subject to a maximum concurrent stream limit (commonly 100) .
- WebSocket requires a `ws://` or `wss://` URI; `http://` URIs are rejected .

### Annotated Complete Code Examples

**Example 1: Basic GET Request**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;

public class HttpClientBasicDemo {
    public static void main(String[] args) throws Exception {
        // Create a reusable client
        HttpClient client = HttpClient.newHttpClient();

        // Build an immutable request
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .header("Accept", "application/json")
            .GET()
            .build();

        // Send synchronously
        HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

        System.out.println("Status: " + response.statusCode());
        System.out.println("Content-Type: " +
            response.headers().firstValue("Content-Type").orElse("unknown"));
        System.out.println("Body (first 100 chars): " +
            response.body().substring(0, Math.min(100, response.body().length())));
    }
}
```

**Expected Output:**

```
Status: 200
Content-Type: application/json
Body (first 100 chars): {
  "args": {}, 
  "headers": {
    "Accept": "application/json", 
    "Host": "httpbin.org", 
    ...
```

**Why This Output Occurs:**
- `HttpClient.newHttpClient()` creates a client with default settings (HTTP/2 preferred) .
- `HttpRequest.newBuilder()` builds an immutable request with the GET method.
- `client.send()` blocks until the response is received.
- `BodyHandlers.ofString()` converts the response bytes to a `String` .

**Step-by-Step Setup Guide:**
1. Create `HttpClientBasicDemo.java`.
2. Compile with `javac HttpClientBasicDemo.java`.
3. Run with `java HttpClientBasicDemo`.
4. Observe the output.

### Real-World Cases

- **API clients**: Consuming REST APIs with reusable client configuration.
- **Health checks**: Periodic GET requests to verify service availability.
- **Data aggregation**: Fetching data from multiple endpoints with a shared client.

### References

- OpenJDK – Introduction to the Java HTTP Client - https://openjdk.org/groups/net/httpclient/intro.html 
- Oracle Help Center – HttpClient (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.net.http/java/net/http/HttpClient.html 
- OpenJDK – HttpRequest (Java 11) - https://cr.openjdk.org/~chegar/httpclient/supplementary/java.net.http/java/net/http/HttpRequest-report.html 


## Core Concept 2: Synchronous vs. Asynchronous Requests (`CompletableFuture` Integration)

### Definitions

**Core Definition**: **Synchronous** requests block the calling thread until the response arrives. **Asynchronous** requests return immediately with a `CompletableFuture` that completes when the response is available.

**Technical Definition**: `HttpClient.send(HttpRequest, BodyHandler)` blocks until the response is fully received and returns `HttpResponse<T>`. `HttpClient.sendAsync(HttpRequest, BodyHandler)` returns `CompletableFuture<HttpResponse<T>>` immediately, allowing the caller to continue other work . The `CompletableFuture` supports functional composition via `thenApply`, `thenAccept`, `thenCompose`, and `allOf` for orchestrating multiple requests .

**Beginner-Friendly Explanation**: Synchronous is like calling a restaurant and waiting on hold until they answer. Asynchronous is like placing an order online—you get a confirmation number (`CompletableFuture`) immediately, and you can do other things until the food arrives (the future completes).

### Purposes

- **Synchronous**: Simple, sequential request-response workflows.
- **Asynchronous**: High-throughput applications, UI responsiveness, and concurrent request orchestration.
- To compose complex asynchronous pipelines without blocking threads.

### Syntax Rules and Structure

**Synchronous:**

```java
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
```

**Asynchronous:**

```java
CompletableFuture<HttpResponse<String>> future =
    client.sendAsync(request, BodyHandlers.ofString());

// Compose with thenApply, thenAccept, etc.
future.thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

**Component Breakdown:**

| Method | Return Type | Behavior |
|--------|-------------|----------|
| `send()` | `HttpResponse<T>` | Blocks until response  |
| `sendAsync()` | `CompletableFuture<HttpResponse<T>>` | Returns immediately  |

**Syntax Rules:**

- `sendAsync()` returns a `CompletableFuture` that is **cancelable**; calling `cancel(true)` attempts to abort the HTTP exchange .
- Multiple futures can be combined with `CompletableFuture.allOf()` .
- The `thenApply` method maps the response to its body type .

**Constraints and Limitations:**

- Asynchronous operations use the client's executor (default: cached thread pool) .
- Blocking operations inside `thenApply` or body handlers can cause thread starvation .
- Virtual threads may be pinned by `HttpResponseInputStream` due to synchronized/lock contention .

### Annotated Complete Code Examples

**Example 1: Asynchronous GET with Composition**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.util.concurrent.CompletableFuture;

public class AsyncHttpDemo {
    public static void main(String[] args) {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .GET()
            .build();

        // Asynchronous request
        CompletableFuture<HttpResponse<String>> future =
            client.sendAsync(request, BodyHandlers.ofString());

        // Compose: extract body, then print
        future.thenApply(HttpResponse::body)
              .thenAccept(body -> System.out.println("Body length: " + body.length()))
              .join(); // Wait for completion (for demo purposes)

        System.out.println("Request completed.");
    }
}
```

**Expected Output:**

```
Body length: 312
Request completed.
```

**Why This Output Occurs:**
- `sendAsync()` returns a `CompletableFuture` immediately .
- `thenApply(HttpResponse::body)` transforms the future to hold the body string.
- `thenAccept()` consumes the body and prints its length.
- `join()` blocks the main thread until the future completes (for demonstration).

### Real-World Cases

- **Microservice aggregation**: Sending multiple async requests and combining results with `allOf`.
- **UI applications**: Keeping the UI responsive while fetching data in the background.
- **Batch processing**: Issuing thousands of concurrent requests with controlled concurrency.

### References

- Oracle Help Center – HttpClient (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.net.http/java/net/http/HttpClient.html 
- OpenJDK – HTTP Client Recipes (Concurrent Requests) - https://openjdk.org/groups/net/httpclient/recipes-incubating.html 
- OpenJDK – Reporting issues with HTTP and virtual threads - https://mail.openjdk.org/pipermail/net-dev/2023-November/022607.html 


## Core Concept 3: Body Handlers and Publishers

### Definitions

**Core Definition**: **`BodyPublisher`** converts Java objects into a stream of byte buffers for request bodies. **`BodyHandler`** converts response body bytes into a higher-level Java type.

**Technical Definition**: `HttpRequest.BodyPublisher` extends `Flow.Publisher<ByteBuffer>`, enabling reactive-streams-based publishing of request data. `HttpResponse.BodyHandler<T>` is a function that inspects the status code and headers, then returns a `BodySubscriber<T>` that consumes the response body bytes. Both interfaces have factory classes: `BodyPublishers` and `BodyHandlers`.

**Beginner-Friendly Explanation**: A **BodyPublisher** is like packing a suitcase (converting objects to bytes) before sending it. A **BodyHandler** is like unpacking a suitcase when it arrives (converting bytes back to objects). You choose the packing and unpacking method based on what you're sending.

### Purposes

- To decouple body encoding/decoding from request/response handling.
- To support streaming large bodies without loading them into memory.
- To provide convenience implementations for common formats (String, bytes, files).
- To enable custom reactive-streams-based body handling.

### Syntax Rules and Structure

**Common BodyPublishers:**

| Publisher | Description |
|-----------|-------------|
| `BodyPublishers.ofString(String)` | String body  |
| `BodyPublishers.ofByteArray(byte[])` | Byte array body  |
| `BodyPublishers.ofFile(Path)` | File contents  |
| `BodyPublishers.ofInputStream(Supplier<InputStream>)` | Stream body  |
| `BodyPublishers.noBody()` | Empty body |

**Common BodyHandlers:**

| Handler | Description |
|---------|-------------|
| `BodyHandlers.ofString()` | Response as String  |
| `BodyHandlers.ofByteArray()` | Response as byte[]  |
| `BodyHandlers.ofFile(Path)` | Save response to file  |
| `BodyHandlers.ofInputStream()` | Response as InputStream  |
| `BodyHandlers.discarding()` | Discard response body  |

**Syntax Rules:**

- A `BodyPublisher` must be supplied for POST, PUT, PATCH requests .
- A `BodyHandler` is **required** for every request sent .
- `ofInputStream()` streams the body without buffering the entire response .
- Custom implementations can be provided for specialized handling.

**Constraints and Limitations:**

- `ofString()` accumulates the entire body in memory.
- Large files should use `ofFile()` or `ofInputStream()` to avoid memory issues.
- The `BodySubscriber` receives `List<ByteBuffer>` chunks, not individual bytes.

### Annotated Complete Code Examples

**Example 1: File Download with `BodyHandlers.ofFile()`**

```java
import java.net.URI;
import java.net.http.*;
import java.nio.file.*;

public class BodyHandlerDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/image/png"))
            .GET()
            .build();

        // Download response directly to a file
        Path output = Path.of("downloaded.png");
        HttpResponse<Path> response = client.send(request, HttpResponse.BodyHandlers.ofFile(output));

        System.out.println("Status: " + response.statusCode());
        System.out.println("Saved to: " + response.body());
        System.out.println("File size: " + Files.size(response.body()) + " bytes");

        Files.delete(output);
    }
}
```

**Expected Output:**

```
Status: 200
Saved to: downloaded.png
File size: 8090 bytes
```

**Why This Output Occurs:**
- `BodyHandlers.ofFile(Path)` streams the response body directly to the specified file .
- The `HttpResponse<Path>` contains the path to the downloaded file.
- `Files.size()` confirms the file was written.

### Real-World Cases

- **File downloads**: Saving large files without memory bloat.
- **Streaming APIs**: Processing response bodies incrementally.
- **File uploads**: Sending file contents via `BodyPublishers.ofFile()`.

### References

- OpenJDK – Introduction to the Java HTTP Client (BodyPublishers/BodyHandlers) - https://openjdk.org/groups/net/httpclient/intro.html 
- OpenJDK – HTTP Client Recipes (Body Handlers) - https://openjdk.org/groups/net/httpclient/recipes-incubating.html 
- Oracle Help Center – HttpRequest (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpRequest.html 


## Core Concept 4: Advanced HTTP Client Capabilities (HTTP/2, WebSocket, Redirect Policies)

### Definitions

**Core Definition**: **HTTP/2** enables multiplexed streams over a single connection. **WebSocket** provides full-duplex communication over a single TCP connection. **Redirect policies** control whether and how the client follows HTTP redirects.

**Technical Definition**: The Java 11 HTTP Client supports HTTP/2 via ALPN negotiation, with automatic fallback to HTTP/1.1. `HttpClient.Version.HTTP_2` sets the preferred protocol . WebSocket support is provided through `HttpClient.newWebSocketBuilder()`, which creates a `WebSocket.Builder` for establishing WebSocket connections . Redirect policies are set via `HttpClient.Builder.followRedirects()` with values `NEVER`, `NORMAL`, and `ALWAYS` .

**Beginner-Friendly Explanation**: **HTTP/2 multiplexing** is like a highway with multiple lanes—many cars (requests) can travel simultaneously on the same road (connection). **WebSocket** is like a phone call—once connected, both sides can talk and listen at the same time. **Redirect policies** determine whether your GPS automatically reroutes you when the road is closed.

### Purposes

- **HTTP/2**: To improve performance through multiplexing and header compression.
- **WebSocket**: To enable real-time, bidirectional communication (chat, live updates).
- **Redirects**: To automatically handle 3xx responses without manual intervention.

### Syntax Rules and Structure

**HTTP/2 Configuration:**

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)  // Preferred
    .build();
```

**Redirect Policy:**

```java
HttpClient client = HttpClient.newBuilder()
    .followRedirects(HttpClient.Redirect.NORMAL)  // Follow same-protocol redirects
    .build();
```

**WebSocket Creation:**

```java
WebSocket.Builder builder = client.newWebSocketBuilder();
WebSocket ws = builder.buildAsync(URI.create("wss://echo.websocket.org"), listener)
    .join();
ws.sendText("Hello", true);
```

**Redirect Policies:**

| Policy | Description |
|--------|-------------|
| `NEVER` | Never follow redirects (default)  |
| `NORMAL` | Follow redirects, except HTTPS → HTTP  |
| `ALWAYS` | Always follow redirects  |

**WebSocket Methods:**

| Method | Description |
|--------|-------------|
| `sendText(CharSequence, boolean)` | Send text message  |
| `sendBinary(ByteBuffer, boolean)` | Send binary message  |
| `sendClose(int, String)` | Initiate close handshake  |
| `request(long)` | Request more messages from listener  |

**Syntax Rules:**

- HTTP/2 is attempted automatically when the server supports it via ALPN .
- WebSocket requires a `ws://` or `wss://` URI; `http://` URIs are rejected .
- The WebSocket listener receives messages via `onText`, `onBinary`, `onClose`, `onError` .
- WebSocket auto-handles Ping/Pong and Close frames per RFC 6455 .

**Constraints and Limitations:**

- HTTP/2 connections have a maximum concurrent stream limit (typically 100) .
- WebSocket over `http://` upgrade is not supported; use `ws://` or `wss://` .
- Redirect following may not be suitable for non-idempotent methods.

### Annotated Complete Code Examples

**Example 1: WebSocket Echo Client**

```java
import java.net.URI;
import java.net.http.*;
import java.util.concurrent.*;

public class WebSocketDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // Create a listener
        WebSocket.Listener listener = new WebSocket.Listener() {
            @Override
            public CompletionStage<?> onText(WebSocket ws, CharSequence data, boolean last) {
                System.out.println("Received: " + data);
                ws.request(1); // Request one more message
                return null;
            }
        };

        // Build and connect WebSocket
        WebSocket ws = client.newWebSocketBuilder()
            .buildAsync(URI.create("wss://echo.websocket.org"), listener)
            .join();

        // Send a message
        ws.sendText("Hello, WebSocket!", true);

        // Wait for response
        Thread.sleep(2000);

        // Close
        ws.sendClose(WebSocket.NORMAL_CLOSURE, "Done").join();
        System.out.println("WebSocket closed.");
    }
}
```

**Expected Output:**

```
Received: Hello, WebSocket!
WebSocket closed.
```

**Why This Output Occurs:**
- `newWebSocketBuilder()` creates a builder from the client .
- `buildAsync()` connects to the WebSocket endpoint and returns a `CompletableFuture<WebSocket>` .
- The listener's `onText()` is invoked when a text message arrives .
- `ws.request(1)` requests one more message from the server .
- `sendText()` sends a message; `sendClose()` initiates the close handshake .

### Real-World Cases

- **Real-time chat**: WebSocket for instant messaging.
- **Live dashboards**: Streaming metrics via WebSocket.
- **API gateways**: HTTP/2 multiplexing for efficient backend calls.

### References

- OpenJDK – Introduction to the Java HTTP Client (HTTP/2) - https://openjdk.org/groups/net/httpclient/intro.html 
- OpenJDK – HttpClient.Builder (Redirect policies) - https://cr.openjdk.org/~iris/se/11/spec/pfd/java-se-11-pfd-spec/api/java.net.http/java/net/http/HttpClient.Builder.html 
- OpenJDK – WebSocket (Java 11) - https://cr.openjdk.org/~chegar/httpclient/02/javadoc/api/java.net.http/java/net/http/WebSocket.html 
- Cleverence – Java 11 HttpClient Guide - https://www.cleverence.com/articles/oracle-documentation/httpclient-java-se-11-jdk-11-4827/ 
- OpenJDK – Reporting issues with HTTP and virtual threads - https://mail.openjdk.org/pipermail/net-dev/2023-November/022607.html 


## Core Concept 5: Resource Management (Connection Pooling and Thread Executors)

### Definitions

**Core Definition**: **Connection pooling** reuses HTTP connections across requests to reduce latency and resource consumption. **Thread executors** manage the threads used for asynchronous operations and response body handling.

**Technical Definition**: Each `HttpClient` instance maintains its own connection pool; there is no global sharing between client instances . The default executor is a cached thread pool, which can grow unbounded and should be replaced with a fixed or virtual-thread-per-task executor for production use . The executor is used for asynchronous tasks, including SSL handshakes and body subscriber callbacks .

**Beginner-Friendly Explanation**: Connection pooling is like keeping a phone line open instead of hanging up and redialing for every call. Thread executors are like having a team of assistants—you can hire a fixed number (fixed pool), let them come and go as needed (cached pool), or hire a new assistant for every task (virtual threads).

### Purposes

- To reduce latency by reusing established connections.
- To control resource consumption through bounded thread pools.
- To improve scalability by using virtual threads for I/O-bound tasks.
- To prevent thread starvation by avoiding blocking operations in executors.

### Syntax Rules and Structure

**Custom Executor:**

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

HttpClient client = HttpClient.newBuilder()
    .executor(executor)
    .build();
```

**Virtual Thread Executor (Java 21+):**

```java
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

HttpClient client = HttpClient.newBuilder()
    .executor(executor)
    .build();
```

**Connection Pool Characteristics:**

| Aspect | Description |
|--------|-------------|
| Scope | Per-`HttpClient` instance  |
| Sharing | No global sharing between clients  |
| HTTP/2 | Multiplexed streams over one connection  |
| Max streams | Typically 100 per HTTP/2 connection  |

**Syntax Rules:**

- The executor is optional; a default cached thread pool is used if not specified .
- The executor can be shared across multiple `HttpClient` instances .
- Blocking operations in body handlers/executors can cause thread starvation .
- HTTP/2 connection limits can be managed with a `Semaphore` .

**Constraints and Limitations:**

- The default cached thread pool can grow unbounded .
- Virtual thread pinning can occur with `HttpResponseInputStream` due to lock contention .
- HTTP/2 connections are not shared between different `HttpClient` instances .

### Annotated Complete Code Examples

**Example 1: HTTP Client with Custom Executor**

```java
import java.net.URI;
import java.net.http.*;
import java.util.concurrent.*;

public class ExecutorDemo {
    public static void main(String[] args) throws Exception {
        // Create a fixed thread pool
        ExecutorService executor = Executors.newFixedThreadPool(4);

        HttpClient client = HttpClient.newBuilder()
            .executor(executor)
            .version(HttpClient.Version.HTTP_2)
            .build();

        // Send multiple async requests
        for (int i = 0; i < 5; i++) {
            final int id = i;
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://httpbin.org/delay/" + (id % 3)))
                .GET()
                .build();

            client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                .thenAccept(response -> System.out.println(
                    "Request " + id + " completed: " + response.statusCode()));
        }

        Thread.sleep(5000);
        executor.shutdown();
        System.out.println("All requests completed.");
    }
}
```

**Expected Output:**

```
Request 0 completed: 200
Request 1 completed: 200
Request 2 completed: 200
Request 3 completed: 200
Request 4 completed: 200
All requests completed.
```

**Why This Output Occurs:**
- A fixed thread pool of 4 threads is used for async operations .
- Each `sendAsync()` returns a `CompletableFuture` that completes when the response arrives.
- `thenAccept()` prints the status code as each request completes.
- The executor is shut down after all requests are sent.

### Real-World Cases

- **High-throughput APIs**: Bounded executors prevent resource exhaustion.
- **Virtual thread adoption**: Java 21+ applications use virtual threads for HTTP I/O.
- **Connection pooling**: Reusing connections reduces TLS handshake overhead.

### References

- OpenJDK – Reporting issues with HTTP and virtual threads - https://mail.openjdk.org/pipermail/net-dev/2023-November/022607.html 
- OpenJDK – JDK-8203438 (Default cached thread pool) - https://bugs.openjdk.org/browse/JDK-8203438 
- Stack Overflow – HttpClient concurrency management - https://stackoverflow.com/revisions/60d8b153-56cd-4dbf-a693-d14999928b9b/view-source 


## References

- OpenJDK – Introduction to the Java HTTP Client - https://openjdk.org/groups/net/httpclient/intro.html 
- Oracle Help Center – HttpRequest (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpRequest.html 
- OpenJDK – HttpClient.Builder (Redirect policies) - https://cr.openjdk.org/~iris/se/11/spec/pfd/java-se-11-pfd-spec/api/java.net.http/java/net/http/HttpClient.Builder.html 
- OpenJDK – WebSocket (Java 11) - https://cr.openjdk.org/~chegar/httpclient/02/javadoc/api/java.net.http/java/net/http/WebSocket.html 
- Oracle Help Center – HttpClient (Java 26) - https://docs.oracle.com/en/java/javase/26/docs/api/java.net.http/java/net/http/HttpClient.html 
- Cleverence – Java 11 HttpClient Guide - https://www.cleverence.com/articles/oracle-documentation/httpclient-java-se-11-jdk-11-4827/ 
- Oracle Help Center – Package java.net.http (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/package-summary.html 
- OpenJDK – Reporting issues with HTTP and virtual threads - https://mail.openjdk.org/pipermail/net-dev/2023-November/022607.html 
- OpenJDK – HTTP Client Recipes (Incubating) - https://openjdk.org/groups/net/httpclient/recipes-incubating.html 
- OpenJDK – HttpRequest (Supplementary) - https://cr.openjdk.org/~chegar/httpclient/supplementary/java.net.http/java/net/http/HttpRequest-report.html 
- Oracle – WebSocket.Builder - https://download.java.net/java/early_access/panama/docs/api/java.net.http/java/net/http/WebSocket.Builder.html 
- Stack Overflow – HttpClient concurrency management - https://stackoverflow.com/revisions/60d8b153-56cd-4dbf-a693-d14999928b9b/view-source 