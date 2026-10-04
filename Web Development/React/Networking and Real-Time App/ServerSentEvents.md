# Uni-directional Event Streams & Fallbacks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Uni-directional event streams are server-to-client communication channels where the server pushes data to the browser over a persistent HTTP connection, without the client needing to request each update. **Fallbacks** are alternative transport mechanisms used when the primary channel (WebSocket or SSE) is blocked by firewalls, proxies, or legacy clients.

**Technical Definition:** Uni-directional event streams encompass two primary transports: **Server-Sent Events (SSE)**, a standardised W3C/WHATWG API where the server responds with a long-lived HTTP response of `Content-Type: text/event-stream`, emitting lines in a simple wire format (`data:`, `event:`, `id:`, `retry:`) that the browser's `EventSource` API parses and dispatches as DOM events; and **Long Polling**, a fallback where the client issues a request that the server holds open until data arrives or a timeout expires, at which point the client immediately re-issues the request. SSE is built on HTTP and requires no special protocol upgrade, making it proxy-friendly and firewall-compatible, but it is limited to server-to-client communication. Fallback architectures implement a transport cascade — typically WebSocket → SSE → Long Polling — selecting the best available transport based on network conditions and browser capabilities.

**Beginner-Friendly Explanation:** Imagine you're waiting for a package delivery. With normal polling, you call the delivery company every few minutes to ask "Is it here yet?" With long polling, you call once and stay on the line until they tell you it's arrived. With SSE, the delivery company calls you whenever they have news, and you just listen. SSE is like a radio broadcast from the server — you tune in and listen, and the server talks to you. You can't talk back on that channel, but for things like notifications, stock prices, or live feeds, listening is all you need.

### Key Characteristics

- **Server-to-Client Only:** SSE is strictly one-way. If the client needs to send data back, it must use a separate channel (e.g., a standard `fetch` POST), making SSE ideal for notifications, feeds, and dashboards but unsuitable for chat or gaming.
- **HTTP-Native:** SSE uses standard HTTP, requiring no protocol upgrade, no special ports, and no WebSocket handshake. This makes it compatible with HTTP/2, corporate proxies, and load balancers that might block WebSocket upgrades.
- **Built-in Reconnection:** The `EventSource` API automatically reconnects if the connection drops, with a configurable retry interval (`retry:` field) and `Last-Event-ID` support for resuming from where the client left off.
- **Connection Limit Problem:** Over HTTP/1.1, browsers limit concurrent connections per domain to 6. Opening multiple SSE connections (e.g., across tabs) exhausts this limit, blocking other HTTP requests. HTTP/2 multiplexing solves this by allowing up to 100 concurrent streams over a single TCP connection.
- **Text-Only:** SSE transmits UTF-8 text only. Binary data must be base64-encoded, which adds overhead compared to WebSocket's native binary frame support.
- **Authentication Headers Limitation:** The native `EventSource` API cannot send custom headers (e.g., `Authorization: Bearer`). This forces workarounds (cookies, query-string tokens, or fetch-based polyfills).

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useEffect`, `useRef`).
- Familiarity with the Fetch API and HTTP fundamentals (headers, status codes, CORS).
- Basic knowledge of event-driven programming and the browser's event model.
- Awareness of HTTP/2 multiplexing and connection limits.
- Experience with a framework that supports Server-Sent Events or a compatible server runtime.

### Related Programming Areas

- **WebSockets:** Full-duplex communication for chat, gaming, and collaborative editing.
- **Long Polling:** Legacy fallback for environments that block WebSockets or SSE.
- **HTTP/2 Multiplexing:** Eliminating the 6-connection limit by multiplexing streams.
- **Server-Sent Events Protocol:** The W3C `text/event-stream` wire format.
- **Fetch-Based SSE:** Using `@microsoft/fetch-event-source` for custom headers and POST bodies.
- **Real-Time UI Patterns:** Presence, high-frequency updates, and optimistic merging.

### Core Concepts / Features

1. Server-Sent Events (SSE)
2. Long-Polling & Architectural Fallbacks
3. Resource Optimization & HTTP/2

---

## Core Concept 1: Server-Sent Events (SSE)

### Definitions

**Core Definition:** Server-Sent Events (SSE) is a standardised browser API and protocol where the server pushes text-based events to the client over a long-lived HTTP connection using the `text/event-stream` content type.

**Technical Definition:** SSE is defined by the WHATWG HTML Living Standard. The client creates an `EventSource` instance pointing to a server endpoint. The server responds with `Content-Type: text/event-stream`, `Cache-Control: no-cache`, and `Connection: keep-alive`, then keeps the response open indefinitely. Events are sent as newline-delimited fields: `data:` (the payload), `event:` (custom event type), `id:` (last event ID for reconnection), and `retry:` (reconnection interval in milliseconds). The browser's `EventSource` parses these fields and dispatches DOM events (`message`, or the custom event type). If the connection drops, the browser automatically reconnects, sending the `Last-Event-ID` header so the server can resume from where the client left off. SSE is supported in all modern browsers (baseline widely available since 2020) and works in Web Workers.

**Beginner-Friendly Explanation:** SSE is like subscribing to a newsletter. You tell the server "I want to receive updates from this URL," and the server keeps the connection open and sends you messages whenever it has news. You don't have to ask for updates — they just arrive. If your connection drops, the browser automatically tries to reconnect and tells the server which message you last received, so you don't miss anything. The catch is that you can only receive messages, not send them. If you need to send data back, you use a separate request.

### Purposes

- To receive real-time updates from the server without polling.
- To implement live notifications, activity feeds, and monitoring dashboards.
- To stream AI/LLM token-by-token responses to the browser.
- To provide automatic reconnection with `Last-Event-ID` for resumable streams.
- To work in environments where WebSocket upgrades are blocked by proxies or firewalls.
- To leverage HTTP/2 multiplexing for multiple concurrent streams without connection exhaustion.

### Syntax Rules and Structure

**General Syntax with Native `EventSource`:**

```jsx
import { useEffect, useState } from 'react';

function Notifications() {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const eventSource = new EventSource('/api/notifications');

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setNotifications((prev) => [...prev, data]);
    };

    eventSource.onerror = (error) => {
      console.error('SSE error:', error);
      eventSource.close();
    };

    return () => {
      eventSource.close();
    };
  }, []);

  return (
    <ul>
      {notifications.map((n, i) => <li key={i}>{n.message}</li>)}
    </ul>
  );
}
```

**Component Breakdown:**
- `new EventSource('/api/notifications')`: Opens a persistent HTTP connection to the SSE endpoint.
- `eventSource.onmessage`: Fires for each `data:` line from the server.
- `eventSource.onerror`: Fires if the connection fails.
- `eventSource.close()`: Closes the connection in the cleanup function to prevent memory leaks.

**Server-Side SSE Response (Express):**

```javascript
app.get('/api/notifications', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();

  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ message: 'New update', time: Date.now() })}\n\n`);
  }, 3000);

  req.on('close', () => {
    clearInterval(interval);
    res.end();
  });
});
```

**Component Breakdown:**
- `res.setHeader('Content-Type', 'text/event-stream')`: Sets the SSE content type.
- `res.flushHeaders()`: Sends headers immediately so the client knows the connection is open.
- `res.write('data: ...\n\n')`: Writes an SSE event (double newline terminates the event).
- `req.on('close', ...)`: Cleans up the interval when the client disconnects.

**SSE Wire Format:**

| Field | Purpose | Example |
|-------|---------|---------|
| `data:` | Event payload (can be multiple lines) | `data: {"message": "hello"}` |
| `event:` | Custom event type | `event: user-joined` |
| `id:` | Last event ID for reconnection | `id: 42` |
| `retry:` | Reconnection interval (ms) | `retry: 5000` |
| `:` | Comment (ignored) | `: keep-alive` |

**Syntax Rules:**
- The server must set `Content-Type: text/event-stream` and `Cache-Control: no-cache`.
- Each event is terminated by a double newline (`\n\n`).
- The `data:` field can span multiple lines; each line is prefixed with `data:`.
- The `id:` field sets the `Last-Event-ID` header on reconnection.
- The `retry:` field tells the browser how long to wait before reconnecting (default is ~3 seconds).
- Native `EventSource` only supports GET requests and cannot send custom headers.
- Always close the `EventSource` in the `useEffect` cleanup function.

**Constraints and Limitations:**
- Native `EventSource` cannot send `Authorization` headers, forcing workarounds (cookies, query-string tokens, or fetch-based polyfills).
- Native `EventSource` only supports GET; POST requests require a polyfill.
- Over HTTP/1.1, the browser limits concurrent connections to 6 per domain, which SSE exhausts across multiple tabs.
- SSE is text-only; binary data must be base64-encoded.
- The browser's automatic reconnection may hammer the server with retries even on fatal errors (e.g., 401), because it cannot inspect the response status.

### Annotated Code Examples

**Example 1: Basic SSE Hook with Reconnection Cleanup**

```jsx
import { useEffect, useRef } from 'react';

function useSSE(url, onMessage) {
  const eventSourceRef = useRef(null);

  useEffect(() => {
    const eventSource = new EventSource(url);
    eventSourceRef.current = eventSource;

    eventSource.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        onMessage(data);
      } catch (error) {
        console.error('Failed to parse SSE message:', error);
      }
    };

    eventSource.onerror = () => {
      console.error('SSE connection error');
      eventSource.close();
    };

    return () => {
      eventSource.close();
      eventSourceRef.current = null;
    };
  }, [url, onMessage]);
}

// Usage
function ActivityFeed() {
  const [activities, setActivities] = useState([]);
  useSSE('/api/activity', (data) => {
    setActivities((prev) => [...prev, data]);
  });

  return (
    <ul>
      {activities.map((a, i) => <li key={i}>{a.text}</li>)}
    </ul>
  );
}
```

**Expected Output:** The hook connects to the SSE endpoint, parses incoming JSON messages, and appends them to the activity feed. On unmount, the connection is closed.

**Why This Output Occurs:** The `EventSource` opens a persistent connection. The `onmessage` handler parses each event's data and invokes the callback. The cleanup function closes the connection to prevent memory leaks.

**Example 2: SSE with `@microsoft/fetch-event-source` for Auth Headers**

```jsx
import { fetchEventSource } from '@microsoft/fetch-event-source';

async function streamChat(prompt, token, onToken) {
  const ctrl = new AbortController();

  await fetchEventSource('/api/chat/stream', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify({ prompt }),
    signal: ctrl.signal,

    async onopen(response) {
      if (response.ok && response.headers.get('content-type')?.includes('text/event-stream')) {
        return;
      }
      throw new Error(`Unexpected response: ${response.status}`);
    },

    onmessage(ev) {
      onToken(JSON.parse(ev.data));
    },

    onerror(err) {
      throw err; // Throw to stop retrying
    },
  });

  return ctrl;
}
```

**Expected Output:** The function POSTs a prompt to the SSE endpoint with a bearer token, validates the response, and streams tokens to the callback. The `AbortController` allows the caller to cancel the stream.

**Why This Output Occurs:** `@microsoft/fetch-event-source` uses the Fetch API instead of `EventSource`, enabling POST bodies, custom headers, and response inspection. The `onopen` callback validates the response before parsing begins. The `signal` allows clean cancellation.

### Real-World Cases

- **AI/LLM streaming:** Token-by-token streaming of ChatGPT-style responses.
- **Live notifications:** Real-time notification feeds in SaaS applications.
- **Monitoring dashboards:** Live metrics and log streams.
- **Activity feeds:** Social media timelines and audit logs.
- **Deployment status:** Real-time CI/CD pipeline updates.
- **Stock tickers:** Live price updates (server-to-client only).

---

## Core Concept 2: Long-Polling & Architectural Fallbacks

### Definitions

**Core Definition:** Long polling is a fallback transport where the client issues an HTTP request that the server holds open until data is available or a timeout expires, at which point the client immediately re-issues the request. Fallback architectures implement a transport cascade (WebSocket → SSE → Long Polling) to handle environments that block or degrade specific protocols.

**Technical Definition:** Long polling simulates server push over standard HTTP by keeping a request open until the server has data to send. Unlike short polling (which asks on a fixed timer and usually receives empty responses), long polling holds the request open server-side, so the client receives data as soon as it is available. The trade-off is that each message requires a new HTTP request/response cycle, including headers, cookies, and TLS overhead. Long polling works through any proxy, firewall, or load balancer because it is indistinguishable from a normal HTTP request that simply takes a long time to complete. SignalR and Socket.IO automatically degrade from WebSockets to SSE and then to long polling when the primary transport is blocked.

**Beginner-Friendly Explanation:** Imagine you're waiting for a table at a restaurant. With short polling, you ask the host every 5 minutes, "Is my table ready?" With long polling, you ask once and the host says, "Wait here — I'll tell you as soon as it's ready." You stand there until they come get you. It's more efficient than asking repeatedly, but you still have to go back and ask again after each table is seated. Long polling is the "just in case" transport — it works everywhere, even on ancient corporate networks, but it's heavier than SSE or WebSocket.

### Purposes

- To provide real-time updates in environments where WebSocket or SSE is blocked.
- To work through any HTTP proxy, firewall, or load balancer without protocol upgrades.
- To support legacy browsers and clients that do not implement `EventSource` or WebSocket.
- To serve as the final fallback in a transport cascade (WebSocket → SSE → Long Polling).
- To maintain real-time functionality in enterprise networks with restrictive security policies.
- To provide a universal baseline for real-time communication.

### Syntax Rules and Structure

**General Syntax for Long Polling (Client-Side):**

```javascript
async function longPoll(url, onMessage) {
  while (true) {
    try {
      const response = await fetch(url, {
        method: 'GET',
        headers: { 'Cache-Control': 'no-cache' },
      });

      if (response.status === 204) {
        // No content — timeout, immediately re-poll
        continue;
      }

      const data = await response.json();
      onMessage(data);
    } catch (error) {
      console.error('Long poll error:', error);
      await new Promise((resolve) => setTimeout(resolve, 5000));
    }
  }
}
```

**Component Breakdown:**
- The `while (true)` loop continuously re-issues the long-poll request.
- The server holds the request open until data arrives or a timeout occurs.
- A `204 No Content` response indicates a timeout; the client immediately re-polls.
- On error, the client waits (e.g., 5 seconds) before retrying.

**General Syntax for Long Polling (Server-Side, Express):**

```javascript
app.get('/api/poll', async (req, res) => {
  const message = await waitForMessage(req.userId, 30000); // 30s timeout

  if (!message) {
    return res.status(204).end();
  }

  res.json(message);
});
```

**Component Breakdown:**
- `waitForMessage(userId, timeout)`: Waits for a message to be available for the user, up to the timeout.
- `res.status(204).end()`: Returns 204 if no message arrived within the timeout.
- `res.json(message)`: Returns the message immediately if available.

**Transport Cascade Architecture:**

```javascript
const transports = [
  { name: 'WebSocket', connect: connectWebSocket, available: () => 'WebSocket' in window },
  { name: 'SSE', connect: connectSSE, available: () => 'EventSource' in window },
  { name: 'LongPolling', connect: connectLongPolling, available: () => true },
];

async function connectWithFallback(url) {
  for (const transport of transports) {
    if (!transport.available()) continue;

    try {
      return await transport.connect(url);
    } catch (error) {
      console.warn(`${transport.name} failed, trying next transport`);
    }
  }

  throw new Error('All transports failed');
}
```

**Component Breakdown:**
- `transports`: An ordered list of transports to try.
- `transport.available()`: Checks if the transport is supported in the current environment.
- `transport.connect(url)`: Attempts to connect; if it throws, the next transport is tried.
- `LongPolling` is always available, ensuring a fallback.

**Syntax Rules:**
- Implement the transport cascade in order: WebSocket → SSE → Long Polling.
- WebSocket is preferred for bidirectional communication; SSE for server-to-client; long polling for maximum compatibility.
- Each transport should implement the same interface (connect, send, close).
- On failure, fall back to the next transport in the list.
- Long polling should include a timeout (e.g., 30 seconds) to prevent hanging requests.
- Use exponential backoff for reconnection after failures.
- SSE should be used as a fallback when WebSocket upgrades are blocked (e.g., corporate proxies).

**Constraints and Limitations:**
- Long polling consumes a full HTTP request/response cycle per message, including headers and TLS overhead.
- Long polling does not scale as well as SSE or WebSocket for high-frequency messages.
- Each long-polling connection occupies a server thread or async slot until it completes.
- The browser's 6-connection-per-domain limit applies to long polling as well.
- Transport downgrade adds complexity: each transport must implement the same logical interface.
- Some proxies may buffer responses, defeating long polling's "hold open" behaviour.

### Annotated Code Examples

**Example 1: WebSocket → SSE → Long Polling Cascade**

```javascript
const transports = [
  {
    name: 'WebSocket',
    connect(url) {
      return new Promise((resolve, reject) => {
        const ws = new WebSocket(url);
        ws.onopen = () => resolve(ws);
        ws.onerror = reject;
      });
    },
  },
  {
    name: 'SSE',
    connect(url) {
      return new Promise((resolve, reject) => {
        const es = new EventSource(url);
        es.onopen = () => resolve(es);
        es.onerror = reject;
      });
    },
  },
  {
    name: 'LongPolling',
    connect(url) {
      // Long polling is always "available"
      return Promise.resolve({ type: 'polling', url });
    },
  },
];

async function connectWithFallback(url) {
  for (const transport of transports) {
    try {
      console.log(`Trying ${transport.name}...`);
      const connection = await transport.connect(url);
      console.log(`Connected via ${transport.name}`);
      return connection;
    } catch (error) {
      console.warn(`${transport.name} failed:`, error.message);
    }
  }
  throw new Error('All transports failed');
}
```

**Expected Output:** The function attempts WebSocket first; if it fails (e.g., blocked by a proxy), it tries SSE; if that fails, it falls back to long polling. The console logs each attempt.

**Why This Output Occurs:** The `connectWithFallback` function iterates through the transports in order, attempting each one. If `connect` throws, the next transport is tried. Long polling is always available as the final fallback.

### Real-World Cases

- **Enterprise applications:** WebSocket blocked by corporate firewalls; fall back to SSE or long polling.
- **Legacy browser support:** Older browsers without `EventSource` or WebSocket support.
- **Restrictive proxies:** Networks that block HTTP upgrades; long polling works everywhere.
- **Socket.IO and SignalR:** Libraries that implement automatic transport downgrade.
- **AnyCable:** Provides long-polling fallback for locked-down networks.
- **Pollyx:** Modern polling library with WebSocket fallback.

---

## Core Concept 3: Resource Optimization & HTTP/2

### Definitions

**Core Definition:** Resource optimization and HTTP/2 in the context of event streams refers to using HTTP/2 multiplexing to eliminate the per-domain connection limit that plagues SSE and long polling, and tuning flow-control windows to prevent stream blocking under high concurrency.

**Technical Definition:** Over HTTP/1.1, browsers limit concurrent connections per domain to 6. Each SSE connection consumes one of these slots, so opening multiple tabs or multiple SSE streams exhausts the limit and blocks other HTTP requests. HTTP/2 solves this through **multiplexing**: a single TCP connection carries multiple independent streams, with a default maximum of 100 concurrent streams negotiated between client and server. However, HTTP/2 introduces a new failure mode: **per-stream flow control windows**. Each stream has a 65 KB window by default. When the server sends events faster than a slow-reading client can consume them, the window fills, the stream blocks, and cascade failures can occur. Production deployments must tune the per-stream window (e.g., 2 MB) and per-connection window (e.g., 16 MB) to support high-concurrency SSE. Connection cycling with jitter (±20%) prevents thundering-herd reconnection storms when many SSE clients reconnect simultaneously.

**Beginner-Friendly Explanation:** HTTP/1.1 is like a highway with only 6 lanes per destination. Every SSE connection takes one lane, so if you open several tabs, you run out of lanes and other traffic (like API calls) gets stuck. HTTP/2 is like a highway with 100 lanes over a single road — you can have many SSE streams without blocking anything else. But there's a catch: each lane has a small buffer (65 KB). If the server sends data faster than the client reads it, the buffer fills up and the lane blocks. You need to increase the buffer size to handle high-frequency streams.

### Purposes

- To eliminate the 6-connection-per-domain limit of HTTP/1.1 for SSE and long polling.
- To support multiple concurrent SSE streams across tabs without blocking other HTTP requests.
- To prevent flow-control window exhaustion under high-concurrency SSE workloads.
- To tune HTTP/2 parameters (stream window, connection window, max concurrent streams) for production SSE.
- To add jitter to connection cycling to prevent synchronized reconnection storms.
- To scale SSE to thousands of concurrent connections without cascade failures.

### Syntax Rules and Structure

**HTTP/1.1 vs HTTP/2 Connection Limits:**

| Protocol | Concurrent Connections | Per-Domain Limit | Notes |
|----------|----------------------|------------------|-------|
| HTTP/1.1 | 6 per domain | Hard limit | SSE exhausts slots; blocks other requests |
| HTTP/2 | 100 streams (default) | Negotiated | Multiplexed over one TCP connection |
| HTTP/3 | 100 streams (default) | Negotiated | Uses QUIC instead of TCP |

**HTTP/2 Flow Control Tuning (Server-Side, Rust/Hyper Example):**

```rust
// Configure HTTP/2 flow control for high-concurrency SSE
let mut builder = hyper_util::server::conn::auto::Builder::new(TokioExecutor::new());

builder.http2()
    .initial_stream_window_size(2 * 1024 * 1024)      // 2 MB per-stream window
    .initial_connection_window_size(16 * 1024 * 1024)  // 16 MB per-connection window
    .adaptive_window(true)                              // Enable adaptive flow control
    .max_concurrent_streams(256)                        // Max 256 concurrent streams
    .keep_alive_interval(Duration::from_secs(20));      // 20s PING keepalive
```

**Component Breakdown:**
- `initial_stream_window_size(2 * 1024 * 1024)`: Increases the per-stream window from 65 KB to 2 MB.
- `initial_connection_window_size(16 * 1024 * 1024)`: Increases the per-connection window to 16 MB.
- `adaptive_window(true)`: Allows the window to grow dynamically based on throughput.
- `max_concurrent_streams(256)`: Allows up to 256 concurrent streams per connection.
- `keep_alive_interval(20s)`: Sends HTTP/2 PING frames every 20 seconds to keep the connection alive.

**Connection Cycling with Jitter:**

```rust
// Add ±20% jitter to connection cycle intervals
fn jittered_duration(base_secs: u64) -> Duration {
    let jitter_factor = 1.0 + (rand::random::<f64>() * 0.4 - 0.2); // ±20%
    Duration::from_secs_f64(base_secs as f64 * jitter_factor)
}
```

**Component Breakdown:**
- `rand::random::<f64>() * 0.4 - 0.2`: Generates a random value between -0.2 and +0.2.
- `base_secs * jitter_factor`: Applies ±20% jitter to the base interval.
- This prevents all SSE connections from cycling simultaneously (thundering herd).

**Syntax Rules:**
- Serve SSE over HTTP/2 (or HTTP/3) to eliminate the 6-connection limit.
- Increase the per-stream flow-control window (2 MB+) for high-frequency SSE streams.
- Increase the per-connection window (16 MB+) for high-concurrency workloads.
- Set `max_concurrent_streams` to a value that matches your server's capacity (e.g., 256).
- Enable adaptive flow control to allow windows to grow dynamically.
- Add ±20% jitter to connection cycle intervals (e.g., 5 min → 4–6 min).
- Configure HTTP/2 PING keepalive (e.g., 20 seconds) to detect dead connections.
- Ensure the client's HTTP/2 implementation matches the server's flow-control settings.

**Constraints and Limitations:**
- HTTP/2 flow-control windows are per-stream; a slow reader can still block its own stream, but not others.
- Default 65 KB per-stream window exhausts quickly under high-frequency streams.
- Forcing HTTP/1.1 downstream can eliminate flow-control issues by giving each SSE stream its own TCP connection.
- HTTP/2 multiplexing requires a single TCP connection; if that connection drops, all streams are affected.
- Some proxies and load balancers may not support HTTP/2 or may downgrade to HTTP/1.1.
- Connection cycling jitter must be bounded to prevent excessive delays.

### Annotated Code Examples

**Example 1: HTTP/2 Flow Control Configuration for SSE (Rust/Hyper)**

```rust
use hyper_util::server::conn::auto::Builder;
use hyper_util::rt::TokioExecutor;
use std::time::Duration;

async fn start_server() {
    let mut builder = Builder::new(TokioExecutor::new());

    builder
        .http2()
        .initial_stream_window_size(2 * 1024 * 1024)      // 2 MB per-stream
        .initial_connection_window_size(16 * 1024 * 1024)  // 16 MB per-connection
        .adaptive_window(true)
        .max_concurrent_streams(256)
        .keep_alive_interval(Duration::from_secs(20));

    // Serve with the configured builder
    // ...
}
```

**Expected Output:** The HTTP/2 server supports 256 concurrent SSE streams with 2 MB per-stream windows, preventing flow-control exhaustion under high concurrency.

**Why This Output Occurs:** The default 65 KB per-stream window exhausts quickly when the server sends events faster than slow clients can read them. Increasing the window to 2 MB and enabling adaptive flow control prevents blocking. The 16 MB connection window ensures the overall connection has enough buffer for all streams.

### Real-World Cases

- **High-concurrency SSE:** Thousands of concurrent SSE streams over a single HTTP/2 connection.
- **Multi-tab applications:** Users opening multiple tabs with SSE connections without blocking API calls.
- **AI streaming platforms:** Token-by-token streaming with high-frequency events.
- **Live dashboards:** Multiple SSE streams for different data sources.
- **Enterprise applications:** SSE over HTTP/2 to avoid connection exhaustion.

---

## References

- react-eventsource – npm: https://www.npmjs.com/package/react-eventsource
- @microsoft/fetch-event-source: Robust Browser SSE – Safeguard: https://safeguard.sh/resources/blog/fetch-event-source-sse-guide
- EventSource – MDN: https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- SSE vs. WebSockets vs. Polling: Choosing a Real-Time Transport – Back4App: https://www.back4app.com/glossary/sse-vs-websockets-vs-polling/
- fix(server): configure HTTP/2 flow control for high-concurrency SSE – GitHub: https://github.com/everruns/everruns/pull/584
- How to Use Server-Sent Events (SSE) in React – Cybrosys: https://www.cybrosys.com/blog/how-to-use-server-sent-events-sse-in-react
- Server-Sent Events – W3C/WHATWG HTML Living Standard: https://html.spec.whatwg.org/multipage/server-sent-events.html
- EventSource – MDN (connection limit): https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- HTTP/2 – RFC 7540: https://datatracker.ietf.org/doc/html/rfc7540
- react-sse-hooks – GitHub: https://github.com/NepeinAV/react-sse-hooks
- react-use-sse – npm: https://www.npmjs.com/package/react-use-sse
- Socket.IO – Transport Fallback: https://socket.io/docs/v4/how-it-works/
- SignalR – Transport Fallback: https://learn.microsoft.com/en-us/aspnet/core/signalr/transports
- AnyCable – Long Polling Fallback: https://docs.anycable.io/