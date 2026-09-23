# HTTP Communication: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**HTTP (Hypertext Transfer Protocol)** is an application-layer protocol for transmitting hypermedia documents, such as HTML, and is the foundation of data communication on the World Wide Web. In Java, HTTP communication is implemented through the **`java.net.http` package** (Java 11+), which provides the `HttpClient`, `HttpRequest`, and `HttpResponse` classes for sending and receiving HTTP messages.

### Technical Definition

The Java HTTP Client API (`java.net.http`) supports HTTP/1.1 and HTTP/2, synchronous and asynchronous request execution, and full control over request methods, headers, bodies, and response handling. An `HttpRequest` is created via its builder, allowing specification of the request URI, method, headers, timeout, and body publisher. An `HttpResponse` encapsulates the status code, headers, and body, with `BodyHandler` implementations for converting the raw response into strings, byte arrays, files, or streams.

### Beginner-Friendly Explanation

Think of HTTP as a conversation between a customer (your program) and a waiter (a server). Your program asks for something (a request), and the server responds with what you asked for or an explanation of why it can't (a response). The request has a method (what you want to do), headers (extra instructions), and sometimes a body (the actual data you're sending). The response has a status code (how it went), headers (server information), and a body (the data you requested).

### Key Characteristics

- **Standardized methods**: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS define the action.
- **Header-driven**: Metadata about the request/response is carried in key-value headers.
- **Status code taxonomy**: Five families (1xx–5xx) categorize outcomes.
- **Body flexibility**: Requests and responses can carry various payload formats.
- **Content negotiation**: `Content-Type` headers define payload encoding.
- **Asynchronous support**: `sendAsync()` enables non-blocking request execution.

### Prerequisites

- Basic Java syntax and exception handling.
- Familiarity with `java.net.URI` and `java.net.http` packages.
- Understanding of HTTP concepts (methods, headers, status codes).

### Related Programming Areas

- **REST APIs**: HTTP is the transport for RESTful web services.
- **Web Scraping**: Fetching and parsing HTML/JSON from websites.
- **Microservices**: Inter-service communication via HTTP.
- **Authentication**: Token-based and basic auth over HTTP.

### Core Concepts / Features

1. HTTP Methods (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS)
2. Headers (Request vs. Response, Content-Type, Authentication)
3. Status Codes (1xx, 2xx, 3xx, 4xx, 5xx families)
4. Request and Response Bodies
5. Data Payload Handling (Form-urlencoded vs. Multipart)
6. Serialization/Deserialization with Jackson/Gson


## Core Concept 1: HTTP Methods

### Definitions

**Core Definition**: HTTP methods (also called verbs) indicate the desired action to be performed on the identified resource.

**Technical Definition**: The Java `HttpRequest.Builder` allows setting the request method via `.method(String, BodyPublisher)` or convenience methods like `.GET()`, `.POST(BodyPublisher)`, and `.PUT(BodyPublisher)`. Methods are classified as **safe** (no side effects, e.g., GET, HEAD) and **idempotent** (multiple identical requests have the same effect as one, e.g., GET, PUT, DELETE).

**Beginner-Friendly Explanation**: The method tells the server what you want to do: GET means "give me something," POST means "here's something new," PUT means "replace this," PATCH means "update part of this," DELETE means "remove this," HEAD means "give me just the headers," and OPTIONS means "what can I do here?"

### Purposes

- To specify the desired action on a resource.
- To enable semantic clarity and server-side routing.
- To support idempotency and safety guarantees for certain operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/resource"))
    .method("GET", HttpRequest.BodyPublishers.noBody())
    .build();
```

**Convenience Methods:**

| Method | Syntax | Body Required |
|--------|--------|---------------|
| GET | `.GET()` | No |
| POST | `.POST(BodyPublisher)` | Yes |
| PUT | `.PUT(BodyPublisher)` | Yes |
| DELETE | `.DELETE()` | No (optional) |
| HEAD | `.method("HEAD", BodyPublishers.noBody())` | No  |
| PATCH | `.method("PATCH", BodyPublisher)` | Yes |
| OPTIONS | `.method("OPTIONS", BodyPublishers.noBody())` | No |

**Syntax Rules:**

- Default method is GET if not specified.
- POST, PUT, and PATCH require a `BodyPublisher`.
- HEAD uses a special publisher that publishes nothing.
- PATCH and OPTIONS require the generic `.method()` form.

**Constraints and Limitations:**

- Not all servers support all methods.
- Idempotency is a server-side guarantee; clients should respect it for PUT/DELETE.
- Custom methods beyond the standard set may cause interoperability issues.

### Annotated Complete Code Examples

**Example 1: GET and POST Requests**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;

public class HttpMethodsDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // GET request
        HttpRequest getRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .GET()
            .build();

        HttpResponse<String> getResponse = client.send(getRequest, BodyHandlers.ofString());
        System.out.println("GET status: " + getResponse.statusCode());

        // POST request with body
        HttpRequest postRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/post"))
            .header("Content-Type", "application/json")
            .POST(BodyPublishers.ofString("{\"name\":\"Alice\"}"))
            .build();

        HttpResponse<String> postResponse = client.send(postRequest, BodyHandlers.ofString());
        System.out.println("POST status: " + postResponse.statusCode());

        // HEAD request (headers only)
        HttpRequest headRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .method("HEAD", BodyPublishers.noBody())
            .build();

        HttpResponse<Void> headResponse = client.send(headRequest, BodyHandlers.discarding());
        System.out.println("HEAD status: " + headResponse.statusCode());
    }
}
```

**Expected Output:**

```
GET status: 200
POST status: 200
HEAD status: 200
```

**Why This Output Occurs:**
- GET retrieves data without a body.
- POST sends the JSON body with `BodyPublishers.ofString()`.
- HEAD uses `.method("HEAD", BodyPublishers.noBody())` and `BodyHandlers.discarding()` since there's no body.

### Real-World Cases

- **REST APIs**: GET for reads, POST for creates, PUT for full updates, PATCH for partial updates, DELETE for removals.
- **Health checks**: HEAD requests to verify endpoints without downloading bodies.
- **CORS preflight**: OPTIONS requests to check allowed methods.

### References

- OpenJDK – Introduction to the Java HTTP Client - https://openjdk.org/groups/net/httpclient/intro.html
- Oracle Help Center – HttpRequest (Java 11) - https://docs.oracle.com/javase/jp/11/docs/api/java.net.http/java/net/http/HttpRequest.html


## Core Concept 2: Headers (Request vs. Response Headers, Content-Type, Authentication)

### Definitions

**Core Definition**: HTTP **headers** are key-value pairs that carry metadata about the request or response. **Request headers** provide additional information to the server; **response headers** provide information about the response.

**Technical Definition**: Headers are set on requests via `.header(String, String)` or `.setHeader(String, String)` on the `HttpRequest.Builder`. Response headers are accessible via `HttpResponse.headers()`. The `Content-Type` header specifies the media type of the body. Authentication headers carry credentials, typically via `Authorization: Basic <base64>` or `Authorization: Bearer <token>`.

**Beginner-Friendly Explanation**: Headers are like the labels on a package—they tell the postal service what's inside, who it's for, and how to handle it. `Content-Type` says "this is JSON" or "this is a form." `Authorization` says "here's my ID badge."

### Purposes

- To convey metadata about the request or response.
- To specify the format of the body (`Content-Type`).
- To authenticate requests (`Authorization`).
- To control caching, compression, and other behaviors.

### Syntax Rules and Structure

**Setting Request Headers:**

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer my-token")
    .header("Accept", "application/json")
    .GET()
    .build();
```

**Reading Response Headers:**

```java
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
response.headers().map().forEach((key, values) -> {
    System.out.println(key + ": " + values);
});
```

**Common Headers:**

| Header | Type | Purpose |
|--------|------|---------|
| `Content-Type` | Request/Response | Media type of body |
| `Accept` | Request | Desired response media types |
| `Authorization` | Request | Credentials (Basic/Bearer)  |
| `Content-Length` | Both | Size of body in bytes |
| `User-Agent` | Request | Client identification |
| `Cache-Control` | Both | Caching directives |

**Syntax Rules:**

- `.header()` adds a value; `.setHeader()` replaces existing values.
- Multiple headers with the same name are allowed (e.g., `Accept`).
- `Content-Type` is required when sending a body.
- Authentication headers may be restricted in some HTTP client implementations.

**Constraints and Limitations:**

- Header names are case-insensitive.
- Some headers are restricted by the client implementation.
- `Authorization` header behavior may vary between JDK versions.

### Annotated Complete Code Examples

**Example 1: Setting and Reading Headers**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;

public class HeadersDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // Request with headers
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/headers"))
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer test-token-123")
            .header("Accept", "application/json")
            .GET()
            .build();

        HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

        // Read response headers
        System.out.println("Response Content-Type: " +
            response.headers().firstValue("Content-Type").orElse("unknown"));

        System.out.println("Response status: " + response.statusCode());

        // Display all response headers
        response.headers().map().forEach((key, values) ->
            System.out.println(key + ": " + String.join(", ", values)));
    }
}
```

**Expected Output (partial):**

```
Response Content-Type: application/json
Response status: 200
content-type: [application/json]
server: [gunicorn/19.9.0]
...
```

**Why This Output Occurs:**
- Request headers are sent to the server via `.header()`.
- Response headers are read via `response.headers().map()`.
- `firstValue()` retrieves the first value of a specific header.

### Real-World Cases

- **API authentication**: Bearer tokens in `Authorization` header.
- **Content negotiation**: `Accept` header to request JSON or XML.
- **Caching**: `Cache-Control` and `ETag` headers.

### References

- Yisu – Java HTTP请求如何进行身份验证 - https://m.yisu.com/zixun/1039479.html
- OpenJDK – JDK-8194729: Authorization header restriction - https://bugs.openjdk.org/browse/JDK-8194729


## Core Concept 3: Status Codes (1xx, 2xx, 3xx, 4xx, 5xx Families)

### Definitions

**Core Definition**: HTTP **status codes** are three-digit integers returned by the server to indicate the outcome of a request. They are grouped into five families based on the first digit.

**Technical Definition**: The first digit defines the response class: **1xx** (Informational), **2xx** (Successful), **3xx** (Redirection), **4xx** (Client Error), **5xx** (Server Error). Status codes can be converted to their family via enumeration utilities like `HttpStatusFamily.of(int)`.

**Beginner-Friendly Explanation**: Status codes are like the server's reply: "I'm working on it" (1xx), "Here's your stuff" (2xx), "Go over there instead" (3xx), "You messed up" (4xx), or "I messed up" (5xx).

### Purposes

- To communicate the result of a request without parsing the body.
- To enable programmatic decision-making (retry, redirect, fail).
- To standardize error reporting across HTTP APIs.

### Syntax Rules and Structure

**Checking Status Code Family:**

```java
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
int status = response.statusCode();

if (status >= 200 && status < 300) {
    // Success
} else if (status >= 400 && status < 500) {
    // Client error
} else if (status >= 500) {
    // Server error
}
```

**Common Status Codes:**

| Code | Name | Meaning |
|------|------|---------|
| 200 | OK | Standard success |
| 201 | Created | Resource created |
| 301 | Moved Permanently | Resource permanently moved |
| 400 | Bad Request | Malformed request |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 500 | Internal Server Error | Server-side failure |

**Syntax Rules:**

- 1xx codes are informational and rarely seen in application code.
- 3xx codes often include a `Location` header for redirection.
- 4xx codes indicate the client's request has a problem.
- 5xx codes indicate the server failed to fulfill a valid request.

**Constraints and Limitations:**

- Not all servers use status codes consistently.
- Some APIs return 200 with error details in the body (anti-pattern).

### Annotated Complete Code Examples

**Example 1: Handling Status Codes**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;

public class StatusCodeDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // Test various status codes
        int[] statuses = {200, 201, 301, 400, 401, 403, 404, 500};

        for (int expected : statuses) {
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://httpbin.org/status/" + expected))
                .GET()
                .build();

            HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
            String family = getFamily(response.statusCode());
            System.out.printf("Status %d: %s%n", response.statusCode(), family);
        }
    }

    static String getFamily(int status) {
        if (status >= 100 && status < 200) return "Informational (1xx)";
        if (status >= 200 && status < 300) return "Successful (2xx)";
        if (status >= 300 && status < 400) return "Redirection (3xx)";
        if (status >= 400 && status < 500) return "Client Error (4xx)";
        if (status >= 500 && status < 600) return "Server Error (5xx)";
        return "Other";
    }
}
```

**Expected Output:**

```
Status 200: Successful (2xx)
Status 201: Successful (2xx)
Status 301: Redirection (3xx)
Status 400: Client Error (4xx)
Status 401: Client Error (4xx)
Status 403: Client Error (4xx)
Status 404: Client Error (4xx)
Status 500: Server Error (5xx)
```

**Why This Output Occurs:**
- `httpbin.org/status/{code}` returns the specified status code.
- The `getFamily()` method categorizes the code based on its first digit.

### Real-World Cases

- **API error handling**: Branching logic based on 4xx vs. 5xx.
- **Retry strategies**: Retrying 5xx errors but not 4xx.
- **Redirect following**: Handling 301/302 with `Location` header.

### References

- AWS SDK – HttpStatusFamily - https://sdk.amazonaws.com/java/api/2.1.3/software/amazon/awssdk/http/HttpStatusFamily.html
- Jakarta EE – Response.Status.Family - https://jakarta.ee/specifications/platform/11/apidocs/jakarta/ws/rs/core/response.status.family


## Core Concept 4: Request and Response Bodies

### Definitions

**Core Definition**: The **request body** carries data from the client to the server; the **response body** carries data from the server to the client.

**Technical Definition**: In Java's HTTP Client, request bodies are provided via `HttpRequest.BodyPublisher` implementations (e.g., `ofString()`, `ofFile()`, `ofByteArray()`). Response bodies are handled via `HttpResponse.BodyHandler` implementations (e.g., `ofString()`, `ofByteArray()`, `ofFile()`, `discarding()`).

**Beginner-Friendly Explanation**: The body is the actual content of the message. If the request is a letter, the headers are the envelope, and the body is the letter inside.

### Purposes

- To send data to the server (POST, PUT, PATCH).
- To receive data from the server (GET responses).
- To support various content types (JSON, XML, binary, form data).

### Syntax Rules and Structure

**Request Body Publishers:**

| Publisher | Description |
|-----------|-------------|
| `BodyPublishers.ofString(String)` | String body |
| `BodyPublishers.ofFile(Path)` | File contents  |
| `BodyPublishers.ofByteArray(byte[])` | Byte array |
| `BodyPublishers.noBody()` | Empty body |

**Response Body Handlers:**

| Handler | Description |
|---------|-------------|
| `BodyHandlers.ofString()` | Response as String |
| `BodyHandlers.ofByteArray()` | Response as byte[] |
| `BodyHandlers.ofFile(Path)` | Save response to file |
| `BodyHandlers.discarding()` | Discard response body  |

**Syntax Rules:**

- A body publisher is required for POST, PUT, PATCH.
- GET and DELETE typically have no body.
- Body handlers determine how the response is consumed.
- `ofString()` uses the charset from the `Content-Type` header.

**Constraints and Limitations:**

- A request body can only be consumed once.
- Large bodies should be streamed to avoid memory issues.
- `discarding()` is useful when the body is not needed.

### Annotated Complete Code Examples

**Example 1: Sending and Receiving Bodies**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;
import java.nio.file.*;

public class BodyDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // Send string body
        HttpRequest stringRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/post"))
            .header("Content-Type", "text/plain")
            .POST(BodyPublishers.ofString("Hello, Server!"))
            .build();

        HttpResponse<String> stringResponse = client.send(stringRequest, BodyHandlers.ofString());
        System.out.println("String POST status: " + stringResponse.statusCode());

        // Send file body
        Path tempFile = Files.createTempFile("upload", ".txt");
        Files.writeString(tempFile, "File content here");

        HttpRequest fileRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/post"))
            .header("Content-Type", "application/octet-stream")
            .POST(BodyPublishers.ofFile(tempFile))
            .build();

        HttpResponse<String> fileResponse = client.send(fileRequest, BodyHandlers.ofString());
        System.out.println("File POST status: " + fileResponse.statusCode());

        Files.delete(tempFile);
    }
}
```

**Expected Output:**

```
String POST status: 200
File POST status: 200
```

**Why This Output Occurs:**
- `BodyPublishers.ofString()` sends a text body.
- `BodyPublishers.ofFile()` reads the file and sends its contents.
- The response is captured as a String via `BodyHandlers.ofString()`.

### Real-World Cases

- **File uploads**: Sending binary data to a server.
- **API calls**: Sending JSON payloads in POST/PUT requests.
- **File downloads**: Saving response bodies to disk.

### References

- OpenJDK – Introduction to the Java HTTP Client (Body Publishers/Handlers) - https://openjdk.org/groups/net/httpclient/intro.html
- OpenJDK – BodyHandler/BodySubscriber differences - https://bugs.openjdk.org/secure/attachment/75351/outboarding.pdf


## Core Concept 5: Data Payload Handling (Form-urlencoded vs. Multipart)

### Definitions

**Core Definition**: **Form-urlencoded** data encodes key-value pairs as `key1=value1&key2=value2` with URL encoding; **multipart/form-data** encodes multiple parts (each with its own headers) separated by a boundary string.

**Technical Definition**: For `application/x-www-form-urlencoded`, the Java HTTP Client does not provide built-in support; you build the string manually and use `BodyPublishers.ofString()`. For `multipart/form-data`, you must manually construct the body with boundaries and use `BodyPublishers.ofByteArrays()`.

**Beginner-Friendly Explanation**: Form-urlencoded is like a simple list: "name=Alice&age=30." Multipart is like a package with multiple labeled boxes inside, each containing different things (a text field, a file, etc.).

### Purposes

- **Form-urlencoded**: Sending simple key-value pairs (login forms, search queries).
- **Multipart**: Sending files along with metadata (file uploads with descriptions).
- To comply with HTML form submission formats.

### Syntax Rules and Structure

**Form-urlencoded Builder:**

```java
public static HttpRequest.BodyPublisher ofFormData(Map<String, String> data) {
    var builder = new StringBuilder();
    for (var entry : data.entrySet()) {
        if (builder.length() > 0) builder.append("&");
        builder.append(URLEncoder.encode(entry.getKey(), StandardCharsets.UTF_8));
        builder.append("=");
        builder.append(URLEncoder.encode(entry.getValue(), StandardCharsets.UTF_8));
    }
    return BodyPublishers.ofString(builder.toString());
}
```

**Multipart Builder (simplified):**

```java
public static BodyPublisher ofMimeMultipartData(Map<String, Object> data, String boundary) throws IOException {
    var byteArrays = new ArrayList<byte[]>();
    byte[] separator = ("--" + boundary + "\r\nContent-Disposition: form-data; name=")
        .getBytes(StandardCharsets.UTF_8);
    for (var entry : data.entrySet()) {
        byteArrays.add(separator);
        if (entry.getValue() instanceof Path path) {
            String mimeType = Files.probeContentType(path);
            byteArrays.add(("\"" + entry.getKey() + "\"; filename=\"" + path.getFileName() +
                "\"\r\nContent-Type: " + mimeType + "\r\n\r\n").getBytes(StandardCharsets.UTF_8));
            byteArrays.add(Files.readAllBytes(path));
            byteArrays.add("\r\n".getBytes(StandardCharsets.UTF_8));
        } else {
            byteArrays.add(("\"" + entry.getKey() + "\"\r\n\r\n" + entry.getValue() + "\r\n")
                .getBytes(StandardCharsets.UTF_8));
        }
    }
    byteArrays.add(("--" + boundary + "--").getBytes(StandardCharsets.UTF_8));
    return BodyPublishers.ofByteArrays(byteArrays);
}
```

**Syntax Rules:**

- Form-urlencoded requires URL encoding of keys and values.
- Multipart requires a `Content-Type` header with the boundary parameter.
- The boundary must not appear in the data.
- Each part has its own headers (Content-Disposition, Content-Type for files).

**Constraints and Limitations:**

- Java 11 HTTP Client lacks built-in multipart support.
- Multipart boundaries must be properly terminated with `--`.
- Large multipart bodies should be streamed.

### Annotated Complete Code Examples

**Example 1: Multipart File Upload**

```java
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.util.*;

public class MultipartDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        // Create a file to upload
        Path file = Files.createTempFile("upload", ".txt");
        Files.writeString(file, "This is the file content");

        // Build multipart body
        String boundary = "Boundary-" + System.currentTimeMillis();
        Map<String, Object> data = new LinkedHashMap<>();
        data.put("description", "Test upload");
        data.put("file", file);

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/post"))
            .header("Content-Type", "multipart/form-data; boundary=" + boundary)
            .POST(buildMultipart(data, boundary))
            .build();

        HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
        System.out.println("Multipart POST status: " + response.statusCode());

        Files.delete(file);
    }

    static HttpRequest.BodyPublisher buildMultipart(Map<String, Object> data, String boundary)
            throws Exception {
        var parts = new ArrayList<byte[]>();
        for (var entry : data.entrySet()) {
            parts.add(("--" + boundary + "\r\n").getBytes(StandardCharsets.UTF_8));
            parts.add(("Content-Disposition: form-data; name=\"" + entry.getKey() + "\"\r\n\r\n")
                .getBytes(StandardCharsets.UTF_8));
            if (entry.getValue() instanceof Path path) {
                parts.add(Files.readAllBytes(path));
            } else {
                parts.add(entry.getValue().toString().getBytes(StandardCharsets.UTF_8));
            }
            parts.add("\r\n".getBytes(StandardCharsets.UTF_8));
        }
        parts.add(("--" + boundary + "--").getBytes(StandardCharsets.UTF_8));
        return BodyPublishers.ofByteArrays(parts);
    }
}
```

**Expected Output:**

```
Multipart POST status: 200
```

**Why This Output Occurs:**
- The multipart body is constructed with boundaries and part headers.
- `Content-Disposition` identifies each part's name and filename.
- The boundary in the `Content-Type` header matches the body.

### Real-World Cases

- **File upload forms**: Sending files with descriptions to web servers.
- **Profile updates**: Uploading avatar images with user metadata.
- **Document management**: Submitting documents with tags.

### References

- GitHub – Multipart file upload with Java 11 HttpClient - https://github.com/asatklichov/multithreading-java-vs-nodejs/blob/master/src/main/java/concurrency/part3/completablefuture/java11/httpclient/N_FileDownloadUpload.java
- Stack Overflow – Form-urlencoded body publisher - https://stackoverflow.com/revisions/62813364/1
- CodeSignal – Uploading Files to an API Using Java's HttpClient - https://codesignal.com/learn/courses/efficient-api-interactions-with-java/lessons/uploading-files-to-an-api-using-javas-httpclient


## Core Concept 6: Serialization/Deserialization with Jackson/Gson

### Definitions

**Core Definition**: **Serialization** converts Java objects to a transferable format (JSON/XML); **deserialization** converts the received format back into Java objects. **Jackson** and **Gson** are popular libraries for this purpose.

**Technical Definition**: Gson uses `Gson.fromJson(String, Class)` and `Gson.toJson(Object)` for basic conversions, and `TypeToken` for generic types. Jackson uses `ObjectMapper.readValue(String, Class)` and `ObjectMapper.writeValueAsString(Object)`. Both libraries handle nested objects, collections, and custom serializers.

**Beginner-Friendly Explanation**: Serialization is like packing your Java objects into a suitcase (JSON) so they can travel over the network. Deserialization is unpacking the suitcase back into Java objects.

### Purposes

- To convert Java objects to JSON for HTTP request bodies.
- To parse JSON responses into Java objects.
- To handle complex nested data structures.
- To standardize data exchange between client and server.

### Syntax Rules and Structure

**Gson:**

```java
// Serialize
Gson gson = new Gson();
String json = gson.toJson(myObject);

// Deserialize simple
MyObject obj = gson.fromJson(json, MyObject.class);

// Deserialize generic (List)
Type listType = new TypeToken<List<MyObject>>(){}.getType();
List<MyObject> list = gson.fromJson(json, listType);
```

**Jackson:**

```java
ObjectMapper mapper = new ObjectMapper();

// Serialize
String json = mapper.writeValueAsString(myObject);

// Deserialize
MyObject obj = mapper.readValue(json, MyObject.class);

// Deserialize generic
List<MyObject> list = mapper.readValue(json, new TypeReference<List<MyObject>>(){});
```

**Syntax Rules:**

- Both libraries require POJOs with getters/setters (or public fields).
- Gson uses `TypeToken` for generics.
- Jackson uses `TypeReference` for generics.
- Both can be configured with custom date formats and naming strategies.

**Constraints and Limitations:**

- Gson and Jackson are third-party dependencies.
- JSON-B (Jakarta JSON Binding) is a standardized alternative.
- Large payloads may require streaming parsers.

### Annotated Complete Code Examples

**Example 1: HTTP + Gson**

```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.util.List;

public class GsonHttpDemo {
    static class User {
        String name;
        int age;
        // Getters/setters omitted for brevity
    }

    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        Gson gson = new Gson();

        // GET and parse JSON response
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://jsonplaceholder.typicode.com/users"))
            .GET()
            .build();

        HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

        // Deserialize JSON array to List<User>
        List<User> users = gson.fromJson(response.body(),
            new TypeToken<List<User>>(){}.getType());

        users.forEach(u -> System.out.println(u.name + " (" + u.age + ")"));

        // Serialize object to JSON for POST
        User newUser = new User();
        newUser.name = "Alice";
        newUser.age = 30;
        String jsonBody = gson.toJson(newUser);

        HttpRequest postRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://jsonplaceholder.typicode.com/users"))
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
            .build();

        HttpResponse<String> postResponse = client.send(postRequest, BodyHandlers.ofString());
        System.out.println("POST status: " + postResponse.statusCode());
    }
}
```

**Expected Output (partial):**

```
Leanne Graham (0)
Ervin Howell (0)
...
POST status: 201
```

**Why This Output Occurs:**
- `gson.fromJson()` with `TypeToken` parses the JSON array into `List<User>`.
- `gson.toJson()` serializes the `User` object to a JSON string.
- The POST request sends the JSON body with `Content-Type: application/json`.

### Real-World Cases

- **REST API clients**: Parsing JSON responses from third-party APIs.
- **Microservices**: Serializing DTOs for service-to-service communication.
- **Configuration**: Reading JSON config files into Java objects.

### References

- Tencent – 从url Java解析JSON对象数组 - https://cloud.tencent.com.cn/developer/information/%E4%BB%8Eurl%20Java%E8%A7%A3%E6%9E%90JSON%E5%AF%B9%E8%B1%A1%E6%95%B0%E7%BB%84-article
- Stack Overflow – How to map HttpResponse in a Object Java - https://stackoverflow.com/questions/68744766/how-to-map-httpresponse-in-a-object-java
- GitHub – Replace Jackson Databind with a simpler JSON processing library - https://github.com/Scyye/TSJA/issues/33


## References

- OpenJDK – Introduction to the Java HTTP Client - https://openjdk.org/groups/net/httpclient/intro.html
- OpenJDK – BodyHandler/BodySubscriber differences - https://bugs.openjdk.org/secure/attachment/75351/outboarding.pdf
- GitHub – Multipart file upload with Java 11 HttpClient - https://github.com/asatklichov/multithreading-java-vs-nodejs/blob/master/src/main/java/concurrency/part3/completablefuture/java11/httpclient/N_FileDownloadUpload.java
- GitHub – Replace Jackson Databind with a simpler JSON processing library - https://github.com/Scyye/TSJA/issues/33
- AWS SDK – HttpStatusFamily - https://sdk.amazonaws.com/java/api/2.1.3/software/amazon/awssdk/http/HttpStatusFamily.html
- Yisu – Java HTTP请求如何进行身份验证 - https://m.yisu.com/zixun/1039479.html
- Oracle Help Center – HttpRequest (Java 11) - https://docs.oracle.com/javase/jp/11/docs/api/java.net.http/java/net/http/HttpRequest.html
- Stack Overflow – Form-urlencoded body publisher - https://stackoverflow.com/revisions/62813364/1
- Stack Overflow – Multipart boundary error - https://stackoverflow.com/revisions/0f4cbbc4-57ba-42a6-bc19-42ed2ceb9f60/view-source
- Tencent – 从url Java解析JSON对象数组 - https://cloud.tencent.com.cn/developer/information/%E4%BB%8Eurl%20Java%E8%A7%A3%E6%9E%90JSON%E5%AF%B9%E8%B1%A1%E6%95%B0%E7%BB%84-article
- Jakarta EE – Response.Status.Family - https://jakarta.ee/specifications/platform/11/apidocs/jakarta/ws/rs/core/response.status.family
- OpenJDK – JDK-8194729: Authorization header restriction - https://bugs.openjdk.org/browse/JDK-8194729
- Apache HttpComponents – Fundamentals - https://hc.apache.org/httpcomponents-client-4.5.x/current/tutorial/html/fundamentals.html
- Stack Overflow – Form-urlencoded body publisher (revision) - https://stackoverflow.com/revisions/62813364/1
- CodeSignal – Uploading Files to an API Using Java's HttpClient - https://codesignal.com/learn/courses/efficient-api-interactions-with-java/lessons/uploading-files-to-an-api-using-javas-httpclient
- Stack Overflow – How to map HttpResponse in a Object Java - https://stackoverflow.com/questions/68744766/how-to-map-httpresponse-in-a-object-java
- AWS SDK – HttpStatusFamily (2.31.18) - https://sdk.amazonaws.com/java/api/2.31.18/software/amazon/awssdk/http/HttpStatusFamily.html
- Apache – Fundamentals (Idempotent methods) - http://svn-master.apache.org/repos/asf/httpcomponents/httpclient/tags/4.5.3-RC1/src/docbkx/fundamentals.xml
- Stack Overflow – Form-urlencoded body publisher (revision 2) - https://stackoverflow.com/revisions/fee9f405-32f3-45dc-a5e9-881b2c4765ac/view-source