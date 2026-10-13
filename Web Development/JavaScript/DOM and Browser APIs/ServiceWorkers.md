# Service Workers — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A service worker is a special type of Web Worker that acts as a programmable network proxy, intercepting network requests and enabling offline functionality, caching, background sync, and push notifications. It runs in a separate thread from the main browser thread.

**Technical Definition:** Per MDN, "A service worker is a type of web worker. It's essentially a JavaScript file that runs separately from the main browser thread, intercepting network requests, caching or retrieving resources from the cache, and delivering push messages". Service workers are defined in the W3C Service Workers specification. They are event-driven and have a lifecycle independent of the web page. A service worker can control the pages of a scope (a URL path prefix) and can cache assets, serve them from cache, and handle network requests even when the user is offline. Service workers require HTTPS (or localhost for development) due to their powerful capabilities. They are the foundation of Progressive Web Apps (PWAs).

**Beginner-Friendly Explanation:** A service worker is like a personal assistant that sits between your web app and the internet. When your app needs a file, the assistant can either fetch it from the internet or hand you a saved copy. When you're offline, the assistant keeps working with the saved copies. The assistant can also receive push notifications and sync data in the background. The best part: the assistant runs on a separate thread, so it doesn't slow down your app. Service workers are what make web apps work offline and feel more like native apps.

### Key Characteristics

- **Programmable network proxy:** Intercepts and handles network requests.
- **Separate thread:** Runs independently of the main browser thread.
- **Event-driven:** Responds to events (`install`, `activate`, `fetch`, `sync`, `push`).
- **Lifecycle independent:** Has its own lifecycle, separate from the web page.
- **HTTPS required:** Only works on secure origins (HTTPS or localhost).
- **Scope-based:** Controls pages within a URL scope.
- **Persistent:** Continues running after the page is closed (for events).
- **No DOM access:** Cannot access `window`, `document`, or `parent`.
- **Cache API integration:** Uses the Cache API for storing responses.

### Prerequisites

- Understanding of Web Workers and JavaScript events.
- Familiarity with Promises and `async/await`.
- Knowledge of HTTP requests, responses, and caching.
- Awareness of the Cache API and IndexedDB.
- Understanding of HTTPS and secure contexts.

### Related Programming Areas

- **Progressive Web Apps (PWAs):** Service workers are a core PWA technology.
- **Offline-first applications:** Serving content without a network.
- **Performance optimisation:** Caching assets for faster loading.
- **Push notifications:** Receiving and displaying push messages.
- **Background sync:** Syncing data when connectivity is restored.
- **Network resilience:** Handling unreliable networks.

---

## Service Workers

### Definitions

**Core Definition:** A service worker is a script that runs in the background, intercepting network requests and enabling offline functionality, caching, and push notifications.

**Technical Definition:** Per the W3C Service Workers specification, "A service worker is an event-driven worker registered against an origin and a path. It takes the form of a JavaScript file that can control the web page/site it is associated with, intercepting and modifying navigation and resource requests, and caching resources in a very granular fashion to complete offline experiences, or to boost performance".

### Purposes

- To enable offline functionality for web applications.
- To intercept and handle network requests programmatically.
- To cache assets and API responses for performance.
- To receive and display push notifications.
- To perform background sync when connectivity is restored.
- To implement advanced caching strategies (cache-first, network-first, stale-while-revalidate).

### Syntax Rules and Structure

**Registration (main thread):**
```js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js", { scope: "/" })
    .then((registration) => console.log("Registered:", registration.scope))
    .catch((error) => console.error("Registration failed:", error));
}
```

**Service worker (sw.js):**
```js
self.addEventListener("install", (event) => { /* ... */ });
self.addEventListener("activate", (event) => { /* ... */ });
self.addEventListener("fetch", (event) => { /* ... */ });
```

### Annotated Code Example: Basic Service Worker

**main.js (main thread):**
```js
// Step 1: Check for service worker support.
if ("serviceWorker" in navigator) {
  // Step 2: Register the service worker.
  navigator.serviceWorker
    .register("/sw.js")
    .then((registration) => {
      console.log("SW registered:", registration.scope);
    })
    .catch((error) => {
      console.error("SW registration failed:", error);
    });
}
```

**sw.js (service worker):**
```js
// Step 1: Install event — cache assets.
self.addEventListener("install", (event) => {
  console.log("Service Worker installing");
  event.waitUntil(
    caches.open("v1").then((cache) => {
      return cache.addAll([
        "/",
        "/index.html",
        "/styles.css",
        "/app.js",
      ]);
    })
  );
});

// Step 2: Activate event — clean up old caches.
self.addEventListener("activate", (event) => {
  console.log("Service Worker activating");
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== "v1").map((key) => caches.delete(key))
      );
    })
  );
});

// Step 3: Fetch event — serve from cache or network.
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

**Expected Output:** The service worker registers, caches the specified assets, and serves them from the cache when requested.

**Why This Output:** The `install` event caches the assets. The `activate` event cleans up old caches. The `fetch` event intercepts requests and serves from cache if available, falling back to the network.

### Real-World Cases

- **Offline web apps:** Serving cached content when offline.
- **PWAs:** Enabling app-like experiences.
- **Performance:** Caching assets for instant loading.
- **Push notifications:** Receiving and displaying notifications.

---

## Service Worker Lifecycle

### Definitions

**Core Definition:** The service worker lifecycle is the sequence of states a service worker goes through from registration to activation and beyond.

**Technical Definition:** Per MDN, "A service worker goes through a lifecycle of events: registration, installation, activation, and fetch. The lifecycle is designed to ensure that only one version of a service worker is active at a time, and that updates are handled gracefully".

### Lifecycle Stages

| Stage | Event | Description |
|-------|-------|-------------|
| Registration | — | The browser registers the service worker script |
| Installation | `install` | The service worker is installed; cache assets |
| Waiting | — | The service worker waits for old workers to release control |
| Activation | `activate` | The service worker takes control; clean up old caches |
| Fetch | `fetch` | The service worker intercepts network requests |
| Termination | — | The service worker is terminated when idle |
| Update | `install` | A new version is detected and installed |
| Redundant | — | An old service worker is discarded |

### Install

**Definition:** The `install` event is fired when the service worker is first installed or when an update is detected.

```js
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("v1").then((cache) => {
      return cache.addAll(["/", "/index.html", "/styles.css"]);
    })
  );
  self.skipWaiting(); // Activate immediately
});
```

### Activate

**Definition:** The `activate` event is fired when the service worker takes control, after installation and waiting.

```js
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== "v1").map((key) => caches.delete(key))
      );
    })
  );
  self.clients.claim(); // Take control of all clients immediately
});
```

### Fetch

**Definition:** The `fetch` event is fired for every network request made by pages within the service worker's scope.

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

### Annotated Code Example: Full Lifecycle

```js
// sw.js:
const CACHE_NAME = "my-app-v1";
const ASSETS = ["/", "/index.html", "/styles.css", "/app.js", "/logo.png"];

// Install:
self.addEventListener("install", (event) => {
  console.log("Installing...");
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      console.log("Caching assets");
      return cache.addAll(ASSETS);
    })
  );
  self.skipWaiting();
});

// Activate:
self.addEventListener("activate", (event) => {
  console.log("Activating...");
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys
          .filter((key) => key !== CACHE_NAME)
          .map((key) => {
            console.log("Deleting old cache:", key);
            return caches.delete(key);
          })
      );
    })
  );
  self.clients.claim();
});

// Fetch:
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      if (response) {
        console.log("Serving from cache:", event.request.url);
        return response;
      }
      console.log("Fetching from network:", event.request.url);
      return fetch(event.request);
    })
  );
});
```

**Expected Output:** The service worker installs, caches assets, activates, and serves cached assets on subsequent requests.

### Real-World Cases

- **App updates:** Gracefully updating the service worker without disrupting users.
- **Cache management:** Cleaning up old caches on activation.
- **Immediate control:** Using `skipWaiting()` and `clients.claim()`.

---

## Caching Strategies

### Definitions

**Core Definition:** Caching strategies are patterns for deciding how to respond to fetch events — whether to serve from cache, fetch from the network, or both.

**Technical Definition:** Per the offline cookbook, common caching strategies include: cache-first, network-first, stale-while-revalidate, cache-only, network-only, and cache-then-network. Each strategy has different trade-offs between speed, freshness, and offline support.

### Common Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Cache-first** | Check cache first; fall back to network | Static assets (CSS, JS, images) |
| **Network-first** | Try network first; fall back to cache | Dynamic content (APIs, HTML) |
| **Stale-while-revalidate** | Serve from cache; update in background | Assets that change occasionally |
| **Cache-only** | Serve only from cache | Offline-only resources |
| **Network-only** | Always fetch from network | Non-cacheable resources |

### Annotated Code Examples

#### Example 1: Cache-First Strategy

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      // Return cached response if available.
      if (cachedResponse) {
        return cachedResponse;
      }
      // Otherwise, fetch from network and cache.
      return fetch(event.request).then((response) => {
        return caches.open("v1").then((cache) => {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    })
  );
});
```

#### Example 2: Network-First Strategy

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(
    fetch(event.request)
      .then((response) => {
        // Cache the fresh response.
        return caches.open("v1").then((cache) => {
          cache.put(event.request, response.clone());
          return response;
        });
      })
      .catch(() => {
        // Fall back to cache if network fails.
        return caches.match(event.request);
      })
  );
});
```

#### Example 3: Stale-While-Revalidate

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      const fetchPromise = fetch(event.request).then((networkResponse) => {
        // Update the cache with the fresh response.
        caches.open("v1").then((cache) => {
          cache.put(event.request, networkResponse.clone());
        });
        return networkResponse;
      });

      // Return cached response immediately, update in background.
      return cachedResponse || fetchPromise;
    })
  );
});
```

### Real-World Cases

- **Static assets:** Cache-first for CSS, JS, and images.
- **APIs:** Network-first or stale-while-revalidate for dynamic data.
- **Offline pages:** Cache-only for the offline fallback page.
- **Third-party resources:** Network-only for non-cacheable resources.

---

## Offline Support

### Definitions

**Core Definition:** Offline support is the ability of a web application to function (fully or partially) when the network is unavailable.

**Technical Definition:** Per MDN, "Service workers enable offline experiences by caching resources. When the network is unavailable, the service worker can serve cached resources, allowing the application to continue functioning".

### Annotated Code Example: Offline Fallback

```js
// sw.js:
const CACHE_NAME = "v1";
const OFFLINE_URL = "/offline.html";

self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll([
        "/",
        "/index.html",
        "/styles.css",
        "/app.js",
        OFFLINE_URL,
      ]);
    })
  );
});

self.addEventListener("fetch", (event) => {
  // For navigation requests, try network first, then cache, then offline page.
  if (event.request.mode === "navigate") {
    event.respondWith(
      fetch(event.request).catch(() => {
        return caches.match(OFFLINE_URL);
      })
    );
    return;
  }

  // For other requests, use cache-first.
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

**Expected Output:** When offline, navigation requests serve the offline fallback page. Other requests serve from cache if available.

### Real-World Cases

- **Offline reading:** News apps, documentation, e-books.
- **Offline forms:** Filling out forms offline and syncing later.
- **Offline maps:** Cached map tiles for navigation.
- **Offline games:** Playing games without a connection.

---

## Background Sync

### Definitions

**Core Definition:** Background sync is a service worker API that allows deferred tasks to be performed when the user has a stable network connection.

**Technical Definition:** Per MDN, "The Background Sync API lets you defer actions until the user has stable connectivity. This is useful for ensuring that whatever the user wants to send, is actually sent". It uses the `sync` event in the service worker and the `SyncManager` interface in the main thread.

### Syntax Rules and Structure

**Main thread:**
```js
const registration = await navigator.serviceWorker.ready;
await registration.sync.register("sync-messages");
```

**Service worker:**
```js
self.addEventListener("sync", (event) => {
  if (event.tag === "sync-messages") {
    event.waitUntil(syncMessages());
  }
});
```

### Annotated Code Example

**main.js:**
```js
async function sendMessage(message) {
  // Store the message in IndexedDB.
  await saveToOutbox(message);

  // Register a sync event.
  const registration = await navigator.serviceWorker.ready;
  await registration.sync.register("sync-messages");
  console.log("Sync registered");
}
```

**sw.js:**
```js
self.addEventListener("sync", (event) => {
  if (event.tag === "sync-messages") {
    event.waitUntil(syncMessages());
  }
});

async function syncMessages() {
  const messages = await getOutboxMessages();

  for (const message of messages) {
    try {
      await fetch("/api/messages", {
        method: "POST",
        body: JSON.stringify(message),
      });
      await removeFromOutbox(message.id);
    } catch (error) {
      // Retry later
      throw error;
    }
  }
}
```

**Expected Output:** Messages are saved offline and synced when connectivity is restored.

### Real-World Cases

- **Offline forms:** Submitting forms when back online.
- **Chat apps:** Sending messages when the connection returns.
- **Data sync:** Syncing local changes with the server.
- **Email clients:** Sending queued emails.

---

## Push Notifications

### Definitions

**Core Definition:** Push notifications are messages sent from a server to a client, displayed by the browser even when the web page is not open, using the Push API and the Notifications API.

**Technical Definition:** Per MDN, "The Push API gives web applications the ability to receive messages pushed to them from a server, whether or not the web app is in the foreground, or even currently loaded, on a user agent". Push notifications require a service worker to handle the `push` event.

### Syntax Rules and Structure

**Main thread (subscribe):**
```js
const registration = await navigator.serviceWorker.ready;
const subscription = await registration.pushManager.subscribe({
  userVisibleOnly: true,
  applicationServerKey: "YOUR_PUBLIC_KEY",
});
// Send subscription to server
```

**Service worker (handle push):**
```js
self.addEventListener("push", (event) => {
  const data = event.data.json();
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: "/icon.png",
    })
  );
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow("/"));
});
```

### Annotated Code Example

**main.js:**
```js
// Step 1: Check for support.
if ("PushManager" in window) {
  // Step 2: Register the service worker.
  const registration = await navigator.serviceWorker.register("/sw.js");

  // Step 3: Subscribe to push.
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array("YOUR_PUBLIC_KEY"),
  });

  // Step 4: Send subscription to server.
  await fetch("/api/subscribe", {
    method: "POST",
    body: JSON.stringify(subscription),
  });
}
```

**sw.js:**
```js
self.addEventListener("push", (event) => {
  const data = event.data ? event.data.json() : {};

  event.waitUntil(
    self.registration.showNotification(data.title || "Notification", {
      body: data.body || "You have a new message",
      icon: "/icon-192.png",
      badge: "/badge.png",
    })
  );
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();

  event.waitUntil(
    clients.matchAll({ type: "window" }).then((clientList) => {
      for (const client of clientList) {
        if (client.url === "/" && "focus" in client) {
          return client.focus();
        }
      }
      return clients.openWindow("/");
    })
  );
});
```

**Expected Output:** A push notification appears even when the page is closed. Clicking it opens the app.

### Real-World Cases

- **Messaging apps:** New message notifications.
- **E-commerce:** Order updates and promotions.
- **News:** Breaking news alerts.
- **Social media:** Likes, comments, and follows.

---

## Service Worker Debugging

### Definitions

**Core Definition:** Service worker debugging is the process of inspecting, testing, and troubleshooting service worker behaviour using browser developer tools.

### Debugging Tools

| Tool | Description |
|------|-------------|
| **Chrome DevTools – Application tab** | View registered service workers, caches, and storage |
| **Chrome DevTools – Service Workers panel** | Inspect and debug service workers |
| **Chrome DevTools – Network tab** | See which requests are served by the service worker |
| **Chrome DevTools – Console** | View service worker logs |
| **Lighthouse** | Audit PWA and service worker performance |
| **Workbox** | Google's library for service worker tooling |

### Debugging Tips

```js
// 1. Log in the service worker.
self.addEventListener("fetch", (event) => {
  console.log("Fetch:", event.request.url);
});

// 2. Use the DevTools Application tab to:
//    - Unregister service workers
//    - Clear caches
//    - Update service workers
//    - Simulate offline mode

// 3. Use "Update on reload" in DevTools to force updates.
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Service worker not registering | HTTPS required (except localhost) | Use HTTPS or localhost |
| Cache not updating | Old service worker still active | Use `skipWaiting()` and `clients.claim()` |
| Fetch not intercepted | Scope mismatch | Check the `scope` option |
| Stale content | Cache-first strategy | Use network-first or stale-while-revalidate |

### Real-World Cases

- **Development:** Testing offline behaviour with DevTools.
- **Production:** Monitoring service worker errors.
- **Updates:** Debugging cache invalidation issues.

---

## Service Worker Best Practices

### Definitions

**Core Definition:** Service worker best practices are guidelines for building reliable, performant, and maintainable service workers.

### Key Practices

**1. Always Use HTTPS**

```js
// Service workers require HTTPS (or localhost).
// Ensure your production site uses HTTPS.
```

**2. Use Versioned Caches**

```js
const CACHE_NAME = "my-app-v1.2.3";
// Update the version when deploying new assets.
```

**3. Clean Up Old Caches**

```js
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== CACHE_NAME).map((key) => caches.delete(key))
      );
    })
  );
});
```

**4. Use `skipWaiting()` and `clients.claim()` Carefully**

```js
// Activate immediately (use with caution in production).
self.skipWaiting();
self.clients.claim();
```

**5. Handle Fetch Errors Gracefully**

```js
self.addEventListener("fetch", (event) => {
  event.respondWith(
    fetch(event.request).catch(() => {
      return caches.match("/offline.html");
    })
  );
});
```

**6. Don't Cache Everything**

```js
// Only cache what you need.
// Avoid caching large files or sensitive data.
```

**7. Use Workbox for Complex Scenarios**

```js
// Workbox simplifies service worker development.
import { registerRoute } from "workbox-routing";
import { CacheFirst, NetworkFirst } from "workbox-strategies";

registerRoute(
  ({ request }) => request.destination === "image",
  new CacheFirst({ cacheName: "images" })
);

registerRoute(
  ({ request }) => request.destination === "document",
  new NetworkFirst({ cacheName: "pages" })
);
```

**8. Test Offline Behaviour**

```js
// Use DevTools to simulate offline mode.
// Test navigation, API calls, and asset loading.
```

**9. Monitor Service Worker Health**

```js
// Log errors and unexpected behaviour.
self.addEventListener("error", (event) => {
  console.error("Service Worker error:", event.error);
});
```

**10. Keep Service Workers Simple**

```js
// Avoid complex logic in service workers.
// Use them primarily for caching and network interception.
```

### Real-World Cases

- **E-commerce:** Caching product images and pages for fast loading.
- **News apps:** Offline reading and push notifications.
- **Social media:** Background sync for posts and messages.
- **SaaS apps:** Offline support and performance optimisation.

---

## References

- MDN Web Docs – Service Worker API — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- MDN Web Docs – Using Service Workers — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers
- MDN Web Docs – ServiceWorker — https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker
- MDN Web Docs – ServiceWorkerRegistration — https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration
- MDN Web Docs – Cache — https://developer.mozilla.org/en-US/docs/Web/API/Cache
- MDN Web Docs – Background Sync API — https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API
- MDN Web Docs – Push API — https://developer.mozilla.org/en-US/docs/Web/API/Push_API
- W3C – Service Workers Specification — https://www.w3.org/TR/service-workers/
- Google Developers – Service Workers: An Introduction — https://developers.google.com/web/fundamentals/primers/service-workers
- Google Developers – The Service Worker Lifecycle — https://web.dev/articles/service-worker-lifecycle
- Google Developers – Offline Cookbook — https://web.dev/articles/offline-cookbook
- Workbox Documentation — https://developer.chrome.com/docs/workbox/
- Jake Archibald – The Service Worker Lifecycle — https://jakearchibald.com/2014/service-worker-lifecycle/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf