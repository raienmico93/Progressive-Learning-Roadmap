# Laravel WebSocket Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel WebSocket Architecture is the end-to-end system design that connects a Laravel backend application to browser clients via persistent, full-duplex WebSocket connections, using a broadcasting driver (Laravel Reverb, Pusher, or Ably) as the message broker between server-originated events and client-side subscriptions managed by Laravel Echo.

**Technical Definition:** Laravel's WebSocket architecture comprises four cooperating layers: (1) a **broadcasting driver** configured in `config/broadcasting.php` that serializes and transmits events, (2) a **WebSocket server or service** (Reverb, Pusher, Ably) that maintains persistent connections and channel state, (3) an **authorization layer** (`/broadcasting/auth`, signed tokens, CSRF protection) that validates private and presence channel subscriptions, and (4) a **client runtime** (Laravel Echo + pusher-js) that manages connections, subscriptions, reconnect strategies, heartbeats, and event dispatch. Scaling is achieved by decoupling the web tier from the WebSocket tier, using Redis pub/sub for cross-node message fan-out, and horizontally scaling Reverb nodes behind a load balancer with sticky sessions or a shared Redis backend.

**Beginner-Friendly Explanation:** Think of Laravel's WebSocket architecture as a postal system with four parts: the sender (your Laravel app), the sorting office (the WebSocket server like Reverb), the postal workers (config, auth, connection management), and the recipients (browsers using Laravel Echo). Messages travel instantly and persistently. When you need to serve a whole city, you add more sorting offices (scaling Reverb) and connect them with a shared radio network (Redis).

### Key Characteristics

1. **Persistent Full-Duplex Connections:** WebSocket connections stay open, enabling bidirectional communication.
2. **Driver Abstraction:** A single application can switch between Reverb, Pusher, and Ably by changing configuration.
3. **Channel-Based Pub/Sub:** Events are routed by channel name; clients subscribe to relevant channels only.
4. **Authentication Handshake:** Private/presence subscriptions require an HTTP authorization call before the WebSocket subscription completes.
5. **Connection Lifecycle Management:** Reconnect strategies, heartbeats, and state monitoring ensure reliable delivery.
6. **Horizontal Scalability:** Reverb nodes can be scaled horizontally; Redis pub/sub fans out messages across nodes.
7. **Client-Side Event Handling:** Laravel Echo abstracts subscription, whisper, and notification APIs.
8. **Separation of Concerns:** Web tier, WebSocket tier, and queue tier are independently scalable.

### Prerequisites

- Laravel 11.x or 12.x with `install:broadcasting` executed
- A broadcasting driver: Laravel Reverb (self-hosted), Pusher Channels, or Ably
- Redis (for scaling Reverb and for queue connections)
- Node.js and npm for Laravel Echo and pusher-js
- A queue worker for queued broadcast events
- Supervisor or systemd for process management in production
- A reverse proxy (Nginx, Caddy) with WebSocket upgrade support
- Authentication configured (Sanctum, session, or custom guard)
- Laravel Horizon (optional, for queue observability)
- Basic understanding of WebSocket protocol (RFC 6455)

### Related Programming Areas

- **Real-Time Networking:** WebSocket protocol, HTTP upgrade handshake, TLS.
- **Message Brokers and Pub/Sub:** Redis pub/sub, Pusher Channels, Ably.
- **Authentication and Security:** CSRF, signed tokens, HMAC signatures, channel authorization.
- **Load Balancing and Infrastructure:** Reverse proxies, sticky sessions, horizontal scaling.
- **Process Supervision:** Supervisor, systemd, container orchestration.
- **Client-Side JavaScript:** Laravel Echo, pusher-js, reconnection logic.
- **Queue Systems:** Laravel Queues, Horizon, Redis.

### Core Concepts / Features

1. Event Broadcasting (Configuring drivers in `config/broadcasting.php`)
2. Client Subscriptions (Laravel Echo + pusher-js integration)
3. Authentication (CSRF, custom endpoints, signature generation)
4. Connection Management (Reconnect strategies, state monitoring, heartbeats)
5. Enhanced: Horizon and Reverb Scaling (Horizontal scaling with Redis, load balancers, Supervisor)
6. Enhanced: Client-to-Client Whispering (`.whisper()` and `.listenForWhisper()`)


## 1. Event Broadcasting (Configuring Drivers like Laravel Reverb, Pusher, or Ably within `config/broadcasting.php`)

### Definitions

**Core Definition:** Event broadcasting driver configuration is the process of selecting and configuring the transport mechanism (Laravel Reverb, Pusher Channels, Ably, or log/null for development) that Laravel uses to deliver broadcast events to WebSocket clients.

**Technical Definition:** The `config/broadcasting.php` file defines a `default` driver and a `connections` array, each entry describing a driver-specific configuration. Laravel instantiates the corresponding `Broadcaster` implementation (e.g., `PusherBroadcaster`, `AblyBroadcaster`, `ReverbBroadcaster`) via the `BroadcastManager`, which reads credentials and options from environment variables. The `BroadcastServiceProvider` registers the manager, loads `routes/channels.php`, and exposes the `/broadcasting/auth` endpoint.

**Beginner-Friendly Explanation:** Choosing a broadcasting driver is like choosing how to send mail—by local courier (Reverb, self-hosted), by a commercial service (Pusher, Ably), or by writing in a diary (log driver for development). The `config/broadcasting.php` file is your address book and preferences for each method.

### Purposes

- To select the transport mechanism for broadcast events.
- To configure credentials and endpoints per environment.
- To support multiple drivers simultaneously (e.g., Reverb in production, log in testing).
- To decouple application code from the transport provider.
- To enable zero-downtime migration between drivers.
- To support local development without external services.
- To configure driver-specific options (TLS, cluster, timeouts, encryption).

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// config/broadcasting.php

return [

    /*
    |--------------------------------------------------------------------------
    | Default Broadcaster
    |--------------------------------------------------------------------------
    |
    | Supported: "reverb", "pusher", "ably", "redis", "log", "null"
    |
    */

    'default' => env('BROADCAST_CONNECTION', 'null'),

    /*
    |--------------------------------------------------------------------------
    | Broadcast Connections
    |--------------------------------------------------------------------------
    */

    'connections' => [

        'reverb' => [
            'driver' => 'reverb',
            'key' => env('REVERB_APP_KEY'),
            'secret' => env('REVERB_APP_SECRET'),
            'app_id' => env('REVERB_APP_ID'),
            'options' => [
                'host' => env('REVERB_HOST'),
                'port' => env('REVERB_PORT', 443),
                'scheme' => env('REVERB_SCHEME', 'https'),
                'useTLS' => env('REVERB_SCHEME', 'https') === 'https',
            ],
            'client_options' => [
                // Guzzle client options: https://docs.guzzlephp.org/en/stable/request-options.html
            ],
        ],

        'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'cluster' => env('PUSHER_APP_CLUSTER'),
                'host' => env('PUSHER_HOST') ?: 'api-' . env('PUSHER_APP_CLUSTER', 'mt1') . '.pusher.com',
                'port' => env('PUSHER_PORT', 443),
                'scheme' => env('PUSHER_SCHEME', 'https'),
                'encrypted' => true,
                'useTLS' => env('PUSHER_SCHEME', 'https') === 'https',
            ],
            'client_options' => [],
        ],

        'ably' => [
            'driver' => 'ably',
            'key' => env('ABLY_KEY'),
        ],

        'log' => [
            'driver' => 'log',
        ],

        'null' => [
            'driver' => 'null',
        ],

    ],

];
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `default` | Selects the active driver (via `BROADCAST_CONNECTION`) |
| `connections.reverb` | Self-hosted WebSocket server config |
| `connections.pusher` | Pusher Channels SaaS config |
| `connections.ably` | Ably SaaS config |
| `connections.log` | Logs broadcasts (dev/testing) |
| `connections.null` | Discards broadcasts (testing) |
| `key` / `secret` / `app_id` | Credentials for the driver |
| `options.host` / `port` / `scheme` | Network endpoints |
| `options.useTLS` / `encrypted` | Transport security |

#### Syntax Rules

1. `config/broadcasting.php` **must** return an array with `default` and `connections` keys.
2. Each connection entry **must** include a `driver` key matching a supported driver name.
3. Credentials should be read from environment variables via `env()`, never hard-coded.
4. The `default` driver is selected via `BROADCAST_CONNECTION` in `.env`.
5. For Reverb, `REVERB_APP_KEY`, `REVERB_APP_SECRET`, and `REVERB_APP_ID` are **required**.
6. For Pusher, `PUSHER_APP_KEY`, `PUSHER_APP_SECRET`, `PUSHER_APP_ID`, and `PUSHER_APP_CLUSTER` are **required**.
7. For Ably, `ABLY_KEY` is **required**.
8. The `log` driver is recommended for local development when no WebSocket server is running.
9. The `null` driver is recommended for automated tests to suppress broadcasts.

#### Constraints and Limitations

- **Driver-Specific Payload Limits:** Pusher limits messages to 10KB; Ably to 16KB; Reverb's limit is configurable but bounded by PHP memory.
- **Driver-Specific Rate Limits:** Pusher free tier: 200k messages/day; Ably free tier: 6M messages/month. Reverb has no built-in rate limit but is bounded by server capacity.
- **TLS Requirement:** Production WebSocket connections **must** use TLS (`wss://`); browsers block mixed content.
- **Config Caching:** After changing `config/broadcasting.php`, run `php artisan config:clear` or `php artisan config:cache`.
- **Credential Exposure:** The `key` is exposed to the client (via Echo config); the `secret` **must never** be exposed.
- **Single Default Driver:** Only one driver is active at a time per environment; multi-driver broadcasting requires custom code.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Configuring Laravel Reverb (Self-Hosted)

**Step-by-Step Setup Guide:**

1. Install Reverb: `php artisan install:broadcasting --reverb`
2. This creates `config/reverb.php`, `config/broadcasting.php`, and updates `.env`.
3. Start Reverb: `php artisan reverb:start`
4. Start the queue worker: `php artisan queue:work`
5. Configure Echo on the client to connect to Reverb.

**Complete Executable Code:**

```bash
# .env — Reverb configuration

BROADCAST_CONNECTION=reverb

REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
REVERB_HOST="localhost"
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"
```

```php
<?php
// config/reverb.php (excerpt — generated by install:broadcasting)

return [
    'default' => env('REVERB_SERVER', 'reverb'),

    'servers' => [
        'reverb' => [
            'host' => env('REVERB_SERVER_HOST', '0.0.0.0'),
            'port' => env('REVERB_SERVER_PORT', 8080),
            'hostname' => env('REVERB_HOST'),
            'options' => [
                'tls' => [],
            ],
            'max_request_size' => env('REVERB_MAX_REQUEST_SIZE', 10_000),
            'scaling' => [
                'enabled' => env('REVERB_SCALING_ENABLED', false),
                'channel' => env('REVERB_SCALING_CHANNEL', 'reverb'),
                'server' => [
                    'url' => env('REDIS_URL'),
                    'host' => env('REDIS_HOST', '127.0.0.1'),
                    'port' => env('REDIS_PORT', '6379'),
                    'password' => env('REDIS_PASSWORD'),
                    'database' => env('REDIS_DB', '0'),
                ],
            ],
            'apps' => [
                [
                    'key' => env('REVERB_APP_KEY'),
                    'secret' => env('REVERB_APP_SECRET'),
                    'app_id' => env('REVERB_APP_ID'),
                    'options' => [
                        'host' => env('REVERB_HOST'),
                        'port' => env('REVERB_PORT', 443),
                        'scheme' => env('REVERB_SCHEME', 'https'),
                        'useTLS' => env('REVERB_SCHEME', 'https') === 'https',
                    ],
                    'allowed_origins' => ['*'],
                    'ping_interval' => env('REVERB_APP_PING_INTERVAL', 60),
                    'activity_timeout' => env('REVERB_APP_ACTIVITY_TIMEOUT', 30),
                    'max_message_size' => env('REVERB_APP_MAX_MESSAGE_SIZE', 10_000),
                ],
            ],
        ],
    ],
];
```

```bash
# Start Reverb server
php artisan reverb:start --host=0.0.0.0 --port=8080

# In a separate terminal, start the queue worker
php artisan queue:work --queue=broadcasts,default
```

```javascript
// resources/js/bootstrap.js — Echo connecting to Reverb

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

**Expected Output:**

- Reverb starts on port 8080 and logs connection attempts.
- The queue worker processes broadcast jobs.
- Clients connect to `ws://localhost:8080/app/my-app-key`.
- Broadcast events are delivered in real time.

**Why This Code Produces That Result:**

- `BROADCAST_CONNECTION=reverb` selects the Reverb driver.
- `config/reverb.php` defines the server, apps, and scaling settings.
- Reverb authenticates clients using the app key/secret.
- Echo connects to Reverb's WebSocket endpoint using the same credentials.

#### Example 2: Configuring Pusher Channels (SaaS)

```bash
# .env — Pusher configuration

BROADCAST_CONNECTION=pusher

PUSHER_APP_ID=123456
PUSHER_APP_KEY=abcdef123456
PUSHER_APP_SECRET=secret123456
PUSHER_APP_CLUSTER=mt1
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

```javascript
// resources/js/bootstrap.js — Echo connecting to Pusher

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true,
});
```

**Expected Output:**

- Clients connect to Pusher's WebSocket endpoint for the specified cluster.
- Broadcast events are delivered via Pusher's infrastructure.
- Pusher's dashboard shows connection counts, message counts, and errors.

**Why This Code Produces That Result:**

- `BROADCAST_CONNECTION=pusher` selects the Pusher driver.
- Pusher's credentials are read from environment variables.
- Echo uses the same credentials to connect to Pusher's WebSocket endpoint.
- Pusher handles scaling, redundancy, and delivery guarantees.

#### Example 3: Configuring Ably

```bash
# .env — Ably configuration

BROADCAST_CONNECTION=ably
ABLY_KEY=your-ably-api-key

VITE_ABLY_KEY="${ABLY_KEY}"
```

```javascript
// resources/js/bootstrap.js — Echo connecting to Ably

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

// Ably exposes a Pusher-compatible endpoint
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

**Expected Output:**

- Clients connect to Ably's Pusher-compatible endpoint.
- Ably handles message delivery, presence, and scaling.
- Ably's dashboard shows real-time metrics.

**Why This Code Produces That Result:**

- `BROADCAST_CONNECTION=ably` selects Ably's broadcaster.
- Ably provides a Pusher-compatible protocol, so Echo uses the `pusher` broadcaster with Ably's endpoint.
- Ably's API key authenticates the connection.

### Real-World Cases

**Case 1: Startup MVP on Pusher**

A startup launches an MVP using Pusher Channels to avoid managing WebSocket infrastructure. As the product scales, they migrate to self-hosted Reverb to reduce costs while keeping the same Laravel code.

**Case 2: Enterprise Self-Hosted with Reverb**

A regulated enterprise hosts Laravel Reverb on-premises to keep all data within its network. Reverb scales horizontally with Redis pub/sub and runs behind an Nginx load balancer.

**Case 3: Global App on Ably**

A globally distributed app uses Ably for edge delivery across continents, leveraging Ably's global network to minimize latency.

### References

- Laravel Broadcasting: Configuration - https://laravel.com/docs/12.x/broadcasting#configuration
- Laravel Reverb Documentation - https://laravel.com/docs/12.x/reverb
- Pusher Channels Documentation - https://pusher.com/docs/channels/
- Ably Documentation - https://ably.com/docs
- Laravel Broadcasting: Driver Prerequisites - https://laravel.com/docs/12.x/broadcasting#driver-prerequisites


## 2. Client Subscriptions (Integrating Laravel Echo with pusher-js to Listen for Backend Events)

### Definitions

**Core Definition:** Client subscriptions are the browser-side mechanism by which a client connects to the WebSocket server, authorizes access to channels, and registers callbacks that fire when the server broadcasts events on those channels.

**Technical Definition:** Laravel Echo is a JavaScript library that wraps `pusher-js` (or a compatible connector) and provides a fluent API (`Echo.channel()`, `Echo.private()`, `Echo.join()`) for subscribing to channels, listening for events, and handling connection lifecycle. Echo sends an HTTP POST to `/broadcasting/auth` for private and presence channels to obtain a signed token, then subscribes on the underlying WebSocket connection. Event names are matched by the client using the dot-notated class name or the value returned by `broadcastAs()`.

**Beginner-Friendly Explanation:** Laravel Echo is like a radio receiver with preset stations. You tell it which "station" (channel) to tune into, and it plays the "songs" (events) that the backend broadcasts. Private stations require a password (authorization); Echo fetches that password automatically before tuning in.

### Purposes

- To connect a browser to the WebSocket server.
- To subscribe to public, private, and presence channels.
- To register callbacks that fire when events arrive.
- To handle authorization for private/presence channels automatically.
- To provide a consistent API across broadcasting drivers.
- To manage connection lifecycle (connect, disconnect, error, reconnect).
- To support advanced features like whispers and notifications.

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// resources/js/bootstrap.js — Echo initialization

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb', // or 'pusher', 'ably'
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
    authEndpoint: '/broadcasting/auth',
    auth: {
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
    },
});
```

```javascript
// Subscribing to channels

// Public channel
Echo.channel('system-alerts')
    .listen('.SystemAlert', (data) => { /* ... */ });

// Private channel
Echo.private(`orders.${orderId}`)
    .listen('.OrderShipped', (data) => { /* ... */ });

// Presence channel
Echo.join(`chat.${roomId}`)
    .here((users) => { /* ... */ })
    .joining((user) => { /* ... */ })
    .leaving((user) => { /* ... */ })
    .listen('.MessageSent', (data) => { /* ... */ });

// Broadcast notifications
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => { /* ... */ });
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `new Echo({...})` | Initializes Echo with driver config |
| `broadcaster` | Selects `reverb`, `pusher`, or `ably` |
| `key` | Public app key (safe to expose) |
| `wsHost` / `wsPort` | WebSocket server endpoint |
| `forceTLS` | Enforces `wss://` |
| `authEndpoint` | URL for channel authorization (default `/broadcasting/auth`) |
| `auth.headers` | Headers sent with auth requests (CSRF) |
| `Echo.channel()` | Subscribes to a public channel |
| `Echo.private()` | Subscribes to a private channel |
| `Echo.join()` | Subscribes to a presence channel |
| `.listen()` | Registers an event callback |
| `.notification()` | Registers a notification callback |
| `.here()` / `.joining()` / `.leaving()` | Presence callbacks |

#### Syntax Rules

1. `window.Pusher` **must** be set before instantiating Echo when using the `pusher` or `reverb` broadcaster.
2. The `key` is public and safe to expose; the `secret` **must never** be sent to the client.
3. Private and presence subscriptions trigger an HTTP POST to `authEndpoint` (default `/broadcasting/auth`).
4. Event names in `.listen()` are prefixed with `.` when using `broadcastAs()` or the dot-notated class name (e.g., `.App.Events.OrderShipped`).
5. Without a leading `.`, the event name is treated as a raw Pusher event (no Laravel namespace).
6. Presence callbacks (`.here()`, `.joining()`, `.leaving()`) **must** be registered before `.listen()` if you want to catch all events.
7. Multiple `.listen()` calls on the same channel are supported.
8. Echo maintains a single WebSocket connection per page; multiple channel subscriptions share it.

#### Constraints and Limitations

- **Single Connection per Page:** Echo uses one WebSocket connection; all channels multiplex over it. A connection failure affects all channels.
- **Auth Endpoint Latency:** Private/presence subscriptions require an HTTP round-trip before the WebSocket subscription completes; this adds latency to first subscription.
- **Event Name Matching:** Event names are case-sensitive; mismatches silently fail (no error is thrown).
- **Channel Leave:** Failing to leave channels on component unmount can cause memory leaks and duplicate listeners.
- **CSRF for SPAs:** SPAs using token auth must send the bearer token in the `Authorization` header; session-based SPAs must send the CSRF token.
- **Broadcaster Compatibility:** Ably uses a Pusher-compatible protocol; not all Echo features are available on all drivers.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Full Echo Setup with Private and Presence Channels

**Step-by-Step Setup Guide:**

1. Install Echo and pusher-js: `npm install --save-dev laravel-echo pusher-js`
2. Configure Echo in `resources/js/bootstrap.js`.
3. Subscribe to channels in your components.
4. Register event listeners and update the UI.
5. Leave channels on component unmount.

**Complete Executable Code:**

```javascript
// resources/js/bootstrap.js

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
    auth: {
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]')?.content,
        },
    },
});
```

```javascript
// resources/js/order-tracker.js

const orderId = window.orderConfig.orderId;
const userId = window.appConfig.userId;

// Subscribe to a private channel for order updates
const orderChannel = Echo.private(`orders.${orderId}`);

orderChannel.listen('.OrderShipped', (data) => {
    console.log('Order shipped:', data);
    document.getElementById('order-status').textContent = 'Shipped';
    document.getElementById('tracking-number').textContent = data.tracking_number;
});

orderChannel.listen('.OrderDelivered', (data) => {
    document.getElementById('order-status').textContent = 'Delivered';
    showToast('Your order has been delivered!');
});

// Subscribe to a presence channel for live support
const supportChannel = Echo.join(`support.${orderId}`)
    .here((users) => {
        console.log('Support agents online:', users.length);
        updateAgentCount(users.length);
    })
    .joining((user) => {
        showToast(`${user.name} joined the support chat`);
    })
    .leaving((user) => {
        showToast(`${user.name} left the support chat`);
    })
    .listen('.SupportMessage', (data) => {
        appendChatMessage(data);
    });

// Clean up on page unload
window.addEventListener('beforeunload', () => {
    Echo.leave(`orders.${orderId}`);
    Echo.leave(`support.${orderId}`);
});
```

**Expected Output:**

- The client connects to Reverb on page load.
- The private `orders.{orderId}` channel is authorized and subscribed.
- Order events update the UI in real time.
- The presence channel shows support agents online; joining/leaving triggers toasts.
- On page unload, channels are left cleanly.

**Why This Code Produces That Result:**

- Echo automatically POSTs to `/broadcasting/auth` for the private and presence channels.
- The CSRF token in `auth.headers` satisfies Laravel's CSRF protection.
- `.listen()` callbacks fire when events with matching names arrive.
- `Echo.leave()` unsubscribes and frees resources.

#### Example 2: Listening for Broadcast Notifications

```javascript
// resources/js/notifications.js

const userId = window.appConfig.userId;

Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        // notification contains the payload from toBroadcast()
        console.log('Notification received:', notification);

        // Update the bell badge
        const badge = document.getElementById('notification-badge');
        badge.textContent = parseInt(badge.textContent || '0', 10) + 1;
        badge.classList.remove('hidden');

        // Show a toast
        showToast(notification.message, notification.sender?.avatar);
    });

// Mark all as read via HTTP (not broadcast)
document.getElementById('mark-all-read').addEventListener('click', async () => {
    await fetch('/notifications/mark-all-read', {
        method: 'POST',
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
    });
    document.getElementById('notification-badge').classList.add('hidden');
});
```

**Expected Output:**

- When a broadcast notification is delivered, the bell badge increments and a toast appears.
- Clicking "mark all read" clears the badge via an HTTP request.

**Why This Code Produces That Result:**

- Laravel's notification system broadcasts on `App.Models.User.{id}`.
- Echo's `.notification()` callback receives the payload from `toBroadcast()`.
- The badge is client-side state; marking as read is a separate HTTP operation.

#### Example 3: Dynamically Leaving and Rejoining Channels

```javascript
// resources/js/chat-room-switcher.js

let currentChannel = null;

function switchRoom(roomId) {
    // Leave the previous channel to prevent duplicate listeners
    if (currentChannel) {
        Echo.leave(`chat.${currentChannel}`);
    }

    currentChannel = roomId;

    Echo.join(`chat.${roomId}`)
        .here((users) => renderOnlineUsers(users))
        .joining((user) => addOnlineUser(user))
        .leaving((user) => removeOnlineUser(user))
        .listen('.MessageSent', (data) => appendMessage(data));
}

// Switch rooms via UI
document.querySelectorAll('[data-room-id]').forEach(btn => {
    btn.addEventListener('click', () => switchRoom(btn.dataset.roomId));
});
```

**Expected Output:**

- Switching rooms leaves the old channel and joins the new one.
- The online user list updates per room.
- No duplicate listeners accumulate.

**Why This Code Produces That Result:**

- `Echo.leave()` unsubscribes from the previous channel.
- Re-joining a new channel triggers fresh `here`/`joining`/`leaving` callbacks.
- Without leaving, listeners from the old room would still fire, causing bugs.

### Real-World Cases

**Case 1: E-Commerce Order Tracking**

A customer's order page subscribes to `orders.{orderId}`. Status changes (shipped, out for delivery, delivered) update the UI live. A presence channel shows the assigned support agent.

**Case 2: SaaS Notification Bell**

A SaaS app subscribes every authenticated user to `App.Models.User.{id}` for broadcast notifications. The bell badge updates live, and clicking a notification marks it read.

**Case 3: Multi-Room Chat**

A chat app switches between rooms dynamically, leaving and joining presence channels as the user navigates. Online user lists and messages update per room.

### References

- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- Laravel Broadcasting: Listening for Events - https://laravel.com/docs/12.x/broadcasting#listening-for-events
- Laravel Broadcasting: Notifications - https://laravel.com/docs/12.x/broadcasting#notifications
- pusher-js Documentation - https://pusher.com/docs/channels/using_channels/client-api/


## 3. Authentication (CSRF Token Passing, Custom Auth Endpoints, Signature Generation)

### Definitions

**Core Definition:** WebSocket authentication is the process of verifying a client's identity and channel access rights before allowing subscription to private or presence channels, using an HTTP authorization endpoint that returns a signed token.

**Technical Definition:** Laravel exposes a POST route `/broadcasting/auth` (registered by `Broadcast::routes()`) that accepts a `channel_name` and `socket_id`, resolves the authenticated user via the configured guard, executes the channel's authorization closure from `routes/channels.php`, and returns a JSON payload containing an HMAC-SHA256 signature computed over `socket_id:channel_name` using the driver's secret. The WebSocket server verifies this signature before permitting the subscription. CSRF protection applies to the auth endpoint in session-based applications; token-based (Sanctum) applications send a bearer token in the `Authorization` header. Custom auth endpoints can be registered via `Broadcast::routes(['prefix' => 'api', 'middleware' => ['auth:sanctum']])`.

**Beginner-Friendly Explanation:** Before entering a private room, you must show ID at the front desk. The front desk is `/broadcasting/auth`. You tell it which room you want and your socket ID; it checks the rules and, if you're allowed, hands you a signed pass. The WebSocket server checks the pass's signature before letting you in. The signature prevents forgery.

### Purposes

- To verify the identity of the user requesting a channel subscription.
- To enforce channel-specific authorization rules.
- To generate cryptographic signatures that the WebSocket server can verify.
- To support both session-based and token-based authentication.
- To protect against CSRF attacks on the auth endpoint.
- To allow custom auth endpoints for SPAs, mobile apps, and APIs.
- To integrate with Laravel's existing authentication guards and middleware.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Providers/BroadcastServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Broadcast;
use Illuminate\Support\ServiceProvider;

class BroadcastServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Default auth route: /broadcasting/auth (web middleware)
        Broadcast::routes();

        // Custom auth route: /api/broadcasting/auth (auth:sanctum)
        // Broadcast::routes([
        //     'prefix' => 'api',
        //     'middleware' => ['auth:sanctum'],
        // ]);

        require base_path('routes/channels.php');
    }
}
```

```javascript
// resources/js/bootstrap.js — CSRF token passing

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    // ...
    authEndpoint: '/broadcasting/auth',
    auth: {
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
    },
});
```

```javascript
// Token-based auth (Sanctum) — bearer token in Authorization header

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    // ...
    authEndpoint: '/api/broadcasting/auth',
    auth: {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('api_token')}`,
            Accept: 'application/json',
        },
    },
});
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `Broadcast::routes()` | Registers `/broadcasting/auth` (web middleware) |
| `Broadcast::routes(['prefix' => 'api', 'middleware' => ['auth:sanctum']])` | Custom auth route for SPAs |
| `authEndpoint` | URL Echo POSTs to for authorization |
| `auth.headers['X-CSRF-TOKEN']` | CSRF token for session-based auth |
| `auth.headers['Authorization']` | Bearer token for token-based auth |
| HMAC-SHA256 signature | Computed server-side, verified by WebSocket server |
| `socket_id` | Unique per-connection identifier |
| `channel_name` | The channel being authorized |

#### Syntax Rules

1. `Broadcast::routes()` **must** be called in `BroadcastServiceProvider::boot()`.
2. The default auth route uses the `web` middleware group (session + CSRF).
3. Custom auth routes can specify `prefix` and `middleware` options.
4. The auth endpoint accepts POST requests with `channel_name` and `socket_id`.
5. The response contains `auth` (the signature) and, for presence channels, `channel_data` (user metadata).
6. For session-based auth, the CSRF token **must** be sent in `X-CSRF-TOKEN` or `X-XSRF-TOKEN`.
7. For token-based auth (Sanctum), the `Authorization: Bearer <token>` header **must** be sent, and CSRF is not required.
8. The signature is HMAC-SHA256 over `socket_id:channel_name` using the app secret.
9. The WebSocket server **must** have the same secret to verify signatures.

#### Constraints and Limitations

- **Secret Sharing:** The WebSocket server and Laravel app **must** share the same secret; mismatches cause auth failures.
- **CSRF for SPAs:** SPAs using session cookies **must** obtain and send the CSRF token; Sanctum token auth avoids this.
- **Auth Endpoint Latency:** Each private/presence subscription triggers an HTTP request; batching or caching can reduce overhead.
- **Custom Guards:** If using a non-default guard, you must configure `Broadcast::routes()` middleware and possibly a custom guard resolver.
- **Presence Channel Data:** The `channel_data` for presence channels is signed and includes the user's metadata; it must be JSON-encoded.
- **Signature Expiry:** Signatures are tied to the `socket_id`, which changes on reconnect; clients must re-authorize after reconnecting.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Session-Based Auth with CSRF

**Step-by-Step Setup Guide:**

1. Ensure `Broadcast::routes()` is called in `BroadcastServiceProvider`.
2. Add a `<meta name="csrf-token">` tag to your Blade layout.
3. Configure Echo with the CSRF token in `auth.headers`.
4. Subscribe to a private channel and verify authorization succeeds.

**Complete Executable Code:**

```blade
{{-- resources/views/layouts/app.blade.php --}}
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <meta name="user-id" content="{{ auth()->id() }}">
</head>
```

```php
<?php
// app/Providers/BroadcastServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Broadcast;
use Illuminate\Support\ServiceProvider;

class BroadcastServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Default auth route with web middleware (session + CSRF)
        Broadcast::routes();

        require base_path('routes/channels.php');
    }
}
```

```javascript
// resources/js/bootstrap.js

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    auth: {
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
    },
});
```

```javascript
// Subscribing to a private channel (auth happens automatically)

Echo.private(`orders.${window.orderConfig.orderId}`)
    .listen('.OrderShipped', (data) => {
        console.log('Order shipped:', data);
    });
```

**Expected Output:**

- Echo POSTs to `/broadcasting/auth` with `channel_name` and `socket_id`.
- The request includes the CSRF token; Laravel validates it.
- The auth endpoint executes the channel closure and returns a signature.
- The WebSocket server verifies the signature and subscribes the client.

**Why This Code Produces That Result:**

- The CSRF token in `auth.headers` satisfies Laravel's `VerifyCsrfToken` middleware.
- `Broadcast::routes()` registers the auth route with the `web` middleware group.
- The channel closure in `routes/channels.php` authorizes the user.

#### Example 2: Token-Based Auth with Sanctum

```php
<?php
// app/Providers/BroadcastServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Broadcast;
use Illuminate\Support\ServiceProvider;

class BroadcastServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Custom auth route for API/SPA clients using Sanctum tokens
        Broadcast::routes([
            'prefix' => 'api',
            'middleware' => ['auth:sanctum'],
        ]);

        require base_path('routes/channels.php');
    }
}
```

```javascript
// resources/js/bootstrap.js

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    authEndpoint: '/api/broadcasting/auth',
    auth: {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('sanctum_token')}`,
            Accept: 'application/json',
        },
    },
});
```

**Expected Output:**

- Echo POSTs to `/api/broadcasting/auth` with the bearer token.
- Sanctum authenticates the request; CSRF is not required.
- The channel closure executes, and a signature is returned.
- The WebSocket server verifies the signature and subscribes the client.

**Why This Code Produces That Result:**

- The custom auth route uses `auth:sanctum` middleware instead of `web`.
- The bearer token authenticates the user without a session.
- `Accept: application/json` ensures Laravel returns JSON errors instead of redirects.

#### Example 3: Custom Auth Endpoint with Additional Logic

```php
<?php
// routes/api.php

use App\Http\Controllers\BroadcastAuthController;

Route::post('/custom/broadcasting/auth', [BroadcastAuthController::class, 'authenticate'])
    ->middleware(['auth:sanctum']);
```

```php
<?php
// app/Http/Controllers/BroadcastAuthController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Broadcast;

class BroadcastAuthController extends Controller
{
    /**
     * Custom auth endpoint that adds logging and rate limiting
     * before delegating to Laravel's built-in broadcaster.
     */
    public function authenticate(Request $request)
    {
        // Rate limit: 60 auth attempts per minute per user
        \RateLimiter::attempt(
            'broadcast-auth:' . $request->user()->id,
            60,
            function () {}
        );

        // Log the authorization attempt for auditing
        \Log::info('Broadcast auth attempt', [
            'user_id' => $request->user()->id,
            'channel' => $request->input('channel_name'),
            'socket_id' => $request->input('socket_id'),
        ]);

        // Delegate to Laravel's broadcaster
        return Broadcast::auth($request);
    }
}
```

**Expected Output:**

- Echo POSTs to `/api/custom/broadcasting/auth`.
- The controller logs the attempt, enforces rate limits, then delegates to `Broadcast::auth()`.
- The signature is returned exactly as Laravel would produce.

**Why This Code Produces That Result:**

- `Broadcast::auth($request)` is the underlying method that the default route calls.
- Wrapping it allows custom middleware logic (rate limiting, logging, IP allow-listing).
- The response format is unchanged, so Echo and the WebSocket server accept it.

### Real-World Cases

**Case 1: SPA with Sanctum**

A Vue SPA uses Sanctum tokens for API auth and a custom `/api/broadcasting/auth` endpoint with `auth:sanctum` middleware. Bearer tokens replace CSRF.

**Case 2: Session-Based Blade App**

A traditional Blade app uses the default `/broadcasting/auth` route with CSRF tokens embedded in the layout's meta tag.

**Case 3: Mobile App with Passport**

A mobile app uses Passport OAuth tokens for auth and a custom auth endpoint with `auth:api` middleware.

### References

- Laravel Broadcasting: Authorizing Channels - https://laravel.com/docs/12.x/broadcasting#authorizing-channels
- Laravel Broadcasting: Defining Authorization Routes - https://laravel.com/docs/12.x/broadcasting#defining-authorization-routes
- Laravel Sanctum Documentation - https://laravel.com/docs/12.x/sanctum
- Laravel Echo: Authentication - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- Pusher Channels: Authentication - https://pusher.com/docs/channels/server_api/authorizing-users/


## 4. Connection Management (Reconnect Strategies, State Monitoring, Heartbeat Pings)

### Definitions

**Core Definition:** Connection management is the set of client- and server-side strategies that maintain a reliable WebSocket connection, detect failures, automatically reconnect, monitor connection state, and keep the connection alive with periodic heartbeats.

**Technical Definition:** The WebSocket connection lifecycle includes `connecting`, `connected`, `disconnected`, `unavailable`, and `failed` states, exposed by `pusher-js` via `Echo.connector.pusher.connection`. Reconnect strategies use exponential backoff with jitter (configurable via `activityTimeout` and `pongTimeout`). Heartbeats are `ping`/`pong` frames exchanged at intervals (default 30s) to detect dead connections. Server-side, Laravel Reverb exposes `ping_interval` and `activity_timeout` settings. On reconnect, clients **must** re-authorize private/presence channels because the `socket_id` changes.

**Beginner-Friendly Explanation:** A WebSocket connection is like a phone call. If the line goes silent, you say "Are you still there?" (heartbeat). If the call drops, you dial again (reconnect). If dialing fails, you wait a bit longer each time (backoff). When you reconnect, you have to re-verify your identity (re-authorization). Connection management is all the rules that keep the call alive and recover gracefully.

### Purposes

- To detect dead or half-open connections quickly.
- To automatically reconnect after network failures.
- To avoid overwhelming the server with rapid reconnect attempts.
- To monitor connection state and surface it in the UI.
- To re-authorize channels after reconnection.
- To tune heartbeat intervals for latency/bandwidth tradeoffs.
- To integrate with offline/online browser events.

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// resources/js/bootstrap.js — Connection management configuration

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],

    // Connection tuning
    activityTimeout: 30000,   // 30s: how long before considering the connection idle
    pongTimeout: 10000,       // 10s: how long to wait for a pong after a ping
    unavailableTimeout: 10000, // 10s: how long before marking connection unavailable

    // Reconnection
    // pusher-js auto-reconnects with exponential backoff by default
});
```

```javascript
// Connection state monitoring

const connection = window.Echo.connector.pusher.connection;

connection.bind('connected', () => {
    console.log('WebSocket connected');
    updateConnectionIndicator('online');
});

connection.bind('disconnected', () => {
    console.log('WebSocket disconnected');
    updateConnectionIndicator('offline');
});

connection.bind('unavailable', () => {
    console.log('WebSocket unavailable');
    updateConnectionIndicator('reconnecting');
});

connection.bind('failed', () => {
    console.log('WebSocket failed permanently');
    updateConnectionIndicator('failed');
});

connection.bind('state_change', (states) => {
    console.log(`Connection state: ${states.previous} -> ${states.current}`);
});
```

```javascript
// Re-authorization after reconnect

connection.bind('connected', () => {
    // Re-subscribe to private/presence channels
    // (socket_id changed, so auth tokens are invalid)
    if (window.currentOrderId) {
        Echo.private(`orders.${window.currentOrderId}`)
            .listen('.OrderShipped', handleOrderShipped);
    }
});
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `activityTimeout` | Idle time before considering the connection stale |
| `pongTimeout` | Time to wait for a pong after sending a ping |
| `unavailableTimeout` | Time before marking connection unavailable |
| `connection.bind('connected', ...)` | Fires on successful connection |
| `connection.bind('disconnected', ...)` | Fires on disconnect |
| `connection.bind('unavailable', ...)` | Fires when connection is unreachable |
| `connection.bind('failed', ...)` | Fires when connection fails permanently |
| `connection.bind('state_change', ...)` | Fires on any state transition |

#### Syntax Rules

1. Echo delegates connection management to the underlying `pusher-js` connector.
2. `activityTimeout` and `pongTimeout` **must** be tuned together; `pongTimeout` should be less than `activityTimeout`.
3. `connection.bind()` callbacks fire on the `pusher-js` connection object, not on Echo directly.
4. Reconnection is automatic by default; `pusher-js` uses exponential backoff.
5. On reconnect, the `socket_id` changes; clients **must** re-authorize private/presence channels.
6. `Echo.leave()` should be called before unsubscribing to prevent stale listeners.
7. Server-side Reverb `ping_interval` and `activity_timeout` settings in `config/reverb.php` control server-initiated heartbeats.

#### Constraints and Limitations

- **Reconnect Storms:** After a server restart, all clients reconnect simultaneously; exponential backoff with jitter mitigates this, but large deployments should stagger.
- **Auth Re-Trigger:** Re-authorization after reconnect requires fresh HTTP requests; if the auth endpoint is down, subscriptions fail.
- **Heartbeat Bandwidth:** Frequent pings consume bandwidth; tune based on network conditions.
- **Mobile Background:** Mobile browsers may suspend WebSocket connections when backgrounded; reconnect on `visibilitychange` is recommended.
- **Browser Limits:** Browsers limit concurrent WebSocket connections per domain (typically 200 on HTTP/1.1; effectively unlimited on HTTP/2).
- **Proxy Timeouts:** Reverse proxies (Nginx, ELB) may close idle WebSocket connections; heartbeats must be shorter than proxy idle timeouts.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Connection State Indicator in the UI

**Step-by-Step Setup Guide:**

1. Configure Echo with connection tuning options.
2. Bind to connection state events.
3. Update a UI indicator (e.g., a colored dot) based on state.
4. Re-authorize channels on reconnect.

**Complete Executable Code:**

```javascript
// resources/js/connection-monitor.js

const connection = window.Echo.connector.pusher.connection;
const indicator = document.getElementById('connection-indicator');

const states = {
    connected:    { color: 'green',  label: 'Connected' },
    connecting:   { color: 'yellow', label: 'Connecting…' },
    disconnected: { color: 'gray',   label: 'Disconnected' },
    unavailable:  { color: 'orange', label: 'Reconnecting…' },
    failed:       { color: 'red',    label: 'Connection Failed' },
};

function updateIndicator(state) {
    const info = states[state] || { color: 'gray', label: state };
    indicator.style.backgroundColor = info.color;
    indicator.title = info.label;
    indicator.textContent = info.label;
}

// Initial state
updateIndicator(connection.state);

// Bind to state changes
connection.bind('state_change', (states) => {
    updateIndicator(states.current);
    console.log(`Connection: ${states.previous} -> ${states.current}`);
});

// Log specific events
connection.bind('connected', () => console.log('Connected'));
connection.bind('disconnected', () => console.log('Disconnected'));
connection.bind('unavailable', () => console.log('Unavailable'));
connection.bind('failed', () => console.log('Failed'));
```

**Expected Output:**

- The indicator shows "Connected" (green) when the WebSocket is up.
- On disconnect, it shows "Reconnecting…" (orange), then "Connected" again.
- If the server is unreachable, it shows "Connection Failed" (red).

**Why This Code Produces That Result:**

- `connection.state` provides the current state.
- `state_change` events update the UI on every transition.
- `pusher-js` handles reconnection automatically; the UI reflects its state.

#### Example 2: Re-Authorizing Channels After Reconnect

```javascript
// resources/js/reconnect-handler.js

const connection = window.Echo.connector.pusher.connection;
const activeSubscriptions = new Map();

/**
 * Register a channel subscription with automatic re-subscription
 * on reconnect. The socket_id changes on reconnect, so auth
 * tokens must be regenerated.
 */
function subscribePrivate(channelName, eventName, handler) {
    activeSubscriptions.set(channelName, { eventName, handler });

    Echo.private(channelName).listen(eventName, handler);
}

function resubscribeAll() {
    console.log('Reconnected — re-authorizing channels');
    activeSubscriptions.forEach(({ eventName, handler }, channelName) => {
        // Echo automatically re-authorizes because the channel
        // was already subscribed; but to be safe, leave and re-join.
        Echo.leave(channelName);
        Echo.private(channelName).listen(eventName, handler);
    });
}

connection.bind('connected', () => {
    // Only resubscribe if this is a reconnect, not the initial connect
    if (window.__hasConnectedBefore) {
        resubscribeAll();
    }
    window.__hasConnectedBefore = true;
});

// Usage
subscribePrivate(`orders.${orderId}`, '.OrderShipped', (data) => {
    console.log('Order shipped:', data);
});
```

**Expected Output:**

- On initial connect, channels are subscribed normally.
- On reconnect, all registered channels are re-authorized and re-subscribed.
- Event handlers continue to fire after reconnection.

**Why This Code Produces That Result:**

- The `socket_id` changes on reconnect, invalidating old auth tokens.
- Re-subscribing triggers a fresh auth request with the new `socket_id`.
- The `activeSubscriptions` map tracks what to re-subscribe.

#### Example 3: Handling Browser Visibility and Network Events

```javascript
// resources/js/visibility-handler.js

const connection = window.Echo.connector.pusher.connection;

// When the tab becomes visible again, check the connection
document.addEventListener('visibilitychange', () => {
    if (!document.hidden) {
        console.log('Tab visible — connection state:', connection.state);
        if (connection.state !== 'connected') {
            connection.connect();
        }
    }
});

// When the browser reports online, try to reconnect immediately
window.addEventListener('online', () => {
    console.log('Browser online — reconnecting');
    connection.connect();
});

// When the browser reports offline, update the UI
window.addEventListener('offline', () => {
    console.log('Browser offline');
    updateConnectionIndicator('offline');
});
```

**Expected Output:**

- Returning to the tab triggers a connection check and reconnect if needed.
- Browser online/offline events update the UI and trigger reconnection.

**Why This Code Produces That Result:**

- Mobile and desktop browsers may suspend WebSocket connections when backgrounded.
- The `online` event fires when network connectivity is restored.
- `connection.connect()` forces a reconnect attempt.

### Real-World Cases

**Case 1: Mobile Web App with Frequent Backgrounding**

A mobile web app reconnects on `visibilitychange` and `online` events to handle iOS Safari's aggressive connection suspension.

**Case 2: Trading Dashboard with State Indicator**

A trading dashboard shows a connection status indicator so traders know when data may be stale. Reconnection is automatic with exponential backoff.

**Case 3: Customer Support Chat with Re-Authorization**

A support chat re-authorizes channels on reconnect to ensure the agent continues receiving messages after a network blip.

### References

- Laravel Echo: Connection Management - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- pusher-js: Connection States - https://pusher.com/docs/channels/using_channels/connection/
- Laravel Reverb: Server Configuration - https://laravel.com/docs/12.x/reverb#configuration
- MDN: Page Visibility API - https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API
- MDN: Navigator.onLine - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine


## 5. Enhanced: Horizon and Reverb Scaling (Horizontally Scaling WebSocket Connections Using Redis, Load Balancers, and Supervisor Processes)

### Definitions

**Core Definition:** Reverb and Horizon scaling is the practice of running multiple WebSocket server nodes and queue worker pools behind a load balancer, using Redis pub/sub to fan out messages across nodes, and Supervisor/systemd to manage processes, enabling horizontal scalability and high availability.

**Technical Definition:** Laravel Reverb supports horizontal scaling via Redis pub/sub: each Reverb node subscribes to a Redis channel (`REVERB_SCALING_CHANNEL`), and when a message is published on any node, Redis broadcasts it to all nodes, which then deliver it to their connected clients. A load balancer (Nginx, HAProxy, AWS ALB) distributes WebSocket connections across nodes. Sticky sessions are not strictly required if all nodes share the same Redis channel, but they improve locality. Laravel Horizon scales queue workers, which process broadcast jobs and publish to Redis for Reverb nodes to pick up. Supervisor ensures Reverb and Horizon processes restart on failure.

**Beginner-Friendly Explanation:** Imagine a stadium with multiple announcers. Each announcer (Reverb node) talks to the fans in their section. When one announcer has news, they radio it to a central station (Redis), and all announcers repeat it to their sections. The ticket gates (load balancer) send fans to different sections. Supervisors are the stadium managers who make sure every announcer keeps working. Horizon is the manager for the queue workers who prepare the announcements.

### Purposes

- To handle more concurrent WebSocket connections than a single server can support.
- To provide high availability (if one node fails, others continue).
- To distribute load across multiple CPU cores and servers.
- To decouple the WebSocket tier from the web tier for independent scaling.
- To ensure broadcast jobs are processed by dedicated queue workers.
- To automatically restart failed processes.
- To monitor queue and connection health via Horizon's dashboard.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# .env — Scaling configuration

BROADCAST_CONNECTION=reverb

REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
REVERB_HOST="ws.example.com"
REVERB_PORT=443
REVERB_SCHEME=https

REVERB_SCALING_ENABLED=true
REVERB_SCALING_CHANNEL=reverb

REDIS_HOST=redis.internal
REDIS_PORT=6379
REDIS_PASSWORD=secret
REDIS_DB=0

QUEUE_CONNECTION=redis
```

```php
<?php
// config/reverb.php (scaling section)

'scaling' => [
    'enabled' => env('REVERB_SCALING_ENABLED', false),
    'channel' => env('REVERB_SCALING_CHANNEL', 'reverb'),
    'server' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'port' => env('REDIS_PORT', '6379'),
        'password' => env('REDIS_PASSWORD'),
        'database' => env('REDIS_DB', '0'),
    ],
],
```

```ini
; /etc/supervisor/conf.d/reverb.conf
[program:reverb]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/artisan reverb:start --host=0.0.0.0 --port=8080
autostart=true
autorestart=true
user=www-data
numprocs=4
redirect_stderr=true
stdout_logfile=/var/log/reverb.log
stopwaitsecs=3600
```

```ini
; /etc/supervisor/conf.d/horizon.conf
[program:horizon]
process_name=%(program_name)s
command=php /var/www/html/artisan horizon
autostart=true
autorestart=true
user=www-data
redirect_stderr=true
stdout_logfile=/var/log/horizon.log
stopwaitsecs=3600
```

```nginx
# /etc/nginx/conf.d/websocket.conf

upstream reverb_backend {
    # ip_hash ensures a client sticks to one node (optional with Redis scaling)
    ip_hash;
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
    server 10.0.0.3:8080;
}

server {
    listen 443 ssl http2;
    server_name ws.example.com;

    ssl_certificate     /etc/ssl/certs/ws.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ws.example.com.key;

    location / {
        proxy_pass http://reverb_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts must exceed heartbeat interval
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `REVERB_SCALING_ENABLED=true` | Enables Redis pub/sub fan-out |
| `REVERB_SCALING_CHANNEL` | Redis channel name for cross-node messages |
| Redis server config | Connection details for scaling |
| Supervisor `numprocs=4` | Runs 4 Reverb processes per server |
| Nginx `upstream` | Load balances across Reverb nodes |
| `ip_hash` | Sticky sessions (optional with Redis scaling) |
| `proxy_read_timeout 3600s` | Prevents proxy from closing idle WebSockets |

#### Syntax Rules

1. `REVERB_SCALING_ENABLED` **must** be `true` on all Reverb nodes for cross-node delivery.
2. All Reverb nodes **must** use the same `REVERB_SCALING_CHANNEL` and Redis instance.
3. All Reverb nodes **must** share the same `REVERB_APP_KEY`, `REVERB_APP_SECRET`, and `REVERB_APP_ID`.
4. The load balancer **must** support WebSocket upgrade (`proxy_set_header Upgrade`, `Connection "upgrade"`).
5. `proxy_read_timeout` **must** exceed the heartbeat interval to prevent premature disconnects.
6. Supervisor **must** be configured to restart Reverb processes on failure (`autorestart=true`).
7. Horizon **must** be configured with dedicated queues for broadcasts to avoid head-of-line blocking.
8. Sticky sessions (`ip_hash`) are optional with Redis scaling but improve locality.

#### Constraints and Limitations

- **Redis Single Point of Failure:** If Redis fails, cross-node delivery fails. Use Redis Sentinel or Cluster for HA.
- **Message Duplication:** Redis pub/sub delivers to all subscribers; each Reverb node delivers only to its own clients, so no duplication occurs at the client level.
- **Ordering:** Redis pub/sub does not guarantee ordering across nodes; include timestamps in payloads.
- **Latency:** Cross-node fan-out adds a small latency (sub-millisecond on LAN, higher on WAN).
- **Memory per Node:** Each Reverb node holds connection state in memory; plan memory accordingly.
- **File Descriptors:** Each WebSocket connection consumes a file descriptor; increase `ulimit -n` on Reverb nodes.
- **Sticky Sessions Tradeoff:** `ip_hash` improves locality but can cause uneven distribution if clients are behind NAT.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Scaling Reverb Across Two Nodes with Redis

**Step-by-Step Setup Guide:**

1. Provision two servers (or containers) with PHP, Laravel, and Reverb.
2. Configure a shared Redis instance.
3. Set `REVERB_SCALING_ENABLED=true` on both nodes.
4. Start Reverb on both nodes via Supervisor.
5. Configure Nginx to load balance across both nodes.
6. Verify cross-node delivery with a test event.

**Complete Executable Code:**

```bash
# On Node 1 (10.0.0.1)
cd /var/www/html
php artisan reverb:start --host=0.0.0.0 --port=8080

# On Node 2 (10.0.0.2)
cd /var/www/html
php artisan reverb:start --host=0.0.0.0 --port=8080
```

```php
// Test event dispatched from the web tier (any node)

use App\Events\TestBroadcast;

TestBroadcast::dispatch('Hello from node 1');
```

```javascript
// Client connected to Node 1 via the load balancer

Echo.channel('test')
    .listen('.TestBroadcast', (data) => {
        console.log('Received:', data); // "Hello from node 1"
    });
```

**Expected Output:**

- The event is dispatched from the web tier (e.g., Node 1).
- Node 1 publishes the message to Redis on the `reverb` channel.
- Node 2 receives the message from Redis and delivers it to its connected clients.
- All clients, regardless of which node they're connected to, receive the event.

**Why This Code Produces That Result:**

- `REVERB_SCALING_ENABLED=true` makes each node subscribe to the Redis channel.
- When any node broadcasts, it publishes to Redis.
- All nodes (including the originator) receive the message and deliver to their local clients.
- This ensures a client connected to Node 2 receives an event dispatched from Node 1.

#### Example 2: Horizon with Dedicated Broadcast Queue

```php
<?php
// config/horizon.php

return [
    'environments' => [
        'production' => [
            'supervisor-broadcasts' => [
                'connection' => 'redis',
                'queue' => ['broadcasts'],
                'balance' => 'auto',
                'processes' => 10,
                'tries' => 3,
                'timeout' => 60,
            ],
            'supervisor-default' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'processes' => 5,
                'tries' => 3,
                'timeout' => 120,
            ],
        ],
    ],
];
```

```bash
# Start Horizon (supervised by Supervisor or systemd)
php artisan horizon

# Monitor via the Horizon dashboard at /horizon
```

**Expected Output:**

- Broadcast jobs are processed by 10 dedicated workers.
- Default jobs are processed by 5 separate workers.
- The Horizon dashboard shows throughput, wait times, and failures per queue.

**Why This Code Produces That Result:**

- Dedicated workers for `broadcasts` prevent broadcast jobs from being delayed by slower default jobs.
- `balance: auto` distributes jobs across workers based on load.
- Horizon's dashboard provides observability into queue health.

### Real-World Cases

**Case 1: Live Event Platform with 100k Concurrent Users**

A live event platform runs 10 Reverb nodes behind an AWS ALB, with Redis ElastiCache for scaling. Horizon processes broadcast jobs on dedicated workers. Supervisor manages Reverb processes.

**Case 2: Multi-Region SaaS with Geo-Distributed Reverb**

A global SaaS deploys Reverb nodes in multiple regions, each with a local Redis instance federated via Redis Cluster. Clients connect to the nearest region, reducing latency.

**Case 3: High-Throughput Trading Platform**

A trading platform uses Horizon with 50 dedicated broadcast workers and 5 Reverb nodes. Supervisor ensures automatic restart on failure, and Prometheus monitors connection counts and queue depth.

### References

- Laravel Reverb: Scaling - https://laravel.com/docs/12.x/reverb#scaling
- Laravel Horizon Documentation - https://laravel.com/docs/12.x/horizon
- Laravel Queues: Supervisor Configuration - https://laravel.com/docs/12.x/queues#supervisor-configuration
- Nginx: WebSocket Proxying - https://nginx.org/en/docs/http/websocket.html
- Redis Pub/Sub Documentation - https://redis.io/docs/interact/pubsub/


## 6. Enhanced: Client-to-Client Whispering (`.whisper()` and `.listenForWhisper()`)

### Definitions

**Core Definition:** Whispering is a Laravel Echo feature that allows clients subscribed to the same channel to send ephemeral messages directly to each other through the WebSocket server, without the message passing through the Laravel backend or being persisted to the database.

**Technical Definition:** `Echo.private(channel).whisper(eventName, data)` sends a `client-{eventName}` event over the WebSocket connection to the WebSocket server, which relays it to all other subscribers of the same channel (excluding the sender). Other clients listen via `.listenForWhisper(eventName, callback)`. Whispering works on private and presence channels (not public channels) and does not trigger any HTTP request, queue job, or database write. It is ideal for ephemeral signals like typing indicators, cursor movements, and read receipts.

**Beginner-Friendly Explanation:** Whispering is like passing notes in class without the teacher (the backend) seeing them. You and your classmates (other clients) are in the same room (channel). You whisper "I'm typing!" and everyone in the room hears it—but the teacher never knows, and nothing is written down. It's fast, private to the room, and doesn't clutter the database.

### Purposes

- To send ephemeral client-to-client signals without backend involvement.
- To reduce server load by avoiding HTTP requests and queue jobs.
- To reduce database writes by keeping ephemeral data out of persistence.
- To enable low-latency typing indicators in chat systems.
- To broadcast cursor positions in collaborative editors.
- To synchronize UI state (e.g., scroll position) among viewers.
- To implement "user is viewing this page" indicators.

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// Client A: Sending a whisper

Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        user_id: currentUserId,
        user_name: currentUserName,
    });
```

```javascript
// Client B: Listening for a whisper

Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (data) => {
        console.log(`${data.user_name} is typing…`);
        showTypingIndicator(data.user_name);
    });
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `Echo.private('channel')` | Whispering requires a private or presence channel |
| `.whisper(eventName, data)` | Sends an ephemeral client-to-client event |
| `.listenForWhisper(eventName, callback)` | Receives whispers from other clients |
| `client-{eventName}` | The internal Pusher event name (auto-prefixed) |
| No HTTP, no queue, no DB | Whisper bypasses the Laravel backend entirely |

#### Syntax Rules

1. Whispering works **only** on private and presence channels; public channels do not support whispers.
2. The sender **must** be subscribed to the channel before whispering.
3. Whisper event names **must** be strings; they are internally prefixed with `client-`.
4. The data payload **must** be JSON-serializable.
5. The sender does **not** receive their own whisper (the WebSocket server relays to others only).
6. `.listenForWhisper()` **must** be called on the same channel as `.whisper()`.
7. Whispers are **not** authenticated by the server beyond channel authorization; any subscriber can send whispers.
8. Whispers are **not** persisted; if a client is offline, it misses the whisper.

#### Constraints and Limitations

- **No Persistence:** Whispers are fire-and-forget; there is no history.
- **No Delivery Guarantee:** If the recipient is offline or disconnected, the whisper is lost.
- **Channel-Wide:** Whispers are delivered to all other subscribers of the channel, not to specific users.
- **Payload Size:** Subject to the WebSocket driver's message size limit (Pusher: 10KB).
- **Rate Limiting:** Pusher limits client events to 10 per second per connection by default; Reverb has similar configurable limits.
- **No Server Validation:** The backend cannot validate whisper content; clients must trust each other.
- **Security:** Whispers can be spoofed by any channel subscriber; do not use for security-sensitive data.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Typing Indicator with Whisper

**Step-by-Step Setup Guide:**

1. Subscribe to a private channel.
2. On input events, throttle and send a `typing` whisper.
3. Listen for whispers and display a typing indicator.
4. Auto-expire the indicator after a timeout.

**Complete Executable Code:**

```javascript
// resources/js/typing-indicator.js

const roomId = window.chatConfig.roomId;
const currentUserId = window.chatConfig.userId;
const currentUserName = window.chatConfig.userName;

// Subscribe to the room's private channel
const channel = Echo.private(`chat.${roomId}`);

// Send a typing whisper on input (throttled to 1 per 2 seconds)
const input = document.getElementById('message-input');
let lastWhisper = 0;

input.addEventListener('input', () => {
    const now = Date.now();
    if (now - lastWhisper < 2000) return;
    lastWhisper = now;

    channel.whisper('typing', {
        user_id: currentUserId,
        user_name: currentUserName,
    });
});

// Listen for typing whispers from others
const typingUsers = new Map();

channel.listenForWhisper('typing', (data) => {
    // Ignore our own whispers (server doesn't echo them, but be safe)
    if (data.user_id === currentUserId) return;

    // Track the user's typing state
    typingUsers.set(data.user_id, data.user_name);
    renderTypingIndicator(Array.from(typingUsers.values()));

    // Auto-expire after 3 seconds of no new whisper
    clearTimeout(typingUsers.get(`timeout_${data.user_id}`));
    const timeout = setTimeout(() => {
        typingUsers.delete(data.user_id);
        renderTypingIndicator(Array.from(typingUsers.values()));
    }, 3000);
    typingUsers.set(`timeout_${data.user_id}`, timeout);
});

function renderTypingIndicator(names) {
    const el = document.getElementById('typing-indicator');
    if (names.length === 0) {
        el.textContent = '';
    } else if (names.length === 1) {
        el.textContent = `${names[0]} is typing…`;
    } else if (names.length === 2) {
        el.textContent = `${names[0]} and ${names[1]} are typing…`;
    } else {
        el.textContent = `${names.length} people are typing…`;
    }
}
```

**Expected Output:**

- As User A types, User B sees "User A is typing…" within ~100ms.
- After 3 seconds of no typing, the indicator disappears.
- No HTTP requests, queue jobs, or database writes occur.
- The whisper is delivered only to other clients in the same channel.

**Why This Code Produces That Result:**

- `.whisper()` sends a `client-typing` event over the WebSocket connection.
- The WebSocket server relays it to all other subscribers of the channel.
- `.listenForWhisper()` receives the event and updates the UI.
- The 2-second throttle prevents flooding; the 3-second timeout prevents stale indicators.

#### Example 2: Collaborative Cursor Positions via Whisper

```javascript
// resources/js/collaborative-cursor.js

const documentId = window.docConfig.documentId;
const editor = document.getElementById('editor');

const channel = Echo.private(`documents.${documentId}`);

// Send cursor position on selection change (throttled to 10/sec)
let cursorThrottle = null;

function broadcastCursor() {
    if (cursorThrottle) return;
    cursorThrottle = setTimeout(() => cursorThrottle = null, 100);

    channel.whisper('cursor-moved', {
        user_id: currentUserId,
        user_name: currentUserName,
        position: editor.selectionStart,
        selection_end: editor.selectionEnd,
        color: userColor,
    });
}

editor.addEventListener('keyup', broadcastCursor);
editor.addEventListener('click', broadcastCursor);
editor.addEventListener('select', broadcastCursor);

// Listen for other users' cursor whispers
const cursors = new Map();

channel.listenForWhisper('cursor-moved', (data) => {
    if (data.user_id === currentUserId) return;
    cursors.set(data.user_id, data);
    renderCursor(data);
});

// Remove cursors when users leave
channel.listen('.pusher:member_removed', (member) => {
    cursors.delete(member.id);
    removeCursor(member.id);
});
```

**Expected Output:**

- Each collaborator's cursor moves in real time on other clients.
- Cursor updates are throttled to 10/sec to prevent flooding.
- No database writes occur for cursor positions.
- Cursors are removed when users leave.

**Why This Code Produces That Result:**

- Whispers deliver cursor positions directly between clients.
- Throttling prevents excessive messages.
- Presence channel membership events handle cursor cleanup.

#### Example 3: Scroll Position Synchronization for Presentations

```javascript
// resources/js/presentation-sync.js

const presentationId = window.presentationConfig.id;
const channel = Echo.private(`presentations.${presentationId}`);

// Presenter broadcasts scroll position (throttled)
let scrollThrottle = null;

window.addEventListener('scroll', () => {
    if (!isPresenter) return;
    if (scrollThrottle) return;
    scrollThrottle = setTimeout(() => scrollThrottle = null, 200);

    channel.whisper('scroll-sync', {
        position: window.scrollY,
        slide: getCurrentSlide(),
    });
});

// Viewers follow the presenter's scroll
channel.listenForWhisper('scroll-sync', (data) => {
    if (isPresenter) return;
    window.scrollTo({ top: data.position, behavior: 'smooth' });
    goToSlide(data.slide);
});
```

**Expected Output:**

- The presenter's scroll position and slide changes are mirrored to all viewers.
- Viewers can follow along without a backend round-trip.
- No database writes occur.

**Why This Code Produces That Result:**

- Whispers relay scroll events directly between clients.
- Throttling to 200ms balances responsiveness and bandwidth.
- The private channel ensures only authorized viewers receive the whispers.

### Real-World Cases

**Case 1: Chat Typing Indicators**

A chat app uses whispers for typing indicators, saving millions of database writes and HTTP requests compared to server-side typing events.

**Case 2: Collaborative Whiteboard**

A whiteboard app uses whispers for cursor positions and temporary drawing strokes, keeping ephemeral data off the server.

**Case 3: Live Presentation Sync**

A presentation tool uses whispers to synchronize slide transitions and scroll positions between a presenter and viewers.

**Case 4: Multiplayer Game Lobby**

A game lobby uses whispers for quick emotes, ready checks, and ephemeral chat, keeping the backend free for authoritative game state.

### References

- Laravel Echo: Client Events (Whispering) - https://laravel.com/docs/12.x/broadcasting#client-events
- Pusher Channels: Client Events - https://pusher.com/docs/channels/using_channels/events/#triggering-client-events
- Laravel Broadcasting: Presence Channels - https://laravel.com/docs/12.x/broadcasting#presence-channels
- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation


## Summary Table of WebSocket Architecture Concerns

| Concern | Server-Side Config | Client-Side Config | Key Files/APIs |
|---------|-------------------|-------------------|----------------|
| Driver Selection | `BROADCAST_CONNECTION`, `config/broadcasting.php` | `broadcaster` in Echo config | `config/broadcasting.php` |
| Subscriptions | Channel authorization in `routes/channels.php` | `Echo.channel/private/join` | `routes/channels.php`, Echo |
| Authentication | `Broadcast::routes()`, channel closures | `authEndpoint`, `auth.headers` | `BroadcastServiceProvider` |
| Connection Management | `ping_interval`, `activity_timeout` in `config/reverb.php` | `activityTimeout`, `pongTimeout`, `connection.bind()` | `config/reverb.php`, pusher-js |
| Scaling | `REVERB_SCALING_ENABLED`, Redis, Supervisor, Nginx | (Transparent to client) | `config/reverb.php`, Supervisor, Nginx |
| Whispering | (None — bypasses backend) | `.whisper()`, `.listenForWhisper()` | Echo client only |

---

## References

- Laravel Broadcasting Documentation (12.x) - https://laravel.com/docs/12.x/broadcasting
- Laravel Broadcasting: Configuration - https://laravel.com/docs/12.x/broadcasting#configuration
- Laravel Reverb Documentation - https://laravel.com/docs/12.x/reverb
- Laravel Reverb: Scaling - https://laravel.com/docs/12.x/reverb#scaling
- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- Laravel Broadcasting: Authorizing Channels - https://laravel.com/docs/12.x/broadcasting#authorizing-channels
- Laravel Broadcasting: Client Events (Whispering) - https://laravel.com/docs/12.x/broadcasting#client-events
- Laravel Horizon Documentation - https://laravel.com/docs/12.x/horizon
- Laravel Queues: Supervisor Configuration - https://laravel.com/docs/12.x/queues#supervisor-configuration
- Laravel Sanctum Documentation - https://laravel.com/docs/12.x/sanctum
- Pusher Channels Documentation - https://pusher.com/docs/channels/
- Pusher Channels: Client Events - https://pusher.com/docs/channels/using_channels/events/
- Pusher Channels: Connection States - https://pusher.com/docs/channels/using_channels/connection/
- Ably Documentation - https://ably.com/docs
- Nginx: WebSocket Proxying - https://nginx.org/en/docs/http/websocket.html
- Redis Pub/Sub Documentation - https://redis.io/docs/interact/pubsub/
- WebSocket Protocol (RFC 6455) - https://datatracker.ietf.org/doc/html/rfc6455
- MDN: Page Visibility API - https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API
- MDN: Navigator.sendBeacon() - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon