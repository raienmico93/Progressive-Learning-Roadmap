# WebSockets — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** WebSocket is a communication protocol that provides full-duplex, bidirectional communication channels over a single TCP connection between a client (typically a browser) and a server.

**Technical Definition:** Per MDN, the `WebSocket` object provides the API for creating and managing a WebSocket connection to a server, as well as for sending and receiving data on the connection. The WebSocket protocol is defined by RFC 6455. Unlike HTTP, which is a request-response protocol, WebSocket enables the server to push data to the client without the client explicitly requesting it. The connection is established via an HTTP upgrade handshake, after which the protocol switches to the WebSocket protocol. The WebSocket API is defined in the WHATWG WebSockets Living Standard and is available in browsers via the `WebSocket` interface. WebSocket connections use the `ws://` (unencrypted) and `wss://` (encrypted) URL schemes.

**Beginner-Friendly Explanation:** WebSocket is like a phone call between your browser and a server. Once the call is connected, both sides can talk at any time — the server doesn't have to wait for you to ask for something. This makes WebSocket perfect for real-time applications like chat, live notifications, multiplayer games, and collaborative editing. It's different from HTTP, where the client always has to ask and the server always responds.

### Key Characteristics

- **Full-duplex:** Both client and server can send messages independently and simultaneously.
- **Persistent connection:** The connection stays open until explicitly closed by either side.
- **Low latency:** No HTTP overhead per message after the initial handshake.
- **Single TCP connection:** All communication goes through one connection.
- **Protocol upgrade:** Starts as an HTTP handshake, then upgrades to the WebSocket protocol.
- **Event-driven:** Uses events (`open`, `message`, `error`, `close`) to notify the application.
- **Binary and text support:** Can send strings, Blobs, and ArrayBuffers.
- **Origin-based security:** Servers can validate the `Origin` header during the handshake.

### Prerequisites

- Understanding of JavaScript events and callbacks.
- Familiarity with HTTP and the request-response model.
- Basic knowledge of TCP and network protocols (helpful).
- Awareness of the same-origin policy and CORS (WebSocket has its own origin model).

### Related Programming Areas

- **Real-time applications:** Chat, notifications, live updates.
- **Multiplayer games:** Real-time game state synchronisation.
- **Collaborative editing:** Google Docs-style real-time collaboration.
- **Financial trading:** Live stock prices and trading data.
- **IoT:** Real-time device communication.
- **Server-Sent Events (SSE):** An alternative for server-to-client streaming.

---

## WebSockets

### Definitions

**Core Definition:** A WebSocket is a persistent, bidirectional communication channel between a client and a server, established via an HTTP upgrade handshake.

**Technical Definition:** Per MDN, the `WebSocket` interface is the primary interface for connecting to a WebSocket server and then sending and receiving data on the connection. The `WebSocket` constructor accepts a URL and optional protocols. The connection is established asynchronously, and the `open` event fires when the connection is ready.

### Purposes

- To enable real-time, bidirectional communication between client and server.
- To eliminate the overhead of repeated HTTP requests (polling).
- To allow the server to push data to the client without the client requesting it.
- To support low-latency applications where every millisecond counts.

### Syntax Rules and Structure

```js
const socket = new WebSocket(url, protocols);
```

**Component Breakdown:**
- `url`: The WebSocket URL (`ws://` or `wss://`).
- `protocols` (optional): A string or array of strings specifying subprotocols.

### Key Properties

| Property | Description |
|----------|-------------|
| `readyState` | The current state of the connection (0–3) |
| `url` | The URL of the WebSocket |
| `protocol` | The subprotocol selected by the server |
| `extensions` | The extensions selected by the server |
| `bufferedAmount` | The number of bytes queued but not yet transmitted |
| `binaryType` | The type of binary data (`"blob"` or `"arraybuffer"`) |

### Ready States

| Constant | Value | Description |
|----------|-------|-------------|
| `WebSocket.CONNECTING` | 0 | Connection is being established |
| `WebSocket.OPEN` | 1 | Connection is open and ready |
| `WebSocket.CLOSING` | 2 | Connection is closing |
| `WebSocket.CLOSED` | 3 | Connection is closed |

### Key Methods

| Method | Description |
|--------|-------------|
| `send(data)` | Sends data (string, Blob, ArrayBuffer) |
| `close(code, reason)` | Closes the connection |

### Annotated Code Example: Basic WebSocket

```js
// Step 1: Create a WebSocket connection.
const socket = new WebSocket("wss://echo.websocket.org");

// Step 2: Handle the open event.
socket.addEventListener("open", (event) => {
  console.log("Connected to WebSocket");
  // Step 3: Send a message.
  socket.send("Hello, server!");
});

// Step 4: Handle incoming messages.
socket.addEventListener("message", (event) => {
  console.log("Message from server:", event.data);
});

// Step 5: Handle errors.
socket.addEventListener("error", (event) => {
  console.error("WebSocket error:", event);
});

// Step 6: Handle close.
socket.addEventListener("close", (event) => {
  console.log(`Disconnected: ${event.code} ${event.reason}`);
});
```

**Expected Output:**
```
Connected to WebSocket
Message from server: Hello, server!
```

**Why This Output:** The connection is established, the `open` event fires, a message is sent, and the server echoes it back via the `message` event.

### Real-World Cases

- **Chat applications:** Real-time messaging between users.
- **Live notifications:** Pushing notifications to users instantly.
- **Stock tickers:** Streaming real-time price updates.
- **Collaborative editing:** Syncing changes between users in real time.
- **Gaming:** Real-time game state synchronisation.

---

## WebSocket Handshake

### Definitions

**Core Definition:** The WebSocket handshake is the initial HTTP request-response exchange that upgrades an HTTP connection to a WebSocket connection.

**Technical Definition:** Per RFC 6455, the WebSocket protocol uses an HTTP upgrade mechanism. The client sends an HTTP GET request with an `Upgrade: websocket` header and a `Sec-WebSocket-Key` header. The server responds with `101 Switching Protocols` and a `Sec-WebSocket-Accept` header computed from the client's key. After this handshake, the connection switches to the WebSocket protocol.

### The Handshake Process

**Client Request:**
```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://example.com
```

**Server Response:**
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### Key Headers

| Header | Direction | Description |
|--------|-----------|-------------|
| `Upgrade` | Both | Must be `websocket` |
| `Connection` | Both | Must be `Upgrade` |
| `Sec-WebSocket-Key` | Client | A random base64-encoded value |
| `Sec-WebSocket-Accept` | Server | Computed from the client's key |
| `Sec-WebSocket-Version` | Client | Must be `13` |
| `Sec-WebSocket-Protocol` | Both | Optional subprotocol negotiation |
| `Origin` | Client | The origin of the client |

### Real-World Cases

- **Security:** Servers validate the `Origin` header to prevent cross-site WebSocket hijacking.
- **Protocol negotiation:** Subprotocols allow different application-level protocols.
- **Load balancing:** Some proxies require special handling for WebSocket upgrades.

---

## WebSocket Events

### Definitions

**Core Definition:** WebSocket events are the events fired by the `WebSocket` object during the connection lifecycle.

**Technical Definition:** Per MDN, the `WebSocket` interface fires four main events: `open`, `message`, `error`, and `close`. These events can be handled via `addEventListener()` or by assigning to `onopen`, `onmessage`, `onerror`, and `onclose` properties.

### `open`

**Definition:** The `open` event is fired when the WebSocket connection is successfully established.

```js
socket.addEventListener("open", (event) => {
  console.log("Connection established");
  socket.send("Hello!");
});
```

### `message`

**Definition:** The `message` event is fired when a message is received from the server. The message data is available in `event.data`.

```js
socket.addEventListener("message", (event) => {
  console.log("Received:", event.data);
});
```

### `error`

**Definition:** The `error` event is fired when an error occurs on the connection.

```js
socket.addEventListener("error", (event) => {
  console.error("WebSocket error");
});
```

### `close`

**Definition:** The `close` event is fired when the connection is closed. The event includes `code` (a numeric close code) and `reason` (a string explanation).

```js
socket.addEventListener("close", (event) => {
  console.log(`Closed: code=${event.code}, reason=${event.reason}`);
});
```

### Close Codes

| Code | Description |
|------|-------------|
| 1000 | Normal closure |
| 1001 | Going away |
| 1002 | Protocol error |
| 1003 | Unsupported data |
| 1006 | Abnormal closure (no close frame) |
| 1008 | Policy violation |
| 1009 | Message too big |
| 1011 | Internal server error |

### Annotated Code Example: Complete Event Handling

```js
const socket = new WebSocket("wss://echo.websocket.org");

socket.onopen = (event) => {
  console.log("Connected");
  socket.send("Hello");
};

socket.onmessage = (event) => {
  console.log("Message:", event.data);
};

socket.onerror = (event) => {
  console.error("Error occurred");
};

socket.onclose = (event) => {
  console.log(`Connection closed: ${event.code} — ${event.reason}`);
  if (event.wasClean) {
    console.log("Clean close");
  } else {
    console.log("Unclean close — connection lost");
  }
};
```

**Expected Output:**
```
Connected
Message: Hello
Connection closed: 1000 — 
Clean close
```

### Real-World Cases

- **UI updates:** Updating the UI when the connection opens or closes.
- **Reconnection:** Attempting to reconnect on unexpected close.
- **Error logging:** Logging errors for debugging.

---

## Sending Messages

### Definitions

**Core Definition:** Sending messages is the process of transmitting data from the client to the server over the WebSocket connection.

**Technical Definition:** Per MDN, the `send()` method of the `WebSocket` interface enqueues the specified data to be transmitted to the server over the WebSocket connection, increasing the value of `bufferedAmount` by the number of bytes needed to contain the data. If the data cannot be sent (e.g., the connection is closing or closed), the browser throws an `InvalidStateError`.

### Syntax Rules and Structure

```js
socket.send(data);
```

**Component Breakdown:**
- `data`: The data to send. Can be a string, Blob, ArrayBuffer, or ArrayBufferView.

### Sending Text

```js
socket.send("Hello, server!");
```

### Sending JSON

```js
const message = { type: "chat", text: "Hello" };
socket.send(JSON.stringify(message));
```

### Sending Binary Data

```js
// Sending an ArrayBuffer:
const buffer = new ArrayBuffer(8);
socket.send(buffer);

// Sending a Blob:
const blob = new Blob(["Hello"], { type: "text/plain" });
socket.send(blob);
```

### Annotated Code Example

```js
const socket = new WebSocket("wss://chat.example.com");

socket.onopen = () => {
  // Send a text message.
  socket.send("Hello!");

  // Send a JSON message.
  socket.send(JSON.stringify({
    type: "message",
    user: "Alice",
    text: "Hi everyone!",
  }));

  // Send binary data.
  const buffer = new Uint8Array([1, 2, 3]).buffer;
  socket.send(buffer);
};
```

### Real-World Cases

- **Chat messages:** Sending text and JSON messages.
- **File transfer:** Sending binary data.
- **Game state:** Sending frequent, small updates.

---

## Receiving Messages

### Definitions

**Core Definition:** Receiving messages is the process of handling data sent from the server to the client over the WebSocket connection.

**Technical Definition:** Per MDN, the `message` event is fired when data is received through the WebSocket. The data is available in the `data` property of the event. The type of `data` depends on the `binaryType` property: for text messages, it is a string; for binary messages, it is a `Blob` or `ArrayBuffer`.

### Handling Text Messages

```js
socket.onmessage = (event) => {
  console.log(event.data); // String
};
```

### Handling JSON Messages

```js
socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(message.type, message.text);
};
```

### Handling Binary Data

```js
socket.binaryType = "arraybuffer"; // or "blob"

socket.onmessage = (event) => {
  if (event.data instanceof ArrayBuffer) {
    const bytes = new Uint8Array(event.data);
    console.log("Binary data:", bytes);
  } else {
    console.log("Text data:", event.data);
  }
};
```

### Annotated Code Example

```js
const socket = new WebSocket("wss://chat.example.com");
socket.binaryType = "arraybuffer";

socket.onmessage = (event) => {
  if (typeof event.data === "string") {
    // Text message
    try {
      const message = JSON.parse(event.data);
      handleJsonMessage(message);
    } catch {
      console.log("Plain text:", event.data);
    }
  } else if (event.data instanceof ArrayBuffer) {
    // Binary message
    const bytes = new Uint8Array(event.data);
    console.log("Binary:", bytes);
  }
};

function handleJsonMessage(message) {
  switch (message.type) {
    case "chat":
      displayChat(message.text);
      break;
    case "notification":
      showNotification(message.title);
      break;
    default:
      console.warn("Unknown message type:", message.type);
  }
}
```

### Real-World Cases

- **Chat:** Displaying incoming messages.
- **Notifications:** Showing real-time notifications.
- **Game state:** Updating the game world from server messages.
- **Data streaming:** Processing real-time data feeds.

---

## Binary Data

### Definitions

**Core Definition:** Binary data in WebSocket refers to non-text data (e.g., images, audio, files) transmitted as `Blob` or `ArrayBuffer` objects.

**Technical Definition:** Per MDN, the `binaryType` property of the `WebSocket` interface controls the type of binary data received via the `message` event. The default is `"blob"`; setting it to `"arraybuffer"` causes incoming binary messages to be delivered as `ArrayBuffer` objects.

### Binary Types

| Value | Description |
|-------|-------------|
| `"blob"` | Incoming binary data is a `Blob` (default) |
| `"arraybuffer"` | Incoming binary data is an `ArrayBuffer` |

### Sending Binary Data

```js
// Sending an ArrayBuffer:
const buffer = new ArrayBuffer(16);
const view = new Uint8Array(buffer);
view[0] = 42;
socket.send(buffer);

// Sending a Blob:
const blob = new Blob(["binary data"], { type: "application/octet-stream" });
socket.send(blob);
```

### Receiving Binary Data

```js
socket.binaryType = "arraybuffer";

socket.onmessage = (event) => {
  if (event.data instanceof ArrayBuffer) {
    const bytes = new Uint8Array(event.data);
    console.log("Received bytes:", bytes);
  }
};
```

### Real-World Cases

- **File transfer:** Sending and receiving files.
- **Audio/video streaming:** Transmitting media data.
- **Game assets:** Sending textures, models, and sounds.
- **Sensor data:** Receiving binary sensor readings.

---

## Connection Management

### Definitions

**Core Definition:** Connection management is the practice of monitoring, maintaining, and gracefully closing WebSocket connections.

**Technical Definition:** Per MDN, the `WebSocket` interface provides the `readyState` property to check the connection state, the `close()` method to close the connection, and events (`open`, `close`, `error`) to monitor the connection lifecycle.

### Checking Connection State

```js
if (socket.readyState === WebSocket.OPEN) {
  socket.send("Message");
}
```

### Closing the Connection

```js
// Graceful close:
socket.close(1000, "Normal closure");

// Check for clean close:
socket.onclose = (event) => {
  if (event.wasClean) {
    console.log("Connection closed cleanly");
  } else {
    console.log("Connection lost");
  }
};
```

### Handling Unexpected Disconnections

```js
socket.onclose = (event) => {
  if (!event.wasClean) {
    console.error("Unexpected disconnect");
    // Attempt to reconnect
    reconnect();
  }
};
```

### Annotated Code Example: Complete Connection Management

```js
class WebSocketManager {
  constructor(url) {
    this.url = url;
    this.socket = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
  }

  connect() {
    this.socket = new WebSocket(this.url);

    this.socket.onopen = () => {
      console.log("Connected");
      this.reconnectAttempts = 0;
    };

    this.socket.onclose = (event) => {
      if (!event.wasClean) {
        this.attemptReconnect();
      }
    };

    this.socket.onerror = (error) => {
      console.error("WebSocket error");
    };
  }

  attemptReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.pow(2, this.reconnectAttempts) * 1000;
      console.log(`Reconnecting in ${delay}ms...`);
      setTimeout(() => this.connect(), delay);
    }
  }

  send(message) {
    if (this.socket?.readyState === WebSocket.OPEN) {
      this.socket.send(message);
    } else {
      console.error("Cannot send: connection not open");
    }
  }

  close() {
    this.socket?.close(1000, "Client closing");
  }
}
```

### Real-World Cases

- **Chat apps:** Maintaining a persistent connection.
- **Live dashboards:** Keeping data flowing.
- **IoT devices:** Maintaining device connections.
- **Trading platforms:** Ensuring continuous market data.

---

## Reconnection

### Definitions

**Core Definition:** Reconnection is the process of re-establishing a WebSocket connection after it has been closed unexpectedly.

**Technical Definition:** Per the WebSocket best practices, reconnection should use exponential backoff with jitter to avoid overwhelming the server with reconnection attempts. The `close` event's `wasClean` property distinguishes between clean and unclean closes.

### Reconnection Strategy

```js
function connectWithRetry(url, maxRetries = 5) {
  let retries = 0;

  function connect() {
    const socket = new WebSocket(url);

    socket.onopen = () => {
      console.log("Connected");
      retries = 0; // Reset on successful connection
    };

    socket.onclose = (event) => {
      if (!event.wasClean && retries < maxRetries) {
        retries++;
        const delay = Math.min(30000, 1000 * 2 ** retries);
        const jitter = delay * (0.5 + Math.random() * 0.5);
        console.log(`Reconnecting in ${Math.round(jitter)}ms...`);
        setTimeout(connect, jitter);
      }
    };

    return socket;
  }

  return connect();
}
```

### Real-World Cases

- **Mobile networks:** Handling intermittent connectivity.
- **Server restarts:** Reconnecting after deployment.
- **Network switches:** Reconnecting when the network changes.

---

## Heartbeats

### Definitions

**Core Definition:** Heartbeats are periodic messages sent over a WebSocket connection to keep it alive and detect failures.

**Technical Definition:** Per WebSocket best practices, heartbeats (ping/pong) are used to detect broken connections that might not trigger a `close` event. Browsers cannot send WebSocket ping frames directly, so applications use custom ping/pong messages. If a pong is not received within a timeout, the connection is considered dead and should be closed and reconnected.

### Heartbeat Implementation

```js
let heartbeatInterval;
let pongTimeout;

function startHeartbeat(socket) {
  // Send ping every 30 seconds
  heartbeatInterval = setInterval(() => {
    if (socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify({ type: "ping" }));

      // Expect a pong within 5 seconds
      pongTimeout = setTimeout(() => {
        console.warn("No pong received — closing connection");
        socket.close();
      }, 5000);
    }
  }, 30000);
}

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if (message.type === "pong") {
    clearTimeout(pongTimeout); // Pong received
  }
};

socket.onopen = () => startHeartbeat(socket);
socket.onclose = () => {
  clearInterval(heartbeatInterval);
  clearTimeout(pongTimeout);
};
```

### Real-World Cases

- **Detecting dead connections:** Finding broken connections before they cause issues.
- **Proxy timeouts:** Keeping connections alive through proxies.
- **Mobile networks:** Detecting network changes.

---

## WebSocket vs HTTP

### Definitions

**Core Definition:** WebSocket and HTTP are both network protocols, but they serve different purposes: HTTP is a request-response protocol, while WebSocket is a full-duplex, persistent communication protocol.

### Comparison

| Feature | WebSocket | HTTP |
|---------|-----------|------|
| **Communication** | Full-duplex (bidirectional) | Half-duplex (request-response) |
| **Connection** | Persistent | Stateless (new connection per request) |
| **Overhead** | Low (after handshake) | High (headers per request) |
| **Latency** | Very low | Higher |
| **Server push** | Yes | No (requires polling, SSE, or HTTP/2 push) |
| **Protocol** | `ws://` or `wss://` | `http://` or `https://` |
| **Use case** | Real-time, low-latency | General web requests |
| **Data types** | Text and binary | Text (and binary with HTTP/2) |

### When to Use WebSocket vs HTTP

| Use WebSocket When | Use HTTP When |
|--------------------|---------------|
| Real-time bidirectional communication | Request-response is sufficient |
| Low latency is critical | Standard web requests |
| Server needs to push data | Client initiates all requests |
| Persistent connection is beneficial | Stateless communication is fine |
| Chat, games, live data | REST APIs, page loads, forms |

### Real-World Cases

- **WebSocket:** Chat, multiplayer games, live trading, collaborative editing.
- **HTTP:** REST APIs, page loads, form submissions, file downloads.
- **Both:** Many applications use HTTP for initial data and WebSocket for real-time updates.

---

## References

- MDN Web Docs – WebSocket — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – WebSocket API — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
- MDN Web Docs – Writing WebSocket client applications — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications
- RFC 6455 – The WebSocket Protocol — https://datatracker.ietf.org/doc/html/rfc6455
- WHATWG – WebSockets Living Standard — https://websockets.spec.whatwg.org/
- MDN Web Docs – WebSocket: readyState — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/readyState
- MDN Web Docs – WebSocket: send() — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/send
- MDN Web Docs – WebSocket: close() — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/close
- MDN Web Docs – WebSocket: binaryType — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/binaryType
- MDN Web Docs – CloseEvent — https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent
- MDN Web Docs – MessageEvent — https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent
- Ably – WebSocket Protocol — https://ably.com/topic/websockets
- PieSocket – WebSocket Best Practices — https://www.piesocket.com/blog/websocket-best-practices
- RFC 6455 Section 4 – Opening Handshake — https://datatracker.ietf.org/doc/html/rfc6455#section-4
- RFC 6455 Section 7.4 – Close Codes — https://datatracker.ietf.org/doc/html/rfc6455#section-7.4
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf