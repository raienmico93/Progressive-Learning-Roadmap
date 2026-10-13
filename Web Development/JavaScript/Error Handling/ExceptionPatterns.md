# Exception Patterns — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Exception patterns are established architectural strategies for detecting, responding to, and recovering from errors in software systems. They define how a program behaves when something goes wrong — whether it fails immediately, degrades gracefully, retries automatically, or reports to a monitoring service.

**Technical Definition:** Exception patterns are reusable design solutions that address error handling across different layers of an application. They encompass fault tolerance (retry, fallback), fault isolation (error boundaries), fault detection (fail fast, logging), and fault observation (monitoring). These patterns operate at multiple abstraction levels — from individual function calls to application-wide middleware — and are prescribed by software engineering best practices, resilience engineering literature (e.g., Nygard's *Release It!*), and industry standards such as the AWS Well-Architected Framework's Reliability Pillar.

**Beginner-Friendly Explanation:** When your code encounters a problem, you have choices about what to do. You can stop immediately and shout for help (fail fast), you can try again (retry), you can show something else instead (fallback), or you can just make sure a small part of the page doesn't break the rest (error boundary). These "patterns" are simply the tried-and-tested ways developers handle errors in real applications.

### Key Characteristics

- **Defence in depth:** Multiple patterns are layered so that if one fails, another catches the problem.
- **Separation of concerns:** Error detection, handling, reporting, and recovery are distinct responsibilities.
- **Configurability:** Patterns such as retry and fallback are parameterised (max attempts, delay strategies, fallback values).
- **Observability:** Patterns like logging and monitoring ensure errors leave a trace for diagnosis.
- **User-centric:** Patterns such as graceful degradation prioritise the end-user experience over technical purity.

### Prerequisites

- Understanding of JavaScript error types, `try...catch`, `throw`, and asynchronous programming (`async/await`, Promises).
- Familiarity with application architecture (layers, modules, frameworks such as React, Vue, or Node.js/Express).
- Basic knowledge of HTTP status codes and network failure modes.

### Related Programming Areas

- **Site Reliability Engineering (SRE):** Error budgets, incident response, post-mortems.
- **Distributed systems:** Circuit breakers, bulkheads, timeouts.
- **Frontend architecture:** React error boundaries, fallback UI, Suspense.
- **Backend architecture:** Express error-handling middleware, Node.js `process.on('uncaughtException')`.
- **Observability:** Structured logging, distributed tracing, error monitoring platforms.

### Core Concepts / Features

---

## Fail Fast

### Definitions

**Core Definition:** Fail fast is a design principle that advocates for immediately halting execution and surfacing an error as soon as an invalid condition is detected, rather than allowing the program to continue in an inconsistent state.

**Technical Definition:** Fail fast is implemented through input validation, precondition checks, and assertion statements that throw errors at the earliest possible point in a function or operation. It contrasts with defensive programming that attempts to silently recover from invalid input. In JavaScript, fail fast is commonly achieved by validating function arguments before any business logic executes, and by using `Promise.all()` (which rejects immediately on the first failure) rather than allowing partial failures to propagate silently.

**Beginner-Friendly Explanation:** "Fail fast" means: if something is wrong, stop immediately and tell me — don't keep going and cause a bigger mess later. If someone calls your function with a missing argument, throw an error right away instead of letting `undefined` silently corrupt your data.

### Purposes

- To prevent invalid data from propagating through the system.
- To reduce debugging time by surfacing errors at their source.
- To enforce contract requirements (preconditions) at function boundaries.
- To avoid corrupting application state with partial or invalid operations.

### Syntax Rules and Structure

Fail fast is a design principle rather than a single syntax construct. It is implemented through:

```js
// Argument validation at function entry:
function processOrder(order) {
  if (!order || typeof order.id !== "number") {
    throw new TypeError("processOrder requires an order object with a numeric id.");
  }
  // Business logic follows...
}
```

```js
// Using dedicated validation libraries (e.g., Zod, Joi, Ajv):
import { z } from "zod";
const OrderSchema = z.object({ id: z.number(), total: z.number().positive() });

function processOrder(order) {
  const validated = OrderSchema.parse(order); // Throws on invalid input
  // Business logic follows...
}
```

**Component Breakdown:**
- **Validation check:** An `if` statement or assertion that tests a precondition.
- **Immediate throw:** A `throw` statement that halts execution with a descriptive `Error`.
- **Validation library (optional):** A schema-based validator that throws structured errors.

**Syntax Rules:**
- Validation should occur before any side effects (mutations, I/O, network calls).
- Error messages should identify the parameter and the expected type/constraint.
- Use `TypeError` for type mismatches, `RangeError` for numeric bounds, and `Error` for generic contract violations.

**Constraints and Limitations:**
- Over-validation can clutter code; use dedicated validation libraries for complex schemas.
- Fail fast is inappropriate for operations where partial success is acceptable (e.g., batch processing where some items may fail independently).
- In asynchronous contexts, a thrown error inside a `Promise` executor rejects the promise rather than throwing synchronously.

### Annotated Code Examples

#### Example 1: Manual Fail-Fast Validation

```js
// Step 1: Define a function with strict preconditions.
function calculateDiscount(price, discountPercent) {
  // Step 2: Validate the first argument.
  if (typeof price !== "number" || price < 0) {
    throw new TypeError(`price must be a non-negative number. Received: ${price}`);
  }
  // Step 3: Validate the second argument.
  if (typeof discountPercent !== "number" || discountPercent < 0 || discountPercent > 100) {
    throw new RangeError(`discountPercent must be between 0 and 100. Received: ${discountPercent}`);
  }
  // Step 4: Business logic executes only after all preconditions pass.
  return price * (1 - discountPercent / 100);
}

// Step 5: Valid call succeeds.
console.log(calculateDiscount(100, 20)); // 80

// Step 6: Invalid call fails immediately.
try {
  calculateDiscount(100, 150);
} catch (err) {
  console.log(err.name);    // "RangeError"
  console.log(err.message); // "discountPercent must be between 0 and 100. Received: 150"
}
```

**Expected Output:**
```
80
RangeError
discountPercent must be between 0 and 100. Received: 150
```

**Why This Output:** The valid call passes both validation checks and returns the discounted price. The invalid call fails at the second check because `150` exceeds the `0–100` range. The `RangeError` is thrown before any business logic executes, preventing an incorrect discount calculation.

#### Example 2: Fail Fast with `Promise.all()`

```js
// Step 1: Create promises where one fails immediately.
const promises = [
  fetch("/api/users").then((r) => r.json()),      // May take time
  Promise.reject(new Error("Database unavailable")), // Fails immediately
  fetch("/api/orders").then((r) => r.json()),     // May take time
];

// Step 2: Promise.all() rejects as soon as any promise rejects.
Promise.all(promises)
  .then((results) => console.log("All succeeded:", results))
  .catch((err) => {
    console.log("Fail-fast rejection:", err.message);
  });
```

**Expected Output:**
```
Fail-fast rejection: Database unavailable
```

**Why This Output:** `Promise.all()` implements fail-fast behaviour: it rejects with the reason of the **first** promise to reject, without waiting for the other promises to settle. The other promises continue executing in the background but their results are discarded.

### Real-World Cases

- **API request validation:** Express middleware validates `req.body` against a schema before passing it to controllers. If validation fails, a `400 Bad Request` is returned immediately.
- **Financial transactions:** A payment function validates the amount, currency, and account balance before initiating any transfer, preventing partial or invalid transactions.
- **Configuration loading:** Application startup validates that all required environment variables are present, throwing an `Error` and preventing the app from starting in a misconfigured state.

---

## Graceful Degradation

### Definitions

**Core Definition:** Graceful degradation is a design principle that allows a system to continue operating with reduced functionality when a component or feature fails, rather than crashing entirely.

**Technical Definition:** Graceful degradation in JavaScript is implemented through fallback mechanisms — cached data, alternative API endpoints, simplified UI, or non-essential feature removal — that maintain core functionality when optional or non-critical components fail. It is closely related to fault tolerance and is a key requirement in resilience engineering frameworks such as the AWS Well-Architected Framework.

**Beginner-Friendly Explanation:** Graceful degradation means: if something breaks, keep the important parts working and drop the less important parts. If a recommendation widget fails to load, don't show a blank page — just hide the widget and let the user continue reading the article.

### Purposes

- To maintain core functionality when non-critical features fail.
- To improve user experience by avoiding full-page crashes.
- To isolate failures so that one broken component does not affect the entire application.
- To provide a controlled, predictable failure mode instead of an unpredictable crash.

### Syntax Rules and Structure

Graceful degradation is implemented through:

```js
// Pattern 1: Try/catch with fallback content:
try {
  const recommendations = await fetchRecommendations();
  renderRecommendations(recommendations);
} catch {
  renderRecommendations([]); // Render empty state instead of crashing
}

// Pattern 2: Optional chaining for safe property access:
const city = user?.address?.city ?? "Unknown";

// Pattern 3: Feature detection with polyfill fallback:
if ("IntersectionObserver" in window) {
  // Use IntersectionObserver
} else {
  // Fall back to scroll event listeners
}
```

**Component Breakdown:**
- **Primary operation:** The feature that may fail (network request, API call, rendering).
- **Fallback behaviour:** The degraded but functional alternative (empty state, cached data, simplified UI).
- **Isolation boundary:** A `try...catch` block or error boundary that prevents the failure from propagating.

**Syntax Rules:**
- The fallback should not itself depend on the failed operation.
- Fallback content should be meaningful to the user (e.g., "Recommendations unavailable" rather than a blank space).
- Use the nullish coalescing operator (`??`) and optional chaining (`?.`) for safe fallback values.

**Constraints and Limitations:**
- Graceful degradation can mask underlying problems if not accompanied by logging and monitoring.
- Determining which features are "non-critical" requires domain knowledge and user research.
- Overuse of fallbacks can lead to silent failures that are difficult to diagnose.

### Annotated Code Examples

#### Example 1: Degraded Dashboard Loading

```js
// Step 1: Simulate fetching multiple dashboard widgets.
async function loadDashboard() {
  const widgets = {};

  // Step 2: Try to load each widget independently.
  try {
    widgets.stats = await fetchStats();
  } catch {
    widgets.stats = { error: "Statistics unavailable." };
  }

  try {
    widgets.chart = await fetchChart();
  } catch {
    widgets.chart = { error: "Chart could not be loaded." };
  }

  // Step 3: The dashboard renders with whatever data was available.
  return widgets;
}

// Step 4: Simulated fetch functions.
async function fetchStats() { throw new Error("Stats service down"); }
async function fetchChart() { return { data: [1, 2, 3] }; }

// Step 5: Load the dashboard.
loadDashboard().then((widgets) => {
  console.log(widgets.stats); // { error: "Statistics unavailable." }
  console.log(widgets.chart); // { data: [1, 2, 3] }
});
```

**Expected Output:**
```
{ error: 'Statistics unavailable.' }
{ data: [ 1, 2, 3 ] }
```

**Why This Output:** Each widget is loaded independently. The statistics widget fails, but its failure is contained; the chart widget still loads successfully. The dashboard degrades gracefully by showing an error message for the failed widget while displaying the successful one.

#### Example 2: Cached Data as Fallback

```js
// Step 1: In-memory cache.
const cache = new Map();

// Step 2: Function that tries the network, falls back to cache.
async function getData(key) {
  try {
    const response = await fetch(`/api/data/${key}`);
    const data = await response.json();
    cache.set(key, data); // Update cache on success
    return data;
  } catch (networkError) {
    // Step 3: Fall back to cached data if available.
    if (cache.has(key)) {
      console.warn("Using cached data due to network failure.");
      return cache.get(key);
    }
    // Step 4: If no cache, re-throw the error.
    throw networkError;
  }
}
```

**Expected Output (first call, network succeeds):**
```
(no output — data fetched and cached)
```

**Expected Output (second call, network fails):**
```
Using cached data due to network failure.
{cached data}
```

**Why This Output:** On the first call, the network request succeeds and the data is cached. On the second call, if the network is unavailable, the function falls back to the cached data, allowing the application to continue functioning with stale but usable data.

### Real-World Cases

- **E-commerce product pages:** If the recommendation API fails, the product details still render; only the recommendation section is hidden or shows a placeholder.
- **News websites:** If the commenting system is down, the article content remains accessible.
- **Progressive Web Apps (PWAs):** Service workers serve cached assets when the network is unavailable, enabling offline functionality.

---

## Error Boundaries

### Definitions

**Core Definition:** An error boundary is a component or construct that catches errors thrown by its descendants, preventing them from propagating further up the component tree.

**Technical Definition:** In React, an error boundary is a class component that implements either `static getDerivedStateFromError()` or `componentDidCatch()` (or both). It catches errors that occur during rendering, in lifecycle methods, and in constructors of the whole tree below them. React error boundaries do not catch errors in event handlers, asynchronous code (`setTimeout`, `fetch`), or server-side rendering. In Aurelia, error boundaries are implemented as services that capture errors globally via `window.addEventListener('error')` and `window.addEventListener('unhandledrejection')`.

**Beginner-Friendly Explanation:** An error boundary is like a safety net around a section of your UI. If something inside that section breaks, the net catches the error and shows a friendly fallback message instead of letting the whole page crash.

### Purposes

- To isolate errors to a specific subtree of the component tree.
- To prevent a single component failure from crashing the entire application.
- To display user-friendly fallback UI when a component fails.
- To log error information for debugging and monitoring.

### Syntax Rules and Structure

**React Class Component (Traditional):**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    // Log the error to a monitoring service
    console.error("Error boundary caught:", error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Component Breakdown:**
- `getDerivedStateFromError(error)`: A static lifecycle method that updates state to trigger the fallback UI.
- `componentDidCatch(error, info)`: An instance method for side effects (logging, reporting).
- `this.state.hasError`: A boolean flag that determines whether to render the fallback.

**Syntax Rules:**
- Error boundaries must be class components (in React); function components cannot be error boundaries without third-party libraries.
- Only errors in the component tree below the boundary are caught; the boundary itself cannot catch its own errors.
- Error boundaries do not catch asynchronous errors; these require additional bridging mechanisms.

**Constraints and Limitations:**
- React error boundaries do not catch errors in event handlers, `setTimeout`, `requestAnimationFrame`, `Promise` rejections, or server-side rendering.
- `try/catch` cannot catch render-time errors in React; only error boundaries can.
- Placing a single top-level error boundary gives a poor user experience because the entire app is replaced by a fallback.

### Annotated Code Examples

#### Example 1: React Error Boundary with Fallback UI

```jsx
// Step 1: Define the error boundary component.
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Step 2: Update state so the next render shows the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Step 3: Log the error to an external service.
    console.error("Component error:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Step 4: Render fallback UI.
      return (
        <div className="error-fallback">
          <h2>Something went wrong.</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }
    // Step 5: Render children normally.
    return this.props.children;
  }
}

// Step 6: Use the boundary around a risky component.
function App() {
  return (
    <ErrorBoundary>
      <RiskyWidget />
    </ErrorBoundary>
  );
}
```

**Expected Output (when `RiskyWidget` throws):**
```
(Error is caught; fallback UI with "Something went wrong." and a "Try again" button is displayed.)
```

**Why This Output:** When `RiskyWidget` throws during rendering, `getDerivedStateFromError` updates the boundary's state, causing `render()` to return the fallback UI. `componentDidCatch` logs the error. The rest of the application (outside the boundary) continues to function normally.

#### Example 2: Three-Layer Boundary Hierarchy

```jsx
// Layer 1: Global boundary (catches everything).
<GlobalErrorBoundary>
  {/* Layer 2: Route-level isolation. */}
  <RouteErrorBoundary>
    <DashboardPage />
  </RouteErrorBoundary>
  <RouteErrorBoundary>
    <SettingsPage />
  </RouteErrorBoundary>
</GlobalErrorBoundary>
```

**Architecture Explanation:** The three-layer hierarchy — Global, Route, and Feature boundaries — provides progressively finer isolation. A crash in `/dashboard` does not affect `/settings` because each route has its own boundary. A broken chart widget fails silently or shows an inline fallback, leaving the rest of the page functional.

### Real-World Cases

- **React SaaS applications:** Global, route-level, and feature-level boundaries prevent a single broken widget from taking down the entire page.
- **Next.js applications:** The `error.js` file convention automatically wraps route segments in error boundaries.
- **Aurelia applications:** A global `ErrorBoundaryService` captures unhandled errors and provides recovery options to the user.

---

## Retry Logic

### Definitions

**Core Definition:** Retry logic is a pattern that automatically re-attempts a failed operation a specified number of times, often with increasing delays between attempts, to recover from transient failures.

**Technical Definition:** Retry logic with exponential backoff is a standard resilience pattern in distributed systems. The delay between retries increases exponentially (e.g., 1s, 2s, 4s, 8s) to avoid overwhelming a recovering service. Jitter — randomised variation in the delay — is added to prevent the "thundering herd" problem where many clients retry simultaneously. The AWS SDK for JavaScript implements exponential backoff with full jitter for HTTP requests.

**Beginner-Friendly Explanation:** Sometimes a network request fails for a temporary reason — like a brief Wi-Fi drop or a server that's momentarily busy. Instead of giving up immediately, you wait a short time and try again. If it fails again, you wait a bit longer and try again. This is retry logic.

### Purposes

- To recover from transient network failures without user intervention.
- To improve reliability in distributed systems where temporary failures are common.
- To reduce the number of user-visible errors caused by momentary glitches.
- To handle rate-limiting responses (HTTP 429) by backing off and retrying later.

### Syntax Rules and Structure

```js
async function retry(fn, options = {}) {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    factor = 2,
    jitter = true,
    shouldRetry = () => true,
  } = options;

  let lastError;
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn(attempt);
    } catch (err) {
      lastError = err;
      if (attempt === maxRetries || !shouldRetry(err)) {
        throw err;
      }
      let delay = baseDelay * Math.pow(factor, attempt);
      if (jitter) {
        delay = delay * (0.5 + Math.random() * 0.5); // ±50% jitter
      }
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
  throw lastError;
}
```

**Component Breakdown:**
- `maxRetries`: Maximum number of re-attempts (not counting the initial attempt).
- `baseDelay`: Initial delay in milliseconds.
- `factor`: Multiplier for exponential backoff.
- `jitter`: Whether to add random variation to the delay.
- `shouldRetry`: A predicate function that determines whether a specific error is retryable.
- `attempt`: The current attempt number (zero-indexed).

**Syntax Rules:**
- Retry should only be applied to **idempotent** operations (operations that can be safely repeated without side effects).
- Non-retryable errors (e.g., `400 Bad Request`, `401 Unauthorized`) should not be retried.
- The total time budget should be bounded to avoid infinite retries.

**Constraints and Limitations:**
- Retrying non-idempotent operations (e.g., `POST` requests that create resources) can cause duplicate side effects.
- Without jitter, synchronised retries from many clients can overwhelm a recovering service.
- Retry logic can mask persistent failures if not accompanied by monitoring and alerting.

### Annotated Code Examples

#### Example 1: Basic Retry with Exponential Backoff

```js
// Step 1: Simulate an unreliable API call.
let attemptCount = 0;
async function unreliableApi() {
  attemptCount++;
  if (attemptCount < 3) {
    throw new Error(`Transient failure #${attemptCount}`);
  }
  return { data: "Success after retries" };
}

// Step 2: Retry wrapper.
async function withRetry(fn, maxRetries = 3, baseDelay = 500) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === maxRetries) throw err;
      const delay = baseDelay * Math.pow(2, attempt); // 500ms, 1s, 2s...
      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
      await new Promise((r) => setTimeout(r, delay));
    }
  }
}

// Step 3: Execute with retry.
withRetry(unreliableApi)
  .then((result) => console.log(result.data))
  .catch((err) => console.error("All retries failed:", err.message));
```

**Expected Output:**
```
Attempt 1 failed. Retrying in 500ms...
Attempt 2 failed. Retrying in 1000ms...
Success after retries
```

**Why This Output:** The first two attempts fail, triggering retries with exponential backoff (500ms then 1000ms). The third attempt succeeds and returns the result. If all attempts fail, the final error is thrown.

#### Example 2: Retry with Conditional Logic

```js
// Step 1: Define retry conditions.
const retryConditions = {
  networkError: (err) => err.message.includes("network") || err.name === "NetworkError",
  httpStatus: (codes) => (err) => err.status && codes.includes(err.status),
};

// Step 2: Retry only on 5xx errors, not on 4xx.
async function fetchWithRetry(url) {
  return withRetry(
    () => fetch(url).then((res) => {
      if (!res.ok) {
        const err = new Error(`HTTP ${res.status}`);
        err.status = res.status;
        throw err;
      }
      return res.json();
    }),
    { shouldRetry: retryConditions.httpStatus([500, 502, 503, 504]) }
  );
}
```

**Expected Output (on a 503 response):** The request is retried up to `maxRetries` times.
**Expected Output (on a 400 response):** The request is **not** retried; the error is thrown immediately.

**Why This Output:** The `shouldRetry` predicate checks the error's status code. Only server errors (5xx) are retried because they are likely transient. Client errors (4xx) indicate a problem with the request itself and will not be resolved by retrying.

### Real-World Cases

- **API clients:** Retrying failed HTTP requests to external services with exponential backoff and jitter.
- **Database connections:** Retrying connection attempts when a database is temporarily unavailable.
- **Message queues:** Retrying failed message processing with backoff to avoid overwhelming the consumer.
- **Payment gateways:** Retrying idempotent payment verification requests when the gateway times out.

---

## Fallback Values

### Definitions

**Core Definition:** A fallback value is a default value returned or used when the primary value is unavailable, `null`, `undefined`, or an error occurs.

**Technical Definition:** In JavaScript, fallback values are implemented through the logical OR operator (`||`), the nullish coalescing operator (`??`), default function parameters, destructuring defaults, and try/catch wrappers. The `||` operator returns the right-hand operand if the left-hand operand is falsy (including `0`, `""`, `null`, `undefined`, `false`, and `NaN`), while `??` returns the right-hand operand only if the left-hand operand is `null` or `undefined`.

**Beginner-Friendly Explanation:** A fallback value is what you use instead of the real value when the real value isn't available. If `user.name` is missing, use `"Guest"`. If the API returns nothing, use cached data. It's like a spare tyre — you only use it when the main one fails.

### Purposes

- To prevent `TypeError` and `ReferenceError` from `null`/`undefined` property access.
- To provide sensible defaults when optional data is missing.
- To maintain application functionality when primary data sources fail.
- To simplify code by avoiding verbose `if` checks for missing values.

### Syntax Rules and Structure

```js
// Pattern 1: Logical OR (for any falsy value):
const name = userInput || "Anonymous";

// Pattern 2: Nullish coalescing (for null/undefined only):
const count = apiResponse.count ?? 0;

// Pattern 3: Default function parameters:
function greet(name = "Guest") {
  return `Hello, ${name}`;
}

// Pattern 4: Destructuring defaults:
const { city = "Unknown", country = "Unknown" } = userAddress;

// Pattern 5: Optional chaining with fallback:
const street = user?.address?.street ?? "No street provided";
```

**Component Breakdown:**
- `||`: Returns the right operand if the left is falsy.
- `??`: Returns the right operand if the left is `null` or `undefined`.
- `= defaultValue`: Assigns a default if the argument is `undefined`.
- `?.`: Short-circuits property access if the base is `null` or `undefined`.

**Syntax Rules:**
- `??` cannot be mixed with `||` or `&&` without parentheses.
- Default parameters apply only when the argument is `undefined`, not when it is `null`.
- Destructuring defaults apply when the property is `undefined`.

**Constraints and Limitations:**
- `||` treats `0` and `""` as falsy, which may be unintended for numeric or string values.
- Fallback values can mask missing data problems if not logged.
- Optional chaining does not work on the left-hand side of an assignment.

### Annotated Code Examples

#### Example 1: `??` vs `||` Behaviour

```js
// Step 1: Demonstrate the difference.
const count = 0;

// Using || — 0 is falsy, so the fallback is used.
console.log(count || 10); // 10 (WRONG if 0 is a valid value)

// Using ?? — 0 is not null/undefined, so it is kept.
console.log(count ?? 10); // 0 (CORRECT)
```

**Expected Output:**
```
10
0
```

**Why This Output:** The `||` operator treats `0` as falsy and returns the fallback `10`. The `??` operator only falls back when the value is `null` or `undefined`, so `0` is preserved.

#### Example 2: Function Parameter Fallbacks

```js
// Step 1: Define a function with default parameters.
function createUser({ name = "Anonymous", role = "viewer", active = true } = {}) {
  return { name, role, active };
}

// Step 2: Call with no arguments.
console.log(createUser());
// { name: 'Anonymous', role: 'viewer', active: true }

// Step 3: Call with partial arguments.
console.log(createUser({ name: "Alice" }));
// { name: 'Alice', role: 'viewer', active: true }

// Step 4: Call with explicit null (null is NOT replaced by default).
console.log(createUser({ name: null }));
// { name: null, role: 'viewer', active: true }
```

**Expected Output:**
```
{ name: 'Anonymous', role: 'viewer', active: true }
{ name: 'Alice', role: 'viewer', active: true }
{ name: null, role: 'viewer', active: true }
```

**Why This Output:** Default parameters apply when the property is `undefined`. Passing `null` explicitly overrides the default because `null` is not `undefined`. This is a common source of confusion and should be handled explicitly if `null` values are possible.

### Real-World Cases

- **User profiles:** Displaying `"Anonymous"` when the user's name is not set.
- **Configuration loading:** Using default settings when environment variables are missing.
- **API responses:** Using empty arrays (`[]`) as fallbacks for list endpoints that return `null`.
- **Form inputs:** Pre-filling fields with saved values or defaults.

---

## Error Logging

### Definitions

**Core Definition:** Error logging is the practice of recording error events, along with contextual metadata, to a persistent store for later analysis, debugging, and auditing.

**Technical Definition:** Structured error logging involves emitting log entries as machine-parseable JSON objects containing fields such as timestamp, severity level, error name, message, stack trace, and request context (user ID, request ID, endpoint). Unlike unstructured `console.log` statements, structured logs are searchable, filterable, and compatible with log aggregation tools such as the ELK stack (Elasticsearch, Logstash, Kibana), Graylog, and cloud-native services. Popular Node.js logging libraries include Pino (performance-focused) and Winston (feature-rich).

**Beginner-Friendly Explanation:** Error logging means writing down what went wrong so you can look at it later. Instead of just `console.log("error")`, you write a detailed record with the time, the error type, the message, and what the user was doing. This helps you fix bugs faster.

### Purposes

- To create an audit trail of errors for post-mortem analysis.
- To provide diagnostic context for debugging production issues.
- To enable alerting and dashboards based on error frequency and severity.
- To satisfy compliance and regulatory requirements for error reporting.

### Syntax Rules and Structure

```js
// Basic structured logging with console.error:
console.error({
  timestamp: new Date().toISOString(),
  level: "error",
  name: err.name,
  message: err.message,
  stack: err.stack,
  userId: currentUser?.id,
  requestId: req.id,
});
```

**Component Breakdown:**
- `timestamp`: ISO 8601 timestamp of the error event.
- `level`: Severity level (`error`, `fatal`, `warn`, etc.).
- `name`: The error type (e.g., `TypeError`, `NetworkError`).
- `message`: The human-readable error description.
- `stack`: The stack trace for debugging.
- `userId` / `requestId`: Contextual identifiers for correlation.

**Syntax Rules:**
- Log at the appropriate severity level: `error` for handled errors, `fatal` for unhandled exceptions.
- Include a correlation ID (request ID, trace ID) to link related log entries.
- Redact sensitive data (passwords, tokens, PII) before logging.

**Constraints and Limitations:**
- Logging every error at `fatal` level causes alert fatigue; reserve `fatal` for process-threatening errors.
- `console.log` in production can degrade performance and leak information; use a logging library with appropriate transports.
- Stack traces may be misleading if source maps are not uploaded for minified production code.

### Annotated Code Examples

#### Example 1: Structured Error Logging with Pino

```js
// Step 1: Import and configure Pino.
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL || "info",
  base: { service: "user-api", version: "1.2.0" },
  redact: ["req.headers.authorization", "password", "creditCard"],
});

// Step 2: Log an error with context.
try {
  await processPayment(userId, amount);
} catch (err) {
  logger.error({
    err,
    orderId: "789",
    step: "payment",
    userId: userId,
  }, "Payment processing failed");
}
```

**Expected Output (JSON log entry):**
```json
{"level":50,"time":1699999999999,"service":"user-api","version":"1.2.0","err":{"type":"Error","message":"Payment gateway timeout","stack":"..."},"orderId":"789","step":"payment","userId":"abc123","msg":"Payment processing failed"}
```

**Why This Output:** The log entry is a single JSON object containing the error (with type, message, and stack), the order ID, the step in the process, and the user ID. This makes it searchable by any field in a log aggregation tool.

#### Example 2: Child Loggers for Request Context

```js
// Step 1: Create a request-scoped logger.
const reqLogger = logger.child({
  requestId: req.id,
  userId: req.user?.id,
});

// Step 2: Log throughout the request lifecycle.
reqLogger.info({ path: req.path }, "Request started");
reqLogger.error({ err, statusCode: 500 }, "Request failed");
reqLogger.info({ statusCode: 200, durationMs: 45 }, "Request completed");
```

**Expected Output:** Every log entry automatically includes the `requestId` and `userId`, allowing all logs for a single request to be correlated.

**Why This Output:** Child loggers inherit the parent's configuration and add bound context. This avoids repeating `requestId` and `userId` in every log call and ensures consistency.

### Real-World Cases

- **Microservices:** Centralised logging with correlation IDs to trace requests across services.
- **E-commerce:** Logging payment failures with order IDs for reconciliation and customer support.
- **SaaS applications:** Logging user actions and errors to meet audit and compliance requirements.

---

## Error Monitoring

### Definitions

**Core Definition:** Error monitoring is the continuous, automated collection, aggregation, and alerting of errors occurring in production applications.

**Technical Definition:** Error monitoring platforms (Sentry, Bugsnag, Rollbar, Datadog RUM) integrate with applications via SDKs that automatically capture unhandled exceptions, unhandled promise rejections, and manually reported errors. They provide features such as stack trace aggregation, release tracking, breadcrumbs, source map support, session replay, and alerting. Sentry is the industry standard, with approximately 4 million weekly downloads and integrations for every major JavaScript framework. Bugsnag focuses on stability monitoring and release health, while Rollbar emphasises real-time debugging with powerful grouping algorithms.

**Beginner-Friendly Explanation:** Error monitoring is like having a security camera for your application's errors. When something breaks in production, the monitoring service records exactly what happened, where it happened, and which users were affected — and then alerts you so you can fix it quickly.

### Purposes

- To detect production errors before users report them.
- To aggregate and prioritise errors by frequency and impact.
- To provide detailed context (stack trace, breadcrumbs, user info) for faster debugging.
- To track error rates across releases and measure the impact of fixes.

### Syntax Rules and Structure

**Sentry Integration (Browser/React):**

```js
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  integrations: [Sentry.replayIntegration()],
});
```

**Component Breakdown:**
- `dsn`: The Data Source Name that identifies your Sentry project.
- `environment`: The deployment environment (`production`, `staging`).
- `tracesSampleRate`: The percentage of transactions to trace for performance monitoring.
- `replaysOnErrorSampleRate`: The percentage of error sessions to record as replays.

**Syntax Rules:**
- Initialise the SDK as early as possible in the application lifecycle.
- Upload source maps during the build process to enable readable stack traces.
- Use `Sentry.captureException(error)` for manual error reporting in catch blocks.
- Configure `beforeSend` to filter out expected errors and redact sensitive data.

**Constraints and Limitations:**
- Error monitoring adds network overhead; sample rates should be tuned to balance visibility and cost.
- Source maps must be uploaded during the build for production stack traces to be readable.
- Error monitoring platforms have rate limits and pricing tiers; high-volume applications need careful configuration.

### Annotated Code Examples

#### Example 1: Manual Error Capture with Context

```js
import * as Sentry from "@sentry/react";

async function processPayment(userId, amount) {
  try {
    await chargeCard(userId, amount);
  } catch (err) {
    // Step 1: Add context to the error before capturing.
    Sentry.setContext("payment", { userId, amount, currency: "USD" });
    Sentry.setTag("payment_provider", "stripe");
    // Step 2: Capture the exception.
    Sentry.captureException(err);
    // Step 3: Re-throw for the caller to handle.
    throw err;
  }
}
```

**Expected Output (in Sentry dashboard):** The error appears with tags `payment_provider=stripe`, context `{ userId, amount, currency }`, and a full stack trace.

**Why This Output:** The context and tags enrich the error event, making it easier to filter and debug in the Sentry dashboard. The error is captured but still propagates to the caller for application-level handling.

#### Example 2: Sentry with Source Maps and Release Tracking

```js
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  release: `my-app@${process.env.APP_VERSION}`,
  environment: process.env.NODE_ENV,
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration({ maskAllText: true }),
  ],
  tracesSampleRate: 0.1,
});
```

**Expected Output:** Errors are tagged with the release version, allowing you to identify which release introduced the bug. Source maps uploaded during the build ensure that stack traces point to the original source code rather than minified bundles.

**Why This Output:** Release tracking enables Sentry to group errors by release and show whether a fix has resolved the issue. Source maps bridge the gap between minified production code and readable source code.

### Real-World Cases

- **SaaS platforms:** Monitoring errors across thousands of user sessions to identify widespread issues.
- **E-commerce:** Tracking payment and checkout errors to protect revenue.
- **Mobile web apps:** Using session replay to understand the user's actions leading up to an error.

---

## References

- MDN Web Docs – try...catch — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch
- MDN Web Docs – Promise.all() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN Web Docs – Default parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- MDN Web Docs – Nullish coalescing operator (??) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing
- MDN Web Docs – Optional chaining (?.) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
- React Documentation – Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React Documentation – error-boundaries lint rule — https://react.dev/reference/eslint-plugin-react-hooks/lints/error-boundaries
- Aurelia 2 Documentation – Error Handling Patterns — https://docs.aurelia.io/advanced-scenarios/error-handling-patterns
- Node.js Best Practices – Fail Fast and Validation — https://github.com/goldbergyoni/nodebestpractices
- Node.js Best Practices – Catch Unhandled Promise Rejections — https://github.com/goldbergyoni/nodebestpractices
- AWS SDK for JavaScript v2 – Exponential Backoff — https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/retries.html
- AWS Well-Architected Framework – Reliability Pillar — https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html
- Sentry Documentation – JavaScript — https://docs.sentry.io/platforms/javascript/
- Pino Documentation — https://getpino.io/
- Winston Documentation — https://github.com/winstonjs/winston
- Bugsnag Documentation — https://docs.bugsnag.com/
- Rollbar Documentation — https://docs.rollbar.com/
- PkgPulse – Best Error Tracking Libraries for JavaScript in 2026 — https://www.pkgpulse.com/guides/best-error-tracking-javascript-2026
- Nygard, M. T. – *Release It!* (Pragmatic Bookshelf) — (ISBN 978-1680502398)
- MDN Web Docs – Error — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error