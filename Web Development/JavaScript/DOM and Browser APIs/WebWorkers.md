# Web Workers — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A Web Worker is a JavaScript feature that allows you to run scripts in background threads, separate from the main execution thread of a web page. This enables computationally expensive tasks to run without blocking the user interface.

**Technical Definition:** Per MDN, "Web Workers makes it possible to run a script operation in a background thread separate from the main execution thread of a web application. The advantage of this is that laborious processing can be performed in a separate thread, allowing the main (usually the UI) thread to run without being blocked/slowed down". Web Workers are defined in the WHATWG HTML Living Standard. A worker is an object created using the `Worker()` constructor that runs a named JavaScript file. Workers run in a different global context called `WorkerGlobalScope` (accessible via `self`), not the `window` context. Communication between the main thread and workers is via message passing using `postMessage()` and the `message` event. Workers cannot directly manipulate the DOM, access `window`, `document`, or `parent` objects. There are three main types of workers: dedicated workers, shared workers, and service workers.

**Beginner-Friendly Explanation:** A Web Worker is like hiring an assistant to do heavy lifting while you keep serving customers. The main thread (you) handles the user interface — clicks, typing, scrolling. The worker (your assistant) handles number-crunching, data processing, or any other heavy task in the background. They communicate by sending messages back and forth. This means your page stays responsive even when doing complex calculations. The catch: workers can't touch the DOM, and they can't share memory with the main thread (except for specific transferable objects).

### Key Characteristics

- **Separate thread:** Workers run in their own thread, independent of the main thread.
- **No DOM access:** Workers cannot access `window`, `document`, or `parent`. They can't manipulate the DOM.
- **Message passing:** Communication is via `postMessage()` and the `message` event.
- **Transferable objects:** Large binary data can be transferred (not copied) between threads for performance.
- **Same-origin policy:** Workers must be loaded from the same origin as the main page.
- **No blocking:** Worker execution doesn't block the main thread.
- **Persistent:** Workers continue running until terminated or the page is closed.
- **Three types:** Dedicated workers (single page), shared workers (multiple pages), service workers (network proxy).

### Prerequisites

- Understanding of JavaScript functions, events, and callbacks.
- Familiarity with the event loop and asynchronous programming.
- Basic knowledge of the DOM and the `window` object.
- Awareness of the same-origin policy.

### Related Programming Areas

- **Performance optimisation:** Offloading CPU-intensive tasks.
- **Data processing:** Handling large datasets without blocking the UI.
- **Image/video processing:** Applying filters, encoding, decoding.
- **Real-time applications:** WebSockets, SSE, and worker-based data handling.
- **Progressive Web Apps (PWAs):** Service workers for offline functionality.
- **Multi-threaded JavaScript:** Using multiple workers for parallel processing.

---

## Web Workers

### Definitions

**Core Definition:** A Web Worker is a JavaScript script that runs in a background thread, separate from the main thread, allowing CPU-intensive tasks to be performed without blocking the UI.

**Technical Definition:** Per MDN, "A worker is an object created using a constructor (e.g., `Worker()`) that runs a named JavaScript file — this file contains the code that will run in the worker thread; workers run in another global context that is different from the current `window`". This context is represented by a `DedicatedWorkerGlobalScope` object (for dedicated workers) or a `SharedWorkerGlobalScope` object (for shared workers).

### Purposes

- To run CPU-intensive tasks without blocking the main thread.
- To keep the UI responsive during heavy computation.
- To process large amounts of data in the background.
- To enable parallel processing using multiple workers.
- To isolate potentially unstable code from the main application.

### Syntax Rules and Structure

```js
// Main thread:
const worker = new Worker("worker.js");

// Worker thread (worker.js):
self.onmessage = (event) => {
  // Handle message
};
```

**Component Breakdown:**
- `new Worker("worker.js")`: Creates a new dedicated worker.
- `self`: The global object in the worker context.
- `onmessage`: The event handler for incoming messages.

### Annotated Code Examples

#### Example 1: Basic Dedicated Worker

**main.js:**
```js
// Step 1: Create a worker.
const worker = new Worker("worker.js");

// Step 2: Send a message to the worker.
worker.postMessage({ type: "calculate", numbers: [1, 2, 3, 4, 5] });

// Step 3: Handle messages from the worker.
worker.onmessage = (event) => {
  console.log("Result from worker:", event.data.result);
};

// Step 4: Handle errors.
worker.onerror = (event) => {
  console.error("Worker error:", event.message);
};

// Step 5: Terminate when done.
// worker.terminate();
```

**worker.js:**
```js
// Step 1: Handle messages from the main thread.
self.onmessage = (event) => {
  const { type, numbers } = event.data;

  if (type === "calculate") {
    // Step 2: Perform the computation.
    const result = numbers.reduce((sum, n) => sum + n, 0);

    // Step 3: Send the result back.
    self.postMessage({ result });
  }
};
```

**Expected Output:**
```
Result from worker: 15
```

**Why This Output:** The main thread sends an array of numbers to the worker. The worker computes the sum and sends the result back. The main thread logs the result.

### Real-World Cases

- **Image processing:** Applying filters or transformations to images.
- **Data analysis:** Processing large datasets (CSV, JSON).
- **Cryptography:** Hashing, encryption, and decryption.
- **Game physics:** Simulating physics in the background.
- **Syntax highlighting:** Parsing code in a text editor.

---

## Worker Creation

### Definitions

**Core Definition:** Worker creation is the process of instantiating a new worker thread using the `Worker()` constructor.

**Technical Definition:** Per MDN, "The `Worker()` constructor creates a `Worker` object that executes the script at the specified URL. This script must obey the same-origin policy". The URL can be a relative or absolute path to a JavaScript file, or a Blob URL for inline workers.

### Syntax Rules and Structure

```js
const worker = new Worker(scriptURL, options);
```

**Component Breakdown:**
- `scriptURL`: The URL of the worker script.
- `options` (optional): An object with a `type` property (`"classic"` or `"module"`) and a `name` property.

### Types of Worker Creation

| Method | Description |
|--------|-------------|
| External file | `new Worker("worker.js")` — standard approach |
| Module worker | `new Worker("worker.js", { type: "module" })` — supports ES modules |
| Blob URL | `new Worker(URL.createObjectURL(blob))` — inline worker |
| Data URL | `new Worker("data:application/javascript,...")` — inline worker |

### Annotated Code Examples

#### Example 1: External Worker File

```js
// main.js:
const worker = new Worker("worker.js");
worker.postMessage("Hello");
```

```js
// worker.js:
self.onmessage = (event) => {
  self.postMessage(`Worker received: ${event.data}`);
};
```

#### Example 2: Inline Worker with Blob

```js
// Step 1: Create the worker code as a string.
const workerCode = `
  self.onmessage = (event) => {
    const result = event.data * 2;
    self.postMessage(result);
  };
`;

// Step 2: Create a Blob and URL.
const blob = new Blob([workerCode], { type: "application/javascript" });
const workerUrl = URL.createObjectURL(blob);

// Step 3: Create the worker.
const worker = new Worker(workerUrl);

worker.onmessage = (event) => {
  console.log("Result:", event.data); // Result: 10
};

worker.postMessage(5);

// Step 4: Revoke the URL when done.
// URL.revokeObjectURL(workerUrl);
```

**Expected Output:**
```
Result: 10
```

#### Example 3: Module Worker

```js
// main.js:
const worker = new Worker("worker.js", { type: "module" });
```

```js
// worker.js (ES module):
import { calculate } from "./math.js";

self.onmessage = (event) => {
  self.postMessage(calculate(event.data));
};
```

### Real-World Cases

- **External files:** Standard approach for reusable workers.
- **Inline workers:** Quick prototyping or single-use workers.
- **Module workers:** Using ES modules in workers.

---

## `postMessage()`

### Definitions

**Core Definition:** `postMessage()` is the method used to send messages from the main thread to a worker, or from a worker to the main thread.

**Technical Definition:** Per MDN, "The `postMessage()` method of the `Worker` interface sends a message to the worker. The first parameter is the data to send to the worker. The data may be any JavaScript object that can be handled by the structured clone algorithm". The worker sends messages back using `self.postMessage()`.

### Syntax Rules and Structure

```js
// Main thread to worker:
worker.postMessage(message, transfer);

// Worker to main thread:
self.postMessage(message, transfer);
```

**Component Breakdown:**
- `message`: The data to send. Can be any structured-cloneable value.
- `transfer` (optional): An array of transferable objects to transfer ownership of.

### Structured Clone Algorithm

The structured clone algorithm supports:

| Type | Supported |
|------|-----------|
| Plain objects | Yes |
| Arrays | Yes |
| Dates | Yes |
| RegExp | Yes |
| Map, Set | Yes |
| ArrayBuffer, TypedArray | Yes (transferable) |
| Blob, File | Yes |
| ImageData | Yes |
| Functions | No |
| DOM nodes | No |
| Errors | Yes |

### Annotated Code Example

```js
// Main thread:
const worker = new Worker("worker.js");

// Send a complex object.
worker.postMessage({
  type: "process",
  data: {
    numbers: [1, 2, 3, 4, 5],
    operation: "sum",
  },
});

worker.onmessage = (event) => {
  console.log("Result:", event.data.result); // 15
};
```

```js
// worker.js:
self.onmessage = (event) => {
  const { type, data } = event.data;

  if (type === "process") {
    const result = data.numbers.reduce((sum, n) => sum + n, 0);
    self.postMessage({ result });
  }
};
```

**Expected Output:**
```
Result: 15
```

### Real-World Cases

- **Data processing:** Sending large datasets to workers.
- **Progress updates:** Workers sending progress messages.
- **Command patterns:** Sending typed commands to workers.

---

## `onmessage`

### Definitions

**Core Definition:** `onmessage` is the event handler that receives messages sent to a worker or to the main thread from a worker.

**Technical Definition:** Per MDN, "The `onmessage` property of the `Worker` interface represents an event handler function to be called when the `message` event occurs — that is, when a message is received from the worker". The event object contains the `data` property with the message payload.

### Syntax Rules and Structure

```js
// Main thread:
worker.onmessage = (event) => {
  console.log(event.data);
};

// Worker:
self.onmessage = (event) => {
  console.log(event.data);
};
```

### Using `addEventListener`

```js
worker.addEventListener("message", (event) => {
  console.log(event.data);
});
```

### Annotated Code Example

```js
// Main thread:
const worker = new Worker("worker.js");

worker.onmessage = (event) => {
  const { type, payload } = event.data;

  switch (type) {
    case "progress":
      updateProgressBar(payload.percent);
      break;
    case "result":
      displayResult(payload.data);
      break;
    case "error":
      showError(payload.message);
      break;
  }
};

worker.postMessage({ type: "start", data: [1, 2, 3, 4, 5] });
```

```js
// worker.js:
self.onmessage = (event) => {
  const { type, data } = event.data;

  if (type === "start") {
    // Send progress updates.
    for (let i = 0; i < data.length; i++) {
      self.postMessage({
        type: "progress",
        payload: { percent: ((i + 1) / data.length) * 100 },
      });
    }

    // Send the final result.
    const result = data.reduce((sum, n) => sum + n, 0);
    self.postMessage({ type: "result", payload: { data: result } });
  }
};
```

**Expected Output:** Progress updates followed by the final result.

### Real-World Cases

- **Progress reporting:** Workers sending progress updates.
- **Streaming results:** Workers sending partial results.
- **Error handling:** Workers sending error messages.

---

## Transferable Objects

### Definitions

**Core Definition:** Transferable objects are objects that can be transferred from one thread to another, with ownership of the underlying memory being transferred rather than copied.

**Technical Definition:** Per MDN, "Transferable objects are objects that own resources that can be transferred from one context to another, ensuring that the resources are only available in one context at a time. Transferring an object makes it unusable in the context it was transferred from". Common transferable objects include `ArrayBuffer`, `MessagePort`, `ReadableStream`, `WritableStream`, `TransformStream`, `ImageBitmap`, and `OffscreenCanvas`.

### Transferable Objects

| Object | Description |
|--------|-------------|
| `ArrayBuffer` | Raw binary data |
| `MessagePort` | Communication channel |
| `ReadableStream` | Stream of readable data |
| `WritableStream` | Stream of writable data |
| `TransformStream` | Transform stream |
| `ImageBitmap` | Image bitmap |
| `OffscreenCanvas` | Offscreen canvas |

### Syntax Rules and Structure

```js
// Transferring an ArrayBuffer:
worker.postMessage(buffer, [buffer]);

// After transfer, buffer is unusable in the main thread.
console.log(buffer.byteLength); // 0
```

### Annotated Code Example

```js
// Main thread:
const worker = new Worker("worker.js");

// Step 1: Create a large ArrayBuffer.
const buffer = new ArrayBuffer(1024 * 1024 * 100); // 100 MB
const view = new Uint8Array(buffer);
view[0] = 42;

// Step 2: Transfer the buffer to the worker (zero-copy).
worker.postMessage({ type: "process", buffer }, [buffer]);

// Step 3: The buffer is now unusable in the main thread.
console.log(buffer.byteLength); // 0

worker.onmessage = (event) => {
  console.log("Processed:", event.data.result);
};
```

```js
// worker.js:
self.onmessage = (event) => {
  const { type, buffer } = event.data;

  if (type === "process") {
    const view = new Uint8Array(buffer);
    const result = view[0] * 2;
    self.postMessage({ result });
  }
};
```

**Expected Output:**
```
0
Processed: 84
```

**Why This Output:** The `ArrayBuffer` is transferred (not copied) to the worker. The main thread's reference becomes unusable (`byteLength` is 0). The worker processes the data and sends back the result.

### Real-World Cases

- **Large file processing:** Transferring file data to workers.
- **Image processing:** Transferring image bitmaps.
- **Video encoding:** Transferring video frames.
- **Audio processing:** Transferring audio buffers.

---

## Shared Workers

### Definitions

**Core Definition:** A shared worker is a worker that can be accessed by multiple browsing contexts (tabs, windows, iframes) from the same origin.

**Technical Definition:** Per MDN, "A shared worker is a worker that can be accessed from several browsing contexts, such as several windows, iframes, or even other workers. They implement a different interface than dedicated workers — their global scope is `SharedWorkerGlobalScope`". Communication is via `MessagePort` objects.

### Syntax Rules and Structure

```js
// Main thread:
const worker = new SharedWorker("shared-worker.js");

worker.port.start();
worker.port.postMessage("Hello");

worker.port.onmessage = (event) => {
  console.log(event.data);
};
```

```js
// shared-worker.js:
const connections = [];

self.onconnect = (event) => {
  const port = event.ports[0];
  connections.push(port);

  port.onmessage = (event) => {
    // Broadcast to all connected ports.
    connections.forEach((p) => p.postMessage(event.data));
  };
};
```

### Annotated Code Example

**Tab 1 (main thread):**
```js
const worker = new SharedWorker("shared-worker.js");
worker.port.start();
worker.port.postMessage("Hello from Tab 1");
worker.port.onmessage = (event) => console.log("Tab 1 received:", event.data);
```

**Tab 2 (main thread):**
```js
const worker = new SharedWorker("shared-worker.js");
worker.port.start();
worker.port.postMessage("Hello from Tab 2");
worker.port.onmessage = (event) => console.log("Tab 2 received:", event.data);
```

**shared-worker.js:**
```js
const ports = [];

self.onconnect = (event) => {
  const port = event.ports[0];
  ports.push(port);

  port.onmessage = (event) => {
    // Broadcast to all connected tabs.
    ports.forEach((p) => p.postMessage(event.data));
  };
};
```

**Expected Output:** Both tabs receive the messages sent by either tab.

### Real-World Cases

- **Cross-tab communication:** Syncing state between tabs.
- **Shared caches:** A single worker managing a shared cache.
- **WebSocket multiplexing:** A single WebSocket connection shared across tabs.
- **Authentication:** Shared authentication state across tabs.

---

## Service Workers

### Definitions

**Core Definition:** A service worker is a special type of worker that acts as a network proxy, intercepting network requests and enabling offline functionality, caching, and push notifications.

**Technical Definition:** Per MDN, "A service worker is a type of web worker. It's essentially a JavaScript file that runs separately from the main browser thread, intercepting network requests, caching or retrieving resources from the cache, and delivering push messages". Service workers are event-driven and have a lifecycle independent of the web page. They enable Progressive Web Apps (PWAs) with offline capabilities.

### Service Worker Lifecycle

1. **Registration:** `navigator.serviceWorker.register("/sw.js")`
2. **Installation:** The `install` event fires. Cache assets.
3. **Activation:** The `activate` event fires. Clean up old caches.
4. **Fetch interception:** The `fetch` event fires for network requests.

### Annotated Code Example

**main.js:**
```js
// Step 1: Register the service worker.
if ("serviceWorker" in navigator) {
  navigator.serviceWorker
    .register("/sw.js")
    .then((registration) => console.log("SW registered:", registration.scope))
    .catch((error) => console.error("SW registration failed:", error));
}
```

**sw.js:**
```js
// Step 1: Install — cache assets.
self.addEventListener("install", (event) => {
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

// Step 2: Activate — clean up old caches.
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== "v1").map((key) => caches.delete(key))
      );
    })
  );
});

// Step 3: Fetch — serve from cache or network.
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

**Expected Output:** The app works offline, serving cached assets when the network is unavailable.

### Real-World Cases

- **Offline-first apps:** Serving cached content when offline.
- **Push notifications:** Receiving and displaying push messages.
- **Background sync:** Syncing data when the connection is restored.
- **Asset caching:** Caching static assets for performance.

---

## Worker Limitations

### Definitions

**Core Definition:** Worker limitations are the constraints and restrictions that apply to code running in a worker thread.

### Key Limitations

| Limitation | Description |
|------------|-------------|
| **No DOM access** | Workers cannot access `window`, `document`, or `parent` |
| **No `alert()`** | Workers cannot use `alert()`, `confirm()`, or `prompt()` |
| **Same-origin policy** | Worker scripts must be loaded from the same origin |
| **No synchronous XHR** | Workers cannot use synchronous `XMLHttpRequest` |
| **Limited APIs** | Some browser APIs are not available in workers |
| **No direct DOM manipulation** | Workers must send messages to the main thread to update the UI |
| **Communication overhead** | Message passing has some overhead |
| **No shared memory (by default)** | Workers cannot share memory with the main thread (except `SharedArrayBuffer`) |
| **Debugging** | Debugging workers can be more difficult |

### Available APIs in Workers

- `fetch()`, `XMLHttpRequest` (async)
- `WebSocket`
- `IndexedDB`
- `Cache API`
- `FileReader`, `FileReaderSync`
- `Web Crypto API`
- `setTimeout`, `setInterval`
- `importScripts()`
- `OffscreenCanvas`
- `BroadcastChannel`

### Annotated Code Example: Checking Available APIs

```js
// worker.js:
console.log("Worker global scope:", self.constructor.name);
// "DedicatedWorkerGlobalScope"

console.log("window available:", typeof window); // "undefined"
console.log("document available:", typeof document); // "undefined"
console.log("fetch available:", typeof fetch); // "function"
console.log("indexedDB available:", typeof indexedDB); // "object"
console.log("WebSocket available:", typeof WebSocket); // "function"
```

### Real-World Cases

- **UI updates:** Workers must send messages to the main thread.
- **API access:** Using `fetch` and `IndexedDB` in workers.
- **Canvas rendering:** Using `OffscreenCanvas` in workers.

---

## Worker Use Cases

### Definitions

**Core Definition:** Worker use cases are the practical scenarios where Web Workers provide significant benefits.

### Common Use Cases

| Use Case | Description |
|----------|-------------|
| **Data processing** | Parsing large JSON/CSV files |
| **Image manipulation** | Applying filters, resizing, cropping |
| **Video processing** | Encoding, decoding, frame extraction |
| **Audio processing** | Audio analysis, effects |
| **Cryptography** | Hashing, encryption, key generation |
| **Machine learning** | Running models in the browser |
| **Game physics** | Simulating physics in the background |
| **Syntax highlighting** | Parsing code for editors |
| **Search indexing** | Building search indexes |
| **Real-time data** | Processing WebSocket/SSE streams |

### Annotated Code Example: Image Processing Worker

**main.js:**
```js
const worker = new Worker("image-worker.js");

// Send image data to the worker.
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
worker.postMessage({ type: "grayscale", imageData }, [imageData.data.buffer]);

worker.onmessage = (event) => {
  // Receive the processed image data.
  ctx.putImageData(event.data.imageData, 0, 0);
};
```

**image-worker.js:**
```js
self.onmessage = (event) => {
  const { type, imageData } = event.data;

  if (type === "grayscale") {
    const data = imageData.data;

    for (let i = 0; i < data.length; i += 4) {
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      data[i] = avg;     // Red
      data[i + 1] = avg; // Green
      data[i + 2] = avg; // Blue
    }

    self.postMessage({ imageData }, [data.buffer]);
  }
};
```

**Expected Output:** The image is converted to grayscale without blocking the UI.

### Real-World Cases

- **Photo editors:** Applying filters without freezing the UI.
- **Data dashboards:** Processing large datasets in the background.
- **Code editors:** Syntax highlighting and linting.
- **Games:** Physics and AI computations.

---

## Worker Performance

### Definitions

**Core Definition:** Worker performance refers to the efficiency and speed of worker-based computation, including communication overhead, memory usage, and parallelisation benefits.

### Performance Considerations

| Factor | Description |
|--------|-------------|
| **Startup cost** | Creating a worker has overhead (~5–50ms) |
| **Communication overhead** | Message passing has cost; use transferables for large data |
| **Memory** | Each worker has its own memory space |
| **Parallelism** | Multiple workers can run in parallel |
| **Transferables** | Zero-copy transfer of binary data |
| **SharedArrayBuffer** | Shared memory for high-performance scenarios |

### Annotated Code Example: Multiple Workers

```js
// Create a pool of workers.
const workerCount = navigator.hardwareConcurrency || 4;
const workers = Array.from({ length: workerCount }, () => new Worker("worker.js"));

// Distribute work across workers.
const tasks = [/* large array of tasks */];
const results = [];

workers.forEach((worker, index) => {
  worker.postMessage({ task: tasks[index] });
  worker.onmessage = (event) => {
    results[index] = event.data;
    if (results.length === tasks.length) {
      console.log("All tasks complete");
    }
  };
});
```

### Real-World Cases

- **Parallel processing:** Using multiple workers for CPU-bound tasks.
- **Large data:** Using transferables to avoid copying.
- **Real-time systems:** Minimising communication overhead.

---

## References

- MDN Web Docs – Web Workers API — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
- MDN Web Docs – Using Web Workers — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers
- MDN Web Docs – Worker — https://developer.mozilla.org/en-US/docs/Web/API/Worker
- MDN Web Docs – SharedWorker — https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker
- MDN Web Docs – Service Worker API — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- MDN Web Docs – Transferable objects — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects
- WHATWG – HTML Living Standard: Web Workers — https://html.spec.whatwg.org/multipage/workers.html
- W3C – Web Workers — https://www.w3.org/TR/workers/
- web.dev – Workers overview — https://web.dev/workers-overview/
- Chrome for Developers – OffscreenCanvas — https://developer.chrome.com/blog/offscreen-canvas/
- SitePoint – Web Workers: A Beginner's Guide — https://www.sitepoint.com/web-workers/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf