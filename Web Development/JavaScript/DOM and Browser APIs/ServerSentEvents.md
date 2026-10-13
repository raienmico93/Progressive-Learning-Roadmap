# Server-Sent Events — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Server-Sent Events (SSE) is a standardised technology that enables a server to push text-based event data to a web client over a single, long-lived HTTP connection. It provides a one-way (server-to-client) communication channel.

**Technical Definition:** Per MDN, "Server-Sent Events (SSE) is a technology that enables a server to send text-based event data to a client over a single, long-lived HTTP connection. The `EventSource` interface is the browser API for receiving Server-Sent Events". SSE is defined by the WHATWG HTML Living Standard (Server-Sent Events specification). The server responds with a `text/event-stream` MIME type and keeps the connection open, sending events as they occur. The client uses the `EventSource` constructor to connect to the endpoint. SSE uses a simple text-based wire format with `data:`, `event:`, `id:`, and `retry:` fields. Unlike WebSockets, SSE is unidirectional (server-to-client only), uses standard HTTP, and automatically reconnects if the connection drops. SSE is often compared to WebSockets for real-time applications but is simpler and better suited for scenarios where only the server needs to push data.

**Beginner-Friendly Explanation:** Server-Sent Events are like a news ticker on a website. The server keeps sending updates to your browser, and your browser displays them as they arrive. It's a one-way street — the server talks, the browser listens. You can't send messages back through SSE (unlike WebSockets), but for things like live scores, stock prices, or notification streams, SSE is simpler and easier to work with. It uses regular HTTP, so it works through most firewalls and proxies without special configuration.

### Key Characteristics

- **Unidirectional:** Server-to-client only. The client cannot send messages back through SSE.
- **HTTP-based:** Uses standard HTTP/HTTPS (`http://` or `https://`), not a custom protocol.
- **Automatic reconnection:** The browser automatically reconnects if the connection drops (with configurable retry intervals).
- **Text-based:** Only UTF-8 text data can be sent; binary data must be base64-encoded.
- **Event stream format:** A simple line-based format with `data:`, `event:`, `id:`, and `retry:` fields.
- **`text/event-stream` MIME type:** The server must respond with this content type.
- **Built-in event IDs:** The `id:` field allows the client to resume from the last received event after reconnection.
- **Lightweight:** No handshake upgrade, no special protocol — just HTTP.
- **Browser support:** Supported in all modern browsers except IE.

### Prerequisites

- Understanding of JavaScript events and event listeners.
- Familiarity with HTTP and the request-response model.
- Basic knowledge of server-side programming (Node.js, Python, etc.).
- Awareness of the EventSource API.

### Related Programming Areas

- **Real-time notifications:** Pushing notifications to users.
- **Live feeds:** Stock tickers, sports scores, news updates.
- **Log streaming:** Tailing logs in real time.
- **Progress updates:** Long-running job progress.
- **IoT dashboards:** Real-time sensor data.
- **WebSockets comparison:** Choosing between SSE and WebSockets.

---

## Server-Sent Events

### Definitions

**Core Definition:** Server-Sent Events is a technology that allows a server to push text-based event data to a client over a single, long-lived HTTP connection.

**Technical Definition:** Per the HTML Living Standard, "This specification defines an API for opening an HTTP connection for receiving push notifications from a server in the form of DOM events. The API is designed to be extendable to other push notification schemes". The server sends events as a stream of text in a specific format, and the browser parses them into DOM events.

### Purposes

- To enable real-time, server-to-client communication without polling.
- To provide a simpler alternative to WebSockets when only server push is needed.
- To leverage standard HTTP infrastructure (proxies, firewalls, load balancers).
- To support automatic reconnection and event resumption.
- To stream text-based data (notifications, updates, logs) efficiently.

### Syntax Rules and Structure

**Client-side (browser):**
```js
const eventSource = new EventSource(url);
```

**Server-side (Node.js/Express example):**
```js
app.get("/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  res.write("data: Hello, client!\n\n");

  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: Date.now() })}\n\n`);
  }, 1000);

  req.on("close", () => {
    clearInterval(interval);
    res.end();
  });
});
```

**Component Breakdown:**
- `EventSource`: The browser API for receiving SSE.
- `text/event-stream`: The required MIME type for SSE responses.
- `data: ...\n\n`: The event stream format.

### Annotated Code Example: Basic SSE

```js
// Step 1: Create an EventSource connection.
const eventSource = new EventSource("/api/events");

// Step 2: Handle the open event.
eventSource.onopen = (event) => {
  console.log("Connection opened");
};

// Step 3: Handle incoming messages.
eventSource.onmessage = (event) => {
  console.log("Message:", event.data);
};

// Step 4: Handle errors.
eventSource.onerror = (event) => {
  console.error("EventSource error");
  // The browser automatically reconnects.
};

// Step 5: Close the connection when done.
// eventSource.close();
```

**Expected Output:**
```
Connection opened
Message: { "time": 1699999999999 }
Message: { "time": 1700000000999 }
...
```

**Why This Output:** The `EventSource` connects to the server, which sends events as a stream. The `onmessage` handler processes each event as it arrives.

### Real-World Cases

- **Live notifications:** Pushing notifications to users.
- **Stock tickers:** Streaming price updates.
- **Sports scores:** Live game updates.
- **Log streaming:** Tailing logs in real time.
- **Progress indicators:** Reporting job progress.

---

## `EventSource`

### Definitions

**Core Definition:** `EventSource` is the browser interface for receiving Server-Sent Events. It connects to an SSE endpoint and fires events for incoming messages.

**Technical Definition:** Per MDN, "The `EventSource` interface is the browser's API for receiving Server-Sent Events. It provides a way to connect to a server and receive events without polling". The `EventSource` constructor accepts a URL and an optional options object. The connection is established immediately upon construction.

### Key Properties

| Property | Description |
|----------|-------------|
| `url` | The URL of the event source |
| `withCredentials` | Whether to send cookies with the request |
| `readyState` | The connection state (0 = CONNECTING, 1 = OPEN, 2 = CLOSED) |

### Ready States

| Constant | Value | Description |
|----------|-------|-------------|
| `EventSource.CONNECTING` | 0 | Connection is being established |
| `EventSource.OPEN` | 1 | Connection is open |
| `EventSource.CLOSED` | 2 | Connection is closed |

### Key Methods

| Method | Description |
|--------|-------------|
| `close()` | Closes the connection |

### Key Events

| Event | Description |
|-------|-------------|
| `open` | Fired when the connection is established |
| `message` | Fired when a message is received (no `event:` field) |
| `error` | Fired when an error occurs |
| Custom event | Fired when the server sends an `event:` field |

### Annotated Code Example: EventSource with Custom Events

```js
const eventSource = new EventSource("/api/notifications");

// Default event (no event: field):
eventSource.addEventListener("message", (event) => {
  console.log("Message:", event.data);
});

// Custom event ("notification"):
eventSource.addEventListener("notification", (event) => {
  const data = JSON.parse(event.data);
  console.log("Notification:", data.title, data.body);
});

// Custom event ("update"):
eventSource.addEventListener("update", (event) => {
  console.log("Update:", event.data);
});

// Handle errors and reconnection.
eventSource.addEventListener("error", (event) => {
  if (eventSource.readyState === EventSource.CLOSED) {
    console.error("Connection closed");
  } else {
    console.warn("Connection error — reconnecting...");
  }
});
```

**Expected Output:**
```
Message: hello
Notification: { title: "New message", body: "You have a new message" }
Update: { id: 1, status: "complete" }
```

**Why This Output:** The server sends events with different `event:` fields. Each custom event type is handled by a dedicated listener.

### Real-World Cases

- **Notification systems:** Handling different notification types.
- **Live updates:** Processing various update events.
- **Dashboards:** Handling multiple event streams.

---

## Event Stream Format

### Definitions

**Core Definition:** The event stream format is a simple, line-based text format used by SSE to send events from the server to the client.

**Technical Definition:** Per the HTML Living Standard, "The event stream format is a simple line-based format. Each line is terminated by a line feed (LF) character. Lines are processed in order". The format uses four fields: `data:`, `event:`, `id:`, and `retry:`.

### Field Reference

| Field | Description |
|-------|-------------|
| `data:` | The data payload. Multiple `data:` lines are concatenated with newlines. |
| `event:` | The event type. If omitted, the default `message` event is fired. |
| `id:` | The event ID. The browser sends this in the `Last-Event-ID` header on reconnection. |
| `retry:` | The reconnection time in milliseconds. |
| `:comment` | A comment line, ignored by the browser. |

### Format Rules

- Each field is followed by a colon and an optional space.
- Events are separated by a blank line (`\n\n`).
- A single event can have multiple `data:` lines.
- The `id:` field sets the last event ID.
- The `retry:` field sets the reconnection delay.

### Annotated Code Example: Server-Sent Events

```js
// Server-side (Node.js/Express):
app.get("/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Simple message:
  res.write("data: Hello, client!\n\n");

  // Event with an ID:
  res.write("id: 1\n");
  res.write("data: First event\n\n");

  // Custom event type:
  res.write("event: notification\n");
  res.write("data: You have a new message\n\n");

  // Multi-line data:
  res.write("data: Line 1\n");
  res.write("data: Line 2\n\n");

  // Retry directive (reconnect after 3 seconds):
  res.write("retry: 3000\n\n");
});
```

**Expected Output:** The client receives:
```
Message: Hello, client!
Message: First event
Notification: You have a new message
Message: Line 1\nLine 2
```

**Why This Output:** Each event is separated by a blank line. The `event:` field determines the event type. Multiple `data:` lines are concatenated with newlines.

### Real-World Cases

- **Live data feeds:** Sending structured data as JSON strings.
- **Event typing:** Using `event:` to categorise messages.
- **Resumption:** Using `id:` to resume after disconnection.
- **Reconnection tuning:** Using `retry:` to control reconnection timing.

---

## Reconnection

### Definitions

**Core Definition:** Reconnection is the automatic process by which the browser re-establishes an SSE connection after it has been closed, resuming from the last received event.

**Technical Definition:** Per MDN, "By default, if the connection between the client and server closes, the connection is restarted. The connection is terminated with the `.close()` method". The browser sends a `Last-Event-ID` header with the ID of the last received event, allowing the server to resume from that point.

### How Reconnection Works

1. The connection drops (network issue, server restart, timeout).
2. The browser waits for the `retry:` interval (default: ~3 seconds).
3. The browser sends a new request with the `Last-Event-ID` header.
4. The server resumes sending events from the last acknowledged ID.

### Annotated Code Example: Server-Side Resumption

```js
// Server-side (Node.js):
let events = [];
let nextId = 1;

app.get("/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Check for Last-Event-ID header.
  const lastEventId = req.headers["last-event-id"];
  if (lastEventId) {
    // Resume from the last received event.
    const lastIndex = events.findIndex((e) => e.id === parseInt(lastEventId));
    if (lastIndex !== -1) {
      events.slice(lastIndex + 1).forEach((event) => {
        res.write(`id: ${event.id}\n`);
        res.write(`data: ${event.data}\n\n`);
      });
    }
  }

  // Send new events as they occur.
  const interval = setInterval(() => {
    const event = { id: nextId++, data: `Event ${nextId}` };
    events.push(event);
    res.write(`id: ${event.id}\n`);
    res.write(`data: ${event.data}\n\n`);
  }, 1000);

  req.on("close", () => {
    clearInterval(interval);
    res.end();
  });
});
```

**Expected Output:** After reconnection, the client receives missed events (if the server implements resumption) followed by new events.

### Real-World Cases

- **Resilient feeds:** Ensuring no events are missed during reconnection.
- **Mobile networks:** Handling intermittent connectivity.
- **Server restarts:** Resuming after deployment.

---

## Event Types

### Definitions

**Core Definition:** Event types in SSE are named events that allow the server to categorise messages, which the client handles with dedicated listeners.

**Technical Definition:** Per MDN, "The `event` field in the event stream specifies the type of event. If the `event` field is not present, the `message` event is fired". Custom event types are handled with `addEventListener(eventType, handler)`.

### Default Event: `message`

```js
eventSource.onmessage = (event) => {
  console.log("Message:", event.data);
};
```

### Custom Events

```js
// Server sends:
// event: notification
// data: You have a new message

// Client handles:
eventSource.addEventListener("notification", (event) => {
  console.log("Notification:", event.data);
});
```

### Event Types in Practice

```js
const eventSource = new EventSource("/api/events");

eventSource.addEventListener("message", (event) => {
  console.log("Default message:", event.data);
});

eventSource.addEventListener("user-joined", (event) => {
  const user = JSON.parse(event.data);
  console.log(`${user.name} joined`);
});

eventSource.addEventListener("user-left", (event) => {
  const user = JSON.parse(event.data);
  console.log(`${user.name} left`);
});

eventSource.addEventListener("new-message", (event) => {
  const message = JSON.parse(event.data);
  displayMessage(message);
});
```

### Real-World Cases

- **Chat applications:** Different event types for joins, leaves, and messages.
- **Notification systems:** Categorising notifications by type.
- **Dashboards:** Handling different update types (data, status, alerts).
- **Game events:** Different event types for player actions, game state, etc.

---

## SSE vs WebSockets

### Definitions

**Core Definition:** SSE and WebSockets are both technologies for real-time communication, but they differ in direction, protocol, complexity, and use cases.

**Technical Definition:** Per MDN, "SSE is a simpler alternative to WebSockets. SSE is unidirectional (server-to-client), while WebSockets are bidirectional. SSE uses standard HTTP, while WebSockets use a custom protocol after an HTTP upgrade handshake".

### Comparison

| Feature | SSE | WebSockets |
|---------|-----|------------|
| **Direction** | Unidirectional (server → client) | Bidirectional (both directions) |
| **Protocol** | HTTP/HTTPS | WebSocket protocol (ws://, wss://) |
| **Complexity** | Simple | More complex |
| **Reconnection** | Automatic | Manual (must implement) |
| **Data types** | Text only (UTF-8) | Text and binary |
| **Message format** | Line-based text | Frames (text or binary) |
| **Browser support** | All modern browsers (not IE) | All modern browsers |
| **Firewall/proxy** | Works through standard HTTP | May require special configuration |
| **Overhead** | Low (HTTP headers once) | Very low (after handshake) |
| **Server push** | Yes | Yes |
| **Client push** | No | Yes |
| **Use case** | Notifications, live feeds | Chat, games, collaborative editing |
| **Max connections** | ~6 per domain (HTTP/1.1) | ~200 per domain |

### When to Use SSE vs WebSockets

| Use SSE When | Use WebSockets When |
|--------------|---------------------|
| Only server-to-client communication is needed | Bidirectional communication is needed |
| You want automatic reconnection | You need full control over the connection |
| You want to use standard HTTP | You need binary data |
| You want simplicity | You need low latency for frequent messages |
| You're streaming text updates | You're building a chat or game |

### Real-World Cases

- **SSE:** Live scores, notifications, log streaming, progress updates.
- **WebSockets:** Chat, multiplayer games, collaborative editing, trading platforms.
- **Both:** Some applications use WebSockets for real-time communication and SSE for notifications.

---

## References

- MDN Web Docs – Server-sent events — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- MDN Web Docs – Using server-sent events — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events
- MDN Web Docs – EventSource — https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- MDN Web Docs – EventSource: withCredentials — https://developer.mozilla.org/en-US/docs/Web/API/EventSource/withCredentials
- WHATWG – HTML Living Standard: Server-Sent Events — https://html.spec.whatwg.org/multipage/server-sent-events.html
- W3C – Server-Sent Events — https://www.w3.org/TR/eventsource/
- DigitalOcean – How to Implement Server-Sent Events in Node.js — https://www.digitalocean.com/community/tutorials/nodejs-server-sent-events
- Ably – Server-Sent Events vs WebSockets — https://ably.com/blog/websockets-vs-sse
- FreeCodeCamp – Server-Sent Events: A Comprehensive Guide — https://www.freecodecamp.org/news/server-sent-events/
- HTML5 Rocks – Stream Updates with Server-Sent Events — https://www.html5rocks.com/en/tutorials/eventsource/basics/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf